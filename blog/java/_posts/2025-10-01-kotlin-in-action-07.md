---
layout: post
title:  "[Kotlin in Action] 7장. 널이 될 수 있는 값"
author: "minseo"
categories: [blog, java]
tags: [Kotlin, Java]
comments: true
image: "/static/img/logo/kotlin-logo.png"
hide_image: true
---

# [Kotlin in Action] 7장. 널이 될 수 있는 값

<br><br><center><img src="../../../static/img/logo/kotlin-horizontal.png" width="30%"></center>
<br><br><br>

> 도서 <코틀린 인 액션 2/e>를 읽으며 코틀린에 대해 이해한 내용을 정리한 글이다.

<br>

7장에서는 드디어 코틀린이 널을 안전하게 다루는 방식이 등장한다. 코틀린은 **널이 될 수 있는 타입을 명시**하며, **널 가능성이 있는 값을 다루기 위한 특별한 문법들**을 제공한다. 여기에 어떻게 자바 코드와 상호 운용이 가능한지까지 함께 살펴보자!

<br>

---

## 7.1 널이 될 수 있는 타입

- **널 가능성(nullability)**

    - `NullPointerException`를 피할 수 있는 코틀린 타입 시스템의 특성이다.
    - 널이 될 수 있는지 여부를 타입 시스템에 추가하여, 실행 시점에서의 `NPE`를 막는다.

### 목적: 널이 될 수 있는 변수 명시

프로그램 내 프로퍼티나 변수에 null을 사용하려면 **널이 될 수 있는 타입임을 명시**해야 한다.

```kotlin
fun strLenSafe(s: String?) = ...
```
- 타입 이름 뒤에 물음표를 붙이면 그 타입의 변수나 프로퍼티에 null 참조를 저장할 수 있다.
- 널이 될 수 있음을 명시하지 않으면 컴파일 타임에 에러가 발생하고, 런타임에도 해당 변수에 널을 저장할 수 없다.

<br>

### 장점: 안전하고 편리한 널 처리

널이 될 수 있는 타입을 명시함으로써, 값이 없는 경우에 대해 예외 없이 안전하게 처리할 수 있도록 보장한다.

- 널이 될 수 있는 타입의 값은 그 값에 대해 수행할 수 있는 연산의 종류가 제한된다.

    - 그 값의 메서드를 직접 호출할 수 없다.
    - 그 값을 널이 될 수 없는 타입의 변수에 대입하거나 널이 아닌 타입의 파라미터를 받는 함수에 전달할 수 없다.

<br>

**널과의 비교**를 통해 일단 널이 아님이 확인되면, 컴파일러는 그 사실을 기억하고 **널이 아닌 것이 보장되는 영역에서는 해당 값을 널이 아닌 타입의 값처럼 사용**할 수 있다.

```kotlin
fun strLenSafe(s: String?): Int =
    if (s != null) s.length else 0

fun main() {
    val x: String? = null
    println(strLenSafe(x))
    // 0
    println(strLenSafe("abc"))
    // 3
}
```
- null 검사를 추가하면 해당 코드는 컴파일된다.

<br>

---

## 7.2 타입이란?

> 타입이란 무엇이고 왜 변수에 타입을 지정해야 하는가?

1976년, 데이빗 파나스는 타입을 **가능한 값의 집합과 그런 값들에 대해 수행할 수 있는 연산의 집합**이라고 정의했다.

### 🤔 자바의 타입 시스템은 널을 제대로 다루지 못한다.

- 참조 타입 변수의 경우, 해당 타입과 null이라는 완전히 다른 두 종류의 값이 들어갈 수 있다.
- null이 들어갈 경우 그 변수에 대해 어떤 연산을 수행할 수 있을지 알 수 없다.
- 프로그래머의 잘못된 확신으로 널 검사를 생략하면 NPE가 발생한다.

대신, NPE 오류를 다루기 위한 다음과 같은 방법들을 사용해 왔다.

<br>

**1. `@Nullable`, `@NotNull` 어노테이션으로 널이 될 수 있는지의 여부를 표시**

