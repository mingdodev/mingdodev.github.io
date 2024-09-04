---
layout: post
title:  "[Java] 자바 자료형 이해하기"
author: "minseo"
categories: [blog, java]
tags: [Java, Spring]
comments: true
image: "/static/img/logo/java-logo.png"
hide_image: true
---
# [Java] 자바 자료형 이해하기

<br><center><img src="../../../static/img/logo/java-horizontal.png" width="35%"></center>
<br><br>

자바에 대한 기본적인 이해를 돕고 모호했던 개념을 확실히 짚고 가기 위해 가장 기초가 되는 자료형<span style="color:#737373; font-size:14px; font-weight:300;"> Data Type </span>에 대해 정리해보았다.

<br>

---

# Java Data Type

자바의 자료형은 크게 기본 자료형 <span style="color:#737373; font-size:14px; font-weight:300;"> Primitive type </span>과 참조 자료형 <span style="color:#737373; font-size:14px; font-weight:300;"> Reference type </span>으로 구분된다.

## Primitive type

- **기본 타입** 또는 **원시 타입**이라고 부른다.
- 자바에서 미리 정의하고 제공하는 8가지의 자료형이다.
- `short, int, long, float, double, char, byte`

- 객체가 아닌 값 자체를 메모리에 저장한다.
- 메모리의 `stack` 영역에 저장된다.

```java

    int a = 2;

```

- `a`: 2라는 값 자체를 `stack`에 저장한다.

## Reference type

- **참조 타입**
- 기본 타입을 제외한 모든 자료형을 말한다.

- `java.lang.Object` 클래스를 상속한다.
> `java.lang.Object` 클래스는 자바의 최상위 클래스<span style="color:#737373; font-size:14px; font-weight:300;"> root of the class hierarchy </span>로 모든 참조 타입의 부모 역할을 한다.
>
>객체의 정보를 문자열로 표현하여 반환하는 `toString()`, 두 객체가 같은 주소값을 참조하는지 비교하는 `equals()` 등의 메서드를 정의하고 있다.

<br>

- 메모리 주소를 통해 객체를 참조한다.

    - 자바에서 **실제 객체**는 `heap` 영역에 저장되며, 참조 타입의 변수는 `stack`에 **실제 객체들의 주소**를 저장한다. 따라서 객체를 사용할 때마다 참조 타입 변수에 저장된 객체의 주소를 통해 객체를 불러와 사용하는 것이다.
    - 이후 `Garbage Collector`가 돌면서 사용하지 않는 객체를 메모리 <span style="color:#737373; font-size:14px; font-weight:300;"> heap </span>에서 해제한다.
    - 따라서 최소 두 번 메모리에 접근해야 하기 때문에 접근 속도가 느리다.

```java

    MyClass myObject = new Myclass();

```

- `new MyClass()`: MyClass 객체를 생성 후 `heap`에 저장한다.
- `myObject`: 참조 타입 변수로 `stack`에 객체의 주소를 저장한다.

<br>

- `Array, Enum, Class, Interface`와 자바 14부터 도입된 `Record` 등이 있다.

<br>

---

# Reference Type

일반적으로 자주 사용하는 참조타입에 대해 구체적으로 알아보자. 단순 사용 방법이나 메서드를 소개하기보다는 어떤 범주에 속하는 개념이며 어떤 주목할만한 특성이 있는지 이해하는 것을 목적으로 정리했다.
참고로 Class 타입을 구분했다고 해서 다른 타입들이 전부 Class로 선언되지 않았다는 뜻이 아니다.

## 1. Class

기본 타입과 다르게 객체를 참조하는 형태이다. 기본적으로 `java.lang.Object`를 상속받는다.  
객체 지향 언어인 자바의 기본적인 빌딩 블록이다.

