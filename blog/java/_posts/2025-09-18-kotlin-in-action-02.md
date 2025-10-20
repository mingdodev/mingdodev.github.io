---
layout: post
title:  "[Kotlin in Action] 1장. 코틀린이란 무엇이며, 왜 필요한가?"
author: "minseo"
categories: [blog, java]
tags: [Kotlin, Java]
comments: true
image: "/static/img/logo/kotlin-logo.png"
hide_image: true
---

# [Kotlin in Action] 2장. 코틀린 기초

<br><br><center><img src="../../../static/img/logo/kotlin-horizontal.png" width="30%"></center>
<br><br><br>

> 도서 <코틀린 인 액션 2/e>를 읽으며 코틀린에 대해 이해한 내용을 정리한 글이다.

<br>

2장에서는 코틀린에서 어떻게 **함수, 변수, 클래스, 그리고 프로퍼티를 정의하고 사용하는지**를 다룬다. 또한 조건문과 반복문 같은 제어 구조의 사용법을 익히고, 특히 **자바와 달리 문(statement)과 식(expression)이 어떻게 구분**되는지 살펴보며 코틀린의 간결한 코딩 스타일을 이해한다.

<br>

---

## 2.1 기본 요소: 함수와 변수

### 2.1.1 프로그램 진입점

```kotlin
fun main() {
    println("Hello, world!")
}
```

- 최상위 main 함수에 인자가 없어도 된다.
    - 문자열 배열(`args: Array<String>`)이 전달되기도 한다.
    - 이 경우 배열의 각 원소는 애플리케이션에게 전달된 각각의 커맨드라인 인자에 대응한다.
- 어떤 경우든 main 함수는 아무 값도 반환하지 않는다.
- 콘솔 출력은 `println`을 사용한다.
- 줄 끝에 세미콜론을 붙이지 않아도 된다.

### 2.1.2. 함수 선언

```kotlin
fun max(a: Int, b: Int): Int {
    return if (a > b) a else b
}
```

- 코틀린에서는 파라미터 이름이 먼저 오고 그 뒤에 파라미터의 타입을 지정한다.
- 반환 타입은 파라미터 목록 괄호 다음에 `:`으로 구분하여 작성한다.

> 코틀린은 if가 **결과를 만드는 식(expression)**이라는 점에 집중한다. 어쨌든 값을 반환하는 것이다.  
> 코틀린의 if 식은 자바 같은 다른 언어의 삼항 연산자와 비슷하게 사용한다.

#### cf. 코틀린에서의 문(statement)과 식(expression)의 구분

코틀린에서 if는 식이지 문이 아니다. **식은 값을 만들어내며 다른 식의 하위 요소로 계산에 참여**할 수 있는 반면, **문은 자신을 둘러싸고 있는 가장 안쪽 블록의 최상위 요소로 존재하며 아무런 값을 만들어내지 않는다**.

자바와 달리, **코틀린에서는 루프(for, while, do/while)를 제외한 대부분의 제어 구조가 식**이다. 제어 구조를 다른 식으로 엮어낼 수 있다는 특징은 여러 일반적인 패턴을 아주 간결하게 표현할 수 있다는 장점을 만든다.

### 2.1.3 식(expression) 본문을 사용해 함수를 더 간결하게 정의

```kotlin
// 식 본문 함수
fun max(a: Int, b: Int): Int = if (a > b) a else b

// 타입 추론: 컴파일러가 함수 본문 식을 분석해서 식의 결과 타입을 함수 반환 타입으로 정해준다
fun max(a: Int, b: Int): Int = if (a > b) a else b
```

- max 함수의 본문이 if 식 하나로만 이뤄져 있기 때문에, 이 식을 함수 본문 전체로 하고 중괄호를 없앤 후 return을 제거할 수 있다.
- 유일한 식은 등호(=) 뒤에 위치시킨다.
- 식 본문 함수는 반환 타입을 생략할 수 있다.

#### [식 본문 함수와 블록 본문 함수]

