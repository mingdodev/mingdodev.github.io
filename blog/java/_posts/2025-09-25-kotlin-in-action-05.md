---
layout: post
title:  "[Kotlin in Action] 5장. 람다를 사용한 프로그래밍"
author: "minseo"
categories: [blog, java]
tags: [Kotlin, Java]
comments: true
image: "/static/img/logo/kotlin-logo.png"
hide_image: true
---

# [Kotlin in Action] 5장. 람다를 사용한 프로그래밍

<br><br><center><img src="../../../static/img/logo/kotlin-horizontal.png" width="30%"></center>
<br><br><br>

> 도서 <코틀린 인 액션 2/e>를 읽으며 코틀린에 대해 이해한 내용을 정리한 글이다.

<br>

5장에서는 람다를 활용해 어떻게 코틀린의 간결함을 실현하는지 살펴보자. 코틀린의 함수형 프로그래밍에 대해서도 고민해보자.

<br>

---

## 5.1 람다식과 멤버 참조

코틀린에서는 람다식과 멤버 참조를 사용해 코드 조각과 행동 방식을 함수에게 전달한다.

### 람다
- **코드 블록을 값으로 다루는 것**을 말한다.
- "이벤트가 발생하면 이 핸들러를 실행하자", "데이터 구조의 모든 원소에 이 연산을 적용하자"와 같은 상황에서 람다는 유용하다.
- 람다는 메서드가 하나뿐인 익명 객체(함수형 인터페이스의 구현) 대신 사용할 수 있다.

#### 자바 8 이전에는? 익명 (내부) 클래스를 통한 함수형 인터페이스 구현

```java
public static void main(String[] args) {
    Thread thread = new Thread(new Runnable() {
        @Override
        public void run() /* ... */
    });
}
```
자바 8 이후에는 람다 표현식으로 다음과 같이 작성할 수 있다.

```java
Thread thread = new Thread(() -> {
    /* ... */
});
// 함수를 매개변수로 전달했다. 이 경우는 Thread 생성자 중 Runnable을 인자로 받는 생성자에 람다식으로 Runnable을 넘긴 것이다. 컴파일러의 타입 추론으로 Runnable 타입을 명시하지 않아도 된다. 만약 모호한 경우가 생기면 명시해야 할 것.
```
함수를 값처럼 다룬다. 람다식을 사용하면 함수를 선언할 필요가 없다.

### 람다와 컬렉션

```kotlin
{ x: Int, y: Int -> x + y }
```

- 코틀린 람다식은 항상 중괄호로 둘러싸여 있다.

```kotlin
fun main() {
    val people = listOf(Person("mingdo", 23), Person("doming", 22))
    println(people.maxByOrNull { it.age })
    // 함수의 인자가 람다일 때는 괄호 생략 가능. 함수 레퍼런스로도 쓸 수 있음. people.maxByOrNull(Person::age)
}
```
- 람다를 이용하면 컬렉션의 좋은 메서드들을 잘 활용할 수 있다.
- 파라미터의 타입 추론이 가능할 때 디폴트 파라미터로 `it`라는 이름을 사용할 수 있다.

> 함수 호출 시 맨 뒤 인자가 람다식이면 그 람다를 괄호 밖으로 빼낼 수 있다.
 람다가 어떤 함수의 유일한 인자이고 괄호 뒤에 람다를 썼다면 호출 시 빈 괄호를 없애도 된다.


```kotlin
fun main() {
    run { println(123) }
}
```
- 인자로 받은 람다를 실행해 주는 라이브러리 함수 `run`

#### 코틀린과 자바 람다의 다른 점

- 코틀린 람다 안에서도 자신을 둘러싼 영역에 정의된 변수에 접근할 수 있다. 파일 영역까지 가능.
- 자바 람다 내에서는 외부 final 변수에만 접근 가능하며, 읽기만 가능하다. 그러나 코틀린 람다 안에서는 파이널 변수가 아닌 변수에 접근하여 값을 변경할 수 있다.

<br>

람다 안에서 접근할 수 있는 외부 변수를 **람다가 캡처한 변수**라고 부른다.

함수가 반환되면 로컬 변수의 생명주기가 끝나는 게 일반적이다. 그러나 어떤 함수가 자신의 로컬 변수가 필요한 람다를 반환하거나 다른 변수에 저장한다면, 함수의 생명주기가 끝나더라도 로컬 변수의 생명주기는 이어져야 하는 경우가 생긴다. 이때 람다와 함께 로컬 변수를 '캡처'하여, 함수가 끝나도 람다의 본문 코드는 여전히 캡처한 변수에 접근할 수 있게 만든다.

지역변수는 스택에 저장되므로 함수가 반환되면 사라져버리니 접근할 수 없다. 따라서 이 변수를 '캡처'하여 힙에 따로 저장해두는 방식을 사용한다.

자바에서는 파이널 변수만 캡처할 수 있다. 자바는 변수의 **복사본**을 캡처하여 힙에 저장해둔다. 따라서 복사본의 값이 변경되면 원본과 달라져버리기 때문에, 읽기만 가능한 파이널 변수만 캡처 가능한 것이다.

코틀린도 파이널 변수(val)를 캡처할 때는 람다 코드와 함께 **복사한** 변수 값을 저장한다. 그러나 람다가 변경 가능한 변수(var)를 캡처하려 할 때에는 변수를 **Ref 클래스 인스턴스**에 넣고, 그 인스턴스에 대한 참조를 파이널로 만들어 캡처한다. 이렇게 하면 람다 내부에서 Ref 인스턴스의 필드로서 변수를 변경할 수 있게 된다.

