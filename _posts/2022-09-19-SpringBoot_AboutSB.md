---
title: "[백엔드|스프링부트] 스프링부트란?"
author: TaemHam
date: 2022-09-19 22:20:00 +0900
categories: [Backend, SpringBoot]
tags: [Tutorial, Backend, Spring, SpringBoot, Java]
---

백엔드 개발자로 웹 개발을 할 때 알야야 할, 특히 한국에서라면 거의 필수가 되는 기술 스택인 **SpringBoot**. 앞으로 몇 주간은 이에 대해 배우며, 공부한 내용을 포스팅 할 계획이다. 첫 포스트인만큼, 먼저 SpringBoot가 무엇인지 알아보자.

## 스프링 프레임워크
***

<img src="https://user-images.githubusercontent.com/95671168/191002952-9bf8da3d-efed-4f68-bce8-3557bfe5deb8.png" width="50%" height="50%"/><em>스프링 로고[^1]</em>

스프링부트는 스프링을 더욱 쉽게 사용할 수 있도록 하기 위해 나온 것이므로, 먼저 스프링에 대해 설명하겠다.

스프링 프레임워크(Spring Framework), 줄여서 스프링 이라고 부르는 이 프레임워크는 자바로 애플리케이션을 개발하는 데 필요한 것들을 제공하고, 쉽게 사용할 수 있도록 도와주는 도구이다. 스프링은 특히 백엔드 개발에 많이 사용되며, 자바스크립트에서의 Node.js와 위치가 같다고 생각하면 편하다. 

스프링은 바닐라 자바와 비교해서 다음과 같은 특징을 가지고 있다.

### 제어 역전 (IoC: Inversion of Control)

<img src="https://user-images.githubusercontent.com/95671168/191036021-e1b16475-63f0-4d66-84f6-1ab22f2c1ef4.png"/><em>스프링의 제어 역전</em>

바닐라 자바로 개발할 때, 객체를 사용하고 싶으면 일반적으로는 개발자의 제어 하에 객체를 만들고 호출하고 없앤다 (객체의 생명주기라고도 함). 하지만 스프링의 경우 외부(스프링 컨테이너)에서 객체를 생성하고, 필요할 때 객체를 불러와 사용하는 방식으로 객체를 관리한다. 이처럼 객체의 제어를 외부에 위임하는 방식을 제어 역전이라고 하고, 스프링은 이 개념이 적용된 프레임워크이다. 제어 역전은 코드의 재사용성과 유지보수성을 높인다는 장점이 있다.

### 의존성 주입 (DI: Dependency Injection)

한 클래스에서 다른 클래스의 객체를 사용할 때, 그 클래스 자체가 아닌 프레임 워크로부터 객체를 받는 방식의 디자인 패턴으로, 위의 제어 역전에서의 '객체를 불러온다'는 부분을 담당하는 제어 역전의 하위 개념이다. 

스프링에서는 `@Autowired` 라는 어노테이션을 통해 의존성을 주입할 수 있고, 의존성을 주입 받는 방법은 다음과 같은 세 가지가 있다:

1. 생성자를 통한 의존성 주입

    ```java
    @RestController
    public class DIController {

        MyService myService;

        @Autowired
        public DIController(MyService myService) {
            this.myService = myService;
        }

        @GetMapping("/di/hello")
        public String getHello() {
            return myService.getHello();
        }

    }
    ```

2. 필드 객체 선언을 통한 의존성 주입

    ```java
    @RestController
    public class FieldInjectionController {

        @Autowired
        private MyService myService;

    }
    ```

3. setter 메서드를 통한 의존성 주입

    ```java
    @RestController
    public class SetterInjectionController {

        MyService myService;

        @Autowired
        public void setMyService(MyService myService) {
            this.myService = myService;
        }

    }
    ```

스프링 공식 문서에서 권장하는 방법은 `1. 생성자를 통해 주입받는 방식`이다. 해당 방식만이 레퍼런스 객체 없이는 객체를 초기화 할 수 없게 설계할 수 있기 때문이다.

### 관점 지향 프로그래밍 (AOP: Aspect-Oriented Programming)

<img src="https://user-images.githubusercontent.com/95671168/191070953-8c8ea8a2-f8ba-4112-853f-04d8095f4ea0.png"/><em>AOP 방식의 애플리케이션 로직</em>

객체 지향 프로그래밍 (Object-Oriented Programming)의 심화 개념으로, OOP로 각 핵심 기능들을 모듈화 해 분리시켜 놓았다면, AOP는 그 분리시킨 모듈들이 가지고 있는 핵심 기능 이외의 공통적인 부분들을 다시 분리 시켜 핵심 기능을 수행할 때마다 해당 부분들을 불러오는 개발 방식을 의미한다. 스프링은 기능 호출 할 때마다 전처리, 후처리를 포함하는 디자인 패턴인 프락시 패턴을 이용해 AOP 기능을 제공하고 있다.  

### 스프링 프레임워크의 다양한 모듈
<img src="https://user-images.githubusercontent.com/95671168/191079807-7a4c5591-a501-49d5-a80e-903a80f50933.png"/><em>스프링 프레임워크의 모듈[^2]</em>

스프링 프레임워크는 기능별로 구분된 약 20여 개의 모듈로 구성돼 있다. 구체적인 모듈들은 버전마다 조금씩 달라질 때도 있지만, 큰 틀은 유사하다. 애플리케이션을 개발 할 때 위의 모듈들을 무조건 포함해서 사용하는 것이 아닌, 필요한 모듈만 선택해서 사용할 수 있게끔 설계 되어 있어, 이를 '경량 컨테이너 설계'라고 부른다.

