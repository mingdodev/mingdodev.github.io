---
layout: post
title:  "[Kotlin in Action] 11장. 제네릭스"
author: "minseo"
categories: [blog, java]
tags: [Kotlin, Java]
comments: true
image: "/static/img/logo/kotlin-logo.png"
hide_image: true
---

# [Kotlin in Action] 11장. 제네릭스

<br><br><center><img src="../../../static/img/logo/kotlin-horizontal.png" width="30%"></center>
<br><br><br>

> 도서 <코틀린 인 액션 2/e>를 읽으며 코틀린에 대해 이해한 내용을 정리한 글이다.

<br>

**제네릭**은 클래스, 인터페이스, 메서드에서 사용할 데이터 타입을 구체적으로 지정하지 않고, 외부에서 지정할 수 있도록 하는 문법이다.
자바에서와 같이, 코틀린 역시 **타입 소거**로 인해 런타임에서는 타입 정보가 사라진다. 그럼에도 불구하고 코틀린은 **컴파일 타임에 타입 안정성**을 보장하며 제네릭을 다루는 고유한 방식을 제공한다. 어떤 것들이 있는지 알아보자.

<br>

---

## 11.1 제네릭 함수와 클래스를 정의하는 방법

코틀린에서 제네릭을 정의하는 방법을 알아보자.

### 제네릭 타입 파라미터

제네릭 타입 파라미터를 받아 인스턴스를 만드는 컬렉션 중 리스트를 예시로 들어 설명한다.

```kotlin
val authors = listOf("cat", "dog", "fox")

val readers: MutableList<String> = mutableListOf()
// 또는
val readers = mutableListOf<String>()
```
- 제네릭 역시 초기화를 통해 컴파일러의 타입 추론을 이용할 수 있다.
- 빈 리스트를 만들어야 한다면 **변수** 또는 **변수를 만드는 함수**의 타입 인자를 지정한다.


### 제네릭 함수

다양한 타입의 원소를 저장하는 리스트뿐 아니라, 이를 다룰 수 있는 함수도 필요하다.  
제네릭 함수는 다음과 같이 쓴다.

```kotlin
fun <T> List<T>.slice(indices: IntRange): List<T>
```
- `fun` 키워드 바로 뒤에 `<T>`처럼 타입 파라미터를 선언한다.
- 역시 타입인자를 지정하거나 타입 추론을 사용할 수 있다.
- 확장 함수에서는 수신 객체나 파라미터 타입에 타입 파라미터를 사용할 수 있다.

> cf. 확장함수로 사용한다면, 자바 컬렉션 프레임워크 기반에 메서드를 덧붙여서 쓸 수 있다!

### 제네릭 확장 프로퍼티

