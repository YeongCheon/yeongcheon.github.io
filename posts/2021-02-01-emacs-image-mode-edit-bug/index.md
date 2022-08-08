# emacs에서 svg 수정 안되는 문제 해결하기


이맥스에서 svg 파일을 열면 기본적으로 image-mode가 활성화 된 상태로 파일이 열립니다. 이 때 버퍼는 svg 코드가 아니라 렌더링 된 이미지를 보여줍니다. 이미지가 아니라 코드를 보고싶을 경우엔 ~C-c C-c~ 단축키를 이용해서 버퍼 상태를 전환해야 합니다. 대부분의 코드 보기 상태에서 별도의 조치 없이 바로 코드 작업을 할 수 있지만 간혹 코드 수정이 안되는 경우가 있습니다. 원인이야 다양할 수 있지만 저같은 경우엔 [[https://github.com/editorconfig/editorconfig-emacs][editorconfig-mode]]가 문제였습니다. 아래는 제 ~.emacs~ 파일의 일부입니다. editorconfig-mode가 기본적으로 활성화 되도록 설정되어 있습니다.

#+BEGIN_SRC lisp
  (editorconfig-mode 1)
#+END_SRC

위의 코드를 제거하거나 주석처리를 한 뒤 이맥스를 재시작하면 이제 svg 파일을 정상적으로 수정할 수 있습니다.

