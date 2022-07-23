# kotlin에서 조건문(blcok문)변수 제대로 쓰기


* 기본 문법
** if 사용법
  kotlin에서는 if, when, try catch문을 활용해서 변수에 값을 편하게 할당할 수 있다. 예를 들어서 자바에서 아래와 같이 쓰는 문법이 있다고 해보자.

  #+BEGIN_SRC kotlin
  // java code
  String name;

  if(true) {
    name = "YeongCheon"
  } else {
    name = null
  }

  // 또는 삼항 연산자로 처리할수도 있다.
  name = true ? "YeongCheon" : null
  #+END_SRC

   위와 동일한 문법을 코틀린에서는 아래처럼 쓸 수 있다.
   #+BEGIN_SRC kotlin
   // kotlin code
   var name = if(true) {
     "YeongCheon"
   } else {
     null
   }

   // 코틀린에서는 삼항 연산자를 지원하지 않는다.
  #+END_SRC

  위와 같이 코드를 작성하면 if문 안에 각 block 안의 가장 마지막 코드가 name 변수 안에 할당된다. 그리고 위 예제에서 name 변수의 타입은 자동으로 *String?* 이 된다. 만약 else 블록에서 마찬가지로 String 타입을 반환하면 name 변수의 타입은 *String*, 그 이외에 다른 타입을 반환하면 *Any* 타입이 된다.

** when 사용법
   자바의 switch문에 해당하는 when, 그리고 try catch 문법도 위처럼 동일하게 사용할 수 있다.

   #+BEGIN_SRC kotlin
   val number = 1;
   val numberString = when(number) {
     0 -> "zero"
     1 -> "one"
     2 -> "two"
     3 -> "three"
  	  else -> null
   }
   #+END_SRC

  위 코드에서 numberString 변수의 타입은 *String?*, 값은 "one" 이 할당된다.

* 내가 했던 실수
  이 섹션이 내가 이번 글을 작성하게 된 이유다. 위에서 설명한 문법을 사용하다가 범하게 된 실수에 대해 이야기하고자 한다. 별것도 아닌 오류이지만 심각한 문제를 야기할 수 있다.

** 잘못된 코드
   #+BEGIN_SRC kotlin
   fun test(number: Int): Boolean {
        val processResult = if(number == 1) {
  	       return true
        } else {
  		    return false
  	   	}

        test02()

        if(processResult) {
            return test03()
        } else {
            return test04()
        }
   }
   #+END_SRC

   test02, test03, test04 메서드가 제대로 작성되어 있다는 가정 하에 위 코드는 문법상으로는 전혀 문제가 없는 코드이다. 위 코드에서 문제점을 발견하였는가?

   위 코드에는 dead code가 존재한다. processResult에 값을 할당하는 코드 이후, 즉 test02 메서드는 number 변수의 값에 상관없이 절대 실행되지 않는데 그 이유는 간단하다. 나는 위 코드에서 processResult에 true, 또는 false가 할당될것이라 기대하고 코드를 작성하였지만 *if문 안에 return 키워드를 쓰고 말았다. 그 결과 test 메서드는 number가 1이면 true, 아니면 false를 반환하는 단순한 메서드가 되버리고 말았다.* 

** 제대로 된 코드
   #+BEGIN_SRC kotlin
   fun test(number: Int): Boolean {
        val processResult = if(number == 1) {
  	         true
        } else {
  		     false
  	   	}

        test02()

        if(processResult) {
            return test03()
        } else {
            return test04()
        }
   }
   #+END_SRC

   *processResult에 값을 할당하는 구문에서 return 키워드를 제거했다.* 이 후 코드는 내가 의도한대로 test02 메서드를 실행한 후 processResult 값에 따라 test03 또는 test04 메서드를 실행시킨 후 그 결과값을 반환한 후 메서드를 종료한다.

* 후기
  이번 글처럼 아주 기초적인 문법 오류 때문에 꽤 심각한 상황을 맞이한 적이 있어서 이 글을 작성하게 되었다. 예제에서는 별 것 아닌것처럼 보이지만 수많은 코드 속에 위에서 설명한 *문법상 문제는 없지만* 의도한대로 작동하지 않는 코드를 잡아내는건 꽤나 힘이 드는 작업이다. 버그를 잡아내놓고 스스로도 어이가 없어서 실소를 했던 기억이 난다. 다른 사람들은 이런 어이없는 버그는 안 만들길 바란다.