- **블록 본문 함수**

    - 본문이 중괄호로 둘러싸인 함수
    - `fun hello(): Int { 본문 }`

- **식 본문 함수**

    - 등호와 식으로 이뤄진 함수
    - `fun hello():Int = 식`

코틀린에서는 식 본문 함수가 자주 쓰인다.

> 코틀린의 이런 설계는 간결함을 자랑하면서도 필요할 때 필요한 정보를 읽을 수 있는 유지보수성을 보여준다.
>
> **블록 본문 함수**의 경우 **반드시 반환 타입을 지정**, **return 문을 사용해 반환값을 명시**해야 한다. 아주 긴 함수에 return문이 여럿 들어있는 경우, 반환 타입과 return을 명시하면 그 함수가 어떤 타입의 값을 어디서 반환하는지 더 쉽게 알아볼 수 있다.  
> 같은 맥락에서, 라이브러리를 작성할 때에도 반환 타입을 명시해야 한다. 또, 실수로 함수 시그니처가 바뀌면서 소비자들의 코드에 오류가 발생하는 것을 막을 수 있다.

### 2.1.4 데이터를 저장하기 위한 변수 선언

- 변수를 정의하며 타입 선언

    ```kotlin
    val question: String = "우주"
    val answer: Int = 42
    ```

<br>

- 타입을 지정하지 않고 초기화: 타입 추론

    ```kotlin
    val question = "우주"
    val answer = 42
    ```

> 타입 추론을 한다고 해서 컴파일러가 막 더 힘들어하고 성능이 떨어지는 것은 아니라고 한다.

<br>

- 변수를 선언하되 초기화하지 않음

    ```kotlin
    val answer: Int // 타입 명시
    answer = 42
    ```

### 2.1.5 읽기 전용 변수 val과 재대입 가능 변수 var

#### [val]

- **읽기 전용 참조**를 선언한다.
- val로 선언된 변수는 **단 한 번만 대입**될 수 있다.
- 일단 초기화하고 나면 다른 값을 대입할 수 없다. `자바의 final`

<br>

```kotlin
fun canPerformOperation(): Boolean {
    return true
}

fun main() {
    val result: String
    if (canPerformOperation()) {
        result = "Success"
    } else {
        result = "Can't perform operation"
    }
}
```

- 해당 변수가 정의된 블록을 실행할 때 정확히 한 번만 초기화되어야 한다. (컴파일러가 초기화 문장이 하나만 실행되는지를 확인한다)

<br>

```kotlin
fun main() {
    val languages = mutableListOf("Java")
    languages.add("Kotlin")
}
```

- 한 번 대입된 다음 그 값을 바꿀 수 없더라도, **그 참조가 가리키는 객체의 내부 값은 변경될 수 있다**.
- 대신 다른 객체로 재할당되는 것은 막을 수 있다.

<br>

#### [var]

- **재대입 가능한 참조**를 선언한다.
- 자바의 일반 변수와 개념적으로 같다.

기본적으로 코틀린에서는 모든 변수를 **val** 키워드를 사용해 선언하는 방식을 지켜야 한다. **읽기 전용 참조**와 **변경 불가능한 객체**를 **부수 효과가 없는 함수**와 조합해 사용하면, 함수형 프로그래밍의 이점을 살릴 수 있다.
필요할 때에만 변수를 **var**로 선언한다.

<br>

```kotlin
var answer = 42
answer = "no" // type mismatch compile error
```

- var 키워드는 변수의 값 변경을 허용하지만 **변수의 타입은 고정**된다.

### 2.1.6 문자열 템플릿

```kotlin
val input = readln()
val name = if (input.isNotBlank()) input else "Kotlin"
println("Hello, $name!")
println("Hi, ${name.length}-letter person!") // {} 구문을 사용해 변수의 프로퍼티 역시 문자열 템플릿에 사용 가능
```

- 코틀린은 `${}` 구문을 사용해 임의의 **식**도 문자열 템플릿에 포함할 수 있다.
- **cf.** `$`를 문자열에 넣으려면 `\$`로 이스케이프

