---
layout: post
title:  "[Kotlin in Action] 13장. DSL 만들기"
author: "minseo"
categories: [blog, java]
tags: [Kotlin, Java]
comments: true
image: "/static/img/logo/kotlin-logo.png"
hide_image: true
---

# [Kotlin in Action] 13장. DSL 만들기

<br><br><center><img src="../../../static/img/logo/kotlin-horizontal.png" width="30%"></center>
<br><br><br>

> 도서 <코틀린 인 액션 2/e>를 읽으며 코틀린에 대해 이해한 내용을 정리한 글이다.

<br>

코틀린 언어 특성을 이용해서 **도메인 특화 언어(DSL, Domain-Specific Language)**를 만들고 사용해 보자. 특히 수신 객체 지정 람다, invoke 관례를 자세히 살펴보며 코틀린다운 API 설계에 대해 알아보자.

목표는 **가독성, 유지보수성**을 가장 좋게 유지하는 것이다.

<br>

---

## 13.1 DSL의 정의와 필요성

우선 API와 DSL의 차이점을 통해 DSL이 왜 필요한지 이해하자.

### API란

- 소프트웨어 간의 상호 작용을 위한 통신 수단을 제공하는 것으로, 주로 **명령적**이며 '어떻게 동작하는가'에 집중한다.
- 예시로는 표준 라이브러리, REST API, 운영체제의 시스템 호출 등이 있다.

> cf. 깔끔한 API란? 코드를 읽는 사람이 어떤 일이 벌어질지 명확히 이해하며, 불필요한 구문 및 번잡한 준비 코드를 최소화한다.


### DSL이란

- 특정 종류의 문제나 작업을 표현하기 위해 설계된 언어로, 주로 **선언적**이며 '무엇을 할 것인가'에 집중한다.
- 예시로는 SQL <span style="color:#737373; font-size:14px; font-weight:300;"> 데이터베이스 질의에 특화 </span>, 정규식 <span style="color:#737373; font-size:14px; font-weight:300;"> 문자열 조작에 특화 </span>, HTML/CSS <span style="color:#737373; font-size:14px; font-weight:300;"> 문서 구조 및 스타일 지정에 특화 </span>, Maven∙Gradle Groovy DSL <span style="color:#737373; font-size:14px; font-weight:300;"> 빌드 과정 정의에 특화 </span> 등이 있다.

<br>

DSL은 간결하며, 선언적이므로 내부적으로 최적화하기 효율적이라는 장점이 있다. 그러나 **범용 언어로 만든 호스트 애플리케이션과 DSL을 직접적으로 통합하기 힘들다**는 단점이 있다. 둘의 상호작용을 컴파일 시점에 제대로 검증 및 디버깅하는 것은 쉽지 않으며, DSL을 위한 IDE 기능 제공도 어렵다.

따라서 코틀린은 **내부 DSL**이라는 것을 만들 수 있게 해준다.

<br>

---

### 내부 DSL

외부 DSL과 반대로 내부 DSL은 범용 언어로 작성된 프로그램의 일부이다.  
동일한 문법이고 같은 언어이지만, **주 언어를 별도의 문법으로 사용하는 것**을 말한다.

<br>

```sql
-- 데이터베이스 조회 작업을 외부 DSL(SQL)로 수행할 때

select Country.name, count(Customer.id)
      from Country
inner join Customer
      on Country.id = Customer.country_id
    group by Country.name
    order by count(Customer.id) desc
    limit 1
```
- **외부 DSL**의 경우 문자열 리터럴에 넣어 사용하며, IDE를 통해 코드 작성과 검증에 도움을 받는다.

<br>

```kotlin
// 데이터베이스 조회 작업을 내부 DSL(Exposed가 제공하는 Kotlin DSL)로 수행할 때

(Country innerJoin Customer)
    .slice(Country.name, Count(Customer.id))
    .selectAll()
    .groupBy(Country.name)
    .orderBy(Count(Customer.id), order = SortOrder.DESC)
    .limit(1)
```
- **내부 DSL**은 내부적으로 SQL과 동일한 프로그램이 생성되고 실행되지만, **일반 코틀린 코드로 작성**하고 **리턴 값을 코틀린 객체로 받을 수 있다**.

