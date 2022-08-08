# 코틀린으로 작성된 스프링 기반 API 트랜잭션 rollback 관리


Kotlin + Spring boot을 조합해 작성한 REST API에서 [[https://kotlinlang.org/docs/reference/exceptions.html#checked-exceptions][CheckedException]]이 발생해도 Rollback이 되지 않는 문제와 그 해결법에 대해 알아봅시다. 이 글은 Spring, RDB, JPA에 대한 기본적인 이해가 있다는 가정 하에 작성되었습니다.

* Checked Exception?
  우선 Checked Exception에 대해서 알아봅시다. Kotlin에서 Exception은 크게 두가지로 나눌 수 있습니다.
  
  + Checked Exception: 컴파일 시 체크되는 예외입니다. Runtime Exception에 해당되지 않는 모든 Exception은 Checked Exception이 됩니다.
  + Runtime(Unchecked) Exception: 컴파일 단계에서 체크되지 않으며(Unchecked) 실행 중(Runtime) 발생하는 예외입니다(Ex: IndexOutOfBoundsException, IllegalArgumentException, etc...).

  더 자세한 내용은 [[http://wonwoo.ml/index.php/post/1542][여기]]를 참고하세요.

* 요구사항
  스프링은 기본적으로 Checked Exception이 발생해도 Transaction을 롤백시키지 않습니다. RuntimeException이 발생할때만 롤백이 된다고 알려져 있는데 Kotlin으로 막상 구현해보니 Checked Exception이 발생해도 Transaction이 롤백되는 현상이 발생하고 있습니다. 이 버그를 수정하고자 합니다.

* Code
** Exception
   우선 Exception 코드를 봅시다.

   #+BEGIN_SRC kotlin
class CustomException(message: String?): Exception(message) {
    constructor(e: Exception): this(message = e.message)
}
   #+END_SRC

   Exception을 상속받아 만든 Checked Exception입니다. *이 예외가 발생할 경우엔 Transaction이 롤백되지 않아야합니다.*
** Entity Model
   JPA에서 사용할 Entity Model입니다.
   #+BEGIN_SRC kotlin
@Entity
@Table(name = "custom")
@Component
class CustomModel(
    @get:Id
    @get:Column(length = 50, updatable = false)
    var name: String = ""
)
   #+END_SRC
** Controller
   API 호출을 위해 사용할 Controller 클래스입니다. @Transactional 어노테이션을 추가하여 API 호출 단위로 Transaction을 관리하고 있습니다.
   #+BEGIN_SRC kotlin
@RestController
@RequestMapping("/{version}/custom")
@Transactional
class CustomController {
    @Autowired
    private lateinit var customService: CustomService

    @PostMapping("")
    fun addCustom(
    ) {
        this.customService.add()
    }
}
   #+END_SRC

** Service
   실제 비즈니스 로직이 들어있는 Service 클래스입니다. 여기서 CustomException을 발생시키지만 7번 라인에서 저장한 CustomModel이 롤백되지 않아야 합니다.
   #+BEGIN_SRC kotlin
@Service
class CustomService {
    @Autowired
    private lateinit var customRepository: CustomRepository

    fun add(): CustomModel {
        this.customRepository.save(CustomModel(name="customName")) // 예외 발생 시 롤백됨.
        throw CustomException("must not rollback")
    }
}
   #+END_SRC
** Repository
   DB에 접근하기 위한 Repository 인터페이스입니다.
   #+BEGIN_SRC kotlin

@Repository
interface CustomRepository : JpaRepository<CustomModel, String> {
}
   #+END_SRC
** Exception Handler
   Service 클래스에서 발생한 예외를 감지해서 API 호출자에게 적절한 형태로 반환하기 위한 ExceptionHandler 클래스입니다.
   #+BEGIN_SRC kotlin
@ControllerAdvice
class GlobalExceptionHandler {
    private final val logger = Logger.getLogger("exception")

    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    @ExceptionHandler(value = [CustomException::class])
    fun handleBaseException(e: CustomException): String {
        return "fail"
    }
}
   #+END_SRC
* FIX
  위의 코드를 실행시켜보면 DB에 값이 들어가지 않는것을 확인할 수 있습니다. 스프링은 기본적으로 RuntimeException 발생시에만 롤백된다고 알고 있는데 동작이 이상합니다.
** 원인
   Kotlin은 기본적으로 예외에 대해서 ~throws~, ~try catch~ 구문을 강제하지 않습니다. 특히 ~throws~ 구문은 Kotlin에서는 아예 존재하지 않는데 대신에 ~@Throws~ 어노테이션을 지원하고 있습니다. @Throws를 이용해서 자바와 마찬가지로 해당 메서드의 호출자에게 이 메서드가 발생시킬 수 있는 예외에 대해서 알려줄 수 있습니다. 위의 코드에서는 이 구문을 작성해주지 않았기 때문에 Unchecked Exception으로 간주되지 않았을까 하는 추측을 해보았습니다.
** 해결방법
   ~CustomService.add~, ~CustomController.addCustom~ 메서드에 모두 ~@throws~ 어노테이션을 추가해주면 Checked Exception으로 간주되어 롤백되지 않을거라 생각하고 코드를 수정해보았습니다.
*** fixed Controller class
	#+BEGIN_SRC kotlin
@RestController
@RequestMapping("/{version}/custom")
@Transactional
class CustomController {
    @Autowired
    private lateinit var customService: CustomService

    @PostMapping("")
    @Throws(CustomException::class) // Throws annotation에 CustomException을 추가해줍니다.
    fun addCustom(
    ) {
        this.customService.add()
    }
}
	#+END_SRC
*** fixed Service class
	#+BEGIN_SRC kotlin
@Service
class CustomService {
    @Autowired
    private lateinit var customRepository: CustomRepository

    @Throws(CustomException::class) // Throws annotation에 CustomException을 추가해줍니다.
    fun add(): CustomModel {
        this.customRepository.save(CustomModel(name="customName")) // 이제 롤백되지 않습니다!
        throw CustomException("must not rollback")
    }
}
	#+END_SRC
* 결과 및 후기
  이제 의도한대로 CustomException이 발생해도 해당 Transaction이 롤백되지 않습니다. 이 방법 이외에도 Transactional 어노테이션에 rolblackFor 속성을 지정하는 방법도 있는데 이 방법은 [[https://pjh3749.tistory.com/269][이 곳]]을 참고하세요.

  웹상에 있는 자료 대부분이 자바 기준으로 설명이 되어있어서 해결법을 찾기가 쉽지 않았네요. 특히 이번에 마주한 문제는 어떻게 보면 상당히 기초적인 문제가 원인이 되어서 더 그렇지만요. 아무튼 스프링에서 공식적으로 Kotlin을 지원하고 있는만큼 Kotlin 기반 스프링 유저들이 좀 더 많이 늘어나 레퍼런스가 많이 쌓였으면 좋겠다는 생각이 들었습니다. 제가 이 글을 쓰는 이유도 미래의 저 자신을 포함한 다른 누군가는 저와 같은 삽질을 하지 않았으면 하는 마음에서 작성하였습니다. 실제 코드는 [[https://github.com/YeongCheon/springboot-playground][이 곳]]에서 확인하실 수 있습니다.

