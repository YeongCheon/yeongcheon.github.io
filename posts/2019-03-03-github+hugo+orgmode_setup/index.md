# hugo + org-mode + github를 이용해 블로그 셋팅하기


[[https://gohugo.io/][hugo]] + [[https://github.com][github]] + [[https://orgmode.org/][org-mode]]를 이용해 블로그를 운영하는 법을 알아보자. github에 hugo를 올리는 방법은 웹에 가이드가 이미 많이 있기 때문에 이 포스트에서는 org-mode를 이용해 hugo에 글을 어떻게 쓰는지를 중점적으로 설명한다.

* hugo를 적용하게 된 계기
** orgmode에 대한 호기심
  원래 =jekyll= + =github= 조합으로 블로그를 운영하고 있었다. github 블로그를 운영할 때 가장 대중적인 조합이고 나 초기 셋팅이 좀 번거로웠던거 빼곤 불만이 없었기에 그대로 사용하고 있었다.
  블로그에 포스트를 작성할 때 [[https://ko.wikipedia.org/wiki/%25EB%25A7%2588%25ED%2581%25AC%25EB%258B%25A4%25EC%259A%25B4][markdown]]을 이용해서 글을 작성했었는데 최근에 emacs의 킬러 콘텐츠로 불리는 [[https://orgmode.org/][org-mode]]에 관심이 가기 시작했다.
  책 한권 분량은 거뜬할 정도로 방대한 기능과 한번 익혀두면 html, markdown, LaTex, pdf 등 원하는 포맷으로 콘텐츠를 export 할 수 있다는게 상당히 매력적으로 보였다.
  그래서 재미삼아 한번 익혀볼 생각으로 일단은 기존에 운영(+방치+)하고 있던 블로그에 제일 먼저 적용해보기로 했다.
** jekyll에서 orgmode 적용 실패
  근데 생각보다 진행하는게 쉽지 않았다. 우선 내가 jekyll을 잘 다룰줄 몰랐고 관련 셋팅에 사용되는 ruby언어에 대해 아는 게 전혀 없었다. 
  여차저차해서 피씨에서 =jekyll serve= 명령어를 이용한 테스트에 성공하고 github에 업로드를 했더니 github에선 동작을 하지 않아서 바로 때려쳤다.
  사실 예전부터 hugo로 갈아타고 싶은 마음이 있었기에 미련없이 jekyll은 버리기로 했다. jekyll을 버리고 hugo로 갈아탄 *개인적인* 이유는 다음과 같다.

  + jekyll은 ruby로 만들어졌다. 그래서 초기 셋팅때 =gem= 같은 명령어를 쓰거나 =gemfile= 관리 등 번거로운 작업이 많았는데 영 익숙치가 않았다(나는 ruby를 전혀 모른다).
	그러다가 알게 된 hugo는 go로 작성되어 있었고 단지 이 이유 하나 때문에 hugo가 마음에 들어버렸다. 물론 그렇다고 내가 블로그 셋팅을 하는데 go의 소스코드를 볼 일은 전혀 없었다.
  + *내 피씨에서 빌드해서 올리고 싶다.*
	jekyll은 일단 기본적으로 작성한 글을 github에 push하면 github 서버에서 빌드된 다음 배포된다. 이게 처음에는 상당히 매력적으로 보였는데 org-mode 관련 셋팅을 하다가 고생을 많이했다.
	내 피씨에서 관련 플러그인을 설치해서 빌드를 하면 정상 동작하던 글이 github 서버에서 빌드되서 배포되면 제대로 동작하지 않는 문제가 있었는데 해결법을 찾기가 너무 힘들었다.(+결국 못찾았다+)
	이럴바엔 그냥 내 PC에서 빌드된 버전을 그대로 서비스하는 방법이 낫겠다 싶었고 그렇다면 빌드속도도 빠르고 트러블 대응이 쉬운 hugo가 낫다고 판단했다. 대응이 쉽다고 생각한 이유는 내가 ruby는 모르지만 go는 조금 알고 있기 때문이다.
  + +그냥+

* hugo + org-mode 설정하기
  org-mode로 hugo에 손쉽게 글을 등록하기 위해서는 [[https://github.com/kaushalmodi/ox-hugo][ox-hugo]]를 이맥스에 설치하는게 좋다. =org= 확장자 파일을 =md= 파일로 export하는 기능이 있는데 기존에 있는 export 기능과 달리 hugo에 특화된 기능이 있다. 
  자세한 설명은 공식 가이드에서 확인하도록 하고 여기서는 예제를 통해서 알아보자.

  간단하게 각 단계를 나열하면 아래와 같다.
  + org 확장자 파일을 만들어 글을 작성한다.
  + org 확장자를 md 파일로 export 한다.(=C-c C-e H H=)
  
  별다른 설정 없이 [[https://gohugo.io/getting-started/quick-start/][hugo 공식 가이드]]를 따랐다면 아마 디렉토리 구조는 아래와 비슷할 것이다.

  #+BEGIN_SRC text

.
├── archetypes
│   └── default.md
├── config.toml
├── content
│   └── posts
├── data
├── layouts
├── public
│   ├── 404.html
│   ├── avatar.jpg
│   ├── categories
│   ├── css
│   ├── dist
│   ├── images
│   ├── index.html
│   ├── index.xml
│   ├── org
│   ├── page
│   ├── posts
│   ├── sitemap.xml
│   └── tags
├── resources
│   └── _gen
├── static
├── syntax.css
└── themes

  #+END_SRC

** org파일 작성
  보통은 위와 같은 구조에서 =content/posts= 디렉토리 아래에 =.md= 형식의 글을 등록한다. 
  하지만 우리 입장에서 md파일은 단순히 org 파일을 export한 결과에 불과하므로 md파일과 org 파일을 각각 따로 관리해 줄 필요가 있다.
  나같은 경우에는 =content= 디렉토리 아래에 =org= 디렉토리를 만든 후 그 안에서 org 파일들을 관리한다. 이제 org파일 샘플을 보도록 하자.

  #+BEGIN_SRC org
#+HUGO_BASE_DIR: ../../
#+HUGO_SECTION: ./posts

#+HUGO_WEIGHT: auto
#+HUGO_AUTO_SET_LASTMOD: t
#+HUGO_TAGS: sometag01 sometag02

#+TITLE: HELLO WORLD
#+AUTHOR: yeongcheon
#+DATE: "2019-03-03 19:00:00 +0900"

HELLO WORLD
  #+END_SRC

** md파일로 내보내기
  =#+HUGO= 로 시작하는 라인은 ox-hugo에서 사용하는 특별한 설정이다. 다른 =#+TITLE= 같은 라인은 평범한 orgmode용 메타데이터라고 생각하면 된다.
  이제 위의 파일을 export하면 org 파일과 동일한 파일명에 확장자만 .md인 파일이 =content/posts= 에 생길것이다.
  이맥스 단축키는 =C-c C-e H H= 다.(+단축키 주제에 길다+)
  파일 내용은 아래와 같다.

  #+BEGIN_SRC markdown
+++
title = "jackson을 이용해 json 변환 시 JPA entity의 id값만 추출하기"
author = ["yeongcheon"]
date = 2019-03-01T19:00:00+09:00
lastmod = 2019-03-03T19:00:00+09:00
tags = ["sometag01", "sometag02"]
draft = false
+++

HELLO WORLD
  #+END_SRC

org mode에서 작성한 정보가 이쁘게 hugo용 markdown 파일로 변환되는걸 볼 수 있다. 
이제 이상태로 github에 push를 해주면 내 블로그에 글이 등록된다. 

github deploy 방법은 [[https://gohugo.io/hosting-and-deployment/hosting-on-github/][이 링크]]를 참고하자

* 마무리
  사실 hugo와 org mode 둘 다 써본지 3일도 안됐다. 물론 제대로 된 글을 써본적도 사실 없지만 markdown에서 org로 갈아타면서 가장 크게 느낀점은 *포맷이 바뀐다고 글 내용이 좋아지지는 않는구나..* 였다.
  명필은 붓을 가리지 않는다는 말이 괜히 생긴게 아니다. markdown이니 org니 형식에만 치중한 나머지 정작 글 본문에는 신경을 쓰지 못하게 되어버린 듯 하다(+쓴다고 크게 달라지진 않는다+).
  앞으로 꾸준히 블로그에 글을 올리며 글 연습도 하고 orgmode도 익힐 계획이지만 잘 될런지...

