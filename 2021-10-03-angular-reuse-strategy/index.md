# angular route reuse strategy


* Intro
웹앱을 개발하다보면 list->detail 패턴의 구조를 흔하게 작성하게 됩니다. list-detail 패턴이란 목록(list)에서 원하는 컨텐츠를 탐색 후 해당 컨텐츠의 상세한 정보를 보는 화면(detail)로 이동하는 방식을 말합니다. 이 때 detail 화면에서 *뒤로 가기를 이용해 목록 화면으로 돌아올 경우 Angular는 해당 화면을 구성하는 컴포넌트를 처음부터 다시 생성(~ngOnInit~ 실행)합니다.* 이럴 경우엔 다음과 같은 문제가 발생합니다.

- 컴포넌트를 다시 생성하기 때문에 성능상 좋지 못합니다.
- 컴포넌트를 다시 그리기 때문에 로딩 화면이 보여지거나 스크롤 위치를 기억하지 등 유저 경험에 악영향을 미칩니다.
- API 서버와 통신을 통해 목록을 불러올 경우에 불필요한 네트워크 통신을 수행하게 됩니다.

뒤로 가기를 이용해 다시 이전 페이지에 접근할 경우엔 이미 이전에 생성해 둔 페이지를 다시 사용할 수 있다면 위에서 나열한 문제를 모두 해결할 수 있습니다. Angular에서는 [[https://angular.io/api/router/RouteReuseStrategy][RouteReuseStrategy]]를 이용해 재사용 기능을 구현할 수 있습니다.

* RouteReuseStrategy 동작 방식
실제 코드를 작성하기 전에 RouteReuseStrategy 인터페이스를 살펴봅시다.

** 구성과 역할
RouteReuseStrategy는 총 5개의 함수로 이루어져 있습니다.

각 함수가 호출되는 순서는 아래 나열되는 순서와 동일합니다.

1. *shouldReuseRoute*: route간 이동이 발생할 때마다 동작합니다. 이 메서드가 true를 반환하면 route 이동은 발생하지 않습니다. false를 반환할 경우 기존 Angular Life Cycle이 그대로 동작합니다.
2. *shouldAttach*: 이동할 페이지(detail 화면)를 ~store~ 함수를 통해 저장한 내역에서 불러올지 여부를 반환하는 함수입니다. true를 반환하면 ~retrieve~ 함수를 호출하여 화면을 불러옵니다.
3. *shouldDetach*: 다른 페이지로 이동하기 전에 현재 페이지를 나중에 재사용 할지 여부를 반환하는 함수입니다. true를 반환하면 ~store~ 함수를 호출하여 화면을 저장합니다.
4. *store*: ~shouldDetach~ 함수의 결과값이 true일 경우 현재 보고 있는 화면(list 화면)을 저장하는 함수입니다.
5. *retrieve*: ~shouldAttach~ 함수의 반환값이 true일 경우 ~store~ 함수를 통해 저장한 내역에서 불러옵니다. 

* 구현
** 시나리오
총 3개의 URL(~/~, ~/list~, ~/list/:itemId~)이 있는 웹앱을 구현하고자 합니다. 이 중 ~/list~ 페이지만 재사용하고 나머지는 항상 새로 init을 하고자 합니다.

위와 같은 시나리오를 기반으로 RoutingModule, RouteReuseStrategy 관련 코드만 살펴보겠습니다. 설명은 각 코드들의 주석으로 대체하겠습니다.

** Code
*** app-routing.module.ts

#+BEGIN_SRC typescript
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';

const routes: Routes = [
  {
    path: '',
    pathMatch: 'full',
    component: AppComponent,
    data: {
      isReuse: false
    }
  },
  {
    path: '/items',
    pathMatch: 'full',
    component: ItemListComponent,
    data: {
      isReuse: true  // isReuse가 true일 경우에만 RouteReuseStrategy에서 재사용됩니다.
    },
  },
  {
    path: '/items/:itemId',
    pathMatch: 'full',
    component: ItemDetailComponent,
    data: {
      isReuse: false  // isReuse가 true일 경우에만 RouteReuseStrategy에서 재사용됩니다.
    }
  }
];

@NgModule({
  imports: [
    RouterModule.forRoot(routes),
  ],
  exports: [RouterModule],
})
export class AppRoutingModule { }

#+END_SRC

*** custom-route-reuse-stragety.module.ts

#+BEGIN_SRC typescript
import {
  ActivatedRouteSnapshot,
  DetachedRouteHandle,
  RouteReuseStrategy,
} from '@angular/router';

export class CustomRouteReuseStrategy implements RouteReuseStrategy {
  private storedRoutes = new Map<string, DetachedRouteHandle>();

  shouldDetach(route: ActivatedRouteSnapshot): boolean {
    return !!route.data.isReuse; // 현재 페이지의 data.isReuse 값이 true인 경우에만 store 함수를 수행.
  }

  store(
    route: ActivatedRouteSnapshot,
    handle: DetachedRouteHandle | null
  ): void {
    this.storedRoutes.set(this.getRouteUrl(route), handle!);
  }

  shouldAttach(route: ActivatedRouteSnapshot): boolean {
    return (
      !!route.data.isReuse && !!this.storedRoutes.get(this.getRouteUrl(route))
    );
  }

  retrieve(route: ActivatedRouteSnapshot): DetachedRouteHandle | null {
    return this.storedRoutes.get(this.getRouteUrl(route)) || null;
  }

  shouldReuseRoute(
    future: ActivatedRouteSnapshot,
    curr: ActivatedRouteSnapshot
  ): boolean {
    console.log('shouldReuseRoute');
    return future.routeConfig === curr.routeConfig && future.data.isReuse;
  }

  /*
    route.routConfig.url을 사용할 경우 하위 route가 있을 경우 오류가 발생하기 때문에
    내부의 _routerState에 직접 접근하여 full path를 추출하여 storedRoutes의 key로 사용한다.
  */
  private getRouteUrl(route: ActivatedRouteSnapshot): string {
    return `${(route)._routerState.url.replace(/\//g, '_')}_${route?.routeConfig?.loadChildren || route?.data?.key}`;
  }
}
#+END_SRC

