# emacs에서 tslint 이용하기


나는 angular6를 이용해 웹 프로젝트를 진행하는걸 선호하는데 툴은 emacs를 주로 쓰고있다(OS는 ubuntu). vscode를 쓸까 했는데 그냥 이맥스가 좋아서(이유는 없다) 이맥스에 꾸역꾸역 개발환경을 구축했다. [tide](https://github.com/ananthakumaran/tide)를 이용해서 환경을 구성했는데 [tslint](https://github.com/ananthakumaran/tide)가 작동하지 않아서 잠깐 했던 삽질을 여기에 짤막하게 기록한다.

## vscode는 그냥 되던데?

이게 내가 헤매게 된 가장 큰 이유다. vscode는 마켓플레이스에 그냥 tslint 패키지를 내려받으면 알아서 잘 동작한다. 개발자가 따로 뭐 설정하고 해줄 필요가 없다.(물론 `tslint.json`이 필요하지만 angular 프로젝트는 생성 시 지가 알아서 만들어준다.)

이맥스도 tide만 깔면 알아서 잘 될줄 알았는데 **그런거 없다.**

## 해결책은?

별거 없다. tslint를 설치하자.

``` bash
# Install the global CLI and its peer dependency
$ yarn global add tslint typescript
```

혹시 yarn이 설치되어 있지 않다면 아래 내용을 터미널에서 실행해보자.

```bash
$ curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
$ sudo apt-get update && sudo apt-get install yarn
```

## 한줄평

툴이 알아서 다 해줄거라는 생각은 버려라.

