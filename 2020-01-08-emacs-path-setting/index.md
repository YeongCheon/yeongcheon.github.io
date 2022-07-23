# emacs에 $PATH 환경변수 인식 시키기


* Intro
  나는 우분투에서 이맥스를 즐겨쓰고 있다. 이맥스에서 [[https://golang.org][고 언어]]를 종종 쓰는데 이번에 노트북을 셋팅하면서 문제가 발생했다.
  일단 나는 이맥스에서 개발할 때 주로 [[https://github.com/emacs-lsp/lsp-mode][lsp-mode]]를 바탕으로 개발환경을 구성하는데, 기본적인 설정은 [[https://github.com/emacs-lsp/lsp-mode#installation][문서]]를 보고 구성하면 무리없이 따라할 수 있다. 이번 글에서는 가이드대로 다 따라했는데 이맥스에서 [[https://github.com/golang/tools/blob/master/gopls/README.md][gopls]]를 인식하지 못하던 문제에 대한 해결법을 기록하고자 한다.
* 문제 상황
  lsp-mode를 사용하면 각 언어에 해당하는 language server를 이용해서 문법 점검, 자동완성, 디버거 연동 등등 IDE에서 흔하게 지원해주는 기능을 사용할 수 있게 해준다. 고 언어의 경우에는 gopls를 이용하는데 설치 자체는 가이드를 참고하면 어렵지 않게 설치할 수 있다. 설치 후 이맥스에서 .go 파일을 열고 lsp-mode를 실행하면 아래와 같은 메세지를 보여준다.
  #+BEGIN_SRC 
The following servers support current file but do not have automatice installation configuration: go-ls go-bingo gopls You may find the installation instructions at httsp://github.com/emacs-lsp/lsp-mode/#supported-languages. Do you want open it?
  #+END_SRC
  *y* 를 입력하면 gopls 설치 가이드를 띄워준다. 하지만 나는 *이미 gopls를 설치한 상태이므로 이맥스에서 gopls가 설치된 경로를 인식하지 못한다고 판단, 해결법을 찾아보았다.*
* 해결법
  [[https://github.com/purcell/exec-path-from-shell][exec-path-from-shell]] 패키지를 이맥스에 설치하자. 이 패키지는 내 PC에 설정된 환경변수를 이맥스에서 인식할 수 있게 도와주는 패키지다.
  #+BEGIN_SRC
  M-x package-install exec-path-from-shell
  #+END_SRC
  설치 후 `.emacs` 파일에 아래의 내용을 추가해준다.
  #+BEGIN_SRC lisp
(when (memq window-system '(mac ns x))
  (exec-path-from-shell-initialize))
(exec-path-from-shell-copy-env "PATH")
  #+END_SRC
  emacs를 재시작 한 후 .go 파일에서 lsp 명령어를 입력하면 gopls를 정상적으로 인식하는 걸 확인할 수 있다. lsp를 항상 직접 켜주기 귀찮다면 [[https://github.com/emacs-lsp/lsp-mode#install-language-server][공식 문서]]나 [[https://arenzana.org/2019/12/emacs-go-mode-revisited/?utm_campaign%3DThe%2520Go%2520Gazette&utm_medium%3Demail&utm_source%3DRevue%2520newsletter][다른 가이드]]를 보고 따라해보자

