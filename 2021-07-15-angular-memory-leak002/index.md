# angular universal 메모리 누수 개선하기 - 002


앵귤러 기반 웹사이트를 개발하면서 발생한 메모리 누수의 다양한 원인들과 그 해결법을 기록합니다.

* 들어가며

  이번 문서에서는 [[https://rxjs.dev/][rxjs]] 쓰다보면 흔하게 접할 수 있는 메모리 누수에 대해 알아보고 [[https://eslint.org/][eslint]]와 [[https://github.com/ngneat/until-destroy][until-destroy]]를 이용해 메모리 누수를 원천적으로 막을 수 있는 방법에 대해 알아봅시다. 이 문서는 rxjs, eslint에 대해 기본적인 이해가 있다는 것을 전제로 작성되었습니다.

* Bad Case

  다음은 흔하게 작성하는 앵귤러 컴포넌트 코드입니다.
 
  #+BEGIN_SRC typescript
@Component({
  selector: 'app-a',
  templateUrl: './a.component.html',
  styleUrls: ['./a.component.css']
})
export class AComponent implements OnInit {
    observable$ = of(1);

    ngOnInit() {
      this.observable$.subscribe(value => console.log(value));
    }
}
   #+END_SRC
  
  위의 코드는 치명적인 메모리 누수 문제를 포함하고 있습니다. A 컴포넌트는 생성 시 내부에 선언된 ~observable~ 객체를 구독하는데 이 구독 객체는 *컴포넌트가 제거되도 사라지지 않습니다.* 개발자는 A 컴포넌트가 제거되면 거기에 종속된 각종 구독 객체도 자연스럽게 소멸되길 기대하지만 *가비지 컬렉터는 Subscription을 수집하지 못합니다.* 따라서 만약 A 컴포넌트를 여러번 생성하고 제거하기를 반복하면 메모리에는 계속 필요없는 Subscription 객체가 남아있어 결국에는 ~heap out of memory~ 에러를 발생시키게 됩니다. 

  이 코드를 memory safe하게 수정한 코드는 다음과 같습니다.

  #+BEGIN_SRC typescript
		@Component({
		  selector: 'app-a',
		  templateUrl: './a.component.html',
		  styleUrls: ['./a.component.css']
		})
		export class AComponent implements OnInit, OnDestroy {
			observable$ = of(1);
			sub: Subscription

			ngOnInit() {
				this.subscription = this.observable$.subscribe(value => console.log(value));
			}

			ngOnDestroy() {
				this.subscription.unsubscribe(); // 구독 객체를 꼭 명시적으로 삭제해주어야 합니다.
			}
		}
  #+END_SRC

  컴포넌트 생성 시 생성한 Observable 구독 객체인 Subscription을 컴포넌트 제거 시 구독 해제를 명시적으로 해줌으로써 메모리 누수를 방지할 수 있습니다.

  하지만 이런 방식은 개발자가 실수를 하기 너무 쉽고, 실수를 알아차리기도 쉽지 않습니다. 따라서 eslint와 until-destroy를 이용해서 unsubscribe를 강제하는 코드를 작성하는 법을 알아보고자 합니다.

* until-destroy 사용

  until-destory는 subscriptions의 생명주기를 컴포넌트와 동일하게 맞추는 데 도움을 주는 라이브러리입니다.

  아래의 해당 라이브러리의 공식 예제입니다.
  
  #+BEGIN_SRC typescript
@UntilDestroy()
@Component({})
export class InboxComponent {
    ngOnInit() {
      interval(1000)
        .pipe(untilDestroyed(this))
        .subscribe();
    }
}
  #+END_SRC

  ~@UntilDestory~ 어노테이션과 ~untilDestroyed~ 함수를 사용하여 크게 힘들이지 않고 InboxComponent를 제거함과 동시에 내부에서 사용중이던 구독 객체를 메모리에서 해제시킬 수 있습니다. 하지만 이 코드조차도 개발자가 실수로 누락시킬 가능성이 있습니다. 따라서 *subscribe를 할때마다 반드시 untilDestoryed를 사용하도록 eslint로 강제시키는 방법이 필요합니다.* [[https://github.com/ngneat/until-destroy#readme][공식 문서]]에서는 [[https://github.com/cartant/eslint-plugin-rxjs-angular/blob/main/docs/rules/prefer-takeuntil.md#options][perfer-takeuntil]], [[https://github.com/cartant/eslint-plugin-rxjs/blob/main/docs/rules/no-unsafe-takeuntil.md#options][no-unsafe-takeuntil]] 두가지 룰을 언급하고 있습니다.

* eslint 적용하기
** 플러그인 다운로드
   
  우선 npm 명령어를 이용해 해당 rule을 다운받습니다.
  
  #+BEGIN_SRC shell
npm i eslint-plugin-rxjs eslint-plugin-rxjs-angular --save-dev
  #+END_SRC
  
** eslint rule 적용

   ~.eslintrc.json~ 파일의 ~rules~ 필드에 아래와 같이 ~rxjs-angular/prefer-takeuntil~, ~rxjs/no-unsafe-takeuntil~ 룰을 적용시켜줍니다.
  #+BEGIN_SRC json
{
  "overrides": [
    {
      ...,
      "rules": {
        "rxjs-angular/prefer-takeuntil": [
          "error",
          {
            "alias": ["untilDestroyed"],
            "checkComplete": false,
            "checkDecorators": ["Component"],
            "checkDestroy": false
          }
        ],
        "rxjs/no-unsafe-takeuntil": [
          "error",
          {
            "alias": ["untilDestroyed"]
          }
        ],
        ...
      }
    }
}
  #+END_SRC

  각 룰이 의미하는 내용은 다음과 같습니다.
  
  + [[https://github.com/cartant/eslint-plugin-rxjs-angular/blob/main/docs/rules/prefer-takeuntil.md#use-takeuntil-and--ngondestroy-prefer-takeuntil][prefer-takeuntil]]: 컴포넌트 내부에서 subscribe 함수를 호출할 때 takeuntil(untilDestroyed) 함수를 호출하지 않을 경우 에러를 발생시키겠다는 의미힙니다. rxjs 기본 연산자인 takeutil이 기본값이지만 alias 필드에 다른 대체 연산자를 명시할 수 있습니다(이 문서에서는 untilDestroyed).
  + [[https://github.com/cartant/eslint-plugin-rxjs/blob/main/docs/rules/no-unsafe-takeuntil.md#avoid-takeuntil-subscription-leaks-no-unsafe-takeuntil][no-unsafe-takeuntil]]: pipe 함수 마지막에 takeutil(untilDestroyed)을 사용하지 않을 경우 에러를  발생시키겠다는 의미입니다.


  eslint 린트 적용이 마무리 된다면 이제 실수로 구독 객체를 unsubscribe하지 않아 메모리 누수가 발생하는 문제는 피할 수 있습니다.

* 마무리

  Observable 관련 메모리 누수는 앵귤러 기반 웹 프로젝트 진행 시 가장 흔하게 만날 수 있는 문제입니다. 메모리 누수에는 다양한 원인이 존재하며 이번 문제 또한 그 중 하나일 뿐이기에 이 문제를 해결했다고 메모리 누수는 발생하지 않을것이라고 방심해서는 절대 안됩니다. 개선 후에도 다른데서 메모리 누수가 발생하는지 꾸준한 모니터링과 점검이 필요합니다.

