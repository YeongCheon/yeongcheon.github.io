# hugo에서 수학 수식 표현하기


이 글은 [[https://themes.gohugo.io/theme/hugo-coder/post/math-typesetting.html][여기]][[https://emacs.stackexchange.com/questions/2650/how-should-i-write-inline-equations-in-org-mode-so-they-export-to-latex-properly][저기]]를 참고해서 작성되었습니다.

* 목표
  이 문서에는 [[https://gohugo.io/][hugo]] + [[https://orgmode.org/][org mode]] + [[https://katex.org/][kaTex]] 를 조합해서 글을 작성하는 법을 설명합니다. hugo와 org mode는 이미 사용 중이라고 가정하고 hugo에 kaTex를 설정하는 법을 중점적으로 설명합니다.

* Installation kaTex
** math.html 파일 생성
   hugo에서 kaTex를 사용하려면 kaTex 모듈을 불러와야 합니다. ~/layouts/partials/math.html~ 파일을 생성하고 아래의 내용을 추가합니다. 혹시 이미 math.html 파일명을 사용중이라면 다른 파일명으로 바꿔서 생성해도 상관없어요.

   #+BEGIN_SRC html
   <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.11.1/dist/katex.min.css" integrity="sha384-zB1R0rpPzHqg7Kpt0Aljp8JPLqbXI3bhnPWROx27a9N0Ll6ZP/+DiW/UqRcLbRjq" crossorigin="anonymous">
   <script defer src="https://cdn.jsdelivr.net/npm/katex@0.11.1/dist/katex.min.js" integrity="sha384-y23I5Q6l+B6vatafAwxRu/0oK/79VlbSz7Q9aiSZUvyWYIYsd+qj+o24G5ZU2zJz" crossorigin="anonymous"></script>
   <script defer src="https://cdn.jsdelivr.net/npm/katex@0.11.1/dist/contrib/auto-render.min.js" integrity="sha384-kWPLUVMOks5AQFrykwIup5lo0m3iMkkHrD0uJ4H5cjeGihAutqP0yW0J6dpFiVkI" crossorigin="anonymous" onload="renderMathInElement(document.body);"></script>
   #+END_SRC
** math.html 불러오기
   이제 위에서 작업한 ~math.html~ 파일을 불러오는 코드를 작성해야 합니다. 이 코드는 기존에 있던 ~/layouts/partials/head.html~ 파일을 열어서 추가해줍니다. 저같은 경우엔 ~head_custom.html~ 파일이 별도로 있어서 거기에 추가한 후 이 파일을 head.html에 추가했지만 어떤 식으로 하든 동작은 동일합니다. 일단 코드를 봅시다.

   #+BEGIN_SRC html
   {{ if or .Params.math .Site.Params.math }}
   {{ partial "math.html" . }}
   {{ end }}
   #+END_SRC
* 실제로 사용해보기
** 수식 작성법
   기본적인 tex 문법은 [[https://ko.wikipedia.org/wiki/%25EC%259C%2584%25ED%2582%25A4%25EB%25B0%25B1%25EA%25B3%25BC:TeX_%25EB%25AC%25B8%25EB%25B2%2595][위키]]에 설명이 잘 되어있습니다.
** inline block(a.k.a Parens)
   본문 사이에 자연스럽게 문법을 추가하고 싶다면(예를 들어 \(E=mc^2\) 이런 식으로)
   #+BEGIN_SRC
    \(E=mc^2\)
   #+END_SRC
** block
   수식을 *똭* 강조해서 표현하고 싶을 경우도 있습니다.
   $$E=mc^2$$
   이렇게 하면 수식이 가운데 정렬이 되면서 눈에 확 들어오죠? 실제 문서에는 다음처럼 작성하면 됩니다.
   #+BEGIN_SRC
    $$E=mc^2$$
   #+END_SRC

