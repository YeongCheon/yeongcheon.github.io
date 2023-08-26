# Android WebView에서 firebase Auth 오류 해결하기


* 증상
안드로이드에서 [[https://developer.android.com/reference/android/webkit/WebView][WebView]] 기반 앱을 개발하고 있습니다. 대부분의 기능을 웹뷰에 의존한 형태의 단순한 앱이고, 웹뷰는 [[https://angular.io][Angular]], [[https://firebase.google.com][Firebase]] 조합의 코드가 동작하고 있습니다. 사용자 인증 관련 기능은 Firebase의 Auth를 사용하였는데, 특정 조건에서 [[https://firebase.google.com/docs/reference/js/v8/firebase.auth.Auth#onauthstatechanged][onAuthStateChanged]] 함수가 동작하지 않는 문제가 발생하였습니다. 해당 조건은 다음과 같습니다.

+ 앱을 실행한다.
+ back button을 눌러 앱을 종료한다([[https://developer.android.com/reference/android/app/Activity#finish()][finish()]]).
+ 5~10분 뒤 앱을 다시 실행한다.
+ 웹뷰 내에서 ~onAuthStateChanged~ 함수가 동작하지 않는다.

이 문제에서 특히 답답한 것은 ~onAuthStateChagned~ 함수가 에러를 발생시키거나 잘못된 값을 반환하는 것이 아닌, 말 그대로 *함수가 응답을 하지 않고 무한히 대기하는 상태가 됩니다.* 게다가 다른 모바일 브라우저나 PC 환경에서는 재현이 되지 않고 오직 안드로이드 WebView에서만 해당 문제가 발생하고 있었습니다.

* 해결법
이 문제의 정확한 원인은 사실 찾지 못했습니다. 아마 안드로이드 앱의 라이프 사이클과 Firebase Auth의 Javascript SDK 구현 방식 사이에 뭔가 원인이 있지 않을까 싶었지만 그렇게 깊게 들여다 볼 여유가 없었습니다. 아무튼 해결법은 의외로 간단했습니다. *안드로이드 Activity의 ~onDestroy~ 메서드에 ~webview.destory()~ 코드를 추가해주면 됩니다.*

#+BEGIN_SRC kotlin
override fun onDestory() {
  super.onDestory()
  this.webview.destory() // 이 코드 추가
}
#+END_SRC

~webview~ 변수가 해당 Activity에 종속된 객체라서 Activity가 destory 될 때 webview도 자동으로 destory 될 줄 알았는데 제가 잘못 알고 있었나 봅니다.

