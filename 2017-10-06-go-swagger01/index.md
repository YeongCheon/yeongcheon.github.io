# go swagger 사용법


# Intro
---------------------
이 문서는 [goswagger][goswagger-github]를 이용해 [golang][go-website]으로 작성된 간단한  REST API SERVER 프로그램을 만드는 방법을 설명한다. 개발환경은 `ubuntu 16.04LTS`, `golang 1.8`을 사용중이다.

이 문서를 작성하며 참고한 사이트는 아래와 같다. ~~사실 이 문서 안보고 아래 목록만 살펴봐도 된다.~~
* [swagger 공식 사이트][swagger-site]
* [goswagger.io][goswagger-io]
* [yundream님의 사이트][goswagger-joinc-tutorial], 상당히 정리가 잘 되어있다.
* 구글링...

# goSwagger 설치
--------------------------
[github 페이지][goswagger-github]의 README 파일을 보면 아래의 명령어를 쓰라고 한다.
``` command
echo "deb https://dl.bintray.com/go-swagger/goswagger-debian ubuntu main" | sudo tee -a /etc/apt/sources.list
```
**근데 동작을 안한다...**


`go get` 명령어를 이용해 소스파일을 직접 컴파일 해도 되지만 여기서는 `apt` 명령어를 이용해 설치하는 방법을 알아보겠다.

``` command
echo "deb [trusted=yes] https://dl.bintray.com/go-swagger/goswagger-debian ubuntu main" | sudo tee -a /etc/apt/sources.list
sudo apt update
sudo apt install swagger
```

위 명령어들 중 첫번째 명령어는 내 apt repository에 dl.bintray.com/swagger 사이트를 추가하겠다는 의미이다. [trusted=yes] 구문이 없을경우 `apt update` 명령어를 제대로 수행하지 못한다. 아마 인증서 관련 문제로 추측하는데 확실하지는 않다. 그 이후 명령어는 repository 목록에 접근하여 swagger를 설치하는 내용이다. 위 명령어들을 에러없이 무사히 마쳤을 경우 `/usr/bin` 경로에 swagger 파일이 생성되어 있을것이다.

***위 방법을 통해 설치한 swagger는 제대로 동작을 하지 않는다. 아마 바이너리 파일이 제대로 업데이트가 안되어 있는듯 하다. 위 방법은 무시하고 아래의 `go get` 명령어를 이용해 설치를 하도록 한다.***

``` command
go get -u github.com/go-swagger/go-swagger/cmd/swagger
```

위 명령어를 실행하면 `$GOPATH/bin` 폴더 아래에 swagger 파일이 생성되어 있을것이다. `$GOPATH/bin`을 PATH에 추가하거나 swaager 파일을 `/usr/local/bin` 폴더로 옮기면 어디서든 swagger 명령어를 사용할 수 있다.



# spec 작성하기
------------------
swagger는 스펙에서 코드를 생성한다. 즉 우리가 스펙만 제대로 정의해주면 코드의 골격은 swagger가 알아서 작성해준다(물론 안의 내용을 구현하는건 개발자 몫이다).그럼 이제 스펙을 만들어보자.

1. 본인의 `$GOPATH/src` 폴더에서 아래의 명령어를 실행한다.
``` command
swagger init spec \
  --title "hello world application" \
  --description "hello go swagger" \
  --version 1.0.0 \
  --scheme http \
  --consumes application/json \
  --produces application/json
```

2. 위 명령어를 실행하면 아래처럼 생긴 `swagger.yml` 파일이 생긴다.
``` yaml
consumes:
- application/json
info:
  description: hello go swagger
  title: hello world application
  version: 1.0.0
paths: {}
produces:
  - application/json
consumes:
  - application/json
schemes:
  - http
swagger: "2.0"
```

3. 위 파일에서 7번째 줄의 `paths:{}` 구문을 우리 입맛대로 고쳐줘야 한다. 이번 예제에선 다른 구문들은 건들지 않아도 된다.

4. 우리는 이번 예제에서 GET 방식으로 `/` URL을 호출할 시 hello world를 반환하는 예제를 만드는 게 주 목적이다. 그러기 위해선 `swagger.yml`의 `paths:{}` 구문을 제거하고 파일 최하단에 아래 코드를 붙여넣기 한다. 추가로 파일 맨 위의 consumes 구문도 지워준다(버그인지 두번 반복되고 있다).

``` yaml
swagger: "2.0"
info:
  description: hello go swagger
  title: hello world application
  version: 1.0.0
produces:
  - application/json
consumes:
  - application/json
schemes:
  - http
paths:
  /:
    get:
      responses:
        200:
          description: list the todo operations
          schema:
            type: array
            description: "result : helloworld"
            items:
              $ref: "#/definitions/item"
definitions:
  item:
    type: object
    required:
      - result
    properties:
      result:
        type: string
```

`swagger.yml`파일을 수정할 때 주의할 점은 `yml` 형식의 파일은 들여쓰기로 `\b`(공백)을 두번 쓴다는 점이다. 탭이나 띄어쓰기 한 칸으로 할 경우 오류가 발생할 수 있다(작성자는 들여쓰기를 탭으로 했다가 피봤다).