### DSL의 구조

- API와 다른 DSL에만 존재하는 특징이 있다. **구조** 또는 **문법**이다.

    - 일반적으로 API의 호출과 다른 호출 사이에는 맥락이 유지되지 않는다. **명령-질의(Command-Query) API**라고 부른다.

- DSL의 메서드 호출은 **문법에 의해 정해지는 더 커다란 구조**에 속한다.

    - 예를 들어 코틀린 DSL로 SQL 질의를 실행하려면 필요한 결과 집합의 여러 측면을 기술하는 **메서드 호출을 조합**해야 한다. <span style="color:#737373; font-size:14px; font-weight:300;"> 바로 앞에서 살펴본 코드 예시 참고 </span> 이는 질의에 필요한 인자를 메서드 호출 하나에 다 넘기는 것보다 읽기 쉽다.

<br>

```kotlin
str should startWith("kot")

// JUnit: assertTrue(str.startsWith("kot"))
```
- 테스트 프레임워크의 단언문도 가독성 좋게 작성할 수 있다.
- Kotlin을 위한 테스트 프레임워크 Kotest는 **메서드 호출을 연쇄**시켜 **구조**를 만든다.

<br>

```kotlin
// file: "build.kts"
dependencies { // 람다 내포. 중복 제거, 선언적
    implementation("org.jetbrains.exposed:exposed-core:0.40.1")
    implementation("org.jetbrains.exposed:exposed-dao:0.40.1")
}
```
- 또한 같은 맥락을 **재사용**할 수 있다. 예를 들어 람다 내포를 통해 구조를 만들어 코드 중복을 없앤다.

<br>

---

## 13.2 코틀린의 특성을 활용해 DSL 만들기

### 수신 객체 지정 람다로 DSL 만들기

```kotlin
fun buildString(
    builderAction: (StringBuilder) -> Unit
): String {
    val sb = StringBuilder()
    builderAction(sb)
    retrun sb.toString()
}

fun main() {
    val s = buildString {
        it.append("Hello, ")
        it.append("World!")
    }
    println(s)
    // Hello, World!
}
```
- 위 코드는 함수의 파라미터로 함수 타입을 받는다. 따라서 람다 본문에서 매번 `it`을 사용해서 `StringBuilder` 인스턴스를 참조해야 한다.

> 복습: 코틀린은 람다에 매개변수가 하나만 있을 때, 그 매개변수 이름을 `it`으로 자동 지정한다.

<br>

아래는 **수신 객체 지정 람다**를 활용한 코드이다. 우선 수신 객체 지정 람다가 어떤 효율을 제공하는지 복습하자.

```kotlin
fun buildString(
    builderAction: StringBuilder.() -> Unit
): String {
    val sb = StringBuilder()
    sb.builderAction() // StringBuilder 인스턴스를 람다의 수신객체로 넘긴다
    retrun sb.toString()
}

fun main() {
    val s = buildString {
        this.append("Hello, ")
        append("World!")
    }
    println(s)
    // Hello, World!
}
```
- 위 코드 함수를 정의할 때 **수신 객체가 지정된 함수 타입의 파라미터**를 선언한다. 따라서 `StringBuilder` 인스턴스를 람다의 수신 객체로 넘길 수 있다.
- 이에 따라 `this` 키워드로 (생략 가능) 수신 객체를 표현할 수 있게 된다.
- 즉 `builderAction`은 `StringBuilder` 클래스 안에 정의된 메서드가 아니며, `sb`는 확장 함수 호출할 때와 동일한 구문으로 호출할 수 있는 함수 타입의 인자일 뿐이다.

> `this`는 자바와 마찬가지로 현재 객체를 표현하거나, 확장 함수에서 수신 객체를 표현할 때 쓴다. 그러나 모호성을 해결해야 할 때만 사용하고 일반적으로는 생략한다.

<br>

