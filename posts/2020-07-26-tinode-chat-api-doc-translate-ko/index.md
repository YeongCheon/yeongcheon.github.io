# tinode series.003 - API document


매번 [[https://github.com/tinode/chat/blob/master/docs/API.md][영어문서]] 보기 빡쳐서 직접 번역기 돌려가며 쓰는 번역본. +번역기와 의역, 오역 범벅입니다.+ 가급적이면 [[https://github.com/tinode/chat/blob/master/docs/API.md][공식 문서]]를 보세요.

* Server API
** How it works?
   Tinode는 IM 라우터이자 스토어입니다. 발행-구독 모델 컨셉을 대략적으로 따릅니다.

   서버는 세션, 사용자, 그리고 토픽을 연결합니다. 세션은 클라이언트 프로그램과 서버 간의 네트워크 연결을 의미합니다. 사용자는 세션을 서버에 연결하는 사람을 의미합니다. 토픽은 세션들끼리 콘텐츠를 주고받는 통신 채널입니다.

   사용자가 토픽은 각각 고유한 ID가 할당됩니다. 사용자 ID는 'usr' 문자로 시작하는 base64-URL 인코딩 6비트 랜덤 문자로 이루어져 있습니다(예: usr2il9suCbuko). 토픽 ID는 아래에서 설명합니다.

   모바일 또는 웹 어플리케이션 클라이언트는 웹소켓 또는 long pulling 방식으로 서버에 연결하여 세션을 생성합니다. 대부분 작업을 수행하기 위해선 클라이언트 인증이 필요합니다. 클라이언트는 {login} 패킷을 전송하여 세션을 인증합니다. 좀 더 자세한 내용은 [[#Authentication][인증 섹션]]을 참고하세요. 인증되면 클라이언트는 나중에 인증에 사용할 토큰을 발급받습니다. 동일한 사용자가 여러 세션을 동시에 설정할 수 있습니다. 로그아웃은 지원하지 않습니다. (필요하지도 않고요).

   세션이 설정되면 사용자는 토픽을 통해 다른 사용자와 통신을 시작할 수 있습니다. 다음과 같은 토픽들을 사용할 수 있습니다.
  
   + ~me~ 는 자기 자신의 프로필을 관리하고 다른 토픽에 대한 알람을 받을 수 있는 토픽입니다. ~me~ 토픽은 모든 사용자가 가지고 있습니다.
   + ~fnd~ 토픽은 다른 사용자나 토픽을 찾을 때 사용합니다. ~fnd~ 토픽 또한 모든 사용자가 가지고 있습니다.
   + Peer to Peer 토픽은 두 사용자 간 1:1 통신을 위한 토픽입니다. 각 사용자는 토픽 이름을 상대방의 ID로 인지합니다. 즉, 'usr' 문자로 시작하는 base64-URL 인코딩 6비트 랜덤 문자(예: usr2il9suCbuko)로 인지합니다.
   + 그룹 토픽은 여러 사용자 간 통신을 위한 토픽입니다. 이 토픽의 이름은 'grp'로 시작하며 11자리의 pseudo 무작위 숫자로 이루어져 있습니다(예: ~grpYiqEXb4QY6s~). 그룹 토픽은 반드시 명시적으로 작성해야 합니다.

   세션은  ~{sub}~ 패킷을 전송하여 토픽에 참여합니다. ~{sub}~ 패킷은 세 가지 기능을 제공합니다. 새 토픽을 만들고, 사용자 토픽을 구독하고, 세션을 토픽에 연결하는 기능을 제공합니다. 더 자세한 내용은 아래 [[#sub][~{sub}~]] 섹션을 참고하세요.

   세션이 토픽에 연결되면, 사용자는 ~{pub}~ 패킷을 이용해 콘텐츠를 생성을 시작합니다. 콘텐츠는 다른 세션들에게 ~{data}~ 패킷 형태로 전송됩니다.

   사용자는 ~{get}~, ~{set}~ 패킷을 이용하여 쿼리를 요청하거나, 토픽의 메타데이터를 수정할 수 있습니다.

   토픽의 설명 변경 또는 다른 사용자의 토픽 참여, 탈퇴 같은 토픽 메타데이터 변경은 ~{pres}~ (presence) 패킷을 이용하여 실시간으로 세션에 전송합니다. ~{pres}~ 패킷은 연관있는 토픽 또는 ~me~ 토픽으로 전송됩니다.

  사용자의 ~me~ 토픽이 온라인 상태가 되면(예: 인증된 세션이 ~me~ 토픽과 연결될 때) ~{pres}~ 패킷이 온라인 된 사용자와 Peer to Perr 형태로 연결된 모든 사용자의 ~me~ 토픽에 전송됩니다.
** General Considerations
   타임스탬프는 항상 [[https://tools.ietf.org/html/rfc3339][RFC 3339]] 형식을 따르는 문자열 형태이며, 밀리 세컨드까지 표시되고 타임존은 항상 UTC를 기준으로 한다. 예: ~"2015-10-06T18:07:29.841Z"~

   앞으로 이 문서에서 base64 인코딩이 언급된다면, 이는 padding characters가 stripped된 base64 URL인코딩을 의미합니다. [[https://tools.ietf.org/html/rfc4648][RFC 4648]] 문서를 참고하세요.

   ~{data}~ 패킷은 서버에서 생성한 순차적 ID를 가집니다. ID는 1부터 시작하여 각 메세지마다 1씩 증가하는 10진수 형태의 숫자입니다. 각 ID는 토픽별로 고유한 값임을 보장합니다. 요청에 대한 응답을 연동하기 위해서 클라이언트는 서버에 할당된 모든 패캣에 메세지 ID를 할당할 수 있습니다. 이 ID는 클라이언트 측에서 정의한 고유한 문자열 값입니다. 클라이언트는 이 ID값을 각 세션별로 고유하게(unique) 만들어야 합니다. 클라이언트가 할당한 아이디는 서버에서는 신경쓰지 않으며, 클라이언트에 그대로 반환됩니다.   
   
** Connecting to the Server 
:PROPERTIES:
:CUSTOM_ID: Connecting to the Server
:END:
   네트워크에서 서버에 연결하는 방법은 3가지가 있습니다. 웹 소켓, 롱 폴링, 그리고 [[https://grpc.io/][gRPC]]입니다.

   클라이언트가 웹 소켓이나 롤 폴링같은 방식으로 HTTP(S)를 통해 서버와 연결 됐을 때, 서버는 다음과 같은 endpoint를 제공합니다.

   + ~/v0/channels~ 는 웹 소켓 연결 시 사용됩니다.
   + ~/v0/channels/lp~ 는 롱 폴링 연결 시 사용됩니다.
   + ~/v0/file/u~ 는 파일 업로드 시 사용됩니다.
   + ~/v0/file/s~ 는 파일 다운로드에 사용됩니다.

   ~v0~ 은 API 버전을 나타냅니다(현재 버전 0). 모든 HTTP(S) 요청은 API 키가 포함되어 있어야 합니다. 서버는 아래의 순서대로 API 키를 확인합니다.

   + HTTP header ~X-Tinode-APIKey~
   + URL query parameter ~apikey~(/v0/file/s/abcdefg.jpeg?apikey=...)
   + Form value ~apikey~
   + Cookie ~apikey~

   편의상 모든 데모 앱에는 기본 API 키가 포함되어 있습니다. [[https://github.com/tinode/chat/tree/master/keygen][~keygen~ 유틸리티]]를 사용하여 프로덕션용 고유 키를 생성하세요.

   서버에 연결되면 클라이언트는 서버에 ~{hi}~ 메세지를 전송해야 합니다. 서버는 이에 성공 또는 에러를 나타내는 ~{ctrl}~ 메세지로 응답합니다. 응답의 ~param~ 필드는 서버의 프로토콜 버전 ~"params": {"ver":"0,15"}~ 를 포함하며, 다른 값을 포함할 수 있습니다.

*** gRPC
	[[https://github.com/tinode/chat/blob/master/pbx/model.proto][proto file]]에서 gRPC API가 어떻게 정의되어 있는지 확인하세요. gRPC API는 루트 사용자가 다른 사용자들 대신해서 메세지를 보내거나 사용자를 삭제하는 등 이 문서에서 설명하는 내용보다 좀 더 많은 기능을 가지고 있습니다. 

	protoubf의 message의 ~bytes~ 필드에는 JSON 인코딩 UTF-8 콘텐츠가 필요합니다. 예를 들어, 문자열은 반드시 UTF-8 bytes로 변환되기 전에 따옴표로 감싸져 있어야 합니다.(Go: ~[]byte("\"\some string"")~), (Python 3: ~'"another string".encode('utf-8')'~)
*** WebSocket
	모든 메세지들은 각 메세지마다 하나의 텍스트 프레임으로 전송됩니다. 바이너리 형식은 추 후에 사용하기 위해 예약되어 있습니다. 기본적으로 서버는 Origin 헤더에 값이 있는 연결을 허용합니다.

*** Long Polling
	롱 폴링은 ~HTTP POST~ 또는 ~GET~ 메소드로 연결됩니다(POST를 권장). 클라이언트의 첫 번째 요청에 대한 응답으로 서버는 ~params~ 에 ~sid~(세션 ID) 값을 포함하는 ~{ctrl}~ 메세지를 전송합니다. 롱 폴링 클라이언트는 첫 번째 이후 모든 요청에 URL 또는 request body에 ~sid~ 를 포함하여야 합니다.

	서버는 모든 origin에 대하여 연결을 허용합니다. 예: ~Access-Control-Allow-Origin: *~

*** Out of Band Large Files
	대용량 파일은 ~HTTP POST~, ~Content-Type: multipart/form-data~ 를 사용하여 전송됩니다. 자세한 내용은 [[#Out-of-Band Handling of Large Files][여기]]를 참고하세요.

*** Running Behind a Reverse Proxy
	Tinode 서버는 NGINX와 같은 리버스 프록시 환경에서도 실행되도록 설정할 수 있습니다. 효율성을 위해 unix 소켓 파일 경로를 설정하여 unix 소켓을 통해 일반 연결, 또는 grpc 연결 등을 허용할 수 있습니다. 예:~unix:/run/tinode.sock~. ~use_x_forwarded_for~ 설정 파라메터를 ~true~ 로 설정하여 ~X-Forwarded-For~ HTTP 헤더에서 클라이언트의 IP 주소를 읽도록 서버를 구성 할 수도 있습니다.

* Users
  사용자(User)는 실제 사람, 즉 메세지를 만들고 사용하는 사람을 의미합니다.
  
  사용자에겐 일반적으로 두 가지 인증 레벨이 있는데, 인증(~auth~), 익명(~anon~)이 있습니다. 이 외의도 ~root~ 레벨이 있는데 이 레벨은 ~gRPC~ 를 통해서면 접근할 수 있고 ~root~ 사용자는 다른 사용자 대신 메세지를 보낼 수 있습니다.

  처음 연결될 때 클라이언트 애플리케이션은 ~{acc}~ 또는 ~{login}~ 메세지를 보내 사용자를 인증할 수 있습니다.

  사용자는 저마다 고유의 ID값을 가지고 있습니다. 이 ID값은 ~user~ 로 시작하는 base64-encoded 64-bit numeric 값입니다(예: ~usr2il9suCbuko~). 사용자는 또한 아래의 속성들을 지닙니다.

  + ~created~: 사용자 레코드가 생성된 시간(timestamp)
  + ~updated~: 사용자의 ~public~ 값이 갱신된 시간(timestamp)
  + ~status~: 사용자 계정의 상태
  + ~username~: ~base~ 인증(ID/PW login)에 사용되는 고유한 값입니다. username은 다른 사용자가 볼 수 없습니다.
  + ~defacs~: 인증 사용자나 익명 사용자와 P2P 대화를 위한 사용자의 기본 액세스 모드를 설명하는 개체입니다. 자세한 내용은 [[#Access Control][Access control]]을 참고하세요.
	- ~auth~: ~auth~ 사용자를 위한 기본 액세스 모드
	- ~anon~: ~anon~ 사용자를 위한 기본 액세스 모드
  + ~public~: 애플리케이션에서 정의한 사용자에 대한 정보가 담긴 오브젝트. 누구든지 쿼리문을 이용해 ~public~ 데이터를 조회할 수 있습니다.
  + ~private~: 애플리케이션에서 정의한 사용자에 대한 고유한 정보가 담긴 오브젝트. 오직 자기 자신만 조회할 수 있습니다.
  + ~tags~: [[#fnd][discovery]] and credentials.

  사용자 계정은 상태값을 가집니다. 상태값 종류는 다음과 같습니다.

  + ~ok~ (normal): 기본 상태, 계정에 아무런 제약이 없고 정상적인 상태임을 의미합니다.
  + ~susp~ (suspended): 사용자를 [[#fnd][검색]]을 통해서도 찾을 수 없을뿐만 아니라 계정에 접근 자체를 할 수 없는 상태를 의미합니다. 관리자가 상태를 복구할 수 있습니다.
  + ~del~ (soft-deleted): 사용자가 삭제 처리되었지만 데이터는 존재하는 상태를 의미합니다. 사용자 삭제는 현재 지원하지 않습니다.
  + ~undef~ (undefined): 관리자가 내부적으로 사용합니다. 다른 곳에서 사용해서는 안됩니다.

  사용자는 서버에 동시에 여러 개의 연결(세션)을 유지할 수 있습니다. 각 세션에는 클라이언트에서 제공하는 ~User-Agent~ 태그가 달리며 이 태그값은 클라이언트 소프트웨어별로 다릅니다.

  로그아웃은 애초에 설계단계부터 지원하지 않았습니다. 만약 애플리케이션에서 사용자를 전환해야 한다면, 새 사용자 인증을 이용해 연결을 새로 하기만 하면 됩니다.
  
** Authentication
   :PROPERTIES:
   :CUSTOM_ID: Authentication
   :END:
   인증(Authentication)은 [[https://en.wikipedia.org/wiki/Simple_Authentication_and_Security_Layer][SASL]]과 컨셉이 매우 유사합니다. 각각 다른 인증 방법을 구현할 수 있도록 어댑터를 제공하고 있습니다. 인증 구현체(Authenticators)는 [[#acc][~{acc}~]] 를 이용해 사용자를 등록하거나 [[#login][~{login}~]] 을 할 때 사용됩니다. 서버는 다음과 같은 인증 방법을 제공합니다.

   + ~token~ 방식은 암호화된 토큰을 이용해 인증합니다.
   + ~basic~ 방식은 login-password 인증합니다.
   + ~anonymous~ 방식은 채팅을 통한 고객 지원 요청 처리와 같은 임시 사용자를 위해 디자인 되었습니다.
   + ~rest~ 방식은 JSON RPC를 통해 외부 인증 시스템을 사용할 수 있도록 하는 [[https://github.com/tinode/chat/tree/master/server/auth/rest][meta-method]]입니다.

   이 외에 다른 인증 방식도 어댑터를 직접 구현하여 사용할 수 있습니다.

   ~token~ 은 기본 인증 방식으로 사용합니다. 이 토큰들은 토큰 인증에 가볍게 사용할 수 있도록 설계되었습니다. 예를 들어, 토큰 인증모듈(authenticator)는 일반적으로 데이터베이스에 접근하지 않고 모든 작업을 메모리 안에서 처리합니다. 다른 모든 인증 방법은 토큰을 얻거나 갱신하는데만 사용합니다. 일단 토큰이 확보되면 이 후 로그인 작업에서 이를 사용합니다.

   ~basic~ 인증 모듈은 username:password 형식의 문자열을 base64-encoded을 이용해 암호화 된 문자열을 사용합니다. 이때 username은 콜론문자(:)를 포함하지 않아야 합니다(ASCII 0X3A).

   ~anonymous~ 계정을 만들 때 사용할 수 있으며, 로그인에는 사용할 수 없습니다. 사용자는 ~anonymous~ 인증 체계를 사용하여 계정을 만들고 해당 계정에 로그인 할 수 있는 암호화 된 토큰을 얻습니다. 이 토큰을 잃어버리거나 만료되면 사용자는 더이상 해당 계정에 액세스 할 수 없습니다.

   컴파일 된 인증 모듈은 설정 파일의 ~logical_names~ 값을 수정하여 변경할 수 있습니다. 예를 들어, 별도 제작된 ~rest~ 인증 모듈을 ~basic~ 인증 모듈을 대신해서 사용하거나 ~token~ 인증 모듈을 사용자로부터 숨길 수 있습니다. 이 기능은 설정 파일의 ~logical_name:actual_name~ 에서 actual_name 값을 바꾸거나 ~actual_name:~ 값을 숨겨서 활성화 할 수 있습니다. 예를 들어, 기본 인증에 ~rest~ 서비스를 사용하고 싶으면 ~"logical_names":["basic:rest"]~ 처럼 설정하면 됩니다.

*** Creating an Account
	새 계정을 만들 때, 사용자는 서버에 나중에 해당 계정에 접근할 인증 방법을 등록해야 합니다. 계정 생성은 ~basic~, ~anonymous~ 인증만 사용할 수 있습니다. ~basic~ 인증은 고유 아이디 및 비밀번호를 서버에 등록해야 합니다. ~anonymous~ 는 인증 관련 내용을 등록하지 않습니다.

	사용자가 ~{acc login=true}~ 를 셋팅했다면 즉시 인증을 위해 새 계정을 사용할 수 있습니다. ~login=false~ 일 경우엔(또는 설정되지 않았다면) 새 계정은 생성되지만 계정을 생성한 세션의 인증 상태는 변경되지 않습니다. ~login=true~ 일 경우 서버는 생성된 새 계정으로 세션 인증을 시도하고 {acc} 요청에 대한 성공 응답에는 인증 토큰이 포함됩니다. 이 룰은 익명 인증 시에 특히 중요합니다.	

*** Logging in
	로그인은 ~{login}~ 요청을 통해서 실행됩니다. 로그인은 ~basic~, ~token~ 인증을 통해서만 가능합니다. 모든 로그인은 200 코드와 ~token~ 인증에 사용할 토큰을 ~{ctrl}~ 메세지를 통해 응답 받거나, 300 코드와 추가 인증과 메소드 종속 문제, 또는 4xx 코드와 추가 정보를 요청합니다.^{(역자 주: 의역이에요.)}

	토큰에는 서버 구성 만료 시간이 있으므로 주기적으로 갱신해야 합니다.

*** Changing Authentication Parameters
	:PROPERTIES:
	:CUSTOM_ID: Changing Authentication Parameters
	:END:
	사용자가 아이디나 패스워드같은 인증 관련 파라메터를 변경하려면 ~{acc}~ 사용해서 요청을 해야한다. 현재는 ~basic~ 인증만 지원한다.

	#+BEGIN_SRC json
acc: {
  id: "1a2b3", // string, client-provided message id, optional
  user: "usr2il9suCbuko", // user being affected by the change, optional
  token: "XMg...g1Gp8+BO0=", // authentication token if the session
                             // is not yet authenticated, optional.
  scheme: "basic", // authentication scheme being updated.
  secret: base64encode("new_username:new_password") // new parameters
}
	#+END_SRC

	패스워드만 바꾸고 싶다면, ~username~ 필드는 비워놓아야 한다(예: ~secret: base64encode("new_password")~).

	세션이 인증되지 않은 상태라면, request는 무조건 ~token~ 을 포함하고 있어야 한다. 이 ~token~ 은 로그인을 통해 얻은 일반 인증 토큰이거나, [[#resetting a password][비밀번호 재설정]] 작업을 통해 얻는 토큰일 수 있습니다. 세션이 인증되면 ~token~ 을 포함하지 않아야 합니다. 만약 ~ROOT~ 레벨로 인증했다면 ~user~ 값에 다른 유효한 사용자의 ID값을 셋팅할 수 있습니다. 그렇지 않다면 이 값을 빈 값으로 유지하거나(기본값: 현재 사용자) 자기 자신의 ID값을 할당해야 합니다.

*** Resetting a Password, i.e. "Forgot Password"
    :PROPERTIES:
    :CUSTOM_ID: resetting a password
    :END:
    아이디나 비밀번호를 초기화 할 때(또는 인증 모듈이 지원하는 인증용 시크릿 토큰), ~{login}~ 메세지를 ~scheme~, ~reset~, 그리고 base64-encoded 문자값("~authentication scheme to reset secret for~:~reset method~:~reset method value~")을 포함한 ~secret~ 값을 전송합니다. 가장 일반적인 케이스로 이메일의 비밀번호 수정을 하는 코드는 아래와 같습니다.

	#+BEGIN_SRC
login: {
  id: "1a2b3",
  scheme: "reset",
  secret: base64encode("basic:email:jdoe@example.com")
}
	#+END_SRC

	여기서 ~jdoe@example.com~ 은 이전에 검증된 사용자의 이메일입니다.

	이메일이 등록된 데이터와 일치하면, 서버는 비밀번호를 재설정 하기 위한 지시 사항과 함께 지정된 방법 및 주소를 사용하여 메세지를 보냅니다. [[#Changing Authentication Parameters][Changing Authentication Parameters]] 섹션에 설명된대로 이메일에는 ~{acc}~ request에 포함할 수 있는 시크릿 코드가 포함되어 있습니다.

** Suspending a User
   사용자 계정은 관리자에 의해 정지될 수 있습니다. 계정이 정지되면 사용자는 더이상 로그인 할 수 없고 서비스도 이용할 수 없습니다.

   ~root~ 사용자만이 다른 계정을 정지시킬 수 있습니다. 관리자에 의해 계정이 정지된 사용자는 아래의 메세지를 받습니다.

   #+BEGIN_SRC
acc: {
  id: "1a2b3", // string, client-provided message id, optional
  user: "usr2il9suCbuko", // user being affected by the change
  status: "suspended"
}
   #+END_SRC

   정지가 해제된 계정은 위와 동일한 메세지를 수신하지만 ~status: "ok"~ 값이 담긴 메세지를 수신합니다. 관리자는 ~{get what="desc"}~ 커맨드를 실행하여 사용자의 ~me~ topic을 조회할 수 있습니다.

** Credential Validation
   서버는 필요하다면 특정 인증 체계를 이용한 사용자 계정 인증 기능을 선택적으로 구성할 수 있습니다. 예를 들어, 사용자에게 고유한 이메일, 전화번호 등을 제공하도록 요구하거나 계정 등록 조건으로 보안 문자를 해결하도록 요구할 수 있습니다.

   서버는 약간의 설정 변경만으로 이메일 인증을 지원할 수 있습니다. 대부분 잘 동작하며, 문자 메세지를 보내기 위해서는 별도 상용 서비스가 필요하기 때문에  전화번호 인증 기능은 제대로 동작하지 않습니다.

   자격 증명이 활성화 된 상태일 경우, 사용자는 항상 유효성 검사에 통과된 상태여야 합니다. 필수 자격 증명을 변경해야 하는 경우엔 사용자가 먼저 새 자격 증명을 추가하고 유효성 검사를 한 다음 이전 자격 증명을 제거해야 합니다.

   자격 증명은 ~{acc}~ 메세지를 보내 할당되고, ~{set topic="me"}~ 를 통해 추가되고, ~del topic="me"~ 를 통해서 삭제됩니다. 자격 증명은 ~{login}~ 또는 ~{acc}~ 메세지를 전송하여 클라이언트 측에서 확인됩니다.

** Access Control
   :PROPERTIES:
   :CUSTOM_ID: Access Control
   :END:
   Access Control은 Access Control 목록(ACLs) 또는 Bearer Token(bearer token은 0.15 버전부터는 구현되지 않음)을 통해 Topic에 대한 Access를 관리합니다.

   Access Control은 대부분 group topic에 사용됩니다. ~me~, P2P Topic에 대해서는 현재 상태 알림을 관리하고 1:1 대화를 시작하거나 대화를 중지하는 등 제한적인 용도로 사용됩니다.
   사용자의 Topic에 대한 Access는  권한을 요청하는 "want", 그리고 Topic에 대한 매니저 권한을 부여하는 "given" 등 두 가지로 나뉩니다. 각 권한은 bitmap에 bit 단위로 표현됩니다. 이는 존재하거나 없을 수 있습니다. 실제 Access는 원하는 권한(want)와 부여된 권한(given)의 bit값을 AND 연산한 결과로 결정됩니다. 연산 결과(즉, 권한)는 ASCCII 문자열 형태로 메세지로 전달됩니다. 다음 목록의 문자열은 설정된 권한 bit를 의미합니다.

   + No Access: ~N~, 권한이 명시적으로 설정되지 않았음을 의미합니다. 일반적으로 기본 권한이 적용되지 않아야 함을 나타냅니다.
   + Join: ~J~, Topic을 구독할 수 있는 권한을 나타냅니다.
   + Read: ~R~, ~{data}~ 패킷을 수신할 수 있는 권한을 나타냅니다.
   + Write: ~W~, ~{pub}~ 토픽에 대한 권한을 나타냅니다.
   + Presense: ~P~, ~{pres}~ 메세지를 수신하여 현재 상태를 갱신할 수 있는 권한을 나타냅니다.
   + Approve: ~A~, Topic 참여 요청을 승인할 수 있는 권한을 나타냅니다. 이 권한을 가진 사용자는 해당 Topic의 관리자입니다.
   + Sharing: ~S~, 다른 사용자를 Topic에 초대할 수 있는 권한을 나타냅니다.
   + Delete: ~D~, 메세지 영구 삭제 권한, 토픽의 소유자만 이 기능을 사용할 수 있습니다.
   + Owner: ~O~, 토픽의 소요자를 의미합니다. Topic 당 최대 한명의 소유자만 존재할 수 있습니다. 일부 Topic은 소유자가 없을 수 있습니다.
   
   Topic의 기본 액세스는 Topic 생성 시 ~{sub, desc, defacs}~ 에 의해 설정되며, 이후에 ~{set}~ 메세지를 이용해 수정할 수 있습니다. 기본 액세스는 인증 사용자와 익명 사용자 두 범주에 대해 정의됩니다. 이 값은 모든 새 참석자(subscription)에 대해 "given" 권한이 적용됩니다.

   클라이언트는 ~{sub}~, ~{set}~ 메세지의 권한을 빈 문자열로 대체하여 Tinode 기본 권한으로 초기화 할 수 있습니다. 클라이언트가 Topic을 생성할 때 기본 액세스 권한을 설정하지 않으면 인증된 사용자는 ~RWP~ 권한을 부여받게 되고 익명 사용자는 빈 권한을 부여받아 관리자에게 별도로 승인을 받아야만 Topic에 참여할 수 있습니다.

   액세스 권한은 ~{set}~ 메세지를 사용해 사용자별로 할당할 수 있습니다.

* Topics 
  Topic은 한명 또는 여러명이서 커뮤니케이션을 하는 채널을 의미합니다. 모든 토픽은 persistent property를 가지고 있습니다. 이러한 토픽의 property는 ~{get what="desc"}~ 메세지를 이용해서 쿼리를 요청할 수 있습니다.

  아래의 Topic property 목록은 쿼리를 호출하는 사용자가 누구든 독립적으로 존재합니다.

  + ~created~: Topic이 생성된 시간(timestap)
  + ~updated~: Topic의 ~public~ 또는 ~private~ 속성이 마지막으로 수정된 시간(timestamp)
  + ~touched~: Topic에 마지막으로 메세지가 전송된 시간(timestamp)
  + ~defacs~: 인증 사용자와 익명 사용자를 위한 액세스 모드를 나타내는 속성. 자세한 내용은 [[#Access Control][Access Control]]을 참고하세요
  + ~auth~: 인증 사용자를 위한 액세스 모드를 나타내는 속성
  + ~anon~: 익명 사용자를 위한 액세스 모드를 나타내는 속성
  + ~seq~: Topic에 전송된 최신 ~{data}~ 메세지의 고유 아이디. 서버측에서 생성한 integer 값.
  + ~public~: Topic을 설명하는 어플리케이션 정의 객체. Topic을 구독할 수 있는 사람은 누구나 Topic의 public data를 조회할 수 있습니다.

  사용자 종속 Topic 속성 목록

  + ~acs~: 현재 사용자의 해당 Topic에 대한 액세스 권한을 나타내는 속성. 자세한 내용은 [[#Access Control][Access Control]]을 참고하세요.
  + ~want~: 현재 사용자가 요청한 접근 권한
  + ~given~: 현재 사용자의 접근 권한
  + ~private~: 현재 사용자 고유의 어플리케이션 정의 객체.

  Topic은 보통 구독자가 있습니다. 구독자 중 한명은 전체 액세스 권한이 있는 Topic 소유자로 지정될 수 있습니다(~O~ 액세스 권한). 구독자 목록은 ~{get what="sub"}~ 메세지를 이용해서 조회할 수 있습니다. 구독자 목록은 ~{meta}~ 메세지의 ~sub~ 섹션 형태로 반환됩니다.

** ~me~ Topic
   ~me~ Topic은 모든 사용자가 각자 계정을 생성할 때 자동으로 생성됩니다. 이 Topic은 계정 정보를 관ㄹ리하고 관심있는 사람과 Topic으로부터 알림을 받는 용도로 사용됩니다. ~me~ Topic은 소유자가 없습니다. 이 Topic은 삭제하거나 구독 취소를 할 수 없습니다. 모든 관련 커뮤니케이션을 중단하고 사용자가 오프라인 상태임을 나타낼 수 있습니다(하지만 사용자는 여전히 로그인 되어있고 다른 Topic을 사용할 수 있습니다.).

   ~me~ Topic에 보낸 ~{get what = "desc"}~ 메세지는 ~{meta}~ 메세지가 포함된 ~desc~ 섹션이 topic 파라메터와 함께 자동으로 반환됩니다([[#Topic][Topic]] 섹션을 참고하세요). ~me~ topic의 ~public~ 파라메터는 사용자 연결에 표시하려는 데이터입니다. ~public~ 파라메터를 변경하면 ~me~ Topic뿐만 아니라 사용자의 ~public~ 정보가 표시된 모든 곳이 변경됩니다.

   다른 Topic에 ~{get what="sub"}~ 메세지를 보내면 해당 토픽의 구독자 목록을 반환하는것과 달리 ~me~ Topic에 보내는 ~{get what="sub"}~ 메세지는 현재 사용자가 구독한 Topic 목록을 반환합니다.

   + seq: 서버측에서 발급한 topic의 마지막 message 고유 ID값
   + recv: 현재 사용자가 수신받은 메세지에 대해 직접 설정한 seq 값
   + read: 현재 사용자가 읽은 메세지에 대해 직접 설정한 seq 값
   + seen: P2P Topic 구독의 경우, 사용자의 마지막 온라인 시간(timestamp) 및 User Agent 값
   + when: 사용자의 마지막 온라인 시간
   + ua: 사용자가 마지막으로 사용한 클라이언트 소프트웨어에 대한 user agent 값

   ~me~ Topic에 보내는 ~{get what="data"}~ 메세지는 거부됩니다.

** ~fnd~ and Tags: Finding Users and Topics
:PROPERTIES:
:CUSTOM_ID: fnd
:END:

   ~fnd~ Topic은 모든 사용자가 각자 계정을 생성할 때 자동으로 생성됩니다. 이 Topic은 다른 사용자나 group Topic을 검색할 때 사용됩니다. 사용자와 group topic은 ~tags~ 키워드를 기준으로 검색합니다. 태그는 Topic 또는 사용자 생성 시 지정할 수 있으며, 이 후에는 ~{set what="tags"}~ 를 사용하여 ~me~ 또는 group Topic의 tags를 수정할 수 있습니다.

   태그는 대소문자를 구분하지 않는 유니코드 문자열(서버에서 강제로 소문자로 적용)이며, 문자 및 숫자 유니코드 [[https://en.wikipedia.org/wiki/Unicode_character_property#General_Category][클래스/카테고리]] 문자뿐만 아니라 ASCII 문자(~_~, ~.~, ~+~, ~-~, ~@~, ~#~, ~!~, ~?~)를 포함할 수 있습니다.

   태그는 네임 스페이스 역할을 하는 접두사가 있을 수 있습니다. 이 접두사는 2-16개 사이의 string 문자열이며 [a-z] 로 시작하며, 소문자 ASCII 문자 및 숫자와 콜론을 포함할 수 있는데(~:~), 예를 들어 휴대전화 태그는 ~tel:+14155551212~ 처럼 나타내며 이메일 주소는 ~email:alice@example.com~ 처럼 나타냅니다. 일부 접두사 태그는 선택적으로 고유도록 적용됩니다(unique). 이 경우 한명의 사용자 또는 Topic만 이러한 태그를 가질 수 있습니다. 특정 태그는 사용자가 변경할 수 없도록 강제할 수 있습니다. 즉, 사용자가 변경 불가능한 태그를 추가하거나 제거하려는 시도는 서버에서 거부됩니다.

   태그는 서버측에서 인덱싱되며 사용자 및 Topic 검색에 사용됩니다. 검색은 일치하는 태그 수를 기준으로 내림차순으로 정렬됩니다.

   사용자 또는 Topic을 찾기 위해 사용하는 ~fnd~ Topic의 ~public~ 또는 ~private~ 파라메터 변수를 검색 쿼리([[#Query Language][Query language]] 참고)로 설정하고 ~{get topic="fnd" what="sub"}~ 메세지를 보냅니다. 만약 ~public~, ~private~ 둘 다 설정했다면, ~public~ 값이 사용됩니다. ~private~ 쿼리는 세션과 디바이스 기기에서 유지됩니다, 예를 들어 모든 유저 세션은 같은 ~private~ 쿼리를 조회하게 됩니다. ~public~ 쿼리의 장점은 휘발성, 즉, 데이터베이스에 저장되거나 사용자 세션끼리 공유되지 않습니다. ~private~ 쿼리는 휴대 전화의 사용자 접속 목록에 있는 모든 사람의 일치 항목을 찾는 것과 같은 자주 변경되지 않는 대규모 작업 쿼리를 위한 것입니다. ~public~ 쿼리는 특정 Topic 또는 내 휴대전화 목록에 없는 사용자를 찾는 것과 같과 같이 간단한 쿼리 작업에 주로 쓰입니다.

   시스템은 발견된 사용자 또는 Topic의 세부정보를 구독 형식으로 ~subsection~ section에 담아 ~{meta}~ message 형태로 반환합니다.

   ~fnd~ Topic은 읽기 전용으로, ~fnd~ Topic에 ~{pub}~ 메세지를 보낼 경우 거절당합니다.

   /현재 지원하지 않음/ 새 사용자가 지정된 쿼리와 일치하는 태그를 등록하면 ~fnd~ Topic이 새 사용자 등록을 알리는 ~{pres}~ 메세지를 반환합니다.

   [[https://github.com/tinode/chat/tree/master/pbx][Plugins]]을 이용해 커스텀 검색 기능을 제공할 수 있습니다.

*** Query Language
:PROPERTIES:
:CUSTOM_ID: Query Language
:END:
    Tinode query language는 사용자와 Topic을 검색하기 위한 언어입니다. 쿼리는 공백 또는 쉼표로 구분된 문자열입니다. 각 검색어는 사용가 또는 Topic의 태그들과 일치합니다. 각 검색어는 RTL 방식으로 쓰여졌을 수 있지미만 쿼리는 항상 왼쪽에서 오른쪽 방향으로 파싱됩니다. 공백은 ~AND~ 연산으로 처리되며, 쉼표(앞뒤에 공백이 있는 쉼표 포함)는 ~OR~ 연산으로 처리됩니다. 연산자의 순서는 무시됩니다. ~A~ND 연산자는 AND 연산자끼리, ~OR~ 연산자는 OR 연산자끼리 그룹화됩니다. ~OR~ 연산이 ~AND~ 보다 우선순위가 높습니다. 태그 앞에 쉼표가 오면 ~OR~ 태그, 그렇지 않으면 ~AND~ 로 취급됩니다. 예를 들어, ~aaa bbb, ccc~ (~aaa AND bbb OR ccc~)는 ~(bbb or CCC) AND aaa~ 로 해석됩니다.

	공백이 포함된 검색어는 공백을 언더바(_)로 치환해서 검색해야 합니다 ~‎~ -> ~_~ (예: ~new york~ -> ~new_york~).
**** Some examples:
	 + ~flowers~: ~flowers~ 태그가 있는 사용자 또는 Topic을 검색합니다.
	 + ~flowers travel~: ~flowers~, ~travel~ 두 태그를 모두 포함한 사용자 또는 Topic을 검색합니다.
	 + ~flowers, travel~: ~flowers~ 또는 ~travel~ 둘 중 하나의 태그라도 포함한 사용자 또는 Topic을 검색합니다(또는 둘 다 포함한).
	 + ~flowers travel, puppies~: ~flowers~ 를 포함하고 ~travel~ 또는 ~puppies~ 를 포함한 사용자 또는 Topic을 검색합니다(~(travel OR puppies) AND flowers~).
	 + ~flowers, travel puppies, kitten~: ~flowers~, ~travel~, ~puppies~, ~kittens~ 중 하나라도 포함한 사용자 또는 Topic을 검색합니다. ~travel~ 과 ~puppies~ 사이에 있는 공백은 ~OR~ 연산이 ~AND~ 연산보다 우선하므로 ~OR~ 로 치환됩니다.
	
*** Incremental Updates to Queries
	/현재 지원하지 않는/ 쿼리입니다. 특히 ~fnd.private~ 는 메세지 크기 제한과 기본 데이버테이스 쿼리 크기 제한에 의해서만 제한될 수 있습니다. 전체 쿼리를 다시 작성하여 검색어를 추가하거나 제거하는 대신, 검색어를 점진적으로 추가하거나 제거할 수 있습니다.

	incremental update 요청은 왼쪽에서 오른쪽으로 처리됩니다. 또한 동일한 검색어를 여러 번 포함할 수 있습니다. 즉, ~-a_tag+a_tag~ 는 유효한 요청입니다.

*** Query Rewrite
login, 전화번호 또는 이메일로 사용자를 찾으려면 ~alice@example.com~ 대신 ~email:alice@example.com~ 같이 접두사를 사용하여 검색어를 작성해야 합니다. 이러한 방법은 사용자가 직접 쿼리를 배워야 하기 때문에 문제가 될 수 있습니다. tinode는 이 문제를 서버에서 /쿼리를 재작성/ 하는 방식으로 문제를 해결했습니다. 만약 검색어가 접두사가 없을 경우, 서버가 적절한 접두사를 붙여 쿼리를 재작성 합니다. ~fnd.public~ 에 대한 쿼리에서 원래 용어도 유지되고(쿼리문 ~alice@example.com~ 은 ~email:alice@example.com OR alice@example~ 로 재작성 됩니다.) ~fnd.private~ 에 대한 쿼리에서는 다시 작성된 검색어만 유지됩니다(~alice@example.com~ 쿼리는 ~email:alice@example.com~ 으로 재작성 됩니다.). ~alice@example~ 처럼 이메일처럼 보이는 모든 검색어는 ~email:alice@example.com OR alice@example.com~ 처럼 재작성 됩니다. 전화번호처럼 보이는 용어는 [[https://en.wikipedia.org/wiki/E.164][E.164]] 형식으로 변환되고 ~tel:+14155551212 OR +14155551212~ 형식으로 재작성됩니다. 그리고 ~fnd.public~ 에 대한 쿼리에서 로그인처럼 보이는 접두사가 없는 모든 용어는 ~alice~ -> ~basic:alice OR alice~ 형식으로 재작성 됩니다.

위에서 설명한대로 전화번호처럼 보이는 태그는 E.164 형식으로 변환됩니다. 이러한 변환에는 ISO3166-1 alpha-2 국가코드가 필요합니다. 전화번호 태그를 E.164 형식으로 변환하는 로직은 아래와 같습니다.

  + 태그에 이미 국가 전화코드가 포함되어 있으면 그대로 사용합니다. ~+1(415)555-1212~ -> ~+14155551212~.
  + 만약 태그에 접두사가 없다면, ~{hi}~ 메세지를 통해 설정된 클라이언트의 ~lang~ 필드를 참고하여 국가코드를 설정합니다.
  + 만약 클라이언트 ~hi.lang~ 값을 통해 국가코드를 추출하지 못했다면, ~tinode.conf~ 파일에 설정된 ~default_country_code~ 필드값을 이용해 국가코드를 설정합니다.
  + ~tinode.conf~ 파일의 ~default_country_code~ 값이 없을 경우, ~us~ 국가코드를 기본값으로 사용합니다.

*** Possible Use Cases
+ 사용자를 조직으로 제한합니다. 변하지 않는 태그를 사용자에게 할당에 사용자가 어느 조직에 소속되어 있는지 표시할 수 있습니다. 사용자가 다른 사용자 또는 Topic을 검색할 때, 검색에 항상 태그를 포함해야 하도록 제한할 수 있습니다. 이 방식은 서로 다른 조직이 서로 검색되지 않도록 사용자를 세분화 하는 데 사용할 수 있습니다.
+ 위치 정보를 기반으로 검색할 수 있습니다. 클라이언트가 주기적으로 사용자의 위치를 기반으로 생성한 [[https://en.wikipedia.org/wiki/Geohash][geohash]] 태그를 등록할 수 있습니다. 특정 지역에서 사용자를 검색하면 geosh 태그가 해당 위치에 속한 사용자를 검색할 수 있습니다.
+ Search by numerical range, such as age range. The approach is similar to geohashing. The entire range of numbers is covered by the smallest possible power of 2, for instance the range of human ages is covered by 27=128 years. The entire range is split in two halves: the range 0-63 is denoted by 0, 64-127 by 1. The operation is repeated with each subrange, i.e. 0-31 is 00, 32-63 is 01, 0-15 is 000, 32-47 is 010. Once completed, the age 30 will belong to the following ranges: 0 (0-63), 00 (0-31), 001 (16-31), 0011 (24-31), 00111 (28-31), 001111 (30-31), 0011110 (30). A 30 y.o. user is assigned a few tags to indicate the age, i.e. ~age:00111~, ~age:001111~, and ~age:0011110~. Technically, all 7 tags may be assigned but usually it's impractical. To query for anyone in the age range 28-35 convert the range into a minimal number of tags: ~age:00111~ (28-31), ~age:01000~ (32-35). This query will match the 30 y.o. user by tag ~age:00111~.

** Peer to Peer Topics
Peer to Peer(P2P) Topic은 오직 단 두 사용자의 소통을 위한 채널입니다. 같은 하나의 Topic이라도 Topic의 이름은 각 사용자마다 다릅니다. 각 사용자들은 상대방의 ID(~usr~ 뒤에 붙는 사용자의 base64 URL-encoded ID)를 Topic의 이름으로 인식합니다. 예를 들어, 사용자 ~usrOj0B3-gSBSs~, ~usrIU_LOVwRNsc~ 둘이서 P2P Topic을 시작했다면, ~usrOj0B3-gSBSs~ 사용자는 Topic의 이름을 ~usrIU_LOVwRNsc~ 로 인식합니다. 반대로 ~usrIU_LOVwRNsc~ 사용자는 Topic의 이름을 ~usrOj0B3-gSBSs~ 로 인식합니다.

P2P Topic은 사용자가 다른 사용자의 ID를 이름의 topic을 구독하면 생성됩니다. 만약 ~usrOj0B3-gSBSs~ 사용자가 ~{sub topic="usrIU_LOVwRNsc"}~ 메세지를 보낸다면 ~usrIU_LOVwRNsc~ 사용자를 대상으로 P2P Topic을 생성할 수 있습니다. Tinode는 위에서 설명한대로 ~{ctrl}~ 패킷과 함께 새로 생성된 Topic의 이름을 반환합니다. 상대 사용자는 액세스 권한이 있는 ~me~ topic을 통해 ~{pres}~ 메세지를 수신합니다.

P2P topic의 'public' 파라메터는 사용자에 따라 다릅니다. 예를 들어 사용자 A와 B 사이의 P2P topic은 사용자 A의 'public' 정보를 B에게 표시하고 반대의 경우도 마찬가지입니다. 사용자가 'public' 정보를 수정하면, 모든 사용자의 P2P topic에 자동으로 'public' 정보가 변경됩니다.

P2P topic의 'private' 파라메터는 다른 topic 타입과 마찬가지로 각 사용자가 각각 개별적으로 정의합니다.

** Group Topics
Group Topic은 여러 사용자끼리 통신할 수 있는 채널을 의미합니다. group topic의 이름은 ~grp~ 또는 ~chn~ 으로 시작하는 base64 URL-encoded 문자열로 구성되어 있습니다. 그룹 이름의 내부 구조나 길이는 무조건 이 형식의 갖추어야 합니다.

Group Topic은 각 참가자별 접근 권한을 통해 참가자 수를 제한할 수 있습니다(설정 파일의 ~max_subscriber_count~ 값에 의해 제어). Group topic은 또한 읽기 전용 사용자 수(~readers~)를 설정할 수 있습니다. 모든 ~reader~ 들은 동일한 접근 권한을 가집니다. ~reader~ 가 활성화 된 group topic을 ~channels~ 라고 합니다.

Group Topic은 topic 필드에 ~new~ 또는 ~nch~ 로 시작하는(예: ~new~, ~newAbC123~) 텍스트를 포함한 ~{sub}~ 메세지를 전송하여 생성할 수 있습니다. Tinode 서버는 새로 생성된 topic의 이름이 포함된 ~{ctrl}~ 메세지를 반환할 것입니다. 예를 들어, ~{sub topic="new"}~ 메세지를 전송하면 ~{ctrl topic="grpmiKBkQVXnm3P"}~ 메세지가 반환됩니다. 만약 topic 생성에 실패하면 원본 topic 이름(예: ~new~, ~newAbC123~)에 오류가 보고됩니다. topic을 생성한 사용자가 해당 topic의 소유자가 됩니다. ~{set}~ message를 통해 소유권을 다른 사용자에게 양도할 수 있지만 한명의 사용자는 항상 소유자로 남아 있어야합니다.

~channel~ topic은 다른 non-channel group topic 비교해서 아래와 같은 차이가 있습니다.
+ Channel topic은 ~{sub topic="nch"}~ 메세지를 전송해서 생성합니다. ~{sub topic="new"}~ 메세지는 channel이 활성화 되지 않은 group topic을 생성합니다.
+ ~{sub topic="chnAbC123"}~ 메세지를 보내면 해당 채널에 대한 ~reader~, 구독이 생성됩니다. 이 메세지를 non-channel에 보내면 거절당합니다.
+ [[#fnd][fnd]]를 이용해 topic을 찾을 때, channel 주소는 ~chn~ 글자로 시작합니다.. non-channel topic은 ~grp~ 글자로 시작합니다.
+ channel topic 구독자가 받는 메세지는 ~From~ 필드가 없습니다. 일반 group topic 사용자가 받는 메세지에는 발신자가 표시된 ~From~ 필드가 포함되어 있습니다.
+ channel과 non-channel group topic의 기본 권한에는 차이가 있습니다. channel topic group에는 어떤 권한도 부여되지 않습니다.
+ 일반 또는 channel이 활성화 된 Topic에 새 사용자가 참여 또는 탈퇴할 경우 다른 적합한 권한이 있는 모든 topic 참여자들에게 ~{pres}~ 메세지를 전송합니다. reader가 참여하거나 탈퇴할 경우엔 ~{pres}~ 메세지를 생성하지 않습니다.

** ~sys~ Topic
~sys~ topic은 시스템 관리자와 항상 사용 가능한 통신 채널입니다. 루트가 아닌 일반 사용자는 ~sys~ topic을 구독할 수 없지만 메세지를 보낼 순 있습니다. 기존 클라이언트는 이 채널을 사용하여 JSON 형식의 보고서를 ~{pub}~ 메세지를 사용하여 악용 사례를 신고합니다. 루트 사용자는 ~sys~ topic을 구독할 수 있습니다. 일단 구독하면, 루트 사용자는 다른 사용자가 ~sys~ topic에 보낸 메세지를 받게됩니다.

* Using Server-Issued Message IDs
Tinode는 서버측에서 생성한 메세지 ID를 ~{data}~ 메세지 형태로 클라이언트 사이드에서 캐싱하는 기능을 기본적으로 지원합니다. 클라이언트는 ~{get what="desc"}~ 메세지를 전송하여 topic의 마지막 메세지 ID를 요청할 수 있습니다. 만약 반환된 ID값이 마지막으로 수신한 메세지의 ID값보다 크다면 클라이언트는 해당 topic에서 읽지 않은 메세지가 있음을, 그리고 그 갯수를 파악할 수 있습니다. 클라이언트는 ~{get what="data"}~ 메세지를 사용하여 이러한 메세지들을 가져올 수 있습니다. 클라이언트는 메세지 ID를 사용하여 메세지 목록 페이징 처리를 할 수 있습니다.

* User Agent and Presence Notifications
하나 이상의 사용자 세션이 ~me~ topic에 연결되면 사용자가 온라인 상태인 것으로 간주합니다. 클라이언트 소프트웨어는 ~{lgoin}~ 메세지를 전송할 때 ~ua~ (user agent) 필드값을 설정하여 서버에 자신을 식별시킬 수 있습니다. user agent는 아래와 같은 방식으로 ~{meta}~ 및 ~{pres}~ 메세지를 게시합니다.

+ 사용자의 첫번째 세션이 ~me~ 에 연결될 때, 해당 세션의 /user agent/ 가 ~{pres what="on" ua="..."}~ 메세지로 형태로 전송됩니다.
+ 여러 사용자가 ~me~ topic에 연결될 때, /user agent/ 는 가장 최근에 동작이 발생한 세션의 ~{pres what="ua" ua="..."}~ 메세지를 기준으로 합니다. 이때 '동작'이란 클라이언트에서 가장 최근에 메세지를 발송함을 의미합니다. 잠재적으로 과도한 트래픽을 방지하기 위해 user agent 변경 사항은 최대 1분에 한번만 설정됩니다.
+ 사용자의 마지막 세션이 ~me~ topic에서 연결이 끊어질 때, /user agent/ 는 해당 세션의 user agent를 타임스탬프와 함께 기록합니다. 이 때 user agent는 ~{pres what="off" ua="..."}~ 메세지에 의해 설정되고 이 때 타임스탬프도 함께 기록됩니다.

빈 ~ua=""~ /user agent/ 값은 허용되지 않습니다. 즉, 사용자가 비어있지 않은 /user agent/ 값을 가지고 ~me~ topic과 연결된 이후 다시 빈 /user agent/ 를 보낸다면 이 변경은 허용되지 않습니다. 빈 /user agent/ 값은 무시됩니다.

* Public and Private Fields
:PROPERTIES:
:CUSTOM_ID: Public and Private Fields
:END:
Topic과 관련 구독들은 ~public~, ~private~ 필드를 가지고 있습니다. 일반적으로, 이런 필드는 응용 프로그램에서 정의됩니다. 서버는 ~fnd~ topic을 제외하고 이러한 필드 구조를 강제하지 않습니다. 동시에, 클라이언트 소프트웨어는 상호 호환성을 위해 동일한 형식을 사용해야 합니다.

** Public
group topic과 p2p topic의 ~public~ 필드 형식은 현재 클라이언트 소프트웨어에서 ~fn~, ~photo~ 필드만 사용되지만 추 후 [[https://en.wikipedia.org/wiki/VCard][vCard]] 형태로 변경될 것으로 예상됩니다.
#+BEGIN_SRC
vcard: {
  fn: "John Doe", // string, formatted name
  n: {
    surname: "Miner", // last of family name
    given: "Coal", // first or given name
    additional: "Diamond", // additional name, such as middle name or patronymic or nickname.
    prefix: "Dr.", // prefix, such as honorary title or gender designation.
    suffix: "Jr.", // suffix, such as 'Jr' or 'II'
  }, // object, user's structured name
  org: "Most Evil Corp", // string, name of the organisation the user belongs to.
  title: "CEO", // string, job title
  tel: [
    {
      type: "HOME", // string, optional designation
      uri: "tel:+17025551234" // string, phone number
    }, ...
  ], // array of objects, list of phone numbers associated with the user
  email: [
    {
      type: "WORK", // string, optional designation
      uri: "email:alice@example.com", // string, email address
    }, ...
  ], // array of objects, list of user's email addresses
  impp: [
    {
      type: "OTHER",
      uri: "tinode:usrRkDVe0PYDOo", // string, email address
    }, ...
  ], // array of objects, list of user's IM handles
  photo: {
    type: "jpeg", // image type
    data: "..." // base64-encoded binary image data
  } // object, avatar photo. Java does not have a useful bitmap class, so keeping it as bits here.
}
#+END_SRC

~fnd~ topic의 ~public~ 필드는 [[#Query Language][검색 쿼리]]를 나타내는 문자열이 될 것으로 예상됩니다.

** Private
group topic과 p2p topic의 ~private~ 필드 형식은 key-value 형태로 구성됩니다. 현재 정의된 키는 다음과 같습니다.

#+BEGIN_SRC
private: {
  comment: "some comment", // string, optional user comment about a topic or a peer user
  arch: true, // boolean value indicating that the topic is archived by the user, i.e.
              // should not be shown in the UI with other non-archived topics.
  accepted: "JRWS" // string, 'given' mode accepted by the user.
}
#+END_SRC

아직 적용되지는 않았지만 사용자 정의 필드는 ~x-~ 로 시작하여 그 뒤에 애플리케이션 이름이 와야 합니다(예: ~x-myapp-value:"abc"~). 필드 타입은 ~string~, ~boolean~, ~number~ 또는 ~null~ 같은 원시 타입만 지원합니다.

~fnd~ topic의 ~private~ 필드는 [[#Query Language][검색 쿼리]]를 나타내는 문자열이 될 것으로 예상됩니다.

* Format of Content
:PROPERTIES:
:CUSTOM_ID: Format of Content
:END:

~{pub}~, ~{data}~ 메세지의 ~content~ 필드 형식은 응용 프로그램에서 정의되므레 서버는 이 필드에 특정 형식을 강요하지 않습니다. 클라이언트 소프트웨어는 상호 호환성을 위해 동일한 형식을 사용해야 합니다. 현재는 다음 두 가지 유형의 콘텐츠가 지원됩니다.

+ Plain Text
+ [[https://github.com/tinode/chat/blob/master/docs/drafty.md][Drafty]]

Drafty를 사용한다면, 메세지 헤더에 ~"head": {"mime": "text/x-drafty"}~ 값을 셋팅해야 합니다.

* Out-of-Band Handling of Large Files 
:PROPERTIES:
:CUSTOM_ID: Out-of-Band Handling of Large Files 
:END:
대용량 파일은 전송 시 다양한 문제를 야기합니다.

+ 메세지가 데이터베이스 필드에 저장되기 때문에 데이터베이스가 제한됩니다.
+ 메세지는 채팅 기록 다운로의 일부이므로 완전히 다운되어야 합니다.

Tinode는 대용량 파일을 위해 업로드용 endpoint ~/v0/file/u~, 다운로드용 endpoint ~v0/files/s~ 등 총 두 가지 endpoint를 제공합니다. 이 endpoint에 접근하기 위해선 [[#Connecting to the Server][API key]]와 로그인 인증정보가 필요합니다. 서버는 아래와 같은 순서로 자격 증명을 확인합니다.

**Login credentials**
+ HTTP header ~Authorization~ (https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Authorization)
+ URL query parameter ~auth~, ~secret~ (/v0/file/s/abcdefg.jpeg?auth=...&secret=...)
+ Form values ~auth~, ~secret~
+ Cookies ~auth~ and ~secret~

** Uploading
파일 업로드는 RFC 2388 multipart request를 생성 후 HTTP POST 형식으로 서버에 전송하는 방식으로 진행된다. 서버는 업로드 된 파일에 접근할 수 있는 URL이 포함된 ~307 Temporary Redirect~ 응답 또는 ~200 OK~ 와 ~{ctrl}~ 메세지를 포함한 responst body를 반환합니다.

#+BEGIN_SRC 
ctrl: {
  params: {
    url: "/v0/file/s/mfHLxDWFhfU.pdf"
  },
  code: 200,
  text: "ok",
  ts: "2018-07-06T18:47:51.265Z"
}
#+END_SRC

~307 Temporary Redirect~ 응답을 받았다면 클라이언트는 반드시 전달받은 URL을 이용에 업로드를 다시 시도해야합니다. 이 후 모든 업로드는 기본 URL을 이용해 업로드를 시도해야합니다.

~ctrl.params.url~ 에는 현재 서버에 업로드된 파일의 위치를 나타냅니다. 이 경로는 ~/v0/file/s/mfHLxDWFhfU.pdf~ 같은 절대경로, 또는 ~./mfHLxDWFhfU.pdf~ 같은 상대경로, 또는 ~mfHLxDWFhfU.pdf~ 같은 단순한 파일명이 될 수 있습니다. 전체 경로를 제외한 모든 항목은 기본 다운로드 endpoint ~/v0/file/s~ 위치를 중심으로 나타냅니다. 예를 들어, 만약 ~mfHLxDWFhfU.pdf~ 을 응답값으로 받았다면 파일 위치는 ~http(s)://current-tinode-server/v0/file/s/mfHLxDWFhfU.pdf~ 가 됩니다.

업로드 직후, 또는 리디렉션 후 파일의 URL이 수신되면 클라이언트는 URL을 사용하여 업로드 된 파일과 함께 ~{pub}~ 메세지를 첨부 파일로 보낼 수 있습니다. URL은 [[https://github.com/tinode/chat/blob/master/docs/drafty.md][Dratfy]]-formatted 형식의 ~pub.content~ 필드를 생성하거나 ~pub.heade.attachments~ 필드에서 참조하는데 사용됩니다.

#+BEGIN_SRC
pub: {
  id: "121103",
  topic: "grpnG99YhENiQU",
  head: {
    attachments: ["/v0/file/s/sJOD_tZDPz0.jpg"],
    mime: "text/x-drafty"
  },
  content: {
    ent: [
    {
      data: {
      mime: "image/jpeg",
      name: "roses-are-red.jpg",
      ref:  "/v0/file/s/sJOD_tZDPz0.jpg",
      size: 437265
    },
      tp: "EX"
    }
  ],
  fmt: [
    {
      at: -1,
    key:0,
    len:1
    }
  ]
  }
}
#+END_SRC

~head.attachments~ 필드에 URL을 나열하는 것이 중요합니다. Tinode 서버는 이 필드를 사용하여 업로드 된 파일의 갯수를 파악합니다. 필드의 list 길이가 0이 되면(예를 들어 URL이 들어있는 메세지가 삭제되거나 클라이언트가 head.attachments 필드에 URL을 포함하지 못했을 경우), 서버는 파일을 삭제하기 위해 garbage collection에 추가합니다. 상대경로 URL을 사용해야만 합니다. ~head.attachments~ 필드의 절대경로 URL는 무시됩니다. URL은 업로드에 대항 응답으로 반환된 ~ctrl.params.url~ 값이어야 합니다.

** Downloading
HTTP GET 형식으로 ~/v0/files/s~ URL로 요청하면 파일을 다운로드 할 수 있습니다. 클라이언트는 이 주소를 중심으로 상대경로를 파악해야 합니다. 예를 들어, ~mfHLxDWFhfU.pdf~ 또는 ~./mfHLxDWFhfU.pdf~ 값은 ~/v0/file/s/mfHLxDWFhfU.pdf~ URL로 서버에 요청해야 합니다.

* Push Notifications 

Tinode는 푸시 알림을 처리하기 위해 컴파일 타임 어댑터를 사용합니다. 서버는 [[https://github.com/tinode/chat/blob/master/server/push/tnpg][Tinode Push Gateway]], [[https://firebase.google.com/docs/cloud-messaging/][Google FCM]], ~stdout~ 어댑터를 제공합니다. Tinode Push Gateway와 Google FCM은 Play Services와 함께 안드로이드를 지원하며(일부 중국폰은 지원하지 않을 수 있습니다.) iOS기기 및 사파리를 제외한 모든 메이저 웹 브라우저를 지원합니다. ~stdout~ 어댑터는 실제로 푸시 알림을 지원하지는 않습니다.이 어댑터는 디버깅이나 테스트, 로깅 등의 작업을 수행할 때 매우 유용합니다. [[https://intl.cloud.tencent.com/product/tpns][TNPS]] 같은 다른 푸시 알림은 직접 어댑터를 작성할 수 있습니다.

만약 직접 커스텀 플러그인을 작성했다면, 알림 payload는 아래와 같은 형식을 따라야 합니다.

#+BEGIN_SRC
{
  topic: "grpnG99YhENiQU", // Topic which received the message.
  xfrom: "usr2il9suCbuko", // ID of the user who sent the message.
  ts: "2019-01-06T18:07:30.038Z", // message timestamp in RFC3339 format.
  seq: "1234", // sequential ID of the message (integer value sent as text).
  mime: "text/x-drafty", // optional message MIME-Type.
  content: "Lorem ipsum dolor sit amet, consectetur adipisci", // The first 80 characters of the message content as plain text.
}
#+END_SRC 

** Tinode Push Gateway

Tinode Push Gateway(TNPG)는 Tinode를 대신하여 푸시 알림을 보내는 Tinode 독점 서비스입니다. 내부적으로 Google FCM을 사용하므로 FCM과 동일한 플랫폼을 지원합니다. TNPG의 주요 장점은 FCM보다 설정을 좀 더 쉽게 할 수 있다는 것입니다. 모바일 클라이언트를 다시 컴파일 할 필요가 없으며 서버만 업데이트 하면 됩니다.

** Google FCM

[[https://firebase.google.com/docs/cloud-messaging/][Google FCM]]은 [[https://developers.google.com/android/guides/overview][Android Play]] 서비스, iPhone 및 iPad 기기 및 Safari를 제외한 모든 메이저 웹 브라우저를 지원합니다. FMC 모바일 클라이언트(iOS, Android)를 사용하려면 Google에서 얻은 사용자 인증 정보로 다시 컴파일해야 합니다. 자세한 내용은 [[https://github.com/tinode/chat/tree/master/server/push/fcm][여기]]를 참고하세요.

** Stdout

~stdout~ 어댑터는 디버깅이나 로깅 작업에 매우 유용합니다. ~STDOUT~ 에 push payload를 전달하여 파일에 기록하거나 다른 프로세스에서 읽도록 할 수 있습니다.

* Messagse

메세지는 논리적 연관 데이터 세트입니다. 메세지는 JSON 형식의 UTF-8 텍스트로 전달됩니다.

모든 클라이언트가 서버로 메세지에는 ~id~ 필드가 있을 수 있습니다. 이는 클라이언트가 메시지를 수신하고 처리했다고 서버에게 알리는 수단으로서 사용됩니다. ~id~ 는 세션 고유 문자열이어야하지만 임의의 문자열이 될 수 있습니다. 서버는 JSON 유효성을 확인하는 것 외에는 어떤 해석을 시도하지 않습니다. ~id~ 는 서버가 클라이언트 메세지에 회신할 때 변경되지 않은 상태로 반환됩니다.

서버는 반드시 쌍따옴표로 감싼 필드명으로 이루어진 엄격한 JSON 규격을 준수해야 합니다. 간결함을 위해 아래의 표기법은 필드 이름과 외부 중괄호 주위의 쌍따옴표를 생략합니다. 예제에서는 설명을 위해서만 ~//~ 주석을 사용합니다. 주석은 실제 서버 통신에서는 사용할 수 없습니다.

어플리케이션 정의 데이터를 업데이트 하기 위한 데이터는 ~{set}~, ~private~ 또는 ~public~ 같은 필드들이 있지만, 서버에서 데이터를 지우고자 할 때는 단일 유니코드 ~␡~(~\u2421~) 문자가 있는 문자열을 사용하세요. 예를 들어, ~"public": null~ 을 보내면 해당 필드는 지워지지 않지만 ~"public":"␡"~ 을 보내면 지워집니다.

인식되지 않는 필드는 서버에서 자동으로 무시됩니다.

** Client to Server Messages 

*** ~{hi}~

클라이언트가 서버에서 버전과 user agent 정보를 알리는 핸드쉐이크 메세지입니다. 이 메세지는 서버에 반드시 가장 먼저 보내야 합니다. 서버는 ~{ctrl}~ 에 서버 빌드 버전, wire 프로토콜 버전, long polling 방식의 경우 서버 세션 ID 및 sid 등등을 ~ctrl.params~ 필드에 담은 응답값을 반환합니다.

#+BEGIN_SRC
hi: {
  id: "1a2b3",     // string, client-provided message id, optional
  ver: "0.15.8-rc2", // string, version of the wire protocol supported by the client, required
  ua: "JS/1.0 (Windows 10)", // string, user agent identifying client software,
                   // optional
  dev: "L1iC2...dNtk2", // string, unique value which identifies this specific
                   // connected device for the purpose of push notifications; not
                   // interpreted by the server.
                   // see [Push notifications support](#push-notifications-support); optional
  platf: "android", // string, underlying OS for the purpose of push notifications, one of
                   // "android", "ios", "web"; if missing, the server will try its best to
                   // detect the platform from the user agent string; optional
  lang: "en-US"    // human language of the client device; optional
}
#+END_SRC

user agent값인 ~ua~ 값은 가급적이면 [[http://tools.ietf.org/html/rfc7231#section-5.5.3][RFC 7231 section 5.5.3]]의 권장사항을 따르겠지만 항상 따르는 것은 아닙니다. ~ua~, ~dev~ 및 ~lang~ 값을 업데이트 하기 위해서 메세지를 두 번 이상 보낼 수 있습니다. 여러번 전송되는 경우 첫번째 메세지 이후의 메세지들의 ~ver~ 값은 첫번째 메세지와 동일하거나 아예 명시하지 않아야 합니다.


*** ~{acc}~
:PROPERTIES:
:CUSTOM_ID: acc
:END:

~{acc}~ 메세지는 사용자를 생성하거나 ~tags~ 정보를 수정, 또는 ~scheme~, ~secret~ 값을 이용해 기존 사용자로 로그인을 할 수 있습니다. 계정을 새로 생성하려면 ~user~ 필드에 ~new~ 값을 설정하고 선택적으로 뒤에 문자열을 추가할 수 있습니다(예: ~newr15gsr~). 인증된 세션이나 인증 세션은 ~{acc}~ 메세지를 보내 새 사용자를 만들 수 있습니다. 인증 데이터를 업데이트하거나 현재 사용자의 인증정보를 확인하려면 사용자를 설정하지 않은 상태로 유지합니다.

~{acc}~ 메세지는 이미 존재하는 사용자의 ~desc~, ~cred~ *값을 수정할 수 없습니다.* 대신 ~me~ topic을 사용해 업데이트하세요.

#+BEGIN_SRC
acc: {
  id: "1a2b3", // string, client-provided message id, optional
  user: "new", // string, "new" to create a new user, default: current user, optional
  token: "XMgS...8+BO0=", // string, authentication token to use for the request if the
               // session is not authenticated, optional
  status: "ok", // change user's status; no default value, optional.
  scheme: "basic", // authentication scheme for this account, required;
               // "basic" and "anon" are currently supported for account creation.
  secret: base64encode("username:password"), // string, base64 encoded secret for the chosen
              // authentication scheme; to delete a scheme use a string with a single DEL
              // Unicode character "\u2421"; "token" and "basic" cannot be deleted
  login: true, // boolean, use the newly created account to authenticate current session,
              // i.e. create account and immediately use it to login.
  tags: ["alice johnson",... ], // array of tags for user discovery; see 'fnd' topic for
              // details, optional (if missing, user will not be discoverable other than
              // by login)
  cred: [  // account credentials which require verification, such as email or phone number.
    {
      meth: "email", // string, verification method, e.g. "email", "tel", "recaptcha", etc.
      val: "alice@example.com", // string, credential to verify such as email or phone
      resp: "178307", // string, verification response, optional
      params: { ... } // parameters, specific to the verification method, optional
    },
  ...
  ],

  desc: {  // object, user initialisation data closely matching that of table
           // initialisation; used only when creating an account; optional
    defacs: {
      auth: "JRWS", // string, default access mode for peer to peer conversations
                   // between this user and other authenticated users
      anon: "N"  // string, default access mode for peer to peer conversations
                 // between this user and anonymous (un-authenticated) users
    }, // Default access mode for user's peer to peer topics
    public: { ... }, // application-defined payload to describe user,
                // available to everyone
    private: { ... } // private application-defined payload available only to user
                // through 'me' topic
  }
}
#+END_SRC

서버는 ~{ctrl}~ 에 새 사용자의 정보를 담고 있는 ~params~ 를 포함한 응답값을 반환합니다. 만약 ~desc.defacs~ 값이 없다면, 서버는 서버 기본 액세스 값을 할당합니다.

계정 생성에 지원되는 유일한 인증 방법은 ~basic~, ~anonymous~ 입니다.

*** ~{login}~
:PROPERTIES:
:CUSTOM_ID: login
:END:


로그인은 현재 세션을 인증하는 데 사용됩니다.

#+BEGIN_SRC
login: {
  id: "1a2b3",     // string, client-provided message id, optional
  scheme: "basic", // string, authentication scheme; "basic",
                   // "token", and "reset" are currently supported
  secret: base64encode("username:password"), // string, base64-encoded secret for the chosen
                  // authentication scheme, required
  cred: [
    {
      meth: "email", // string, verification method, e.g. "email", "tel", "captcha", etc, required
      resp: "178307" // string, verification response, required
    },
  ...
  ],   // response to a request for credential verification, optional
}
#+END_SRC

서버는 ~{login}~ 패킷에 ~{ctrl}~ 메세지를 응답값으로 반환합니다. 메세지의 ~params~ 에는 현재 로그인 한 사용자의 ~user~ 정보가 포함되어 있습니다. ~token~ 값에는 인증에 사용할 수 있는 암호화 된 문자열이 포함되어 있습니다. ~expires~ 값은 토큰의 만료시간을 나타냅니다.


*** ~{sub}~
:PROPERTIES:
:CUSTOM_ID: sub
:END:

~{sub}~ 패킷은 다음과 같은 기능을 제공합니다.
+ 새 topic 생성
+ 이미 존재하는 topic 구독
+ 이전에 구독한 topic에 세션 연결
+ topic 데이터 가져오기

사용자는 ~{sub}~ 패킷의 ~topic~ 필드에 ~"new"~ 값을 담아 전송하면 새 그룹 topic을 생성할 수 있습니다. 서버는 topic을 생성하고 새로 생성된 topic의 이름을 세션에 응답값으로 반환합니다.

사용자는 ~{sub}~ 패킷의 ~topic~ 필드에 대상 사용자의 ID값을 담아 전송하면 1대1(peer to peer) topic을 생성할 수 있습니다.

topic을 새로 생성하면 생성한 사용자는 항상 해당 topic을 구독하며 현재 연결된 세션에 topic이 연결됩니다.

사용자가 topic과 관계가 없을 경우 ~{sub}~ 패킷을 보내면 topic이 생성됩니다. 구독은 세션의 사용자와 topic 사이에 관계가 연결되는 것을 의미합니다.

topic에 참여하는 것은 세션이 topic에서 제공하는 콘텐츠를 소비하는 것을 의미합니다. 서버는 context에 따라 구독과 가입/연결을 자동으로 구분합니다. 만약 사용자가 topic과 관련이 없을 경우, 서버는 사용자를 해당 topic을 구독시키고 현재 세션을 topic에 연결합니다. 만약 사용자가 topic과 관련이 있을 경우, 세션을 topic에 연결하기만 합니다. 구독할 때, 서버는 사용자의 접근 권한과 topic의 접근 제어목록을 비교하여 접근을 허용 또는 거부할 수 있으며, topic 관리자에게 접근 승인을 요청할 수 있습니다.

서버는 ~{sub}~ 패킷에 대한 응답값으로 ~{ctrl}~ 을 반환합니다.

~{sub}~ 메세지는 ~{get}~, ~{set}~ 메세지에 각각 대응하는 ~{get}~, ~{set}~ 필드를 포함할 수 있습니다. 포함된 경우 서버는 동일한 이 후 동일한 topic에 대한 subsequent는 동일한 ~{set}~, ~{get}~ 메세지로 처리됩니다. 응답값에는 ~{meta}~, ~{data}~ 메세지가 포함될 수 있습니다.

#+BEGIN_SRC
sub: {
  id: "1a2b3",  // string, client-provided message id, optional
  topic: "me",  // topic to be subscribed or attached to
  bkg: true,    // request to attach to topic is issued by an automated agent, server should delay sending
                // presence notifications because the agent is expected to disconnect very quickly
  // Object with topic initialisation data, new topics & new
  // subscriptions only, mirrors {set} message
  set: {
  // New topic parameters, mirrors {set desc}
    desc: {
      defacs: {
        auth: "JRWS", // string, default access for new authenticated subscribers
        anon: "N"    // string, default access for new anonymous (un-authenticated)
                     // subscribers
      }, // Default access mode for the new topic
      public: { ... }, // application-defined payload to describe topic
      private: { ... } // per-user private application-defined content
    }, // object, optional

    // Subscription parameters, mirrors {set sub}. 'sub.user' must be blank
    sub: {
      mode: "JRWS", // string, requested access mode, optional;
                   // default: server-defined
    }, // object, optional

    tags: [ // array of strings, update to tags (see fnd topic description), optional.
        "email:alice@example.com", "tel:1234567890"
    ],

    cred: { // update to credentials, optional.
      meth: "email", // string, verification method, e.g. "email", "tel", "recaptcha", etc.
      val: "alice@example.com", // string, credential to verify such as email or phone
      resp: "178307", // string, verification response, optional
      params: { ... } // parameters, specific to the verification method, optional
    }
  },

  get: {
    // Metadata to request from the topic; space-separated list, valid strings
    // are "desc", "sub", "data", "tags"; default: request nothing; unknown strings are
    // ignored; see {get  what} for details
    what: "desc sub data", // string, optional

    // Optional parameters for {get what="desc"}
    desc: {
      ims: "2015-10-06T18:07:30.038Z" // timestamp, "if modified since" - return
              // public and private values only if at least one of them has been
              // updated after the stated timestamp, optional

    },

    // Optional parameters for {get what="sub"}
    sub: {
      ims: "2015-10-06T18:07:30.038Z", // timestamp, "if modified since" - return
              // public and private values only if at least one of them has been
              // updated after the stated timestamp, optional
    user: "usr2il9suCbuko", // string, return results for a single user,
                            // any topic other than 'me', optional
    topic: "usr2il9suCbuko", // string, return results for a single topic,
                            // 'me' topic only, optional
      limit: 20 // integer, limit the number of returned objects
    },

    // Optional parameters for {get what="data"}, see {get what="data"} for details
    data: {
      since: 123, // integer, load messages with server-issued IDs greater or equal
            // to this (inclusive/closed), optional
      before: 321, // integer, load messages with server-issued sequential IDs less
            // than this (exclusive/open), optional
      limit: 20, // integer, limit the number of returned objects,
                 // default: 32, optional
    } // object, optional
  }
}
#+END_SRC

[[#Public and Private Fields][Public and Private Fields]] 섹션을 참고하여 ~private~ 와 ~public~ 필드 구성방법을 확인하세요.

*** ~{leave}~

~{sub}~ 메세지에 대응하는 메세지입니다. 이 메세지는 아래와 같은 두가지 기능을 제공합니다.

+ 구독취소 없이 topic을 탈퇴합니다(~unsub=false~).
+ 구독을 취소합니다(~unsub=true~).

서버는 ~{leave}~ 패킷에 대한 응답값으로 ~{ctrl}~ 을 반환합니다. 구독취소 없이 topic을 탈퇴할 경우 현재 세션에서만 구독취소가 됩니다. 구독취소를 포함해서 탈퇴할 경우 사용자의 모든 세션에서 탈퇴가 이루어집니다.

#+BEGIN_SRC
leave: {
  id: "1a2b3",  // string, client-provided message id, optional
  topic: "grp1XUtEhjv6HND",   // string, topic to leave, unsubscribe, or
                              // delete, required
  unsub: true // boolean, leave and unsubscribe, optional, default: false
}
#+END_SRC
	
*** ~{pub}~
:PROPERTIES:
:CUSTOM_ID: pub
:END:	

topic의 구독자들에게 콘텐츠를 발송하는 메세지입니다.

#+BEGIN_SRC
pub: {
  id: "1a2b3", // string, client-provided message id, optional
  topic: "grp1XUtEhjv6HND", // string, topic to publish to, required
  noecho: false, // boolean, suppress echo (see below), optional
  head: { key: "value", ... }, // set of string key-value pairs,
               // passed to {data} unchanged, optional
  content: { ... }  // object, application-defined content to publish
               // to topic subscribers, required
}
#+END_SRC

topic 구독자들은 ~content~ 가 포함된 [[#data][~{data}~]] 패킷을 수신합니다. 기본적으로 원래 세션은 현재 topic에 연결된 다른 세션과 마찬가지로 ~{data}~ 의 사본을 가져옵니다. 세션이 방금 발송한 메세지의 복사본을 받지 않으려면 ~noecho~ 값을 ~true~ 로 설정합니다.

[[#Format of Content][Format of Content]] 섹션을 참고하여 ~content~ 필드 구성방법을 확인하세요.

다음 목록은 ~head~ 필드를 위해 정의된 내용들입니다.

+ ~attachments~: 이 메세지에 포함된 미디어의 경로를 나타냅니다: ~["/v0/file/s/sJOD_tZDPz0.jpg"]~
+ ~auto~: 챗봇이나 자동 응답기처럼 자동으로 발송된 메세지의 경우 ~true~ 값이 할당됩니다.
+ ~forwarded~: 해당 메세지가 포워딩 된 메세지인지 여부를 나타냅니다. 원본 메세지의 고유 ID값이 할당됩니다: ~"grp1XUtEhjv6HND:123"~
+ ~hashtags~: 해시태그 목록을 ~#~ 기호없이 나타냅니다: ~["onehash", "twohash"]~
+ ~mentions~: 언급하려는 사용자의 고유 아이디(~@alice~)들을 나타냅니다(~~).
+ ~mime~: 메세지 내용의 MIME-type을 나타냅니다(~"text/x-drafty"~). null값이거나 해당 값이 누락됐을 경우엔 ~"text/plain"~ 값으로 간주합니다.
+ ~priority~: 메세지 표시 우선 순위를 나타냅니다. 메세지가 일정 시간동안 더 눈에 띄게 표시되어야 한다는 내용을 나타냅니다. 현재는 ~"high"~ 값만 정의되어 있는 상태입니다(~{"level": "high", "expires": "2019-10-06T18:07:30.038Z"}~). ~priority~ 값은 topic 소유자 또는 관리자(~A~ permission)만 설정할 수 있습니다. ~"expires"~ 값은 옵션입니다.
+ ~replace~: 다른 메세지를 수정/대체하는 메세지인지 여부를 나타냅니다. 수정/대체하고자 하는 메세지의topic 고유의 아이디를 나타냅니다: ~":123"~
+ ~reply~: 다른 메세지에 대한 리플 여부를 나타냅니다. 원본 메세지의 고유 ID값을 나타냅니다:~"grp1XUtEhjv6HND:123"~
+ ~sender~: 서버가 해당 메세지를 전송할 때 원본 메세지를 발송한 사람의 고유 아이디 값을 나타냅니다: ~"usr1XUtEhjv6HND"~
+ ~thread~: an indicator that the message is a part of a conversation thread, a topic-unique ID of the first message in the thread, ~":123"~; ~thread~ is intended for tagging a flat list of messages as opposite to a creating a tree.

어플리케이션 별 필드는 ~x-<application-name>~ 로 시작해야 합니다. 서버는 아직 이 규칙을 적용하지 않지만 향 후 이 규칙이 적용될 수 있습니다.

메세지의 고유 아이디는 가능하다면 ~"grp1XUtEhjv6HND:123"~ 처럼 반드시 ~<topic_name>:<seqId>~ 형식으로 시작해야 합니다. 만약 ~":123"~ 처럼 topic을 누락했다면 현재 topic으로 간주합니다.

*** ~{get}~

설명, 구독자 목록 또는 쿼리 메세지 기록같은 메타데이터에 대한 쿼리 topic입니다. 전체 응답을 받으려면 요청하는 사용자가 [[#sub][구독하고 topic에 소속]]되어 있어야 합니다. 일부 제한된 ~desc~, ~sub~ 정보의 경우엔 소속되어 있지 않아도 조회할 수 있습니다.

#+BEGIN_SRC
get: {
  id: "1a2b3", // string, client-provided message id, optional
  topic: "grp1XUtEhjv6HND", // string, name of topic to request data from
  what: "sub desc data del cred", // string, space-separated list of parameters to query;
                        // unknown values are ignored; required

  // Optional parameters for {get what="desc"}
  desc: {
    ims: "2015-10-06T18:07:30.038Z" // timestamp, "if modified since" - return
          // public and private values only if at least one of them has been
          // updated after the stated timestamp, optional
  },

  // Optional parameters for {get what="sub"}
  sub: {
    ims: "2015-10-06T18:07:30.038Z", // timestamp, "if modified since" - return
          // public and private values only if at least one of them has been
          // updated after the stated timestamp, optional
    user: "usr2il9suCbuko", // string, return results for a single user,
                          // any topic other than 'me', optional
    topic: "usr2il9suCbuko", // string, return results for a single topic,
                           // 'me' topic only, optional
    limit: 20 // integer, limit the number of returned objects
  },

  // Optional parameters for {get what="data"}
  data: {
    since: 123, // integer, load messages with server-issued IDs greater or equal
                // to this (inclusive/closed), optional
    before: 321, // integer, load messages with server-issed sequential IDs less
               // than this (exclusive/open), optional
    limit: 20, // integer, limit the number of returned objects, default: 32,
               // optional
  },

  // Optional parameters for {get what="del"}
  del: {
    since: 5, // integer, load deleted ranges with the delete transaction IDs greater
              // or equal to this (inclusive/closed), optional
    before: 12, // integer, load deleted ranges with the delete transaction IDs less
                // than this (exclusive/open), optional
    limit: 25, // integer, limit the number of returned objects, default: 32,
               // optional
  }
}
#+END_SRC

+ ~{get what="desc"}~
쿼리 topic에 대해 설명합니다. 서버는 ~{meta}~ 메세지에 요청한 데이터를 첨부하여 반환합니다. 자세한 내용은 ~{meta}~ 데이터를 참고하세요. ~ims~ 값이 정의되어 있고 데이터가 업데이트 되지 않았다면, 메세지는 ~public~, ~private~ 필드를 생략합니다.

제한된 정보는 topic에 [[#sub][소속]]되어 있지 않아도 가능합니다.

[[#Public and Private Fields][Public and Private Fields]] 섹션을 참고하여 ~private~ 와 ~public~ 필드 구성방법을 확인하세요.
+ ~{get what="sub"}~
구독중인 사용자 목록을 요청합니다. 서버는 ~{meta}~ 메세지에 요청한 구독중인 사용자 목록을 첨부하여 반환합니다. 자세한 내용은 ~{meta}~ 데이터를 참고하세요. ~me~ topic에 이 쿼리를 전송할 경우엔 내가 구독하고 있는 목록을 반환합니다. ~ims~ 값이 정의되어 있고 데이터가 업데이트 되지 않았다면, 메세지는 ~{ctrl}~ 패킷에 "not modified"를 포함한 메세지를 반환합니다.

topic에 [[#sub][소속]]되지 않고 사용자의 구독 목록만을 반환합니다.

+ ~{get what="tags"}~
인덱싱 된 태그들을 요청합니다. 서버는 ~{meta}~ 메세지에 태그 문자열 목록을 첨부하여 반환합니다. 자세한 내용은 ~{meta}~, ~{fnd}~ topic을 참고하세요. ~me~ 와 group topic만 지원합니다.

+ ~{get what="data"}~
메세지 기록을 요청합니다. 서버는 전달받은 쿼리의 ~data~ 필드값과 동일한 메세지들은 ~{data}~ 패킷에 담아 반환합니다. 데이터 메세지의 ~id~ 필드는 데이터 메세지의 일반 필드이므로 제공되지 않습니다. 모든 ~{data}~ 패킷 전송이 완료되면 ~{ctrl}~ 메세지가 전송됩니다.

+ ~{get what="del"}~
기록을 지웁니다. 서버는 ~{meta}~ 메세지에 메세지 삭제 범위를 첨부하여 반환합니다.

+ ~{get what="cred"}~

Query [[#credentail-validation][credentials]]. 서버는 ~{meta}~ 메세지에 인증 목록을 첨부하여 반환합니다. ~me~ topic만 지원합니다.

*** ~{set}~
topic의 메타데이터 수정, 또는 메세지나 topic을 삭제합니다. 요청하는 사용자는 일반적으로 topic을 [[#sub][구독하고 소속]]되어 있어야 합니다. ~desc.private~ 와 요청한 사용자의 ~sub.mode~ 값만 구독하지 않고도 수정할 수 있습니다.

#+BEGIN_SRC
set: {
  id: "1a2b3", // string, client-provided message id, optional
  topic: "grp1XUtEhjv6HND", // string, name of topic to update, required

  // Optional payload to update topic description
  desc: {
    defacs: { // new default access mode
      auth: "JRWP",  // access permissions for authenticated users
      anon: "JRW" // access permissions for anonymous users
    },
    public: { ... }, // application-defined payload to describe topic
    private: { ... } // per-user private application-defined content
  },

  // Optional payload to update subscription(s)
  sub: {
    user: "usr2il9suCbuko", // string, user affected by this request;
                            // default (empty) means current user
    mode: "JRWP" // string, access mode change, either given ('user'
                 // is defined) or requested ('user' undefined)
  }, // object, payload for what == "sub"

  // Optional update to tags (see fnd topic description)
  tags: [ // array of strings
    "email:alice@example.com", "tel:1234567890"
  ],

  cred: { // Optional update to credentials.
    meth: "email", // string, verification method, e.g. "email", "tel", "recaptcha", etc.
    val: "alice@example.com", // string, credential to verify such as email or phone
    resp: "178307", // string, verification response, optional
    params: { ... } // parameters, specific to the verification method, optional
  }
}
#+END_SRC
	
*** ~{del}~
메세지, 구독, topic, 사용자 등등을 삭제할 수 있습니다.

#+BEGIN_SRC
del: {
  id: "1a2b3", // string, client-provided message id, optional
  topic: "grp1XUtEhjv6HND", // string, topic affected, required for "topic", "sub",
               // "msg"
  what: "msg", // string, one of "topic", "sub", "msg", "user", "cred"; what
               // to delete - the entire topic, a subscription, some or all messages,
               // a user, a credential; optional, default: "msg"
  hard: false, // boolean, request to hard-delete vs mark as deleted; in case of
               // what="msg" delete for all users vs current user only;
               // optional, default: false
  delseq: [{low: 123, hi: 125}, {low: 156}], // array of ranges of message IDs
               // to delete, inclusive-exclusive, i.e. [low, hi), optional
  user: "usr2il9suCbuko" // string, user being deleted (what="user") or whose
               // subscription is being deleted (what="sub"), optional
  cred: { // credential to delete ('me' topic only).
    meth: "email", // string, verification method, e.g. "email", "tel", etc.
    val: "alice@example.com" // string, credential being deleted
  }
}
#+END_SRC

+ ~what="msg"~
사용자는 soft-delete(~hard=false~: 기본값) 또는 hard-delete(~hard=true~) 형태로 메세지를 삭제할 수 있습니다. soft-delete는 요청한 사용자에게는 삭제되겠지만 스토리지에는 여전히 데이터가 남아있습니다. soft-delete를 하기 위해서는 ~R~ 권한이 필요합니다. hard-delete는 메세지가 남아있는 모든 스토리지(~head~, ~content~)에서 메세지가 삭제됩니다. 이는 모든 사용자가 영향을 받습니다. hard-delete를 하기 위해서는 ~D~ 권한이 필요합니다. 메세지는 ~delseq~ 파라메터를 이용해 한 건 또는 여러 건을 한번에 삭제할 수 있습니다. 각 삭제 동작에는 각각의 고유한 ~delete ID~ 가 할당됩니다. The greatest ~delete ID~ is reported back in the ~clear~ of the ~{meta}~ message.

+ ~what="sub"~
구독을 삭제하면 topic 구독자에서 지정된 사용자가 삭제됩니다. 이 동작은 ~A~ 권한이 필요합니다. 사용자는 자기 자신의 구독을 삭제할 수 없습니다. 대신에 ~{leave}~ 를 사용하세요. 구독을 soft-delete(기본값) 했다면, 실제 데이터베이스에서 레코드를 삭제하지 않고 삭제된 것으로 표시됩니다.

+ ~what="topic"~
topic과 topic에 포함된 모든 구독 및 메세지를 삭제합니다. topic의 소유자만 삭제할 수 있습니다.

+ ~what="user"~
사용자 삭제는 매우 무거운 작업입니다. 주의하세요.

+ ~what="cred"~
인증정보를 삭제합니다. 유효한 인증정보와 인증 시도가 없는 인증정보는 삭제됩니다. 유효하지 않은 인증정보는 일시 삭제되어 같은 사용자가 재사용 할 수 없습니다.

*** ~{note}~

topic에 소속된 다른 클라이언트에게 전송 확인, 타이핑 알림같은 정보를 전달하기 위해 클라이언트에서 생성된 임시 알림입니다. 이 메세지는 "fire and forget" 방식으로써, 디스크에 기록되지 않고 서버가 인증해주지도 않습니다. 유효하지 않은 것으로 간주되는 메세지는 자동으로 삭제됩니다. ~{note.recv}~ 및 ~{note.read}~ 는 서버의 영구 상태를 변경합니다. 데이터는 ~{meta.sub}~ 메세지의 해당 필드에 저장 및 기독됩니다.

#+BEGIN_SRC
note: {
  topic: "grp1XUtEhjv6HND", // string, topic to notify, required
  what: "kp", // string, one of "kp" (key press), "read" (read notification),
              // "rcpt" (received notification), any other string will cause
              // message to be silently ignored, required
  seq: 123,   // integer, ID of the message being acknowledged, required for
              // rcpt & read
  unread: 10  // integer, client-reported total count of unread messages, optional.
}
#+END_SRC

현재 인식되는 값들은 아래와 같습니다.
+ kp: key press, 타이핑 알림을 의미합니다. 현재 사용자가 새 메세지를 작성하고 있음을 의미합니다.
+ recv: 클라이언트 소프트웨어가 ~{data}~ 메세지를 수신하였지만 사용자가 아직 읽지 않은 상태를 의미합니다.
+ read: 사용자가 ~{data}~ 메세지를 읽었음을 의미합니다. ~recv~ 를 포함하고 있습니다.

~read~, ~recv~ 알림은 클라이언트가 아직 읽지 않은 메세지의 총 갯수를 나타내는 ~unread~ 필드를 포함할 수 있습니다. 각 사용자별 ~unread~ 갯수는 서버가 결정합니다. 이 갯수는 새 ~{data}~ 메세지가 사용자에게 전송될 때 증가하고 ~{note unread=...}~ 메세지에서 전송한 값으로 재설정됩니다. ~unread~ 값은 절대 서버에 의해서 줄어들지 않습니다. 이 값은 iOS에서 알림 뱃지를 표시하기 위한 푸시 알림에 포함됩니다.

** Server to Client Messages

특정 요청에 대한 응답으로 생성된 세션에 대한 메세지에는 원래 메세지의 ID와 동일한 ~id~ 필드가 포함됩니다. 이 ~id~ 는 서버에서 읽지 않습니다.

서버에서 클라이언트로 전송하는 대부분의 메세지에는 서버에서 메세지를 생성한 시간을 담고있는 ~ts~ 필드가 존재합니다.
   
*** ~{data}~
:PROPERTIES:
:CUSTOM_ID: data
:END:

topic에 있는 콘텐츠를 발송합니다. 이 메세지들은 데이터베이스에 유지되는 유일한 메세지입니다. ~{data}~ 메세지들은 topic에 ~R~ 권한이 있는 모든 구독자들에게 전송됩니다.

#+BEGIN_SRC
data: {
  topic: "grp1XUtEhjv6HND", // string, topic which distributed this message,
                            // always present
  from: "usr2il9suCbuko", // string, id of the user who published the
                          // message; could be missing if the message was
                          // generated by the server
  head: { key: "value", ... }, // set of string key-value pairs, passed
                               // unchanged from {pub}, optional
  ts: "2015-10-06T18:07:30.038Z", // string, timestamp
  seq: 123, // integer, server-issued sequential ID
  content: { ... } // object, application-defined content exactly as published
              // by the user in the {pub} message
}
#+END_SRC

Data 메세지들은 서버에서 생성한 numeric ID를 가지고 있는 ~seq~ 필드를 포함하고 있습니다. 이 ID들은 각 topic별로 고유한 숫자들입니다. ID들은 각 topic별로 [[#pub][~{pub}~]] 메세지가 성공적으로 전송될 경우 1부터 1씩 증가합니다.

[[#Format of Content][Format of Content]] 섹션을 참고하여 ~content~ 필드 구성방법을 확인하세요.

~head~ 필드에 사용할 수 있는 값들에 대해 알고 싶다면 [[#pub][~{pub}~]] 메세지를 확인하세요.

*** ~{ctrl}~

에러 또는 성공 조건을 나타내는 일반적인 응답입니다. 메세지는 원본 세션으로 전송됩니다.

#+BEGIN_SRC
ctrl: {
  id: "1a2b3", // string, client-provided message id, optional
  topic: "grp1XUtEhjv6HND", // string, topic name, if this is a response in context
                            // of a topic, optional
  code: 200, // integer, code indicating success or failure of the request, follows
             // the HTTP status codes model, always present
  text: "OK", // string, text with more details about the result, always present
  params: { ... }, // object, generic response parameters, context-dependent,
                   // optional
  ts: "2015-10-06T18:07:30.038Z", // string, timestamp
}
#+END_SRC

*** ~{meta}~

topic이나 구독자에 대한 메타데이터 정보로써, ~{get}~, ~{set}~ 또는 ~{sub}~ 메세지의 응답으로 원본 세션으로 전송됩니다.

#+BEGIN_SRC
meta: {
  id: "1a2b3", // string, client-provided message id, optional
  topic: "grp1XUtEhjv6HND", // string, topic name, if this is a response in
                            // context of a topic, optional
  ts: "2015-10-06T18:07:30.038Z", // string, timestamp
  desc: {
    created: "2015-10-24T10:26:09.716Z",
    updated: "2015-10-24T10:26:09.716Z",
    status: "ok", // account status; included for `me` topic only, and only if
                  // the request is sent by a root-authenticated session.
    defacs: { // topic's default access permissions; present only if the current
              //user has 'S' permission
      auth: "JRWP", // default access for authenticated users
      anon: "N" // default access for anonymous users
    },
    acs: {  // user's actual access permissions
      want: "JRWP", // string, requested access permission
      given: "JRWP", // string, granted access permission
    mode: "JRWP" // string, combination of want and given
    },
    seq: 123, // integer, server-issued id of the last {data} message
    read: 112, // integer, ID of the message user claims through {note} message
              // to have read, optional
    recv: 115, // integer, like 'read', but received, optional
    clear: 12, // integer, in case some messages were deleted, the greatest ID
               // of a deleted message, optional
    public: { ... }, // application-defined data that's available to all topic
                     // subscribers
    private: { ...} // application-defined data that's available to the current
                    // user only
  }, // object, topic description, optional
  sub:  [ // array of objects, topic subscribers or user's subscriptions, optional
    {
      user: "usr2il9suCbuko", // string, ID of the user this subscription
                            // describes, absent when querying 'me'.
      updated: "2015-10-24T10:26:09.716Z", // timestamp of the last change in the
                                           // subscription, present only for
                                           // requester's own subscriptions
      touched: "2017-11-02T09:13:55.530Z", // timestamp of the last message in the
                                           // topic (may also include other events
                                           // in the future, such as new subscribers)
      acs: {  // user's access permissions
        want: "JRWP", // string, requested access permission, present for user's own
              // subscriptions and when the requester is topic's manager or owner
        given: "JRWP", // string, granted access permission, optional exactly as 'want'
        mode: "JRWP" // string, combination of want and given
      },
      read: 112, // integer, ID of the message user claims through {note} message
                 // to have read, optional.
      recv: 315, // integer, like 'read', but received, optional.
      clear: 12, // integer, in case some messages were deleted, the greatest ID
                 // of a deleted message, optional.
      public: { ... }, // application-defined user's 'public' object, absent when
                       // querying P2P topics.
      private: { ... } // application-defined user's 'private' object.
      online: true, // boolean, current online status of the user; if this is a
                    // group or a p2p topic, it's user's online status in the topic,
                    // i.e. if the user is attached and listening to messages; if this
                    // is a response to a 'me' query, it tells if the topic is
                    // online; p2p is considered online if the other party is
                    // online, not necessarily attached to topic; a group topic
                    // is considered online if it has at least one active
                    // subscriber.

      // The following fields are present only when querying 'me' topic

      topic: "grp1XUtEhjv6HND", // string, topic this subscription describes
      seq: 321, // integer, server-issued id of the last {data} message

      // The following field is present only when querying 'me' topic and the
      // topic described is a P2P topic
      seen: { // object, if this is a P2P topic, info on when the peer was last
              //online
        when: "2015-10-24T10:26:09.716Z", // timestamp
        ua: "Tinode/1.0 (Android 5.1)" // string, user agent of peer's client
      }
    },
    ...
  ],
  tags: [ // array of tags that the topic or user (in case of "me" topic) is indexed by
    "email:alice@example.com", "tel:+1234567890"
  ],
  cred: [ // array of user's credentials
    {
      meth: "email", // string, validation method
      val: "alice@example.com", // string, credential value
      done: true     // validation status
    },
    ...
  ],
  del: {
    clear: 3, // ID of the latest applicable 'delete' transaction
    delseq: [{low: 15}, {low: 22, hi: 28}, ...], // ranges of IDs of deleted messages
  }
}
#+END_SRC
	
*** ~{pres}~

Tinode는 중요한 이벤트를 알릴 때 ~{pres}~ 메세지를 사용합니다. [[https://docs.google.com/spreadsheets/d/e/2PACX-1vStUDHb7DPrD8tF5eANLu4YIjRkqta8KOhLvcj2precsjqR40eDHvJnnuuS3bw-NcWsP1QKc7GSTYuX/pubhtml?gid=1959642482&single=true][별도의 문서]]에서 가능한 모든 사용 사례를 설명합니다.

#+BEGIN_SRC
pres: {
  topic: "me", // string, topic which receives the notification, always present
  src: "grp1XUtEhjv6HND", // string, topic or user affected by the change, always present
  what: "on", // string, what's changed, always present
  seq: 123, // integer, "what" is "msg", a server-issued ID of the message,
            // optional
  clear: 15, // integer, "what" is "del", an update to the delete transaction ID.
  delseq: [{low: 123}, {low: 126, hi: 136}], // array of ranges, "what" is "del",
             // ranges of IDs of deleted messages, optional
  ua: "Tinode/1.0 (Android 2.2)", // string, a User Agent string identifying client
             // software if "what" is "on" or "ua", optional
  act: "usr2il9suCbuko",  // string, user who performed the action, optional
  tgt: "usrRkDVe0PYDOo",  // string, user affected by the action, optional
  acs: {want: "+AS-D", given: "+S"} // object, changes to access mode, "what" is "acs",
                          // optional
}
#+END_SRC

~{pres}~ 메세지는 아주 일시적입니다. 메세지는 저장되지 않으며 일시적으로 사용할 수 없을 경우 나중에 재전송을 시도하지 않습니다.

*** ~{info}~

클라이언트에서 생성한 ~{note}~ 알림입니다. 서버는 메세지가 규격을 준수하고 ~topic~ 과 ~form~ 필드의 내용이 정확함을 보장합니다. The other content is copied from the ~{note}~ message verbatim and may potentially be incorrect or misleading if the originator so desires.


#+BEGIN_SRC
info: {
  topic: "grp1XUtEhjv6HND", // string, topic affected, always present
  from: "usr2il9suCbuko", // string, id of the user who published the
                          // message, always present
  what: "read", // string, one of "kp", "recv", "read", see client-side {note},
                // always present
  seq: 123, // integer, ID of the message that client has acknowledged,
            // guaranteed 0 < read <= recv <= {ctrl.params.seq}; present for rcpt &
            // read
}
#+END_SRC

