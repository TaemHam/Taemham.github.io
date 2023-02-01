---
title: "[백엔드|스프링부트] 서버와 서버사이 요청은 어떻게 주고 받을까?"
author: TaemHam
date: 2023-01-30 09:00:00 +0900
categories: [Backend, SpringBoot]
tags: [Tutorial, Backend, Spring, SpringBoot, Java, RestTemplate]
---
![모놀리식vs마이크로서비스](https://shareditassets.s3.ap-northeast-2.amazonaws.com/production/uploads/upload_file/file/6913/8.%EB%A7%88%EC%9D%B4%ED%81%AC%EB%A1%9C_%EB%AA%A8%EB%86%80%EB%A6%AD_%EB%B9%84%EA%B5%90.png)

## 개요

지금까지 공부해왔던 웹 API는 클라이언트에서 데이터가 필요하면, 서버 하나에서 응답에 필요한 데이터를 지지고 볶아 내어주는 "모놀리식 아키텍처(Monolithic Architecture)" 로 구현해왔다. 하지만 최근에 개발되는 서비스들은 서로 다른 데이터를 처리하는 서버를 여러 개 두고 서버끼리 통신해 데이터를 만들어나가는 "마이크로서비스 아키텍처(Microservice Architecture)"를 주로 채택하고 있다. 그렇다는 것은 클라이언트에서 서버로 요청을 보내는 것 뿐만이 아니라 **서버에서 서버로 요청을 보낼 수 있어야 한다**는 말인데, 이런 웹 요청은 어떻게 자바 코드로 보낼 수 있을까? 스프링부트는 RestTemplate, WebClient를 통해 다른 서버로 웹 요청을 보내고 응답을 받을 수 있게 도와준다.

## Rest Template

### 개념

* RestTemplate은 스프링에서 **HTTP 통신 기능을 손쉽게 사용하도록 설계된 템플릿**이다.
* HTTP 서버와의 통신을 단순화해 RESTful 원칙을 지키는 서비스를 편하게 만들 수 있다. 

### 특징

* 이름처럼 **RESTful 형식을 갖춘 템플릿**
* JSON, XML, 문자열 등 **다양한 형식의 응답 제공**
* 블로킹(blocking) I/O 기반의 동기 방식을 사용
* HTTP 프로토콜의 메서드에 맞는 여러 메서드를 제공
* **현재는 지원 중단(deprecated)된 상태**로 WebClient 방식을 사용하는걸 추천하나, 현업에서는 아직 많이 사용됨

### 동작 원리

![RestTemplate 동작 원리](https://velog.velcdn.com/images%2Fsoosungp33%2Fpost%2F686a5e4e-9f77-4beb-a62c-895724d1bd36%2F%EB%8B%A4%EC%9A%B4%EB%A1%9C%EB%93%9C.png)

위 그림의 왼쪽 파란 사각형이 우리가 작성하는 애플리케이션 코드이고, 오른쪽 끝의 REST API가 다른 서버에 존재하는 API 이다. 여기서 RestTemplate 메서드를 호출하면 그 둘 사이에서 무슨 일이 일어나는지 알아보자.

1. 어플리케이션이 RestTemplate를 생성하고, URI, HTTP 메소드 등의 헤더를 담아 요청

2. RestTemplate는 HttpMessageConverter를 사용하여 requestEntity를 요청 메세지로 변환

3. RestTemplate는 ClientHttpRequestFactory로 부터 ClientHttpRequest를 가져와서 요청을 보냄

4. ClientHttpRequest 는 요청메세지를 만들어 HTTP 프로토콜을 통해 서버와 통신

5. RestTemplate 는 ResponseErrorHandler 로 오류를 확인하고 있다면 처리로직을 실행

6. ResponseErrorHandler 는 오류가 있다면 ClientHttpResponse 에서 응답데이터를 가져와서 처리

7. RestTemplate 는 HttpMessageConverter 를 이용해서 응답메세지를 java object(Class responseType) 로 변환

8. 어플리케이션에 반환

### 메서드 종류

|메서드|HTTP 형태|설명|
|---|---|---|
|getForObject|GET|GET 형식으로 요청한 결과를 객체로 반환|
|getForEntity|GET|GET 형식으로 요청한 결과를 ResponseEntity로 반환|
|postForLocation|POST|POST 형식으로 요청한 결과를 헤더에 저장된 URI로 반환|
|postForObject|POST|POST 형식으로 요청한 결과를 객체로 반환|
|postForEntity|POST|POST 형식으로 요청한 결과를 ResponseEntity로 반환|
|delete|DELETE|DELETE 형식으로 요청|
|put|PUT|PUT 형식으로 요청|
|patchForObject|PATCH|PATCH 형식으로 요청한 결과를 객체로 반환|
|optionsForAllow|OPTION|해당 URI에서 지원하는 HTTP 메서드를 조회|
|exchange|any|HTTP 헤더를 임의로 추가할 수 있고, 어떤 메서드 형식에서도 사용할 수 있음|
|execute|any|요청과 응담에 대한 콜백을 수정|

### 구현

![예제 도식](https://user-images.githubusercontent.com/95671168/216025912-058f5b58-bb6d-4a75-8360-5ed5a51f927d.png)

클라이언트가 서버1에게 요청을 보내면, 서버1은 RestTemplate을 사용해 서버2의 REST API를 호출하는 형식으로 구현해보았다.

* 서버2 컨트롤러

서버2는 API가 호출당하는 서버다. 웹 서비스라면 컨트롤러 이외에도 여러 계층이 존재하겠지만, 간단히 테스트 하기 위해 컨트롤러만 작성했다.

```java
@RestController
@RequestMapping("api/v1/test") // application.properties 를 통해 포트 9090 할당
public class Server2Controller {

    // 아무것도 추가 안된 GET 메서드
    @GetMapping
    public String getSomething() {
        return "Something";
    }

    // PathVariable을 받는 GET 메서드 
    @GetMapping(value = "/{variable}")
    public String doSomethingWithPathVariable(@PathVariable String variable) {
        return variable;
    }

    // RequestParam을 받는 GET 메서드
    @GetMapping("/param")
    public String doSomethingWithParam(@RequestParam String parameter) {
        return parameter;
    }

    // RequestBody를 받는 POST 메서드
    @PostMapping
    public ResponseEntity<Dto> getDto(
            @RequestBody RequestDto requestDto
    ) {
        Dto dto = new Dto(); // name과 email에 대한 getter, setter만 존재하는 DTO
        dto.setName(requestDto.getName());
        dto.setEmail(requestDto.getEmail());
        return ResponseEntity.status(HttpStatus.OK).body(dto);
    }

    // RequestHeader 로 HTTP 헤더를 받는 POST 메서드
    @PostMapping(value = "/header")
    public ResponseEntity<Dto> doSomethingWithHeader(
        @RequestHeader("my-header") String header,
        @RequestBody RequestDto requestDto
    ) {
        return ResponseEntity.status(HttpStatus.OK).body(requestDto);
    }
}
```

* 서버1 컨트롤러

Postman으로 클라이언트 측에서 요청을 보내는 것 처럼 하기 위해 간략히 작성한 컨트롤러이다.

```java
@RestController
@RequestMapping("/rest-template")
@AllArgsConstructor
public class Server1Controller {

    private final Server1Service server1Service;

    @GetMapping
    public String getSomething() {
        return server1Service.getSomething();
    }

    @GetMapping("/path-variable")
    public String doSomethingWithPathVariable() {
        return server1Service.doSomethingWithPathVariable();
    }

    @GetMapping("/parameter")
    public String doSomethingWithParam() {
        return server1Service.doSomethingWithParam();
    }

    @PostMapping
    public ResponseEntity<Dto> postWithBody() {
        return server1Service.postWithBody();
    }

    @PostMapping("/header")
    public ResponseEntity<Dto> postWithHeader() {
        return server1Service.postWithHeader();
    }

}
```

* 서버1 서비스

서비스 계층에서 RestTemplate 을 사용해 서버2의 API를 호출하도록 작성했다. RestTemplate을 생성해 사용하는 방법은 여러가지가 있는데, 그 중 가장 보편적으로 사용하는 것은 UriComponentsBuilder이다. UriComponentsBuilder는 스프링 프레임워크에서 제공하는 클래스로, 여러 파라미터를 연결해서 URI 형식으로 만드는 기능을 수행한다.

```java
@Service
public class Server1Service {

    // GET 형식의 RestTemplate

    // 1. PathVariable 이나 파라미터를 사용하지 않는 호출 방법
    public String getSomething() {
        URI uri = UriComponentsBuilder
                .fromUriString("http://localhost:9090")
                .path("/api/v1/test")
                .encode() // 인코딩 문자셋, 디폴트는 UTF-8
                .build()
                .toUri();

        RestTemplate restTemplate = new RestTemplate();
        ResponseEntity<String> responseEntity = restTemplate.getForEntity(uri, String.class);

        return responseEntity.getBody();
    }

    // 2. PathVariable을 사용한 호출 방법
    public String doSomethingWithPathVariable() {
        URI uri = UriComponentsBuilder
                .fromUriString("http://localhost:9090")
                .path("/api/v1/test/{variable}") // path 에서 중괄호를 사용해 변수명을 입력
                .encode()
                .build()
                .expand("foobar") // expand 에서 순서대로 값 입력 (복수의 값을 넣어야 할 경우 쉼표(,)로 추가)
                .toUri();

        RestTemplate restTemplate = new RestTemplate();
        ResponseEntity<String> responseEntity = restTemplate.getForEntity(uri, String.class);

        return responseEntity.getBody();
    }

    // 3. 파라미터를 사용한 호출 방법
    public String doSomethingWithParameter() {
        URI uri = UriComponentsBuilder
                .fromUriString("http://localhost:9090")
                .path("/api/v1/test/param")
                .queryParam("parameter", "foobaz")
                .encode()
                .build()
                .toUri();

        RestTemplate restTemplate = new RestTemplate();
        ResponseEntity<String> responseEntity = restTemplate.getForEntity(uri, String.class);

        return responseEntity.getBody();
    }

    // POST 형식의 RestTemplate

    // 1. RequestBody를 넣는 호출 방법
    public ResponseEntity<Dto> postWithBody() {

        URI uri = UriComponentsBuilder
                .fromUriString("http://localhost:9090")
                .path("/api/v1/test")
                .encode()
                .build()
                .toUri();

        Dto dto = new Dto();
        dto.setName("name");
        dto.setEmail("email@mail.com");

        RestTemplate restTemplate = new RestTemplate();
        return restTemplate.postForEntity(uri, dto, Dto.class);
    }

    // 1. Header를 넣는 호출 방법
    public ResponseEntity<MemberDto> postWithHeader() {

        URI uri = UriComponentsBuilder
                .fromUriString("http://localhost:9090")
                .path("/api/v1/test/header")
                .encode()
                .build()
                .toUri();

        Dto dto = new Dto();
        dto.setName("name");
        dto.setEmail("email@mail.com");

        RequestEntity<MemberDto> requestEntity = RequestEntity
                .post(uri)
                .header("my-header", "barbaz")
                .body(dto);

        RestTemplate restTemplate = new RestTemplate();
        return restTemplate.exchange(requestEntity, Dto.class);
    }
}
```

## 참고 자료
***

* [스프링 RestTemplate 정리(요청 함)](https://velog.io/@soosungp33/%EC%8A%A4%ED%94%84%EB%A7%81-RestTemplate-%EC%A0%95%EB%A6%AC%EC%9A%94%EC%B2%AD-%ED%95%A8)
* [Spring-boot 공부 8 - RestTemplate(Server to Server 간의 통신)](https://ugo04.tistory.com/93)