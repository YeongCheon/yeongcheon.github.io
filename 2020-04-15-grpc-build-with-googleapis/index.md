# grpc-gateway를 이용한 JSON 통신



* 목표
  [[https://blog.banksalad.com/tech/production-ready-grpc-in-golang/#grpc-gateway%25EB%25A5%25BC-%25EC%259D%25B4%25EC%259A%25A9%25ED%2595%259C-json-%25ED%2586%25B5%25EC%258B%25A0][grpc-gateway를 이용한 통신]]을 하기 위해 작성한 ~.proto~ 파일을 컴파일 하는 방법을 알아보겠습니다.
* source code
  우선 컴파일 하고자 하는 소스코드를 살펴봅시다. 이 코드는 [[https://github.com/grpc-ecosystem/grpc-gateway#usage][grpc-gateway]]에 있는 예제와 동일합니다.
  #+BEGIN_SRC proto
  syntax = "proto3";
  package example;
  // myService.proto

  import "google/api/annotations.proto";

  message StringMessage {
    string value = 1;
  }

  service YourService {
    rpc Echo(StringMessage) returns (StringMessage) {
      option (google.api.http) = {
        post: "/v1/example/echo"
        body: "*"
      };
    }
  }
  #+END_SRC
  위의 예제코드에서 신경써야 할 부분은 14-17 라인에서 사용된 ~google.api.http~ option이다. 이 어노테이션은 사용자가 작성한 게 아니라 [[https://github.com/grpc-ecosystem][grpc-ecosystem]]의 일부입니다(정확히는 [[https://github.com/grpc-ecosystem/grpc-gateway][grpc-gateway]]).
* 에러 살펴보기
  우선 단순하게 ~protoc -I. test.proto --go_out=plugins:grpc.~ 명령어를 이용해 컴파일 하면 다음과 비슷한 메세지가 뜰 것입니다.

  #+BEGIN_SRC
  google/api/annotations.proto: File not found.
  myService.proto:4:1: Import "google/api/annotations.proto" was not found or had errors.
  #+END_SRC

  이제부터 이 에러를 고쳐봅시다.
* FIX
  에러 메세지가 아주 명료하니 긴 말 하지말고 바로 방법을 알아보죠.
** googleapis Download
   우선 googleapis 프로젝트를 git clone 명령어를 이용해 원하는 위치에 다운받습니다. 저같은 경우엔 해당 프로젝트의 하위 폴더에 submodule 형태로 내려받았습니다.
   #+BEGIN_SRC
   git clone https://github.com/googleapis/googleapis.git
   #+END_SRC
   
** fix compile command
   컴파일 명령어에 위에서 내려받은 googleapis 프로젝트의 경로를 추가해줍니다.
   #+BEGIN_SRC
   protoc -I. -I./googleapis/googleapis/ --go_out=plugins=grpc:. myService.proto
   #+END_SRC
   
   그럼 이제 정상적으로 컴파일이 완료되는걸 확인할 수 있습니다.