### 멤버 참조

```kotlin
val getAge = { p: Person -> p.getAge() }

val getAge = Person::age
```
- 단순히 프로퍼티/메서드 하나만 꺼내서 쓰겠다는 의도가 드러난다.
- 간결하다.

<br>

```kotlin
fun hi() = println("hi")
fun bye() = println("bye")

fun main() {
    val functions = listof(::hi, ::bye)
    functions.forEach { it() }
}
```
- 최상위 함수나 프로퍼티를 참조할 수도 있다.

<br>

```kotlin
val createPerson = ::Person
```
- 위와 같은 문법으로 생성자 참조도 가능하다.

<br>

```kotlin
mingdo::name
```
- 클래스의 멤버뿐만 아니라 특정 객체 인스턴스의 메서드 호출에 대한 참조를 만들 수도 있다.

<br>

---

## 5.2 SAM: 함수형 인터페이스

SAM이란, **Single Abstract Method**로 **함수형 인터페이스**의 근본 개념을 지칭한다.

### 자바의 함수형 인터페이스

- 람다의 파라미터는 인터페이스의 유일한 추상 메서드의 파라미터와 대응한다.
- 자바 메서드 중 함수형 인터페이스를 받는 경우, 코틀린의 깔끔란 람다를 전달할 수 있다.
- 함수형 인터페이스를 명시적으로 구현하는 익명 객체를 만들어 넘길 수도 있으나 이 경우 매번 새로운 각체가 생성된다. 람다를 사용하면 람다에 해당하는 익명 객체가 재사용된다. <span style="color:#737373; font-size:14px; font-weight:300;"> 그때그때 다른 인스턴스를 써야 하기 때문 </span>

### SAM 변환

```kotlin
fun createAllDoneRunnable(): Runnable {
    return Runnable { println("All done!")}
}
```

- 함수형 인터페이스를 구현한 클래스의 인스턴스를 반환해야 하는 경우(create 메서드 같은) 사용한다.
- 함수형 인터페이스 이름이랑 똑같다.

### 코틀린에서의 SAM 인터페이스 정의

```kotlin
fun interface IntCondition {
    fun check(i: Int): Boolean
    fun checkString(s: String) = check(s.toInt())
}
```

<br>

---

## 5.3 수신 객체 지정 람다

- 수신 객체를 명시하지 않고 람다의 본문 안에서 다른 객체의 메서드를 호출할 수 있게 한다.
- 코틀린 표준 라이브러리로 `with, apply, also`를 쓸 수 있다.

### with 함수

- 첫 번째 인자로 객체를, 두 번째 인자로 람다를 받는다.
- 첫 번째 인자를 람다의 수신 객체로 만든다.

```kotlin
fun alphabet() = with(StringBuilder()) {
    for (letter in 'A'..'Z') { // this: StringBuilder. this를 써도 되고 생략도 가능
            append(letter)
        }
        append("\nfinish")
        toString() // 결과 반환
}
```

### apply 함수

- with와 거의 동일하지만 **항상 자신에게 전달된 객체를 반환**한다.

```kotlin
fun alphabet() = StringBuilder().apply {
    for (letter in 'A'..'Z') { // this: StringBuilder. this를 써도 되고 생략도 가능
            append(letter)
        }
        append("\nfinish")
}.toString()
```
- 반환값도 StringBuilder
- **인스턴스를 만들면서 즉시 프로퍼티 중 일부를 초기화해야 하는 경우** 사용할 수 있다.

> 자바에서의 Builder 객체처럼!

### also 함수

- 수신 객체에 대해 어떤 동작(로깅, 디버깅, 값 변경 등)을 수행한 후 수신 객체를 돌려준다.
- apply와의 달리 람다 안에서 수신 객체를 인자로 참조한다. 파라미터 이름을 부여하거나 it을 사용해야 한다.
- 원래의 수신 객체를 인자로 받는 동작을 실행할 때 유용하다.

```kotlin
fun main() {
    val name = "Alice"
        .also { println("Original: $it") }
        .uppercase()
        .also { println("Uppercased: $it") }

    println("Result = $name")
}
// Original: Alice
// Uppercased: ALICE
// Result = ALICE
```

### cf. 람다를 받아, 내부에서 수신 객체를 참조할 수 있게 하는 스코프 함수 5가지

| 함수        | 수신자 참조             | 반환값       | 주 용도                          | 예시                                                            |
| --------- | ------------------ | --------- | ----------------------------- | ------------------------------------------------------------- |
| **let**   | `it`               | 람다 결과     | **변환(transform), 안전 호출 후 연산** | `val len = str?.let { it.length }`                            |
| **run**   | `this`             | 람다 결과     | **객체 초기화, 여러 연산 묶기**          | `val txt = person.run { "$name@$company" }`                   |
| **with**  | `this` (인자로 객체 전달) | 람다 결과     | **여러 프로퍼티 접근 시 간결화**          | `with(address) { println(streetAddress); "$city, $country" }` |
| **apply** | `this`             | **객체 자신** | **빌더 패턴, 초기화**                | `val p = Person().apply { name="Kim"; age=20 }`               |
| **also**  | `it`               | **객체 자신** | **사이드 이펙트 (logging, 디버깅)**    | `user.also { println("created $it") }`                        |

- **스코프 함수** 또는 **영역 함수**라고 부른다.
- 이 함수들은 **코드 블록을 어떤 객체의 맥락에서 실행**해준다.
- 람다 안에서 대상 객체를 어떻게 접근하는지와 반환값이 무엇인지에 따라 구분해서 사용할 수 있다.