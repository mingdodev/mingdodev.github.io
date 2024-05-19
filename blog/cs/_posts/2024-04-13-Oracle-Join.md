---
layout: post
title:  "[Database] Join"
author: "minseo"
categories: [blog, cs]
tags: [Oracle, DB]
comments: true
image: "/assets/img/post-cover.png"
hide_image: true
---
# [Database] Join

<center><img src="https://allvectorlogo.com/img/2017/02/oracle-database-logo.png" width="50%"></center>

# Join이란?

복잡한 Join 연산을 공부하기에 앞서…  
Join 연산이 필요한 이유를 떠올려보자.  
어쨌든 우리는 여러 개의 테이블에 흩어져있는 데이터를 하나로 모아 함께 살펴보고 싶은 것이다.  

***“ Join은 2개 이상의 테이블을 결합하여 데이터를 조회하는 연산이다. “***  

이것 하나만 머릿속에 새겨두고 하나씩 차근차근 이해해보도록 하자!  

## Types of Joins

Cross Join / Cartesian Product, Inner Join, Natural Join, Non-equi Join, Self Join, Outer Join에 대해 알아보자.  
<br>

> 기본 SQL Join에 관한 내용을 Oracle Database를 기반으로 설명한다.  
> Oracle에서 사용하는 문법과, 조인의 종류를 명시하는 ANSI 문법을 비교하여 소개한다.  

<br>

---

# Cross Join / Cartesian Product

- 크로스 조인 / 카티션 곱
- 두 개의 테이블 사이 모든 가능한 조합 생성
- 카티션 곱을 사용할 일은 거의 없다.  
    따라서 조인 결과가 카티션 곱이라면 쿼리문에 오류가 있는지 다시 한번 살펴보도록 하자…

### ex. 사원 테이블과 부서 테이블의 크로스 조인

- Oracle 문법: `FROM`절에 테이블을 `,`로 연결하고 `WHERE`절에 조인 조건을 작성한다.  
            조인 조건이 없으므로 크로스 조인된다.

```sql
    SELECT E.last_name, D.department_id
    FROM employees E, departments D;
```

- ANSI 문법: `CROSS JOIN`을 사용하고 `ON`과 `USING`을 작성하지 않는다.  

```sql
    SELECT E.last_name, D.department_id
    FROM employees E CROSS JOIN departments D;
```


# Inner Join

- 내부 조인
- 조인 조건이 정확히 일치할 때 조회
- 기본적으로 `JOIN` 또는 `FROM`절에서 `,` 을 사용하여 연결하면 Inner join을 수행한다.
- 조건이 설정되지 않은 Inner join → Cross join
    - 개념적으로 이해하자! 실제로 ANSI 문법을 사용할 경우에는, `JOIN`을 사용했지만 `ON`으로 조인 조건을 작성하지 않을 경우 오류(ORA-00905: missing keyword)가 발생한다.

### ex. 사원과 부서 테이블을 부서 번호를 기준으로 내부 조인

```sql
    SELECT E.last_name, D.department_id
    FROM employees E JOIN departments D
    ON E.department_id = D.department_id;
```
    
### ❓ cf. Equi Join (동등 조인)과 Inner Join (내부조인)

전부 조인 조건을 만족하는 컬럼만 반환하는 연산이지만,  
엄밀히 말하면 동등 조인과 내부 조인에는 차이가 있다.  
**동등 조인**은 `=` 연산자를 사용, **내부 조인**은 `=, >, <, <>` 연산자를 전부 사용하는 연산을 포함한다.  
여기서 동등 조인이란 세타 조인 중에서 비교연산자가 =인 조인을 의미한다.  
따라서, 세타 조인의 개념을 떠올려보면 좀 더 쉽게 이해할 수 있다!  

- **Theta join**

    - 두 릴레이션 R(A1, A2, … , An)과 S(B1, B2, …, Bm)의 세타 조인 결과는, 차수가 n+m이고, 애트리뷰트가 (A1, A2, …, An, B1, B2, …, Bm)이며, 조인 조건을 만족하는 투플들로 이루어진 릴레이션이다.
        
        cf. 릴레이션: 테이블, 차수: 애트리뷰트 수, 애트리뷰트: 속성. 학생 테이블의 ‘이름, 나이, 학번…’ 
        
    - 세타 조인의 결과는 두 릴레이션(테이블)의 카티션 곱에 조인 조건을 적용한 결과와 같다.

<center><img src="../../../static/img/240403/Theta Join.png" width="90%"></center>

- 여기서 또 한 가지 의문이 생긴다. 그렇다면 세타 조인과 내부 조인은 어떤 차이가 있는가?

    - 일반적으로 ‘세타 조인’이라고 하면 내부 조인의 개념으로 사용하는 경우가 많다. 즉 그냥 내부 조인의 일반화된 형태라고 보면 된다.  
    세타 조인이 포괄적인 개념의 느낌, 내부 조인은 실제로 사용하는 도구의 명칭인 느낌..?  
    아무튼 같은 선상에 놓고 비교할 일은 아닌 것 같다. <span style="color:lightgray">그냥 궁금한 걸 못 참아서 알아봤다</span>

