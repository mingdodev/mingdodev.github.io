---
layout: post
title:  "[Kotlin in Action] 3장. 함수 정의와 호출"
author: "minseo"
categories: [blog, java]
tags: [Kotlin, Java]
comments: true
image: "/static/img/logo/kotlin-logo.png"
hide_image: true
---

# [Kotlin in Action] 3장. 함수 정의와 호출

<br><br><center><img src="../../../static/img/logo/kotlin-horizontal.png" width="30%"></center>
<br><br><br>

> 도서 <코틀린 인 액션 2/e>를 읽으며 코틀린에 대해 이해한 내용을 정리한 글이다.

<br>

3장에서는 자바와 다른 코틀린 함수 정의의 특징와 편리한 함수 호출에 대해 학습한다. 특히 확장 함수와 확장 프로퍼티를 이해하고 잘 활용할 수 있도록 연습한다.

<br>

---


## 3.1 코틀린의 컬렉션

- 코틀린의 컬렉션 역시 자바 컬렉션 프레임워크로 구현되어있다.
- 코틀린 컬렉션은 인터페이스 이름에 따라 읽기 전용과 변경 가능한 것으로 구별할 수 있다.

### 읽기 전용

| 종류 | 인터페이스/타입 |
|------|----------------|
| 리스트 | `List<T>` |
| 집합   | `Set<T>` |
| 맵     | `Map<K, V>` |
| 시퀀스 | `Sequence<T>` |

### 변경 가능

| 종류 | 인터페이스/타입 |
|------|----------------|
| 리스트 | `MutableList<T>` |
| 집합   | `MutableSet<T>` |
| 맵     | `MutableMap<K, V>` |

<br>

```kotlin
val list = listOf(1, 2, 3)
val map = mapOf(1 to "one", 3 to "three", 5 to "five")
```
> 그럼 list는 val로, mutablelist는 var로만 선언해야 하는가?  
> 그게 자연스럽지만, 컬렉션 내 원소가 아닌 객체에 대한 참조 자체를 바꿔야 한다면 읽기 전용 컬렉션도 var로 선언할 수도 있겠다.

<br>

---

## 3.2 코틀린의 편리한 함수 호출

자바와 달리 코틀린에서 지원하는 편리한 함수 호출 방식에 대해 알아보자.

### 3.2.1 이름 붙인 인자 (Named Variable)

```kotlin
hello("Cat", 3, "Seoul") // 위치 기반 인자 전달
hello(name = "Cat", city = "Seoul", age = 3) // 이름 붙인 인자
```

- 이름 붙인 인자로 가독성 및 실수를 개선할 수 있다.
- 위치 기반 인자 전달도 가능하다.

### 3.2.2 디폴트 파라미터

```kotlin
fun hello(name: String, age: Int = 0, city: String = "Unknown") { ... }
```

- 디폴트 파라미터를 사용해 오버라이딩이 많아지는 문제를 개선한다.
- 파라미터 값을 지정하지 않으면 디폴트 값이 들어간다.

> 다른 언어와 달리 디폴트 값이 있는 인자들이 뒤에 와야 할 필요는 없지만, 이 경우 디폴트 값을 사용하려면 위치 기반 인자를 활용하기 애매해진다.

### 3.2.3 최상위 함수와 최상위 프로퍼티

#### [최상위 함수]

- 정적인 유틸리티 클래스를 없앤다.

    - 모든 메소드가 클래스 안에서 동작해야 하는 자바 언어에서는, 다양한 곳에서 사용되거나 API를 크게 만들고 싶지 않은 경우에도 클래스를 만들어 사용해야 한다 → 특별한 상태나 행위 없이 정적인 메서드만 있는 클래스가 생겨남

    - 마찬가지로 해당 패키지를 임포트해야 하지만, 불필요하게 클래스로 감싸지는 게 없어진다.

