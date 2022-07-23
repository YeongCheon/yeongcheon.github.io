# S2 library를 이용하여 가까운 위치의 사용자 찾기


구글에서 제공하는 [S2](http://s2geometry.io/) 라이브러리를 이용하여 현재 내 위치에서 가장 가까운 사용자 목록을 추출해보자. 서버 환경은 [GAE](https://cloud.google.com/appengine/docs/), [golang standard environment](https://cloud.google.com/appengine/docs/standard/go/) 이다. [이 글](https://blog.nobugware.com/post/2016/geo_db_s2_geohash_database/)을 참고했다.

## S2 라이브러리란?

S2 라이브러리는 구글에서 비공식적으로([not an official](https://github.com/google/s2geometry#disclaimer)) 제공하는 구(球)형상 라이브러리이다(~~번역기 발췌~~). 기존의 대다수 라이브러리는 2차원 평면을 기준으로 좌표시스템을 구축하였지만 S2 라이브러리는 3차원 구를 기준으로 좌표계를 사용한다. 실제 지구는 평면보단 구(球)에 훨씬 가깝기 때문에 좀 더 정교하고 왜곡없는 지리 데이터베이스를 구축할 수 있다(~~이 역시 번역기 발췌~~).

추가로 [geohash](http://geohash.org/)와 비교해보면 도움이 많이 된다.

## 개발 요구사항

1. 사용자의 현재 위치를 위도, 경도 형태로 전달받아 서버에 저장한다.
2. 서버에 저장된 사용자들을 특정 좌표(위도, 경도)에 가장 가까운 순으로 정렬해서 결과값으로 반환한다.

## 제약사항

1. DB는 [datastore](https://cloud.google.com/appengine/docs/standard/go/datastore/)를 사용한다(mysql, mariadb, elasticsearch, mongoDB 등등 유명한 DB 사용불가).
2. InMemory DB는 [Memcache](https://cloud.google.com/appengine/docs/standard/go/memcache/)를 사용한다(redis 사용 불가).
3. 서버 인스턴스는 GAE standard 환경 위에서 가동된다(보편적인 서버환경에 비해 제약사항이 꽤 있다).


사실 위 제약사항들은 모두 비용을 최소화 하느라 어쩔수 없이 발생한 문제들이다(ㅠㅠ). 살림살이에 여유가 있다면 GCE로 별도 인스턴스를 띄워 [mysql](https://stackoverflow.com/questions/4687312/querying-within-longitude-and-latitude-in-mysql)이나 [elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-geo-distance-query.html)같은 디비를 설치해 쉽게 검색쿼리를 이용할 수 있다.


## 검색 방법

### 위치정보 갱신

1. **과거 위치 삭제** : userID를 key로 사용하여 memcache 아이템을 조회, 값이 있을 경우 해당 아이템에 들어있는 LatLng 값을 사용하여 CellID를 추출, CellID를 key로 사용하여 memcache 아이템을 조회, 본인의 userID를 제거한다.
2. **현재 위치를 userID key를 이용해 저장** : `key: userID`, `value: LatLng` 형식의 아이템을 memcache에 셋팅한다.
3. **CellID 추출** : LatLng를 이용해서 Level25의 CellID를 구한다(레벨이 높을수록 Cell 당 범위가 좁아진다. 최대레벨은 30).
4. **CellIdList에 cellId 추가** :  CellID를 `key: "cellIdList"`, `value: []string` 아이템에 추가한다. 이 때 []string 배열 안에 이미 내 userId가 들어있을 경우 추가하지 않고 생략한다.
5. **CellID를 key로 사용하여 내 userID 추가** : `key: CellID`, `value: map[string]bool` 아이템을 추가한다. Map에는 `key: userId`, `value: true` 형식으로 값 추가.

### 조회

1. **기준 좌표의 CellId 추출** :  입력받은 기준값 LatLng를 이용해 Level25짜리 CellID를 추출한다(레벨이 높을수록 Cell 당 범위가 좁아진다. 최대레벨은 30). 이 때 CellID는 위의 위치정보 갱신 로직에서 사용한 레벨과 동일해야 한다.
2. **기준 좌표와 가까운 순으로 모든 CellIdList 추출** :  cellIdList에 있는 CellIdNumber와 기준값과의 차이를 절댓값으로 구한 뒤 오름차순으로 정렬한다(이 차이가 적을수록 가깝다고 판단하는데 맞는지 모르겠다).
3. **cellId에 속해있는 사용자 Id 추출** : 정렬된 순서대로 cellId 추출해서 key: CellID, value: map[string]bool 아이템을 조회, 유저 아이디 목록을 뽑는다.
4. **사용자 상세정보 조회** : 유저 아이디를 key로 사용해서 datastore에서 사용자 정보를 조회, 목록으로 만들어 반환한다.


## Code

``` golang

func (usersBiz *UsersBiz) GetUserList(ctx context.Context, userId string, latitude, longtitude float64, page int) (interface{}, error) {
	userDao := usersBiz.UserDao

	guidePoint := s2.LatLngFromDegrees(latitude, longtitude)
	guideCellID := s2.CellIDFromLatLng(guidePoint)

	customCache := customMemcache.Memcache{
		Context: ctx,
	}

	var userResponseList []models.UserResponse
	if page == 1 {
		cellIDs := customCache.GetCellIdList()
		similarList := make([]map[string]interface{}, len(cellIDs))

		for idx, cellID := range cellIDs {
			similarMap := make(map[string]interface{})

			cellIdNumber := uint64(cellID)
			guideCellNumber := uint64(guideCellID)

			similarMap["cellID"] = cellID
			similarMap["diffPoint"] = math.Abs(float64(cellIdNumber - guideCellNumber))

			similarList[idx] = similarMap
		}

		sort.Slice(similarList, func(i, j int) bool {
			return similarList[i]["diffPoint"].(float64) > similarList[j]["diffPoint"].(float64)
		})

		userIdList := make([]string, 0)
		for _, similarMap := range similarList {
			cellID := similarMap["cellID"].(s2.CellID)

			userMap, err := customCache.GetUserMapInCell(cellID)
			if err != nil {
				log.Errorf(ctx, "%s", err)
			}

			for userId, _ := range userMap {
				if !util.IsContains(userIdList, userId) {
					userIdList = append(userIdList, userId)
				}
			}
		}

		if len(userIdList) == 0 {
			return make([]interface{}, 0), nil
		}

		userList, err := userDao.GetUserList(ctx, userIdList)
		if err != nil {
			log.Errorf(ctx, "%s", err)
		}

		userResponseList = convertUserListToUserResponseList(ctx, userDao, &guidePoint, userList, userId)

		sort.Slice(userResponseList, func(i, j int) bool {
			return userResponseList[i].Distance < userResponseList[j].Distance
		})

		customCache.UpdateNearestUserList(userId, userResponseList)
	} else {
		personalData, _ := customCache.GetPersonalData(userId)
		userResponseList = personalData.NearestUserResponseList
	}

	if len(userResponseList) <= 0 {
		return models.ERR_NO_CONTENT, nil
	}

	var startIndex int
	var endIndex int

	startIndex = (page - 1) * 20
	if page*20 > len(userResponseList) {
		endIndex = len(userResponseList)
	} else {
		endIndex = page * 20
	}

	if startIndex > endIndex {
		return models.ERR_NO_CONTENT, nil
	}

	userResponseList = userResponseList[startIndex:endIndex]

	return userResponseList, nil
}

```


로직도 엉성하고 코드도 엉성하다. [개선점, 피드백은 언제나 환영](mailto:kyc1682@gmail.com).

