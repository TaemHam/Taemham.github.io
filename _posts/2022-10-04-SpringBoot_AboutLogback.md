---
title: "[백엔드|스프링부트] Logback으로 로깅하기"
author: TaemHam
date: 2022-10-04 15:30:00 +0900
categories: [Backend, SpringBoot]
tags: [Tutorial, Backend, Spring, SpringBoot, Java, Logback]
---

<img src="https://user-images.githubusercontent.com/95671168/193757141-bc371e9f-d2ce-4f0f-a7d8-53704c3a51d0.png"/><em>Logback 로고[^1]</em>

운영 중인 웹 어플리케이션이 문제가 발생했을 경우, 문제의 원인을 파악하려면 문제가 발생했을 때 당시의 정보가 필요하다. 이런 정보를 얻기 위해서 Exception이 발생했거나, 중요 기능이 실행되는 부분에서는 적절한 로그를 남겨야한다.

자바에서는 이런 로깅을 도와주는 툴로 log4j나 Logback을 많이 사용한다. Logback은 기존의 log4j에서 불편한 점들을 개선해 나온 툴로, 오늘은 이 Logback의 특징과 적용 방법에 대해 글을 쓰려 한다.

## 특징
***

* 로그 레벨 필터를 설정해, 로그 파일이 불필요한 로그로 채워지는 걸 방지할 수 있다.
  * 5개의 로그 레벨:
    * ERROR: 요청을 처리하는 중 오류가 발생한 경우
    * WARN: 처리 가능한 문제, 향후 시스템 에러의 원인이 될 수 있는 경우
    * INFO: 상태 변경 등의 정보를 표시하는 경우
    * DEBUG: 디버깅을 위해 메시지 표시가 필요한 경우
    * TRACE: DEBUG보다 훨씬 상세한 정보가 필요한 경우
  * ex) INFO로 설정하는 경우 하위 레벨인 DEBUG 와 TRACE레벨의 로그는 출력되지 않는다.
* 내부 알고리즘으로 설정 파일을 일정 시간마다 스캔해 로드한다.
  * 설정 파일 변경 후 서버를 재시작하지 않아도, 수정된 설정 사항으로 로그가 출력된다.
* `spring-boot-starter-web`에 내장되어 있어, 의존성 추가 없이 사용 가능하다.

## 적용 방법
***

프로젝트에 Swagger를 적용하려면, 다음과 같은 3가지만 처리해주면 된다.

1. xml 작성
2. 어노테이션 추가

### 예제 환경

* Spring boot 2.5.6
* Maven

### XML 작성

resources 디렉토리 안에 Logback 설정 파일을 추가한다. 스프링부트 환경에서는 `logback-spring.xml` 이라는 이름으로, 그 외는 `logback.xml` 이라는 이름으로 생성한다. 

다음은 Logback 설정파일에 대한 예시이다. [^2]

#### Property 영역

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true" scanPeriod="60 seconds">

    <!-- 속성 또는 변수 설정 -->
    <property name="${name}" value="${value}"/>
```

`configuration`의 속성값으로 `scan` 과 `scanPeriod`를 부여할 수 있다. 설정시, 서버 가동 중 해당 파일을 몇 초 간격으로 다시 읽어들일지 변경할 수 있다.

`property` 태그로 변수 값을 설정할 수 있다. 다른 태그에서 `${name}` 형식으로 불러올 수 있다.

#### Appender 영역

```xml
    <!-- 어떤 속성의 appender를 사용할지 클래스 및 이름 설정 -->
    <appender name="${name}" class="${appender class}">
        <!-- 각 append class에 맞는 설정값이 다름.
             ch.qos.logback.core.ConsoleAppender,
            ch.qos.logback.core.FileAppender,
            ch.qos.logback.core.rolling.RollingFileAppender,
            ch.qos.logback.classic.db.DBAppender,
            ch.qos.logback.classic.net.SMTPAppender 등
        -->
    </appender>
