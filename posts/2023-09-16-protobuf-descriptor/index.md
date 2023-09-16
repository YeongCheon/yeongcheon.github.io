# Descriptor of Protobuf



* Descriptor의 정의와 목적
protobuf에는 Descriptor라는 개념이 있습니다. 각 출처별로 Descriptor의 정의에 대해 알아보고자 합니다.
** 출처: Buf
*** 정의
[[https://buf.build/docs/reference/descriptors][이 곳]]에 설명된 정의에 따르면 Descriptor의 정의는 다음과 같습니다.

#+BEGIN_QUOTE
The term “descriptors” refers to models that describe the data types defined in Protobuf sources. They resemble an AST (Abstract Syntax Tree) of the Protobuf IDL, using Protobuf messages for the tree nodes.

"descriptor"라는 용어는 Protobuf 소스에 정의된 데이터 유형을 설명하는 모델을 의미합니다. 이는 트리 노드에 Protobuf 메시지를 사용하는 Protobuf IDL의 AST(추상 구문 트리)와 유사합니다.
#+END_QUOTE

*** 목적
- *Code generation*
- *Reflection*
- *Dynamic messages & dynamic RPC*
** 출처: Chat GPT
*** 정의
#+BEGIN_QUOTE
descriptor는 protobuf 메세지와 서비스에 대한 메타데이터를 이진 형태로 포함하는 파일입니다. 이 메타데이터는 메시지의 구조, 필드의 이름, 유형 등을 설명하며, 런타임 중에 프로그램이 protobuf 메세지를 인식하고 다루는 데 사용됩니다.
#+END_QUOTE
*** 목적
- *동적 로딩(Dynamic Loading)*: 프로그램이 런타임 중에 Protocol Buffers 메시지를 동적으로 로드하고 사용할 때 유용합니다. 이는 플러그인 시스템이나 플러그인 아키텍처를 구현할 때 특히 유용합니다.
- *API 문서 생성*: 디스크립터 세트를 사용하여 프로그램의 API 문서를 생성할 수 있습니다. 이는 서비스와 메시지를 설명하고 문서화하는 데 도움이 됩니다.
- *기타 메타정보 활용*: 프로그램이 Protocol Buffers 메시지를 처리하거나 저장할 때 메타정보가 필요한 경우에 활용될 수 있습니다.
* Descriptor(.pb) 생성 방법
[[https://grpc.io/docs/protoc-installation/][protoc]] 명령어를 기준으로 Descriptor를 생성하는 방법은 다음과 같습니다.

#+BEGIN_SRC bash
protoc \
    --include_imports \
    --proto_path=. \
    --descriptor_set_out=example_descriptor.pb \ # descriptor 파일 생성
    example.proto
#+END_SRC

* 실제 사용 예
제가 실제로 겪어본 Descriptor가 사용되는 케이스는 다음과 같습니다.
- [[https://github.com/fullstorydev/grpcurl#descriptor-sources][Descriptor Source를 활용한 grpcurl 호출]]
- [[https://cloud.google.com/endpoints/docs/grpc/transcoding?hl=ko][GCP endpoints 제품에서 HTTP/JSON을 gRPC로 트랜스코딩]]

* 마치며
Descriptor는 proto 파일을 실제 개발자가 사용할 [[https://protobuf.com/docs/descriptors][코드를 생성할 때 사용되는 요소]]입니다. 보통 Server/Client 프로그램을 작성할 때는 다룰 일이 많지 않습니다만, 위에서 언급한 예처럼 사용할 일이 종종 있으니 간단한 내용만이라도 숙지하고 있는 것이 좋겠습니다.

