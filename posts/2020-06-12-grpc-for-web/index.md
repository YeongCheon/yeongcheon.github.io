# gRPC를 웹브라우저에 호출해보자(a.k.a Typescript)


* 목표
  브라우저에서 rpc를 호출하고 싶습니다. 하지만 2020년 7월 기준으로 브라우저에서 gRPC의 rpc를 직접 호출하는 건 불가능합니다. 그치만 [[https://www.envoyproxy.io/][envoy Proxy]]와 함께라면 가능합니다. *이 문서에서는 grpc-web을 이용해 생성된 Typescript 파일을 이용해 [[https://grpc.io/docs/what-is-grpc/core-concepts/#server-streaming-rpc][server streaming rpc]]를 호출하는 방법을 설명합니다.* 서버는 [[https://go.dev/][Go]], 클라이언트는 [[https://www.typescriptlang.org/][Typescript]] 언어를 사용합니다.

* proto 파일 작성
  특정 채팅방에 접속해서 server stream을 얻는 rpc를 작성해봅시다. 양방향 스트림이 아니라 서버사이드 스트림으로 한 이유는 현재 grpc-web에서는 양방향 스트림을 지원하지 않기 때문입니다(클라이언트 스트림도 지원하지 않습니다). 만약 채팅방에 메세지를 전송하고 싶다면 해당 기능을 가진 rpc를 별도로 작성하여야 합니다.
  #+BEGIN_SRC protobuf
syntax = "proto3";

package chat;

service ChatService {
    rpc Entry(EntryRequest) returns (stream ChatMessageResponse);
    // rpc Broadcast(BroadcastRequest) returns (stream BroadcastResponse);
}

message EntryRequest {
    string room_id = 1;
}

message ChatMessageResponse {
    string content = 1;
}

  #+END_SRC
* [[https://github.com/grpc/grpc-go][grpc-go]] 설치
  서버 코드를 생성하기 위해서 [[https://github.com/grpc/grpc-go][grpc-go]]를 설치합니다. 자세한 내용은 [[https://github.com/grpc/grpc-go/blob/master/README.md#installation][여기]]를 참고합니다.

* [[https://github.com/grpc/grpc-web][grpc-web]] 설치
  클라이언트 코드를 생성하기 위해서 [[https://github.com/grpc/grpc-web][grpc-web]]을 설치합니다. 자세한 내용은 [[https://www.npmjs.com/package/grpc-web][여기]]를 참고합니다.
* generate source code
  이제 위에서 작성한 proto파일을 기반으로 각 언어별 맞춤 파일을 생성해야 합니다. proto 파일이 있는 위치에서 ~gen/go~, ~gen/typescript~ 폴더를 각각 생성 후 터미널 창을 열어서 아래의 명령어를 입력하세요.
** generate source code for Go
   #+BEGIN_SRC bash
   protoc -I. \
       --go_out=plugins=grpc:gen/go *.proto
   #+END_SRC
** generate source code for Typescript
   #+BEGIN_SRC bash
   protoc -I. \
       --js_out=import_style=commonjs,binary:gen/typescript \
       --grpc-web_out=import_style=typescript,mode=grpcwebtext:gen/typescript *.proto
   #+END_SRC
* envoy 설치
  위에서 말했다시피 브라우저에서 RPC를 직접 호출할 순 없습니다. envoy Proxy를 통해서 간접적으로 호출해야 합니다. 이 예제에서는 envoy를 도커를 이용해서 실행하는 방법을 설명합니다.
** envoy.yaml 작성
   #+BEGIN_SRC yaml
admin:
  access_log_path: /tmp/admin_access.log
  address:
    socket_address: { address: 0.0.0.0, port_value: 9901 }

static_resources:
  listeners:
  - name: listener_0
    address:
      socket_address: { address: 0.0.0.0, port_value: 8080 }
    filter_chains:
    - filters:
      - name: envoy.filters.network.http_connection_manager
        typed_config:
          "@type": type.googleapis.com/envoy.config.filter.network.http_connection_manager.v2.HttpConnectionManager
          codec_type: auto
          stat_prefix: ingress_http
          stream_idle_timeout: 0s
          route_config:
            name: local_route
            virtual_hosts:
            - name: local_service
              domains: ["*"]
              routes:
              - match: { prefix: "/" }
                route:
                  cluster: chat_service
                  max_grpc_timeout: 0s
                  timeout:
                    seconds: 0
              cors:
                allow_origin_string_match:
                - prefix: "*"
                allow_methods: GET, PUT, DELETE, POST, OPTIONS
                allow_headers: keep-alive,user-agent,cache-control,content-type,content-transfer-encoding,custom-header-1,x-accept-content-transfer-encoding,x-accept-response-streaming,x-user-agent,x-grpc-web,grpc-timeout,authorization
                max_age: "1728000"
                expose_headers: custom-header-1,grpc-status,grpc-message
          http_filters:
          - name: envoy.filters.http.grpc_web
          - name: envoy.filters.http.cors
          - name: envoy.filters.http.router
  clusters:
  - name: chat_service
    connect_timeout: 0.25s
    type: logical_dns
    http2_protocol_options: {}
    lb_policy: round_robin
    load_assignment:
      cluster_name: cluster_0
      endpoints:
        - lb_endpoints:
            - endpoint:
                address:
                  socket_address:
                    address: 127.0.0.1
                    port_value: 9090
   #+END_SRC
** envoy Dockerfile 설정
   위에서 작성한 설정파일을 사용한 DockerFile을 작성합니다.
   #+BEGIN_SRC Dockerfile
FROM envoyproxy/envoy-dev:latest
COPY ./envoy.yaml /etc/envoy/envoy.yaml
CMD /usr/local/bin/envoy -c /etc/envoy/envoy.yaml
   #+END_SRC

** docker 실행
   이제 작성된 ~Dockerfile~ 을 이용해서 실제로 컨테이너를 빌드하고 실행시켜 봅시다.
   #+BEGIN_SRC bash
    sudo docker build -t chat-envoy -f ./envoy.Dockerfile .
    sudo docker run -d -p 8080:8080 -p 9901:9901 --network=host chat-envoy   
   #+END_SRC 

   실행 후 [[http://localhost:9901][http://localhost:9901]] 에 접근해서 제대로 envoy가 실행되었는지 확인해봅니다. 제대로 실행되었다면 아래와 같은 화면이 뜹니다.

   [[/images/2020-07-20 23-40-09.png]]

* server 작성
  이제 서버 코드를 작성해봅시다. 이번 예제에선 채팅 메세지 전송 기능은 없으니 stream에 초당 한 번씩 메세지를 보내는 서버를 작성하겠습니다. 

  ~go mod init~ 명령어를 이용해 Go 프로젝트를 하나 생성 후, 위에서 proto 파일을 이용해 생성한 ~gen/go~ 폴더를 Go 프로젝트 안으로 복사한 후 해당 폴더명을 pb로 바꿔줍니다(protobuf의 약자입니다). 이 과정을 모두 마무리 하면 Go 프로젝트의 파일 트리는 아래와 비슷해집니다.
  
  #+BEGIN_SRC
  .
  ├── go.mod
  └── pb
      └── chatService.pb.go
  #+END_SRC

  이제 Go 프로젝트의 root 폴더에 ~main.go~를 작성합니다.

  #+BEGIN_SRC go
package main

import (
    pb "MY_GO_MODULE/pb" // go mod init 명령어를 사용할 때 입력한 module명을 써주세요.
    "google.golang.org/grpc"
    "io"
    "log"
    "net"
    "time"
)

type ChatServer struct {
}

func (p *ChatServer) Entry(entryRequest *pb.EntryRequest, stream pb.ChatService_EntryServer) error {
    for {
        <-time.Tick(1 * time.Second)
        err := stream.Send(&pb.ChatMessageResponse{
            Content: "tick",
        })

        if err == io.EOF {
            return err
        }
    }
}

func main() {
    lis, err := net.Listen("tcp", ":9090")
    if err != nil {
        log.Fatalf("failed to listen : %v", err)
    }

    opts := []grpc.ServerOption{}

    grpcServer := grpc.NewServer(opts...)
    pb.RegisterChatServiceServer(grpcServer, &ChatServer{})
    if err := grpcServer.Serve(lis); err != nil {
        panic(err)
    }
}
  #+END_SRC

  이제 ~go build~ 명령어를 이용해 바이너리 파일을 생성하고, 생성된 파일을 실행해보세요.

* client 작성
  서버가 준비되었으니 이제 클라이언트(웹 페이지)를 작성해봅시다. 

  서버를 작성할때와 마찬가지로 위에서 proto 파일을 이용해 생성한 ~gen/typescript~ 폴더를 본인의 웹 프로젝트 안으로 복사한 후 해당 폴더명을 pb로 바꿔줍니다.

  특정 프론트 프레임워크에 종속되지 않도록 pure Typescript로 아주 간단한 클래스만 작성해보겠습니다. 본인이 선호하는 환경(angular, react ,vue, etc..)에서 아래의 코드를 이용해 서버에 접속해보세요.

  #+BEGIN_SRC typescript
import { ChatServiceClient } from './pb/ChatServiceServiceClientPb';
import { ChatMessageResponse, EntryRequest } from './pb/chatService_pb';
import { ClientReadableStream } from 'grpc-web'; // https://www.npmjs.com/package/grpc-web

class ChatService {
    client: ChatServiceClient;
    stream: ClientReadableStream<ChatMessageResponse>;
  
    constructor() {
    }

    public connect(url: string) {
        this.client = new ChatServiceClient(url);

        const entryRequest = new EntryRequest();

        const metadata = {
          // authorization: SOME_TOKEN
        }

        this.stream = this.client.entry(entryRequest, metadata);

        this.stream.on('error', (err)=>{
          console.error(err.code, err.message);
        });


        this.stream.on('status', (status)=>{
          console.log(status.code, status.details);
        });

        this.stream.on('data', (data: ChatMessageResponse)=>{
          console.log(data.getContent());
        });
    }
}
  #+END_SRC

* 마무리
  이로써 웹에서 server stream RPC를 호출하는 작업이 모두 마무리 되었습니다. 처음에 언급했다시피 채팅방에 메세지를 전송하는 기능을 구현하려면 메세지 전송용 unary RPC를 별도로 작성하여야 합니다. unary RPC에 대한 예제는 웹 상에 자료가 많이 나와 있으므로 [[https://blog.breezymind.com/2019/11/19/grpc-%25EA%25B5%25AC%25ED%2598%2584-unary-rpc-1/][여기]][[https://velog.io/@kyusung/grpc-web-example][저기]] 참고하시길 바랍니다.

