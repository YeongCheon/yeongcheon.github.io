# 이맥스에서 여러 파일 찾아 바꾸기


대부분의 편집기에서 기본적으로 ~Ctrl + Shift + h~ 단축키에 해당하는 ~파일에서 찾기/바꾸기~ 기능을 emacs에서 사용해보자.

* 단축키 요약
  구구절절 설명하기 전에 우선 단축키 요약본을 보자

  #+BEGIN_SRC
  C-x d  (Dired mode 실행)
  Q (찾기/바꾸기 실행)
  Y (모든 변경 내역 승인)
  C-x s ! (모든 변경 내역 저장)
  #+END_SRC

* step by step
** Dired mode 실행
   ~C-x d~ : emacs에 기본적으로 탑재된 [[https://www.emacswiki.org/emacs/DiredMode][dired mode]] 를 실행해보자.
** 찾기/바꾸기 실행
   ~Q~ : query-replace-regexp 명령을 실행시키는 단축키이다. 검색하고자 하는 단어를 입력한 뒤 엔터, 그리고 치환할 단어를 입력하고 엔터를 입력하면 검색된 내용을 정말 변경할건지 다시 한번 물어본다 ~Y~ (대문자 Y)를 입력하면 검색된 모든 내용들이 변경된 buffer가 생성된다(실제 파일이 저장되는게 아니다.).
** 모든 변경 내역 저장
   ~C-x s !~ : 이제 열려있는 모든 버퍼를 저장하면 작업이 완료된다.