```kotlin
String.(Int, Int) -> Unit
// . 왼쪽: 수신 객체 타입
// 괄호 내부: 파라미터 타입
// 화살표 오른쪽: 반환 타입
```
- 확장 함수의 타입 정의는 위와 같이 이루어진다.

<br>

실제 표준 라이브러리의 구현은 `apply` 함수를 사용하여 더 간결하다.

```kotlin
fun buildString(builderAction: StringBuilder.() -> Unit): String =
    StringBuilder().apply(builderAction).toString()
```
- `apply`는 인자로 받은 람다나 함수를 호출할 때, **자신의 수신 객체**를 **인자로 받은 람다나 함수의 암시적 수신 객체로 사용**한다.

> cf. `with`도 제공받은 수신 객체로 확장 함수 타입의 람다를 호출한다는 건 같은데, 얘는 수신 객체를 첫 번째 인자로 받고, 수신 객체가 아닌 람다 호출 결과를 반환한다. 결과를 받아서 쓸 필요가 있냐 없냐에 따라 바꿔 쓸 수 있음.

<br>

#### e.g. HTML 빌더 안에서 수신 객체 지정 람다 사용

- HTML 빌더는 HTML을 만들기 위한 코틀린 DSL으로, 타입 안전성을 보장한다.

> cf. 빌더는 객체 계층 구조를 선언적으로 정의할 수 있다. 따라서 UI 컴포넌트 레이아웃을 정의할 때 유용하다.

<br>

```kotlin
fun createSimpleTable() = createHTML().
    table {
        tr {
            td { +"cell" }
        }
    }
```
- 각 함수는 고차함수로, 수신 객체 지정 람다를 인자로 받는다.
- `table` 함수에 넘겨진 람다에서는 `tr` 함수를 사용해 `<tr>` HTML 태그를 만든다. 그러나 이 람다 밖에서는 `tr`이라는 함수를 찾을 수 없다. 나머지 함수들도 각자의 상위 구조에서만 접근 가능하다. **(문법, 구조)** 이러한 API 설계로 인해, HTML 언어의 문법을 따르는 코드만 작성할 수 있게 된다. **(도메인 특화)**

<br>

```kotlin
open class Tag

// 대문자 -> 유틸리티 클래스. 메서드들은 해당 타입을 수신 객체로 받는 람다를 인자로 받는다.

class TABLE : Tag {
    fun tr(init: TR.() -> Unit)
}

class TR : Tag {
    fun td(init: TD.() -> Unit)
}

class TD : Tag
```
- 각 블록의 이름 결정 규칙은 각 람다의 수신 객체에 의해 결정된다.
- 내포된 람다에서 외부 람다의 수신 객체를 참조한다면 헷갈릴 수 있다. 이를 막기 위해 `@DslMarker` 어노테이션이 제공된다.

<br>

이렇게 DSL을 정적 타입 언어와 함께 사용하면 **추상화**된 구조를 만들어 도메인 규칙을 지키도록 만들 수 있으며, 반복되는 코드 조각을 새 함수로 묶어 **재사용**할 수 있다.

### invoke 관례로 DSL 만들기

invoke 관례를 사용하면 어떤 커스텀 타입의 객체를 함수처럼 호출할 수 있다.

```kotlin
class Greeter(val greeting: String) {
    operator fun invoke(name: String) {
        println("$greeting, $name!")
    }
}

fun main() {
    val myGreeter = Greeter("Hello")
    myGreeter("Cat")
    // Hello, Cat!
}
```
- `Greeter` 인스턴스인 `myGreeter`를 함수처럼 호출할 수 있다. 내부적으로는 `myGreeter.invoke("Cat")`으로 컴파일된다.

> 관례란 특별한 이름의 함수를 일반 함수처럼 이름으로 호출하지 않고 다른 간결한 표기를 사용해 호출하는 것을 말한다. 예를 들어 `get` 함수 정의는 `foo.get(bar)`를 `foo[bar]`로 쓸 수 있게 만든다.

<br>

```kotlin
// 인자를 2개 받는 함수를 표한하는 함수형 인터페이스
interface Funcion2<in P1, in P2, out R> {
    operator fun invoke(p1: P1, p2: P2)
}
```
- 일반적인 람다 호출(`lambda()`)이 실제로는 이 관레를 적용한 것
- 람다를 함수처럼 호출하면 `invoke` 메서드 호출로 변환된다.

