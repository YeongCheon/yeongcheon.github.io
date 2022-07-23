# Cloud Functions로 실시간 이미지 리사이징 하기


[[https://medium.com/daangn/lambda-edge%EB%A1%9C-%EA%B5%AC%ED%98%84%ED%95%98%EB%8A%94-on-the-fly-%EC%9D%B4%EB%AF%B8%EC%A7%80-%EB%A6%AC%EC%82%AC%EC%9D%B4%EC%A7%95-f4e5052d49f3][당근마켓]], [[http://engineering.vcnc.co.kr/2016/05/ondemand-image-resizing/][비트윈]] 등에서 작성한 글을 참고하여 GCP 기반으로 비슷한 기능을 하는 코드를 작성하여 봅시다. 동작 원리는 앞서 말한 글에 자세히 나와있으니 서론은 생략하고 바로 시작합니다.

이 문서에서는 GCP의 아래 제품들을 사용합니다.

- [[https://cloud.google.com/functions/docs?hl=ko][Cloud Functions]]
- [[https://cloud.google.com/storage?hl=ko][Cloud Stroage]]
- [[https://cloud.google.com/cdn/docs/using-cdn?hl=ko][Cloud CDN]]

* Cloud Storage
  실제 이미지들이 저장될 storage bucket을 생성해야 합니다. 여기선 특별할게 없으니 [[https://cloud.google.com/storage/docs/creating-buckets?hl=ko][공식 가이드]]로 대체합니다.

* Cloud Functions
** 코드 수정 및 배포
Cloud Function을 호출하는 방법은 여러가지가 있습니다만 이 문서에서는 http 요청을 통해 호출하는 방법을 사용합니다. Cloud Functions에 배포할 코드는 [[https://github.com/YeongCheon/imagick-cf][여기]]에 있습니다. 프로젝트를 받으신 후 소스코드([[https://github.com/YeongCheon/imagick-cf/blob/a311deffdb085cb277d7fbfbf78006958d0145a5/main.go#L32][32번 라인]])에서 BUCKET_NAME을 이전 단계에서 생성한 실제 버킷 이름으로 교체해야 합니다. 버킷명 수정 후 프로젝트 루트 폴더에서 아래의 명령어를 실행해서 코드를 배포합니다. 이 명령어를 통해 배포하면 인증되지 않은 사용자도 http 요청을 통해서 함수를 호출할 수 있습니다.

#+BEGIN_SRC bashㅍ
gcloud functions deploy OptimizeImage \
 --runtime go113 \
 --trigger-http \
 --region asia-northeast3 \
 --allow-unauthenticated
#+END_SRC

배포가 성공하면 콘솔창(https://console.cloud.google.com/functions/list)에서 아래와 같이 배포된 내역을 확인할 수 있습니다.

[[/images/Screenshot from 2021-01-11 22-21-04.png]] 

** 트리거 실행

콘솔창에서 해당 함수 이름을 클릭하면 트리거 탭이 있습니다. 트리거 탭에 접근하면 아래와 같이 트리거 URL을 확인할 수 있습니다.

[[/images/2021-01-11 22-38-12.jpg]] 

표시된 URL 뒤에 접근할 이미지 이름과 원하는 이미지 포맷, 사이즈 등등을 입력하면 결과를 확인할 수 있습니다(예시 URL: https://asia-northeast3-PROJECT-NAME.cloudfunctions.net/OptimizeImage/IMAGE_NAME.jpg?format=webp&width=1024).

* Cloud CDN
cloud function이 정상적으로 동작하는 게 확인됐으면 이제 Cloud CDN에 생성한 function을 연결할 차례입니다. 이것도 별거 없으므로 [[https://cloud.google.com/cdn/docs/setting-up-cdn-with-serverless?hl=ko][공식 가이드]]로 대체합니다.


이상입니다.