```java

    class Cat{
        private int age;

        Cat(int age) {
            this.age = age;
        }

        public int getAge() {
            return age;
        }

        public void setAge(int age) {
            this.age = age;
        }
    }

    public class ClassExample {
        public static void main(String[] args) {
            Cat Tom = new Cat(1);
            Cat Nabi = new Cat(2);
            System.out.println(Tom.getAge()); // 1
            Tom = Nabi;
            System.out.println(Tom.getAge()); // 2
            Nabi.setIndex(3);
            System.out.println(Tom.getAge()); // 3. 같은 객체를 참조하기 때문
        }
    }

```

### 📌 String Class

- 참조 타입에 속하지만, 기본적인 사용은 원시 타입처럼 사용한다. 
- **Immutable**하다.

    - 값을 변경하는 메서드들이 존재하지만, 실제로 값을 바꾼다는 것은 **새로운 String Class 객체를 만들어내는 것**이다.

    - 따라서 객체 자체는 생성 이후 값이 변하지 않는다.

- 데이터를 비교할 때 `.equals()` 메서드를 사용한다.

    - `java.lang.Object`에 정의된 메서드를 **오버라이딩**한 메서드로 주소값이 아닌 **문자열의 내용을 비교**한다.

    - **cf.** 대소문자를 구분하지 않을 때는 `equalsIgnoreCase()`를 사용한다.

<br>

- 두 가지 방식으로 문자열을 생성할 수 있다.

```java

    // 생성자를 사용
    String str1 = new String("Hi there!");

    // 문자열 리터럴을 사용
    String str2 = "Hi there!";

```

- 생성자를 호출하여 문자열을 만드는 방법은 매번 새로운 객체를 생성하여 `heap`에 저장한다.  
    즉 객체들 중 동일한 문자열을 갖는 객체가 있더라도 새로운 객체를 만든다.

- 문자열 리터럴을 사용하면 `String Pool`이라는 특별한 메모리 영역에 저장한다. 그래서 생성할 때 이미 문자열 풀에 동일한 값의 문자열이 존재한다면 이미 존재하는 해당 객체를 참조하도록 한다.  
따라서 생성자를 호출하는 것보다 메모리 효율이 좋다. <span style="color:#737373; font-size:14px; font-weight:300;"> 문자열 리터럴을 사용하면 마치 원시 타입처럼 편리하게 사용할 수 있다 </span>

### 📌 Wrapper Class

- 원시 타입의 자료형을 객체로 다루어야 할 때가 있다. 이러한 경우 원시 타입을 객체로 표현하기 위한 클래스가 **래퍼 클래스**이다.
- `Short, Integer, Long, Float, Double, Character, Byte`

    - 이러한 래퍼 클래스로 원시 타입을 감싸<span style="color:#737373; font-size:14px; font-weight:300;"> wrap </span> 사용한다.

- 래퍼 클래스가 감싸고 있는 값은 외부에서 변경할 수 없다.
    변경을 위해서는 새로운 래퍼 클래스 객체로 생성해야 한다.

- 원시 타입을 래퍼 클래스로 감싸는 것을 박싱, 래퍼 클래스로부터 원시 타입의 값을 꺼내는 것을 언박싱이라고 한다.

```java

    // 박싱
    int i = 10;
    Intger num = new Integer(i);

    // 언박싱
    Intger num = new Integer(10);
    int i = num.intValue();

```

## 2. Interface

- 클래스가 구현해야 하는 메서드의 집합을 정의해놓은 타입이다.
    - 추상 메서드로 정의된다.
    - `public static final` 속성의 상수를 가질 수 있다.
- 클래스와 달리 직접 인스턴스를 생성할 수 없기 때문에 인터페이스의 추상 메서드를 구현할 클래스가 필요하다.

```java

    interface Animal {
        String TO_KOREAN = "동물";

        public int getAge();
        public void setAge(int age);
    }

```

## 3. Array

- 배열이란 엄밀히 말하면 자료형의 종류가 아니라 **자료형의 집합**이다.
- 자료형 바로 옆에 `[]`을 붙여 나타낸다. 기본형, 참조형 모두 배열로 만들 수 있다.
- 배열형 변수 또한 해당 배열의 주소를 가지는 것이다.

```java

    public class ArrayExample {
        public static void main(String[] args) {
            String[] animals = {"dog", "cat", "fox", "bear", "otter"};
            int[] a = new int[4];
            a[0] = 1;
        }
    }

```

