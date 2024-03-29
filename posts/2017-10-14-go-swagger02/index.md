# go swagger 서버파일 구조 분석


# Intro
---------------------
[지난번 포스트][last-post]에서 만든 hello world 프로젝트의 구성요소들을 분석해보자.

# 폴더 구조
---------------------
``` tree
├── cmd
│   └── helloworld-server
│         └── main.go
├── github.com
├── golang.org
├── gopkg.in
├── models
│    └── item.go
├── restapi
│    ├── configure_helloworld.go
│    ├── doc.go
│    ├── embedded_spec.go
│    ├── operations
│    │   ├── get.go
│    │   ├── get_parameters.go
│    │   ├── get_responses.go
│    │   ├── get_urlbuilder.go
│    │   └── helloworld_api.go
│    └── server.go
└── swagger.yml
```
[지난번 포스트][last-post]를 처음부터 끝까지 다 따라했다면 `$GOPATH/src` 폴더가 위와 같은 구조로 이루어져 있을 것이다. 위의 구조에서 `swagger generate server` 명령어로 인해 생긴 폴더는 `cmd`, `models`, `restapi` 이렇게 세 개로 구성되어 있으며 이번에 주로 살펴볼 폴더들이다. 나머지 폴더들은 `go get` 명령어로 인해 생성된 외부 라이브러리 관련 폴더들이다.

## cmd
--------------------
서버를 실행시킬 때 사용하는 파일이 들어있다. 지난번 포스트에서 서버를 실행시킬 때 사용했던 명령어는 아래와 같다.
``` command
go run cmd/helloworld-server/main.go --port=9000 --host=127.0.0.1 --socket-path=/tmp/helloworld.sock --scheme=http
```
구성요소들은 상당히 간소하다. `helloworld-server` 폴더 아래에 `main.go` 파일이 전부이다. 여기서 `helloworld-server`는 해당 프로젝트의 이름으로 `go generate` 명령어를 사용할 때 `-A` 옵션을 이용해 지정할 수 있다. 아래의 명령어는 지난 포스트에서 사용했던 명령어이다.
``` command
swagger generate server -A helloworld -f ./swagger.yml
```
cmd 폴더 아래의 내용들은 사실 서버를 실행시키기 위한 파일에 불과할 뿐 우리가 `swagger.yml`에 선언한 내용들이 거의 들어가 있지 않다. 실제 프로그램 구동에 크게 영향을 미치는 파일들은 `restapi`, `models` 폴더에 들어있다.

## models
---------------------
`models` 폴더는 `swagger.yml` 파일에서 `definitions` 하위에 선언했었던 객체 모델을 코드로 구현해놓은 곳이다. [MVC 패턴][mvc-pattern]의 M에 해당한다고 생각하면 된다. 그래서 폴더 이름이 models인 것이다. 이 예제에선 `item.go` 파일이 구현되어 있다. `item.go` 는 파일명부터 내용까지 다 철처하게 `swagger.yml` 파일을 반영해서 생성된 것이다. 아래의 코드는 `swagger.yml`에서 `item.go` 파일을 정의한 내용이다. 이 내용은 `swagger.yml`의 `definitions` 아래에 작성되어 있다.
``` yaml
  item:
    type: object
    required:
      - result
    properties:
      result:
        type: string
```
첫번째 줄의 `item`이 모델명, 즉 `item.go` 파일 이름이 item인 이유다. item 대신 user를 쓰면 생성된 파일은 user.go가 될 것이다. 두번째 줄은 type의 종류를 선언하는 영역인데 모델은 대부분 여러개의 필드를 가지고 있기 때문에 object로 선언한다. 만약 여러개 속성없이 단순 문자열만 반환한다면 string으로 선언해도 무방할 것이다(실험을 안해봐서 확실하지 않다).

세번째 줄의 `required`는 이 모델의 속성에서 필수로 입력해야 되는 속성들을 선언하는 영역이다. 이번 예제에서는 `result` 필드를 필수로 하겠다고 선언되어 있다.

`properties`는 이 모델이 가지고 있는 속성들을 작성하는 곳이다. 이번 예제에서는 `result` 하나만 사용했지만 여러개를 추가할 수 있다. 이름도 얼마든지 마음대로 선언할 수 있다(프로그램 변수선언 규칙 내에서).