위 문장 붙여넣기까지 완료한 `swagger.yml` 파일의 최총 코드는 아래와 같다.
``` yaml
swagger: "2.0"
info:
  description: hello go swagger
  title: hello world application
  version: 1.0.0
produces:
  - application/json
consumes:
  - application/json
schemes:
  - http
paths:
  /:
    get:
      responses:
        200:
          description: list the todo operations
          schema:
            type: array
            description: "result : helloworld"
            items:
              $ref: "#/definitions/item"
definitions:
  item:
    type: object
    required:
      - result
    properties:
      result:
        type: string

```

# 유효성 검사 및 서버코드 생성
---------------------------------
`swagger.yml`을 완성했으면 이 파일을 통해 REST API의 기본 코드를 생성할 수 있다. 그 전에 우리가 작성한 `swagger.yml` 파일의 유효성 검사를 해 해당 파일에 문제가 있나 확인을 해야한다.

다음 명령어를 통해 유효성 검사를 할 수 있다.

``` command
swagger validate ./swagger.yml
```

위 명령어 수행 후 **The swagger spec at "./swagger.yml" is valid against swagger specification 2.0** 메세지가 출력되면 검사결과 문제가 없다는 뜻이다. 이제 골격 코드를 생성해보자. 아래의 명령어를 실행하면 된다.
``` command
swagger generate server -A helloworld -f ./swagger.yml
```

위 명령어는 helloWorld라는 이름의 helloworld라는 이름의 go server application 코드를 생성하는 명령어다. 수행이 완료되면 콘솔창 마지막 부분에 **Generation Complete!** 라는 메세지가 출력되면 정상적으로 수행이 완료된 것이다. 그리고 이 코드를 컴파일 하기 위해서 외부 라이브러리가 몇개 필요하다고 메세지가 뜨는데 `go get` 명령어를 통해서 내려받을 수 있다. 목록은 아래와 같다. ~~코드생성에 성공했으면 그냥 터미널 창 보고 직접 확인하면 된다.~~

* github.com/go-openapi/runtime
* github.com/tylerb/graceful
* github.com/jessevdk/go-flags

그런데 generate 명령 실행 시 위 파일만 내려받으면 된다고 하는데 막상 실행하려고 하면 추가로 더 필요한 라이브러리가 있다. 아래의 목록도 마저 내려받도록 하자.

* github.com/docker/go-units
* github.com/go-openapi/analysis
* github.com/go-openapi/spec
* github.com/go-openapi/validate

만약 [dep](https://github.com/golang/dep)를 사용하고 있다면 그냥 `dep ensure` 명령어만 쓰면 끝난다.~~세상 편하다.~~

# 서버 실행 및 소스 수정
---------------------------------
유효성 검사결과에 이상이 없을 경우 이제 `swagger.yml` 파일을 이용해 server code를 생성할 수 있다. 아래의 명령어를 이용해 서버를 실행시켜보자.

``` command
go run cmd/helloworld-server/main.go --port=9000 --host=127.0.0.1 --socket-path=/tmp/helloworld.sock --scheme=http
```

위 명령을 실행하면 generate로 생성된 서버 코드가 동작한다. 서버를 실행시킨 상태에서 http://127.0.0.1:9000 URL에 접근하면 **"operation .Get has not yet been implemented"** 메세지가 뜬다.

[swagger][swagger-site]는 우리가 작성한 스펙, 즉 `swagger.yml`에 따라 end point와 추가로 선언한 모델만 작성해 줄 뿐 실제 비즈니스 로직은 우리가 직접 구현해야 한다. 이번 문서에서 우리는 복잡한 로직이 아니라 단순히 helloworld만 출력하는게 목적이므로 아래의 내용에 따라 소스코드를 살짝만 고쳐주자.

1. `src/restapi/configure_helloworld.go` 파일에 접근하여 `import` 구문 바로 아래에 아래의 코드를 추가한다. 사실 `configure_helloworld.go` 파일 안이라면 위치는 자기 마음대로 해도 된다(문법이 허용하는 범위 내에).

``` golang
type ResultResponse struct {
	Result string `json:"result"`
}

func (result *ResultResponse) WriteResponse(w http.ResponseWriter, producer runtime.Producer) {
	producer.Produce(w, result)
}
```

2. `src/restapi/configure_helloworld.go` 파일의 `configureAPI` 함수 안에 `api.getHandler` 구문의 내부를 수정해줘야 한다. 내부 코드를 살펴보면 아래와 같은 구문이 있을 것이다.

``` golang
return middleware.NotImplemented("operation .Get has not yet been implemented")
```

위 코드는 해당 함수를 아직 개발자가 구현해주지 않았다는 뜻이다. 위 구문 때문에 코드 생성 후 최초로 접근했을 때 **"operation .Get has not yet been implemented"** 메세지가 떳던 것이다. 이제 이 구문을 삭제, 또는 주석처리 하고 아래의 코드를 입력한다.

``` golang
return &ResultResponse{
	Result: "hello world",
}
```

이 후 파일 내용을 저장하고 서버를 재시작 하고 http://127.0.0.1:9000 URL에 접근하면 아래와 같은 json 형식의 내용을 출력한다.

``` json
{"result":"hello world"}
```

우리가 출력하고자 한 hello world 구문이 정상적으로 출력된 것을 확인할 수 있다. 이 후 별도의 문서에서 generate 명령어를 이용해 생성된 코드의 구조를 좀 자세히 알아보고자 한다.

[swagger-site]: http://swagger.io/
[go-website]: http://golang.org
[goswagger-github]: https://github.com/go-swagger/go-swagger
[goswagger-io]: https://goswagger.io
[goswagger-joinc-tutorial]: https://www.joinc.co.kr/w/man/12/swagger