- 배열의 크기를 지정하고 인덱스를 통해 값을 대입하거나 배열 생성 시 `{}`로 값을 초기화할 수 있다.
- 배열을 생성하고 값을 대입하지 않으면 자료형의 디폴트 값이 들어간다.
    - 객체 타입은 `null`, 숫자 타입은 `0`, 불리언 타입은 `false`

## 4. Enum

- **한정된 상수 데이터의 집합**을 정의하고 사용할 때 `enum`을 이용한다.

    - `요일, 달, 계절 등`은 전부 한정된 몇 가지 상수값을 가지는 데이터 주제이다.

### 🤔 왜 enum으로 상수를 관리하는 것이 좋은가?

자바에서 Enum 타입을 사용하지 않고도 상수를 정의하는 방법이 몇 가지 존재한다.

1. `final`로 불변성을 보장하고 `static`으로 메모리에 한번만 할당되도록 만들기

    - 일반적으로 상수에 정수값을 저장하여 상수를 구분하고 관리한다.
    - 그러나 이 방법으로 상수를 정의하면 접근 제어자 + `static fianl`까지 작성해야 하기 때문에 가독성이 좋지 않다.

2. 이를 개선하기 위해 **인터페이스 내부의 상수**로 관리하면 `public static fianl` 속성을 생략할 수 있어 가독성은 좋아진다. 그러나 애초에 이 방식은 상수를 정의하고 사용하는 취지에 어긋난다. 상수는 불변성도 중요하지만 특정한 의미를 가지는 **고유한 값**이다. 따라서 정수와 같은 단순히 구분을 위한 데이터로 관리하는 방법은 논리적이지 않으며, 잘못된 상수값이 할당되어도 결국은 정수값이기 때문에 컴파일 에러가 발생하지 않는 등의 문제가 발생할 수 있다.

3. 따라서 **상수를 독립된 고유의 객체로 선언**하여 사용하기 위해 자체 클래스를 인스턴스화한 후 `static final`로 불변성을 보장하는 방식이 등장했다.

    ```java

        class Season {
            public static final Season SPRING = new Season();
            public static final Season SUMMER = new Season();
            public static final Season FALL = new Season();
            public static final Season WINTER = new Season();
        }

    ```
    - 그러나 이 방식 역시 가독성이 좋지 않다.
    - 또한 이렇게 자체 클래스로 선언하게 되면 사용할 수 있는 데이터 타입이 한정적인 `switch`문을 사용하기 어려워진다. 일반 클래스는 조건문에 들어갈 수 없기 때문이다.

<br>

이러한 가독성과 사용성 문제를 해결하기 위해, 자바에서는 상수를 편리하게 관리하고 사용할 수 있는 `enum`을 만들게 되었다.

```java

    // 열거 상수 정의
    public enum Season {
        SPRING, SUMMER, FALL, WINTER;
    }

    public class EnumExample {
        public static void main(String[] args) {
            Season today = Season.SUMMER; // 열거 상수 대입

            // 열거 상수 사용
            if (today == Season.SUMMER) {
                System.out.println("It's boiling hot!");
            } else {
                System.out.println("Nice.");
            }
        }
    }

```

> 🧐 String, Array와 다르게 Enum을 정의할 때에는 public을 붙여주는 이유?
>
> enum의 상수는 클래스의 인스턴스 개념이기 때문이다. 따라서 enum을 다른 클래스에서 사용하려면 당연히 public 접근 제어자를 사용해줘야 한다. 생략한다면 default로 동일한 패키지 내에서만 접근할 수 있게 된다.

## 5. Record

- 자바 14부터 등장하여 자바 16에서 정식 기능이 된 클래스 타입이다.

- 불변 데이터로 사용하는 **데이터 전송 객체(Data Transfer Object, DTO) 또는 값 객체(Value Object, VO)**를 간결하게 정의하는 데 주로 사용한다.

```java

    public record Member(
        Long id,
        String name,
        int Age
    ) { }

```

