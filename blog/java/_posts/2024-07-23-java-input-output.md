---
layout: post
title:  "[Java] 자바 표준 입출력 "
author: "minseo"
categories: [blog, java]
tags: [Java, Spring]
comments: true
image: "/static/img/logo/java-logo.png"
hide_image: true
---
# [Java] 자바 표준 입출력

<br><center><img src="../../../static/img/logo/java-horizontal.png" width="35%"></center>
<br><br>

자바에서의 입출력은 크게 콘솔에서의 입출력 <span style="color:#737373; font-size:14px; font-weight:300;"> 표준 입출력 </span>과 파일에서의 입출력으로 나뉜다.

일반적으로 데이터의 입출력이라 여겨지는 것이 **콘솔에서의 입출력**으로, 표준 입력/표준 출력 스트림을 사용한다. **파일 입출력**은 파일에 데이터를 쓰거나 파일에서 데이터를 읽어오는 것을 말한다.

자바에서 **스트림(Stream)**이라는 개념에 의해 이루어지는 **콘솔 입출력**에 대해 알아보자.

<br>

---

# Stream

- 한 마디로 `데이터의 흐름` 이라고 할 수 있다.
    - 데이터 입출력 시 모든 데이터를 **형태와는 관계 없이 일련된 흐름으로 전송**하는 것이 스트림 입출력 모델의 기본 개념이다.
    
- 데이터를 연속적으로 읽거나 쓰는 데 사용되는 추상화된 개념이다.

- TCP 프로토콜의 네트워크에 가상 경로를 설정하고 그 경로에 데이터를 바이트로 흘려보내는 `스트림` 방식에서 유래되었다.

- 스트림에서 가장 중요한 것은 데이터가 있는 소스와 데이터가 전달되는 목적지이다.

- 프로그래밍에서는 다음과 같은 것들을 스트림이라고 할 수 있다.
    - 파일 데이터: 파일은 그 시작과 끝이 있는 데이터의 스트림이다.
    - HTTP 송수신 데이터: 브라우저가 요청하고 서버가 응답하는 HTTP 형태의 데이터도 스트림이다.
    - 키보드 입력: 사용자가 키보드로 입력하는 문자, 숫자, 특수문자 등은 스트림이다.

### 특징

- FIFO <span style="color:#737373; font-size:14px; font-weight:300;"> First In First Out. 먼저 들어온 것이 먼저 나간다. </span>
- 스트림의 데이터는 순차적으로 흘러간다.
- 스트림은 단방향이다.
    - 입력과 출력이 동시에 되지 않는다.
    - 입출력을 동시에 하려면 각각의 스트림을 하나씩 열어서 사용해야 한다.
- 스트림은 지연될 수 있다.
    - 스트림에 들어간 데이터가 처리되기 전까지는 스트림에 사용하는 스레드가 지연(blocking) 상태가 된다.

### Java의 스트림

- 바이트 스트림

    - 8비트 바이트 단위로 데이터를 처리한다.
    - 음악, 이미지, 영상 등 바이트로 구성된 파일을 처리하기에 적합하다.

    - InputStream
    - OutputStream

- 문자 스트림

    - 16비트 유니코드 문자 단위로 데이터를 처리한다.
    - 텍스트 데이터를 읽고 쓸 때 주로 사용한다.

    - Reader
    - Writer

<br>

---

# 스트림을 통한 콘솔 입출력

사용자에게 문자열을 출력하여 보여주는 것을 **콘솔 출력**, 출력된 질문에 사용자가 답변을 입력하는 것을 **콘솔 입력**이라고 한다. 인텔리제이의 콘솔 창, 윈도우 cmd 창 등 사용자의 입력을 받거나 문자열을 출력해주는 역할을 한다면 전부 콘솔이라고 할 수 있다.

## 콘솔 입력

- InputStream
- InputStreamReader
- BufferedReader
- Scanner

### InputStream

자바에서는 `InputStream`의 객체인 `System.in`을 사용하여 사용자가 입력한 문자열을 얻을 수 있다.