```kotlin
val <T> List<T>.penultimate: T // 마지막에서 두 번째 요소
    get() = this[size - 2]
```
- 제네릭 확장 프로퍼티를 선언할 수 있다.
- 확장 프로퍼티는 값을 저장하지 않고 계산해서 반환하기 때문이다. [상태를 저장하지 않는다.](https://mingdodev.github.io/blog/java/2025-09-20-kotlin-in-action-03/#확장-프로퍼티)
- 일반 프로퍼티는 타입 파라미터를 가질 수 없다. 클래스 프로퍼티에 여러 타입의 값을 저장할 수 없기 때문이다. 클래스 프로퍼티는 **값을 저장**해야 하기 때문에, 런타임 객체 생성 시점에 어떤 타입일지 파악이 되어야 메모리를 확보할 수 있다.

### 제네릭 클래스 선언

```kotlin
interface List<T> {
    // 인덱스 접근에 대한 연산자 오버로딩
    operator fun get(index: Int): T 
    // ...
}
```
- 타입 파라미터를 넣은 홑화살표 구문 `<>`을 클래스나 인터페이스 이름 뒤에 붙이면 본문에서 일반적인 타입처럼 사용 가능하다.

<br>

---

### 타입 파라미터 제약

제네릭 클래스나 함수는 사용할 수 있는 타입 인자를 제한할 수 있다.

```kotlin
// 합계는 숫자 타입에만 유효하므로 이런 경우 타입을 제한하여 목적에 맞게 사용할 수 있다.
fun <T : Number> List<T>.sum(): T
```
- 파라미터 뒤에 상계를 지정함으로써 제약을 정의할 수 있다.
- 타입 인자는 반드시 이 **상계 타입**이거나 이 **상계 타입의 하위 타입**이어야 한다.

> cf. 자바에서는 `<T extends Number> T sum(List<T> list)`처럼 extends를 쓴다.

<br>

```kotlin
fun <T> ensureTrailingPeriod(seq: T)
    where T : CharSequence, T : Appendable {
        /*...*/
    }
```
- 여러 개의 제약을 가해야 하는 경우는 위와 같이 쓴다.

#### 타입 파라미터를 널이 될 수 없는 타입으로 명시

```kotlin
class Processor<T : Any> {
    fun process(value: T) {
        value.hashCode()
    }
}
```
- 아무런 상계를 정하지 않은 타입 파라미터는 `T: Any?`와 같다.
- 널이 될 수 없는 타입을 사용해 상계를 정하면 타입 파라미터를 널이 아닌 타입으로 제약할 수 있다.
    - 필요하다면 `Int?`처럼 널이 될 수 있는 타입으로 상계를 정하되, 내부 널 처리가 필요하다.

<br>

```kotlin
// JBox<T>: Java
class KBox<T>: JBox<T> {
    override fun put(t: T & Any) { /*...*/ }
    override fun putNotNull(t: T) { /*...*/ }
    fun otherMethod(t: T) { /*...*/ }
}
```
- 상속 등 자바와의 상호운용이 필요할 때는, 제네릭 파라미터를 정의한 최초 위치가 아니라 **타입을 사용하는 지점에서 널이 될 수 없음을 표시**한다.
    - 최초 위치에서 `KBox<T : Any> : JBox<T>`처럼 제한할 수도 있다. 그러나 이 경우 코틀린 클래스만의 기능에서도 널을 다룰 수 없어진다.
- `t: T & Any`와 같이 다른 언어에서의 교집합 타입같은 형태로 쓴다.

<br>

---

## 11.2 실행 시점에서 제네릭스의 동작

- JVM의 제네릭스는 **타입 소거**를 통해 구현된다. 런타임에는 제네릭 클래스의 인스턴스에 타입 정보가 들어있지 않다.

## 타입 소거

```kotlin
fun readNumbersOrWords(): List<Any> {
    val input = readln()
    val words: List<String> = input.split(",")
    val numbers: List<Int> = words.mapNotNull { it.toIntOrNull() }

    // numbers가 비어있으면 words 반환, 아니면 numbers 반환
    return if (numbers.isNotEmpty()) numbers else words
}

fun printList(list: List<Any>) {
    when (list) {
        is List<String> -> println("Strings: $list")
        is List<Int> -> println("Integers: $list")
        // Error: Cannot check for an instance of erased type
    }
}

fun main() {
    val list = readNumbersOrWords()
    printList(list)
}
```
- 저장해야 하는 타입 정보의 크기가 줄어 메모리를 덜 쓴다.
- 그러나 위와 같이 실행 시점에 들어오는 타입에 따른 동작을 구현할 수 없다.

<br>

```kotlin
fun printSum(c: Collection<*>) {
    // 여기서 경고: Unchecked cast: List<*> to List<Int>
    val intList = c as? List<Int> ?: throw IllegalArgumentException("List<Int> 필요")
    println(intList.sum())
}
```
- **인자를 알 수 없는 제네릭 타입을 표현할 때**는 스타 프로젝션 `*`을 쓴다. (자바 `?`와 유사)
- 원소에 담긴 것이 무엇인지 모르지만 제네릭 타입을 사용하는 자료구조가 컬렉션임을 알 수 있다.
- 어느 타입이 들어와도 요소로 받기 때문에 `as`를 통한 캐스팅이 언제나 성공한다.
    - 컴파일러가 타입에 대해 알지 못하기 때문에 경고는 띄워주지만, 컴파일은 성공한다.
- 위 코드같은 경우 리스트 내 원소를 다룰 때 `sum()`이 숫자 객체로 다루려고 하여, `String`같은 타입이 들어올 경우에는 `ClassCastException`이 발생한다.

## 실체화된 타입 파라미터

- 코틀린은 함수를 인라인으로 선언하여 타입 인자가 지워지지 않게할 수 있다. 즉 **타입 파라미터를 실체화**할 수 있다.
- 이에 따라 **타입 인자를 실행 시점에 언급**할 수 있다.

### cf. 인라인 함수란?

```kotlin
inline fun greet() = println("Hello!")

fun main() {
    greet() // -> println("Hello!")
}
```
- `inline` 키워드를 붙여 선언한다.
- 컴파일러가 함수 호출식을 그 함수의 구현 코드로 바꾼다.
- **호출 오버헤드가 사라진다**는 장점이 있다.

<br>

```kotlin
inline fun repeatTwice(action: () -> Unit) {
    action()
    action()
}

fun main() {
    repeatTwice { println("Hi") }
}
```
- **람다를 인자로 넘기는 함수에서 익명 클래스와 객체가 생성되지 않아** 특히 더 성능이 좋다.
- 인라인이 아니라면 내부적으로 익명 클래스를 정의하고, 함수(람다로 전달된)를 담는 객체가 생성이 되지만 인라인이라면 그냥 함수 실행 코드이기 때문

<br>

인라인 함수의 또 다른 장점 하나가 바로 타입 인자 실체화이다.

```kotlin
fun <T> isA(value: Any) = value is T
// error
```
- 위 코드는 에러가 발생한다.

<br>

```kotlin
inline fun <reified T> isA(value: Any) = value is T
```
- 그러나 위와 같이 인라인 함수로 정의하고, `<reified T>`로 타입 인자가 실체화되었음을 나타내면 실행 시점에도 타입이 지워지지 않아 **타입 검사**가 가능하다. <span style="color:#737373; font-size:14px; font-weight:300;"> reified: 실체화된, 구체화된 </span> 

> 표준 라이브러리에서 이를 사용한 대표적인 예시가 `fileterIsInstance`

### 동작 원리

- 컴파일러는 인라인 함수 본문을 구현한 바이트코드를 그 함수가 호출되는 모든 지점에 삽입한다.
- 이때 컴파일러는 타입 정보를 가지고 있으므로, 타입 인자로 쓰인 구체적인 클래스를 참조하는 바이트 코드를 생성해 삽입할 수 있다.

<br>

자바에서는 인라이닝이 불가해 코틀린의 인라인 함수를 일반 함수처럼 호출한다. **자바 코드를 실체화된 타입 파라미터와 함께 쓸 경우는 오류가 발생할 수 있다.**

따라서 **실체화된 타입 파라미터**가 필요하거나, **성능 향상**이 필요한 경우에만 인라인 함수를 사용하자. 또한 함수가 너무 커지지 않게, 실체화 타입에 의존하지 않는 부분은 뽑아내자.

<br>

---

## 11.3 변성(Variance)

변성은 기저 타입이 같고 타입 인자가 다른 여러 타입이 서로 어떤 관계가 있는지 설명하는 개념이다.  
예를 들어, `List<String>`과 `List<Any>` 사이 관계를 다룬다.

> 기저 타입이란 제네릭 클래스∙인터페이스에서 유효한 개념으로, 제네릭 타입에서 타입 인자를 제거했을 때 남는 "틀"을 말한다.

### 변성은 인자를 함수에 넘겨도 안전한지 판단하게 해준다

```kotlin
fun addAnswer(list: MutableList<Any>) {
    list.add(42)
}

fun main() {
    val strings = mutableListOf("ab", "bc")
    addAnswer(strings)
    println(strings.maxBy { it.length })
}
```
- 이 식이 컴파일이 된다면, 실행 시점에 `ClassCastException`이 발생할 것이다.
- 즉, `List<Any>` 타입의 파라미터를 받는 함수에 `List<String>`을 넘기면 안전하지 않다. 위처럼 원소 추가와 같은 **변경이 일어날 수 있기 때문**이다.

- 그러나 리스트의 변경이 없는 경우에는 안전하다.
- 따라서 **읽기 전용 리스트**를 받는다면, 위와 같이 더 구체적인 타입의 원소를 갖는 리스트를 함수에 넘길 수 있다.

<br>

이렇게 "읽기만 가능한 기저 타입은 하위 타입을 넘겨도 안전하다", "쓰기도 가능한 기저 타입은 예외가 발생할 수 있기 때문에 막아야 한다" 등을 정의하는 것이 **변성**의 개념이다. 변성은 코드에서 **위험할 여지가 있는 메서드를 호출할 수 없게 만들어**, 제네릭 타입의 인스턴스 역할을 하는 클래스 인스턴스를 잘못 사용하는 일이 없게 방지한다.

<br>

---

<br>

다음으로 **변성을 표시하는 방법**에 대해 알아보자.  
변성을 일반화하기 전, 타입과 하위 타입이라는 개념을 알아야 한다.

### cf. 타입과 하위 타입, 클래스

- **타입과 클래스의 차이**

    - 클래스는 객체를 만들기 위한 설계도, 타입은 변수에 담길 수 있는 값들의 집합.
    - 클래스는 하나지만 그로부터 여러 타입이 파생될 수 있다. (널 가능성, 기저 타입 등)

- **타입과 하위 타입**

    - 어떤 타입 A의 값이 필요한 모든 장소에 어떤 타입 B의 값을 넣어도 아무런 문제가 없다면 타입 B는 타입 A의 하위 타입이다.
    - 컴파일러는 변수 대입이나 함수 인자 전달 시 하위 타입 검사를 매번 수행한다.

<br>

- 대부분 하위 타입은 하위 클래스와 근본적으로 같다.
- 널이 될 수 없는 타입은 널이 될 수 있는 타입의 하위 클래스가 아니지만, 하위 타입이다.
- 제네릭 타입에서는 다음과 같은 개념을 사용한다.

    - **공변**: A가 B의 하위 타입이면 `List<A>`가 항상 `List<B>`의 하위 타입인 경우
    - **무공변**: 서로 다른 두 타입 A, B에 대해 `MutableList<A>`가 항상 `MutableList<B>`의 하위 타입도 아니고 상위 타입도 아닌 경우

> 하위 클래스는 상속의 개념이고, 하위 타입은 클래스 상속뿐 아니라 인터페이스 구현, 공변∙반공변 규칙 등을 다 포함하는 더 넓은 개념이다.

<br>

### 공변성: 하위 타입 관계를 유지한다

```kotlin
interface Producer<out T> {
    fun produce(): T
}
```
- 제네릭 클래스가 타입 파라미터에 대해 공변적임을 표시하려면 타입 파라미터 이름 앞에 `out`을 넣는다.
- 공변성을 표시해주지 않으면, 아무리 하위 타입이 들어와도 `Producer<A>`와 `Producer<B>`는 하위 타입의 관계가 아니다. 따라서 타입 에러가 발생한다.

```kotlin
class Herd<out T : Animal> {
    val size: Int get() = /* ... */
    operator fun get(i: Int): T { /* ... */ }
}
```
- 그렇다고 모든 클래스를 공변적으로 만들 수는 없다. 공변적으로 선언할 경우, 컴파일러는 타입 파라미터가 쓰이는 위치를 제한한다.
- 타입 안전성을 보장하기 위해, 공변적 파라미터는 항상 **아웃** 위치에 있어야 한다. 클래스가 T 타입의 값을 생산할 수는 있지만, 소비할 수는 없다.

    - **인**: 함수의 파라미터 (소비)
    - **아웃**: 함수의 반환 값 (생산)
    - 생성자 파라미터는 둘 중 어느쪽도 아니라, 자유롭게 사용 가능

> 이를 PECS (Producer-Extends, Consumer-Super)라고 한다. 이 원칙은 자바에서도 통용되는 개념이지만, 자바는 문법의 강제성이 없어 개발자가 스스로 지켜야 한다.

<br>

### 반공변성: 하위 타입 관계를 뒤집는다

- **반공변**: A가 B의 하위 타입이면 `List<B>`가 `List<A>`의 하위 타입인 경우

```kotlin
interface Comparator<in T> {
    fun compare(e1: T, e2: T): Int { /* ... */ }
}
```

- `in` 키워드를 사용한다.
- 타입 파라미터가 인 위치에서만 쓰인다. 즉 타입의 값을 소비하기만 한다.

<br>

```kotlin
open class Fruit
class Apple : Fruit()
class Pear : Fruit()

interface Consumer<in T> {
    fun consume(item: T)
}
// Consumer<Fruit>은 Consumer<Apple>의 하위 타입이 된다.
// 사과만 먹을 수 있는 소비자보다, 과일을 먹을 수 있는 소비자가 더 범용적이고 안전하기 때문!
```
- **하위 타입 관계**를 뒤집는다.
- 타입의 값을 소비할 때 사용하므로, 특정 타입보다 더 일반적인 타입에 대해 소비할 수 있는 함수를 만드는 게 안전하기 때문이다.

<br>

### 선언 지점 변성과 사용 지점 변성

- 자바의 와일드카드 타입과 같이 타입 파라미터가 있는 타입을 사용할 때마다 그걸 상위 타입이나 하위 타입 중 어떤 타입으로 대치할 수 있는지 명시하는 방법을 **사용 지점 변성**이라고 한다.

- 클래스를 선언하면서 변성을 지정하면 그 클래스를 사용하는 모든 곳에 영향을 끼칠 수 있다. 이런 방식을 **선언 지점 변성**이라 한다.

    - 쓰는 쪽에서 코드가 간결해진다.

<br>

- 코틀린에서도 사용 지점 변성을 쓸 수 있다.
- 타입을 소비하는 동시에 생산할 수 있는 인터페이스/클래스가 존재하는데, 특히 어떤 함수 안에서는 생산자 또는 소비자 중 하나의 역할만 담당할 때 사용할 수 있다.

```kotlin
fun <T> copyData(source: MutableList<T>, destination: MutableList<T>) {
    for (item in source) {
        destination.add(item)
    }
}
```
- 여기서 원본 컬렉션은 읽기만, 대상 컬렉션은 쓰기만 한다는 사실이 보인다.

```kotlin
fun <T> copyData(source: MutableList<out T>, destination: MutableList<T>) {
    for (item in source) {
        destination.add(item)
    }
}
```
- 위와 같이 MutableList라는 클래스를, 사용 시점에 `out`을 통해 공변성을 지정할 수 있다!
- `in`도 넣으면 상위 타입까지 커버하니, 좀 더 안전하게 사용할 수 있음.
- 이런 변경자를 붙이면, **타입 프로젝션**이 일어나 컴파일러가 타입 파라미터 T를
in 또는 out 위치(붙인 변경자에 따라)에서 사용하지 못하게 막는다.

<br>

### 스타 프로젝션 `*`을 사용할 때 주의할 점

*는 제네릭 타입 인자 정보가 없음을 표현할 때 사용한다.
타입 파라미터를 시그니처에서 언급하지 않거나 데이터를 읽기는 하지만 타입은 무관할 때 쓴다.

> 모른다고 아무거나 다 담아도 된다는 뜻이 아니다!

<br>

#### [타입 인자를 알 수 없으므로 구체적인 타입으로는 사용 불가]

- `MutableList<*>`는 `MutableList<Any?>`와 같지 않다.

    - 후자는 모든 타입의 원소를 담을 수 있음을, 프로젝션은 어떤 정해진 구체적인 타입만을 담을 수 있지만 그 원소의 타입을 정확히 모름을 표현한다.

- 예를 들어, 컴파일러 입장에서는 validate(input: T)에서 T가 뭔지 모르니까, String이든 Int든 아무 것도 안전하게 넣을 수 없다. 그래서 FieldValidator<*> 타입 변수에서는 validate("문자열") 같은 호출이 막힌다.

#### [읽기는 가능하지만, 쓰기는 안전하지 않음]

- `MutableList<*>`에서 원소를 꺼낼 때는 `Any?`로 받을 수 있다.

- 하지만 원소를 추가하려 하면 컴파일 에러가 발생한다. 어떤 타입인지 알 수 없으니 안전하지 않기 때문.

#### [캐스팅이 필요할 때가 있음 (unsafe cast)]

- 특정 타입 전용 동작을 하려면 `as FieldValidator<String>` 같은 캐스팅을 해야 한다.

- 이 경우 컴파일러는 경고(unchecked cast)를 주고, 잘못 캐스팅하면 런타임에 ClassCastException이 터질 수 있다. (7장에서 언급했듯)

#### [맵/컨테이너에 섞어 담을 때 특히 위험]

- `mutableMapOf<KClass<*>, FieldValidator<*>>()`처럼 여러 타입의 밸리데이터를 한 곳에 모아두면, 꺼낼 때마다 타입 정보가 지워져 있어서 캐스팅이 필요해진다.
- 따라서 잘못된 캐스팅으로 인한 런타임 오류 가능성이 있다.

<br>

### 타입 별명

- 여러 제네릭 타입을 조합한 타입(복잡한 제네릭 타입, 함수형 타입 등)을 다룰 때, **기존 타입에 다른 이름을 부여**해 편리하게 사용할 수 있다.

```kotlin
typealias NameCombiner = (String, String, String, String) -> String

val authorsCombiner: NameCombiner = { a, b, c, d -> "$a et al." }
val bandCombiner: NameCombiner = { a, b, c, d -> "$a, $b & The Gang" }

fun combineAuthors(combiner: NameCombiner) {
    println(combiner("aaa","bbb","ccc","ddd"))
}
```
- `typealias` 키워드 뒤에 별명을 적어 선언한다.
- 이는 단지 원래의 타입으로 치환될 뿐 타입 안전성을 추가하지는 못한다. 타입 안전성이 필요하다면 인라인 클래스를 사용하여 타입을 검사하라!

> 새로운 개발자가 코드를 읽을 때 정의를 찾아 이해하는 데 시간이 소비될 수 있다는 트레이드 오프 등을 생각하고 쓰면 될 것~