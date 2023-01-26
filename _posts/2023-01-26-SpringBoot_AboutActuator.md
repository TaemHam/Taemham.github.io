---
title: "[백엔드|스프링부트] 운영중인 웹 서비스를 Actuator로 모니터링 해보자"
author: TaemHam
date: 2023-01-26 09:00:00 +0900
categories: [Backend, SpringBoot]
tags: [Tutorial, Backend, Spring, SpringBoot, Java, Actuator]
---

## 개요

애플리케이션을 운영하는 단계에 접어들면, 애플리케이션이 정상적으로 동작하는지 모니터링하는 환경을 만들어야 한다. 이 때 사용할 수 있는 기능이 스프링부트 액추에이터로, 애플리케이션의 건강 상태 정보, 사용 중인 메모리, 특정 엔드포인트가 받은 요청 횟수 등의 정보를 HTTP 요청을 통해 받아 볼 수 있다. 이 기능을 어떻게 추가하며, 어떻게 사용할 수 있는지 알아보자.

### 의존성 추가

액추에이터를 사용하려면 먼저 의존성을 추가해야 한다. 스프링부트 액추에이터 역시 `spring-boot-starter-parent` 에 포함되어 있으므로, 별도의 버전 관리 없이 쉽게 추가 할 수 있다.

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

### 액추에이터 엔드포인트

위의 의존성을 추가했다면, 웹 서비스의 URL에 `/actuator` 를 포함한 여러 엔드포인트들로 요청을 보내 애플리케이션의 내부 상황을 모니터링 할 수 있다.

자주 활용되는 액추에이터의 엔드포인트는 다음과 같다.

|HTTP 메서드|엔드포인트|설명|기본 활성화 여부|
|---|---|---|---|
|GET|/auditevents|호출된 Audit 이벤트 정보를 표시한다. (`AuditEventRepository` 빈이 필요)|X|
|GET|/beans|애플리케이션에 있는 모든 스프링 빈 리스트를 표시한다.|X|
|GET|/caches|사용 가능한 캐시를 표시한다.|X|
|GET|/conditions|자동 구성 조건 내역을 생성한다.|X|
|GET|/configprops|`@ConfigurationProperties`의 속성 리스트를 표시한다.|X|
|GET,POST,DELETE|/env|애플리케이션에서 사용할 수 있는 환경 속성을 표시, 수정, 삭제한다.|X|
|GET|/health|애플리케이션의 상태 정보를 표시한다.|O|
|GET|/httptrace|가장 최근에 이뤄진 100건의 요청 기록을 표시한다.|X|
|GET|/info|애플리케이션의 정보를 표시한다|O|
|GET|/integrationgraph|스프링 통합 그래프를 표시한다. (`spring-integration-core` 모듈에 대한 의존성을 추가해야 동작)|X|
|GET,POST|/loggers|애플리케이션의 로거 구성을 표시하고 수정한다.|X|
|GET|/mappings|모든 `@RequestMapping`의 매핑 정보를 표시한다.|X|
|GET|/metrics|애플리케이션의 메트릭 정보를 표시한다.|X|
|GET|/quartz|Quartz 스케줄러 작업에 대한 정보를 표시한다.|X|
|GET|/scheduledtasks|애플리케이션에서 예약된 작업을 표시한다.|X|
|GET,DELETE|/sessions|스프링 세션 저장소에서 사용자의 세션을 검색하고 삭제할 수 있다.(스프링 세션을 사용하는 서블릿 기반 웹 애플리케이션 필요)|X|
|POST|/shutdown|애플리케이션을 정상적으로 종료할 수 있다.|X|
|GET|/startup|애플리케이션이 시작될 때 수집된 시작 단계 데이터를 표시한다.(BufferingApplicationStartup으로 구성된 스프링 애플리케이션 필요)|X|
|GET|/threaddump|스레드 덤프를 수행한다.|X|

#### 엔드포인트의 활성화와 비활성화

위의 표를 보면, 기본적으로 활성화 되어 있는 엔드포인트는 /health와 /info 밖에 없다는 것을 알 수 있다. **대부분의 액추에이터 엔드포인트는 민감한 정보를 제공**하므로 보안 처리가 되어야 하기 때문에, 기본적으로 활성화 되어 있지 않다. 이 부분은 스프링 시큐리티를 사용해 보안을 강화할 수 있다. 그러나 액추에이터 자체는 보안 처리가 되어 있지 않기 때문에 활성화에 주의가 필요하다.

엔드포인트 노출 관리는 `.properties`혹은 `.yaml`에서 할 수 있고, 노출 여부는 `management.endpoints.web.exposure.include`와 `management.endpoints.web.exposure.exclude` 구성 속성을 사용해 제어한다.

```properties
management.endpoints.web.exposure.include=health, info, beans, conditions
management.endpoints.web.exposure.exclude=heapdump, threaddump
```

### 액추에이터 기능

(추후 작성 예정)

### 액추에이터 보안 문제

애플리케이션 모니터링 및 관리 측면에서 개발자에게 편의를 주는 기능이나, 잘못 사용할 경우 비밀번호, API KEY, Token 등 Credential들이나 내부 서비스 도메인, IP 주소와 같은 중요 정보들이 유출될 수 있다. 자세한 정보는 [여기](https://techblog.woowahan.com/9232/)서 확인하자.

## 참고 자료

***
* [스프링 부트 액추에이터 사용하기](https://incheol-jung.gitbook.io/docs/study/srping-in-action-5th/chap-16.)
* [Actuator 안전하게 사용하기](https://techblog.woowahan.com/9232/)