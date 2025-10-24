---
layout: post
title:  "[Kotlin in Action] 4장. 클래스, 객체, 인터페이스"
author: "minseo"
categories: [blog, java]
tags: [Kotlin, Java]
comments: true
image: "/static/img/logo/kotlin-logo.png"
hide_image: true
---

# [Kotlin in Action] 4장. 클래스, 객체, 인터페이스

<br><br><center><img src="../../../static/img/logo/kotlin-horizontal.png" width="30%"></center>
<br><br><br>

> 도서 <코틀린 인 액션 2/e>를 읽으며 코틀린에 대해 이해한 내용을 정리한 글이다.

<br>

4장에서는 코틀린에서 **인터페이스, 클래스**를 통해 **객체**를 다루는 방법을 공부한다. 코틀린이 자바의 어떤 부분들을 개선하는지, 어떻게 객체지향을 실현하는지에 집중하면서 읽었다.

<br>

---

## 4.1 클래스, 인터페이스

### 인터페이스

```kotlin
interface Clickable {
    fun click()
    fun showOff() = println("I'm clickable!")
}

class Button : Clickable {
    override fun click() = println("I was clicked")
}

fun main() {
    Button.click()
}
```

- inheritance, composition 모두 클래스 이름 뒤 콜론을 붙여 사용한다.

> 인터페이스 구현은 개수 제한이 없으나 클래스 extention은 하나만 가능

- `override` 변경자는 필수!

#### [디폴트 메서드]

- 메서드 본문을 구현하면 된다.
- 두 인터페이스에 동일한 이름의 디폴트 메서드가 있고 이 둘을 구현한 하나의 클래스가 있다면 둘 다 선택되는 것이 아니다. _(컴파일 오류)_ 하위 구현에서 **오버라이딩 메서드**를 직접 제공해야 한다.

> 자바 코드는 코틀린의 디폴트 구현을 사용할 수 없다. 코틀린 인터페이스를 구현하려면 모든 메서드 본문을 작성해야 한다. 코틀린은 자바 8 이전 버전과의 호환성을 위해, 인터페이스를 `추상 메서드 선언만 있는 인터페이스` + `디폴트 메서드를 정적 메서드로 구현한 코드가 있는 숨겨진 클래스`로 구현하기 때문이다. 자바는 이를 상속하면 숨겨진 클래스의 존재를 알지 못한다.

<br>

---

### 클래스

우선, open, final, abstract 변경자에 대해 알아보며 클래스를 정의하는 방법을 이해하자.

#### [변경자]

**(1) 기본 `final`**

- 코틀린에서 모든 클래스와 메서드는 기본적으로 final이다. 클래스간 상속, 하위 클래스의 오버라이딩이 불가능하다.

- 자바의 상속 시스템에서는 **기반 클래스 구현을 변경**함으로써 하위 클래스들이 잘못된 동작을 할 위험이 있기 때문이다. (취약한 기반 클래스)

> 이펙티브 자바: **상속을 위한 설계와 문서를 갖춰라. 그럴 수 없다면 상속을 금지하라.** 기반 클래스의 의도에 따라 메서드를 오버라이드 해야 하며, 오버라이드가 의도되지 않은 클래스와 메서드는 모두 final로 만들어야 한다.

<br>

**(2) 상속을 허용하는 `open`**

예를 들어, 단순한 버튼을 제공하는 UI를 개선해 클릭 가능한 RichButton을 만든다고 할 때, 하위 클래스는 자신만의 애니메이션을 제공하되, 버튼의 기본 기능을 깰 수는 없어야 한다. 이러한 요구 사항을 다음과 같이 코드로 나타낼 수 있다.

```kotlin
open class RichButton : Clickable {
    fun disable() { /* ... */ }
    open fun animate() { /* ... */ }
    final override fun click() { /* ... */ } // 인터페이스의 멤버를 오버라이드 한 경우에는 기본적으로 open
}

class ThemedButton : RichButton() { // class 상속은 ()을 붙여야 하나요?
    override fun animate() { /* ... */ }
    override fun showOff() { /* ... */ }
}
```

