# angular universal 메모리 누수 개선하기 - 003


앵귤러 기반 웹사이트를 개발하면서 발생한 메모리 누수의 다양한 원인들과 그 해결법을 기록합니다.

* 들어가며

  이번 문서에서는 Chrome Inspect를 이용해서 서버 사이드 렌더링 시 발생하는 메모리 누수를 확인하고 이를 수정하는 방법에 대해 알아봅니다. 이 문서는 [[https://angular.kr/guide/universal][Angular Universal]]에 대해 기본적인 이해가 있다는 것을 전제로 작성되었습니다.

* 앱 작성

  직접 밑바닥부터 코드를 작성해가며 문제를 재현해봅시다.

** 앵귤러 앱 생성

   다음 명령어를 이용해 앵귤러 앱을 생성해줍니다.

   #+BEGIN_SRC shell
   ng new sample
   #+END_SRC

** universal 모듈 추가

   다음 명령어를 이용해 universal 관련 모듈을 추가해줍니다.

   #+BEGIN_SRC shell
   ng add @nguniversal/express-engine
   #+END_SRC

** 앱 동작 확인

   다음 명령어를 이용해 앵귤러 서버를 실행 후 앱이 동작하는지 확인합니다.

   #+BEGIN_SRC shell
   npm run dev:ssr
   #+END_SRC

** Code 작성

   *문제가 발생하는* 코드를 작성해봅시다. ~app.component.ts~ 파일의 ~ngOnInit~ 내부에 다음과 같은 코드를 작성해줍니다.

   #+BEGIN_SRC typescript

import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.scss'],
})
export class AppComponent {
  title = 'sample';

  ngOnInit() {
    setInterval(() => {
      const promise = new Promise<string>((resolve, reject) => {
        const wrongPromise = new Promise<string>((resolve, reject) => { });

        setInterval(() => {
          wrongPromise.then(() => {
            console.log('never execute');
          });
        }, 100);
      });

      promise.then((res) => {
        console.log(res);
      });
    }, 3000);
  }
}
   #+END_SRC

   컴포넌트 생성 시 100ms마다 절대로 실행되지 않는 Promise를 생성하는 코드입니다. 생성된 Promise는 컴포넌트가 Destroy되어도 계속 메모리에 상주하여 성능에 문제를 일으키케 됩니다. 

* 크롬 개발자 도구를 이용해 메모리 누수 확인

** 메모리 스냅샷 생성

*** 클라이언트 환경에서 스냅샷 생성

   위에서 작성한 프로그램을 ~ng serve~ 명령어를 이용해 실행 후 크롬 브라우저를 이용해 해당 페이지에 접근, ~F12~ 키를 눌러 개발자 도구를 실행한 뒤 ~Memory~ 탭에 접급합니다.

   [[/images/Screenshot from 2021-08-08 16-53-15.png]]

   ~Take snapshot~ 버튼을 3~5초 간격으로 눌러서 시간별 메모리 스냅샷을 기록합니다. 그 후 'Heap Snapshots' 목록에서 가장 최근에 생성된 스냅샷을 클릭 후  ~(closure)~ 리스트를 열어보면 다음과 같이 여러번 생성되어 메모리에 남아있는 Promise를 확인할 수 있습니다.

	[[/images/Screenshot from 2021-08-08 17-20-40.png]]

*** 서버 환경에서 스냅샷 생성

	브라우저가 아닌 서버에서 메모리 누수가 있는지 확인하기 위해선 우선 universal용으로 빌드를 새로 해야합니다. 
	#+BEGIN_SRC shell
	  ng build && ng run <app-name>:server --configuration development
	#+END_SRC
	명령어 입력 시 ~<app-name>~ 자리에는 사용자가 ng new 명령어 실행 시 입력한 어플리케이션 이름을 입력해야 합니다. 그리고 ~--configuration development~ 옵션을 추가한 이유는 sourceMap을 활성화 하기 위해서입니다. 기본적으로 앵귤러는 서버 빌드 시 production 설정으로 빌드를 수행하기 때문에 빌드 결과값이 minify되어 사람이 읽기 어려운 형태로 출력됩니다. 따라서 개발자가 작성한 소스코드를 크롬 개발자 도구에서 바로 연동해서 볼 수 있도록 sourceMap 옵션이 활성화 되어있는 development 설정값을 기본으로 셋팅하여 빌드를 수행하여야 합니다.

	빌드가 완료됐으면 ~package.json~ 파일을 열어 ~serve:ssr~ 명령어에 ~--inspect~ 인자를 추가해야 합니다.

	#+BEGIN_SRC json
	"scripts": {
		"ng": "ng",
		"start": "ng serve",
		"build": "ng build",
		"watch": "ng build --watch --configuration development",
		"test": "ng test",
		"dev:ssr": "ng run sample:serve-ssr",
		"serve:ssr": "node --inspect dist/sample/server/main.js", // 여기에 --inspect 인자값을 추가해야 합니다.
		"build:ssr": "ng build && ng run sample:server",
		"prerender": "ng run sample:prerender"
	},
	...
	#+END_SRC JSON

	이 후 ~npm run serve:ssr~ 명령어를 이용해 서버를 실행한 후 크롬의 주소창에 ~chrome://inspect~ 를 입력하면 다음과 같은 화면을 볼 수 있습니다.

	 [[/images/Screenshot from 2021-08-08 17-37-39.jpg]]

	Remote Target 하단의 main.js가 조금 전에 실행시킨 서버입니다. inspect를 누르면 브라우저 환경에서 Memory 탭에 접근했을때와 동일한 화면이 나타납니다.

	 [[/images/Screenshot from 2021-08-08 20-36-51.png]]

	이제 크롬에서 http://localhost:4000 링크에 접근 후 위의 Devtool에서 Take snapshoot을 누르면 서버의 메모리 상태를 캡쳐할 수 있습니다. 다만 현재 이 예제의 소스코드는 setInterval이 무한히 실행되기 때문에 서버가 응답을 하지는 않습니다. 하지만 응답을 기다리는 동안 Devltool에서 일정 간격으로 서버 메모리 상태를 캡쳐하여 closure 목록을 열어보면 다음과 유사한 모습을 확인할 수 있습니다.

	[[/images/Screenshot from 2021-08-08 20-42-32.png]]

	위의 이미지처럼 동일한 코드라인이 반복된다면 메모리 누수를 의심해 볼 수 있습니다.

	실제 개발을 하다보면 클라이언트에서 응답을 받았음에도 서버에서는 위와 같이 계속 메모리에 값이 쌓여 결국 서버가 죽어버리는 상황이 발생하기도 합니다.

* 마무리

  메모리 누수는 다양한 원인으로 인해 발생할 수 있습니다. 하지만 위에서 설명한 내용대로 테스트를 진행하면 메모리 누수의 원인이 무엇이든 어디서 누수가 발생하는지 쉽게 파악할 수 있습니다. 만약 개발시에는 문제없이 동작하는 코드가 실제 배포 후 일정 주기로, 또는 이유없이 종종 서버가 죽어버리는 경우엔 메모리 누수를 의심해보고 테스트를 꼼꼼히 해보시는걸 권장합니다.

