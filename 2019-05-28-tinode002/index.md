# tinode series.002 - Install & Run


* 서론
  소스코드를 살펴보기에 앞서서 일단 프로그램 실행을 시켜보자. 이 글에서 설명하는 예제는 커밋 아이디 기준 [[https://github.com/YeongCheon/chat/commit/2558db90a50c80c974c2fccfec6b87ea44e4758b][2558db90a50c80c974c2fccfec6b87ea44e4758b]]을 바탕으로 설명한다. 사실 이 글을 안봐도 [[https://github.com/tinode/chat/blob/master/INSTALL.md][여기]]를 따라하면 보통을 무난하게 설치 및 실행을 할 수 있다. 이 가이드는 공식 가이드 중 [[https://github.com/tinode/chat/blob/master/INSTALL.md#building-from-source][Building from Source]], Mysql을 기준으로 설명한다.

* 실행환경 및 선행조건

  * Ubuntu 18.04 LTS

  * Golang이 설치되어 있어야 한다(기준 버전 : =1.12.5=).

  * Mysql이 설치되어 있어야 한다(기준 버전: =5.7=)

* 설치 및 실행하기

** source 파일 내려받기 및 바이너리 파일 install
다음 명령어를 입력한다.

#+BEGIN_SRC shell
go get -tags mysql github.com/tinode/chat/server && go install -tags mysql github.com/tinode/chat/server
go get -tags mysql github.com/tinode/chat/tinode-db && go install -tags mysql github.com/tinode/chat/tinode-db
#+END_SRC

첫번째 줄의 명령어는 실제로 동작하게 될 채팅서버와 연관된 소스코드를 다운로드 후 해당 소스코드를 컴파일하여 =$GOPATH/bin/= 경로에 =server= 라는 파일을 생성하는 명령어다.

두번째 줄의 명령어는 데이터베이스 셋팅을 위한 소스코드를 다운로드 후 해당 소스코드를 컴파일하여 위와 마찬가지로 =$GOPATH/bin/= 경로에 =tinode-db= 라는 파일을 생성하는 명령어다.

위의 명령어에서 눈여겨 볼 점은 =go get= 명령어와 =go install= 명령어 실행 시 모두 =-tags= 라는 옵션에 mysql이라는 값을 주었다는 점이다. =-tags= 옵션에 대한 자세한 설명은 [[https://golang.org/cmd/go/][공식문서]] 또는 [[https://jacking75.github.io/go_build/][블로그 글]]을 참고해보자.

** 데이터베이스 스키마 생성하기

MySQL에 데이터베이스 및 테이블을 생성하는 법을 알아보자.
우선 =$GOPATH/src/github.com/tinode/chat/tinode-db/tinode.conf= 파일을 열어서 db 접속정보가 올바른지 확인하자. 만약 접속정보와 다르다면(+당연히 대부분 다를거다+) 자신의 db 설정과 동일하게 고쳐주자. 다른부분은 건드릴 필요 없고 =store_config.adapters.mysql.dsn= 필드만 수정해주면 된다. 설정이 완료됐다면 아래의 명령어를 실행해주자.

#+BEGIN_SRC shell
$GOPATH/bin/tinode-db -config=$GOPATH/src/github.com/tinode/chat/tinode-db/tinode.conf
#+END_SRC

이때 터미널의 위치는 가급적이면 =$GOPATH/src/github.com/tinode/chat= 로 이동시킨 후 실행시키자. conf 파일 중 상대경로 옵션이 적용되어 있는 경우가 있어서 다른 위치에서 명령어를 실행 시 에러를 발생시키기도 한다. 그리고 혹시 테스트용 더미 데이터를 추가하고 싶은 경우엔 위의 명령어 뒤에 = -data=$GOPATH/src/github.com/tinode/chat/tinode-db/data.json= 를 붙여주자. 당연히 필수는 아니다.

명령어가 정상적으로 실행되었다면 터미널 창에 *All done.* 이라는 메세지가 뜬다.

** 채팅서버 실행시키기

이제 마지막으로 실제 동작할 채팅서버를 실행시켜보자. 채팅서버를 실행하기에 앞서 우선 폴더 몇개를 복사해서 이동시켜야 한다.

  * =$GOPATH/src/tinode/chat/server/templ= 폴더를 복사해서 =$GOPATH/bin/= 아래에 붙여넣는다. 이 폴더에는 사용자에게 발송할 메일, sms 등의 템플릿이 담겨있다. 이 템플릿은 고 언어의 기본 패키지인 [[https://golang.org/pkg/html/template/][http/template]] 패키지의 룰을 따른다.
  * =$GOPATH/src/tinode/chat/server/uploads= 폴더를 복사해서 =$GOPATH/bin/= 아래에 붙여넣는다(+이건 사실 빈 폴더라서 굳이 복붙까진 필요없다.+).

그리고 위에서 얘기한대로 터미널의 위치를 =$GOPATH/src/github.com/tinode/chat= 로 이동시킨 후 아래의 명령어를 실행해보자.

#+BEGIN_SRC shell
$GOPATH/bin/server -config=$GOPATH/src/github.com/tinode/chat/server/tinode.conf
#+END_SRC

실행 후 터미널에 *Listening for client HTTP connections on [:6060]* 메세지가 뜬다면 성공이다.

* 부록 - Go Modules를 이용해서 설치하기

  *go 1.11* 버전 이후로는 *Go Modules* 라는 개념을 제공한다. 앞으로 Go는 기존의 =$GOPATH= 를 이용한 방법 대신 Go Module를 이용해서 패키지 관리를 한다고 한다. Go Modules에 대한 자세한 내용은 [[https://velog.io/@kimmachinegun/Go-Go-Modules-%25EC%2582%25B4%25ED%258E%25B4%25EB%25B3%25B4%25EA%25B8%25B0-7cjn4soifk][여기]]와 [[https://blog.golang.org/using-go-modules][여기]]를 참고하자. 

  *참고로 이 부록은 위에서 설명한 가이드를 모두 따라하고 데이터베이스 관련 셋팅이 모두 끝났다고 가정한다.*

** Go modules 한방 스크립트

   쉘 커맨드 명령어를 우선 한번에 작성한 후 한줄 한줄 설명하도록 하겠다.

#+BEGIN_SRC shell
git clone https://github.com/tinode/chat
cd chat/server
go mod init github.com/YeongCheon/chat/server
go build -tags mysql
mkdir static
./server -config=./tinode.conf
#+END_SRC


1, 2번 라인은 별도로 설명하지 않겠다.

*3번 라인이 제일 중요하다.*
go.mod 파일을 만드는 명령어인데 *init 뒤에 붙는 패키지 경로를 자신이 정한 고유의 경로를 써줘야 한다.* 처음에 아무생각 없이 =github.com/tinode/chat/server= 를 입력한다면 아래와 같은 에러메세지를 볼 수 있다.

#+BEGIN_SRC
can't load package: package github.com/tinode/chat/server: unknown import path "github.com/tinode/chat/server": ambiguous import: found github.com/tinode/chat/server in multiple modules:
	github.com/tinode/chat/server (/~~~~~~~~~~~~~~~~~~~~~~~~/chat/server)
	github.com/tinode/chat v0.15.14 (/~~~~~~~~/go/pkg/mod/github.com/tinode/chat@v0.15.14/server)
#+END_SRC

init 작업이 끝나면 의존성 패키지 목록이 이쁘게 정리된 =go.mod= 파일과 =go.sum= 파일을 볼 수 있다.

그 이후의 작업은 =go install= 명령어를 이용해 설치했을때와 동일하다. =go build= 명령어 실행 시 *-tags=mysql* 을 붙여서 mysql을 사용하는 바이너리 파일을 생성한 후 동일한 위치에 statics 폴더를 생성한다. 그리고 마지막으로 바이너리 파일 실행 시 *-config=./tinode.conf* 옵션을 추가하면 정상적으로 동작한다.이 때 혹시 동작하지 않는다면  conf파일 안의 설정값을 확인해보자.

