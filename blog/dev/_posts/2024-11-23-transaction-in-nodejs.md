---
author: "minseo"
categories: [blog, dev]
tags: [DB, nodejs, mysql]
comments: true
image: "/static/img/logo/nodejs-logo.png"
hide_image: true
---
# [Node.js] Transaction으로 동시성 문제 해결하기

<br><br>
<center><img src="../../../static/img/logo/node-js.png" width="40%"></center>
<br><br>

항공기 예약, 은행, 신용 카드 처리, 대형 할인점 등에서는 대규모 데이터베이스에 수백, 수천 명 이상의 사용자들이 동시에 접근한다. 즉 많은 사용자들이 동시에 서로 다른 데이터 또는 동일한 데이터에 접근하며 데이터베이스를 사용한다.

데이터베이스의 동일한 부분에 여러 사용자가 동시에 접근하게 된다면 그 결과가 어떻게 처리되어야 할까? 이런 경쟁 상태로 인해 데이터의 결과가 비결정적으로 나오는 문제를 해결하기 위해 데이터베이스에서는 **트랜잭션**이라는 개념을 사용한다.

이전에 개발 도중 트랜잭션 처리를 해주지 않아 의도치 않은 결과가 나왔던 적이 있다. 문제를 해결하려면 프로젝트의 전반적인 구조 자체를 변경해야 했고, 그대로 유지보수가 중단되어 넘겨버렸던 그때의 문제 상황을 **트랜잭션을 통해 어떻게 해결할 수 있을지** 생각해보고자 한다. 이참에 트랜잭션의 개념도 간단히 정리해보려고 하는데 실제 코드에 적용하는 부분이 궁금하다면 [여기](https://mingdodev.github.io/blog/dev/2024-11-23-transaction-in-nodejs/#transaction이-필요한-문제-상황)로 바로 넘어가도 좋다.

<br>

---

# Transaction

트랜잭션이란 데이터베이스의 상태를 변화시키기 위해 수행하는 작업의 단위이다.  
트랜잭션 내에서 실행한 하나 또는 여러 개의 작업은 **하나의 단위**로 취급된다.

<br>

DBMS의 관점에서 조금 더 구체적으로 말하자면

- 데이터베이스에서 하나의 논리적인 단위를 수행하는 데이터베이스 연산들의 모임

- 데이터 객체에 접근하거나 갱신하는 프로그램 수행의 단위

라고 볼 수 있다.

## 동시성 제어와 회복

동시성 제어와 회복은 데이터베이스의 필수적인 메커니즘이다.  
동시성 제어와 회복을 위한 처리들은 트랜잭션이 ACID 특성을 가지도록 보장해준다.

### 동시성 제어 (Concurrency Control)

- 다수의 사용자가 데이터베이스를 동시에 접근하도록 허용하면서 데이터베이스의 일관성을 유지하는 것

- 동시에 수행되는 트랜잭션들이 데이터베이스에 미치는 영향은, 이들을 순차적으로 수행했을 때 데이터베이스에 미치는 영향과 같도록 보장되어야 한다.

### 회복 (Recovery)

- 데이터베이스를 갱신하는 도중에 시스템이 고장 나도 데이터베이스의 일관성을 유지하는 것

- 트랜잭션이 수행되다가 시스템이 다운되면 완료되지 못한 트랜잭션이 롤백되거나 <span style="color:#737373; font-size:14px; font-weight:300;"> UNDO </span> 이미 완료된 작업이 다시 반영되어야 한다. <span style="color:#737373; font-size:14px; font-weight:300;"> REDO </span>

## 트랜잭션의 특성 (ACID)

트랜잭션은 다음과 같은 네 가지 특성을 만족해야 한다.

### 1. 원자성 (Atomicity)

- 하나의 트랜잭션 내의 모든 연산들이 완전히 수행되거나 전혀 수행되지 않는다. <span style="color:#737373; font-size:14px; font-weight:300;"> all or nothing </span>

- 데이터베이스 관리 시스템의 회복 모듈은 트랜잭션의 원자성을 보장한다.

### 2. 일관성 (Consistency)

- 어떤 트랜잭션이 수행되기 전에 데이터베이스가 일관된 상태를 가졌다면 트랜잭션이 수행된 후 데이터베이스는 또 다른 일관된 상태를 가진다.

- 트랜잭션이 수행되는 도중에는 데이터베이스가 일시적으로 일관된 상태를 갖지 않을 수도 있다.

### 3. 고립성 (Isolation)

- 다수의 트랜잭션들이 동시에 수행되더라도 그 결과는 어떤 순서에 따라 트랜잭션들을 하나씩 차례대로 수행한 결과와 같다.

- 하나의 트랜잭션이 데이터를 갱신하는 동안, 이 트랜잭션이 완료되기 전에는 갱신 중인 데이터에 다른 트랜잭션들이 접근하지 못한다.

- DBMS의 동시성 제어 모듈이 트랜잭션의 고립성을 보장한다. <span style="color:#737373; font-size:14px; font-weight:300;"> 다양한 고립수준을 제공 </span>

### 4. 지속성 (Durability)

- 완료된 트랜잭션의 효과는 지속적이다.

- 일단 한 트랜잭션이 완료되면 이 트랜잭션이 갱신한 것은 그 후에 시스템에 고장이 발생하더라도 손실되지 않는다.

- DBMS의 회복 모듈은 시스템이 다운되는 경우에도 트랜잭션의 지속성을 보장한다.

<br>

---

# Transaction이 필요한 문제 상황

피로그래밍에서 사용할 동아리 전용 앱을 개발할 때 트랜잭션 처리를 해주지 않아 치명적인 문제가 발생했었다.

나는 동아리 운영을 위한 핵심 기능 중 **과제**와 관련한 기능을 맡아, 매 세션마다 나오는 과제를 채점하여 그 결과에 따라 회원들의 보증금을 유지하거나 차감하는 기능을 구현했다.

<center><img src="../../../static/img/241119/piro-app-assignment.png" width="100%"></center>
<div class="figcaption"> 앱 내에서의 과제 채점 프로세스 </div>

요구 사항을 충족시키기 위해서는 다음과 같은 순서로 데이터베이스 갱신이 이루어져야 한다.

1. 과제 결과 **CREATE** <span style="color:#737373; font-size:14px; font-weight:300;"> 업데이트는 또 다른 로직과 연관되기 때문에, 보다 명확한 설명을 위해 다루지 않겠다. </span>

    - 과제 결과 튜플(행)은 채점 결과 <span style="color:#737373; font-size:14px; font-weight:300;"> 제출, 미흡, 지각, 미제출 </span> 데이터를 가지며, 외래키로 과제 ID와 사용자 ID를 가진다.

    - 즉 누가 어떤 과제에 대해 어떤 점수를 부여받았는지를 기록하는 테이블이다.

2. 회원의 현재 보증금 금액 **READ**

3. 과제 결과에 따른 보증금 금액 **UPDATE**

### DB 구조

관계형 데이터베이스 MySQL을 사용하였으며, 해당 기능 구현에는 **과제, 과제 결과, 회원** 테이블이 관여한다.

<img src="../../../static/img/241119/piro-app-relations.png" width="80%">

- 과제와 회원 간의 다대다 관계에 의해 과제 결과 테이블이 필요해졌음을 알 수 있다.

- 과제 테이블 : 과제 결과 테이블 `1 : N`

    - 과제 결과는 **특정** 과제에 대한 결과이다.

    - 과제에 대한 과제 결과는 없을 수도 있고 있을 수도 있다.

- 과제 결과 테이블 : 회원 테이블  `M : 1`

    - 과제 결과는 **특정** 회원에 대한 과제 채점 결과이다.

    - 특정 과제에 대한 과제 결과는 회원 당 0개 또는 1개만 가질 수 있다.

<br>

> 해당 프로젝트에서는 ORM 기술을 활용하지 않고 직접 쿼리를 작성해 `MySQL` 데이터베이스에 요청하는 방식을 사용했다.

### 기존 Controller 코드

과제 최초 채점시 과제 결과를 생성하고 수정하는 코드에서 문제가 발생한다. 수정 작업은 연관 관계가 훨씬 복잡하기 때문에, 간단한 과제 결과 생성 기능을 중점적으로 문제를 파악하고 개선해보려고 한다.

- `Router` → `Controller` → `Model`의 흐름으로 컨트롤러가 모델을 호출하여 DB 쿼리 작업을 수행한다.

    - 하나의 트랜잭션으로 수행되어야 하는 쿼리들이 모델의 메서드 `assignModel.createGrade()`, `depositModel.updateDeposit()`에서 독립적으로 수행되고 있다.

    - 심지어 같은 모델이 아닌 서로 다른 모델에서 처리되고 있다.

    ```js
        // file: "assingmentController.js"
        createGrade: async (req, res) => {
        
            // 응답으로부터 필요한 변수 추출 (생략)

            // 회원의 과제 결과 생성
            const data = await assignModel.createGrade(assignScheduleId, userId, inputGrade);

            // 과제 결과에 따른 보증금 금액을 업데이트
            switch (inputGrade) {
                case 0:
                    await depositModel.updateDeposit(userId, -20000);
                    break;
                case 1:
                case 2:
                    await depositModel.updateDeposit(userId, -10000);
                    break;
            }
            ...
        }

    ```

### 기존 Model 코드와 Query

- `assignModel`에서 과제 결과를 생성하는 `MySQL` 쿼리를 수행한다.

    ```js
        // file: "assignmentModel.js"
        createGrade: async (assignScheduleId, userId, inputGrade) => {
            const query = `
                INSERT INTO
                Assign (user_id, grade, assignschedule_id)
                VALUES (? , ? , ?);
                `;

            await db.query(query, [userId, inputGrade, assignScheduleId]);
        }

    ```

- `depositModel`에서 보증금을 업데이트하는 `MySQL` 쿼리를 수행한다.

    ```js
        // file: "depositModel.js"
        updateDeposit: async (userId, adder) => {
            const query = `
                UPDATE User
                SET deposit = deposit + ?
                WHERE user_id = ?;
                `;

            await db.query(query, [adder, userId]);
        }

    ```

## 어떤 문제가 발생했는가?

평소와 같이 과제를 채점하던 중, 보증금 금액이 원래 차감되어야 하는 값보다 더 많이 깎여있는 회원 A를 발견했다.

<img src="../../../static/img/241119/piro-app-deposit.png" width="100%">

- 과제 채점 전, A의 보증금은 5만원이었다.

- 과제 채점 이후, 1만원이 차감된 4만원으로 보증금이 업데이트되어야 한다. 그러나 A의 보증금은 3만원이 되었다.

- 과제 결과 데이터는 중복 생성될 수 없도록 클라이언트 단에서 예외 처리가 되어있다.

    - ❗️ **문제 발생 가능성 1.** 그러나 현재 코드상에서는 과제 결과에 대한 CREATE 작업이 완료되기 전에 READ 작업이 수행될 수 있다. 각각의 작업이 여러 개의 독립적인 트랜잭션이기 때문이다. 따라서 과제 결과가 중복 생성되어 보증금 차감이 중복으로 수행될 가능성이 있다. <span style="color:#737373; font-size:14px; font-weight:300;"> 데이터베이스에서도 유일성을 보장해주는 처리가 없었기에, 오류 없이 중복 생성될 것이다.. </span>

- 위의 보증금 현황은 회원 테이블의 컬럼에서 조회해오는 값이고, 아래의 보증금 내역은 과제 결과 테이블의 데이터를 이용해 따로 계산되는 값이다. 따라서 아래의 보증금 내역은 동일한 과제 결과가 두 개 이상 생성되지 않는다면 항상 올바른 결과를 도출한다.

    - 보증금 내역을 확인했을 때에는 과제 결과 데이터가 하나만 정상적으로 생성되었다. 따라서 **1번의 이유 때문에 문제가 발생한 것은 아니다.**

따라서 이러한 문제가 발생한 이유는 다음과 같다.

## 왜 문제가 발생했는가?

수행되는 작업을 단순화해보면 아래와 같은 두 메서드의 실행으로 볼 수 있다.

```js
    // file: "assingmentController.js"
    createGrade: async (req, res) => {
        ...
        await assignModel.createGrade(assignScheduleId, userId, inputGrade);
        ...
        await depositModel.updateDeposit(userId, -10000);
        ...
    }

```

<br>

<center><img src="../../../static/img/241119/piro-app-lost-update.png" width="90%"></center>

이를 기준으로 메서드의 실행 순서에 따른 결과를 추적해보면 위와 같다. 트랜잭션으로 묶이지 않은 각각의 작업들이 어떤 순서에 의해 독립적으로 실행되어, 잘못된 값이 도출되었음을 알 수 있다.

개념적으로 하나의 단위로 실행되어야 하는 작업들이기 때문에 **Task의 작업들이 모두 성공하거나 모두 실패해야 하는데, 일부만 성공**했기 때문에 의도치 않은 결과를 초래한 것이다.

따라서 이 메서드들, 즉 작업들이 **프로그램에서 다른 Task가 개입할 여지가 없는 하나의 단위로 실행**되도록 트랜잭션 처리를 해 문제를 해결할 것이다.

<br>

다음 글에서는 프로젝트 구조 변동과 더불어, 서버 애플리케이션 측에서 트랜잭션 처리를 하는 방법, MySQL 쿼리를 통해 트랜잭션 처리를 하는 방법으로 나눠 동시성 문제를 해결하는 아이디어에 대해 소개하도록 하겠다!

<!-- <br>

---

# Transaction 적용하여 해결해보기

당시 섣불리 트랜잭션을 구현하지 못했었는데, 그 이유는 묶어야 할 쿼리를 제어하는 메서드가 여러 컨트롤러에 분산되어있었기 때문이다. 채점 결과가 수정되는 경우는 로직이 훨씬 복잡했고 과제 기능뿐만 아니라 다른 기능도 함께 변경되어야 했다. 그래서 실제 애플리케이션에 반영은 못하더라도 프로젝트 구조를 개선할 방법에 대해 몇 가지 아이디어를 떠올려보고 학습에 활용하고자 한다.

- 프로젝트 구조 변동

- 을 떠나서 일단 쿼리를 짜보기

- mysql -->