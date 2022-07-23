# Android Cross-App Scripting 해결하기


* 문제
운영중인 안드로이드 앱을 플레이스토어에 업데이트 하다가 구글 측에서 [[https://support.google.com/faqs/answer/9084685][교차 앱 스크립팅]] 취약정 문제가 있다는 메일을 받았습니다. 구글 측에서 제공한 가이드대로 조치를 취했는데도 여전히 문제가 있다는 리포트를 받아서 문제의 원인 해결방법을 기록하고자 합니다.

* 원인
FCM을 통해 전달된 URL이 문제였습니다([[https://blog.ysoft.kr/40][링크 참고]]). 운영 중인 앱은 FCM을 통해 전달받은 URL을 별도의 조치 없이 그대로 webview에서 로드하는 방식으로 운영되고 있었습니다. 이 URL은 저희 측에서 만든 서버에서 보내주는 URL이라 별도로 유효성 검사 없이 제공해도 문제가 없다고 판단해서 이러한 방식을 선택했는데 이 부분이 문제였습니다.

* 해결 방법
URI는 앱 내부에 별도 변수로 관리, URL은 FCM에서 넘겨준 데이터를 그대로 사용하지 말고 uri 뒤의 path, queryParameter 부분만 추출하여 앱 내부에서 관리하는 URI와 조합하여 사용합니다. 아래와 같이 작업하면 이슈가 해결된 것으로 판단하여 스토어 업로드가 승인됩니다.

#+BEGIN_SRC kotlin
val URI = "https://example.com"
val data: String = intent.dataString // "https://example.com/path?param=test"
val url = "${URI}${data.replace(URI, "")}"

webview.loadUrl(url)
#+END_SRC

*구글은 외부에서 넘어오는 모든 URL을 신용하지 않는 것으로 추측됩니다.*

