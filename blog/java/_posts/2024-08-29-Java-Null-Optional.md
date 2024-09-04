---
layout: post
title:  "[Java] 자바 Null과 Optional"
author: "minseo"
categories: [blog, java]
tags: [Java, Spring]
comments: true
image: "/static/img/logo/java-logo.png"
hide_image: true
---
# [Java] 자바 Null과 Optional

<br><center><img src="../../../static/img/logo/java-horizontal.png" width="35%"></center>
<br><br>
   
자바를 모른채 스프링으로 개발할 때 자주 눈에 들어왔던 Optional에 대해 언젠가는 정리하고 싶었다. 여러 강의에서 굉장히 중요한 개념이라고 언급되었는데, 얘가 어떤 좋은 역할을 해낸 건지 정확히 뭐가 중요한 건지 알아보아야겠다.

<br>

Optional은 값을 감싸주는 안전장치라고 들었는데, 어떤 위험 요소 때문에 필요한 것일까?  
이러한 Optional의 역할을 이해하기 위해서는 먼저 자바에서 null이 가지는 의미에 대해 명확히 짚고 넘어가야 한다.

<br>

---

# Java에서 Null이란?

### ❗️ 주소값이 없음

- 일반적으로 **값이 없음**을 표현하는 키워드로, 자바에서는 참조 타입 변수가 **아무런 객체도 참조하고 있지 않음**을 나타낸다.

> null의 의미 자체에 대해 생각해보기 위해 C의 포인터 개념을 떠올려보자. 포인터 변수는 생성된 **메모리의 주소**를 가리킨다.   
> 만약 포인터 변수의 값을 초기화 하지 않고 선언만 한다면 포인터는 아무런 주소도 가리키지 않을 것이다.  
> 이러한 `주소가 존재하지 않는 상태`를 어떻게 표현해야 하는가?
>
> 이러한 표현이 `null` 키워드로 제공되는 것이다.

- 값이 `null`인 경우 **할당되는 메모리가 없다.**

- 참조 타입에서 값이 없을 때 사용하거나 기본값으로 사용된다.

- 원시 타입에는 대입할 수 없다.

    - `null`을 원시 타입에 대입하면 컴파일 에러가 발생한다.

    - 주소를 통해 데이터에 접근하는 개념이 아니므로 키워드의 의미를 생각해보았을 때 당연한 논리이다.

    - 따라서 값이 없음을 표현하기 위해 자료형에 따라 따로 정해진 값을 사용한다. 초기화하지 않았을 때의 기본값도 동일하다. 
        - **e.g.** `0, false, '\u0000'` <span style="color:#737373; font-size:14px; font-weight:300;"> 출력하면 아무것도 없는 제어 문자이다. 제어 문자는 특정 작업을 지시하는 역할이다.</span>

### ❗️ 에러 발생의 요인

할당되는 메모리 자체가 없기 때문에 `null`을 다룰 때에는 특별히 주의가 필요하다.

<br>

- **원시 타입에 `null`이 들어가려는 경우**: 자바에서 자동 형변환 시 에러

    `Integer`나 `Double`과 같은 래퍼 클래스가 `null`을 참조하고 있는데 원시 타입으로 사용하기 위해 자동 형변환이 일어나는 경우 에러가 발생하게 된다.

```java

    Integer n = new Integer(84);

    n = null; // 모종의 이유로 래퍼 클래스 객체에 null이 들어가게 되면

    int m = n; // 이때 참조 타입을 원시 타입으로 사용하기 위해 자동으로 언박싱, NullPointerException

```

<br>

- **`null` 객체의 메서드 호출**

`null` 객체에서 메서드를 호출하게 되면 `NullPointerException`이 발생한다.

```java

    String str = null;

    System.out.println(str.length());

```

<br>

- **배열을 초기화하지 않고 접근**

배열 변수도 객체의 참조를 저장하기 때문에 `null`로 초기화할 수 있다.

```java

    int[] nums = null;

    System.out.println(nums[0]); // NullPointerException

```

배열이 아무런 주소도 가리키고 있지 않은데 인덱스로 접근하려고 하면 에러가 발생한다.