```

`appender` 영역은 로그의 형태를 설정하고 어떤 방법으로 출력할지를 설정하는 곳이다. 

다음과 같이 `class` 값을 변경해 출력 값을 어디에 저장할지 선택할 수 있다.

* `ch.qos.logback.core.ConsoleAppender`

  콘솔에 로그를 출력한다. 로그를 OutputStream에 작성하여 콘솔에 출력되도록 한다.

  ```xml
  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
      <encoder>
          <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%level] [%logger] : %msg%n</pattern>
      </encoder>
  </appender>
  ```

* `ch.qos.logback.core.FileAppender`

  모든 로그를 각각의 파일에 저장한다. 최대 보관 일 수 등를 지정할 수 있다.

  ```xml
  <timestamp key="bySecond" datePattern="yyyyMMdd'T'HHmmss"/>

  <appender name="FILE" class="ch.qos.logback.core.FileAppender">
      <file>log-${bySecond}.txt</file>
      <append>true</append>
      <!-- set immediateFlush to false for much higher logging throughput -->
      <immediateFlush>true</immediateFlush>
      <!-- encoders are assigned the type
          ch.qos.logback.classic.encoder.PatternLayoutEncoder by default -->
      <encoder>
          <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>
      </encoder>
  </appender>
  ```

* `ch.qos.logback.core.rolling.RollingFileAppender`

  파일을 순회하며 로그를 저장한다. `rollingPolicy` 태그로 파일 순회 시점을 설정한다. 

  ```xml
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
      <file>logFile.log</file>
      <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
        <!-- daily rollover -->
        <fileNamePattern>logFile.%d{yyyy-MM-dd}.log</fileNamePattern>

        <!-- keep 30 days' worth of history capped at 3GB total size -->
        <maxHistory>30</maxHistory>
        <totalSizeCap>3GB</totalSizeCap>

      </rollingPolicy>

      <encoder>
        <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>
      </encoder>
    </appender> 
  ```


* `ch.qos.logback.classic.net.SMTPAppender`

  로그를 메일로 보낸다.

  ```xml
  <appender name="EMAIL" class="ch.qos.logback.classic.net.SMTPAppender">
      <smtpHost>ADDRESS-OF-YOUR-SMTP-HOST</smtpHost>
      <to>EMAIL-DESTINATION</to>
      <to>ANOTHER_EMAIL_DESTINATION</to> <!-- additional destinations are possible -->
      <from>SENDER-EMAIL</from>
      <subject>TESTING: %logger{20} - %m</subject>
      <layout class="ch.qos.logback.classic.PatternLayout">
          <pattern>%date %-5level %logger{35} - %message%n</pattern>
      </layout>       
  </appender>
  ```

* `ch.qos.logback.classic.db.DBAppender`

  DB(데이터베이스)에 로그를 쌓는다. 이 경우, logging_event, logging_event_property, logging_event_exception 세 개의 데이터 베이스 테이블을 insert 하므로 DBAppender를 사용하기 전에 먼저 테이블을 생성해야 한다. [DB 작성법](https://agileryuhaeul.tistory.com/entry/Logback-%EC%9D%B4%EB%9E%80#:~:text=DBAppender%20%EC%82%AC%EC%9A%A9%EC%8B%9C%20DB%20%ED%95%84%EC%9A%94%ED%95%9C%20Table) [^2]

  ```xml
  <property resource="application.properties" />
  <springProperty name="spring.datasource.driverClassName" source="spring.datasource.driverClassName"/>
  <springProperty name="spring.datasource.url" source="spring.datasource.url"/>
  <springProperty name="spring.datasource.username" source="spring.datasource.username"/>
  <springProperty name="spring.datasource.password" source="spring.datasource.password"/>

  <appender name="DB" class="ch.qos.logback.classic.db.DBAppender">
      <connectionSource class="ch.qos.logback.core.db.DriverManagerConnectionSource">
          <driverClass>${spring.datasource.driverClassName}</driverClass>
          <url>${spring.datasource.url}</url>
          <user>${spring.datasource.username}</user>
          <password>${spring.datasource.password}</password>
      </connectionSource>
  </appender>
  ```

#### Encoder 영역

```xml
    <appender name="${name}" class="${appender class}">
    <!-- 어떤 형식으로 로그할지 정의-->
        <encoder>
            <pattern>${pattern}</pattern>
        </encoder>
    </appender>
