# 도커 컨테이너 안에서 외부 URL 호출하기


도커 컨테이너 내부에서 외부 URL을 호출해야 하는 경우가 종종 있습니다(Ex: 네이버 API 호출). 평소에는 아무 생각없이 불러오던 URL이 도커 컨테이너에서 사용하면 에러가 발생하는 경우가 있습니다. 에러 메세지는 ~x509: certificate signed by unknown authority~ 대충 이런 모양인데 이는 *도커 컨테이너에 사용한 이미지의 TLS 인증서에 문제가 있음을 의미합니다.*

우분투 이미지 기반 컨테이너를 기준으로 해결방법은 아래와 같습니다. 사용하고 있는 ~Dockerfile~ 에 다음 내용을 입력합니다.

#+BEGIN_SRC docker
RUN apt update
RUN apt install -y ca-certificates
RUN update-ca-certificates
#+END_SRC

컨테이너를 다시 빌드하면 이제 에러 없이 URL을 정상적으로 호출하는걸 확인할 수 있습니다.

