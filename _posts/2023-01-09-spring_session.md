---
title: "Spring으로 API 호출하기 (1)"
excerpt: ""

categories:
  - web
tags:
  - ["spring", "session", "api request"]

date_created: 2022-01-0T22:20:00+09:00
last_modified_at: 2023-01-09T20:01:06+09:00
toc: true
toc_sticky: true
---

# Spring으로 세션 생성하고 api 호출하기
최근 새로 맡은 업무로 항상 기술 검토만 진행하다가 코드를 좀 짜봐야겠다 싶어서 보고있는 api문서를 바탕으로 간단한 데모 api를 만들어보고있다. spring은 항상 혼자 공부만하고 강의만 듣다가 사실상 처음 써보는 거라고 해도 무방할 듯 하다. (더 바빠지기 전에 사이드 프로젝트 시작하기로 했었는데..)

어쨌든 그 첫 번째 시작은 basic auth를 통해 api로 session id를 발급받고 해당 session id로 다른 api를 호출하는 것이다. 오늘 그렇게 많은 시간을 투자하지 못했는데, 우선 api로 session id를 발급받는 것 까지 만들었다. 이 과정에서 별거 아니지만 생각하지 못해서 해결하는데 시간이 좀 걸린 부분이 있다.

- Web Client를 사용해 GET, POST request 보내기
- api 호출 시 인증 정보 추가하기
- **SL 사용하지만 인증서가 없기 때문에 인증서 무시하기**

이렇게 적고보니 투자한 시간 대비 너무한게 없어보이는데.. 어쨌든 세 번째 부분을 생각하지 못해서 session id를 받아올 수 없었다.

### Spring WebClient
WebClient는 스프링에서 API를 호출하기 위해 사용하는 HTTP client 모듈이다. 자바에서는 RestTemplate를 사용했다고 한다. 그런데 RestTemplate가 사라지고 WebClient로 대체될 것이라고 한다. 그럼 왜 기존 모듈 대신 WebClient를 사용하느냐? WebClient가 Non-Blocking 방식을 지원하기 때문이다. 동기/비동기, Blocking/Non-Blocking에 대한 이야기는 따로 정리해야 할 것 같고 지금은 그냥 넘어가기로 한다.
그리고 회사 소스에서는 모두 WebClient를 사용하고 있기도 하고 새로 시작하는데 굳이 사라질 RestTemplate을 사용해야 할 필요는 없을 것 같아서 나도 WebClient를 사용하기로 헀다.
그래서 WebFlux Dependency를 추가해주고 사용해야 했다.

### API 호출하기
현재 할 작업은 사실 아직 Non-blocking을 사용할 이유는 없다. 다른 api를 호출하기 위해서는 session ID가 필요하기 때문에 항상 가장 먼저 값을 얻어와야 한다. 따라서 WebClient의 `block()`을 사용하면 기존의 Blocking 방식의 호출을 사용할 수 있다. 그래서 아직은 깊게 공부할 것 없이 blocking 방식으로 호출하는 것 까지 정리해두겠다.

가장 먼저 HTTP Client를 만들어야 한다.
```Java
// WebClientConfig.java

@Configuration
public class WebClientConfig {

    public WebClient getClient() {
        WebClient client = 
    }

    코드 추가
}
```

그리고 호출하는 방법은 매우 쉽다.

```Java
public String createSession() {
    WebClient client = webClientConfig.getClient();

    return client.post()
    .uri("/uri/uri/uri")
    .retrieve()
    .bodyToMono(String.class)
    .block();
}
```

```Java

@PostMapping("/session")
public String session() {

    return webClientService.createSession();

}
```

이렇게 작성한 것만 보면 단순히 외부 API를 통해 세션 ID를 발급 받고 해당 ID를 Response Body로 리턴한 것이다.
이제 다음 스텝으로 해야할 일은 이렇게 발급받은 ID를 다른 API를 호출하는데 사용하는 것이다. 
호출할 때 header에 넣어서 인증하는데 사용하면 될 것이다. 그리고 다른 API를 호출하는 부분을 만들어 직접 호출해 볼 것이다.