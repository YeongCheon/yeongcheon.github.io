# IntelliJ 업데이트 후 appengine local 실행 이슈 해결하기


IntelliJ를 쓰기 시작한지 아직 1년도 안됐지만 거의 분기(?)마다 자동 업데이트가 되는듯 싶은데 이때마다 잘 쓰고있던 appengine local server가 제대로 동작하지 않아서 화가 났었는데 최근에 명확한 솔루션을 찾았기에 이를 기록하고자 한다.

내가 그동안 해왔던 삽질 목록은 아래와 같다.

+ 로컬서버 설정 삭제 후 재셋팅
+ rebuild(clean) project
+ cloud code 플러그인 삭제 후 재설치
+ intelliJ 삭제 후 재설치(...)

*위에 내용들 다 쓸모없으니 따라하지 않아도 된다.* 깔끔한 해결법은 아래에 있다.

#+BEGIN_SRC
$ProjectRootDirectory/build 폴더 삭제 후 로컬서버 실행
#+END_SRC

clean project를 하면 build 폴더도 알아서 비워질 줄 알았는데 아닌가보다. 앞으로는 뭐가 잘 안된다 싶으면 일단 *build 폴더를 지우고 다시 빌드를 해보자.*