#### cf. 한글을 문자열 템플릿에서 사용할 경우 주의

```kotlin
fun main() {
    val name = "Hi"
    println("$name님, 반가워요!") // Unresolved reference compile error
}
```
한글을 변수명 뒤에 바로 붙이면 컴파일러가 둘을 한번에 식별자로 인식하게 된다.

```kotlin
fun main() {
    val name = "Hi"
    println("${name}님, 반가워요!")
}
```
따라서 다음과 같이 변수 이름을 {}로 감싸서 사용할 것!

<br>

```kotlin
println("Hi, ${if (name.isBlank()) "someone" else name}!")
```
위와 같이 문자열 템플릿이 조건식을 둘러쌀 수도 있다.

<br>

---

## 2.2 행동과 데이터 캡슐화: 클래스와 프로퍼티

우선 POJO(Plain Old Java Object) 클래스와 코틀린 클래스를 비교하며 코틀린의 간결함을 느껴보자.

#### POJO Class 

```java
public class Person {
    private final String name;

    public Person(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
```
- 위 자바 클래스는 필드, 생성자, 게터를 모두 직접 작성하고 있다.
- 코틀린에서는 같은 클래스를 다음과 같이 정의할 수 있다.

#### Kotlin Class

```kotlin
// 코드 없이 데이터만 저장하는 클래스
class Person(val name: String)
```

- 코틀린은 간결한 클래스 정의가 가능하다.
- 코틀린의 기본 가시성은 public이며, 생략 가능하다.

### 2.2.1 프로퍼티

프로퍼티는 클래스와 데이터를 연관시키고 접근 가능하게 만든다.

```kotlin
class Person {
    val name: String, // 읽기 전용, 비공개 필드와 공개 getter 생성
    var isStudent: Boolean // 변경 가능, 비공개 필드와 공개 getter, 공개 setter 생성
}
```

- 코틀린은 자바의 비공개 필드 및 getter, setter로 이루어진 **프로퍼티**를 기본 기능으로 제공한다.
- 코틀린의 네이밍: is로 시작하는 프로퍼티의 getter는 get이 붙지 않고 원래 이름을 사용, setter는 is를 set으로 바꾼 이름을 사용한다.
- new 키워드를 사용하지 않고 생성자를 호출한다.
- **프로퍼티 이름을 직접 사용**해도 자동으로 **getter/setter**를 호출한다.

> 코틀린의 네이밍 방식이 행위를 분명하게 나타내는가? 써봐야 알 것 같다.

### 2.2.2 커스텀 접근자(getter)

프로퍼티 값을 저장하지 않고 계산할 때 사용한다.

```kotlin
class Rectangle(val height: Int, val width: Int) {
    val isSquare: Boolean
        get() { // 프로퍼티 게터 선언
            return height == width)
        }
}
```

직사각형 클래스에서 높이와 너비를 저장할 때, 둘이 같으면 true를 돌려주는 `isSquare` 프로퍼티를 필드에 저장하지 않고도 제공할 수 있다. 이를 on the go property라고 하며, 커스텀 getter를 사용해 제공할 수 있다.

> 커스텀 getter는, 자바에서처럼 파라미터가 없는 함수를 정의하는 것과 성능 차이는 없다.  
> 일반적으로 클래스의 특성을 기술하고 싶다면 프로퍼티로, 행동을 기술하고 싶다면 멤버 함수를 고르면 된다.

### 2.2.3 디렉터리와 패키지

코틀린의 소스코드 구조에 대해 알아보자.

#### 패키지

- 패키지 문이 있는 파일에 들어있는 모든 선언은 해당 패키지에 들어간다.
- 코틀린은 클래스 임포트와 함수 임포트를 구분하지 않는다.
- 코틀린은 여러 클래스를 같은 파일에 넣을 수 있고, 이름도 자유롭다. 디렉터리와 소스코드 파일의 위치도 관계없다.
    - 자바에서는 디렉터리 구조에 의존하는 패키지 구조, 클래스명을 따라가는 파일명