- 기반 클래스나 인터페이스의 멤버를 오버라이드 한 경우에는 기본적으로 `open`이다. 금지하고 싶다면 `final`을 명시해야 한다.

> 왜 오버라이드는 기본이 open일까? 이미 오버라이드가 되었다는 사실 자체가 해당 멤버는 오버라이드가 필요하다는 반증일까?  
> 그럴 것이다. 오버라이드가 된 멤버도 기본이 final이라면 다형성의 확장성이 제한된다.

<br>

**cf. 클래스 상속 문법**

```kotlin
class ThemedButton : RichButton()
```

- 여기서 괄호가 붙는 이유: 저것 자체가 RichButton 클래스의 생성자를 호출하는 문법이다.
- 만약 생성자가 파라미터를 필요로한다면 자식 클래스는 부모 생성자에 파라미터를 전달해야 한다!

<br>

**(3) 추상 클래스를 만드는 `abstract`**

- abstract로 선언한 추상 클래스는 인스턴스화할 수 없다.
- 하위 클래스에서 **반드시 오버라이드해야만 하므로**, 항상 열려 있다. 기본이 `open`

> 마찬가지, 인터페이스의 멤버도 기본이 `open`이고, `abstract`이며  `final`로 변경할 수 없다.

#### [가시성과 접근 제어자]

- 기본적으로 public이다.
- package 가시성이 없다.

> 자바의 패키지 전용 가시성은 프로젝트 외부의 코드라도 패키지가 같은 클래스를 만들기만 하면 패키지 내부 전용 선언에 접근할 수 있어, 캡슐화가 쉽게 깨질 수 있다는 단점이 있다.

<br>

| 변경자 | 클래스 멤버 | 최상위 선언 |
| :--- | :--- | :--- |
| **`public`** | 모든 곳에서 접근 가능 | 모든 곳에서 접근 가능 |
| **`internal`** | 동일한 모듈 내에서만 접근 가능 | 동일한 모듈 내에서만 접근 가능 |
| **`protected`** | 하위 클래스에서만 접근 가능 | (사용 불가) |
| **`private`** | 동일한 클래스 내에서만 접근 가능 | 동일한 파일 내에서만 접근 가능 |

- 라이브러리 개발 시 public은 외부 사용자를 위한 API에, internal은 내부 구현을 위한 코드로 사용할 수 있다. internal 코드를 변경해도 사용자 코드에 영향이 가지 않게 된다. 이 둘은 장기적인 유지보수성에 차이를 갖는 듯.

<br>

---

### 내포 클래스 (Nested Class)

자바는 클래스 안에 클래스를 만들면 **내부 클래스**로 생성되며, 이는 다음과 같은 문제를 만든다.

#### 내부 클래스 (Inner Class) vs. 내포 클래스 (Nested Class)

- **내부 클래스**: 바깥쪽 클래스의 인스턴스에 대한 암묵적인 참조를 가짐. 바깥쪽 클래스의 객체가 있어야만 존재 가능

- **내포 클래스**: 바깥쪽 클래스의 인스턴스와는 아무런 관계가 없음. 정적 클래스 같이!

내부 클래스는 바깥쪽 클래스에 대한 참조로 인해 원하는 행위 수행에 제약이 생긴다.

<br>

코틀린은 이를 해결하기 위해 기본적으로 모든 중첩 클래스를 **내포 클래스**로 간주한다. 자바와 달리 바깥쪽 클래스 인스턴스에 대한 접근 권한이 없다.

> 자바는 명시적으로 `static`을 통해 내포 클래스로 만들어야 한다.

내부 클래스를 만들기 위해서는 `inner class A`와 같이 명시하고, 외부 클래스의 참조에 접근하려면 `this@Outer`라고 쓴다.