- 외부 조인도 `=` 연산자를 사용하는 경우가 있다. 따라서 외부 조인도 동등 조인이 될 수 있다고 보는 견해가 있으나, 세타 조인의 정의를 생각해보면 틀린 의견이라고 생각한다. 외부 조인은 조건을 만족하지 않는 컬럼도 전부 반환하기 때문이다. 이 논점에 대해 정확히 알고 있는 사람이 있다면 댓글로 알려주세요! 궁금 🤔


# Natural Join
- 자연 조인
- 두 개의 테이블에서 **이름이 같은 모든 컬럼**을 조인 조건으로 설정하여 조회
- 같은 이름의 컬럼이지만 다른 데이터 형식을 갖고 있다면 에러를 반환한다.
- 내부 조인에 속한다.
- Oracle에서 `NATURAL JOIN`이라고 명시하여 사용
- Alias, 즉 Qualifier을 사용할 수 없다.

### ex. 부서 테이블(부서 번호, 부서 이름, 위치 번호)과 위치 테이블(위치 번호, 도시)을 자연 조인

```sql
    SELECT department_id, department_name, location_id, city
    FROM departments NATURAL JOIN locations;
```


# Non-equi Join
- 비동등 조인
- 동등 연산자가 아닌 연산자를 포함하는 조인
    - 비등호(<, >, <=, >=) 또는 비교 연산자(!=, <>, BETWEEN, LIKE 등)
- 테이블의 어떤 컬럼도 조인할 테이블과 일치하지 않는 경우 사용할 수 있다

### ex. 월급이 특정 범위인 사원들의 이름과 부서 이름을 조회

```sql
    SELECT E.last_name, D.department_name
    FROM employees E JOIN departments D
    WHERE E.department_id = D.department_id
    AND E.salary between 10000 and 20000;
```


# Self Join
- 셀프 조인
- 자체 테이블에서 조인하여 조회
- 말 그대로 조인의 대상이 자기 자신이라는 것  
    - e.g. `학생` 테이블의 `짝꿍` 컬럼에 `학생` 테이블의 `학생 번호`가 들어갈 수 있다.  
    즉 `학생` 테이블의 컬럼에서 또 다시 `학생` 테이블의 정보를 참조하는 것.

### ex. employee 테이블에서 사원과 그의 매니저 이름을 함께 조회

```sql
    SELECT worker.last_name || 'works for' || manager.last_name
    FROM employees worker, employees manager
    WHERE worker.manager_id = manager.employee_id;
```
다음과 같이 Alias를 사용하여 employee 테이블을 셀프 조인할 수 있다.


# Outer Join
- 외부 조인
- 조인 조건을 만족하지 않는 행도 조회
- RIGHT OUTER JOIN, LEFT OUTER JOIN, FULL OUTER JOIN
    - 오른쪽 외부 조인은 조인의 오른쪽 테이블이 기준, 왼쪽 외부 조인은 조인의 왼쪽 테이블이 기준 테이블이다.
        **모든 행을 살리고 싶은 테이블**을 기준 테이블로 설정하면 된다!
    - FULL OUTER JOIN은 양쪽 테이블 모두에서 조인 조건과 일치하지 않는 행도 반환한다.

- Oracle에서 외부 조인 연산자는 `(+)`이다.
    - `(+)`이 붙은 테이블이 조인 대상 테이블, 없는 테이블이 기준 테이블이다.
    - 일치하는 행이 없는 테이블의 열 이름 뒤에 `(+)`를 붙인다.

### ex. 모든 사원의 이름, 부서 번호, 부서 이름을 조회

- ORACLE 문법
```sql
    SELECT e.last_name, e.department_id, d.department_name
    FROM employees e, departments d
    WHERE e.department_id = d.department_id(+);
```

- ANSI 문법
```sql
    SELECT e.last_name, e.department_id, d.department_name
    FROM employees e LEFT OUTER JOIN  departments d
    ON d.department_id = e.department_id;
```

<br>

---

<br>

복잡한 조인 연산의 종류에 머리 아플 수 있지만..  
개념의 포함 관계에 얽매이기보다, 각 조인의 핵심적인 특징이 무엇인지 정확히 알아두면  
원하는 데이터를 얻고싶을 때 충분히 잘 사용할 수 있을 것이다! 
<br><br>

❗️ <a href="https://livesql.oracle.com/" target="_blank">Oracle Live SQL</a>
<br><br>

Oracle Live SQL 코드 라이브러리의 HR 데이터베이스를 사용하면 글에서 소개한 예제와 동일한 데이터로 실습을 해볼 수 있다!  
우리 함께 익숙해질 때까지 연습해보자 ~~  
모두들 오늘도 화이팅 😎