<br>

- **하지만 대부분의 경우 자바처럼 패키지별로 디렉터리를 구성하는 편이 낫다!** <span style="color:#737373; font-size:14px; font-weight:300;">  호환성도 무시 못한다</span>
- 그렇지만 여러 클래스를 한 파일에 넣는 것을 주저하지 말자.

<br>

---

## 2.3 선택 표현과 처리: Enum과 when

`when`은 자바의 `switch`를 대신하는 문법이다.

### 2.3.1 이넘 클래스와 이넘 상수 정의

```kotlin
enum class Color {
    RED, ORANGE, YELLOW, BLUE
}
```

- 자바와 다르게 **enum class**로 선언한다.

> enum은 소프트 키워드로, class 앞에서는 특별한 의미를 지니지만 다른 곳에서는 일반적인 이름으로 사용이 가능하다. class는 하드 키워드다.

<br>

```kotlin
enum class Color(
    val r: Int,
    val g: Int,
    val b: Int
) {
    RED(255, 0, 0),
    ...
    BLUE(0, 0, 255); // 코틀린에서 유일하게 세미콜론이 필수이다.

    val rgb get() = (r 256 + g) 256 + b // 읽기 전용 프로퍼티, 프로퍼티 게터
    fun printColor() = printin("$this is $rgb") // 메서드
}
```
- 이넘 상수의 프로퍼티를 정의하고, 각 상수를 생성할 때 그에 대한 프로퍼티 값을 정의할 수 있다!

### 2.3.2 when으로 이넘 클래스 다루기

```kotlin
fun getMnemonic(color: Color) =
    when (color) {
        Color.RED, Color.ORANGE -> "Richard Of"
        Color.YELLOW -> "York"
        Color.GREEN -> "Gave"
        Color.BLUE -> "Battle"
        Color.INDIGO -> "In"
        Color.VIOLET -> "Vain"
        else -> throw Exception("Noting")
    }

fun main() {
    println(getMnemonic(Color.BLUE));
    // 출력: Battle
}
```

- when도 **식(expression)**이다.
- 자바와 달리 각 분기 끝에 break를 넣지 않아도 된다.
- 여러 값과의 비교는 콤마로 구분이 가능하다.
- 만족하는 분기가 없으면 else 분기를 계산한다.

<br>

```kotlin
import ch02.colors.Color.*; // enum 상수를 임포트

fun measureColor() = ORANGE

fun getWarmthFromSensor (): String {
    val color = measureColor()
    return when (color) {
        ...
    }
}
```
- enum 상수를 임포트하면, 이름만으로 사용이 가능하다.

#### +) 디폴트 값을 지정하는 else에 대한 고찰 💭 

```kotlin
enum class Color(
    val r: Int,
    val g: Int,
    val b: Int
) {
    RED (255, 0, 0),
    ORANGE (255, 165, 0),
    YELLOW(255, 255, 0),
    GREEN(0, 255, 0),
    BLUE(0, 0, 255),
    INDIGO (75, 0, 130),
    VIOLET (238, 130, 238);
}
fun getMnemonic(color: Color) =
    when (color) {
        Color.RED, Color.ORANGE -> "Richard Of"
        Color.YELLOW -> "York"
        Color.GREEN -> "Gave"
        Color.BLUE -> "Battle"
        Color.INDIGO -> "In"
        Color.VIOLET -> "Vain"
    }
```

이 경우는 else가 없어도 모든 경우에 대해 값을 반환하기 때문에 컴파일 오류가 나지 않는다! 그러나,

```kotlin
when (color) {
        Color.RED, Color.ORANGE -> "Richard Of"
        Color.YELLOW -> "York"
        Color.GREEN -> "Gave"
        Color.BLUE -> "Battle"
        Color.INDIGO -> "In"
        // 빠졌다
    }
```
값을 반환할 수 없는 코드는 when을 컴파일 할 수 없다고 오류가 뜬다. 똑똑하다.
이 경우엔 else로 디폴트 값을 지정해줘야 한다.