<br>

- **대소 비교 관계 연산자 사용**

값의 크고 작음을 비교하는 관계 연산자 `<, >, <=, >=` 는 숫자형 원시 타입과 함께 사용될 때 의미가 있다. 따라서 우선 객체 타입과 함께 사용할 수 없다. <span style="color:#737373; font-size:14px; font-weight:300;"> 컴파일 에러가 발생한다 </span>

```java

    Integer n = null;

    if (n <= 1) {
        System.out.println("hi");
    }

```

또한 다음과 같은 래퍼 클래스 객체에 `null`이 들어있을 경우 자동으로 **언박싱**이 되며 컴파일 에러는 피해갈 수 있지만, 코드를 실행할 때 `NullPointerException`이 발생한다. `n <= 1`이 실행될 때 실제로는 `n.intValue() <= 1`로 변횐되는데, 이때 n이 `null`이므로 메서드 호출이 불가능하기 때문에 에러가 발생하는 것이다. 이는 두 번째 에러 발생과 유사한 경우이다.

<br>

---

# NullPointerException

그렇다면 `null`로 인해 발생하는 에러는 어떠한 에러이며 어떻게 다뤄야 하는가?  
`NullPointerException, NPE`는 프로그램 실행 도중 발생하는 런타임(Runtime) 에러이다.

### 🤔 런타임 에러가 큰 문제가 되는 이유가 무엇일까?

컴파일 에러는 프로그램이 실행되기 전 발견할 수 있으므로 예방이 가능하다. 하지만 런타임 에러는 컴파일 시점에는 문제가 되지 않아 발견하기 어려울 뿐만 아니라, 실행 도중 발생하므로 예측 불가능한 동작을 일으킬 수 있다. 개발 도중에 발견하고 고치기 어려워 프로덕션 환경이나 실사용 환경에서 발생할 수 있는 에러이기 때문에 문제가 된다.

> 오죽하면 컴파일 에러만으로 오류를 잡아낼 수 있다면 굉장히 잘 설계한 코드라는 말도 들어본 적이 있다.

### 🧐 어떠한 상황에서 발생하는가?

예제 코드를 통해 발생하기 쉬운 널포인터 에러에 관해 알아보자.

```java

    class User {
        private Address address;
        private String name;

        User(String name) {
            // 생성자에서는 name만 초기화하고 있음
            this.name = name;
        }

        public Info getAddress() {
            return this.address;
        }
    }

    class Address {
        private City city;

        // 생성자에서 필드값을 초기화하지 않는 경우

        public City getCity() {
            return this.city;
        }
    }

    class City {
        public String printCity() {
            return "Seoul";
        }
    }

    public class Main( ) {
        public static void main(String[] args) {
            User user = new User("Cat");

            // user의 Address 객체는 초기화되지 않았다.

            user.getAddress().getCity().printCity(); // NullPointerException
        }
    }

```

다음과 같이 여러 객체 클래스가 정의되어 있고 최종적으로 다루는 객체는 중첩된 객체이다.  
메인 클래스를 살펴보면 `user.getAddress()`에서 널포인터 에러가 발생함을 알 수 있다. `user`의 `address` 객체가 `null`이므로 널 객체의 메서드를 참조할 수 없기 때문이다.

<br>

이러한 경우 자바스크립트 진영에서는 옵셔널 체이닝 연산자 `?.`, 코틀린에서는 엘비스 연산자 `?:`을 사용하여 간편하게 처리할 수 있다고 한다.

그러나 자바 진영에서는 이러한 연산자를 지원하지 않으며, 엘비스 연산자에 대한 논의가 있었으나 이를 도입하면 `null`을 더 많이 사용할 것이라 판단하여 승인되지 않았다고 한다.

## 📕 예방법

가장 기본적인 예방법부터 권장되는 방식까지 흐름에 따라 알아보자.

### 기본값을 사용하도록 설정하기

우선 객체를 선언할 때 초기값을 사용하는 것을 권장한다. 가장 직관적이다.

```java

    class User {
        private Address address;
        private String name;

        User() {
            this.address = new Address(); // 물론 이 객체의 생성자에서도 초기화가 필요할 것이다. (생략)
            this.name = "unknown";
        }
    }

```