#### e.g. Gradle 의존관계 선언에서의 invoke 관례

```kotlin
class DependencyHandler { // 일반적인 명령형 API
    fun implementation(coordinate: String) {
        println("Added dependency on $coordinate")
    }

    operator fun invoke( // DSL 스타일 API
        body: Dependencyhandler.() -> Uint) {
            body()
        }
}

fun main() {
    val dependencies = DependencyHandler()

    // 의존성이 하나일 때 간결함
    dependencies.implementaton("...")

    // 여러 개일 때
    dependencies {
        implementaton("..."),
        ...
    }
}
```
- 두 번째 메서드는 관례를 활용한 DSL 스타일 API이다. `dependencies`를 함수처럼 호출하면서 람다를 인자로 넘긴다.

<br>

이렇게 재정의한 invoke 메서드로 인해 DSL API의 **유연성**이 훨씬 커진다. 이런 패턴은 일반적으로 적용할 수 있는 패턴이니 필요한 경우 기존 코드를 크게 변경하지 않고도 사용할 수 있다!

<br>

---

## 13.3 실용적인 DSL 구성 예제

실제로 DSL이 빛을 발하는 몇 가지 예제들을 살펴보며, 앞으로 개발할 때 사용해보자.

### Kotest: 중위 호출로 간결하게 테스팅 메서드 호출

```kotlin
import io.kotest.matchers.should
import io.kotest.matchers.string.startWith
import org.junit.jupiter.api.Test

class PrefixTest {
    @Test
    fun testKPrefix() {
        val s = "kotlin".uppercase()
        s should startWith("K")
    }
}
```
- `should` 함수를 중위 호출하여 단언문 코드를 거의 일반 영어처럼 읽을 수 있게 된다.

<br>

```kotlin
infix fun <T> should(matcher: Matcher<T>) = matcher.test(this)

...

interface Matcher<T> {
    fun test(value: T)
}

fun startWith(prefix: String): Matcher<String> {
    return object : Matcher<String> {
        override fun test(value: String) {
            if(!value.startsWith(prefix)) {
                throw AssertionError("$value does not start with $prefix")
            }
        }
    }
}
```
- `Matcher`는 값에 대한 단언문을 표현하는 제네릭 인터페이스이다.
- 라이브러리에서 다양한 단언을 할 수 있는 `Matcher`를 제공하고 있다. 따라서 간결하게 코드를 작성할 수 있다.
- 라이브러리로 표현할 수 없는 복잡한 비즈니스 로직이 있다면 커스텀 Kotest Matcher를 만들어 보는 것도 좋겠다.

> 복습: 함수형 인터페이스의 조건은 SAM이다. 자바는 `@FunctionalInterface`으로 컴파일러에게 이를 명시하는 걸 권장한다. 반면 코틀린은 SAM일 때 정의상 함수형 인터페이스라고 말할 수 있지만, 이를 람다로 구현하려면 `fun interface`라고 명시적으로 지정해주어야 한다.

<br>

### `kotlinx.datetime` 라이브러리: 날짜 리터럴

```kotlin
val now = Clock.System.now()
val yesterday = now - 1.days
val later = now + 5.hours
```
- `days, hours`는 `Int` 타입의 확장 프로퍼티이다.
- 두 시점 사이 시간 간격을 표현하는 `Duration` 타입을 반환한다.

<br>

### Exposed: 멤버 확장을 사용한 데이터베이스 질의 DSL

```kotlin
object Country : Table() {
    val id = integer("id").autoIncrement()
    val name = varchar("name", 50)
    override val primaryKey = PrimaryKey(id)
}
```
- Exposed 프레임워크에서 SQL로 테이블을 다루려면 Table 클래스를 확장해 정의한다.

```kotlin
fun main() {
    val db = Database.connect("jdbc:h2:mem:test", driver = "org.h2.Driver")
    transaction(db) {
        SchemaUtils.create(Country)
    }
}
```
- 테이블을 만들려면 트랜잭션과 함께 스키마 유틸 메서드를 호출한다. 이 코드는 SQL문을 만들어낸다.

