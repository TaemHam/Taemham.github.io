---
title: "[백엔드|스프링부트] 스프링 WEB 관련 모듈"
author: TaemHam
date: 2022-09-25 21:10:00 +0900
categories: [Backend, SpringBoot]
tags: [Tutorial, Backend, Spring, SpringBoot, Java]
---

<img src="https://user-images.githubusercontent.com/95671168/191079807-7a4c5591-a501-49d5-a80e-903a80f50933.png"/><em>스프링 프레임워크의 모듈[^1]</em>

이전 포스트에서도 언급했듯이, 스프링 프레임워크는 기능별로 구분된 약 20여 개의 모듈을 제공한다. 이 중에서 웹과 관련된 모듈들은 다음과 같다:

* `spring-web`
* `spring-webmvc`
* `spring-websocket`
* `spring-webmvc-portlet` 

오늘은 이 네가지 모듈들이 각각 무엇인지, 왜 사용하는지에 대해 적어보려 한다.

## 웹 모듈 `spring-web`
***

스프링 프레임워크 공식문서[^1]에 따르면, `spring-web`은 1. 다른 웹 프레임워크와의 통합, 2. 기본적인 웹 지향 통합 기능을 제공한다고 한다. 웹 지향 통합 기능의 예로는, 단일/다중 파일 업로드 기능을 담당하는 의존성 `commons-fileupload`와 빈 `MultipartResolver`가 있고, 어플리케이션에 웹 지향 어플리케이션 컨텍스트를 연동시키는 기능의 의존성인 `AnnotationConfigApplicationContext` 등이 있다.

어플리케이션 컨텍스트(Application context)란, 빈의 생성과 관계설정 기능이 있는 빈 팩토리(Bean Factory)를 상속 받아 확장한 것으로, 빈이 만들어지는 방식과 시점 및 전략 등을 다르게 하거나 후처리나 정보의 조합, 인터셉트 등과 같은 다양한 기능을 이용하고 싶을 때 사용한다고 한다.[^2] 공식 문서에서는 이러한 추가 기능 때문에 개발자는 빈 팩토리보다 어플리케이션 컨텍스트를 사용하는 것을 선호한다고 한다.

## 서블릿 모듈 `spring-webmvc`
***

<img src="https://user-images.githubusercontent.com/95671168/192155742-0e11e914-65c1-49b6-bcb7-00fe778915d4.png"/><em>스프링 프레임워크의 서블릿 관계도[^3]</em>

`spring-webmvc` 는 자체 MVC 프레임워크를 제공함으로써 서블릿 기반의 웹 어플리케이션을 보다 쉽게 만들기 위한 스프링 프레임워크 모듈이다. `@WebServlet` 어노테이션으로 해당 클래스가 어느 요청을 처리할 것인지 선언하고, 반환값은 HTTP 규격에 맞춘 Response로 바꿔주는 `@ResponseBody`로 감싸 반환 해주면 된다. [^4]

서블릿은 클라이언트 요청에 따라 동적으로 컨텐츠를 만들 수 있는 기능을 위해 만들어지는 클래스 파일이다. 다른 동적 컨텐츠를 만드는 기술로는 CGI가 있는데, 이는 멀티 프로세싱 방법을 사용하기 때문에 멀티 스레딩으로 요청을 처리하는 서블릿보다 서버 자원을 더 많이 차지한다.[^5]


## 웹소켓 모듈 `spring-websocket` 
***

이름에서 알 수 있듯이, 웹 소켓 통신 기능을 편리하게 도입할 수 있도록 해주는 모듈이다. 바닐라 자바로 웹 소켓을 구현하려면 Java Socket으로 소켓 통신의 과정을 일일이 구현해야 했다. 하지만 스프링프레임워크에서는 다른 모듈처럼 의존성을 주입시키고 핸들러만 구현해 등록시키면 다른 과정은 알아서 처리된다.[^6]

웹소켓을 사용하는 이유를 알기 위해서는 기존의 방식인 HTTP가 어떤 특징을 가지는지 알아야한다.

