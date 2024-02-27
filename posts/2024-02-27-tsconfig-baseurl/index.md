# 타입스크립트 import 절대경로로 설정하기


타입스크립트 개발을 하다 보면 ~import~ 구문이 절대경로가 아니라 상대 경로로 포함되는 경우가 있습니다.

#+BEGIN_SRC typescript
import { MY_MODULE } from '../../../../my_module'; // incorrect

import { MY_MODULE } from 'src/modules/my_module'; // correct
#+END_SRC

~import~ 구문을 절대 경로로 포함하길 원한다면 ~tsconfig.json~ 파일의 ~compilerOptions~ 속성 내부에 ~baseUrl~ 속성을 지정하면 됩니다.

#+NAME: tsconfig.json
#+BEGIN_SRC json

{
  "compilerOptions": {
    "baseUrl": "./",
    ...
  }
}
#+END_SRC

