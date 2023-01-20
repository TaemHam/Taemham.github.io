---
title: "[백엔드|스프링부트] 서버로 들어오는 값에 대한 유효성 간단하게 검사하는 법"
author: TaemHam
date: 2023-01-20 18:00:00 +0900
categories: [Backend, SpringBoot]
tags: [Tutorial, Backend, Spring, SpringBoot, Java, Validation]
---

![Springboot](https://images.velog.io/images/kimkevin90/post/c7296624-0ce1-4a9e-9c1f-a576ec9eafc4/springboot.png)

## 개요

회원 가입 같은 기능들은 유저가 값을 제대로 입력 했는지 검사를 해야한다. 보통 이런 경우는 프론트엔드에서 입력 값에 대한 검증을 하고 서버로 보내지만, 그렇다고 [프론트엔드에서'만' 유효성 검사를 해도](https://jojoldu.tistory.com/157) 문제가 생길 수 있다. 따라서 백엔드 측에서도 검증 코드를 작성해야만 한다.
스프링부트는 필드에 어노테이션을 붙이는 것만으로 검증 로직을 만들 수 있다. 어떻게 하는지 알아보도록 하자.

## 유효성 검사 방법

### 의존성 추가

스프링부트의 유효성 검사 기능을 사용하려면 먼저 아래와 같은 의존성을 추가해주면 된다. 추가 후에 Refresh 하는걸 잊지 말자.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

### 어노테이션 추가

컨트롤러 계층에서 파라미터에 `@RequestBody` 가 붙은 DTO 클래스로 이동해 검증이 필요한 필드에 어노테이션을 붙여주면 된다.

다음은 회원 가입시 필요한 정보에 검증 어노테이션을 붙인 간단한 예제이다.

1. 먼저 컨트롤러에 검증이 필요한 객체(`@RequestBody`가 붙어있는)에 `@Valid` 어노테이션을 붙인다.

```java
@RestController
@AllArgsConstructor
@RequestMapping("/user")
public class ValidationController {

    private final Logger LOGGER = LoggerFactory.getLogger(ValidationController.class);
    private final UserService userService;


    @PostMapping("/join")
    public ResponseEntity<String> join(
            @Valid @RequestBody JoinRequestDto dto) {
        return ResponseEntity.status(HttpStatus.OK).body(userService.join(UserDto.from(dto)));
    }
```

2. 다음, 유효성 검사가 필요한 객체 필드에 어노테이션을 붙인다.

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@ToString
@Builder
public class JoinRequestDto {

    @NotBlank
    String id;

    @Email
    String email;

    @Pattern(regexp = "01(?:0|1|[6-9])[.-]?(\\d{3}|\\d{4})[.-]?(\\d{4})$")
    String phoneNumber;

    @Min(value = 0) @Max(value = 150)
    int age;

    @Size(min = 0, max = 40)
    String description;
}
```

위처럼 각 필드에 어노테이션을 선언함으로 유효성 검사를 위한 조건을 설정할 수 있다.

대표적인 어노테이션은 다음과 같다.

* 문자열 검증 (String)
    * `@Null`: null 값만 허용한다.
    * `@NotNull` : null을 허용하지 않는다. 빈 문자열은 허용한다.
    * `@NotEmpty` : null과 ""을 허용하지 않는다. 공백(" ")은 허용한다.
    * `@NotBlank` : null, "", " " 모두 허용하지 않는다.

* 최댓값/최솟값 검증 (BigDecimal, BigInteger, Integer, Long)
    * `@DecimalMax(value = "[숫자형 문자열]"` : [숫자형 문자열] 보다 작은 값을 허용한다.
    * `@DecimalMin(value = "[숫자형 문자열]"` : [숫자형 문자열] 보다 큰 값을 허용한다.
    * `@Max(value = [숫자]` : [숫자] 보다 작은 값을 허용한다.
    * `@Min(value = [숫자]` : [숫자] 보다 큰 값을 허용한다.

* 값의 범위 검증 (BigDecimal, BigInteger, Integer, Long)
    * `@Positive` : 양수를 허용한다.
    * `@PositiveOrZero` : 0을 포함한 양수를 허용한다.
    * `@Negative` : 음수를 허용한다.
    * `@NegativeOrZero` : 0을 포함한 음수를 허용한다.

* 시간에 대한 검증 (Date, LocalDate, LocalDateTime)
    * `@Future` : 현재보다 미래의 날짜를 허용한다.
    * `@FutureOrPresent` : 현재를 포함한 미래의 날짜를 허용한다.
    * `@Past` : 현재보다 과거의 날짜를 허용한다.
    * `@PastOrPresent` : 현재를 포함한 과거의 날짜를 허용한다.

* 이메일 검증 (String)
    * `@Email` : 이메일 형식을 검사한다. ""는 허용한다.

* 자릿수 범위 검증 (BigDecimal, BigInteger, Integer, Long)
    * `@Digits(integer = [정수 자릿수], fraction = [소수 자릿수])` : [정수 자릿수] 와 [소수 자릿수]를 허용한다.

* Boolean 검증 (Boolean)
    * `@AssertTrue` : true인지 체크한다. null은 체크하지 않는다.
    * `@AssertFalse` : false인지 체크한다. null은 체크하지 않는다.

* 문자열 길이 검증 (String)
    * `@Size(min = [최솟값], max = [최댓값])` : 문자열의 길이가 [최솟값] 이상, [최댓값] 이하인지 확인한다.

* 정규식 검증 (String)
    * `@Pattern(regexp = "[정규표현식]")` : 문자열이 정규표현식으로 일치하는지 확인한다.

이 외에도 카드 번호의 형식을 확인하거나, URL의 형식을 확인하는 등 여러가지 어노테이션이 있다.
해당 어노테이션은 IntelliJ 에서 화면 우측의 [Bean Validation] 탭을 눌러 확인 가능하다.
    
### 예외 처리

위 처럼만 작성해도 유효성 검사에 실패할 데이터가 들어오면 컨트롤러에서 막아준다. 하지만 이렇게 끝내면 응답으로 400 Bad Request만 줄 뿐, 어느 필드가 어떤 이유 때문에 실패한 건지 알려주지 않는다.
따라서 해당 정보들을 보내주기 위해서는 `@RestControllerAdvice`를 이용해서 예외를 처리하는 코드를 작성해주어야 한다.

```java
@RestControllerAdvice
public class GlobalControllerAdvice {

    @ExceptionHandler({MethodArgumentNotValidException.class})
    public ResponseEntity<Map<String, List<String>>> validException(
            MethodArgumentNotValidException ex) {
        List<String> errors = ex.getBindingResult().getFieldErrors()
                .stream().map(FieldError::getDefaultMessage).collect(Collectors.toList());

        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(getErrorsMap(errors));
    }

    private Map<String, List<String>> getErrorsMap(List<String> errors) {
        Map<String, List<String>> errorResponse = new HashMap<>();
        errorResponse.put("errors", errors);
        return errorResponse;
    }
}
```

`@RestControllerAdvice` 어노테이션이 붙은 전역 예외처리 클래스에서 `@ExceptionHandler` 로 발생할 예외를 잡아 처리해주는 코드이다. 그 바로 다음을 보면 알겠지만, 검증 실패시 발생하는 예외는 `MethodArgumentNotValidException` 이다. 

검사에 탈락한 필드들에 대한 정보는 `ex.getBindingResult().getFieldErrors()` 를 통해 접근할 수 있다. 여기서 스트림을 통해 `.getDefaultMessage()` 메서드를 실행시키면, 검증 실패시 기본적으로 설정된 메시지를 받을 수 있다.

만약 이 기본 메시지를 직접 편집하고 싶다면, 다시 검증 어노테이션이 붙은 DTO로 찾아가 각 어노테이션마다 `message = "[검증 실패시 보여질 메시지]"` 속성을 붙이면 된다. 

```java
public class JoinRequestDto {

    @NotBlank(message = "아이디가 공백입니다.")
    String id;
    //...
}
```

## 참고 자료
***
* [프론트엔드에서"만" 유효성 검사가 문제인 이유](https://jojoldu.tistory.com/157)
* [SpringBoot API 요청 값 검증하고 Validation Exception Handing하기](https://shinsunyoung.tistory.com/72)
* [Validation and Exception Handling in Spring Boot](https://salithachathuranga94.medium.com/validation-and-exception-handling-in-spring-boot-51597b580ffd)