- 클래스가 있어야 컴파일 가능한 JVM 환경에서 실제로는 어떻게 실행되는가?

    - 해당 파일명 (e.g. `join.kt` → `joinKt`) 으로 클래스를 만들어 컴파일한다. 이름을 명시적으로 지정하고 싶으면 `@file:JvmName` 애노테이션을 사용하면 된다.

    > 자바와의 호환은 대체로 애노테이션이 처리하는 모습을 볼 수 있다.

#### [최상위 프로퍼티]

- 최상위 프로퍼티로 상수를 정의할 수 있다. `const val = public static final`

    > val은 final과 같은 효과. const로 최상위 프로퍼티를 정의하면 전역적으로 쓸 수 있으니 static으로 보는 듯하다.

### 3.2.4 확장 함수와 확장 프로퍼티

- 특정 클래스 밖의 함수/프로퍼티를 특정 클래스의 인스턴스 메서드/프로퍼티로 쓸 수 있게 한다.

#### [확장 함수]

```kotlin
fun String.lastChar(): Char = this.get (this.length - 1)
```

- `수신객체.메서드명`
    - `this` 생략가능

- **확장이라는 개념은 캡슐화를 깨지 않는다**. 확장 함수 내에서 내부 메서드/프로퍼티 다 사용할 수 있지만 `private, protected`는 불가능하다.
- 수신객체 입장에서는 확장함수나 멤버 메서드나 다 똑같은 메서드이다.

- 내부적으로 정적 메서드로 구현된다. 첫 번째 인자가 수신객체.
    - 따라서 자바에서도 쉽게 호환 가능하다.

> 하나의 파일 내에 여러 같은 이름이 있다면 임포트시 as 키워드로 이름을 바꿔 부르자. 길게 임포트 경로를 써서 구분하는 것보다는 짧게 쓰는 것이 코딩 컨벤션.

#### 확장 함수는 오버라이드 할 수 없다

- **확장 함수는 클래스의 일부가 아니기 때문**이다.
- 상속 관계와 다르게, 확장 함수는 타겟 클래스의 밖에 선언된다.
- 실제 호출될 함수는 확장 함수 호출 시 **수신 객체로 지정한 변수의 컴파일 시점의 타입**에 결정되지, 실행 시간에 그 변수에 저장된 객체의 타입에 의해 결정되지 않는다.

```kotlin
fun View.showOff() {
    println("I'm a view!")
}
fun Button.showOff() {
    println("I'm a button!")
}
fun main() {
    val view: View = Button() // 실제 런타임에는 변수에 Button 객체가 할당되겠지만,
    view.showOff() // 확장 함수는 정적 바인딩이므로 View.showOff()가 호출된다.
}
```

> cf. 확장 함수의 이름과 멤버 함수의 이름이 같다면 멤버 함수가 우선이다.

#### [확장 프로퍼티]

- 확장 함수와 마찬가지로 확장 프로퍼티를 사용하면 **함수가 아니라 프로퍼티 형식의 구문으로 사용할 수 있는 API**를 추가할 수 있다. <span style="color:#737373; font-size:14px; font-weight:300;"> 짧아서 편하다 </span>
- 단, 기존 자바 인스턴스에 필드를 추가할 방법이 없기에 실제로 **확장 프로퍼티는 아무 상태도 가질 수 없다**.

```kotlin
var StringBuilder.lastChar: Char
    get() = get(length - 1)
    set(value: Char) {
        setCharAt(length - 1, value)
    }
```

- 필드가 없어 기본 게터, 세터 구현이 제공되지 않는다. 따라서 직접 정의해야 한다.

> StringBuilder는 var 가변 객체라 setter를 만들 수 있다는 것도 한 번 생각해주기. String은 안 된다!

#### 🤔 확장 프로퍼티가 확장 함수에 비해 지니는 장점이 짧다 밖에 없나?

