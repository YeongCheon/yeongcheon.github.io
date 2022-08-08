# emacs 명령어 목록


# emacs 명령어 목록

> 이맥스를 쓰면서 개인적으로 유용하게 쓰고있는 명령어 리스트, 대부분 `M-x`를 누른 후 실행한다.



## common
* `grep` : grep을 이용한 문자열 검색 (`lgrep`, `rgrep`도 있다.). [위키링크](https://www.emacswiki.org/emacs/GrepMode)
* `find-name-dired` : 특정 폴더 아래에서 지정된 파일이름 패턴과 동일한 목록을 뽑아 dired모드로 출력해주는 명령어. 폴더 아래의 모든 파일에서 `replace` 명령어를 수행할 때 유용하다. [링크](https://www.gnu.org/software/emacs/manual/html_node/emacs/Dired-and-Find.html)

  [특정폴더 아래의 모든 파일(하위폴더 포함)에서 replace를 실행하는 방법](https://stackoverflow.com/a/271136/5961346)
  
  1. `M-x find-name-dired` 명령어 실행 후 폴더 지정, 파일패턴 지정
  2. `t`를 눌러서 dired 모드에 있는 모든 목록을 선택
  3. `Q`를 눌러서 `Query-Replace in Files...` 명령 실행.
  4. `y`, `n` 또는 `!`를 눌러서 `Query-Replace in Files...` 명령 진행(`!`는 매치되는 모든 문자열을 치환하겠다는 의미이다(all yes)).
  5. `C-x s`를 눌러서 변경내용 저장