아래의 코드는 위의 코드를 확장에서 `user` 모델을 추가로 선언하는 `swagger.yml` 코드의 `definitions` 관련 구문이다.
``` yaml
definitions:
  item:
    type: object
    required:
      - result
    properties:
      result:
        type: string
  user:
    type: object
    required:
      - id
    properties:
      id:
        type: string
      name:
        type: string
      email:
        type: string
```
위 코드가 포함된 `swagger.yml`파일을 사용해 `go generate` 명령어를 수행하면 `models` 폴더 아래에 `user.go`, `model.go` 파일이 각각 생성되어 있을 것이다. 그리고 models 관련 내용 중 가장 주의해야 할 점은  **models 폴더 아래의 파일들은 직접 수정하면 안된다!** 코드를 까보면 알겠지만 소스파일 최상단에 코드를 수정하지 말라는 경고주석이 작성되어 있다. 이러한 경고가 있는 이유는 크게 두가지가 있다.

첫번째. `swagger.yml` 파일을 수정하고 다시 `swagger generate` 명령을 수행할 경우 models 폴더의 모든 내용은 swagger가 덮어씌우기 때문이다. 따라서 만약 `models` 폴더의  내용을 바꾸고 싶다면 `swagger.yml` 파일을 수정하고 다시 `swagger ganerate` 명령어를 실행하여야 한다. 물론 안의 코드를 직접 수정해도 제대로 수정만 한다면 프로그램은 문제없이 돌아간다. **하지만 `swagger.yml`을 업데이트 해야 할 경우는 반드시 생긴다. 절대 직접 수정하는 일은 없도록 하자.**

두번째. 코드의 내용이 문서와 달라질 가능성이 생긴다. **우리가 swagger를 쓰는 목적은 REST API 문서를 자동으로 생성하기 위해서이다.** swagger는 `swagger.yml`의 내용을 기반으로 spec을 생성, 이 spec이 곧 문서가 된다. 그리고 swagger는 코드의 변경내용을 파악하지 못한다. 근데 `swagger.yml`을 수정하지 않고 직접 코드를 수정할 경우 우리가 추후 생성할 문서에는 나타나지 않는 변경점이 코드에 반영될 수 있다. **우리가 코드에 직접 작성해야 할 내용은 오직 로직뿐이다.** 모델, Path 등등은 직접 코드로 작성하지 않도록 하자.

## restapi
----------------------
Handler, 즉 MVC 패턴의 C에 해당하는 부분이다. `swagger generate` 명령어로 생성된 대부분의 파일은 개발자가 코드를 직접 수정하는걸 금지하한다. 하지만 이 폴더의 **`configure_helloworld.go`는 우리가 직접 수정할 수 있다.** 사실상 우리가 건드려도 되는 **유일한 파일**인데  실제 비즈니스 로직을 여기에 작성하면 된다. `configure_helloworld.go` 파일을 살펴보면 다음과 같은 주석이 달려있다.
``` golang
// This file is safe to edit. Once it exists it will not be overwritten
```
위 주석은 `swagger generate` 명령어를 여러번 실행해도 이 파일은 덮어씌워지지 않는다는 소리다. 사실 프로그램을 구현하려면 이 파일을 수정할 수 밖에 없는 구조인데 swagger.yml을 업데이트 하고 `swagger generate` 명령어를 실행할 때마다 이 파일이 덮어씌워진다면 제대로 프로젝트 진행하기가 상당히 까다로울 것이다. 그리고 **이 파일을 수정할 수 밖에 없는 이유는 이 파일의 코드에 선언된 모든 Handler가 `NotImplemented error`를 반환하기 때문이다.** [지난번 포스트][last-post]에서도 `NotImplemented error` 코드를 주석처리하고 실제 원하는 결과를 출력하도록 코드를 수정하는 내용이 있다.

restapi 폴더에선 `configure_helloworld.go` 파일이 핵심이라 다른 파일들에 대한 얘기가 너무 소홀했는데 간단하게 말하자면 아래와 같다. ~~사실 몰라도 된다.~~
* doc.go : `swagger.yml` 에서 선언한 프로젝트명, 호스트, 기본경로 등등 프로젝트의 기초가 되는 정보들을 가지고 있는 파일이다. godoc에서 쓰라고 만든 파일인 것 같다.
* embedded_spec.go : `swagger.yml` 파일을 json 형식으로 변환한 spec을 가지고 있는 파일이다.
* opertaions 폴더 : REST API 프로그램에 필요한 필수사항들(파라메터 처리, Method 별 분기처리, 기타 등등..)을 구현한 코드. 예제에서는 GET Method만 사용하지 때문에 GET 관련 파일들만 보이지만 추가로 POST, PUT, DELETE Method를 사용할 경우 파일이 추가 생성된다(확실하지 않음).

# 한줄 요약
**`configure_helloworld.go` 파일을 제외한 다른 파일들은 직접 수정하지 말고 `swagger.yml`을 수정하여 `swagger generate` 명령어를 이용하자.**

[last-post]: http://yeongcheon.github.io/2017/10/06/go-swagger01/
[mvc-pattern]: https://opentutorials.org/course/697/3828