그러나 기본값을 사용하게 되면 의도하지 않거나 불필요한 값이 생성되는 느낌이다. 또한 반드시 알아야 할 이름과 같은 정보에 기본값을 사용하는 것은 일반적이지 않고 그래서는 안 된다. 이렇게 서비스 구조상 어쩔 수 없이 마주하게 되는 `null`값이 존재할 것이다.

### 에러 발생 가능성이 있는 부분 예외 처리해주기

자바8 이전에는 `NullPointerException`의 가능성이 있는 코드를 조건문 중첩 코드로 회피하였다고 한다.

```java

    private Scanner scanner;

    public NullCheckExample() {
        // 널인 경우에 대한 예외 처리
        if(scanner == null) {
            scanner = new Scanner(System.in);
        }
        int a = scanner.nextInt();
        System.out.println(a);
    }

```

조건문을 사용한 코드는 직관적이기 때문에 이런 간단한 코드에서는 오히려 좋을 수도 있다.

그러나 서비스가 방대해지고 메서드 개수가 많아진다면, 여러 개의 <span style="color:#737373; font-size:14px; font-weight:300;">심지어는 중첩된</span> 조건문 코드를 작성하는 방식은 불편하다! 코드의 양이 어마무시하게 증가할 것이다.

<br>

따라서 어쩔 수 없이 `null`을 직접 마주하게 되는 경우를 위해 등장한 것이 `Optional`이다.
새로운 연산자를 도입하는 등의 방식 대신, 에러를 예방하기 위한 방법으로 `Optional`을 사용하는 것을 권유하고 있다.

<br>

---

# Optional이란?

- `java.util.Optional`

- 래퍼 클래스 <span style="color:#737373; font-size:14px; font-weight:300;">Wrapper Class</span>의 일종이다.

- `null`이 될 가능성이 있는 객체를 감싸준다.

    - 값이 없는 객체만을 다룬다기 보다, `null`을 둘러싼 모든 상황을 위한 도구이다. 옵셔널이 제공하는 메서드를 보면 느낄 수 있을 것이다.

- 예상치 못한 `NullPointerException`를 옵셔널이 제공하는 메서드로 간단하게 회피할 수 있다.

## Null을 다루는 Optional의 메서드

- `empty()`

    - 객체의 값이 없는 경우
    - `Optional` 클래스는 내부에서 `static` 변수로 빈 옵셔널 객체를 가지고 있다.
    따라서 필요할 때마다 매번 빈 객체를 생성하지 않고 같은 객체를 참조하도록 설계되어있다고 한다.

- `of(T value)`

    - 객체가 반드시 `null`이 아닌 값을 가지는 경우
    - 값이 `null`인 경우 `NullPointerException`을 던진다.

- `ofNullable(T value)`

    - 객체의 값이 있을 수도 있고 없을 수도 있는 경우
    - `null`이 아닌 값이면 해당 값을 감싼 옵셔널을 반환하고, `null`이라면 빈 옵셔널 객체 `Optional.empty()`를 반환한다.

<br>

```java

    // 객체가 null일 때
    Optional<String> n = Optional.empty();

    // 객체가 절대 null이 아닐 때
    Optional<String> m = Optional.of();

    // 객체의 값이 null일수도 있고 아닐 수도 있을 때
    Optional<String> l = Optional.ofNullable();

```

<br>

또한 옵셔널 객체가 비어있는 경우인 `Optional.empty()`가 들어오는 때에 대한 추가적인 작업이 다시 필요해진다. `orElse()` 또는 `orElseGet()`을 사용하여 처리할 수 있다.

```java

    public String getName() {...}

    Optional<String> o = Optional.ofNullable(getName());

    // orElse() 사용
    String name = o.orElse("default");

    // 또는 orElseGet() 사용
    String name = o.orElseGet(()->"default");

```

### 💡 orElse() vs orElseGet()

- `orElse()`

    ```java

        // java API Docs
        public T orElse(T other)

        // If a value is present, returns the value, otherwise returns other.

    ```

    - `null`을 갖는 경우 **값**을 넘길 때

