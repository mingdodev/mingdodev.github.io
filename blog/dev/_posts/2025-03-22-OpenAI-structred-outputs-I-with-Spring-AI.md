---
layout: post
title:  "[Spring] Spring AI로 OpenAI 구조화된 출력(Structured Outputs) 사용하기"
author: "minseo"
categories: [blog, dev]
tags: [Spring, Logging, NCP]
comments: true
image: "/static/img/logo/spring-ai-logo.png"
hide_image: true
---
# [Spring AI] OpenAI ChatClient로 구조화된 출력(Structured Outputs) 사용하기

<br>
<center><img src="../../../static/img/logo/spring-ai-logo-big.png" width="50%"></center>
<br><br><br>

화면 구성에 직접 관여하는 데이터는 요구사항을 반드시 만족해야 한다. 사용자가 그 결과를 곧바로 확인할 수 있기 때문이다.

프로젝트에서 Clova LLM을 사용할 때, 프롬프트를 통해 출력 형식을 지정해도 요구하는 응답 형식을 벗어나는 경우가 많았다. 문제는 LLM의 응답이 UI 구성에 직접 관여하기 때문에 잘못된 응답이 곧바로 화면에 드러나 사용자에게 혼란을 주었다는 점이다.

하이퍼파라미터를 조정하고 서버 자체 필터링을 거쳐도 전체적인 응답 형태가 다양하니 원하는 데이터를 파싱하기 어려웠다. 따라서 근본적인 문제 해결을 위해, 응답 형식을 강제하는 기술인 **OpenAI의 구조화된 출력(Structured Outputs)**을 프로젝트에 도입하게 되었다.

<br>

> 해당 프로젝트는 `Spring Boot 3.4.3` , `java 17`, `Spring AI M6` 을 사용하였다.

<br>

---

## OpenAI Structured Outputs

- 기존에 지원했던 JSON mode의 진화형이다.

- JSON mode는 모델이 유효한 JSON 형식을 생성하도록 보장하지만, 출력이 원하는 특정 스키마와 일치한다는 보장은 없다.

- **구조화된 출력(Structured Outputs)**는 모델의 응답이 사전에 정의된 JSON 스키마를 엄격히 준수하도록 보장하는 기술이다.

- 공식 문서에서도 JSON mode 대신 구조화된 출력을 사용할 것을 권장한다.

#### 원리

일반적인 모델들이 출력을 생성할 때에는 다음에 출력할 값으로 어떤 토큰이든 선택할 수 있다. 이러한 자유로움으로 인해 모델은 언제든 실수할 수 있게 된다. 구조화된 출력은 이 과정에서 **선택할 수 있는 토큰을 제한**함으로써 제공된 스키마와 완전히 일치하는 출력을 생성하도록 보장한다. 이러한 기술을 **Constrained decoding**이라고 부른다.

예를 들어 모델에게 ‘너 이름이 뭐야?’라고 물었을 때,
일반적인 출력 방식에서는 "제 이름은 고양이입니다.", "고양이예요!", "음... 비밀!"처럼 다양한 자연어로 응답할 수 있다.

하지만 구조화된 출력에서는 모델이 반드시 아래와 같은 JSON 형식으로만 대답하게 제한된다.

```json

    { "name": "고양이" }

```

