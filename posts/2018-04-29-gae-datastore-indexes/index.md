# Googlde App Engine(GAE) datastore index 관련 에러 해결하기


## 들어가며
google app engine을 이용해 REST API를 작성하던 중 발생했던 datastore의 index 관련 이슈와 그 해결법에 대해 기록한다. [참고](https://cloud.google.com/datastore/docs/concepts/indexes), [여기도 참고](https://groups.google.com/forum/#!topic/google-appengine-java/zKjWwNjcTsE)

## 에러발생 상황
내 프로필을 조회한 사람들의 목록을 반환해주는 API를 작성했다. 코드는 딱히 별거 없고 그냥 datastore에 있는 데이터들을 불러오는 내용이다. 핵심코드는 아래와 같다.
``` golang 
q := datastore.NewQuery(userDao.VisitorEntity).Filter("UserId=", userId).Order("-VisitTime")
```
UserId 필드의 값을 기준으로 필터링을 하고 VisitTime을 최근순으로 정렬하는 간단한 코드다. 위 쿼리문을 베이스로 한 API는 로컬테스트는 무사히 통과하고 배포까지 정상적으로 완료, 이 후 클라우드에 올라간 API를 테스트 했는데 다음과 같은 에러로그를 내뿜었다.

```
API error 4 (datastore_v3: NEED_INDEX): no matching index found. recommended index is:
- kind: kindname
  properties:
  - name: fieldName1
  - name: FiendName2
    direction: desc"
```

## 해결방안

위 에러메세지는 배포 명령 실행 시 파라메터를 누락시켰기 때문에 발생하는 메세지이다. 코드 작성 후 로컬 dev_appserver.py 를 이용해 로컬에서 서버를 실행하면 `index.yaml` 파일이 자동으로 생성된다. 이 파일은 맨 위에 작성된 쿼리문을 바탕으로 **인덱싱 정보들을 담아놓은 파일**인데 deploy 명령어를 쓸 때 직접 명시를 해주어야 한다. 기존에 내가 쓰던 deploy 명령어는 아래와 같다.

``` shell
gcloud app deploy
```

**위 명령어를 이용해 배포하면 제대로 동작하지 않는다.** 아래의 명령어를 살펴보자.

``` shell
gcloud app deploy index.yaml
```

위와 같이 자동 생성된 `index.yaml` 파일을 직접 명시해주지 않으면 배포될 때 서버에 함께 올라가지 않는다.[공식문서](https://cloud.google.com/sdk/gcloud/reference/app/deploy). 공식 문서의 내용을 굳이 설명하자면 depoy 명령어 뒤에 yaml을 명시하지 않으면 기본값으로 app.yaml 파일을 사용한다. `index.yaml` 파일도 함께 deploy 했다면 이제 API를 다시 호출해보자. 새로운 에러 메세지를 만날 수 있다.

```
API error 4 (datastore_v3: NEED_INDEX): The index for this query is not ready to serve. See the Datastore Indexes page in the Admin Console.
```

일단 해결법부터 말하자면 **그냥 기다리면 된다.** `index.yaml` 파일의 내용을 기반으로 한 인덱싱 작업이 아직 완료되지 않아서 사용할 수 없다는 내용이다. 클라우드 콘솔 페이지(https://console.cloud.google.com/datastore/indexes?project=project-name)에 접근하면 열심히 인덱싱 작업중이라는 메세지가 나온다. 인덱싱 작업에 걸리는 시간은 기준은 정확이 모르지만 필자의 경우 datastore에 데이터가 거의 없고 쿼리내용도 별로 복잡하지 않았는데 약 30분 정도 걸렸다. 구글 검색결과 해외의 어느 유저분은 4시간이 걸렸다고 한다. 자세한 내용은 [공식 문서](https://cloud.google.com/datastore/docs/concepts/indexes)를 참고해보자.


## 수정내용(2018-05-01)

`gcloud app deploy index.yaml` 명령어로 배포를 할 경우 서버코드는 배포되지 않고 인덱스 설정만 배포된다. 서버코드를 수정하고 배포할 경우 아래의 명령어를 사용해야 한다.

``` shell
gcloud app deploy app.yaml index.yaml
```

인덱스 내용에 변경사항이 없을 경우 `index.yaml`은 생략해도 된다.

