# 프로토 버퍼 언어 안내(proto3) 번역본


매번 [[https://developers.google.com/protocol-buffers/docs/proto3#simple][영어문서]] 보기 빡쳐서 직접 번역기 돌려가며 쓰는 번역본. +번역기와 의역, 오역 범벅입니다.+ 가급적이면 공식 문서를 보세요.

* Language Guide(proto3)
** Defining A Message Type
   우선 매우 간단한 예제를 살펴보겠습니다.. 검색 요청을 위한 메세지 타입을 정의하고자 하는데 이 메세지는 쿼리 문자열^{(역자 주: query)}, 요청 페이지^{(역자 주: page_number)}, 그리고 페이지당 결과 갯수^{(역자 주: result_per_page)}를 가지고 있습니다. 

   #+BEGIN_SRC proto
   syntax = "proto3";
   
   message SearchRequest {
     string query = 1;
     int32 page_number = 2;
     int32 result_per_page = 3;
   }

   #+END_SRC

   + 첫 번째 줄은 이 파일이 *proto3* 문법을 사용하고 있음을 나타냅니다. 만약 이 구문이 없으면 컴파일러는 이 파일이 [[https://developers.google.com/protocol-buffers/docs/proto][proto2]] 문법을 사용하고 있다고 인식합니다. 이 구문은 무조건 파일의 제일 첫 번째 줄에 와야하며 앞에 공백이나 주석조차 허용하지 않습니다.
   + ~SearchRequest~ 메세지는 세 가지 필드를 정의하고 있습니다(이름/값 쌍), 각 필드는 이름과 타입으로 이루어져 있습니다.
*** Specifying Field Types
	위 예제에서 모든 필드는 [[https://developers.google.com/protocol-buffers/docs/proto3#scalar][스칼라 타입]]입니다(int32 타입 2개, string 타입 1개). 그렇지만 [[https://developers.google.com/protocol-buffers/docs/proto3#enum][열거형]]이나 다른 메세지를 타입으로 지정할 수 있습니다.
*** Assigning Field Numbers
	위에서 봤듯이, 각 필드는 저마다 *고유한 번호*를 가지고 있습니다. 이 필드 번호들은 메세지를 [[https://developers.google.com/protocol-buffers/docs/encoding][바이너리 포맷]]으로 변환할 때 사용되므로 한번 정의한 이 후 절대로 바꿔서는 안됩니다. 인코딩 시 필드 번호는 1-15 범위까지는 1 바이트, 16에서 2047 범위까지는 2 바이트를 사용합니다(자세한 내용은 [[https://developers.google.com/protocol-buffers/docs/encoding#structure][proto Buffer Encoding]]에서 확인). 그러므로 자주 발생하는 메세지 요소들은 1-15 범위에 할당하는게 좋습니다. 나중에 빈번하게 접근할 요소가 추가될 수 있으므로 미리 여유 공간을 확보해 놓으세요.

	지정할 수 있는 가장 작은 필드 번호는 1, 가장 큰 필드 번호는 2^{29} - 1 또는 536870911 입니다. 그리고 19000-19999 범위의 필드 번호는 프로토콜 버퍼 구현을 위해 미리 예약되어 있으므로 사용할 수 없습니다. 만약 당신의 ~.proto~ 파일에서 예약된 필드 번호 중 하나를 사용하면 프로토콜 버퍼 컴파일러는 에러를 반환할 것입니다. 이와 마찬가지로, 이전에 [[https://developers.google.com/protocol-buffers/docs/proto3#reserved][예약된]] 필드 번호도 사용할 수 없습니다.
*** Specifying Field Rules
	메세지 필드는 다음 중 하나일 수 있습니다,

	+ 단일 필드: 잘 디자인 된 메세지는 0개 또는 하나의 단일 필드 형식을 갖습니다(하나 이하). 그리고 단일 필드는 프로토 버퍼3 문법에서 기본 필드입니다.
	+ 반복 필드: 이 필드는 0을 포함해서 여러번 반복될 수 있습니다. 반복되는 값은 순서는 유지됩니다.

	~proto3~ 에서, 스칼라 숫자 타입 ~반복~ 필드는 기본적으로 ~압축(packed)~ 인코딩을 사용합니다.

	~압축~ 인코딩에 대해서는 [[https://developers.google.com/protocol-buffers/docs/encoding#packed][Protocol Buffer Encoding]] 에서 자세히 알아볼 수 있습니다.
*** Adding More Message Types
	여러개의 메세지 타입을 하나의 ~.proto~ 파일 안에 정의할 수 있습니다. 만약 당신이 다수의 서로 연관된 메세지를 정의할 때 매우 유용할 것입니다. 예를 들어, ~SearchResponse~ 메세지를 응답 형식으로 작성하고 싶다면 같은 ~.proto~ 파일에 추가할 수 있습니다.
	#+BEGIN_SRC proto
	message SearchRequest {
      string query = 1;
      int32 page_number = 2;
      int32 result_per_page = 3;
	}
   
	Message Searchresponse {
      ...
	}

	#+END_SRC
*** Adding Comments
	C/C++ 처럼 ~//~ 또는 ~/* ... */~ 스타일의 주석 문법을 ~.proto~ 파일에서 사용할 수 있습니다.
	#+BEGIN_SRC proto
	/* SearchRequest represents a search query, with pagination options to
     * indicate which results to include in the response. */

	message SearchRequest {
      string query = 1;
      int32 page_number = 2;  // Which page number do we want?
      int32 result_per_page = 3;  // Number of results to return per page.
	}

	#+END_SRC
*** Reserved Fields
	필드를 완전히 삭제 또는 주석 처리하여 메세지를 업데이트 하면 향후 사용자가 메세지를 업데이트 할 때  필드 번호를 재사용 할 수 있습니다. 이는 만약 이전 버전의 ~.proto~ 파일을 사용할 때 데이터 손상이나 개인정보 문제 등등 심각한 문제가 발생할 수 있습니다. 삭제된 필드의 필드 번호(또는 json 직렬화 문제를 일으킬 수 있는 이름)를 예약하도록 지정하면 이러한 문제가 발생하는 것을 피할 수 있습니다. 만약 향후 사용자가 예약된 필드 식별자^{(역자 주: 필드 번호 또는 이름)}를 사용하려고 할 경우 프로토콜 버퍼 컴파일러가 경고를 해 줄 것입니다.
	#+BEGIN_SRC proto
	message Foo {
      reserved 2, 15, 9 to 11;
      reserved "foo", "bar";
	}
	#+END_SRC
	동일한 ~예약~ 구문에서는 필드 이름과 이름 번호를 혼용할 수 없습니다.
*** What's Generated From Your *.proto*?
	~.porto~ 파일을 프로토 버퍼 컴파일러를 이용해 컴파일 하면 컴파일러는 당신이 선택한 언어로 코드를 생성합니다. 당신은 해당 메세지 타입에 해당하는 필드값 입출력, 메세지 출력 스트림에 대한 직렬화 작업, 그리고 입력 스트림에 대한 파싱 작업을 수행해야 합니다.

	+ *C++* 언어의 경우, 컴파일러는 ~.proto~ 파일마다 각 메세지별로 클래스 파일, ~.h~ , 그리고 ~.cc~ 파일을 생성합니다.
	+ *Python* 언어는 조금 다릅니다. 파이썬 컴파일러는 ~.proto~ 파일에 일는 메세지마다 정적 디스크립터 모듈을 생성한 후 메타 클래스와 런타임에 필요한 데이터 접근 클래스를 생성하는 데 사용합니다.
	+ *Go * 언어의 경우, 컴파일러는 파일 안의 각 메세지마다 ~.pb.go~ 파일을 생성합니다.
	+ *Ruby* 언어의 경우, 컴파일러는 메세지 타입마다 *.rb* 파일을 생성합니다.
	+ *Objective-C* 언어의 경우, 컴파일러는 파일에 설명 된 각 메시지 유형에 대한 클래스와 함께 각 ~.proto~ 에서 ~pbobjc.h~ 및 ~pbobjc.m~ 파일을 생성합니다.
	+ *C#* 언어의 경우, 컴파일러는 각 ~.proto~ 파일에 설명 된 각 메시지 유형에 대한 클래스와 함께 ~.cs~ 파일을 생성합니다.
	+ *Dart* 언어의 경우, 컴파일러는 파일의 각 메시지 유형에 대한 클래스와 함께 ~.pb.dart~ 파일을 생성합니다.
	다른 언어에 대한 튜토리얼을 통해 API 사용법을 좀 더 자세히 알아볼 수 있습니다(proto3 버전은 출시 예정). 더 자세한 [[https://developers.google.com/protocol-buffers/docs/reference/overview][API 사용법]]은 관련 API를 참조(proto3 버전은 출시 예정).
** Scalar Value Types
   스칼라 메세지 필드는 다음 유형 중 하나를 가질 수 있습니다. 아래의 표는 ~.proto~ 파일에 지정된 유형과 그에 상응하는 자동 생성된 클래스의 타입을 보여줍니다.
| .proto type | Notes                                                                                                                       | C++ Type | Java Type  | Python Type | Go Type | Ruby Type                      | C# Type    | PHP            | Dart Type |
|-------------+-----------------------------------------------------------------------------------------------------------------------------+----------+------------+-------------+---------+--------------------------------+------------+----------------+-----------|
| double      |                                                                                                                             | double   | double     | float       | float64 | Float                          | double     | float          | double    |
| float       |                                                                                                                             | float    | float      | float       | float32 | Float                          | float      | float          | double    |
| int32       | 가변 길이 인코딩을 사용합니다. 음수를 인코딩 하는 데 비효율적입니다. 필드에 음수가 있는 경우 sint32를 사용하는 게 좋습니다. | int32    | int        | int         | int32   | Fiexnum or Bignum(as required) | int        | integer        | int       |
| int64       | 가변 길이 인코딩을 사용합니다. 음수를 인코딩 하는 데 비효율적입니다. 필드에 음수가 있는 경우 sint64를 사용하는 게 좋습니다. | int64    | long       | int/long    | int64   | Bignum                         | long       | integer/string | Int64     |
| unit32      | 가변 길이 인코딩을 사용합니다.                                                                                              | unit32   | int        | int/long    | uint32  | Fixnum or Bignum(as required)  | uint       | integer        | int       |
| unit64      | 가변 길이 인코딩을 사용합니다.                                                                                              | unit64   | long       | int/long    | uint64  | Bignum                         | ulong      | integer/string | Int64     |
| sint32      | 가변 길이 인코딩을 사용합니다. 부호있는 int 값. 일반 int32보다 음수를 더 효율적으로 인코딩합니다.                           | int32    | int        | int         | int32   | Fixnum or Bignum(as required)  | int        | integer        | int       |
| sint64      | 가변 길이 인코딩을 사용합니다. 부호있는 int 값. 일반 int64보다 음수를 더 효율적으로 인코딩합니다.                           | int64    | long       | int/long    | int64   | Bignm                          | long       | integer/string | Int64     |
| fixed32     | 항상 4 바이트 크기를 갖습니다. 2^{28} 이상의 값을 사용할 경우 uint32보다 효율적입니다.                                       | uint32   | int        | int/long    | uint32  | Fixnum or Bignum(as required)  | uint       | integer        | int       |
| fixed64     | 항상 8 바이트 크기를 갖습니다. 2^{28} 이상의 값을 사용할 경우 uint64보다 효율적입니다.                                       | unit64   | long       | int/long    | uint64  | Bignum                         | ulong      | integer/string | Int64     |
| sfixed32    | 항상 4 바이트 크기를 갖습니다.                                                                                              | int32    | int        | int         | int32   | Fixnum or Bignum(as required)  | int        | integer        | int       |
| sfixed64    | 항상 8 바이트 크기를 갖습니다.                                                                                              | int64    | long       | int/long    | int64   | Bignum                         | long       | integer/string | int64     |
| bool        |                                                                                                                             | bool     | boolean    | bool        | bool    | TrueClsas/FalseClass           | bool       | boolean        | bool      |
| string      | 문자열 타입은 항상 UTF-8 또는 7비트 아스키 텍스트로 구성되어야 하며, 2^{32} 길이보다 길 수 없습니다.                         | string   | String     | str/unicode | string  | String(UTF-8)                  | string     | string         | String    |
| bytes       | 2^{32} 길이 이하의 임이의 바이트 시퀀스를 포함 할 수 있습니다.                                                                                                              | string   | ByteString | str         | []byte  | String(ASCII-8BIT)             | ByteString | string         | List<int> |
   [[https://developers.google.com/protocol-buffers/docs/encoding][프로토콜 버퍼 인코딩]]에서 메세지를 직렬화 할 때 이러한 타입딀 인코딩 되는 방법에 대해 자세히 알아볼 수 있습니다.
** Default Values
   메세지를 파싱할 때 인코딩 된 메세지에 특정 특이 요소가 포함되어 있지 않으면 파싱된 객체의 해당 필드가 해당 필드의 기본값으로 설정됩니다. 이 기본값은 유형별로 다릅니다.

   + String 타입의 경우, 기본값은 빈 문자열 값입니다.
   + bytes 타입의 경우, 기본값은 빈 bytes 값입니다.
   + bools 타입의 경우, 기본값은 false 입니다.
   + 숫자 타입의 경우, 기본값은 0 입니다.
   + [[https://developers.google.com/protocol-buffers/docs/proto3#enum][enums]] 타입의 경우, 기본값은 가장 처음 정의된 enum 값이며 0이어야 합니다.
   + 메세지 필드의 경우, 필드 값은 셋팅되지 않습니다. 정확한 값은 종속된 언에 따라 다릅니다. 자세한 내용은 [[https://developers.google.com/protocol-buffers/docs/reference/overview][코드 생성 가이드]]를 참고하세요.
   반복 필드 타입의 기본값은 빈 값입니다(일반적으로 프로그래밍 언어에선 빈 list 타입을 갖습니다).

   스칼라 메세지 필드의 경우, 한번 메세지가 파싱되면 필드가 기본값(ex: boolean 필드가 false로 설정)이 명시적으로 설정었는지 또는 설정되지 않았는지 알 방법이 없습니다. 메세지 타입을 정의 할 때 주의하세요. 예들 들어, 만약 당신이 이런 동작을 원치 않는다면 일부 필드값을 ~false~ 로 설정하는 작업은 하지 마세요^{(역자 주: 번역이 잘 된건지 모르겠네요..)}. 또한 스칼라 타입 메세지 필드가 기본값 설정된 경우, 그 값은 직렬화 되지 않습니다.

   [[https://developers.google.com/protocol-buffers/docs/reference/overview][코드 생성 가이드]]에서 당신이 사용하는 언어에 대해서 기본적으로 어떻게 코드를 생성하는지 자세히 알아보세요.
** Enumerations
   메세지를 선언할 때, 미리 정의된 값들이 필요한 경우가 있습니다. 예를 들어, 당신이 각 ~SearchRequest~ 마다 ~corpus~ 필드를 추가하고 싶다고 가정해봅시다. corpus 필드는 ~UNIVERSAL~, ~WEB~, ~IMAGES~, ~LOCAL~, ~NEWS~, ~PRODUCTS~, ~VIDEO~ 값 중 하나를 가질 수 있습니다. 당신의 메세지 타입에 간단하게 열거형^{(역자 주: enum)} 상수값을 추가할 수 있습니다.

   #+BEGIN_SRC proto
   message SearchRequest {
     string query = 1;
     int32 page_number = 2;
     int32 result_per_page = 3;
     enum Corpus {
       UNIVERSAL = 0;
       WEB = 1;
       IMAGES = 2;
       LOCAL = 3;
       NEWS = 4;
       PRODUCTS = 5;
       VIDEO = 6;
     }
     Corpus corpus = 4;
   }
   #+END_SRC
   위 예제에서 보시다시피, ~Corpus~ enum의 첫번재 상수값은 0이어야 합니다. 모든 enum은 *무조건* 첫번째 상수값으로 0을 가지고 있어야 합니다. 그 이유는 아래와 같습니다.
   + 0이어야 하는 이유는 0이 숫자 타입의 [[https://developers.google.com/protocol-buffers/docs/proto3#default][기본값]]이기 때문입니다.
   + 0이 첫번째 요소에 있어야 하는 이유는 [[https://developers.google.com/protocol-buffers/docs/proto][proto2]] 문법과 호환을 위해서인데, proto2에선 항상 enum의 첫번째 요소가 기본값입니다.
   동일한 enum 값에 서로 다른 별칭을 지정할 수 있습니다. 이를 위해서는 ~allow_alias~ 옵션이 ~true~ 이어야 합니다. ~true~ 로 지정하지 않고 별칭을 사용한다면 컴파일러는 에러를 발생할 것입니다.
   #+BEGIN_SRC proto
   enum EnumAllowingAlias {
     option allow_alias = true;
     UNKNOWN = 0;
     STARTED = 1;
     RUNNING = 1;
   }
   enum EnumNotAllowingAlias {
     UNKNOWN = 0;
     STARTED = 1;
     // RUNNING = 1;  // Uncommenting this line will cause a compile error inside Google and a warning message outside.
   }
   #+END_SRC
   열거형 상수값의 범위는 32bit integer 까지입니다. ~enum~ 값으로 [[https://developers.google.com/protocol-buffers/docs/encoding][가변형 인코딩]] 또는 음수값을 사용할 경우 비효율적이므로 추전하지 않습니다. 이전 예제에서 보았듯이 당신은 ~enum~ 을 메세지와 함께 정의할 수 있고 또한 ~enum~ 은 당신의 ~.proto~ 파일 내부의 다른 메세지에도 재사용 할 수 있습니다. 또한 ~enum~ 은 한 메세지 안에서 정의하고 다른 다른 메세지에서도 사용할 수 있습니다. 다음의 문법을 사용해서요. ~MessageType.EnumType~

   ~enum~ 이 포함된 ~.proto~ 파일을 프로토콜 버퍼 컴파일러로 실행할 때, 생성된 코드는 Java, C++에 해당하는 enum을 가지며, 파이썬의 경우에는 런타임 시 사용되는 정수 상수값으로 이루어진 ~EnumDescriptor~ 클래스가 생성됩니다.

   역직렬화 할 때는, 인식할 수 없는 enum 값은 언어마다 표현방식이 다르긴 하지만 일단은 메세지 안에 보존됩니다. C++이나 Go처럼 지정된 enum값을 벗어난 개방형 enum을 지원하는 언어의 경우, 알 수 없는 enum 값은 기본 정수값으로 저장됩니다. Java와 같이 비개방 enum의 경우, 인식되지 않은 값을 사용하여 표현하며 기본 정수는 특수 접근자를 이용해 접근할 수 있습니다. 두 경우 모두 메세지가 직렬화 될 때 인식할 수 없는 값이 메세지와 함께 직렬화 된다.

   ~enum~ 타입이 당신의 어플리케이션에서 어떻게 동작하는지 자세히 알고싶다면, 해당 언어에 해당하는 [[ In either case, if the message is serialized the unrecognized value will still be serialized with the message. ][코드 생성 가이드]]를 참고하세요.
*** reserved Values
	만약 enum 타입 값을 완전히 제거하거나 주석처리하여 enum을 업데이트 하면 향 후 사용자가 직접 업데이트 할 때 해당 숫자 값을 재사용 할 수 있습니다. ~.proto~ 파일의 이전버전을 사용할 경우 이는 데이터 손상, 개인 정보 보호 이슈 등등 심각한 문제가 발생할 수 있습니다. 삭제된 enum 값의 필드 번호(또는 json 직렬화 문제를 일으킬 수 있는 이름)를 ~예약~ 하도록 지정하면 이러한 문제가 발생하는 것을 피할 수 있습니다. 만약 향후 사용자가 예약된 enum 값의 식별자를 사용하려고 할 경우 프로토콜 버퍼 컴파일러가 경고를 해 줄 것입니다. ~max~ 키워드를 사용하면 예약 된 숫자 값 범위를 최대 값^{(역자 주: 32bit integer의 최댓값)}까지 지정할 수 있습니다.
   #+BEGIN_SRC proto
   enum Foo {
     reserved 2, 15, 9 to 11, 40 to max;
     reserved "FOO", "BAR";
   }
   #+END_SRC
   동일한 ~예약~ 구문에서는 필드 이름과 이름 번호를 혼용할 수 없습니다.
** Using Other Message Types
   다른 메세지 타입을 필드 타입으로 사용할 수 있습니다. 예를 들어, ~SearchResponse~ 메세지 타입에 ~Result~ 메세지 타입을 포함하고 싶다고 가정해봅시다. 우리는 ~Result~ 메세지 타입을 동일한 ~.proto~ 파일에 정의한 후 ~SearchResposne~ 안에 ~Result~ 필드 타입을 명시하기만 하면 됩니다.
   #+BEGIN_SRC proto
   message SearchResponse {
     repeated Result results = 1;
   }

   message Result {
     string url = 1;
     string title = 2;
     repeated string snippets = 3;
   }
   #+END_SRC
*** Importing Definitions
	위 예제에서, ~Result~ 메세지는 ~SearchResponse~ 메세지와 동일한 파일 안에 정의되었습니다. 만약 다른 ~.proto~ 파일 안에 정의된 메세지를 필드 타입으로 사용하고 싶을 경우엔 어떻게 해야 할까요?

	다른 ~.proto~ 파일을 사용하고 싶을 땐 /importing/ 문법을 사용할 수 있습니다. 다른 ~.proto~ 파일을 import 할 땐 당신의 파일에 import 구문을 넣으면 됩니다.
   #+BEGIN_SRC proto
   import "myproject/other_protos.proto";
   #+END_SRC
   기본적으로 당신은 ~.proto~ 파일을 직접 import 하는 방법만 사용할 수 있습니다. 그치만 당신은 가끔씩 `.proto` 파일을 다른 위치로 옮겨야 할 때가 있습니다. ~.proto~ 파일을 직접 이동하고 모든 참고를 한번에 업데이트 하는 대신, 더미 ~.proto~ 파일을 기존 위치에 두어 `import public` 구문을 통해 모든 import를 옮겨진 새 위치로부터 가져올 수 있습니다. ~import public~ 개념은 import 된 다른 파일의 `import public` 문법에 의존적입니다. 예를 들어봅시다.
   #+BEGIN_SRC proto
   // new.proto
   // All definitions are moved here
   #+END_SRC
   #+BEGIN_SRC proto
   // old.proto
   // This is the proto that all clients are importing.
   import public "new.proto";
   import "other.proto";
   #+END_SRC
   #+BEGIN_SRC proto
   // client.proto
   import "old.proto";
   // You use definitions from old.proto and new.proto, but not other.proto
   #+END_SRC
   프로토콜 컴파일러는 명령어 실행 시 ~-I~ / ~-proto_path~ 옵션을 사용하여 지정된 폴더들 안에서 import 된 파일들을 찾아옵니다. 만약 옵션을 지정하지 않은 경우엔 현재 컴파일러가 실행된 폴더를 기준으로 파일들을 찾아옵니다. ~--proto_path~ 옵션을 당신의 프로젝트 루트로 설정하고 모든 import문에 명시된 파일들을 가져오는 게 보통입니다.
*** Using proto2 Message Types
	proto2 메세지 타입을 proto3 메세지에 가져와서 사용하는 게 가능하고, 그 반대도 가능합니다. 그러나 proto2의 enum 타입은 proto3 구문에서 직접적으로 사용할 수 없습니다(import한 proto2 메세지에서 사용하는 경우는 괜찮습니다).
** Nested Types
   다른 메세지 타입 안에서 메세지 타입을 정의하고 사용할 수 있습니다. 아래의 예제에선 ~Result~ 메세지를 `SearchResponse` 메세지 안에서 정의하고 있습니다.
   #+BEGIN_SRC proto
   message SearchResponse {
     message Result {
       string url = 1;
       string title = 2;
       repeated string snippets = 3;
     }
     repeated Result results = 1;
   }
   #+END_SRC
   만약 부모 메세지 타입 밖에서 해당 메세지를 재사용 하고 싶다면 ~Parent.Type~ 처럼 사용할 수 있습니다.
   #+BEGIN_SRC proto
   message SomeOtherMessage {
     SearchResponse.Result result = 1;
   }
   #+END_SRC
   원하는 만큼 메세지를 중첩할 수 있습니다.
   #+BEGIN_SRC proto
   message Outer {                  // Level 0
     message MiddleAA {  // Level 1
       message Inner {   // Level 2
         int64 ival = 1;
         bool  booly = 2;
       }
     }
     message MiddleBB {  // Level 1
       message Inner {   // Level 2
         int32 ival = 1;
         bool  booly = 2;
       }
     }
   }
   #+END_SRC
** Updating A Message Type
   기존에 존재하는 메세지 타입이 더이상 요구사항을 충족하지 못하는 경우가 생길 수 있습니다. 예를 들어, 메세지 타입 형식은 유지하고 새로운 필드 타입을 추가하여 메세지 타입을 업데이트 하는 것은 매우 간단합니다. 아래의 규칙을 따라주세요.
   + 기존에 존재하는 필드 번호를 변경하지 마세요
   + 만약 당신이 새 필드를 추가해도 기존의 메세지들도 여전히 새로 생성된 코드를 통해서 파싱 후 직렬화 될 것입니다. 새로 추가된 코드와 기존 코드가 잘 상호작용 할 수 있도록 [[https://developers.google.com/protocol-buffers/docs/proto3#default][기본값]]에 신경쓰세요. 새 코드로 작성된 메세지는 여전히 기존 코드를 파싱할 수 있습니다. 기존 코드는 새로 추가된 필드를 무시합니다. 자세한 내용은 [[https://developers.google.com/protocol-buffers/docs/proto3#unknowns][알 수 없는 필드]] 섹션을 참고하세요.
   + 메세지 필드는 제거할 수 있지만, 해당 필드 번호가 추 후 재사용 되지 않도록 주의해야 합니다.
   + 필드 이름을 바꾸고 싶다면 필드 이름 앞에 "OBSOLETE_" 접두사를 추가하거나 필드 번호를 [[https://developers.google.com/protocol-buffers/docs/proto3#reserved][예약]]하여 향 후 ~.proto~ 사용자가 실수로라도 번호를 재사용 할 수 없도록 할 수 있습니다.
   + ~int32~, ~uint32~, ~int64~, ~uint64~, ~bool~ 타입은 모두 호환 가능합니다. 이 말은 이러한 필드 타입을 변형 없이 다른 필드 타입으로 변경할 수 있다는 뜻입니다^{(역자 주: int32 -> uint32 또는 int64 -> uint64 변경 가능)}. 만약 규격을 벗어난 숫자를 타입 변경하면 C++ 언어에서 해당 타입으로 숫자를 캐스팅 한 것과 같은 효과를 얻을 수 있습니다(예: 64비트 숫자를 int32로 읽은 경우, 32bit로 잘립니다).
   + ~sint32~, ~sint64~ 필드는 서로 호환되지만 다른 integer 타입과는 호환되지 않습니다.
   + ~string~, ~bytes~ 타입의 경우 bytes 타입이 유효한 UTF-8 값이면 서로 호환됩니다.
   + 메세지 필드 타입의 경우 bytes에 인코딩 된 메세지의 버전이 포함되어 있는 경우 bytes와 호환됩니다.
   + ~fixed32~ 타입은 ~sfixed32~ 타입과 호환됩니다. 그리고 ~fixed64~ 타입은 ~sfixed64~ 필드와 호환됩니다.
   + ~enum~ 타입은 ~int32~, ~uint32~, ~int64~, ~uint64~ 필드와 유효한 값 범위 내에서 호환됩니다(값이 만약 범위를 벗어날 경우 숫자가 잘립니다). 그러나 메세지가 역직렬화 될 때 클라이언트 코드가 코드를 다르게 취급할 수 있습니다. 예를 들어, 인식할 수 없는 proto3 버전의 ~enum~ 타입은 메세지에 보존되지만 메세지가 역질렬화 할 때 이 값이 어떻게 표현되는지는 언어에 따라 다릅니다. Int 필드는 항상 값을 유지해야 합니다.
   + 단일 멤버를 *새* ~멤버~ 로 변경하는 것은 안전하고 바이너리 호환이 가능합니다. 두 개 이상의 ~멤버~ 가 코드에 한번에 셋팅되지 않은 경우엔 여러 개의 기존 필드를 새 필드로 안전하게 옮길 수 있습니다. 특정 멤버를 를 기존 ~멤버~ 로 옮기는 것은 안전하지 않습니다.
** Unknown Fields
   알 수 없는 필드는 올바른 규격의 직렬화 된 프로토콜 버퍼를 파싱할 때 인식하지 못하는 필드를 의미합니다. 예를 들어, 구버전의 바이너리 데이터를 새로 추가된 필드와 함께 새 바이너리 파일로 전송할 때, 새로 추가된 필드는 구버전의 바이너리 데이터에선 인식하지 못합니다.

   원래 proto3 메세지는 파싱할 때 알 수 없는 필드를 항상 폐기하도록 설계되었지만 버전 3.5부터는 proto2 동작과 일치하도록 알 수 없는 필드를 계속 보존하도록 변경되었습니다. 3.5 버전 이상에서는 파싱과 직렬화 출력 시 알 수 없는 필드가 계속 보존됩니다.
** Any
   ~Any~ 메세지 타입을 사용하면 특정 메세지 타입을 .proto 파일에서 알지 못해도 그 메세지 타입을 필드로 사용할 수 있습니다. 메세지를 직렬화 할 때 ~Any~ 타입은 임의의 ~bytes~ 타입으로 변환된 후 해당 데이터를 가르키는 URL을 전역 고유 식별자로 갖습니다. ~Any~ 타입을 사용할 때는 ~google/protobuf/any.proto~ 를 [[https://developers.google.com/protocol-buffers/docs/proto3#other][import]]하여야 합니다.
	#+BEGIN_SRC proto
    import "google/protobuf/any.proto";

    message ErrorStatus {
      string message = 1;
      repeated google.protobuf.Any details = 2;
    }
	#+END_SRC
	기본적으로 메세지 타입 URL은 ~type.googleapis.com/packagename.messagename~ 형식을 갖습니다.

	다른 언어에서 ~Any~ 타입 구현은 런타임 라이브러리를 지원하며 typesafe한 압축 및 압축 해제를 지원합니다. 예를 들어, 자바에서는 ~Any~ 타입에 ~pack()~, ~unpack()~ 메서드가 있고, C++에서는 ~PackFrom()~, ~UnpackTo()~ 메서드가 있습니다.
	#+BEGIN_SRC proto
    // Storing an arbitrary message type in Any.
    NetworkErrorDetails details = ...;
    ErrorStatus status;
    status.add_details()->PackFrom(details);
    
    // Reading an arbitrary message from Any.
    ErrorStatus status = ...;
    for (const Any& detail : status.details()) {
      if (detail.Is<NetworkErrorDetails>()) {
        NetworkErrorDetails network_error;
        detail.UnpackTo(&network_error);
        ... processing network_error ...
      }
    }
	#+END_SRC
	*현재 모든 언어에서 Any 타입을 지원하기 위한 라이브러리가 개발중입니다.*

	만약 [[https://developers.google.com/protocol-buffers/docs/proto][proto2 문법]]에 이미 익숙하시다면, ~Any~ 타입 대신 [[https://developers.google.com/protocol-buffers/docs/proto#extensions][extenstions]]을 사용하실 수도 있습니다.
** Oneof
   많은 필드를 가진 메시지가 있고, 동시에 최대 하나의 필드가 설정되는 경우, oneof 기능을 통해 해당 동작을 시행하여 메모리를 절약할 수 있습니다.

   Oneof 필드는 일반 필드와 유사하지만 oneof 필드 공유 메모리의 필드와는 다르고^{(역자 주: 번역이 이상한거 같군요..ㅠㅠ)}, 동시에 최대 하나의 필드를 설정할 수 있습니다. 멤버 중 하나를 설정하면 다른 모든 멤버가 자동으로 oneof 필드에 채워집니다. ~oneof~ 필드가 존재한다면 선택한 언어에 따라 특별한 ~case()~ 또는 ~WhichOneof()~ 메서드를 사용하여 oneof 값이 설정되어 있는지 확인할 수 있습니다.
*** Using Oneof
	~.proto~ 파일에 ~oneof~ 를 정의하려면 ~oneof~ 키워드와 ~oneof~ 이름을 지정해야 합니다(아래의 예제에선 ~test_oneof~).
	#+BEGIN_SRC proto
    message SampleMessage {
      oneof test_oneof {
        string name = 4;
        SubMessage sub_message = 9;
      }
    }
	#+END_SRC
	이 후 ~oneof~ 필드를 ~oneof~ 정의에 추가하세요. 당신은 어떤 타입이든 추가할 수 있지만 ~repeated~ 필드는 추가할 수 없습니다.
	
	생성된 코드에서, oneof 필드에는 일반 필드에 대하여 같은 getters 와 setters 를 가지고 있습니다. 그리고 어떤 값이 oneof 설정이 이루어졌는지 확인할 수 있는 특별한 메서드를 또한 제공합니다(oneof 필드가 존재할 경우). 언어별로 oneof API에 대한 더 자세한 정보를 살펴보시려면 [[https://developers.google.com/protocol-buffers/docs/reference/overview][API reference]]를 참고하세요.
*** Oneof Features
	+ oneof 필드를 셋팅하면 다른 모든 oneof 멤버가 초기화됩니다. 그러므로 만약 여러개의 oneof 필드를 셋팅하고 싶어도 마지막으로 설정한 필드값만 유효합니다.
	  #+BEGIN_SRC proto
      SampleMessage message;
      message.set_name("name");
      CHECK(message.has_message.mutable_sub_message();   // Will clear name field.
      CHECK(!message.has_name());
	  #+END_SRC
	+ 파서가 여러 oneof 멤버를 발견하면, 파서는 마지막으로 파싱 된 oneof 멤버만 사용하여 메세지를 파싱합니다.
	+ oneof는 ~repeated~ 구문을 사용할 수 없습니다.
	  + Reflection APIs도 oneof 필드에 사용할 수 있습니다.
	  + oneof 필드를 기본값으로 설정하면(예: int32를 oneof 필드로 설정할 경우 0 셋팅) oneof 필드의 "case"가 설정되며, 이 값은 직렬화 될 때 사용됩니다.
	  + 만약 C++를 사용한다면, 코드가 메모리 충돌을 일으키지 않도록 주의하세요. 아래의 예제 코드에선 ~sub_message~ 가 ~set_name()~ 메서드 호출됨으로 인해 삭제되어 충돌을 일으킵니다.
  	    #+BEGIN_SRC proto
        SampleMessage message;
        SubMessage* sub_message = message.mutable_sub_message();
        message.set_name("name");      // Will delete sub_message
        sub_message->set_...            // Crashes here
        #+END_SRC
	  + 다시 C++에서, ~Swap()~ 메서드로 oneof 필드가 있는 두 메세지를 교체하면, 각 메세지는 다른 하나의 경우로 끝나게 된다. 아래의 예제에서 ~msg1~ 은 ~sub_message~ 를 갖게 되고 ~msg2~ 는 ~name~ 을 갖게 된다.
  	    #+BEGIN_SRC proto
        SampleMessage msg1;
        msg1.set_name("name");
        SampleMessage msg2;
        msg2.mutable_sub_message();
        msg1.swap(&msg2);
        CHECK(msg1.has_sub_message());
        CHECK(msg2.has_name());
        #+END_SRC
*** Backwards-compatibility issues
	oneof 필드늘 추가하거나 삭제할 땐 주의하여야 합니다. oneof 필드가 ~None~/~NOT_SET~ 을 반환하면 oneof가 설정되지 않았거나 다른 버전에서 oneof 필드가 설정되었음을 의미합니다. 실제 동작 중에는 알 수 없는 필드가 oneof 필드 멤버 중 하나인지 알 수 있는 방법이 없습니다.
**** Tag Reuse Issues
	 + *oneof 요소로 이동 또는 제외할 경우*: 메세지가 직렬화, 파싱될 경우 일부 정보가 손실될 수 있습니다(일부 필드는 지워짐). 그러나 단일 필드를 새 필드로 안전하게 이동할 수 있으며 oneof 요소로 하나만 설정되어 있으면 여러 필드를 이동할 수 있습니다.
	 + *oneof 필드 요소로 추가하거나 제거할 경우*: 메세지가 직렬화, 파싱된 후에는 현재 셋팅된 oneof 필드가 지워질 수 있습니다.
	 + *onfof 분할 또는 병합할 경우*: 이 이슈는 일반 필드 이동 이슈와 유사합니다.
** Maps
   데이터 정의 시 Map 형식을 사용하고 싶을 경우, 프로토콜 버퍼에선 이를 위해 유용한 구문을 제공합니다.
   #+BEGIN_SRC proto
   map<key_type, value_type> map_field = N;
   #+END_SRC
   ~key_type~ 위치에는 interal 또는 string 타입이 올 수 있습니다(또는 부동소수점, ~bytes~ 타입을 제외한 아무  [[https://developers.google.com/protocol-buffers/docs/proto3#scalar][스칼라]] 타입). enum 타입은 ~key_type~ 위치에 사용할 수 없습니다. ~value_type~은 다른 map 타입을 제외한 모든 타입을 사용할 수 있습니다.

   예를 들어, string 타입의 키를 사용하고  ~Proejct~ 메세지 타입을 값으로 갖는 map을 생성하고 싶을 경우엔 아래와 같이 정의할 수 있습니다.
   #+BEGIN_SRC proto
   map<string, Project> projects = 3;
   #+END_SRC
   + map 필드엔 ~repeated~ 구문을 사용할 수 없습니다.
   + 포맷 순서, map의 값에는 순서가 정의되어 있지 않습니다. 따라서 map 내부의 키, 값의 순서는 신뢰할 수 없습니다.
   + ~.proto~ 파일 내부 텍스트를 정리(formatting)할 때, map은 key값 순서대로 정렬됩니다. 숫자타입의 key는 오름차순으로 정렬됩니다.
   + 데이터를 파싱하거나 병합(머지)할 때, map의 key가 중복될 경우 마지막으로 선언된 key 값이 사용됩니다. ~.proto~ 파일에서 파싱할 때 중복이 있을 경우엔 파싱이 실패합니다.
   + 만약 키만 있고 값이 없는 맵이 있을 경우, 각 언어에 의존하여 처리됩니다. C++, Java, Python 같은 경우엔 기본값으로 직렬화 되고, 그 외의 언어의 경우엔 아무것도 직렬화 되지 않습니다.
   map 생성 API는 현재 proto3를 지원하는 모든 언어에서 사용 가능합니다. map API에 대해 각 언어별로 자세히 알아보고 싶으면 [[https://developers.google.com/protocol-buffers/docs/reference/overview][API reference]]를 참고하세요.
*** Backwards compatibility
	map 문법(syntax)는 실제론 다음과 같이 처리됩니다. 따라서 map을 지원하지 않는 언어를 사용할 경우에도 여전히 map 데이터를 처리할 수 있습니다.
    #+BEGIN_SRC proto
    message MapFieldEntry {
      key_type key = 1;
      value_type value = 2;
    }

    repeated MapFieldEntry map_field = N;
    #+END_SRC
	map을 지원하는 프로토콜 버퍼 구현은 위 예제와 같은 형식으로도 사용할 수 있도록 데이터를 작성하여야 합니다.
** Packages
   메세지에서 이름이 중복되어 충돌되는 상황을 막기 위해서 ~.proto~ 파일에 ~package~ 구문을 사용할 수 있습니다.
   #+BEGIN_SRC proto
   package foo.bar;
   message Open { ... }
   #+END_SRC
   위와 같이 정의한 후 다른 메세지에서 ~Open~ 메세지를 필드 타입으로 사용할 때 다음과 같이 패키지명을 사용할 수 있습니다.
   #+BEGIN_SRC proto
   message Foo {
     ...
     foo.bar.Open open = 1;
     ...
   }
   #+END_SRC
   패키지 구문이 생성된 코드에 영향을 주는 방식은 선택한 언어마다 다릅니다.
   + *C++* 의 경우에는 작명 규칙에 따라 클래스들이 생성됩니다. 예를들어, ~Open~ 은 ~foo::bar~ 안에 있을 것입니다.
   + *Java* 의 경우에는 ~.proto~ 파일에 ~open java_package~ 라고 명시하지 않는 한 패키지를 자바 패키지로 사용합니다.
   + *Python* 의 경우에는 파일 시스템의 위치에 따라 Python 모듈이 구성되므로 패키지 구문을 무시합니다.
   + *Go* 의 경우에는 ~.proto~ 파일에 ~option go_package~ 라고 명시하지 않는 한 패키지를 Go 패키지 이름으로 사용합니다.
   + *Ruby* 의 경우에는 Ruby 작명 규칙에 따라 클래스가 생성되며 Ruby 특유의 대문자 표기 스타일로 변경됩니다(첫번째 문자는 대문자로 변경되며, 알파벳이 아닐 경우엔 앞에 ~PB_~ 문자가 붙습니다). 예를 들어, ~Open~ 은 ~Foo::Bar~ 안에 있을 것입니다.
   + *C#* 의 경우에는 ~.proto~ 파일에 ~option csharp_namespace~ 라고 명시하지 않는 한  파스칼 형식으로 변환된 후 작명 규칙에 따라 코드를 생성합니다. 예를 들어, ~Open~ 은 ~Foo::Bar~ 안에 있을 것입니다.
*** Packages and Name Resolution
	프로토콜 버퍼 언어에서 타입 이름 확인은 C++과 비슷하게 동작합니다. 먼저 가장 안쪽의 범위를 탐색하고 그 다음 가장 안쪽부터 두번째, 세번째.. 이런식으로 탐색하여 각 패키지를 상위 패키지의 "내부" 패키지로 간주합니다. 가장 앞쪽의 '.' (예: ~.foo.bar.Baz~)는 최상위 범위에서 시작하는 것을 의미합니다.

	프로토콜 버퍼 컴파일러는 모든 ~.proto~ 파일을 가져와서 파싱한 후 모든 유형 이름을 분석합니다. 각 언어별 코드 생성기는 범위 지정 규칙이 다르더라도 해당 언어가 각 유형을 참조하는 방법을 알고 있습니다.
** Defining Services
   메세지를 RPC(Remote Procedure Call) 시스템과 함께 쓰고 싶다면, ~.proto~ 파일에 RPC 서비스 인터페이스를 정의하고 이를 프로토콜 버퍼 컴파일러를 이용해서 원하는 언어의 서비스 인터페이스 코드를 생성할 수 있습니다. 예를 들어, ~SearchRequest~ 를 입력받아 ~SearchResponse~ 를 반환하는 메서드를 가진 RPC 서비스를 생성하고 싶다면 ~.proto~ 파일에 다음과 같이 정의할 수 있습니다.
   #+BEGIN_SRC proto
   service SearchService {
     rpc Search (SearchRequest) returns (SearchResponse);
   }
   #+END_SRC
   프로토콜 버퍼를 가장 쉽게 사용할 수 있는 RPC 시스템은 [[https://grpc.io/][gRPC]]입니다. gPRC는 언어, 플랫폼 중립적이고, 오픈소스이며 구글에 의해 개발된 RPC 시스템입니다. gRPC는 특히 프로토콜 버퍼와 잘 작동하며, 특수한 프로토콜 버퍼 컴파일러 플러그인을 사용하여 ~.proto~ 파일로부터 직접 RPC 코드를 생성할 수 있습니다.

   만약 gRPC를 사용하고 싶지 않다면, 다른 RPC 시스템에서도 프로토콜 버퍼를 사용할 수 있습니다. [[https://developers.google.com/protocol-buffers/docs/proto#services][Proto2 Language Guide]]를 참고하세요.

   프로토콜 버퍼에 대한 RPC 구현을 위한 많은 서드파티 프로젝트들이 있습니다. [[For a list of links to projects we know about, see the third-party add-ons wiki page. ][서드파티 애드온 위키]]를 참고하세요.
** JSON Mapping
   Proto3는 표준 JSON 인코딩을 지원하며 이는 시스템끼리 쉽게 데이터를 공유할 수 있게 해줍니다. 인코딩은 유형별로 아래 표에 설명되어 있습니다.

   만약 JSON 인코딩 된 값이 없거나 ~null~ 일 경우, 프로토콜 버퍼로 파싱될 때 적절한 [[https://developers.google.com/protocol-buffers/docs/proto3#default][기본값]]으로 간주됩니다. 프로토콜 버퍼에서 필드가 기본값을 가질 경우, JSON 인코딩 데이터에선 공간 확보를 위해 생략합니다. 구현 시 JSON 인코딩 출력에서 기본값으로 필드를 생성하는 옵션을 제공할 수 있습니다.

| proto3                 | JSON          | Json example                             | Notes                                                                                                                                                                                                                                                                             |
|------------------------+---------------+------------------------------------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| message                | object        | {"fooBar": v, "g": null, ...}            | JSON 객체를 생성합니다. 메세지 필드 이름은 lowerCamelCase 형식이 되며 JSON 객체의 키로 사용됩니다. 만약 *json_name* 필드가 명시되어 있다면, 명시된 값은 key 대신 사용됩니다. 파서는 lowerCamelCase 이름과 원본의 프로토 필드 이름 둘 다(또는 하나의 *json_name* 옵션) 허용합니다. |
| enum                   | string        | "FOO_BAR"                                | enum 이름은 값으로도 사용됩니다. 파서는 enum의 이름 또는 정수값을 모두 허용합니다.                                                                                                                                                                                                |
| map<K,v>               | object        | {"k", v, ...}                            | 모든 키값은 string으로 변환됩니다.                                                                                                                                                                                                                                                |
| repeated V             | array         | [v, ...]                                 | *null* 은 빈 목록 [] 대신 사용할 수 있습니다.                                                                                                                                                                                                                                     |
| bool                   | true, false   | true, fasle                              |                                                                                                                                                                                                                                                                                   |
| string                 | string        | "Hello, World"                           |                                                                                                                                                                                                                                                                                   |
| bytes                  | base64 string | "YWJjMTIzIT8kKiYoKSctPUB+"	           | 패딩값과 함께 bas64 인코딩 방식을 사용하여 string 타입으로 데이터를 인코딩합니다.                                                                                                                                                                                                 |
| int32, fixed32, uint32 | number        | 1, -10, 0                                | 10진수 숫자입니다. 숫자 또는 문자열도 허용됩니다.                                                                                                                                                                                                                                 |
| int64, fixed64, uint64 | string        | "1", "-10"                               | 10진수 문자열입니다. 숫자 또는 문자열도 허용됩니다.                                                                                                                                                                                                                               |
| float, double          | number        | 1.1, -10.0, 0, "NaN", "Infinity"         | 숫자 또는 "NaN", "Infinity", "-Infinity" 중 하나입니다. 숫자, 문자열, 또는 지수 표기법도 허용됩니다.                                                                                                                                                                              |
| Any                    | object        | {"@type": "url", "f": v, ...}}           | Any 타입에 특별한 형식의 값이 매핑되었다면 다음과 같은 모양으로 변환될 것입니다. *{"@type": xxx, "value": yyy}* . 그렇지 않으면 값이 JSON 객체로 변환되고 "@type" 필드가 추가되어 실제 데이터 유형을 나타냅니다.                                                                  |
| Timestamp              | string        | "1972-01-01T10:00:20.021Z"               | RFC 3999 형식을 따릅니다. 생성된 값은 항상 z-normalized를 사용하며 0, 3, 6 또는 9자리의 소수가 생성됩니다. "Z" 이외의 오프셋도 허용됩니다.                                                                                                                                        |
| Duration               | string        | "1.000340012s", "1s"                     | 필요한 정밀도에 따라 0, 3, 6 또는 9자리의 소수가 생성되며 접미어 "s"가 붙습니다. nano-seconds 자릿수에 맞고 접미어 "s"가 필요한 한 모든 부분 자릿수가 허용됩니다.                                                                                                                 |
| Struct                 | object        | { ...}                                   | 어떤 JSON 객체든 될 수 있습니다. *struct.proto* 를 살펴보세요.                                                                                                                                                                                                                    |
| Wrapper types          | various types | 2, "2", "foo", true, "true", null, 0, … | 랩퍼는 기본 유형과 JSON에서 동일한 표현을 사용하지만, 데이터 변환 및 전송 시에도 null이 허용되고 유지됩니다.                                                                                                                                                                      |
| FieldMask              | string        | "f.fooBar,h"                             | *field_mask.proto* 를 살펴보세요.                                                                                                                                                                                                                                                 |
| ListValue              | array         | [foo, bar, ...]                          |                                                                                                                                                                                                                                                                                   |
| Value                  | value         |                                          | 모든 JSON 값                                                                                                                                                                                                                                                                      |
| NullValue              | null          |                                          | JSON null                                                                                                                                                                                                                                                                         |
| Empty                  | object        | {}                                       | 비어있는 JSON 객체                                                                                                                                                                                                                                                                            |

*** JSON options
	proto3의 JSON에선 아래의 옵션들을 제공할 수 있습니다..
	+ *기본값을 가진 필드*: proto3의 JSON 출력값에선 기본값이 있는 필드는 기본적으로 생략됩니다. 실제 구현 및 동작 시 기본값 출력을 기본값으로 재정의하는 옵션을 제공할 수 있습니다.
	+ *알 수 없는 필드 제외*: Proto3의 JSON 파서는 기본적으로 파싱 시 알 수 없는 필드가 있을 시 거부되지만, 알 수 없는 필드를 그냥 무시하는 옵션을 제공할 수 있습니다.
	+ *lowerCamelCase 이름 대신 proto 필드 이름 사용*: proto3의 JSON에선 기본적으로 필드 이름을 lowerCamelCase로 변환해서 사용합니다. 실제 구현시에는 proto 필드 이름을 Json 이름으로 대신 사용할 수 있는 옵션을 제공할 수 있습니다. 변환된 lowerCamelCase 이름과 proto 필드 이름을 모두 사용하려면 Proto3 JSON 파서가 필요합니다.
	+ *enum 값을 string 대신 숫자로 출력*: JSON 출력 시 enum은 기본적으로 값의 이름이 출력됩니다. 이 대신 enum 값의 숫자 값을 사용하는 옵션을 제공할 수 있습니다.

** Options
   ~.proto~ 파일에는 각 파일마다 주석으로 여러가지 옵션을 추가할 수 있습니다. 옵션은 파일의 정의 자체를 변경하진 않지만 특정 상황에서 처리되는 방식에 영향을 줄 수 있습니다. 사용 가능한 옵션 전체 목록은 ~google/protobuf/descriptor.proto~ 에 있습니다.

   일부 옵션을 파일 레벨의 옵션으로, 메세지, num, 서비스 레벨이 아닌 최상위 영역에 작성해야 합니다. 일부 옵션은 메세지 레벨의 옵션으로서, 반드시 메세지 정의 안에 작성해야 합니다. 일부 옵션은 필드 레벨 옵션으로서, 반드시 필드 정의 안에 작성해야 합니다. enum 타입, enum 값, 서비스 타입, 서비스 메서드에 대한 옵션을 작성할 수도 있습니다. 그러나 현재는 유용한 옵션이 없습니다.


   아래는 일반적으로 쓰이는 일부 옵션들입니다.
   + ~java_package~ (file option): 생성되는 자바 클래스 파일들을 위한 패키지를 선언합니다. ~java_package~ 옵션을 ~.proto~ 파일에 작성하지 않은 경우에는 기본적으로 proto 패키지(~.proto~ 파일에 "package" 키워드를 사용하여 선언한 값)가 사용됩니다. 그러나 프로토 패키지는 리버스 도메인으로 시작하지 않기 때문에 좋은 Java 패키지를 만들지 않습니다. 만약 자바 코드를 생성하지 않는다면 이 옵션은 적용되지 않습니다..
   #+BEGIN_SRC proto

   option java_package = "com.example.foo";

   #+END_SRC
   + ~java_multiple_files~ (file option):  최상위 레벨 메세지, enum, 서비스가 ~.proto~ 파일의 이름을 딴 외부 클래스가 아닌 패키지 수준에서 정의되도록 합니다.
   #+BEGIN_SRC proto

   option java_multiple_files = true;

   #+END_SRC
   + ~java_outer_classname~ (file_option): 클래스 이름(및 파일 이름)을 가장 최상위에 있는 자바 클래스 이름으로 생성합니다. ~.proto~ 파일에 ~java_outer_classname~ 이 선언되지 않은 경우엔 ~.proto~ 파일 이름을 camel-case로 변환하여 클래스 이름으로 사용합니다(예를 들어 foo_bar.proto는 FooBar.java가 됩니다). Java 코드를 생성하지 않는다면 이 옵션은 적용되지 않습니다.
   #+BEGIN_SRC proto

   option java_outer_classname = "Ponycopter";

   #+END_SRC
   + ~optimize_for~ (file_option): ~SPEED~, ~CODE_SIZE~, ~LITE_RUNTIME~ 중 하나를 사용할 수 있습니다. 이는 아래와 같은 방식으로 C++과 Java 코드(및 서드파티 코드 생성) 생성에 영향을 미칩니다.
     - ~SPEED~ (default): 프로토버퍼 컴파일러는 직렬화, 파싱, 그리고 작업을 수행하기 위한 코드를 생성합니다. 이 코드는 고도로 최적화되어 있습니다.
     - ~CODE_SIZE~: 프로토콜 버퍼 컴파일러는 최소한의 클래스를 생성하며 직렬화, 파싱 및 기타 다양한 작업을 구현하기 위해 공유 리플렉션 기반 코드를 사용합니다. 생성된 코드는 ~SPEED~ 옵션을 사용해 생성한 코드보다는 크기가 훨씬 작지만 동작 속도는 느려집니다. 클래스들은 ~SPEED~ 옵션을 사용해 구현한 코드와 동일한 public API를 구현합니다. 이 모드는 매우 많은 수의 .proto 파일이 포함된 앱에서 가장 유용합니다. 모든 파일이 맹목적으로 빠를 필요는 없으니깐요.
     - ~LITE_RUNTIME~: 프로토콜 버퍼 컴파일러가 "lite" 런타임 라이브러리에만 의존하는 클래스를 생성합니다(~libprotobuf~ 대신 ~libprotobuf-lite~ 에 의존). lite 런타임 라이브러리는 전체 라이브러리보다 훨씬 작지만(약 10배정도) descriptor와 reflection과 같은 특정 기능을 생략합니다. 이 기능은 모바일 기기같은 제한된 플랫폼에서 실행되는 앱에 특히 유용합니다. 컴파일러는 ~SPEED~ 모드에서와 같이 여전히 빠른 메서드를 구현합니다. 생성된 클래스는 각 언어로 MessageLite 인터페이스를 구현하며, 전체 Message 인터페이스 메소드 중 일부만 제공합니다.
   #+BEGIN_SRC proto

   option optimize_for = CODE_SIZE;

   #+END_SRC
   + ~cc_enable_arenas~ (file_option): C++ 코드를 생성할 때 [[https://developers.google.com/protocol-buffers/docs/reference/arenas][arena allocation]]을 활성화합니다.
   + ~objc_class_prefix~ (file_option): ~.proto~ 파일에서 생성된 Objective-C 클래스와 enum에 접두어를 붙입니다. 이건 기본값이 아닙니다. [[https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Conventions/Conventions.html#//apple_ref/doc/uid/TP40011210-CH10-SW4][Apple에서 권장]]하는대로 3-5자의 대문자로 된 접두사를 사용하는게 좋습니다. 참고로  Apple은 두 글자의 접두사를 모두 선점했습니다.
   + ~deprecated~ (field option): ~true~ 로 설정할 경우, 이 필드는 향 후 새로 생성되는 코드에선 더이상 사용하지 않음을 표시합니다. 대부분의 언어에서 이것은 실제로 효과가 없습니다. Java의 경우 ~@Deprecated~ 어노테이션이 생성됩니다. 향 후 각 언어별 코드 생성기는 필드 접근자에 대해 지원 중단을 경고하는 주석을 생성할 수 있으며, 이로 인해 코드를 컴파일 할 때 경고가 발생합니다. 다른 사람이 필드를 사용하지 않고 새 사용자가 필드를 사용하지 못하게 하려면 필드 선언을 [[https://developers.google.com/protocol-buffers/docs/proto3#reserved][예약 구문]]으로 바꾸세요.
   #+BEGIN_SRC proto

   int32 old_field = 6 [deprecated=true];
   #+END_SRC

*** Custom Options
	프로토콜 버퍼는 고유한 옵션을 정의하고 사용할 수 있습니다. 이것은 대부분의 사람들은 필요하지 않는 고급 기능입니다. 자신만의 옵션을 만들 필요가 있다면 [[https://developers.google.com/protocol-buffers/docs/proto#customoptions][Proto2 언어 가이드]]를 참고하세요. 참고로 사용자 정의 옵션을 만들 때 [[https://developers.google.com/protocol-buffers/docs/proto#extensions][Extensions]]을 사용하며, 오직 proto3 의 사용자 정의 옵션에서만 허용됩니다.

** Generating Your Classes
   ~.proto~ 파일에 정의된 메세지로 Java, Python, C++, Go, Ruby, Objective-C, C# 코드를 생성하기 위해선 ~protoc~ 컴파일러를 이용해 ~.proto~ 파일을 컴파일 하여야 합니다. 컴파일러가 설치되어 있지 않다면 [[https://developers.google.com/protocol-buffers/docs/downloads][패키지를 다운]]받아 README 파일을 보고 따라하세요. Go의 경우엔 컴파일러를 위한 특별한 코드 생성 플러그인이 필요합니다. 설치 방법은 Github의 [[https://github.com/golang/protobuf/][golang/protobuf]] 저장소에서 찾을 수 있습니다.

   프로토콜 컴파일러는 다음과 같이 호출합니다.
   #+BEGIN_SRC proto

   protoc --proto_path=IMPORT_PATH --cpp_out=DST_DIR --java_out=DST_DIR --python_out=DST_DIR --go_out=DST_DIR --ruby_out=DST_DIR --objc_out=DST_DIR --csharp_out=DST_DIR path/to/file.proto

   #+END_SRC
   + ~IMPORT_PATH~ 는 ~import~ 구문을 수행하기 위해 ~.proto~ 파일이 위치한 디렉토리 위치를 지정합니다. 만약 입력하지 않았다면 기본적으로 현재 디렉토리가 사용됩니다. 여러 개의 폴더를 지정해야 할 경우엔 ~--proto_path~ 옵션을 여러번 입력하여 지정할 수 있습니다. 순차적으로 검색될 것입니다. ~--proto_path~ 명령어를 짧게 사용하고 싶다면 ~-I=/IMPORT_PATH/~ 명령어를 사용할 수 있습니다.
   + 하나 이상의 코드 생성 지시문을 제공할 수 있습니다.
     - ~--cpp_out~ 명령어는 C++ 코드를 ~DST_DIR~ 안에 생성합니다. [[https://developers.google.com/protocol-buffers/docs/reference/cpp-generated][C++ 코드 생성 레퍼런스]]를 참고하세요.
     - ~--java_out~ 명령어는 Java 코드를 ~DST_DIR~ 안에 생성합니다. [[https://developers.google.com/protocol-buffers/docs/reference/java-generated][Java 코드 생성 레퍼런스]]를 참고하세요.
     - ~--python_out~ 명령어는 Python 코드를 ~DST_DIR~ 안에 생성합니다. [[https://developers.google.com/protocol-buffers/docs/reference/python-generated][Python 코드 생성 레퍼런스]]를 참고하세요.
     - ~--go_out~ 명령어는 Go 코드를 ~DST_DIR~ 안에 생성합니다. [[https://developers.google.com/protocol-buffers/docs/reference/go-generated][Go 코드 생성 레퍼런스]]를 참고하세요.
     - ~--ruby_out~ 명령어는 Ruby 코드를 ~DST_DIR~ 안에 생성합니다. 루비 코드 생성 레퍼런스는 곧 공개됩니다!
     - ~--objc_out~ 명령어는 Objective-C 코드를 ~DST_DIR~ 안에 생성합니다. [[https://developers.google.com/protocol-buffers/docs/reference/objective-c-generated][Objective-C 코드 생성 레퍼런스]]를 참고하세요.
     - ~--csharp_out~ 명령어는 C# 코드를 ~DST_DIR~ 안에 생성합니다. [[https://developers.google.com/protocol-buffers/docs/reference/csharp-generated][C# 코드 생성 레퍼런스]]를 참고하세요.
     - ~--php_out~ 명령어는 PHP 코드를 ~DST_DIR~ 안에 생성합니다. [[https://developers.google.com/protocol-buffers/docs/reference/php-generated][PHP 코드 생성 레퍼런스]]를 참고하세요.
     편의상 ~DST_DIR~ 이 ~.zip~, ~.jar~ 로 끝날 경우 컴파일러는 지정된 이름으로 된 단일 zip 형식의 압축 파일에 코드를 생성합니다. ~.jar~ 파일 생성 시에도 JAVA JAR 파일에 필요한 매니페스트 파일이 제공됩니다. 참고로 압축파일이 이미 존재할 경우엔 덮어씌워집니다. 압축 파일에 파일을 추가할 정도로 컴파일러가 똑똑하진 못해요.
   + 하나 이상의 ~.proto~ 파일들을 입력할 수 있습니다. 한번에 여러 ~.proto~ 파일들을 지정할 수 있습니다. 파일 이름은 현재 디렉토리와 연관이 있지만 컴파일러가 표준 이름을 결정할 수 있도록 각 파일은 ~IMPORT_PATH~ 중 하나에 있어야 합니다.