<br>

---

### 봉인된 클래스/인터페이스: `sealed`

- 상속 가능한 하위 클래스들을 컴파일 시점에 미리 정의하고 제한하는 클래스이다.

- `when`식을 사용할 때 모든 가능한 경우를 완벽하게 처리하도록 강제하는 것이 주 목적이다.

    - `else` 분기는 모든 경우에 대응하기 때문에 의도와 다른 동작을 수행하여 오류를 만들 수 있다.
    - `sealed`를 사용하면 컴파일러는 이미 모든 하위 클래스를 알고 있기 때문에, `else` 분기가 필요 없으며 명확한 동작을 정의해 코드 안정성을 높일 수 있다.
    - 새로운 하위 클래스가 추가될 때, 변경이 필요한 부분을 컴파일러가 알려줄 수 있게 된다!

- 봉인된 인터페이스도 마찬가지로 모듈이 컴파일되고 나면 그에 대한 새로운 구현을 외부에서 추가할 수 없다.

<br>

---

## 4.2 생성자와 프로퍼티

### 주 생성자와 초기화 블록

```kotlin
class User(val nickname: String, val isSubscribed: Boolean = true)
```

- **주 생성자**
    - 클래스 이름 뒤 괄호로 둘러싸인 코드
- 목적
    - 생성자 파라미터 지정
    - 생성자 파라미터로 초기화되는 프로퍼티 정의
- 내부에서 `val` 키워드를 통해 프로퍼티를 정의할 수 있다.
- 생성자 파라미터에도 기본값을 정의할 수 있다.

> 모든 파라미터에 기본값을 정의하면 컴파일러가 자동으로 파라미터 없는 생성자를 만들어준다.

<br>

```kotlin
class User constructor(_nickname: String) {
    val nickname: String

    init {
        nickname = _nickname // 사실 이 경우는 초기화 블록이 필요 없음
    }
}
```

- **초기화 블록**
    - 클래스의 객체가 만들어질 때 실행되는 코드
    - 주 생성자와 함께 사용된다.

<br>

```kotlin
class Secretive private constructor(private val agentName: String) {}
```

- 비공개 생성자를 만들기 위해 `private constructor`를 사용할 수 있다.

> 접근 제어자나 애너테이션이 필요할 때는 `constructor` 키워드가 필요하다.

#### 🤔 자바에서 비공개 생성자가 필요한 경우를 코틀린은 어떻게 대처할까?

**1. 유틸리티 클래스의 인스턴스 생성 막기**

```java
// file: 'MathUtils.java'
public class MathUtils {
    private MathUtils() {}

    public static int add(int a, int b) {
        return a + b;
    }
}
```
- 자바: 인스턴스 생성을 막기 위해 private 생성자를 사용한다.

```kotlin
// file: 'math_utils.kt'
fun add(a: Int, b: Int): Int {
    return a + b
}
```
- 코틀린: 별도의 클래스 없이 파일 최상위에 함수를 선언한다.

<br>

**2. 싱글톤 클래스를 위해 인스턴스 생성 막기**

```java
// file: 'Singleton.java'
public class Singleton {
    private Singleton() {}

    private static final Singleton INSTANCE = new Singleton();

    public static Singleton getInstance() {
        return INSTANCE;
    }
}
```
- 자바의 경우 역시 private 생성자로 추가 인스턴스 생성을 막고, 클래스 내부에서 단 하나의 인스턴스만 생성하도록 작성한다.

```kotlin
// object 선언으로 간결하게 싱글턴 구현
object Singleton {
    fun doSomething() {
        // ...
    }
}
```

- 코틀린의 경우, `object` 키워드가 유일한 인스턴스를 생성해준다.

### 부 생성자

- 디폴트 파라미터와 named parameter로도 해결되지 않는, 여러 개의 생성자를 사용해야 하는 경우가 있다.