### 2.3.3 when식의 대상을 변수에 캡처

```kotlin
fun getWarmthFromSensor (): String {
    return when (val color = measureColor()) {
        ...
    }
}
```

- 변수의 유효범위를 명확하게 하기 위해, when식 내에서만 사용할 수 있도록 위와 같이 **변수를 캡쳐**할 수 있다.

### 2.3.4 when의 분기 조건에 임의의 객체 사용

- when 식은 인자로 아무 객체나 사용할 수 있다.

```kotlin
when (setOf(c1, c2)) {
    setOf(RED, YELLOW) -> ORANGE
    ...
}
```

### 2.3.5 인자 없는 when 사용

```kotlin
when {
    (c1 == RED && c2 == YELLOW) ||
    (c1 == YELLOW && c2 == RED) ->
        ORANGE
}
```

- 분기의 조건이 불리언 결과를 계산하는 식이라면 인자 없이 when을 사용할 수 있다.
- 인자 없이 사용하면 불필요한 객체 생성을 막을 수 있다.
- 다만 가독성은 떨어질 수 있다. _(trade-off)_

### 2.3.6 스마트 캐스트

> 타입 검사와 타입 캐스트를 한번에

```kotlin
fun eval(e: Expr): Int {
    if (e is Num) {
        val n = e // as Num이라는 변환이 불필요
        return n.value
    }
    ...
}
```

- is 검사가 완료되었다면, e를 검사한 타입으로 타입 캐스팅할 필요가 없다.
- 단, **검사한 값이 바뀔 수 없는 경우**에만 스마트 캐스트가 작동한다.
    - 클래스의 프로퍼티에 대해 사용하려면 그 프로퍼티는 val이어야 한다! (커스텀 접근자를 사용한 경우에도 안 됨)

#### cf. 마커 인터페이스

- 여러 타입의 식 객체를 아우르는 공통 타입의 역할만 수행하는 인터페이스
- 클래스 선언 시 `:` 뒤에 인터페이스 이름을 작성하여 구현함을 나타낸다.

```kotlin
interface Expr
class Num(val value: Int) : Expr
class Sum(val left: Expr, val right: Expr) : Expr
```

### 2.3.7 if를 when으로 변경하는 방법 (리팩터링)

```kotlin
fun evale: Expr): Int =
    if (e is Num) e. value
    else if (e is Sum) eval(e.right) + eval(e.left)
    else throw IllegalArgumentException ("Unknown expression")
```

when으로 변경하면 아래와 같이 읽기 쉽게 리팩토링할 수 있다.

```kotlin
fun eval(e: Expr): Int =
    when (e) {
        is Num -> e.value
        is Sum -> eval(e.right) + eval(e.left)
        else -> throw IllegalArgumentException ("Unknown expression")
    }
```

### 2.3.8 if와 when의 분기에서 블록 사용

- 분기에 블록을 사용하는 경우, **블록의 마지막 문장이 블록 전체의 결과**가 된다.
- 이 규칙은 블록이 값을 만들어내야 하는 경우 항상 성립한다.
    - 일반적인 **함수의 경우에는 명시적인 return문이 반드시 있어야** 한다.

<br>

---

## 2.4 대상 이터레이션: while과 for 루프

### 2.4.1 while 루프

```kotlin
while (조건) {
    if (shouldExit) break
}

do {
    if (shouldSkip) continue
} while (조건)
```
- `while, do while` 루프는 **식이 아닌 문**이다.

#### [레이블(Label) 지정]

```kotlin
outer@ while (조건) {
    while (조건) {
        if (shouldExitInner) break
        if (shouldExitOuter) break@outer
    }
}
```

- `break`는 레이블을 지정하지 않으면 자신이 속한 가장 가까운 안쪽 루프에 대해 동작한다.
- 레이블을 지정하면 지정한 루프에 대해 동작한다.

### 2.4.2 수에 대한 이터레이션: 범위와 순열

