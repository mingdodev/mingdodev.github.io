---
layout: post
title:  "[Kotlin in Action] 6장. 컬렉션과 시퀀스"
author: "minseo"
categories: [blog, java]
tags: [Kotlin, Java]
comments: true
image: "/static/img/logo/kotlin-logo.png"
hide_image: true
---

# [Kotlin in Action] 6장. 컬렉션과 시퀀스

<br><br><center><img src="../../../static/img/logo/kotlin-horizontal.png" width="30%"></center>
<br><br><br>

> 도서 <코틀린 인 액션 2/e>를 읽으며 코틀린에 대해 이해한 내용을 정리한 글이다.

<br>

6장의 학습 목표는 _코틀린으로 컬렉션을 우아하게, 간결하게 다루자!_ 이다.
컬렉션과 관련된 API들을 소개하는 장이라 필요할 때 이들을 떠올릴 수 있게 인지하고 있으면 좋을 듯하다. 자바의 스트림과 유사한 **시퀀스**에 대해 이해하고, 컬렉션에서 보다 효율적인 연산을 수행하는 방법을 배운다.

<br>

---

## 6.1 컬렉션에 대한 함수형 API

컬렉션을 다룰 때 유용하게 쓸 수 있는 함수형 API들은 다음과 같다.

### filter, map

기본적으로 출력 컬렉션을 조작한다. <span style="color:#737373; font-size:14px; font-weight:300;"> 입력 컬렉션의 원소들은 그대로이다 </span>

```kotlin
people.filter { it.age >= 30 }
```
- 이름 그대로 조건에 맞는 원소들을 필터링해 출력 컬렉션을 반환한다.

```kotlin
numbers.map(it * it)
```
- **입력 컬렉션의 원소를 지정된 함수로 변환**해 출력 컬렉션을 반환한다.

### reduce, fold

컬렉션의 정보를 합치는 데 사용한다.

```kotlin
list.reduce { acc, element ->
    acc + element
}
```
- 컬렉션의 첫 번째 값을 누적기에 넣고 호출마다 다음 인자가 전달된다.
- `fold`는 시작 값을 선택할 수 있고, 첫 번째 원소부터 누적 값으로 전달된다.

### all, any, none, count, find

조건을 만족하는지 판단한다.

```kotlin
val numbers = listOf(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)

// all: 모든 원소가 짝수인가?
println(numbers.all { it % 2 == 0 })

// any: 하나라도 짝수인가?
println(numbers.any { it % 2 == 0 })

// none: 짝수가 하나도 없는가?
println(numbers.none { it % 2 == 0 })

// count: 짝수의 개수
println(numbers.count { it % 2 == 0 })

// find: 조건을 만족하는 첫 번째 짝수
println(numbers.find { it % 2 == 0 })

// false
// true
// false
// 5
// 2
```
- `all, any, none`은 조건을 만족하는지 판단한다.
- `count`는 조건을 만족하는 원소의 개수를, `find`는 조건을 만족하는 첫 번째 원소를 반환한다.

### partition

조건에 따라 리스트를 분할해 리스트의 쌍(Pair)으로 만든다.

```kotlin
val (even, odd) = listOf(1, 2, 3, 4, 5).partition { it % 2 == 0 }
// Pair를 반환
```
- 리스트를 2개로 분할한다. (조건 참, 조건 거짓)

### groupBy

리스트를 여러 그룹으로 이루어진 맵(Map)으로 바꾼다.

```kotlin
val grouped = listOf("apple", "banana", "avocado").groupBy { it.first() }
println(grouped) // {a = [apple, avocado],  b = [banana]}
```

### associate, associateWith, associateBy

컬렉션을 맵으로 변환한다.

```kotlin
val assoc = listOf("a", "bb", "ccc").associate { it to it.length }
println(assoc) // {a=1, bb=2, ccc=3}
```
- `associate`는 원소를 키와 값으로 직접 지정한다.

```kotlin
val assocWith = listOf("a", "bb", "ccc").associateWith { it.length }
println(assocWith) // {a=1, bb=2, ccc=3}

```
- `associateWith`는 원소를 키로, 람다 결과를 값으로 매핑한다.

```kotlin
val assocBy = listOf("a", "bb", "ccc").associateBy { it.length }
println(assocBy) // {1=a, 2=bb, 3=ccc}
```
- `associateBy`는 람다 결과를 키로, 원소를 값으로 매핑한다.

### replaceAll, fill

**가변 컬렉션**의 원소를 변경한다.