```kotlin
open class MyDownloader : Downloader {
    constructor(url: String?) : this(URI(url)) {
        // ...
    }

    constructor(uri: URI?) : super(uri) {
        // ...
    }
}
```

- 자바 상호운용성, 클래스 인스턴스를 생성할 때 파라미터 목록이 다른 생성 방법이 여럿 존재하는 경우을 위해 필요하다.
- 주 생성자가 없으면 부 생성자가 상위 클래스 초기화를 담당해야 한다.

<br>

---

### 추상 프로퍼티

- 인터페이스를 구현하는 클래스가 **추상 프로퍼티의 값을 얻을 수 있는 방법을 제공해야 한다**는 뜻이다.

    - **주 생성자**로, **본문 필드와 커스텀 게터**로, **프로퍼티 초기화 식**으로 구현할 수 있다.

    - 인터페이스에 게터, 세터가 있는 프로퍼티도 선언할 수 있는데, 이때는 하위 클래스가 이를 상속할 수 있다.

> 다시, **언제 함수 대신 프로퍼티를 사용할까?**  
> 예외를 던지지 않는 경우
> 계산 비용이 적게 들거나 최초 실행 후 결과를 캐시해 사용할 수 있는 경우
> 객체 상태가 바뀌지 않으면 여러 번 호출해도 항상 같은 결과를 돌려주는 경우

### 커스텀 세터

```kotlin
class user(val name: String) {
    var address: String = "unspecified"
        set(value: String) {
            println(
                """
                Address was changed for $name:
                "$field -> $value".
                """.trimIndent()  // field를 통해 뒷받침하는 필드 값에 접근할 수 있다.
            )
            field = value
        }
}

fun main() {
    ...
    user.address = "..." // 필드 설정 구문을 사용, 내부적으로 setter 호출
}
```

- 프로퍼티에 값을 저장하되 값에 접근할 때마다 정해진 로직을 실행하기

### 뒷받침하는 필드를 생성하지 않는 프로퍼티

- 그 자신이 아무 정보도 저장하지 않을 경우, 컴파일러는 필드를 생성하지 않는다.

### 비공개 접근자

- `private set`과 같이 외부에서 세터에 접근하지 못하도록 선언할 수 있다.

<br>

---

## 4.3 데이터 클래스와 클래스 위임

모든 클래스가 정의해야 하는 메서드 `toString, equals, hashCode`를 직접 구현해보자.

```kotlin
class Customer(val name: String, val postalCode: Int) {
    override fun toString() = "Customer(name=$name, postalCode=$postalCode)"
    override fun equals(other: Any?): Boolean {
        if (other == null || other !is Customer)
            return false
        return name == other.name && postalCode == other.postalCode
    } 
}
```

- 코틀린에서 `==` 연산자는 객체 동등성을 검사한다. 따라서` equals`를 호출하는 식으로 컴파일된다.

    - `===`로 참조 동일성을 검사한다.

```kotlin
fun main() {
    val processed = hashSetOf(Customer("고양이", 1234))
    println(processed.contains(Customer("고양이", 1234)))
    // false
}
```

- `equals()`가 true를 반환하는 두 객체는 반드시 같은 `hashCode()`를 반환해야 한다.
- processed 집합은 해시셋으로, 원소 비교 시 객체의 해시 코드를 비교하고 해시 코드가 같은 경우에만 실제 값을 비교한다.

```kotlin
class Customer(val name: String, val postalCode: Int) {
    /* ... */
    override fun hashCode(): Int = name.hashCode() * 31 + postalCode
}
```

### 데이터 클래스

- 데이터 클래스는 위 메서드들을 자동으로 생성한다. IDE의 힘을 빌리지 않더라도!

```kotlin
data class Customer(val name: String, val postalCode: Int)
```

- 주 생성자에 정의된 프로퍼티를 고려해 `equals, hashCode`를 만든다.
    - 모든 프로퍼티를 고려한다. 따라서 비즈니스 의미에 따라 변경해야 할 경우는 직접 오버라이드가 필요하다.
