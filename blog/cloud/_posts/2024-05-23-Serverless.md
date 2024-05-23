---
layout: post
title:  "[AWS] Serverless"
author: "minseo"
categories: [blog, cloud]
tags: [AWS, ACC]
comments: true
image: "/static/img/logo/aws-logo.png"
hide_image: true
---
# [AWS] Serverless

<br><br>
<center><img src="../../../static/img/240330/AWS-logo.png" width="30%"></center><br><br><br>

<span style="font-size:20px;"><strong>Serverless란!</strong></span> 서버가 없다는 뜻이 아니다.  

개발자가 서버를 직접 관리할 필요가 없는 아키텍처를 Serverless라고 한다.

- `Auto Scaling` : 사용량에 따라 자원이 할당된다.
- 사용한 만큼 값을 지불한다.
- 높은 가용성과 보안을 가진다.
- 서버에 대해 일절 관리할 필요가 없어 비즈니스 로직에 집중할 수 있다.

### BaaS(Backend as a Service) vs FaaS(Function as a Service)

<center><img src="../../../static/img/20240522/serverless-model.png" width="80%"></center>

# AWS lambda

- 서버리스 컴퓨팅 **FaaS**
- Function 단위로 애플리케이션을 올린다.
- 이벤트가 발생했을 때 Function 실행
- Serverless Application
    - 프로비저닝 관리할 필요 없이 코드 실행 가능
    - 실행한 만큼 과금
    - 사용량에 따라 자동으로 확장
- 에러 발생 시 자동 Failure Handling
- 독립적인 보안

### 장점

- 비용 절감 (순수한 코드 실행에 따른 요금)
- 여러 언어 지원 (Java, Go, JS, Ruby, Python)
- 완전한 자동화
- 분석과 **모니터링 기능**
    - CloudWatch

### 단점

- Cold Start
- 긴 시간을 요하는 작업에 불리함
- Stateless
- 클라우드 제공 플랫폼에 종속적

특정 이벤트가 발생했을 때 Lambda가 실행되어서 코드에 작성해둔 로직을 수행한다.

Amazon CloudWatch로 애플리케이션을 실시간 모니터링하여, 권한 부여된 Lambda가 EC2를 종료하거나 실행할 수 있도록 한다. 사용하지 않을 때에는 비용이 들지 않도록 만들 수 있다.

람다를 호출할 수 있는 함수를 지닌 서비스들이 다양하다 ~

## vs EC2

- AWS Lambda
    - 서버 관리 부담을 줄이고 싶은 경우
    - 특정 시간/기간만 사용하는 경우
- Amazon EC2
    - 상태 정보를 유지할 때 (Stateful)
    - 대용량 데이터 처리 및 고성능 컴퓨팅 작업일 때

# Amazon API Gateway

- 개발자가 쉽게 API를 발행할 수 있도록 도와주는 서비스
- Serverless 애플리케이션 구축
- 백엔드 서비스와 API 사용자 사이에 위치하여
API 엔드포인트에 대한 HTTP 요청을 처리하고 올바른 백엔드로 라우팅


<br><br>
<details>
<summary> &nbsp; 📁 참고 자료</summary>
<div markdown="1">
- ACC KHU 8주차 세션
</div>
</details>