둘은 말 그대로 프로퍼티냐, 함수냐가 차이점이다. 이는 코드의 **역할**을 파악할 수 있게 돕는다. 예를 들어 방금 나온 마지막 글자에 대한 동작은 어떤 행동보다는 **속성**에 가깝다. <span style="color:#737373; font-size:14px; font-weight:300;"> 세터는 메서드 같긴 한데, 원래 게터와 세터를 추상화해서 가지고 있는 게 프로퍼티다 </span> 반면 메서드는 코드를 읽었을 때, 얘가 상태 변화나 부수적 효과를 가져올 것이라고 기대할 수 있다.

<br>

---

## 3.3 컬렉션 처리

### [가변 인자 함수]

```java
printf(String format, Object... args)
```

- 자바에서는 `...`를 사용하며, 배열에 들어있는 원소들은 그대로 넘길 수 있다.

```kotlin
// 표준 라이브러리의 listOf()
fun listOf<T> (vararg values: T): List<T> { /* ... */ }
```
- 가변 길이 인자 `vararg`를 사용하면 가변적인 인자의 개수만큼 배열에 그 값들을 넣어준다.
- 코틀린은 배열에 들어있는 원소들을 풀어서 하나씩 전달해야 한다. 따라서 배열 앞에 스프레드 연산자 `*`를 붙여야 한다.

### [중위 함수 호출과 구조 분해 선언]

```kotlin
1.to("one")
1 to "one"
```

- **인자가 하나뿐인** 일반 메서드/확장 함수에 중위 호출을 사용할 수 있다.

```kotlin
infix fun Any.to(other: Any) = Pair(this, other)
```
- 중위 호출을 사용하려면 infix 변경자를 함수 선언 앞에 추가한다.

```kotlin
val (number, name) = 1 to "one"
```
- 구조 분해 선언을 통해, 생성된 쌍(튜플)이 두 변수를 즉시 초기화한다.
- `Pair, Map` 등 다양하게 사용 가능.

<br>

---

## 3.4 문자열과 정규식 다루기

### [파싱에 특화된 코틀린의 여러 메서드들]

- 코틀린 문자열은 자바 문자열과 똑같다.

```kotlin
"12.345-6.A".split('.', '-')
"12.345-6.A".split("\\.|-".toRegex())
```
- Regex를 받는 자바의 `split()`에 더해 문자열을 받아 더 직관적으로 문자열을 나눌 수 있는 `split()`을 제공한다.
- 심지어 여러개의 구분자를 받을 수 있다.

### [3중 따옴표 문자열]

- `"""`을 사용하면 어떤 문자도 이스케이프할 필요가 없다.
- 줄바꿈도 그대로 들어간다.
- 문자열 템플릿을 쓰고 싶으면 내포 식 `${}`을 쓰자.

> 자바의 문자열 블록과 거의 유사, 대신 자바는 문자열 템플릿을 쓰려면 `String.format()`을 써야 하고, """ 뒤 줄바꿈이 필요했다.

<br>

---

## 3.5 로컬 함수와 확장

### 로컬 함수로 중복을 피하면서도 읽기 쉬운 코드 만들기

- DRY 원칙을 피하려다 보면 메서드가 흩어져서 오히려 읽기 어려울 때가 있다.
- 코틀린의 **로컬 함수**를 통해 추출한 함수를 원래 함수에 포함시킬 수 있다.

<br>

```kotlin
class User(val id: Int, val name: String, val address: String)
// 만약 private로 만들고 싶다면 private val로 선언해야 함

fun User.validateBeforeSave() {
    fun validate(value: String, fieldName: String) {
        if (value.isEmpty()) {
            throw IllegalArgumentException("Can't save user $id: empty $fieldName")
        }
    }
    validate(name, "Name")
    validate(address, "Address")
}

fun saveUser(user: User) {
    user.validateBeforeSave()
}
```
- `validate`라는 로컬 함수를 만들어 비어있는지 확인하고 예외를 던지는 로직을 재사용한다.
- 이 함수는 오직 `validateBeforeSave`에서만 필요하므로, 다른 곳에 노출되지 않고 원래 함수에 캡슐화된다.