# jackson을 이용해 json 변환 시 JPA entity의 id값만 추출하기


* 들어가며
  REST API 서버를 java언어에서 사용되는 대표적인 json(또는 xml, yaml, etc...) 라이브러리인 [[https://github.com/FasterXML/jackson][jackson]] 은 kotlin에서도 사용이 가능하다. 
  JPA를 사용하다보면 entity를 json형식으로 반환할 때 해당 entity가 다른 entity를 필드로 참조하고 있을 경우 참조하는 entity의 id값만을 반환하고 싶을 경우가 있다.
  이 문제를 jackson을 이용해  json형식으로 이쁘게 변환하는 법을 알아보자.
* 요구사항
  데이터베이스에는 현재 user 테이블과 shelter 테이블이 존재한다. shelter 테이블에는 user 테이블을 참조는 외래키 제약조건이 걸려있다. 자세한 내용은 다음 섹션의 소스코드를 참고해보자.

  현재 사용자가 shelter 정보를 조회하는 API를 호출할 경우 쉘터 정보 안에 쉘터 소유자의 상세한 정보가 포함되어 있다고 한다. json 포맷을 보면 다음과 같다.

  #+BEGIN_SRC json original_json
{
  "id": 1,
  "name": "awesome shelter"
  "owner": {
    "id": "kyc1682",
	"name": "yeongcheon"
  }
}
  #+END_SRC

  위와 같이 소유자의 상세한 정보가 포함되어 넘어오고 있다. 

  만약에 *사용자가 owner 필드에 사용자의 정보가 담긴 json object가 아닌 owner의 id 정보만 원한다고 가정해보자.*
  그렇다면 API의 응답값은 아래처럼 바뀔것이다.

  #+BEGIN_SRC json want_json
{
  "id": 1,
  "name": "awesome shelter"
  "owner": "kyc1682"
}
  #+END_SRC

  응답결과를 이런식으로 참조 객체의 id값만 출력하는 형식으로 바꿀 경우 생기는 이점은 아래와 같다.

  1. 최초 json 결과값을 출력하기 위해서는 shelter에 포함된 User의 정보를 불러오기 위해 데이터베이스에서 불필요한 join 쿼리를 실행한다.
	 하지만 개선된 json 결과값은 join 쿼리를 실행할 필요 없이 단순한 쿼리로도 충분하다. 이는 데이터베이스 부하가 줄어듦을 의미한다.
  2. 통신데이터의 크기가 줄어든다. 현재 user 테이블은 고작 2개의 필드밖에 없지만 실제로는 훨씬 많은 수의 필드들이 존재한다.
     shelter 1개를 조회할때는 json의 크기가 크게 차이가 안나겠지만 만약에 20개, 30개의 배열 형식으로 데이터를 요청한다면? json의 크기는 엄청나게 불어난다.
	 개선된 json 결과값의 경우엔 json의 크기가 훨씬 줄어들기 때문에 서버와 클라이언트 사이에 통신하는 데이터의 크기가 줄어들고 API 응답속도가 빨라진다.

  이제 실제로 사용된 소스를 살펴보자.
  
* source
** intro
   예제에서 사용될 코드를 소개한다. 크게 모델과 IdResolver 클래스가 있다. 
   이 예제에서는 UserModel과 ShelterModel, 그리고 jackson 라이브러리의 ~ObjectIdResolver~ 인터페이스를 상속받아 구현한 EntityIdResolver를 준비했다.
** EntityIdResolver
#+BEGIN_SRC kotlin entityIdResolver
// jpa entity 모델 json 변환 시 id만 반환할 수 있도록 해주는 클래스
class EntityIdResolver(
        private val entityManager: EntityManager) : ObjectIdResolver {

    override fun bindItem(
            id: ObjectIdGenerator.IdKey,
            pojo: Any) {
    }

    override fun resolveId(id: ObjectIdGenerator.IdKey): Any {
        return this.entityManager.find(id.scope, id.key)
    }

    override fun newForDeserialization(context: Any): ObjectIdResolver {
        return this
    }

    override fun canUseFor(resolverType: ObjectIdResolver): Boolean {
        return false
    }
}
#+END_SRC

[[https://stackoverflow.com/questions/44007188/deserialize-json-with-spring-unresolved-forward-references-jackson-exception][출처]]

** UserModel
#+BEGIN_SRC kotlin userModel
import com.fasterxml.jackson.annotation.*
import com.shelter.EntityIdResolver
import java.io.Serializable
import java.time.LocalDateTime
import javax.persistence.*
import javax.validation.constraints.Email

typealias UserId = String

@Entity
@Table(name = "user")
@JsonIgnoreProperties("deleted", "fcmToken")
data class UserModel(
        @Id
        @Column(length = 50, updatable = false)
        var id: UserId = "",

        @Column(length = 50, updatable = true, unique = true)
        var name: String = "",
)
  
#+END_SRC

** ShelterModel

#+BEGIN_SRC kotlin ShelterModel
import com.fasterxml.jackson.annotation.*
import com.fasterxml.jackson.databind.annotation.JsonDeserialize
import com.shelter.EntityIdResolver
import java.io.Serializable
import java.time.LocalDateTime
import javax.persistence.*

typealias ShelterId = Long

@Entity
@Table(name = "shelter")
data class ShelterModel(
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        var id: ShelterId = 0,

        var name: String = "",

        @JsonIdentityInfo(
                generator = ObjectIdGenerators.PropertyGenerator::class,
                property = "id",
                resolver = EntityIdResolver::class,
                scope = UserModel::class)
        @JsonIdentityReference(alwaysAsId = true)
        @OneToOne(fetch = FetchType.EAGER, cascade = [CascadeType.DETACH], orphanRemoval = false)
        @JoinColumn(name = "owner_id", updatable = false)
        var owner: UserModel = UserModel()
)
#+END_SRC

* 해설
=EntityIdResolver= 를 제외하면 단순한 모델 클래스가 전부이다. 
모델 클래스에 어노테이션이 이것저것 붙어있지만 jackson 관련된 어노테이션은 =JsonIdentityInfo=, =JsonIdentityReference= 두 개 뿐이다.

** JsonIdentityInfo
   딴거 다 필요없고 =@JsonIdentity= 어노테이션이 핵심이다.
   코드를 보면 이 어노테이션에 총 4개의 parameter가 존재하는데 하나씩 살펴보도록 하자.
   
   + generator: 순환참조에 대한 식별자를 생성하는데 사용된다. 
     여기에 값으로 할당된 =Objectidgenerators.PropertyGenerator= 는 식별자를 생성하는데 해당 object의 property 중 하나를 사용하겠다는 뜻이다.
   + property: 어떤 property를 id로 사용할 지 선언한다. 이 예제에서는 UserModel의 =id= 필드를 기본키(primary key)로 사용한다. 
     만약에 =id= 필드가 아닌 =name= 필드를 id로 사용하고 싶을 값을 name으로 할당하면 된다.
   + resolver: [[https://fasterxml.github.io/jackson-annotations/javadoc/2.4/com/fasterxml/jackson/annotation/ObjectIdResolver.html][javadoc]]의 설명에 따르면 *객제 식별자로부터 POJO를 구성하기 위해 사용되는 API의 정의* +(feat 구글 번역기)+ 라고 한다.
   + scope: 대상이 되는 entity 클래스를 입력한다.

  위의 parameter 중에서 =serialize= 를 하기 위해서는 =generator=, =property= 만 지정하면 된다(아닐수도 있다). 여기서 serialize란 entity 객체를 json 형식으로 변환하는걸 의미한다.
  하지만 =deserialize= (json 데이터를 entity 객체로 변환)를 하기 위해서는 =resolver=, =scope= parmater도 설정을 해주어야 한다.
** JsonIdentityReference
   아주 간단하다. 객체를 항상 id필드만 반환할지 여부를 선언해주는 어노테이션이다. 이 예제에서는 =alwaysAsId= 값에 true를 할당해주었다.

* 후기
  최대한 jackson과 연관된 내용만을 다루기 위해 다른 설정이나 관련 코드들(jpa, spring, etc...)은 최대한 생략하고 글을 작성해보았다.
  내용도 공식 문서를 본게 아니라 일을 하면서 구글링한 내용들을 짜집기 해서 내가 보기좋은 형태(...)로 작성해놓은건데 다른분들이 볼때도 확 와닿았으면 좋겠다.

  그리고 이번 포스트는 처음으로 emacs의 [[https://orgmode.org/][org-mode]] 에서 작성해봤는데 뭔가 설정이 잘못된건지 코드 하이라이팅 기능이 제대로 동작하지 않는다.
  관련 설정을 좀 더 찾아보고 영 아니다 싶으면 [[https://gohugo.io][hugo]]같은 다른 블로그 엔진으로 갈아타야겠다.

  글 마지막에 늘 하는말이지만 혹시 잘못된 내용이나 추가되었으면 하는 내용이 있으면 [[mailto:kyc1682@gmail.com][여기]]로 메일을 주시거나 아님 github에 pull request를 보내주시면 감사하겠습니다.

