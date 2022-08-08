# Spring Pointcut 표현식에 부모 클래스 표현하기


* 들어가며
이 문서는 기본적으로 Spring AOP에 대해 사전 지식이 있다는 전제 하에 작성되었습니다.
* 요구사항
=Fruit= 클래스를 상속받는 =Apple=, =Banana= 클래스가 있습니다. =Apple= 또는 =Banana= 클래스를 반환하는 모든 메서드 마지막에 로그를 남기는 기능을 작성하고자 합니다.
* Basic Code
** Model
#+BEGIN_SRC kotlin
package dev.yeongcheon.example.aop.model

abstract class Fruit(
    val name: String
)

class Apple: Fruit(
    name = "apple"
)

class Banana: Fruit(
    name = "banana"
)
#+END_SRC
** Controller
#+BEGIN_SRC kotlin
package dev.yeongcheon.example.aop.controller

import dev.yeongcheon.example.aop.model.Apple
import dev.yeongcheon.example.aop.model.Banana
import org.springframework.web.bind.annotation.GetMapping
import org.springframework.web.bind.annotation.RestController

@RestController
class FruitController {
    @GetMapping("/apple")
    fun getApple(): Apple {
        return Apple()
    }

    @GetMapping("/banana")
    fun getBanana(): Banana {
        return Banana()
    }
}
#+END_SRC
* Aspect Code
아래의 코드는 =/apple=, =/banana= API를 호출할때마다 해당 과일의 =name= 필드를 화면에 찍는 코드입니다.

하지만 *아래의 코드는 의도한대로 동작하지 않습니다.*
** Incorrect Code
#+BEGIN_SRC kotlin
package dev.yeongcheon.example.aop.aop

import dev.yeongcheon.example.aop.model.Fruit
import org.aspectj.lang.annotation.AfterReturning
import org.aspectj.lang.annotation.Aspect
import org.springframework.stereotype.Component

@Aspect
@Component
class FruitAspect {
    @AfterReturning("execution(dev.yeongcheon.example.aop.model.Fruit *..*.*(..))", returning = "fruit") // Incorrect Code
    fun afterReturnFruit(fruit: Fruit) {
        println(fruit.name)
    }
}
#+END_SRC
위의 코드에서 문제가 되는 부문은 Pointcut 표현식입니다. =Fruit= 클래스를 반환하는 모든 메서드를 대상으로 하는 코드인데, 당연히 ~Fruit~ 클래스를 상속받는 Apple, Banana 클래스를 반환해도 위의 표현식에 걸릴것이라고 예상하지만 위의 표현식은 정확히 Fruit 클래스"*만*" 대상으로 합니다. 따라서 *해당 클래스를 상속받는 다른 클래스들은 위의 표현식에 걸리지 않습니다.*
** Correct Code
위의 표현식에서 Apple, Banana 같이 Fruit 클래스를 상속받은 클래스도 표현식에 잡히도록 하려면 표현식에서 Fruit 클래스 뒤에 "*+*"를 붙여주면 됩니다.
#+BEGIN_SRC kotlin
@Aspect
@Component
class FruitAspect {
    @AfterReturning("execution(dev.yeongcheon.example.aop.model.Fruit+ *..*.*(..))", returning = "fruit") // use '+'
    fun afterReturnFruit(fruit: Fruit) {
        println(fruit.name)
    }
}
#+END_SRC
* 응용
** Generic
만약 =List<Fruit>= 타입을 반환하는 메서드를 대상으로 하고자 한다면 제네릭 표현식 안에 위에서 했던것과 동일하게 '+' 기호를 써주면 됩니다.
#+BEGIN_SRC kotlin
@Aspect
@Component
class FruitAspect {
    @AfterReturning("execution(java.util.List<dev.yeongcheon.example.aop.model.Fruit+> *..*.*(..))", returning = "fruit")
    fun afterReturnFruit(fruit: Fruit) {
        println(fruit.name)
    }
}
#+END_SRC
** Generic in Generic
만약 List 말고도 Set같은 다른 java.util.Collection 클래스를 상속받는 모든 클래스를 대상으로 하고자 한다면 아래와 같이 작성하면 됩니다.
#+BEGIN_SRC kotlin
@Aspect
@Component
class FruitAspect {
    @AfterReturning("execution(java.util.Collection<dev.yeongcheon.example.aop.model.Fruit+>+ *..*.*(..))", returning = "fruit")
    fun afterReturnFruit(fruit: Fruit) {
        println(fruit.name)
    }
}
#+END_SRC

