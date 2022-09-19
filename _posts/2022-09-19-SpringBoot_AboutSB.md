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

스프링 프레임워크는 기능별로 구분된 약 20여 개의 모듈로 구성돼 있다. 


## 출처

[^1]: <https://en.wikipedia.org/wiki/Spring_Framework>
[^2]: <https://docs.spring.io/spring-framework/docs/4.0.x/spring-framework-reference/html/overview.html#overview-modules>