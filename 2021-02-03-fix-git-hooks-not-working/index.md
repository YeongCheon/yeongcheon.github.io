# git hooks 안되는 문제 해결하기


* What
필자는 프로젝트에서 commit을 하기 전에 [[https://ko.wikipedia.org/wiki/%EB%A6%B0%ED%8A%B8_(%EC%86%8C%ED%94%84%ED%8A%B8%EC%9B%A8%EC%96%B4)][lint]] 명령어를 이용해서 코드 스타일을 관리하고 있습니다. 초기에는 commit을 하기 전에 직접 수동으로 lint를 실행해서 코드를 점검해왔지만 아무래도 사람이 하는지라 종종 까먹고 lint 실행을 까먹는 경우가 발생하였습니다. 이 후 이 동작들을 자동화 하기 위해서 [[https://git-scm.com/book/ko/v2/Git%EB%A7%9E%EC%B6%A4-Git-Hooks][git hooks]]의 pre-commit을 이용하여 자동화를 시켜놨는데 집에서 사용하는 랩탑에서는 pre-commit이 동작하지 않는 문제가 발생하였습니다. 이를 해결하기 위해 열심히 구글링을 해봤지만 답이 나오지 않아 직접 이것저걸 삽질을 진행하다가 문제를 해결하여 이를 기록하고자 합니다.

* config가 문제다.
결론부터 얘기하자면 ~$HOME/.gitconfig~ 파일이 문제였습니다. git은 기본적으로 각 프로젝트 별로 존재하는 config 파일(기본경로: ~$PROJECT_DIR/.git/config~)을 1순위로 참고하여 동작합니다. 그리고 2순위로 ~$HOME~ 폴더에 있는 ~.gitconfig~ 를 참고합니다. 이 config 파일에는 git hook을 실행하기 위해서 참조하는 폴더의 위치를 [core] 아래에 기록할 수 있는데(기본값: ~$GIT_DIR/hooks~) 아래와 비슷한 모양입니다.

#+BEGIN_SRC
[core]
     hooks = /WRONG_DIR
#+END_SRC

이런 식으로 config 파일이 작성되어 있을 경우엔 *hooks 값을 지우면 의도한대로  ~$PROJECT_DIR/.git/hooks~ 폴더 위치를 참고하여 git hook이 동작하게 됩니다.* 다만 ~$HOME/.gitconfig~ 파일을 수정할 경우엔 다른 git 프로젝트에 영향이 갈 수 있으므로 주의해야 합니다.

