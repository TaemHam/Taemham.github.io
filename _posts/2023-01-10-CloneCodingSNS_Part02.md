---
title: "[SNS 클론코딩] 회원가입과 로그인 기능 개발"
author: TaemHam
date: 2023-01-10 20:00:00 +0900
categories: [Backend, SpringBoot]
tags: [Backend, Spring, SpringBoot, Java, CloneCoding, SNS]
---

![image.jpg1](https://user-images.githubusercontent.com/95671168/211244489-247008e7-0b4c-40e3-a190-6c504595ad32.png) |![image.jpg2](https://user-images.githubusercontent.com/95671168/211468415-6c562451-09f3-4fb4-9cd3-d7144b9aebad.png)
--- | --- | 

## 진행 상황

* 위의 시퀀스 다이어그램을 따라 회원가입, 로그인 관련 테스트 코드 작성.
* 회원가입과 로그인 기능에 필요한 엔티티, DTO, 컨트롤러, 서비스, 리포지토리 작성.
* 예외를 일괄적으로 처리하기 위해 @RestControllerAdvice를 사용해 `GlobalControllerAdvice` 정의

## 배운 점
![JWT](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbE1ajd%2Fbtq6SvTtY7B%2FdfBjF0kq3E7GJg4LB2gi90%2Fimg.png)

* JWT 구현 방법

JWT란, 토큰 기반 인증 시스템의 대표적인 구현체이다. `.`을 기준으로 `헤더.내용.서명`으로 나타난다.

```java
public static String generateToken(String userName, String key, long expiredTimeMs) {
    //Claims: 내용 부분에 담을 것들
    Claims claims = Jwts.claims(); 
    claims.put("userName", userName);

    return Jwts.builder()
            // 내용
            .setClaims(claims)
            // 발생 일시
            .setIssuedAt(new Date(System.currentTimeMillis()))
            // 만료 일시
            .setExpiration(new Date(System.currentTimeMillis() + expiredTimeMs))
            // 사용할 암호화 알고리즘
            .signWith(getKey(key), SignatureAlgorithm.HS256)
            .compact();
}

private static Key getKey(String key) {
    byte[] keyBytes = key.getBytes(StandardCharsets.UTF_8);
    return Keys.hmacShaKeyFor(keyBytes);
}
```

## 아이디어 메모

* JWT에 관해 조사해보다가, [JWT 토큰이 탈취되는 경우](https://velog.io/@park2348190/JWT%EC%97%90%EC%84%9C-Refresh-Token%EC%9D%80-%EC%99%9C-%ED%95%84%EC%9A%94%ED%95%9C%EA%B0%80)에 대해 알게 되었다. 만약 공격자가 이 토큰을 탈취 할 경우, 토큰이 만료될 때 까지 공격자는 정상적인 사용자인 척 위장할 수 있게 된다. 이를 방어하기 위해서 인증 유지에 필요한 Access 토큰 과 Refresh 토큰을 따로 두고, Access 토큰의 유효 기간을 Refresh 토큰보다 짧게 설정해, 유효 기간이 지날 때마다 Access 토큰을 재발급 받도록 한다고 한다. 오늘 구현한 내용에 반영하기엔 방대한 내용이라 적용하진 못했지만, 프로젝트 백로그에 적어두고, 추후 구현해보아야겠다.