# gRPC interceptor


이 문서는 [[https://github.com/grpc/grpc-go/blob/master/examples/features/interceptor/README.md][grpc-go Interceptor 문서]]를 번역한 것입니다(+대부분 번역기 발췌, 작성자는 영어를 되게 못해요+).

* Interceptor
  gRPC는 clientConn/server 단위로 인터셉터를 구현하고 설치할 수 있는 간단한 API를 제공합니다. 인터셉터는 각 RPC 호출을 중간에서 가로채는 역할을 합니다. 사용자는 인터셉터를 사용하여 로깅, 인증/권한부여, 메트릭 수집 등 RPC 전체를 아우르는 공용 기능을 수행할 수 있습니다.

* Try it
  #+BEGIN_SRC bash
    go run server/main.go
  #+END_SRC

  #+BEGIN_SRC bash
    go run client/main.go
  #+END_SRC

* Explanation
  gRPC에서 인터셉터는 RPC 호출 유형에 따라 크게 두 가지로 구분할 수 있습니다. 첫 번째는 *unary 인터셉터* 로, unary RPC 호출을 중간에서 가로챕니다. 두 번째는 *stream 인터셉터* 로, stream RPC 호출을 가로챕니다. unary RPC와 stream RPC에 대한 설명은 [[https://grpc.io/docs/what-is-grpc/core-concepts/#rpc-life-cycle][여기]]^(역자 주: 원본에 걸린 링크 주소가 만료되어 다른 주소로 대체)를 참조하세요. 각각의 클라이언트/서버에는 저마다 고유의 unary/stream 인터셉터를 가지고 있습니다. 따라서, gRPC에는 총 4가지 타입의 인터셉터가 있습니다.

** Client-side
*** Unary Interceptor
	[[https://godoc.org/google.golang.org/grpc#UnaryClientInterceptor][UnaryClientInterceptor]]는 클라이언트측 unary 인터셉터입니다. 기본적으로 ~func(ctx context.Context, method string, req, reply interface{}, cc *ClientConn, invoker UnaryInvoker, opts ...CallOption) error~ 와 같은 함수 형태를 가집니다. unary 인터셉터의 구현은 크게 전처리, RPC 호출, 후처리 등 세 부분으로 나눌 수 있습니다.

	사용자는 RPC 호출의 컨텍스트, 메서드 문자열, 전송 요청, CallOption 설정 등의 정보를 전처리에 사용할 수 있습니다. 이 정보들을 이용해 사용자는 RPC 호출을 수정할 수 있습니다. 예를 들어, CallOptions 목록을 검토하여 호출 권한이 있는지 확인하고, 만약 권한이 없다면 oauth2 토큰을 "some-secret-token"과 함께 보조용(fallback)으로 사용하도록 구성하세요. [[https://github.com/grpc/grpc-go/tree/master/examples/features/interceptor][이 예제]]에서는 fallback을 사용하기 위해서 의도적으로 인증을 생략했습니다.

	전처리가 끝났다면, 이제 ~invoker~ 를 호출하여 RPC 요청을 호출할 수 있습니다.

	invoker가 응답과 오류를 반환하면 사용자는 RPC 호출의 사후 처리를 수행할 수 있습니다. 일반적으로 응답 및 오류를 받아 처리합니다. [[https://github.com/grpc/grpc-go/blob/master/examples/features/interceptor/client/main.go][이 예제]]에서는 RPC 응답시각과 에러를 기록합니다.

	~ClientConn~ 에 unary 인터셉터를 추가하려면 ~DialOption~, [[https://godoc.org/google.golang.org/grpc#WithUnaryInterceptor][WithUnaryInterceptor]] 를 사용하여 ~Dial~ 을 구성하세요.

*** Stream Interceptor
	StreamClientInterceptor는 클라이언트측 stream 인터셉터입니다. 기본적으로 ~func(ctx context.Context, desc *StreamDesc, cc *ClientConn, method string, streamer Streamer, opts ...CallOption) (ClientStream, error)~ 와 같은 함수 형태를 가집니다. stream 인터셉터의 구현은 전처리, stream 수행 인터셉트를 포함합니다.

	전처리는 unary 인터셉터와 비슷합니다.

	그러나 unary 인터셉터의 후처리 대신, stream 인터셉터는 사용자의 stream을 가로챕니다. 우선 인터셉터는 ClientStream을 가져와 감싸고(wrapping) 해당 메서드를 오버로딩 합니다. 마지막으로 감싼 ClientStream을 사용자에게 반환합니다.

	[[https://github.com/grpc/grpc-go/blob/master/examples/features/interceptor/client/main.go][이 예제]]에서는 ~ClientStream~ 을 포함한 새 구조체 ~wrappedStream~ 를 정의합니다. 그 후 ~SendMsg~, ~RecvMsg~ 메서드를 ~wrappedStream~ 에 구현(오버로딩)하여 ~ClienStream~ 의 기능을 대체합니다. 이 예제에서는 메세지 유형 정보와 시간 정보를 기록하는게 인터셉터의 작성 이유입니다.

	~ClientConn~ 에 stream 인터셉터를 추가하려면 ~DialOption~, [[https://godoc.org/google.golang.org/grpc#WithStreamInterceptor][WithStreamInterceptor]] 를 사용하여 ~Dial~ 을 구성하세요.

** Server-side
   서버측 인터셉터는 클라이언트측 인터셉터와 비슷하지만 약간 다른 정보를 제공합니다.

*** Unary Interceptor
	[[https://godoc.org/google.golang.org/grpc#UnaryServerInterceptor][UnaryServerInterceptor]]는 서버측 unary 인터셉터입니다. 기본적으로 ~func(ctx context.Context, req interface{}, info *UnaryServerInfo, handler UnaryHandler) (resp interface{}, err error)~ 와 같은 함수 형태를 가집니다.

	자세한 구현 설명은 클라이언트 측 unary Interceptor 섹션을 참고하세요.

	서버에 unary 인터셉터를 추가하려면 ~ServerOption~ [[https://godoc.org/google.golang.org/grpc#UnaryInterceptor][UnaryInterceptor]]를 사용하여 ~NewServer~ 를 구성해야 합니다.

*** Stream Interceptor
	[[https://godoc.org/google.golang.org/grpc#StreamServerInterceptor][StreamServerInterceptor]]는 서버 측 stream 인터셉터입니다. 기본적으로 ~func(srv interface{}, ss ServerStream, info *StreamServerInfo, handler StreamHandler) error~ 함수 형태를 가집니다.

	자세한 구현 설명은 클라이언트 측 stream Interceptor 섹션을 참고하세요. 

	서버에 stream 인터셉터를 추가하려면 ~ServerOption~ [[https://godoc.org/google.golang.org/grpc#StreamInterceptor][StreamInterceptor]]를 사용하여 ~NewServer~ 를 구성해야 합니다.

