# 이맥스 폰트, 세벌식 셋팅하기



아래의 내용을 =.emacs= 파일 안에 복붙해주자. 세벌식 버전은 최종이 아닌 390버전이다.

#+begin_src lisp
(set-fontset-font t 'hangul (font-spec :family "D2Coding"))

(set-language-environment "Korean")
(prefer-coding-system 'utf-8)
(setq default-input-method "korean-hangul390")
(setq default-korean-keyboard "390")
(global-set-key (kbd "<S-kana>") 'toggle-input-method)
#+end_src

