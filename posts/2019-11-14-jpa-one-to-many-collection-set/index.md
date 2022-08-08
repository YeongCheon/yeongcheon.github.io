# JPA OneToMany 필드의 StackOverflowError


JPA를 사용하다 보면 StackOverflow 에러를 종종 만날 수 있다. 이 글에서는 필드 타입이 [[https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-set/index.html][Set]]인 Collection 타입에 아이템을 추가할 경우 StackVoerflow 에러가 발생하는 원인과 그 해결법에 대해 알아보자.

`hashcode`, `equals` 메서드와 관계된 이야기이다.

* 에러 발생 시나리오
** 구현 목표
	부모객체와 자식객체들을 서버 데이터베이스에 저장하는 기능을 가진 코드가 있었다. 부모객체와 자식객체는 1:n관계로서 하나의 부모가 여러 개의 자식을 가질 수 있고 이 두 객체는 서로의 존재를 알고 있는 양방향 관계다. 이 관계를 OneToMany를 이용해서 표현하고 있었는데 이 때 사용하고 있던 Collecion 타입이 리스트(List)였다. 근데 여기저기 인터넷을 뒤지다 보니 OneToMany는 List보단 Set을 사용하는게 좋다는 글을 보게 되었고([[https://jojoldu.tistory.com/165][링크]]) 이 글이 추천하는 방법대로 Entity Model을 바꿔보기로 했다.
** 에러 발생 조건
	수정 자체는 그리 어렵지 않았다. 그냥 List 타입을 Set으로 바꿔주면 끝날줄 알았다(+뭔가를 변경할때는 방심하지 말자+).

	수정 후 테스트를 해보았다. 
	+ 부모만 단독으로 추가할 경우 성공
	+ 부모객체 하나에 자식객체 하나를 추가할 경우 성공
	+ *부모객체 하나에 2개 이상의 자식 객체를 담아서 추가할 경우 실패* - StackOverFlowError 발생

 비즈니스 로직 코드는 원래 문제가 없었으니 이번에 수정한 Entity Model 관련 코드를 살펴보자.
* Entity Model code
** Parent.kt
	#+BEGIN_SRC kotlin
@Entity
@Table(name = "parent")
data class Parent(
    @get:Id
    @get:GeneratedValue(strategy = GenerationType.IDENTITY)
    var id: Long,

    @get:OneToMany(mappedBy = "parent")
    var childs: MutableSet<Child> // 이 부분이 MutableList에서 MutableSet으로 변경되었다.
) {}
	#+END_SRC
** Child.kt
	#+BEGIN_SRC kotlin
@Entity
@Table(name = "child")
data class Child(
    @get:Id
    @get:GeneratedValue(strategy = GenerationType.IDENTITY)
    var id: Long,

    @get:ManyToOne(cascade = [CascadeType.DETACH], fetch = FetchType.LAZY)
    @get:JoinColumn(name = "parent_id")
    var parent: Parent
) {}
	#+END_SRC
* 원인 파악

  우선 발생하는 에러는 순환 참조로 인해 발생한 에러로 JPA를 사용하다 보면 제법 자주 만나볼 수 있는 에러다. 자세한 내용은 구글에 [[https://www.google.com/search?newwindow=1&sxsrf=ACYBGNRvc-j9wUzYmyxwQeA9hKkEnp7H1g%253A1574606204506&source=hp&ei=fJXaXdnWHNv6-QbcsqrgBw&q=jpa+%25EC%2596%2591%25EB%25B0%25A9%25ED%2596%25A5+%25EC%2588%259C%25ED%2599%2598%25EC%25B0%25B8%25EC%25A1%25B0&oq=jpa+%25EC%2588%259C%25ED%2599%2598&gs_l=psy-ab.3.1.0j0i8i30.1581.4291..5790...1.0..0.152.1185.3j8......0....1..gws-wiz.......0i3j0i131j35i39.msmwejOXyx0][JPA 양방향 순환참조]]를 검색해보자.

  기존에 잘 돌아가던 기능이었는데 Collection을 List -> Set으로 바꿨다고 갑자기 발생하는 이유가 뭘까? List와 Set에 대해서 간략하게 알아보자. JPA(hibernate) 내부에서 어떻게 동작하는지는 [[http://wonwoo.ml/index.php/post/992][다른 글]]에서 알아보고 이번 글에서는 각 Collection 타입의 특징을 몇개 짚어보자.

  - List Collection
    + 순서가 있는 데이터의 모임
    + *중복을 허용한다.*
    + 대표적으로 ArrayList 와 LinkedList 가 있다.

  - Set Collection
    + 순서가 없는 데이터의 모임(구현체에 따라 순서가 있는 경우도 있다.)
    + *중복을 허용하지 않는다.*
    + 대표적으로 HashSet 이 있다.

  이 중에서 우리가 주목해야 할 특징은 중복 여부이다. 앞서 테스트 케이스에서 성공 케이스와 실패 케이스를 살펴보면 자식객체가 2개 이상일 경우에만 에러가 발생했는데 이 때 코드가 동작하는 순서는 아래와 같다.

  1. Parent 객체를 생성한다.
  2. Parent 객체에 종속된 Child 객체를 생성해서 Parent.childs(MutableSet) 필드에 추가한다.
  3. Child 객체를 하나 더 생성해서 마찬가지로 Parent.childs 필드에 추가한다.
  4. 이 때 childs 필드에 이미 객체가 존재하므로 추가하려는 객체와 동일한 객체인지 확인한다. - *이 때 에러 발생*
  5. 동일한 객체가 아닐 경우 Parent.childs 필드에 객체를 추가한다.

  동일한 객체인지 판단할때 에러가 발생하는 이유를 알고자 하면 우선 Set Collection이 중복 체크를 어떻게 하는지 알아야 한다.

  자바 계열의 언어에선 [[https://opentutorials.org/module/516/6241][모든 객체가 Object class를 상속받은 채로 생성된다]](최상위 클래스). 이 Object 클래스에는 [[https://nesoy.github.io/articles/2018-06/Java-equals-hashcode][equals, hashcode]] 메서드가 구현되어 있는데 Set Collection이 객체의 중복 여부를 체크할 때 이 두 메서드를 사용해서 중복 여부를 판단한다. 즉, Child 객체의 중복여부를 판단하기 위해서 Child 객체의 모든 필드값을 다 읽는 작업을 하게 되는데 이 때 Child.parent -> Parent.childs -> Child.parent -> Parent.childs ... 와 같은 순환참조 에러가 발생하게 된다.

* 해결법
  equals, hashcode 메서드 호출 중 에러가 발생하므로 이 두 메서드를 오버라이딩 해주면 해결할 수 있다. Parent는 건드릴 필요 없고 Child만 재정의 해주면 되는데 설명은 그만하고 수정된 코드를 보자.

** Child.kt
	#+BEGIN_SRC kotlin
@Entity
@Table(name = "child")
data class Child(
    @get:Id
    @get:GeneratedValue(strategy = GenerationType.IDENTITY)
    var id: Long,

    @get:ManyToOne(cascade = [CascadeType.DETACH], fetch = FetchType.LAZY)
    @get:JoinColumn(name = "parent_id")
    var parent: Parent
) {
    override fun equals(other: Any?): Boolean {
        return this.id == (other as SponsorshipGoalModel).id
    }

    override fun hashCode(): Int {
        return this.id.hashCode()
    }
}
	#+END_SRC


  데이터베이스 기준으로 생각해보면 어차피 id 필드가 기본키로 설정되어 있기 때문에 id 필드만 중복되지 않으면 서로 다른 객체로 판단해도 문제가 없을거라고 판단했기에 id 필드만 체크하도록 수정해줬고 이 후에는 순환참조에러 없이 잘 동작한다.

