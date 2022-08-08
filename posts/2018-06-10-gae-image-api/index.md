# Google App Engine Image API 삽질 후기


오늘은 GAE에서 제공하는 [Image API](https://cloud.google.com/appengine/docs/standard/go/images/)를 사용하면서 겪은 어려움과 후기를 공유하고자 한다.

## 요구사항
* GAE를 통해 업로드 된 이미지를 [Google Cloud Storage](https://cloud.google.com/go/getting-started/using-cloud-storage)(이하 GCS)에 보관, 파일 관련 정보를 datastore에 별도로 저장한다(업로드 한 사용자, 업로드 파일명, 업로드 날짜 등등). 이 이미지를 웹사이트를 통해 서비스를 하고싶다. 이 이미지는 사이즈가 모두 제각각인데 DSLR로 찍은 수십메가짜리 이미지부터 조그마한 썸네일 이미지까지 크기가 천차만별이다.
* 동일한 이미지를 여러가지 사이즈로 출력해야 한다. 가령 DSLR로 찍은 이미지를 제공할 경우 우선 100px짜리 썸네일로 제공, 사용자가 썸네일을 클릭할 경우 이미지의 원본을 보여주고 싶다.
* 사용자가 주로 모바일을 통해 서비스를 사용할것으로 예측되는데 사용자의 디바이스마다 최적화 된 크기의 이미지를 제공하고 싶다.

## 고민
위와 같은 요구사항을 충족시키기 위해 인터넷을 꽤 오래 뒤지고 다녔다. 처음에는 [비트윈의 블로그 글](http://engineering.vcnc.co.kr/2016/05/ondemand-image-resizing/)을 보고 이걸 따라하면 되겠다 싶었다. 하지만 문제가 있었다. 이미지 업로드를 GAE에서 구현하다보니 [이미지를 webp 형태로 변환하는 golang 라이브러리](https://github.com/chai2010/webp)를 쓸 수가 없었다. 마찬가지로 온디멘드 리사이징도 GAE 위에서 돌리기엔 문제가 좀 있었다(내가 해결책을 못 찾은걸수도 있다). 결국 며칠동안 고민만 하고 코드를 한줄도 작성을 못해서 일단 무식한 방법으로라도 구현을 해보기로 했다. 그래서 아래와 같은 방법을 후보로 선정했다.

## 해결책 후보 리스트
1. 제일 단순한 방법. GCS 보관되어 있는 이미지를 그대로 제공한다. 이 방법은 사용자가 모바일에서 DSLR 이미지를 호출할 경우 데이터 소모량이 너무 크다. 그리고 느린 와이파이 환경에서 접근하면 이미지 로딩 시간이 엄청 길 것으로 예상된다.
2. 사이즈가 일정 크기 이상 되는 이미지들은 GCS에 업로드 되는 시점에 여러 크기로 resize된 사본 이미지를 미리 생성후 사용자 기기에 맞는 이미지를 제공한다. 하지만 디바이스의 종류가 너무 많고 사본 이미지를 미리 생성할 경우 얼마만큼의 사이즈로 몇개나 생성해야 할 지 알 수 없다. 그리고 GCS에 용량을 너무 많이 잡아먹을것으로 예상된다.
3. GAE에서 제공하는 [Images Go API](https://cloud.google.com/appengine/docs/standard/go/images/)를 사용한다.

사실 세번째 방법은 처음에 존재조차 몰랐다. 1,2번 방법을 떠올렸지만 너무 마음에 들지 않아서 이것저것 찾아보다가 발견했다. 문서를 등한시 한 내 잘못이다. 항상 공식문서를 꼼꼼히 보는 습관이 참 중요하다는 것을 다시 한번 뼈저리게 깨달았다. 

Images API는 내 요구사항을 정말 완벽하게 지원하고 있었다. 사이즈 조절 및 썸네일 생성을 URL 파라메터를 통해서 지원한다. 심지어 [가격도 무료다](https://cloud.google.com/appengine/docs/standard/java/images/#quotas-limits-pricing)(Java문서긴 하지만 Go도 동일하게 적용되지 않을까 싶다).


아무튼 여러모로 장점이 많아보이니 세번째 방법을 적용해보자.

## 코드 구현
아래 코드는 GAE golang 1.8을 기준으로 작성했다. 내가 구현한 코드의 일부를 발췌해서 약간 수정한 버전으로 syntax 에러가 있을 수 있다. 

``` golang
import (
	"google.golang.org/appengine/blobstore"
	"google.golang.org/appengine/image"
	"google.golang.org/appengine/log"
	
	"context"
)

func GetImageURL(ctx context.Context) string {
	blobKey, err := blobstore.BlobKeyForFile(ctx, "/gs/imagename.jpg") // gcs에 있는 이미지 경로를 넣어준다.
	
	if err != nil {
		log.Errorf(ctx, "blobKeyForFile error : %s", err)
		continue
	}

	opts := &image.ServingURLOptions{
		Secure: true,
	}

	url, err := image.ServingURL(ctx, blobKey, opts)
	if err != nil {
		log.Errorf(ctx, "create ServingURL error : %s, %s", err, profileImage)
		return ""
	}
	
	return url.String()
}
```

위의 코드를 실행하기 위해서 deploy 명령에 옵션을 추가해줘야 한다. 어떤 GCS 버킷을 바라볼건지 명시를 해줘야 하는데 아래의 deploy 명령을 보자.

``` shell
gcloud app deploy app.yaml --bucket gs://projectname.appspot.com
```
위와 같이 어떤 버켓을 기본값으로 바라볼지 명시를 해줘야 한다. 위의 명령어를 이용해 deploy했다면 GetImageURL 함수를 호출하여 해당 파일을 볼 수 있는 URL을 반환받을 수 있다.

## 특이점

* **로컬 서버에서는 제대로 동작하지 않는다.**(내가 방법을 못 찾은 걸수도 있다.) 로컬에서 `dev_appserver.py`를 이용해 위 코드를 테스트하면 비정상적인 URL이 반환되는데 이 URL은 동작하지 않는다. 테스트는 꼭 실제 GAE에서 하도록 하자. 정상적인 URL은 아래와 비슷한 모양이다.
```
https://lh3.googleusercontent.com/tUcBqKywA1YYVTll2HZzjl7LE80ta8jOseulgwBRuv6FEX989G1E49s00d9yt4pJ0ql1htNcr9MCIlp33oKyha
```

* url을 생성하고 GCS에서 해당 이미지를 지워도 URL이 정상적으로 동작한다. 아마 구글 인프라 어딘가에 캐시가 되어있는거 같은데 기간이 생각보다 길다. 개인적으로 테스트를 해봤는데 파일이 지워져도 3일 이상은 계속 URL을 통해서 이미지를 볼 수 있었다. 즉 **이미지를 지울 생각이면 GCS에서 삭제는 물론이고 [DeleteServingURL](https://cloud.google.com/appengine/docs/standard/go/images/reference#DeleteServingURL)을 이용해서 URL도 꼭 제거를 해줘야 한다.**