<br>

```kotlin
class Table {
    fun integer(name: String): Column<Int>
    fun varchar(name: String, length: Int): Column<String>
    // ...
}
```
- 테이블 클래스는 데이터베이스 테이블에 대해 정의할 수 있는 모든 타입을 정의한다.

<br>

```kotlin
val id = integer("id").autoIncrement().primaryKey()
```
- 각 칼럼의 속성을 지정할 때, 멤버 확장이 쓰인다.
- Column에 대해 autoIncrement 같은 메서드를 호출해 속성을 지정할 수 있다. **각 메서드는 자신의 수신 객체를 다시 반환**하기 때문에, **메서드를 연쇄 호출**할 수 있다.

<br>

```kotlin
class Table {
    fun Column<Int>.autoIncrement(): Column<Int>
    ...
}
```
- 위 함수는 테이블 클래스의 멤버이기는 하지만 여전히 Column의 확장 함수이기도 하다.
- 이것이 이런 메서드를 멤버 확장으로 정의해야 하는 이유를 보여준다. 메서드가 적용되는 범위를 제한해야 하기 때문이다. 테이블이라는 맥락이 없으면 컬럼의 프로퍼티를 정의해도 아무 의미가 없다.
- 또한 수신 객체 타입을 제한할 수 있다. 테이블 안의 어떤 컬럼이든 기본 키가 될 수 있지만 자동 증가 컬럼이 될 수 있는 건 정수 타입인 컬럼뿐이다.

#### cf. 근데 실제로는 주석으로만 제한되어 있다! 데이터베이스 논리까지 컴파일러가 잡아주지는 않는다는 사실 🤔

```kotlin
/**
* Make @receiver column an auto-increment column to generate its values in a database.
* **Note:** Only integer and long columns are supported (signed and unsigned types).
* Some databases, like PostgreSQL, support auto-increment via sequences.
* In this case a name should be provided using the [idSeqName] param and Exposed will create a sequence.
* If a sequence already exists in the database just use its name in [idSeqName].
*
* @param idSeqName an optional parameter to provide a sequence name
*/
fun <N : Any> Column<N>.autoIncrement(idSeqName: String? = null): Column<N> =
    cloneWithAutoInc(idSeqName).also { replaceColumn(this, it) }
```

주석 두 번째 줄을 보면 이를 알 수 있다.

<br>

```kotlin
object User : Table() {
    val id = integer("id").autoIncrement()
    val nickname = varchar("nickname", 20).autoIncrement()
}
```
직접 확인해보기 위해 위와 같은 코드를 작성해 보았는데, 이 코드에도 아무런 경고가 뜨지 않았다. 두 번째 라인은 결국 SQL 수준에서 실패하지 않나?  
유연함을 위해 이렇게 설계했다는데, 아직 완전히 와닿지는 않는다.  
데이터베이스 같은 외부 논리를 컴파일러가 모를 수밖에 없지~ 라기에는 얘는 도메인 특화 언어 아닌가요?!

<br>

> **gemini의 첨언**  
> Exposed는 내부 DSL이다. 내부 DSL은 코틀린 언어의 문법적 제약(컴파일러의 타입 검사) 안에서 작동하기에, 외부 DSL처럼 SQL 규칙 전체를 완벽하게 통합하기 어렵다. SQL 규칙은 **의미론적 제약**이기 때문이다!
> 이러한 유연성을 준 이유는, Column<Int>와 Column<Long>처럼 여러 숫자 타입을 모두 처리하는 코드를 한 번만 작성하고 싶었기 때문일 것. 이러한 유연성은 라이브러리 작성자의 생산성을 높인다.

<br>

결론. DSL은 편리하지만 마냥 의존해서는 안 된다.  DSL을 사용할 때는 **외부 도메인이 가진 자체적인 논리에 더욱 신경을 써야겠다**. 그것이 개발자의 책임. 테스트를 꼼꼼히 하는 것도 좋은 방법일 것이다!