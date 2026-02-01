---
layout: post
title:  "[Spring] Spring Cloud Gateway MSA 환경에서 Swagger UI 연동하기 (1)"
author: "minseo"
categories: [blog, dev]
tags: [spring, msa, swagger]
comments: true
image: "/static/img/logo/spring-cloud-logo.png"
hide_image: true
---

# [Spring] Spring Cloud Gateway MSA 환경에서 Swagger UI 연동하기 (1)

<br><br><center><img src="../../../static/img/logo/spring-cloud-logo-big.png" width="40%"></center><br><br><br><br>

최근 강의를 통해 Spring Cloud 기반의 MSA 프로젝트를 학습하며 전체적인 아키텍처를 익히고 있었다. 강의에서는 Swagger UI를 단순히 개별 마이크로서비스에 붙여 개념을 소개하는 정도로만 다뤘는데, 나는 조금 더 보완해 실제 MSA 환경처럼 **Gateway 뒤에 숨겨진 서비스들의 문서를 통합 관리하는 환경**을 직접 구축해보고 싶었다.

프로젝트의 완성도를 높여보겠다는 가벼운 마음으로 시작했지만, Gateway를 통해 각 서비스의 Swagger UI에 접근하려 하자 예상치 못한 경로 매칭 문제가 발생했다. 이번 글에서는 그 과정에서 겪은 트러블 슈팅을 정리해 보았다.

<br>

---

## 문제 상황

현재 시스템의 기본 구조는 다음과 같다.

- **API Gateway**: 외부 요청의 단일 진입점
- **Microservices**: Gateway 뒤에 위치하며, 서비스별 prefix(예: `/user-service/**`)를 기준으로 라우팅
- **Path Rewrite**: 내부 서비스로 요청을 전달할 때 `RewritePath` 필터를 통해 prefix 제거

### 발생한 문제

각 서비스(예: User Service)에서 직접 `/swagger-ui/index.html`을 호출하면 정상적으로 동작하지만, Gateway(8080)를 통해 접근하면 **401 Unauthorized** 에러가 발생했다.

```yaml
# file: 'application.yml'
- id: user-service-public-docs
  uri: lb://USER-SERVICE
  predicates:
    - Path=/user-service/swagger-ui/**, /user-service/v3/api-docs/**
  filters:
    - RemoveRequestHeader=Cookie
    - RewritePath=/user-service/(?<segment>.*), /$\{segment}
```

Gateway 설정을 통해 swagger 관련 요청들은 인증 필터를 거치지 않도록 설정해준 상태였음에도, `http://localhost:8000/user-service/swagger-ui/index.html`와 같은 주소로 접속하자 401 에러가 떴다.

<br>

---

## 원인 분석: Swagger UI의 동작 방식

문제의 핵심은 Swagger UI가 단순한 정적 페이지가 아니라는 점에 있었다. Swagger UI는 렌더링을 위해 내부적으로 다음과 같은 추가 요청을 보낸다.

1. `/swagger-ui/index.html` 로드

2. 내부 요청 발생: `/v3/api-docs`, `/swagger-config` 등 API 정의 데이터 호출

3. 응답 데이터를 기반으로 UI 렌더링

이때 2번 과정에서 **경로 불일치**가 발생한다. Gateway 설정에서 prefix를 제거하고 있었기 때문에, Swagger UI가 내부적으로 보내는 요청에는 /user-service라는 정보가 사라진 상태였다.

Gateway 입장에서는 `/v3/api-docs`라는 요청만 보고는 이것이 어떤 서비스로 가야 하는지 판단할 수 없었고, 결국 라우팅 규칙에 매칭되지 않아 인증 필터에서 차단된 것이다.

<br>

---

## 해결 방향

이 문제를 해결하기 위해 두 가지 구조적 대안을 검토했다.

1. **각 서비스가 UI를 제공하고 Gateway는 라우팅만 담당 (현재 구조)**

    - 장점: Gateway의 책임을 최소화하고 각 서비스가 독립적으로 문서를 관리할 수 있음.
    - 단점: Swagger UI가 원래의 prefix를 인지할 수 있도록 추가 설정이 필요함.

2. **Gateway가 모든 서비스의 문서를 통합 렌더링**

    - 장점: 사용자가 Gateway 주소 하나로 모든 API 명세서를 확인할 수 있어 편의성이 높음.
    - 단점: Gateway에 Swagger 의존성이 추가되며 설정이 다소 복잡해질 수 있음, Gateway에 부하가 몰림.

실제로 활용 사례들을 검색해보니 후자의 구조로 운영하는 경우가 대다수였다. 아무래도 서비스가 늘어날수록 개별 라우팅을 관리하기 번거롭고, 사용자 입장에서도 한곳에서 모든 문서를 보는 것이 편하기 때문일 것이다.