```java

    import java.io.IOException;
    import java.io.InputStream;

    public class InputStreamSample {
        public static void main(String[] args) throws IOException {

            InputStream in = System.in;

            // 1 byte 입력
            int n;
            n = in.read();

            // 3 byte 입력
            byte[] s = new byte[3];
            // read 메서드의 입력값으로 배열을 전달하면 콘솔 입력이 해당 배열에 저장
            in.read(s);

            // n 출력
            System.out.println(n);

            // s 출력
            System.out.println(s[0]);
            System.out.println(s[1]);
            System.out.println(s[2]);
        }
    }

```

- `InputStream`은 자바의 내장 클래스이다.
    - `java.lang` 패키지에 속해 있지 않은 자바 내장 클래스 `import java.io.InputStream;`를 임포트해서 사용해야 한다.
    - **cf.** `import java.io.IOException;`은 입력 작업 중 발생할 수 있는 예외를 처리하기 위해 임포트해준다.

- `InputStream`의 `read()`는 1byte 크기의 입력을 받는다.
    - 그러나 이 데이터는 `byte` 자료형으로 저장되지 않고 `int`로 저장된다.
    - 0-255 사이의 아스키코드값으로 저장된다.
    - 1바이트 이상의 데이터를 전달할 시 1바이트만 읽을 수 있다.

### InputStreamReader

`InputStream`을 이용한 입력은 int 타입의 아스키코드값으로 저장이 되어 값을 그대로 사용하기에 불편하다는 문제점이 있다. 이렇게 **byte 대신 문자**로 입력 스트림을 읽기 위해 `InputStreamReader`를 사용한다.

```java

    import java.io.IOException;
    import java.io.InputStream;
    // InputStreamreader를 사용하기 위한 import
    import java.io.InputStreamReader;

    public class InputStreamReaderSample {
        public static void main(String[] args) throws IOException {

            InputStream in = System.in;

            // 생성자 입력으로 InputStream 객체 in을 사용
            InputStreamReader reader = new InputStreamReader(in);
            char[] c = new char[3];
            reader.read(c);

            // 출력
            System.out.println(c);
        }
    }

```

- `InputStreamReader`는 객체 생성시 생성자의 입력으로 `InputSteam` 객체가 필요하다.

- `InputStreamReader`의 `read()`가 입력을 받는다.
- 내부의 바이트스트림 `InputStream`에서 바이트를 읽고 문자열로 변환해준다.
- 따라서 이전 코드에서는 읽을 값을 `byte` 배열에 담았는데, `InputStreamReader`를 사용하면 `char` 배열을 사용할 수 있게 된다.

- 프로그램을 실행하고 입력을 완료하면 입력된 문자열이 한꺼번에 출력된다.


### BufferedReader

`InputStreamReader`를 이용한 방식은 고정된 길이로만 스트림을 읽어올 수 있다. 따라서 이 문제점을 해결하기 위해 `BufferedReader`를 사용할 수 있다.

```java

    import java.io.IOException;
    // BufferedReader를 사용하기 위한 import
    import java.io.BufferedReader;
    import java.io.InputStream;
    import java.io.InputStreamReader;

    public class BufferedReaderSample {
        public static void main(String[] args) throws IOException {

            InputStream in = System.in;
            InputStreamReader reader = new InputStreamReader(in);
            // InputStreamReader의 객체 reader를 사용
            BufferedReader br = new BufferedReader(reader);

            // 입력
            String s = br.readLine();

            // 출력
            System.out.println(s);
        }
    }

```

- `BufferedReader`는 객체 생성시 생성자의 입력으로 `InputStreamReader`의 객체가 필요하다.

- 내부적으로 **버퍼**를 사용하여 입력을 효율적으로 처리한다. 따라서 사용자가 입력한 문자열 전부를 읽을 수 있게 된다.

    - 버퍼를 사용하지 않으면 사용자가 키보드를 누름과 동시에 데이터가 전달된다. 즉 입력한 문자가 입력할 때마다 하나하나 전송된다고 이해할 수 있다. 그러나 버퍼를 사용하면 버퍼가 다 차거나 개행문자가 나타날 때 모아둔 데이터를 한번에 전송한다. 따라서 버퍼를 사용하면 효율적으로 데이터를 전송할 수 있어 입력 <span style="color:#737373; font-size:14px; font-weight:300;"> 출력도 마찬가지 </span> 속도가 빠르다.

