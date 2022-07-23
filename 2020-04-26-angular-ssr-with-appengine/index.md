# 구글 앱엔진에 ssr angular app 배포하기


* 들어가며
이 문서는 기본적으로 [[https://angular.io/guide/universal][앵귤러 공식 SSR 가이드]]를 따라하며 학습한 내용을 정리한 것입니다. SSR에 대해 궁금하시다면 [[https://velog.io/@zansol/%25ED%2599%2595%25EC%259D%25B8%25ED%2595%2598%25EA%25B8%25B0-%25EC%2584%259C%25EB%25B2%2584%25EC%2582%25AC%25EC%259D%25B4%25EB%2593%259C%25EB%25A0%258C%25EB%258D%2594%25EB%25A7%2581SSR-%25ED%2581%25B4%25EB%259D%25BC%25EC%259D%25B4%25EC%2596%25B8%25ED%258A%25B8%25EC%2582%25AC%25EC%259D%25B4%25EB%2593%259C%25EB%25A0%258C%25EB%258D%2594%25EB%25A7%2581CSR][이 링크]]를 참고하세요. 이 문서는 angular, appEngine을 사용할 줄 아는 사람을 대상으로 작성되었습니다.

* 프로젝트 생성
  우선 앵귤러 프로젝트를 하나 생성해봅시다. 이 예제는 angular9 버전을 기준으로 작성되었습니다.

  #+BEGIN_SRC bash
    ng new test-project
  #+END_SRC
* express-engine 추가
  생성된 프로젝트 내부에서 아래의 명령어를 실행합니다.
  #+BEGIN_SRC bash
    ng add @nguniversal/express-engine
  #+END_SRC

  명령어를 성공적으로 실행했다면 아래의 파일이 추가되었을 것입니다.
  + server.ts
  + src/app/app.server.module.ts
  + src/main.server.ts
  + tsconfig.server.json
  그리고 기존에 내용이 바뀐 파일들도 있습니다.
  + angular.json
  + package-lock.json
  + package.json
  + src/app/app-routing.module.ts
  + src/app/app.module.ts
  + src/main.ts

  당장은 파일 내용들을 신경쓸 필요는 없고 일단 실행부터 시켜보죠.
* local server 실행
  아래의 명령어를 이용해서 앵귤러 프로젝트를 실행해봅니다.
  #+BEGIN_SRC bash
    npm run dev:ssr
  #+END_SRC
  http://localhost:4200 에 접속해보면 이쁜 화면을 볼 수 있습니다.

* package.json 수정
  우리는 이 어플리케이션을 appEngine에 배포하는게 목적입니다. 즉, node express engine을 배포한다는 소리인데 그러기 위해선 ~package.json~ 파일을 수정해야 합니다. package.json의 정보를 아래처럼 수정해줍시다(참고로 필자는 nodejs를 잘 모릅니다).
  {{< highlight json "linenos=table,hl_lines=4 7,linenostart=1" >}}
  {
    "name": "test",
    "version": "0.0.0",
    "main": "main.js",
    "scripts": {
      "ng": "ng",
      "start": "node dist/test/server/main.js",
      "build": "ng build",
      "test": "ng test",
      "lint": "ng lint",
      "e2e": "ng e2e",
      "dev:ssr": "ng run test:serve-ssr",
      "serve:ssr": "node dist/test/server/main.js",
      "build:ssr": "ng build --prod && ng run test:server:production",
      "prerender": "ng run test:prerender"
    },
  .
  .
  .
  {{< / highlight >}}
  ~main~ 속성을 추가하고 scripts.start 속성의 내용을 바꿔주었습니다.
* build for deploy
  이제 배포를 하기 위한 빌드를 할 차례입니다. 아래의 명령어를 입력해보죠.
  #+BEGIN_src bash
    npm run build:ssr
  #+END_SRC
  빌드를 완료하면 ~dist~ 폴더 아래에 다음과 같은 파일들이 생성되어 있을것입니다..

  #+BEGIN_SRC tree
  dist/
  └── test
      ├── browser
      │   ├── 3rdpartylicenses.txt
      │   ├── favicon.ico
      │   ├── index.html
      │   ├── main-es2015.4442acaf2e469b2bab94.js
      │   ├── main-es5.4442acaf2e469b2bab94.js
      │   ├── polyfills-es2015.690002c25ea8557bb4b0.js
      │   ├── polyfills-es5.1fd9b76218eca8053895.js
      │   ├── runtime-es2015.1eba213af0b233498d9d.js
      │   ├── runtime-es5.1eba213af0b233498d9d.js
      │   └── styles.09e2c710755c8867a460.css
      └── server
          └── main.js
  #+END_SRC
  이제 앱엔진 배포를 하기 위해서 앵귤러 측에서 할 작업은 모두 끝났습니다. 다음은 appEngine 설정을 위한 ~app.yaml~ 파일을 작성해봅시다.
* app.yaml 생성 및 배포
  프로젝트 최상단에 ~app.yaml~ 파일을 생성합니다. 이 파일은 앱엔진에 배포를 하기 위한 파일이며 내용은 아래와 같이 작성해줍니다.
  #+BEGIN_SRC yaml
    runtime: nodejs10
  #+END_SRC
  내용은 간단합니다. 그냥 nodejs 환경을 사용하겠다는 의미입니다. 이제 이 파일과 이전 단계에서 빌드한 파일들을 기반으로 실제 *배포* 를 해봅시다.

  #+BEGIN_SRC bash
    gcloud app deploy
  #+END_SRC
  위 동작이 마무리 되면 ~gcloud app browse~ 명령어를 사용해 실제 앱엔진이 정상적으로 동작하는 걸 확인할 수 있습니다. 
* 마무리
  이 예제만 가지고는 ssr로 인해 얻는 이점이 잘 드러나지 않습니다. 하지만 이 후에 프로젝트가 점점 커지면서 초기에 구동할 스크립트가 무거워지거나, [[https://mygumi.tistory.com/24][dynamic open graph]]를 적용할 때 매우 유용하게 사용할 수 있습니다. 앵귤러에서 dynamic open graph를 구현할때는 [[https://dev.to/andreilm/dynamic-social-media-tags-with-angular-7-3a5][이 링크]]를 참고하세요.