- 필드가 한번 생성되면 값을 변경할 수 없다.
- 모든 필드를 매개변수로 받는 생성자를 자동으로 정의한다.
- 필드를 기반으로 `getter, equals, hashCode, toString` 메서드를 자동으로 정의한다.

### 🧐 등장 배경 

**1. 자바 언어에서 불변 객체를 표현하기 위해서는 많은 양의 보일러 플레이트 코드가 필요하여 복잡했다.**

>  cf. 보일러 플레이트 코드
>    - 특정 작업 수행을 위해 반복적으로 작성해야 하는 코드
>    - 자바에서는 `getter/setter, equals, hashCode, toString` 등이 있다.

- 불변성을 위해 모든 필드를 `final`로 선언해야 하며 각 필드에 대한 `getter` 메서드가 필요하다.

- 또한 객체 자체가 아닌 객체가 가진 데이터로 비교를 수행해야 하므로 `equals, hashCode, toString`을 오버라이딩한 메서드가 필요하다.

- lombok이나 IDE를 통해 코드를 줄일 수 있지만 이는 근본적인 문제 해결이 아니다.

자바 언어가 가진 이러한 근본적인 문제를 해결하기 위해 등장한 레코드는 객체 지향에 알맞은 간결한 데이터 표현을 제공한다.

<br>

**2. 단순히 데이터만을 위한 데이터 클래스가 필요했다.**

- 일반적으로 데이터베이스, 쿼리의 결과, 서비스에서 사용하는 정보 등을 담기 위해 클래스를 작성한다. 보통 이러한 데이터들은 변경되어서는 안 되는 값이다.
- 이러한 특성을 반영한 클래스를 작성하면 보일러 플레이트를 포함한 추가 코드 때문에 데이터를 담은 클래스라는 목적이 명시적으로 드러나지 않게 된다.
- 또한 새로운 필드가 추가될 때마다 클래스가 자동으로 업데이트되지 않기 때문에 메서드를 수동으로 변경해줘야 한다.

따라서 불변을 보장하며 목적이 명확한 데이터 클래스를 새로 만든 것이라 볼 수 있다.

<br>

---

# Primitive Type vs Reference Type

### 1. null 입력 값

- 참조 타입은 null을 입력 값으로 받을 수 있지만, 원시 타입은 불가능하다.
- **cf**. 따라서 참조 타입 변수가 null을 가진다면 이후 이 변수로 메서드를 호출할 때 `NullPointerException`이 발생할 수 있게 된다. 이를 방지하는 것이 `Optional type`으로 자세한 설명은 [여기](https://mingdodev.github.io/blog/java/2024-08-29-Java-Null-Optional/)를 참고하자.


### 2. Generic type

- 클래스 내부에서 사용할 데이터 타입을 외부에서 지정해 주는 것을 `Generic`이라고 한다.
- 참조 타입은 제너릭 타입으로 사용할 수 있지만, 원시 타입은 불가능하다.

```java

    LinkedList<int> hi; // 불가
    LinkedList<Integer> hi; // 가능

```


<br><br>
<details>
<summary> &nbsp; 📁 참고 자료</summary>
<div>
    <div>
    ❗️ <a href="https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EC%97%B4%EA%B1%B0%ED%98%95Enum-%ED%83%80%EC%9E%85-%EB%AC%B8%EB%B2%95-%ED%99%9C%EC%9A%A9-%EC%A0%95%EB%A6%AC#enum_%EC%83%81%EC%88%98" target="_blank">자바 Enum 열거형 타입 문법 & 응용 정리</a>
    </div>
    <div>
    ❗️ <a href="https://jdm.kr/blog/213" target="_blank">자바 자료형 정리</a>
    </div>
    <div>
    ❗️ <a href="https://www.baeldung.com/java-record-keyword" target="_blank">Java Record Keyword</a>
    </div>
    <div>
    ❗️ <a href="https://velog.io/@pp8817/record" target="_blank">[Java] record에 대하여</a>
    </div>
    <div>
    ❗️ 점프 투 자바
    </div>
</div>
</details>