- IDE를 활용해 NPE 발생의 여지가 있는 코드를 찾아준다.
- 표준 API는 아니다. 표준 자바 컴파일 절차의 일부도 아니기 때문에 일관성 있게 적용된다는 보장을 할 수 없다.

**2. Optional 래퍼 타입 사용**

- null 값을 코드에서 절대로 쓰지 않기 위해, 어떤 값이 정의되거나 정의되지 않을 수 있을 때 **옵셔널(Optional)**로 이를 감싸 처리한다.
- 그러나 옵셔널은 가독성을 해치며, 래퍼가 추가됨에 따라 런타임 성능이 저하된다.
- 일관성 있게 처리하기 어렵다. 자바 생태계 전체가 옵셔널을 사용하지는 않기 때문에 외부 코드로부터 오는 null을 처리해야 한다.

> 자바 진영에서도 엘비스 연산자(널이면 대체 값을 쓰자)에 대한 논의가 있었으나 이를 도입하면 null을 더 많이 사용할 것이라 판단하여 승인되지 않았다고 한다. 다만 최근 널을 다루는 타입을 표기하자는 [드래프트](https://openjdk.org/jeps/8303099?utm_source=chatgpt.com)는 나왔다~~

<br>

코틀린은 널이 될 수 있는 타입과 널이 아닌 타입을 구분하기에, **각 타입의 값에 대해 어떤 연산이 가능한지 명확히 이해**할 수 있고, **런타임에 예외를 발생시킬 수 있는 연산을 판단할 수 있어 이를 아예 금지**할 수 있다.

<br>

---

## 7.3 코틀린이 널이 될 수 있는 타입을 다루지만 불편하지 않은 이유

> 널이 등장할 수 있다는 것 자체가 불편한 거 아닌가요?!  
> 아닙니다. 오히려 널 가능성에 대해 명확히 인지하고 있으면 예방이 가능하겠죠!

### 안전한 호출(safe call) 연산자 `?.`

```kotlin
val allCaps: String? = str?.uppercase()
// if (str != null) str.uppercase() else null
```

- `?.` 연산자는 널 검사와 메서드 호출을 한 연산으로 수행한다.
- 호출하려는 값이 null이 아니면 일반 메서드 호출처럼 작동하고, null이라면 호출값은 무시되고 결괏값은 null이 된다.
    - 즉 안전한 호출의 결과 타입도 널이 될 수 있는 타입이다.

```kotlin
val country = this.company?.address?.country
```

- 널이 될 수 있는 중간 객체가 여럿 있을 때 진정한 효과를 보인다. <span style="color:#737373; font-size:14px; font-weight:300;">  중첩 if문을 없앤다!</span>

<br>

### null에 대한 기본값을 제공하는 엘비스(elvis) 연산자 `?:`

```kotlin
val recipient: String = name ?: "unnamed"
println("Hello, $recipient!")
```

- `?:` 연산자는 널 검사 후 null 대신 사용할 기본값을 편리하게 지정한다. 
- 왼쪽 값이 널이면 오른쪽을 실행해서 결과를 쓴다.

```kotlin
fun strLenSafe(s: String): Int = s?.length ?: 0
```

- 안전한 호출 연산자와 엘비스 연산자를 함께 사용해, 객체가 null인 경우에 대비한 기본값을 지정할 수 있다.

<br>

### 안전한 캐스트(safe-cast) 연산자 `as?`

대상 값을 as로 지정한 타입으로 바꿀 수 없으면 `ClassCastException`이 발생한다. 코틀린은 예외를 방지하기 위한 타입 검사와 타입 변환을 한 번에 수행하는 연산자를 지원한다.

```kotlin
class Person(val firstName: String, val lastName: String) {
    override fun equals(o: Any?): Boolean {
        val otherPerson = o as? Person ?: return false // cf1.

        return otherPerson.firstName == firstName &&
            otherPerson.lastName == lastName
    }

    /* hashcode override ... */
}

fun main() {
    val p1 = Person("Dmitry", "Jemerov")
    val p2 = Person("Dmitry", "Jemerov")

    println(p1 == null) // cf2. == 연산자로 equals 메서드 호출
    // false
    println(p1 == p2)
    // true
    println(p1.equals(42)) // 타입이 맞지 않는 경우
    // false
}
```
- `as?` 연산자는 대상 값의 타입을 검사한 후 지정한 타입으로 변환할 수 있다면 변환하고, 그럴 수 없다면 null을 반환한다.

- **cf1**. **제어 구문 `return`은 expression으로 사용할 수 있다.**

    - 엘비스 연산자의 오른쪽에는 표현식만 올 수 있다. `return, throw, break, continue`는 제어를 옮기면서 함수/스코프를 끊어버리는 표현식으로 취급되기 때문에 저 자리에 올 수 있는 것이다. 실제 동작은 엘비스 연산자의 왼쪽이 널이니 오른쪽을 실행하는데, 오른쪽이 return 제어 구문이니 함수 끝.

- **cf2.** **코틀린 `==`의 내부 동작**

    - 코틀린 `==`는 널 안전한 연산자이다. 내부적으로 왼쪽 값에 대한 널 검사를 먼저 실시하고 equals를 호출한다. `a == b` 에서 왼쪽이 null이면 양쪽 다 null인지 비교하고, 왼쪽이 널이 아니면 `a.equals(b)`를 호출한다.

<br>

### 널 아님 단언(not-null assertion)연산자 `!!`

- 컴파일러에게 어떤 값이 실제로는 null이 아니라는 사실을 알려준다.

- 어떤 값이든 널이 아닌 타입으로 강제로 바꿀 수 있다.

    - null에 대해 적용하면 NPE가 발생한다.

    - 예외가 발생한다면 null 값을 사용하는 코드가 아닌 단언문이 위치한 곳을 가리킨다.

    > 스택 트레이스는 라인 정보만 주고 어떤 식에서 예외가 났는지 알려주지는 않으므로, 편한 디버깅을 위해 한 줄에 여러 !!를 쓰지 말자.

```kotlin
class SelectableTextList (
    val contents: List<String>,
    var selectedIndex: Int? = null,
)

class CopyRowAction(val list: SelectableTextList) {
    fun isActionEnabled(): Boolean =
        list.selectedIndex != null

    fun executeCopyRow() {
        val index = list.selectedIndex!!
        val value = list.contents[index]
    }
}
```
- 이 예제에서는 `isActionEnabled`가 `true`인 경우에만 `executeCopyRow`를 호출한다고 가정한다.
    - 즉 `executeCopyRow`가 실행될 때, list.selectedIndex`가 `null`이 아님은 당연하다.
    - 컴파일러는 이걸 모르지만 개발자는 알고 있다.

- **호출된 함수가 언제나 다른 함수에서 널이 아닌 값을 전달받는** 사실이 분명할 때, 널 아님 단언문을 쓸 수 있다.

<br>

---

### let 함수

```kotlin
fun sendEmailTo(email: String) { /*...*/ }
fun main() {
    val email: String? = "foo@bar.com"
    email?.let { sendEmailTo(it) }
}
```
- let 함수를 **안전한 호출 연산자**와 함께 사용하면, 널이 될 수 없는 인자만 받는 함수에 널이 될 수 있는 값을 넘길 수 있다.
- let 함수는 자신의 수신 객체를 인자로 전달받은 람다에 넘긴다. 이때 수신 객체(`email`)가 널이면 아무일도 일어나지 않고, 널이 아닐 때만 전달 받은 람다를 실행한다.

> 인스턴스가 널이면 호출이 불가능한 메서드와 다르게, 널이 될 수 있는 타입의 값에 대해 안전한 호출을 사용하지 않고 let을 사용하면 람다의 인자는 널이 될 수 있는 타입으로 추론된다!

<br>

- 아주 긴 식이 있고 그 값이 널이 아닐 때 수행해야 하는 로직이 있는 경우 사용하면 좋다.
- 여러 값을 검사할 때는 let 호출을 내포시켜 처리할 수 있지만, 가독성 측면에서 좋지 않을 수 있다.

> cf. with, apply, let, run, also 모두 스코프 함수이다.

<br>

### 지연 초기화 프로퍼티

- 코틀린에서는 일반적으로 생성자에서 모든 프로퍼티를 초기화해야 한다.
- 프로퍼티 타입이 널이 될 수 없다면 반드시 널이 아닌 값으로 그 프로퍼티를 초기화해야 한다.
    - 그러나 생성자에서 초기화하지 못하는 경우도 있다. 이에 프로퍼티를 널 가능으로 선언하면 널 검사 및 단언문으로 코드가 더러워진다.

이 문제를 해결하기 위해 코틀린은 **지연 초기화(late-initialize)**를 제공한다.

```kotlin
/* 클래스 생성자에서 초기화하지 않고 JUnit의 @BeforeAll에서 초기화하는 예시 */

class MyService {
    fun performAction(): String = "Action Done"
}

@TestInstance(TestInstance.Lifecycle.PER_CLASS)
class MyTest {
    // 코틀린에서 널이 될 수 없는 타입의 멤버 변수는 반드시 선언과 동시에 초기화되어야 함
    private lateinit var myService: MyService // 초기화하지 않고 널이 아닌 프로퍼티를 선언

    @BeforeAll fun setUp() {
        myService = MyService() // 별도의 널 검사 없이 프로퍼티 사용
    }

    @Test fun testAction() {
        assertEquals("Action Done", myService.performAction())
    }
}
```
- 생성자 밖에서 값을 바꿔야하므로 지연 초기화 프로퍼티는 항상 var여야 한다.
- 프로퍼티 초기화 전에 접근하면 `NPE`가 아닌 `UninitialiedPropertyAccessException`이 발생한다.

<br>

**cf. 스프링 DI 중 필드 주입**

```kotlin
@Service
class MyService {
    @Autowired
    private lateinit var myController: MyController
    
    fun someLogic() {
        myController.doWork() 
    }
}
```
- 그치만 주로 생성자 주입으로 불변성을 유지할 것이니, 참고만~

<br>

### 널이 될 수 있는 타입에 대한 확장 함수 정의

- 메서드 호출 전 수신 객체 역할을 하는 변수가 널이 아니라고 보장하는 대신, 메서드 호출이 널을 수신 객체로 받고 내부에서 널을 처리하게 할 수 있다.
- 널이 될 수 있는 타입의 확장함수는 자신의 수신 객체가 널일 때 어떻게 해야 하는지 스스로 알고 있기 때문에, 안전한 호출 없이도 호출 가능하다.

> 확장 함수가 내부적으로 정적 함수처럼 처리되어 객체(인스턴스)를 통한 디스패치에 의존하지 않고, null이 그대로 함수에 인자로 전달될 수 있기 때문이다. 일반 객체는 디스패치를 통해 메서드를 호출할 때 널 검사를 안 하니, 수신 객체가 null이면 NPE가 발생한다.

<br>

```kotlin
fun String?.isNullOrBlank(): Boolean =
    this == null || this.isBlank() // isBlank는 널이 아닌 값에 대해서만 호출될 수 있음. 스마트 캐스트로 호출가능한 것
```
- 자바는 메서드 안의 this가 무조건 널이 아니지만, 코틀린에서는 **널이 될 수 있는 타입의 확장 함수 안에서는 this가 널이 될 수도 있다**는 점에 유의!

> 확장함수를 작성할 때 널이 될 수 있는 타입에 대해 정의할지 고민된다면, 일단 처음에는 널이 될 수 없는 타입에 대해 정의하라. 추후 필요할 때 안전하게 바꿀 수 있으므로~

<br>

### 타입 파라미터의 널 가능성

> 물음표가 붙어있지 않은 타입 파라미터가 널이 될 수 있는 타입일 수도 있다?!

- 함수나 클래스의 모든 타입 파라미터는 기본적으로 널이 될 수 있다.
- 타입 파라미터 T를 타입 이름으로 사용하면 물음표 없이도 널이 될 수 있는 타입이다.
    - 널이 될 수 있는 타입을 표시하려면 반드시 물음표를 타입 이름 뒤에 붙여야 한다는 규칙의 유일한 예외

```kotlin
fun <T> printHashCode(t: T) {
    println(t?.hashCode()) // 안전 호출에 의해 null 반환
}

fun main() {
    printHashCode(null) // T의 타입은 Any?로 추론된다.
    // null
}
```
- T에 대해 추론한 타입은 널이 될 수 있는 Any? 타입이다.
- **cf.** println() 함수는 널 허용 객체를 인자로 받았을 때, 그 객체가 실제로 null이면 콘솔에 문자열 "null"을 출력

```kotlin
fun <T: Any> printHashCode(t: T) { // T는 널이 될 수 없는 타입
}

fun main() {
    printHashCode(null) // 컴파일 에러
    printHashCode(42)
    // 42
}
```
- 타입 파라미터가 널이 아님을 확실히 하려면 널이 될 수 없는 타입 상계(upper bound)를 지정해야 한다.

<br>

---

## 7.4 널 가능성과 자바

널 가능성을 지원하지 않는 자바와 코틀린을 조합하면 어떤 일이 생길까?


### 1. 자바 어노테이션으로 표시된 널 가능성 정보를 인식

- 이런 정보가 코드에 있으면 코틀린도 그 정보를 활용한다.
- **e.g.** 자바의 `@Nullable String`은 코틀린의 `String?`, 자바의 `@NotNull String`은 코틀린의 `String`
- JSR-305 표준(자바 표준이 아님), 안드로이드, JetBrains 등이 지원하는 널 가능성 어노테이션들을 인식한다.

### 2. 플랫폼 타입

자바 코드로부터 널 가능성 정보를 인식할 수 없는 경우, 자바 타입은 코틀린의 **플랫폼 타입**이 된다. 플랫폼 타입은 코틀린이 널 관련 정보를 알 수 없는 타입을 말한다.

- 이 될 수 있는 타입으로 처리해도 되고 널이 될 수 없는 타입으로 처리해도 된다.
- 컴파일러가 모든 연산을 허용한다. _(like 자바)_
- 코틀린에서 플랫폼 타입을 선언할 수는 없고, 자바 코드에서 가져온 타입만 플랫폼 타입이 된다.

따라서 자바 API를 다룰 때는 조심해야 한다. 오류를 피하기 위해 사용하려는 자바 메서드의 문서를 참조하여 적절한 null 검사를 추가해야 한다.

> cf. 모든 자바 타입을 널이 될 수 있는 타입으로 다룬다면.. `널 안전성으로 얻는 이익 < 검사에 드는 비용`

<br>

---

<br>

코틀린과 자바를 혼합한 클래스 계층을 선언할 때 다음을 주의하자.

### 상속

```java
interface StringProcessor {
    void process(String value);
}
```
- 위 코드는 자바 인터페이스이다.

```kotlin
class StringPrinter : StringProcessor {
    override void process(value: String) {
        print(value)
    }
}

class NullableStringPrinter : StringProcessor {
    override void process(value: String?) {
        if (value != null) {
            print(value)
        }
    }
}
```
- 위 코드는 코틀린으로 구현한 클래스이다.
- 코틀린 컴파일러는 위 두 구현을 다 받아들인다.

- **코틀린에서 자바 메서드를 오버라이드**할 때 그 **메서드의 파라미터와 반환 타입**을 잘 결정해야 한다. (널이 될 수 있는 타입/널이 될 수 없는 타입)
    - 구현 메서드를 다른 코틀린 코드가 호출할 수 있으므로 컴파일러는 널이 될 수 없는 타입으로 선언한 모든 파라미터에 대해 널이 아님을 검사하는 단언문을 만들어준다. 그런데 자바 코드가 그 메서드에게 널을 넘기면 이 단언문이 발동해 예외가 발생하게 된다.