```kotlin
val names = mutableListOf("Martin", "Samuel")
names.replaceAll { it.uppercase() }
names.fill("(redacted)")
```
- `replaceAll`은 각 원소를 람다의 결과를 사용해 대체한다.
- `fill`은 컬렉션의 모든 원소를 지정된 값으로 덮어쓴다.

### ifEmpty

컬렉션 입력이 비어있지 않는 경우에만 처리하고 싶을 때 사용한다. <span style="color:#737373; font-size:14px; font-weight:300;"> 다른 언어들의 isEmpty랑 이름이 다르다 </span>

```kotlin
val empty = emptyList<String>()
println(empty.ifEmpty { listOf("no", "values", "here" )})
```
- 컬렉션이 비어있을 때 기본값을 생성하는 람다를 제공
- cf. 문자열에서는 ifBlank를 사용

### chunked, windowed

**연속적인** 데이터 값들에서 유용하게 쓸 수 있다.

```kotlin
val parts = (1..10).chunked(3)
println(parts) // [[1, 2, 3], [4, 5, 6], [7, 8, 9], [10]]
```
- 주어진 크기의 서로 겹치지 않는 부분으로 나누기

```kotlin
val windows = (1..5).windowed(3)
println(windows) // [[1, 2, 3], [2, 3, 4], [3, 4, 5]]
```
- 슬라이딩 윈도우 사용하기

### zip

두 개의 컬렉션을 합친다.

```kotlin
val zipped = listOf("a", "b", "c").zip(listOf(1, 2, 3))
println(zipped) // [(a, 1), (b, 2), (c, 3)]
```
- 반환 값은 `List<Pair<T, R>>`


### flatMap, flatten

**내포된 컬렉션**의 원소를 처리한다.

```kotlin
val lilbrary = listOf(
    Book("A", listOf("man1", "man2", "man3")),
    Book("B", listOf("man12", "man23", "man32")),
)

fun main() {
    val authors = library.flatMap { it.authors }
    // List<List<String>>이 아닌 List<String>이 나온다.
}
```

- `flatmap()` 함수는 다음과 같이 컬렉션 내 컬렉션을 평평한 리스트로 반환한다.
- 변환할 것이 없고 컬렉션의 컬렉션을 평평하게 만들 때 `flatten()`을 사용한다.

<br>

---

## 6.2 시퀀스

> 지연 계산 컬렉션 연산

자바 8의 스트림과 비슷하게 **중간 임시 컬렉션을 사용하지 않고 컬렉션 연산을 연쇄**하는 방법이다.

```kotlin
people.map(Person::name).filter { it.startsWith("A") }
```

리스트를 반환하는 연산이 연쇄되어 있다. 이 연쇄 호출은 리스트를 2개 만든다.
이정도로는 괜찮겠지만 원소가 수백만 개가 되면 매우 효율이 떨어진다.

코틀린에서는 각 연산이 컬렉션을 직접 사용하는 대신, 시퀀스를 사용하게 만든다.

```kotlin
people
    .asSequence()
    .map(Person::name)
    .filter { it.startsWith("A") }
    .toList()
```

- `asSequence` 확장 함수를 통해 어떤 컬렉션이든 시퀀스로 바꿀 수 있다.
- 내부적으로 `Iterator`를 기반으로 동작하며, 매번 새로운 저장 구조를 만들지 않는다.
- **최종 연산**인 `toList()`**에 와서야 실제 계산이 시작**되는, 지연 계산이다.
- 모든 연산은 각 원소에 대해 순차적으로 적용된다.
    - **각 원소마다** map → filter를 거쳐 최종 결과로 바로 전달된다.

> 궁금한 점. 정렬같은 경우는 이터레이션만으로 처리가 안 될 것 같은데?  
> 찾아보니 순회로 원소를 모으고, 중간 컬렉션을 생성하여 정렬을 수행한 뒤 다시 시퀀스로 반환한다고 한다.
> 정렬하고 다시 뭔가를 할 게 아니면 굳이 시퀀스에서 정렬할 필요가 없을 듯

- 컬렉션에 대해 수행하는 연산의 순서도 성능에 영향을 끼친다. 더 빨리 원소를 제거하면 할수록 코드 성능이 좋아진다.

### 시퀀스 만들기

```kotlin
import java.io.File

fun File.isInsideHiddenDirectory() =
    generateSequence(this) { it.parentFile }.any { it.isHidden }

fun main() {
    val file = File("/Users/svtk/.HiddenDir/a.txt")
    println(file.isInsideHiddenDirectory())  // true
}
```
- 일반적으로 객체의 조상들로 이루어진 시퀀스를 만들고, 조상의 시퀀스에 대해 어떤 특성을 알아내는 식으로 사용한다.