- `BufferedReader`의 `readLine()`을 사용하면 곧바로 `String` 타입의 변수에 읽어온 데이터를 저장할 수 있다.


### Scanner

J2SE <span style="color:#737373; font-size:14px; font-weight:300;"> Java 2 Standard Edition </span> 5.0부터 `java.util.Scanner` 클래스가 추가되어 조금 더 쉽게 콘솔 입력을 할 수 있게 되었다.

```java

    // 클래스 import
    import java.util.Scanner;

    public class ScannerSample {
        public static void main(String[] args) {

            // 입력
            Scanner sc = new Scanner(System.in);

            // 출력
            System.out.println(sc.next());
        }
    }

```

- `java.utill.Scanner` 클래스 임포트가 필요하다.
- `Scanner` 클래스는 생성자 입력으로 `InputStream`의 `System.in` 객체가 필요하다.
    - 입력 소스가 콘솔 입력 `InputStream`이고, `Scanner`는 입력 데이터를 손쉽게 파싱하기 위한 메서드를 제공하는 것이다.
- `Scanner` 객체의 `next()`는 한 개의 토큰을 읽어들인다.

    > cf. 토큰?
        정보의 최소 단위.
        공백으로 구분되는 단어, 숫자, 기호 등의 문자열.

<br>

- **`Scanner`의 메서드**

    - `next()`: String을 읽음. 한 개의 토큰을 읽음, 즉 띄어쓰기 뒷부분은 읽지 않음.
    - `nextline()`: String을 읽음. Enter 입력 전 띄어쓰기를 포함한 한 줄을 읽음.
    - `nextInt()`: int를 읽음.
    - `nextLong()`: long을 읽음.
    - `nextBoolean()`: boolean을 읽음.

<br>

---

## 콘솔 출력

- System.out <span style="color:#737373; font-size:14px; font-weight:300;"> PrintStream 클래스의 객체 </span>

### System.out.println

앞서 등장했던 `System.out.println`메서드를 사용하여 콘솔에 문자열을 출력하거나 디버깅을 하는 용도로 사용할 수 있다.

```java

    public class PrintSample {
        public static void main(String[] args) {
            System.out.println("안니옹하세요.");
            System.out.print("자바의 콘솔");
            System.out.print(" 출력입니다. ");
            System.out.printf("%d월 화이팅 ~", 7);
            
            // 안니옹하세요.
            // 자바의 콘솔 출력입니다. 7월 화이팅 ~
        }
    }

```

- `System.out`은 `PrintStream` 클래스의 객체다.
    - **cf.** `OutputStream`을 상속받는 `PrintStream`은 데이터를 출력할 수 있게 해주는 자바의 출력 스트림 클래스이다.
    - `PrintStream` 클래스를 직접 사용하지 않으므로 따로 클래스를 임포트할 필요가 없다.

- 콘솔에 문자열을 출력할 때 `System.out`을 사용한다.
- 오류 메시지를 출력할 때에는 `System.err`를 사용한다.

    - `print`: 줄바꿈 없이 데이터를 출력함.
    - `println`: 데이터를 출력한 뒤 줄바꿈을 함.
    - `printf`: 변환명세를 사용하여 형식화된 데이터를 출력함. 줄바꿈 없음.

<br><br>
<details>
<summary> &nbsp; 📁 참고 자료</summary>
<div>
    <div>
    ❗️ <a href="https://wikidocs.net/226" target="_blank">점프 투 자바 06-01 콘솔 입출력</a>
    </div>
    <div>
    ❗️ <a href="https://blog.naver.com/tndus0450/220156491872" target="_blank">[자바입출력]자바의 입출력</a>
    </div>
    <div>
    ❗️ <a href="https://velog.io/@cse_pebb/Java-Scanner-%EB%A9%94%EC%84%9C%EB%93%9C-%EC%B4%9D%EC%A0%95%EB%A6%AC" target="_blank">Java Scanner 메서드 총정리</a>
    </div>
</div>
</details>