- 프로퍼티가 var이어도 되지만 val로 불변을 유지하는 걸 추천한다. 특히 컨테이너에 담기는 경우에는 프로퍼티 변경으로 해시값이 바뀌면 객체의 동등성 비교가 어려워진다!
- `copy()`메서드를 제공하여 불변성을 유지한다.

> 자바 레코드는 private final 프로퍼티여야 하며, 상위 클래스 상속이 불가, 본문 안에 프로퍼티 정의가 불가하다. 본문 안에 프로퍼티 안 두는 건 명확하고 좋은 듯?

### 클래스 위임: `by`

- `open`으로 변경이 가능한 부분에 대해 열어두었지만, 종종 상속을 허용하지 않는 클래스에 새로운 동작을 추가해야할 때가 있다. 보통 데코레이터 패턴을 사용해 새로운 메서드를 추가하는데, 이는 많은 준비 코드를 요구한다.

> 데코레이터 클래스가 기존 클래스와 동일한 인터페이스를 제공하고, 기존 클래스는 데코레이터 내부 필드로 유지한다. 새로운 동작은 데코레이터 메서드로 정의하고, 기존 동작은 데코레이터가 기존 클래스의 메서드에게 요청을 전달하여 수행한다.

<br>

```kotlin
class DelegatingCollection<T>(
    innerList: Collection<T> = mutableListOf<T>()
) : Collection<T> by innerList
```
- `by` 키워드를 통해 `Collection` 인터페이스에 대한 구현을 다른 객체에 위임 중이라고 명시할 수 있다! 원래라면 클래스 내에 기존 동작 전달용 메서드를 다 작성해야 했는데~~
- 또, innerList의 내부 동작에 의존하지 않으므로 내부 구현이 바뀌어도 영향을 받지 않는다.

<br>

---

## 4.4 object 키워드

- 클래스 선언과 인스턴스 생성을 한꺼번에 한다.

### 주로 사용되는 상황

#### 1) 객체 선언: 싱글턴 정의

```kotlin
object Payroll { // 사내 장부는 단 하나만 필요하니 싱글턴에 알맞다.
    val allEmployees = arrayListOf<Person>()

    fun calculateSalary() { /* ... */}
}
```

- 단 하나의 인스턴스만 필요한 경우, 구현 내부에 하나의 상태만 필요한 경우
- 생성자를 쓸 수 없다.
- 제공하고픈 초기 상태가 있으면 본문에서 직접 제공해야 한다.

> 싱글턴은 객체 생성을 제어할 수 없고 생성자 파라미터 지정이 불가하므로 객체의 의존관계를 바꿀 수 없다. 단위 테스트나 시스템 설정 변경 등에서 불리하므로 **일반 코틀린 클래스 + 의존 관계 주입**으로 사용해야 할 때도 있다.

- 내포 객체로 하나의 인스턴스를 사용할 수도 있다.

#### 2) 동반 객체(Companion Object): 팩토리 메서드와 정적 멤버가 들어갈 장소

자바의 static을 대신하기 위해 주로 최상위 함수(또는 객체 선언)를 쓰는데, 최상위 함수는 private에 접근할 수 없다는 문제가 있다. 대표적으로 팩토리 메서드는 객체 생성을 책임지기 때문에 클래스의 private 멤버에 접근할 수 있어야 한다.

```java
class User {
    private String name;

    protected User() {};

    public static String create(String name) {
        User user = new User();

        user.name = name;

        return user;
    }
}
```

- 이런 자바에서 흔히 볼 수 있는 팩토리 메서드..

```kotlin
class User private constructor(val name: String) {

    // 동반 객체는 클래스명.메서드() 형태로 호출 가능
    companion object {
        fun create(name: String): User {
            // private 생성자에 접근하여 User 객체 생성
            return User(name)
        }
    }
}
```