<img src="https://user-images.githubusercontent.com/95671168/192146765-03d1bf70-3c96-45b7-a03a-1aa93ad5b533.png"/><em>HTTP 와 웹소켓의 차이[^7]</em>

HTTP의 특징:[^8]
* 단방향성 → 요청이 있어야만 무언가를 보낼 수 있음
* 비연결성 → 무언가를 보내고 나면 연결이 끊어짐

HTTP는 tcp 기반으로 만들어진 통신 프로토콜이므로 3-way handshaking으로 연결하고 4-way handshaking으로 끊는데, 연결이 유지되지 않기 때문에 매 번 맺고 끊는 과정의 비용이 발생한다. 또, 메시지를 보낼 때마다 헤더가 포함되어야 하므로, 적은 데이터를 실시간으로 계속해서 보내야 하는 경우 (채팅 등) 부담이 커진다. 이런 문제점들을 해결하기 위해 나온 것이 웹소켓 프로토콜이다. 

따라서 웹소켓 프로토콜은 실시간 통신을 필요로 하는 서비스(채팅, 게임, 주식 거래 등)에 주로 사용된다.

## 포틀릿 모듈 `spring-webmvc-portlet`
***

<img src="https://user-images.githubusercontent.com/95671168/192189472-1961118e-2da7-4482-a47c-4129c0b758eb.png"/><em>포틀릿의 모습[^9]</em>

서블릿과 마찬가지로 포틀릿 환경에서 사용할 MVC 구현을 제공해주는 모듈이다. 포틀릿 모듈이 가진 특징을 설명하기 이전에, 포틀릿이 무엇인지 알아야할 것 같다. 포틀릿은 포털을 구성하는 요소로, 웹 기반 컨텐츠나 어플리케이션 등에 접근할 수 있도록 해주는 재사용 가능한 웹 모듈이다. 기능만으로 봤을 때는 삼성 갤럭시 핸드폰에서 제공하는 위젯 기능과 비슷하다. 지메일 위젯을 예로 들면, 위젯을 바둑판 배열 가운데 원하는 위치에 원하는 크기로 배치할 수 있으며, 로그인 정보에 따라 각기 다른 메일함을 보여주는 것이다. 이렇게 개인화(Personalization)가 가능하다는 것이 포털 및 포틀릿의 가장 큰 특징이다. [^10]

포틀릿과 서블릿은 정말 많은 부분에서 유사하다. 둘 모두 컨테이너에 배치되어 동적/정적 컨텐츠를 생성하는 웹 구성 요소이고, 클라이언트의 요청에 의해서 응답이 생성되는 방식으로 작동한다. 그렇지만 포틀릿은 html의 모든 것을 생성하지 못하고 마크업 조각만을 생성해 삽입될 수 있으며, 포틀릿에게는 URL이 주어지지 않아 포틀릿의 컨텐츠를 사용하고 싶으면 포틀릿을 포함한 URL을 호출해야 한다. [^11]

## 출처
***

[^1]: <https://docs.spring.io/spring-framework/docs/4.0.x/spring-framework-reference/html/overview.html#overview-modules>
[^2]: <https://mangkyu.tistory.com/151>
[^3]: <https://docs.spring.io/spring-framework/docs/current/reference/html/web.html>
[^4]: <https://jwdeveloper.tistory.com/105>
[^5]: <https://joonyk.tistory.com/16>
[^6]: <https://dev-gorany.tistory.com/3>
[^7]: <https://blog.scaleway.com/iot-hub-what-use-case-for-websockets/>
[^8]: <https://velog.io/@rhdmstj17/%EC%86%8C%EC%BC%93%EA%B3%BC-%EC%9B%B9%EC%86%8C%EC%BC%93-%ED%95%9C-%EB%B2%88%EC%97%90-%EC%A0%95%EB%A6%AC-2>
[^9]: <https://docs.webix.com/desktop__portlet.html>
[^10]: <https://blog.outsider.ne.kr/934>
[^11]: <https://blog.daum.net/tomayoon/7095404>