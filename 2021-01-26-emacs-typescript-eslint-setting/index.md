# emacs에서 Typescript 개발환경 구축하기


[[https://www.typescriptlang.org/][Typescript]]는 마이크로소프트(이하 MS)의 주도로 개발이 되고 있기 때문인지 Vscode에서 아주 편안하게 개발이 가능합니다. 사실 이번 글에서 주로 사용할 [[https://emacs-lsp.github.io/lsp-mode/page/installation/][lsp-mode]]도 MS에서 개발한 LSP에 기반하여 작성된 모드입니다. 이 외에도 [[https://emacs-lsp.github.io/lsp-mode/page/lsp-css/][css languag server]] 등등 많은 부분을 MS에서 작성한 프로그램에 의존하고 있는데 이쯤되면 그냥 속편하게 Vscode를 쓰는게 낫지 않을까 싶지만 몇년동안 정들었던 이맥스를 포기하기엔 아쉬움이 많이 남기에 Typescript 개발환경 구축 방법을 기록하고자 합니다. 이 글은 다음과 같은 내용을 기준으로 작성되었습니다.
+ emacs 27.1, ubuntu 20.04 LTS 버전을 기준으로 작성되었습니다.
+ npm이 설치되어 있다고 가정합니다.
+ lsp-mode는 이미 설치되어 있다고 가정합니다.
+ [[https://www.flycheck.org/en/latest/][flycheck]]는 이미 설치되어 있다고 가정합니다.
+ [[https://company-mode.github.io/][company-mode]]는 이미 설치되어 있다고 가정합니다.


이번 글도 마찬가지로 개인 기록이 목적이기에 [[https://emacs-lsp.github.io/lsp-mode/page/installation/][공식 가이드]]를 보시는걸 좀 더 추천드립니다.

* typescript-language-server 설치
npm 명령어를 이용해서 typescript-language-server, typescript 패키지를 설치합니다. 명령어 수행 중 권한 관련 에러가 발생한다면 sudo 권한으로 설치를 진행합니다([[https://emacs-lsp.github.io/lsp-mode/page/lsp-typescript/][공식 가이드 참고]]).
#+BEGIN_SRC bash
npm i -g typescript-language-server; npm i -g typescript
#+END_SRC

* eslint 설치
[[https://www.npmjs.com/package/tslint][tslint]]는 deprecated 되었기 때문에 [[https://eslint.org/][eslint]]를 사용해야 합니다. 이맥스에서 M-x(알트-x)를 누른 후 ~lsp-install-server~ 를 입력 후 ~eslint~ 를 입력하고 엔터를 누릅니다([[https://emacs-lsp.github.io/lsp-mode/page/lsp-eslint/][공식 가이드 참고]]).

* .emacs 구성
필요한 플러그인은 모두 설치했으니 이제 ~.emacs~ 파일을 구성해봅시다. 각 코드의 역할은 주석으로 표시해놓았습니다.
#+BEGIN_SRC lisp
(require 'lsp-mode)

;; typescript-mode가 활성화되면 lsp-mode도 활성화합니다.
(use-package lsp-mode
  :hook (typescript-mode . lsp)
  :commands lsp)

;; typescript-mode가 활성화되면 수행할 함수를 정의합니다.
(defun setup-typescript-mode ()
  (interactive)
  (flycheck-mode +1)
  (setq flycheck-check-syntax-automatically '(save mode-enabled))

  ;; M-.(jump to definition) 명령어를 통해 이동한 커서 위치를 이전으로 되돌리는 명령어입니다.
  (local-set-key (kbd "M-*") 'pop-tag-mark) 
  ;; 디버깅용 breakpoint를 설정/해제하는 토글 단축키입니다.
  (local-set-key (kbd "<f8>") 'dap-breakpoint-toggle)

  ;; 저장하기 이전에 eslint를 이용해서 코드 formatting을 진행합니다. 
  ;; 어떤 설정이 문제인지는 몰라도 테스트 결과 typescript는 lsp-format-buffer가 올바르게 동작하지 않기 때문에 
  ;; lsp-eslint-fix-all 명령어로 대체했습니다.
  (add-hook 'before-save-hook 'lsp-eslint-fix-all) 

  ;; company-mode를 활성화합니다.
  (company-mode +1)
)

;; typescript-mode가 활성화되면 위에서 선언한 setup-typescript-mode 함수가 실행됩니다.
(add-hook 'typescript-mode-hook #'setup-typescript-mode)

#+END_SRC

* 마무리
위의 과정이 모두 마무리 되었다면 이맥스를 재시작 후 *.ts 파일에 접근하면 lsp를 실행 여부를 묻는데 이 때 yes를 입력하면 이 후 항상 lsp가 실행됩니다. 이 때 해당 프로젝트에 .eslintrc.json이나 tsconfig.json같은 설정파일이 존재해야 합니다.