## 스프링 프레임워크 vs. 스프링 부트
***
<img src="https://user-images.githubusercontent.com/95671168/191237763-3ed9a1cc-cc21-42dc-acbb-fc54b75cbda2.png" width="50%" height="50%"/><em>스프링부트 로고[^3]</em>
스프링은 이렇게 기존 개발 방식의 문제와 한계를 극복하기 위해 다양한 기능을 제공하는데, 오히려 기능이 너무 많아 설정이 복잡해졌다는 문제점이 생겨버렸다. [이 블로그](https://mirotic91.tistory.com/4)[^4]를 보면, 웹 애플리케이션 개발을 위해 설정해야 하는 사항들이 스프링에서는 정말 복잡하게 나열되어 있는데 반해, 스프링 부트는 설정할 파일도 적고, 파일의 내용 또한 많이 축소 되어 있다. 이는 스프링부트가 스프링에 비해 다음과 같은 방면에서 뛰어나기 때문이다.

### 의존성 관리

<details>
<summary>pom.xml 예시</summary>
<div markdown="1">

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.5.6</version>
    <relativePath/> <!-- lookup parent from repository -->
  </parent>

  <groupId>com.wikibooks</groupId>
  <artifactId>chapter1</artifactId>
  <version>0.0.1-SNAPSHOT</version>

  <name>chapter1</name>
  <description>chapter1</description>

  <properties>
    <java.version>11</java.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>

</project>
```

</div>
</details>

스프링, 스프링부트 모두 라이브러리에 대한 의존성 관리는 `pom.xml`(Project Object Model)이라는 파일을 통해 관리하게 된다. 이 때, 스프링이라면 사용할 라이브러리의 각각의 모듈에 대해 <dependencies> 태그 내에 <groupId>, <artifactId>, <version> 태그를 모두 작성해주어야 한다. 때문에 pom.xml 파일이 길어지는 것은 물론, 어느 한 라이브러리의 버전을 올릴 때에도 연관된 다른 라이브러리들의 버전을 모두 신경써주어야 한다. 하지만 스프링부트에서는 `spring-boot-starter`라는 의존성을 주입해 의존성 조합을 제공받고, <parent> 태그 내의 `start-parent`가 의존성 조합들간 충돌 문제가 없는 검증된 버전들을 알아서 추가 한다. 때문에 `pom.xml` 에 <version> 태그들을 일일히 적을 필요가 없는 것이다. 

다음은 많이 사용되는 `spring-boot-starter` 라이브러리들이다.
* `spring-boot-starter-web`: 스프링 MVC를 사용하는 RESTful 애플리케이션을 만들기 위한 의존성. 기본적으로 톰캣(Tomcat)이 포함되어 있어 jar 형식으로 실행 가능하다.
* `spring-boot-starter-test`: JUnit Jupiter, Mockito등 단위 테스트 라이브러리를 포함한다.
* `spring-boot-starter-jdbc`: HikariCP 커넥션 풀을 활용한 JDBC 기능을 제공한다.
* `spring-boot-starter-security`: 스프링 시큐리티(인증, 권한, 인가 등) 기능을 제공한다.
* `spring-boot-starter-data-jpa`: 하이버네이트를 활용한 JPA 기능을 제공한다.
* `spring-boot-starter-cache`: 스프링 프레임워크의 캐시 기능을 지원한다.

### 자동 설정 

스프링 부트는 스프링 프레임워크의 기능을 사용하기 위한 자동 설정(Auto Configuration)을 지원한다. `@SpringBootApplication`이라는 어노테이션을 작성해 놓으면, 프레임워크가 다음 어노테이션들을 읽어내어 애플리케이션을 개발하는 데 필요한 의존성을 추가할 때 자동으로 관리해준다. [^5]
* `SpringBootConfiguration`: 빈 팩토리를 위한 오브젝트를 설정을 담당하는 클래스라고 인식 할 수 있도록 알려주는 어노테이션이다.
* `EnableAutoConfiguration`: Spring boot의 핵심으로써, 미리 정의되어 있는 Bean들(spring-boot-autoconfigure > META-INF > spring.factories)을 가져와서 등록해준다. 
* `ComponentScan`: 현재 패키지 이하에서 아래와 같은 어노테이션이 붙어 있는 클래스들을 찾아서 빈으로 등록하는 역할을 한다.
    * `@Component`
    * `@Configuration`
    * `@Repository`
    * `@Service`
    * `@Controller`
    * `@RestController`

### 내장 WAS

스프링 부트의 각 웹 애플리케이션에는 내장 WAS(Web Application Server)가 존재하며, 웹 의존성인 `spring-boot-starter-web`의 경우, 톰캣을 내장한다. 때문에 특별한 설정 없이도 톰캣을 실행할 수 있다. 필요에 따라서 톰캣이 아닌 다른 웹 서버(Jetty, Undertow 등) 으로 대체 가능하다.

### 모니터링
서비스를 운영할 때 시스템이 사용하는 스레드, 메모리, 세션 등의 주요 요소를 모니터링 해야하는데, 스프링 부트네는 스프링 부트 액추에이터(Spring Boot Actuator)라는 자체 모니터링 도구가 내장되어 있다.

## 출처
***

[^1]: <https://en.wikipedia.org/wiki/Spring_Framework>
[^2]: <https://docs.spring.io/spring-framework/docs/4.0.x/spring-framework-reference/html/overview.html#overview-modules>
[^3]: <https://spring.io/projects/spring-boot>
[^4]: <https://mirotic91.tistory.com/4>
[^5]: <https://velog.io/@suwon-city-boy/%EC%8A%A4%ED%94%84%EB%A7%81%EA%B3%BC-%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8%EC%9D%98-%EC%B0%A8%EC%9D%B4>