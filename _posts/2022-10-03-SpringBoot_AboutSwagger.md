---
title: "[백엔드|스프링부트] 스프링부트에 Swagger 적용하기"
author: TaemHam
date: 2022-10-03 11:50:00 +0900
categories: [Backend, SpringBoot]
tags: [Tutorial, Backend, Spring, SpringBoot, Java, Swagger]
---

<img src="https://user-images.githubusercontent.com/95671168/193753621-9d46dd66-2cca-428d-b39f-a1d0e2bfa472.png"/><em>Swagger 로고[^1]</em>

API는 간단하게 설명하자면 함수와도 비슷한 것이다. 특정 데이터를 넣으면, 함수가 처리한 값을 되돌려주는 것이다. 하지만 이런 함수는 그냥 무턱대고 쓸 수는 없다. 숫자를 받아 +1한 값을 반환해주는 함수에 a를 넣을 수는 없다. 이렇게 함수를 사용하려면, 이 함수가 무슨 값을 받는지, 이 값을 처리하면 무엇이 반환 되는지 등, 이 함수를 이용할 사용자에게 여러 정보를 알려주어야 한다. 

API에 사용 방법에 대한 이런 정보들을 문서로 정리하거나 공통의 기준을 정한 것을 API 명세라고 한다. API 명세는 직접 스프레드시트 등으로 직접 작성해 공유하거나, 혹은 작성을 도와주는 툴을 사용해 공유할 수도 있다. 만약 툴을 사용하지 않고 직접 작성하는 경우, API의 추가나 수정이 있을 때마다 명세를 수정해 배포해야하는 번거로움이 있다. 이런 번거로움을 해결하기 위해, 스프링부트에서는 Swagger 라는 API 명세 문서화 툴을 제공한다. 개발자는 간단한 설정만 해주면 API 명세 문서를 자동으로 만들어 배포해주는 것이다.

## 적용 방법
***

프로젝트에 Swagger를 적용하려면, 다음과 같은 3가지만 처리해주면 된다.

1. 의존성 주입
2. 문서 설정
3. 어노테이션 추가(선택사항)

### 예제 환경

* Spring boot 2.5.6
* Maven
* Swagger ui 2.9.2
* Swagger2 2.9.2

### 의존성 주입

먼저 Swagger 라이브러리에 대한 의존성 주입을 해준다. `pom.xml` 파일의 `<dependencies>` 태그 밑에 아래와 같은 태그들을 추가해준다. 

```xml
  <dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
  </dependency>
  <dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
  </dependency>
```

특이사항으로 다른 라이브러리들과 다르게 `<version>` 태그를 기재해주는데, `spring-boot-starter-parent`에 기본적으로 명세되어있는 라이브러리가 아니기 때문이다.


### 문서 설정

다음으로 Swagger와 관련된 설정 코드를 작성해준다. API 명세를 노출시킬 HTML 문서를 작성하는 과정이다. 기존 프로젝트의 패키지 안에 다음과 같이 작성된 클래스를 추가해준다.

```java

package com.springboot.api.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Configuration
@EnableSwagger2

public class SwaggerConfiguration {

    @Bean
    public Docket api() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.springboot.api"))
                .paths(PathSelectors.any())
                .build();
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("Spring Boot Open API Test with Swagger")
                .description("설명 부분")
                .version("1.0.0")
                .build();
    }
}

```

다음은 설정 코드에 삽입되는 코드들에 관한 설명이다.[^2]

* `@Configuration`: 설정 파일임을 선언.
* `@EnableSwagger2`: Swagger 활성화.
* `Docket`: HTML문서의 템플릿을 제공하는 플러그인. Docket 클래스에 `DocumentationType.SWAGGER_2` 심볼을 넣고 다음과 같은 메서드를 실행시키면 된다.
  * `.apiInfo()`: API 에 대한 기본적인 설명을 추가하는 메서드. 매개변수로 ApiInfo 타입의 객체를 받는다.
  * `.select()`: ApiSelectorBuilder를 생성하는 메서드.
  * `.apis()`: `@RestController`가 선언된 패키지안에서 `@RequestMapping`이 선언된 API들을 지정하는 메서드. 
  * `.paths()`: 위에서 선택된 API중 path조건에 맞는 것들만 걸러내는 메서드
  * `.useDefaultResponseMessages()`: 불필요한 응답코드와 메시지를 제거하는 메서드. 매개변수로 false를 넣어 제거할 수 있다.
  * `.groupName()`: Docket을 여러개 사용 할 경우 groupName이 충돌하지 않도록 지정하는 메서드.
* `ApiInfo`: API의 설명을 Docket에 추가하기 위한 객체. 매개변수로 넣을 때 `ApiInfo` 클래스를 호출해 한번에 넣을 수도 있지만, `ApiInfoBuilder`의 메서드로 하나씩 추가할 수 있다.
  * `.title()`: API의 이름을 넣는 메서드.
  * `.description()`: 설명을 넣는 메서드.
  * `.version()` : API의 버전을 넣는 메서드.
  
### 어노테이션 추가

마지막으로 각 API에 대한 설명을 넣어주면 된다. 기존 코드에 어노테이션만 추가해주면 된다.

```java

    @ApiOperation(value= "GET 메서드 예제", notes = "@RequestParam을 활용한 GET Method")
    @GetMapping(value = "/request1")
    public String getRequestParam1(
            @ApiParam(value = "이름", required = true) @RequestParam String name,
            @ApiParam(value = "이메일", required = true) @RequestParam String email,
            @ApiParam(value = "회사", required = true) @RequestParam String organization) {
        return name + " " + email + " " + organization;
    }

```

* `@ApiOperation()`: 매핑 어노테이션과 함께 쓰여, 해당 API의 설명을 추가한다.
  * `value=`: API의 이름을 추가한다. URI 옆에 나타나며, 기본값은 메서드명이다.
  * `notes=`: API의 설명을 추가한다. 클릭해 확장시키면 나타난다. 
* `@ApiParam()`: 매개변수에 대한 설명 및 설정을 추가한다. 
  * `value=`: 매개변수의 설명을 추가한다.
  * `required=`: 해당 매개변수가 필수인지 표기한다.

이 외에도 [여러가지 어노테이션](https://velog.io/@gillog/Swagger-UI-Annotation-%EC%84%A4%EB%AA%85)으로 설명을 추가할 수 있다 [^3]

## 출처
***

[^1]: <https://swagger.io/>
[^2]: <https://otrodevym.tistory.com/entry/spring-boot-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0-4-Swagger-%EC%84%A4%EC%A0%95-%EB%B0%8F-%EC%82%AC%EC%9A%A9-%EB%B0%A9%EB%B2%95>
[^3]: <https://velog.io/@gillog/Swagger-UI-Annotation-%EC%84%A4%EB%AA%85>