하지만 이번에는 **1번 방식**을 선택했다. 서비스 규모가 크지 않았고, 무엇보다 **프록시 환경에서 Swagger UI가 경로를 재구성하는 원리**를 명확히 이해하고 싶었기 때문이다. 또한 API Gateway는 라우팅 본연의 역할에 충실하게 두는 것이 설계상 더 자연스럽게 느껴지기도 했다.

<br>

---

## X-Forwarded-Prefix 활용

### 1. Gateway 설정: 사라진 정보 알려주기

```yaml
# file: 'application.yml'
- id: user-service-public-docs
  uri: lb://USER-SERVICE
  predicates:
    - Path=/user-service/swagger-ui/**, /user-service/v3/api-docs/**
  filters:
    - AddRequestHeader=X-Forwarded-Prefix, /user-service # 추가한 설정
    - RemoveRequestHeader=Cookie
    - RewritePath=/user-service/(?<segment>.*), /$\{segment}
```

Gateway에서 요청을 전달할 때, 삭제된 prefix 정보를 헤더에 담아 넘겨주도록 `X-Forwarded-Prefix`에 prefix 값을 담는다.

### 2. Microservice 설정: 헤더 해석 전략 결정

전달받은 `X-Forwarded-*` 헤더를 애플리케이션이 어떻게 처리할지 결정해야 한다.  
Spring Boot는 `server.forward-headers-strategy` 설정을 통해 이를 관리한다.

```yaml
# file: 'application.yml'
server:
  forward-headers-strategy: framework
```

#### Strategy 옵션 차이

| 옵션 | 설명 |
| :--- | :--- |
| **None (Default)** | 프록시 관련 헤더를 별도로 처리하지 않는 기본 상태이다. |
| **Native** | WAS(Tomcat, Jetty 등) 자체의 기능을 사용해 헤더를 처리한다. 웹 서버 설정이 추가로 필요할 수 있다. |
| **Framework** | Spring 프레임워크 레벨에서 필터(`ForwardedHeaderFilter`)를 사용해 헤더를 처리한다. |

#### ForwardedHeaderFilter의 역할

`framework` 전략을 선택하면 Spring은 `ForwardedHeaderFilter`를 활성화한다. 이 필터는 공식 문서에 명시된 바와 같이 다음과 같은 역할을 수행한다.

- **요청 정보 수정**: Forwarded 또는 X-Forwarded-* 헤더를 기반으로 요청의 host, port, scheme 및 path 정보를 수정한다.
- **헤더 제거**: 이후 애플리케이션 로직에 영향을 주지 않도록 사용된 프록시 헤더들을 제거한다.
- **요청 래핑(Request Wrapping)**: 기존 요청을 래핑하여 정보를 변경하기 때문에, `RequestContextFilter`처럼 수정된 요청 정보가 필요한 다른 필터들보다 앞선 순서로 실행되어야 한다.

이 설정을 통해 Microservice는 Gateway가 삭제했던 `/user-service` prefix 정보를 인식하게 된다. 결과적으로 Swagger UI가 내부적으로 보내는 `/v3/api-docs` 요청도 Gateway 기준인 `/user-service/v3/api-docs`로 정상적으로 재구성되어 호출될 수 있다.

<br>

---

## 정리

- **Swagger UI**는 내부적으로 추가 API 호출을 수행하므로 프록시 환경에서 경로 유실이 빈번하다.

- Gateway에서 삭제된 경로는 필요할 때 `X-Forwarded-Prefix` 헤더를 통해 서비스에 전달할 수 있다.

- 마이크로서비스 측에서 `forward-headers-strategy: framework` 설정을 통해 프록시 헤더를 인식하게 만들 수 있다.

<br>

결과적으로 Gateway는 라우팅에만 집중하고, 문서화 책임은 각 서비스가 가져가는 구조를 완성할 수 있었다. 다음 포스팅에서는 여러 마이크로서비스의 API 문서를 Gateway 단에서 하나로 묶어 보여주는 통합 Swagger UI 구성 방식도 구현해 보아야 겠다.

<br><br>
<details>
<summary> &nbsp; 📁 참고 자료</summary>
<div>
    <a href="https://springdoc.org/faq.html?#_how_can_i_deploy_springdoc_openapi_starter_webmvc_ui_behind_a_reverse_proxy" target="_blank">Spring Doc FAQ - How can I deploy springdoc-openapi-starter-webmvc-ui behind a reverse proxy?</a>
    </div>
    <div>
    <div>
    <a href="https://docs.spring.io/spring-boot/how-to/webserver.html#howto.webserver.use-behind-a-proxy-server" target="_blank">Spring Boot Doc - Running Behind a Front-end Proxy Server</a>
    </div>
</div>
</details>