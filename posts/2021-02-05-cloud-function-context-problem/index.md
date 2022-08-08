# cloud function 호출 시 context의 중요성에 대해 알아보자.


[[/posts/2021-01-06-image-resize-cloud-function.org][예전 글]]에서 언급한 [[https://github.com/YeongCheon/imagick-cf][imagick-cf]]를 테스트 하는 도중 마주한 알 수 없는 특이한 오류와 이를 해결했던 과정을 기록합니다.

* 문제
1. url을 통해서 함수를 실행하면 최초 실행 후 일정 시간동안은 몇번을 호출하든 결과가 정상적으로 반환합니다. 
2. 하지만 이후 500 에러를 반환하기 시작합니다. 심지어 이 전에 정상적으로 반환했던 url을 그대로 호출해도 500 error를 반환하는 어처구니 없는 모습을 보여줍니다. 
3. 게다가 500 에러를 반환하는 경우엔 로그조차 남지 않아서 원인을 지레짐작으로 파악할 수 밖에 없는 상황입니다. 
4. 물론 local test시에는 매우 잘 동작합니다.

절망하다가 차츰 화가 나기 시작합니다.

* 원인 추측
처음에는 에러가 무작위로 발생한다고 생각했지만 여러번의 테스트를 해본 결과, *함수 배포 후 첫 호출 이후 60초 동안은 파라메터가 잘못되지 않는 이상 절대 에러가 발생하지 않았습니다.* 이 *60초* 라는 시간이 대체 어디서 나온 녀석인지 알아보기 위해서 클라우드 콘솔 화면을 여기저기 뒤적이다가 다음과 같은 화면을 발견했습니다.

[[/images/2021-02-05-cloud-function-context-problem-cloudfunction-setting.jpg]]

60초라는 키워드가 등장하는 제한 시간 옵션입니다. 개발자가 별도로 설정하지 않는다면 기본값으로 60초가 할당되는데 문제가 발생하는 기준인 최초 호출 이후 60초라는 제한 시간이 여기서 나왔다고 추측했습니다. 이후 이 값을 10초로 수정 후 테스트를 진행했더니 실제로 10초 뒤에 동일한 에러가 발생하였습니다.

물론 이 사실을 안다고 해서 버그를 수정하는데 직접적으로 도움이 되는건 아니지만 개발자의 직감에 도움이 됩니다. 함수 제한 시간과 관련이 있단 사실을 알고나니 갑자기 [[https://golang.org/pkg/context/][context]]가 원인이지 않을까 싶어서 관련 코드를 좀 살펴보았고, *결과부터 말하자면 context 관련 문제가 맞았습니다.*

* 대응
이제 실제 코드를 살펴봅시다. imagick-cf에서 context 관련 버그를 수정한 커밋은 [[https://github.com/YeongCheon/imagick-cf/commit/706a9cd35a617e35fd3db2c156024e508f620188][여기]]입니다. 어떻게 수정했는지는 커밋내역을 살펴보시고 여기서는 간략하게 일부분만 떼어서 살펴보겠습니다.

** Wrong code

#+BEGIN_SRC go
func OptimizeImage(w http.ResponseWriter, r *http.Request) {
...
originalImage := bucket.Object(imageName)
originalImageReader, err := originalImage.NewReader(context.Background()) // we have problem.
...
}
#+END_SRC

[[https://cloud.google.com/functions/docs/concepts/go-runtime?hl=ko#contextcontext][구글 공식 가이드]]에서는 http trigger로 함수 호출 시에는 ~http.Request.Context()~, background 실행 시에는 ~context.Background()~ 를 사용할 것을 권고하고 있습니다. 하지만 OptimizeImage 함수는 cloud function에서 http trigger로 동작하는 함수임에도 ~context.Background()~ 를 사용하고 있었습니다. 그럼 이제 구글이 하라는대로 코드를 고쳐봅시다.

** Good Code

#+BEGIN_SRC go
func OptimizeImage(w http.ResponseWriter, r *http.Request) {
...
originalImage := bucket.Object(imageName)
originalImageReader, err := originalImage.NewReader(r.Context()) // good!
...
}
#+END_SRC

context를 backgroud 대신 request에서 얻어와서 사용하도록 코드를 수정하였습니다. 위에 첨부된 코드 말고도 여러 영역에서 ~Context.background()~ 함수를 호출하고 있었기 때문에 관련 부분을 모두 수정해주었습니다. 아, 하지만 무조건 ~Context.background()~ 를 사용하지 말라는건 아닙니다. [[https://golang.org/doc/effective_go.html#init][func init()]] 안에서는 request를 통해서 context를 얻는게 불가능하기 때문에 Context.background()를 이용해 context를 사용해도 괜찮습니다.


* 결론

직접 띄우는 서버가 아니라 cloud function에서 가볍게 돌아갈 코드라고 너무 대충 작성한 제 과오를 반성합니다. 실제 클라우드 환경에서만, 그것도 로그가 안남는 버그가 발생해서 많이 당황했지만 그래도 아무생각 없이 대충 써왔던 context에 대해서 다시 한번 신경쓰는 계기가 되는 경험이었습니다. 문제를 해결하고 보니  다른 언어를 사용했으면 이런 context 관련 문제는 마주치지 않아도 되지 않았을까 생각했지만 그래도 golang은 여전히 저한테는 매력적인 언어입니다. 언어를 탓하지 않고 제가 짠 코드를 탓하는게 맞겠지요. 아무튼 재미있는 경험이었습니다.

*한 줄 요약: http trigger로 호출한 함수 안에서는 ~context.Background()~ 대신 ~request.Context()~ 를 사용합시다.*

