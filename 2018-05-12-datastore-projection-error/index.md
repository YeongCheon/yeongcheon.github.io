# datastore에서 projection query 실행 시 time.Time이 int64로 반환되는 문제 해결하기


## 요구사항
datastore에 내 프로필을 방문한 사용자의 ID와 방문시간을 각각 `string`, `time.Time` 형식으로 저장하고 있다. 구조체로 보면 다음과 같다.

``` go
type Visitor struct {
	UserId string //조회한 프로필의 Id
	VisitorId string //방문자 Id
	VisitTime time.Time
}
```

이 데이터를 단순히 조회하면 한 사용자가 나를 여러번 방문할 경우 결과 리스트에 그 방문자가 여러번 출력된다. 나는 동일한 사용자가 여러번 반복해도 가장 최근에 방문한 기록만 보길 원했다. 즉 datastore에서 SQL 문법의 **group by**를 사용하고 싶었다.  내 요구사항을 SQL로 표현하자면 아래와 같다.

```sql
SELECT VisitorId, MAX(VisitTime) WHERE UserId = ? GROUP BY VisitorId Order by VisitTime desc
```

## Projection Query 사용하기
[datastore client](https://godoc.org/cloud.google.com/go/datastore)에서 Group by를 사용하기 위해선 [Projection Query](https://cloud.google.com/appengine/docs/standard/go/datastore/projectionqueries)를 사용해야 한다. 앞에 링크된 문서를 참고해서 아래와 같은 코드를 작성했다.

```go
dsClient, _ := datastore.NewClient(ctx, projectID)

query := datastore.NewQuery(userDao.VisitorEntity).
		Filter("UserId =", userId).
		Project("VisitorId", "VisitTime").
		DistinctOn("VisitorId").
		Order("-VisitTime").Order("VisitorId")
		
visitorList := make([]models.Visitor, 0, 1)
_, err := dsClient.GetAll(ctx, query, &visitorList)
```

자세하게 설명하기에 앞서 미리 얘기하자면 이 쿼리가 맞는 쿼리인지는 솔직히 확신이 서지 않는다. 일단 더미데이터를 이용해서 테스트를 했을때 잘 나오기는 했는데 VisitTime이 항상 최신 내용(Max)으로 나올지 자신이 없다. 추 후 관련 내용을 좀 더 찾아보고 업데이트 하겠다.

아무튼 위 코드를 실행하면 아래와 같은 에러메세지를 내뿜는다.
```text
datastore: cannot load field "VisitTime" into a "model.Visitor": type mismatch: int versus time.Time
```

내용을 살펴보면 VisitTime 필드의 내용이 int형인데 time.Time 형식으로 받으려고 했기 때문에 에러가 났다는 내용이다. 하지만 데이터를 입력할 때도 동일한 구조체를 사용했고 실제로 클라우드 콘솔에서 데이터를 조회해봐도 날짜형식으로 이쁘게 데이터가 입력되어 있었다. 열심히 검색을 해보니 [나와 같은 문제를 겪은 분이 있었다](https://github.com/GoogleCloudPlatform/google-cloud-python/issues/1770).

원인을 설명하자면 `time.Time`를 datastore 내부에 저장할 때 timestamp 형식으로 저장되는데 이게 projection query를 사용하면 저장되어 있는 int값 그대로 나온다는 것이다. 개발진들이 의도한 것인지 버그인지는 모르겠지만 일단 나름대로 해결책을 찾았다. 아래의 코드를 보자.

```go

func (visitor *Visitor) Save() ([]datastore.Property, error) {
	return []datastore.Property{
		{
			Name:  "UserId",
			Value: visitor.UserId,
		},
		{
			Name:  "VisitorId",
			Value: visitor.VisitorId,
		},
		{
			Name:  "VisitTime",
			Value: visitor.VisitTime,
		},
	}, nil
}

func (visitor *Visitor) Load(ps []datastore.Property) error {
	for idx, property := range ps {
		if property.Name == "VisitTime" {
			if s, isInt := property.Value.(int64); isInt {
				t := time.Unix(0, int64(s)*int64(time.Microsecond))
				ps[idx].Value = t
			}
		}
	}

	return datastore.LoadStruct(visitor, ps)
}

```


위 코드는 [공식 예제](https://cloud.google.com/appengine/docs/standard/go/datastore/reference#hdr-The_PropertyLoadSaver_Interface)를 참고해서 만든것이다. `Visitor` 구조체에 [PropertyLoadSaver](https://cloud.google.com/appengine/docs/standard/go/datastore/reference#PropertyLoadSaver) 인터페이스를 구현해놓은 것인데 위 코드에서는 `Load` 함수를 중점으로 보면 된다. 간단한게 설명하자면 `datastore.LoadStruct` 함수는 구조체의 각 필드이름과 `property.Name`의 값을 비교에서 동일할 경우 해당 필드에 `property.Value` 값을 넣는다. 이 때 타입이 서로 다를 경우 아까와 같은 에러가 발생하기 때문에 반복문을 이용해 VisitTime을 찾아 직접 timestamp를 `time.Time` 형식으로 바꿔준다. 이 후 다시 쿼리문을 실행시켜보면 정상적으로 작동한다. 간혹 Index 관련 에러가 발생할 수 있는데 그건 [이전 포스트](/2018/04/29/gae-datastore-indexes/)를 참고하자.