코틀린은 C와 같은 for 루프가 없다. 대신 범위를 사용한다.

<br>

```kotlin
val oneToTen = 1..10
```

- 범위 연산자는 `..`로, 양끝을 포함한다.
- 범위 값을 일정한 순서로 이터레이션하는 것을 순열이라고 부른다.

```kotlin
for (i in 1..100)
for (i in 100 downTo 1 step 2) // 100부터 1까지의 범위에서 2씩 감소
```

- downTo로 역방향 순열을 만들 수 있다. 증가 값이 -1이 되는 것.
- step은 증가 값의 방향을 유지하면서 절댓값을 바꿔준다.

```kotlin
for (i in 0..< size) // 0..size - 1
```

- 끝 값을 포함하지 않는 반만 닫힌 범위를 만들 수도 있다.

### 2.4.3 컬렉션에 대한 이터레이션

```kotlin
for (color in collection)
```

- 컬렉션에 대한 이터레이션은 위와 같이 사용할 수 있다.

#### [맵에 대한 이터레이션]

```kotlin
val binaryReps = mutableMapOf<Char, String>() // 알파벳과 그에 대한 이진 표현을 저장
...

// 구조 분해 선언: 컬렉션의 원소를 푼다
for ((letter, binary) in binaryReps) {
    println("$letter = $binary")
}

// 키만 필요할 때
for (letter in binaryReps.keys) {
    println(letter)
}
```

- **cf.** 코틀린은 `map[key] = value` 이런 스타일의 코드가 가능

### 2.4.4 in

컬렉션이나 범위의 원소를 검사한다.

```kotlin
fun isLetter(c: Char) = c in 'a'..'z' || c in 'A'..'Z'
```

- `java.lang.Comparable`을 구현한 클래스에 모두 사용 가능
- when 식에서도 사용 가능

<br>

---

## 2.5 코틀린 예외 처리

### [예외 던지기]

```kotlin
val percentage =
    if (number in 0..100)
        number
    else
        throw IllegalArgumentException("...")
```

- 자바와 달리 **코틀린의 throw는 식이**다.

### [예외 처리]

```kotlin
fun readNumber(reader: BufferedReader): Int? {
    try {
        ...
    }
    catch (e: NumberFormatException) {
        ...
    }
    finally {
        ...
    }
}
```

- 자바와 마찬가지로 try, catch, finally를 사용한다.
- 대신 함수가 던질 수 있는 예외를 명시할 필요가 없다.
    - `throws` 절이 없다!

#### 코틀린 컴파일러는 예외 처리를 강제하지 않는다

> **명시적으로 처리해야 하는 자바의 체크 예외**  
> IOException은 체크 예외이기 때문에 throws를 통해 명시적으로 처리해 주어야 한다.

- 코틀린은 체크 예외와 언체크 예외를 구별하지 않는다.
- 따라서 잡아내고 싶은 예외와 그렇지 않은 예외를 직접 결정할 수 있다.
    - 자바의 체크 예외에 대한 처리가 무의미한 경우, 예외 처리가 오류를 방지하지 못하는 경우를 생각한 설계이다.

#### cf. try-with-resource는?

특별한 문법은 없고, 따로 라이브러리 함수가 존재한다.

#### try를 식으로 사용할 수 있다

- try는 문이지만 식이기도 하다.

```kotlin
fun readNumber(reader: BufferedReader) {
    val number = try {
        Integer.parseInt(reader.readLine()) // 블록의 마지막 문장이 결과값
    } catch (e: NumberFormatException) {
        null // 예외가 발생하면 null을 사용한다. return 문 대신 값 반환도 가능
    }
}
```

> 자바에서는 try 바깥에서 변수를 선언하고 try 내에서 변수에 값을 할당해야 했는데!!

try를 식으로 사용하면 중간 변수를 도입하는 것을 피함으로써 코드를 좀 더 간결
하게 만들고, **더 쉽게 예외에 대비한 값을 대입**하거나 **try를 둘러싼 함수를 반환시킬 수 있다**. 