```

Appender 태그 안에 존재하는 영역으로, 로그 표현 형식을 패턴으로 정의한다. 

|패턴|의미|
|---|------|
| %logger{length}|Logger name을 축약 할 수 있다. {length}는 최대 글자 수 ex)logger{35}|
| %-5level|로그 레벨, -5는 출력의 고정폭 값(5글자) ex) [INFO ]|
| %msg|로그 메세지 ( = %message)|
| ${PID:-}|프로세스 아이디|
| %d|로그 기록시간 (뒤에 [yyyy-MM-dd HH:mm:ss.SSS] 의 형식이 따라온다.)|
| %p|로깅 레벨|
| %F|로깅이 발생한 프로그램 파일명|
| %M|로깅일 발생한 메소드의 명|
| %I|로깅이 발생한 호출지의 정보|
| %L|로깅이 발생한 호출지의 라인 수|
| %thread|현재 Thread 명|
| %t|로깅이 발생한 Threrad명|
| %c|로깅이 발생한 카테고리|
| %C|로깅이 발생한 클래스 명|
| %m|로그 메세지|
| %n|줄바꿈|
| %%|%를 출력|
| %r|애플리케이션 시작 이후 부터 로깅이 발생한 시점 까지의 시간 (ms)|

#### Root 영역

```xml
    <!-- 전체 로그 출력설정 -->
    <root level="${property}">
        <appender-ref ref="${appender}"/>
        <appender-ref ref="${appender}"/>
        <appender-ref ref="${appender}"/>
    </root>

    <!-- or -->

    <!-- 패키지 별 로그 출력설정 -->
    <logger name="org.hibernate" level="debug">
        <appender-ref ref="${appender}"/>
        <appender-ref ref="${appender}"/>
    </logger>
    <!-- 해당 패키지 하위 로그를 출력하고 싶지 않을때 additivity="false" -->
    <logger name="org.springframework" level="debug" additivity="false">
        <appender-ref ref="${appender}"/>
        <appender-ref ref="${appender}"/>
    </logger>

</configuration>
```

앞서 정의한 Appender 들을 어느 로그 레벨로 출력할지 설정하는 곳이다. 레벨별로 일괄적으로 설정하고 싶다면 `root` 태그를, 패키지별로 설정하고 싶다면 `logger` 태그를 사용하면 된다.

level 속성에는 INFO, DEBUG 등의 로그 레벨을, name 속성에는 패키지 단위로 로깅이 적용될 범위를, additivity 속성에는 앞에서 지정한 패키지 범위에 하위 패키지를 포함할지 여부를 넣는다.

### 어노테이션 추가

기존에 작성한 코드 중 어디에서 로그를 출력할지 어노테이션으로 추가한다. 로그 기능을 사용할 클래스들에 다음과 같은 코드를 정적 변수로 추가한다.

```java
public class [클래스명] {
    private final Logger LOGGER = LoggerFactory.getLogger([클래스명].class);

    // 중략
}
```

그리고 로그를 출력하고 싶은 곳 마다 다음과 같이 메서드를 호출해주면 된다.

```java
    LOGGER.[로그 레벨]("메세지, {}", {}에 들어갈 변수)
```

## 출처
***

[^1]: <https://logback.qos.ch/>
[^2]: <https://agileryuhaeul.tistory.com/entry/Logback-%EC%9D%B4%EB%9E%80>