<br>

- `orElseGet()`

    ```java

        public T orElseGet(Supplier<? extends T> supplier)

        // If a value is present, returns the value, otherwise returns the result produced by the supplying function.

    ```

    - `null`을 갖는 경우 **메서드의 실행**이 필요할 때
    
    - 값이 없는 경우 `supplier.get()`을 호출하여 생성한 대체값을 반환하도록 한다.

    > cf. Supplier
    >
    > 자바의 `java.util.function`패키지에 포함된 함수형 인터페이스 중 하나로, 입력 없이 결과를 반환한다. 참고로 추상 메서드를 단 하나만 갖는 함수형 인터페이스의 특징은 입력 없이 값을 반환하는 동작을 수행하는 `get()` 메서드를 통해 나타난다.

<!-- cf. orElse에도 메서드를 넣을 수 있는데, 이 경우는 좀 다르다. 이때는 메서드가 일단 실행이 되고 값이 나온다. 하지만 orElseGet에 메서드가 들어갈 때는 null인 것이 확인된 이후 메서드가 실행된다. 즉 전자는 null이든 아니든 일단 메서드가 실행이 된다고 한다. -->

<br>

---

<br>

메서드를 어떤 식으로 코드에 적용하는지 알아보기 위해 `Optional`을 문제가 발생했던 코드에 적용해보겠다. 옵셔널이 어떤 느낌인지 알아보기 위한 예시일 뿐 실무에 가깝거나 가장 좋은 코드라고 할 수는 없다.

<br>

```java

    import java.util.Optional;

    class User {
        private Address address;
        private String name;

        // 여전히 이름만 초기화하는 생성자
        User(String name) {
            this.name = name;
        }

        // Optional.empty()를 반환
        public Optional<Address> getAddress() {
            return Optional.ofnullable(this.address);
        }
    }

    class Address {
        private City city;

        public Optional<City> getCity() {
            return Optional.ofNullable(this.city);
        }
    }

    class City {
        public String printCity() {
            return "Seoul";
        }
    }

```

<br>

이후 빈 객체를 다루는 로직을 설계하여 에러 발생 없이 객체를 처리할 수 있다.

```java

    public class Main {
        public static void main(String[] args) {
            User user = new User("Cat");

            Optional<Address> address = user.getAddress();

            // Optional의 isPresent 메서드를 이용한 조건문 처리
            if (address.isPresent()) {
                ...
            } else {
                ...
            }
        }
    }

```

옵셔널에서 제공해주는 `isPresent()` 메서드를 통해 값이 존재하면 `true`를 반환하여 조건문 처리를 직관적이고 편리하게 할 수 있다.  

> 좀 더 편리하게 처리하려면 객체들의 메서드와 필드를 조정하고 `orElse()` 또는 `orElseGet()`을 사용하는 것이 좋을 것 같다는 생각이 든다.
> 앞으로 서비스를 설계할 때 더 사용해보고 감을 잡아야겠다.

<br><br>
<details>
<summary> &nbsp; 📁 참고 자료</summary>
<div>
    <div>
    ❗️ <a href="https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EA%B0%9C%EB%B0%9C%EC%9E%90%EB%93%A4%EC%9D%84-%EA%B4%B4%EB%A1%AD%ED%9E%88%EB%8A%94-NULL-%ED%8C%8C%ED%97%A4%EC%B9%98%EA%B8%B0" target="_blank">개발자들을 괴롭히는 자바 NULL 파헤치기</a>
    </div>
    <div>
    ❗️ <a href="https://le2ksy.tistory.com/65" target="_blank">스프링캠프 2019 - 자바에서 null을 안전하게 다루는 방법</a>
    </div>
    <div>
    ❗️ <a href="https://kdhyo98.tistory.com/40" target="_blank">[Java] Optional – orElse() vs orElseGet() 차이점 알고 쓰자.</a>
    </div>
    <div>
    ❗️ <a href="https://mangkyu.tistory.com/70" target="_blank">[Java] Optional이란? Optional 개념 및 사용법</a>
    </div>
</div>
</details>