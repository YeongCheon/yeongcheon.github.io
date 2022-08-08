# 이맥스에서 특정 명령어 단축키로 지정하기


이맥스에서 단축키를 직접 설정하는 법을 알아봅시다. 아는 게 없어서 [[http://ergoemacs.org/emacs/keyboard_shortcuts.html][여기]] [[http://emacslife.com/read-lisp-tweak-emacs/beginner-3-make-things-more-convenient.html][저기]] 참고를 많이 했어요.
* 기본 사용법
  제일 단순한 예제를 먼저 살펴봅시다.
  
  #+BEGIN_SRC lisp
  (global-set-key (kbd "M-a") 'backward-char) ; Alt+a

  (global-set-key (kbd "C-a") 'backward-char) ; Ctrl+a

  (global-set-key (kbd "C-c t") 'backward-char) ; Ctrl+c t

  (global-set-key (kbd "<f7> <f8>") 'whitespace-mode)    ; F7 F8
  #+END_SRC

  딱히 설명이 필요 없을 정도로 간단하지만 굳이 첨언을 하자면 모든 buffer에서 사용할 수 있는 단축키를 지정할때는 위와 같이 사용합니다. 

  명령어가 아니라 자주 쓰는 텍스트를 등록해서 사용할 수도 있어요. 아래는 이메일 주소를 입력하는 단축키 등록 예제입니다.

  #+BEGIN_SRC lisp
   (global-set-key (kbd "M-0") "kyc1682@gmail.com")
  #+END_SRC

  기본 사용법: ~(global-set-key (kbd "원하는_단축키") `원하는_명령어)~
* 특정 mode에서만 동작하는 단축키 지정
  이맥스를 쓰다보면 특정 모드에서만 사용하고 싶은 단축키가 생길 수 있습니다.. 예를 들어 [[https://github.com/dominikh/go-mode.el][go-mode]]에서 *F8* 키를 *dap-breakpoint-toggle* 명령 단축키로 지정하고 싶다면 아래와 같이 입력하면 됩니다.

  #+BEGIN_SRC lisp
(defun my-go-mode-hook ()
  (local-set-key (kbd "<f8>") 'dap-breakpoint-toggle)
)
(add-hook 'go-mode-hook 'my-go-mode-hook)
  #+END_SRC

  위 코드에는 [[https://www.gnu.org/software/emacs/manual/html_node/emacs/Hooks.html][hook]]이라는 개념이 등장하는데 hook에 대해서는 다음 시간에 알아도록 하고, 위의 코드를 간단하게 얘기하자면 go-mode를 실행하면 미리 선언해놓은 이름이 my-go-mode-hook인 LIST(위 코드에선 단순히 함수라고 생각해도 무방하다)를 추가로 실행하겠다는 의미이다.