- 코틀린은 **비공개 생성자를 지닌 클래스 내에서 동반 객체를 통해** 팩토리 메서드를 정의할 수 있다.

- 동반 객체는 클래스와 함께 존재하는 유일한 객체이다. 이 객체는 클래스 인스턴스와 독립적으로 존재하며, 마치 자바의 static 멤버처럼 클래스 이름으로 접근할 수 있도록 해주는 역할을 한다.

    - 즉 자바와 다른점, 해당 클래스의 인스턴스는 동반 객체의 멤버에 접근할 수 없다.

```kotlin
class User {
    val nickname: String

    constructor(email: String) {
        nickname = email.substringBefore('@')
    }

    constructor(socialAccountId: Int) {
        nickname = getSocialNetworkName(socialAccountId)
    }
}
```
- 이렇게 부 생성자 두 개를 제공하지 않고,

```kotlin
class User private constructor(val nickname: String) {

    companion object {
        fun createWithEmail(email: String) =
            User(email.substringBefore('@'))
        fun createWithSocialAccountId(socialAccountId: Int) =
            User(getSocialNetworkName(socialAccountId))
    }
}
```

- private 생성자를 사용한 뒤 동반 객체로 팩토리 메서드를 만들면 안전하고, 필요 없는 객체 생성을 막을 수 있다.

- 다만 동반 객체 멤버는 하위 클래스에서 오버라이드할 수 없으므로, 클래스를 확장해야 하는 경우는 여러 생성자를 사용하는 게 더 낫다.

- 동반 객체는 클래스 내에 정의된 일반 객체다. 이름을 붙이거나 인터페이스를 상속하거나 내부에 확장 함수와 프로퍼티를 정의할 수 있다.

- **확장함수로 비즈니스 모듈이 특정 데이터 타입에 의존하는 것을 없애자**

    - 빈 동반 객체를 정의해두고, 해당 동반 객체에 대한 확장 함수를 정의해 비즈니스 모듈을 타입 바깥으로 뺄 수 있다.

#### 3) 객체 식: 익명 내부 클래스를 다른 방식으로 작성

```kotlin
fun main() {
    Button(object: MouseListener { // 임의의 MouseListener 구현을 생성해 버튼 생성자에 넘김
        override fun onEnter() ...
    })
}
```

- 객체 식을 사용해 익명 객체를 만들 수 있다.
- 식이 포함된 함수의 변수에 접근할 수 있다. (final이 아닌 변수도 가능)

> Java에서는 익명 클래스가 람다나 익명 클래스가 정의된 메서드 내의 지역 변수에 접근하려면 그 변수가 반드시 final이어야 한다. 이는 자바가 익명 클래스 객체를 생성할 때, 해당 변수의 값을 복사하기 때문. 만약 변수가 final이 아니라면, 나중에 값이 변경될 수 있어 복사된 값과 원본 값 사이에 불일치가 발생할 수 있기 때문에 컴파일러가 이를 막는다.

<br>

---

## 4.5 인라인 클래스

지출한 돈을 저장하는 메서드가 있을 때, 인자의 화폐 단위를 다양하게 안전하게 사용할 수 있을까?
모든 화폐를 저장하는 객체를 매번 만들기에는 비효율적이다.

```kotlin
@JvmInline
value class UsdCent(val amount: Int)
```

- 실행 시점에 이 클래스의 인스턴스는 감싸진 프로퍼티로 대체되다.
- 래퍼 타입으로 타입 안정성을 챙기면서도 불필요한 객체 생성을 막는다.
- 클래스가 프로퍼티를 하나만 가져야 하며, 이는 주 생성자에서 초기화되어야 한다.
- 인라인 클래스는 클래스 계층이 아니다. (클래스간 상속 불가) 인터페이스 상속, 메서드 정의, 계산된 프로퍼티는 가능
- 주로 기본 타입 값의 의미를 명확하게 하고 싶을 때 사용한다.