즉, `{`, `name`, `:`, `고양이` 같은 정해진 토큰만 생성 가능하게 막는 것이다. <span style="color:#737373; font-size:14px; font-weight:300;">자세한 원리가 궁금하다면 [해당 포스팅](https://openai.com/index/introducing-structured-outputs-in-the-api/)과 CFG의 개념을 참고하면 좋을 듯하다.</span>

#### 활용 방법

공식 문서를 보면, REST API 통신에서는 필드를 추가하는 것만으로 구조화된 출력을 요청할 수 있다. API 요청을 보낼 때, `response_format` 필드에 다음과 같이 데이터를 보내기만 하면 된다.  <span style="color:#737373; font-size:14px; font-weight:300;">function calling인 경우는 [이곳](https://platform.openai.com/docs/guides/function-calling#strict-mode)을 참조!</span>

```json

    response_format: { "type": "json_schema", "json_schema": … , "strict": true }

```

- **type:** `json_schema`로 지정한다. (cf. `json_object`로 지정하면 기존의 JSON mode를 사용할 수 있다)

- **json_schema:** 원하는 스키마를 JSON 형식으로 작성한다.

- **strict:** true로 설정하면, 출력 형식을 스키마에 엄격히 맞춰야 하며, 모델이 형식에 맞지 않는 응답을 시도할 경우 refusal 응답이 반환된다.
(이 경우, 요청이 윤리적인 문제를 포함하고 있을 수 있다.)

<br>

---

## OpenAI with Spring

**Java/Spring**을 사용하는 우리 프로젝트에서 구조화된 출력을 가장 간편하게 사용하는 방법이 무엇일까? `RestTemplate, WebClient 등`으로 직접 REST API 요청을 보내는 방법도 있지만 유지보수를 고려해 가장 유연한 방식으로 구성하고 싶었다. 특히 Python, JS 환경이라면 OpenAI SDK 내부에서 스키마와 객체 관계를 손쉽게 다룰 수 있는데, Java 언어에서도 이런 편리함을 가져가고 싶었다.

<br>

- **OpenAI Java API Library**

    - OpenAI에서 공식적으로 제공하는 자바 전용 라이브러리이다.

    - 베타 버전으로만 제공되고 있으며, 다른 라이브러리에 비해 개발된지 오래 되지 않았다. Repository를 살펴보니 구조화된 출력을 올바르게 요청할 수는 있는데, 객체에서 JSON 스키마를 추출하는 부분은 아직 TODO로 남아있다.


- **Simple-OpenAI**

    - OpenAI 공식 문서에도 소개되어 있는 비교적 개발이 많이 진행된 라이브러리이다.

    - [Structured Outputs](https://github.com/sashirestela/simple-openai/blob/ea7259126e630ac2129c53bfdf48c5f0f834226b/src/main/java/io/github/sashirestela/openai/common/ResponseFormat.java#L24)를 올바르게 요청하는 모듈이 구현되어 있다.

    - [구조화된 출력 예제 코드](https://github.com/sashirestela/simple-openai?tab=readme-ov-file#chat-completion-with-structured-outputs)를 통해 어떤 프로세스로 요청과 응답이 이뤄지는지 확인할 수 있다.

<br>

여기까지는 단순히 자바 프로그래밍을 위한 라이브러리이다. `Simple-OpenAI`의 경우 충분히 선택할만 했지만, 코드 구성 자체가 유지보수하기 좋다고 느껴지지는 않았다. <span style="color:#737373; font-size:14px; font-weight:300;">사용하는 객체들이 완전히 추상화되지 않아 기존에 사용하던 WebClient 호출의 단점이 여전히 남아있는 느낌이다</span> 그렇게 내 요구사항이 스프링 프레임워크의 추구미와 일치한다는 것을 깨닫고.. 작년 Google I/O Extended에서 접했던 **Spring AI**를 떠올리게 되었다.

<br>

- **Spring AI**

    - LLM을 이용해 애플리케이션 생성을 단순화하도록 설계된 Spring 프레임워크이다.

    - OpenAI, Microsoft, Amazon 등 메이저한 벤더사의 LLM 모델을 대부분 지원한다.

    - 주로 Python 환경 중심으로 발전한 생성형 AI를 Java 개발자도 손쉽게 활용할 수 있도록 하기 위해 탄생했다.

    - 아직 정식 릴리즈를 앞둔 기술이라 내부 구현이 변경될 수 있다.

<br>

스프링 AI는 추후 다른 LLM 모델로 전환이 일어나더라도 변동이 쉽고, OpenAI에 특화된 기술이 아니지만 [Structured Outputs를 완벽 지원한다는 공식 블로그](https://spring.io/blog/2024/08/09/spring-ai-embraces-openais-structured-outputs-enhancing-json-response) 게시글이 나와있었다. 개발이 실시간으로 이루어지고 있어 잠깐 고민했으나 오히려 에러가 발생하더라도 즉각 패치가 가능할테고, 애초에 Spring AI는 장기적으로 개발 및 유지보수가 이어질 프레임워크라고 판단했다. 따라서, 프로젝트의 확장성과 안정성을 모두 고려했을 때 **Spring AI**가 가장 적합하다고 생각해 사용할 기술로 채택하게 되었다.

<br>

---

## Spring AI with OpenAI Structured Outputs

이제 본격적으로 프로젝트에서 구조화된 출력을 활용한 방법에 대해 소개해보겠다.

#### Spring AI Dependency 설정

가장 먼저 의존성을 추가해준다. 여기서 프로젝트의 특성을 고려하여 결정할 부분이 있다.

- 우리 프로젝트가 **Spring AI를 활용하여 여러 개의 LLM 모델을 활용**할 것인가?

    - Spring AI로 여러 개의 모델을 사용할 것이라면 Spring AI BOM(Bill of Materials) 의존성을 추가해 의존성 관리를 맡기자. 나는 Spring AI가 지원하지 않는 Clova LLM 외에는 활용 계획이 아직 없어 사용하지 않았다.

- 아직 정식 릴리즈 전이므로 **어떤 버전**을 사용할지 결정해야 한다.

    - 나는 실사용에는 무리가 없는 가장 최신의 마일스톤 버전 M6를 선택했다.

    - M6의 경우 Maven Central에 릴리즈되어 있어 아래와 같이 의존성을 바로 추가해주면 된다. 그러나 이전 마일스톤 버전이나 스냅샷 버전을 사용하려면 빌드 파일 `repositories` 부분에 스냅샷 저장소 url을 추가해야 사용할 수 있다.

```gradle

    implementation 'org.springframework.ai:spring-ai-openai-spring-boot-starter:1.0.0-M6'

```

#### Auto Configuration

```yml
    # file: "application.yml"
    spring:
      ai:
        openai:
          api-key: ${OPENAI_API_KEY}
        chat:
          options:
          model: gpt-4o-mini

```

- Spring AI에서도 Spring Boot 자동 구성 기능을 활용할 수 있다.

- 세부적인 옵션은 추후 커스텀하기 위해 공통적으로 사용되는 API 키, 모델만 자동 구성으로 설정해주었다.


#### OpenAiConfig

```java

    @Configuration
    public class OpenAiConfig {

        @Bean
        ChatClient chatClient(ChatClient.Builder builder) {
            return builder.build();
        }
    }

```

**ChatClient**는 Spring AI에서 OpenAI와 대화형 요청을 주고받을 수 있도록 제공하는 주요 컴포넌트이다.
이후 서비스 로직 등에서 주입받아 자유롭게 사용하기 위해, ChatClient.Builder를 사용해 ChatClient를 생성하고 Spring 빈으로 등록하는 설정 파일을 만들었다.


#### OpenAiClient

OpenAI를 도입하면서 이전에 구상했던 API 추상화 구조도 함께 적용해보았다. LLM 요청 로직이 변경될 때에 대비해 AiClient라는 인터페이스를 정의하고, 서비스 계층은 이 인터페이스에만 의존하도록 코드를 리팩토링하였다. 실제 API 요청 로직은 OpenAiClient와 같은 구현체에 위임되어, 구현체를 바꾸거나 로직을 수정할 때도 비즈니스 로직은 그대로 유지할 수 있다.

```java

    public interface AiClient {

        AiCorrectResponse correct(String content);

        AiRecommendTagsResponse recommendTags(String content, String originTags);

        AiSearchTypeResponse checkSearchType(String question);

        AiKeywordsResponse extractKeywords(String question, String originTags);
    }

```

- 기존에는 해당 메서드들도 여러 클라이언트에 분리되어 있었으나 `AI API 호출`이라는 명확한 역할의 인터페이스로 묶어버렸다.

```java

    @Component
    @RequiredArgsConstructor
    public class OpenAiClient implements AiClient {

        private final ChatClient chatClient;

        @Override
        public AiCorrectResponse correct(String content) {

            String prompt = """system prompt ...""";

            return callWithStructuredOutput(content, prompt, AiCorrectResponse.class);
        }
    ...
    }

```

- 구현체의 각 메서드에서 `callWithStructuredOutput`를 통해 OpenAi API를 호출한다.

- 사용자 입력, 시스템 프롬프트와 응답 스키마만 변경되고 호출 방식은 동일하기에 `callWithStructuredOutput`라는 메서드로 추출하여 구현했다.


#### callWithStructuredOutput

- 실제로 구조화된 출력을 사용하는 부분은 여기다.

```java

    private <T> T callWithStructuredOutput(String user, String system, Class<T> clazz) {

        BeanOutputConverter<T> outputConverter = new BeanOutputConverter<>(clazz);
        String jsonSchema = outputConverter.getJsonSchema();

        Prompt prompt = new Prompt(user,
                OpenAiChatOptions.builder()
                        .responseFormat(new ResponseFormat(Type.JSON_SCHEMA, jsonSchema))
                        .build());

        ChatResponse chatResponse = this.chatClient.prompt(prompt)
                .system(system)
                .call()
                .chatResponse();

        return outputConverter.convert(extractSafeContent(chatResponse));
    }

```

- 구조화된 응답을 다양한 타입으로 받을 수 있도록, 제네릭 메서드를 만들어 재사용 가능한 API 호출 로직을 구성했다.
이 메서드는 사용자 입력, 시스템 프롬프트, 그리고 응답 타입(Class<T>)을 받아서 OpenAI로 요청을 보내고, 스키마에 맞는 응답을 자바 객체로 변환해 반환한다.

<br>

- `extractSafeContent`를 따로 분리하여, 모델로부터 받아온 응답값에서 예외 처리를 진행할 수 있도록 만들었다.

```java

    private String extractSafeContent(ChatResponse response) {

        if (response == null) {
            throw new RuntimeException(ErrorMessage.OPEN_AI_SERVER_ERROR.toString());
        }

        AssistantMessage message = response.getResult().getOutput();
        Map<String, Object> meta = message.getMetadata();

        if (!Objects.toString(meta.get("refusal"), "").isBlank()) {
            throw new RuntimeException(ErrorMessage.OPEN_AI_REFUSAL.toString());
        }

        return message.getText();
    }

```

- 응답이 비어있을 때, 그리고 앞서 언급했던 `refusal`을 반환할 때 클라이언트에게 이를 알릴 수 있도록 예외를 반환하는 로직을 구성해두었다.

- Docs에는 필드 자체의 존재 여부에 따라 구분할 수 있다고 했는데, 테스트 해 보니 `refusal` 필드는 정상 응답에서도 존재하지만 비어있는 형태로 나타났다. 따라서 빈 값이 아닌 경우를 거절로 간주하고 에러 메시지를 반환하도록 구현했다.

<br>

이렇게 구현을 완료하면, 더이상 모델이 엉뚱한 답변 또는 임의의 데이터를 응답값에 포함하지 않게 된다!