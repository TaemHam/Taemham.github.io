---
title: "[CS] HTTP 1.0 부터 3.0까지: HTTP 규약은 어떻게 변화해 왔나"
author: TaemHam
date: 2023-02-13 09:00:00 +0900
categories: [CS, Network]
tags: [CS, Network, HTTP, HTTPS]
---

![](https://www.investopedia.com/thmb/Sh6RKE3ItNkCDYbjI40DcuLlaS4=/750x0/filters:no_upscale():max_bytes(150000):strip_icc():format(webp)/IPaddress-1e113dd325f74135ab2183d432ea5865.jpg)

## 개요

인터넷에서 웹 서버와 사용자 컴퓨터에 설치된 웹 브라우저 사이에 문서를 전송하기 위한 통신 규약인 HTTP 프로토콜은, 당연하게도 처음부터 완벽한 상태로 나오지는 않았다. 개발자들은 '어떻게 하면 더 빨리, 더 안전하게 응답을 보낼 수 있을까'에 대해 고민하며 데이터 송수신 방식을 발전시켜왔고, 그에 따라 여러가지 규약들이 생겨났다. 이번엔 이 HTTP의 규약이 처음엔 어땠고, 어떤 변화를 거쳐왔는지 알아보도록 하자.

## HTTP/1.0

![](https://media.licdn.com/dms/image/C5612AQE9qQFC78CzbA/article-inline_image-shrink_1000_1488/0/1623005286462?e=1681948800&v=beta&t=x7U297-FWx5nRJ5whRCGxxBcmcmP9TY6bolhBEunzCg)

### 특징
* TCP 기반으로 연결을 설정한다.
* 한 연결에 하나의 요청을 처리하도록 설계 되었다. 
* GET, HEAD, POST의 메서드만을 지원한다.
  * GET : Request-URI에서 지정한 정보를 Entity Body로 전달해달라는 요청
  * HEAD : Header의 정보만 요구
  * POST : Request 메시지의 body에 포함된 자원을 Request-URI로 넘겨주는 경우 사용

### 한계
* 서버로부터 파일을 가져올 때마다 TCP의 3-웨이 핸드셰이크를 열어야해서 RTT(라운드 트립, 패킷 왕복 시간)가 증가하는 단점이 있다.
  * RTT 증가에 대한 해결책으로 이미지 스프라이트(여러 개의 이미지를 하나의 이미지로 합쳐서 관리하는 이미지), 코드 압축(빈 칸 삭제), 이미지 Base64 인코딩 방식을 사용했다. 

## HTTP/1.1

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FxFnJz%2FbtrNosPHQNi%2F03ZVFkvTWwVcGfC9K9ULi1%2Fimg.png)

### 특징
* HTTP/1.0에서 발전한 것으로, 매번 TCP 연결을 하는 것이 아니라, 한번 **TCP를 초기화 한 후 Keep-alive 옵션으로 여러개의 파일을 송수신** 할 수 있게 바뀌었다.
* **여러 개의 요청을 동시에 보내**는 Pipelining을 지원해 응답 속도를 높일 수 있게끔 했다.
* OPTION, PUT, DELETE, TRACE의 메서드가 추가되었다.
  * OPTION : 통신과 관련된 선택사항들에 대한 정보를 요구하는 경우
  * PUT : Request 메시지에 포함되어 있는 data를 지정한 Request-URI로 저장하기 위함
  * DELETE : 특정 resource를 지우기 위함
  * TRACE : 최종 destination까지의 Loopback을 테스트하기 위함

### 한계
* 문서 안에 포함된 다수의 리소스를처리하려면, 요청할 리소스 개수에 비례해서 대기 시간이 길어지는 단점이 있다.
* HOL Blocking(Head Of Line Blocking)으로 전송이 빠르지만 나중에 도착한 패킷(css 등)보다 전송이 느리지만 먼저 도착한 패킷(이미지 등)을 먼저 전송해야해서 성능 저하 현상이 발생한다.
* HTTP/1.1의 헤더에는 쿠키 등 많은 메타데이터가 들어 있고 압축이 되지 않아 무겁다.

## HTTP/2

![](https://file.okky.kr/images/1645506175857.png)

### 장점
* **여러 개의 스트림을 사용해 송수신**하는 멀티플렉싱을 지원한다. 멀티플렉싱은 하나의 TCP 연결로 동시에 여러개의 메시지를 주고 받고, 응답을 순서에 상관없이 스트림으로 주고받는 방식을 말한다. 즉, HTTP 상의 HOL Blocking을 해결하게 되었다.
* **중복되는 헤더의 반복 전송을 낮추**기 위해, 허프만 코딩 압축 알고리즘(중복 횟수가 많은 정보는 적은 비트 수를 사용해 표현하며 데이터의 표현에 필요한 비트양을 줄이는 방식)을 사용해 헤더를 HPACK 압축 형식으로 압축해서 전송한다.
* 서버 푸시 기능을 통해 클라이언트가 **요청하지 않은 데이터라도 서버가 스스로 전송**해줄 수 있도록 했다. html에 css와 js가 포함되어있다면, html 요청 한 번에 html, css, js 응답 3개를 모두 전송하는 것이다.

### 한계
* TCP 기반으로 동작하기에 핸드셰이크 과정이 필수적이고, 그에 따른 지연 시간이 발생한다.
* 패킷이 유실되거나 오류가 있을때 재전송을하는데, 이 패킷을 재전송하는 동안 전체 패킷 전송이 중단되게 된다. HOL Blocking 문제가 여전히 발생하는 것이다.

## HTTP/3

![](https://velog.velcdn.com/images%2Fwsong0101%2Fpost%2Fc0cb6743-6d56-4203-8068-0a12180bad24%2F3.png)

### 특징
* TCP 기반이 아닌, UDP 기반으로 **QUIC 프로토콜을 사용해 연결 설정**을 하기 때문에 3-웨이 핸드셰이크 과정을 필요로 하지 않는다. 또, 보안 통신을 위해 TLS를 이용할 때에도 TCP+TLS 로 다수의 라운드 트립이 필요했던 HTTP/2와 다르게, Quic은 TLS 인증서를 내포하고 있기 때문에 인증도 단축되었다. 결과적으로 첫 연결 설정에 한 번의 라운드 트립만 필요하고, 덕분에 **초기 연결의 시간이 단축**되었다.
* 순방향 오류 수정 메커니즘이 적용되어, 패킷이 손실 되었어도 수신 측에서 에러를 검출하고 수정할 수 있어 **패킷 손실률이 낮다**.
* 각 스트림을 스트림 식별자로 식별함으로 스트림 중 하나에서 **어떤 패킷이 손실되더라도 해당 스트림만 멈추게 된다**. 즉, TCP상의 HOL Blocking 문제까지 해결하게 되었다. 
* QUIC은 Connection ID를 사용하여 서버와 연결을 생성하기 때문에, 네트워크 변경과 같이 **IP가 바뀌는 상황에도 연결을 유지**할 수 있다.

### 한계
* QUIC은 패킷별로 암호화를 한다. 이는 기존의 TLS-TCP에서 패킷을 묶어서 암호화하는 것보다 더 큰 리소스 소모를 불러올 수 있다는 단점이 있다.
* UDP 기반이기 때문에 패킷 차단에 필요한 정보가 거의 없어 Ddos 공격에 대응하기 어렵고, 또 IP 스푸핑으로 IP를 속인 채 요청을 대량으로 보내면 피해 IP는 원치 않는 데이터를 대량으로 수신하게 되는 반사 공격을 할 수도 있다. 

## 예상 질문

* HTTP/1.0 에 대해 설명해주세요.

<details>
<summary>답변</summary>

1. HTTP/1.0는 TCP를 기반으로 한 번의 연결로 하나의 요청을 처리할 수 있도록 설계된 버전입니다.
2. 요청 하나마다 3-웨이 핸드셰이킹으로 연결하고 4-웨이 핸드셰이킹으로 종료해야 하므로, 라운드 트립이 여러번 발생한다는 단점이 있습니다.
3. 이런 단점에 대한 해결책으로 이미지 스프라이트나 Base64 인코딩, 코드 압축 같은 기술이 도입되었습니다.
  * 이미지 스프라이트란 여러 개의 이미지를 하나로 합쳐 관리하는 기법입니다.
  * Base64 인코딩은 이미지파일을 64진법으로 이루어진 문자열로 인코딩해 이미지에 대한 요청을 없애는 기법입니다.
  * 코드 압축은 코드상의 개행 문자 같은 빈칸을 없애 크기를 최소화하는 방법입니다.

</details>

---

* HTTP/1.1은 1.0과 무슨 차이가 있는지 설명해주세요.

<details>
<summary>답변</summary>

1. HTTP/1.1은 한번의 연결로 많은 요청을 처리할 수 있도록 하는 Keep-alive 옵션이 생겼습니다.
2. 또, 파이프 라이닝을 지원해 여러 개의 요청을 동시에 보낼 수 있도록 할 수 있습니다.
3. 기존의 GET, POST, HEAD 메서드에서 추가되어 PUT, DELETE, OPTION 등의 메서드가 추가되었습니다.

</details>

---

* HTTP/2.0에 대해 설명해주세요.

<details>
<summary>답변</summary>

1. HTTP/2.0는 1.0보다 지연 시간을 줄이고 응답 시간을 빠르게 할 수 있도록 설계된 버전입니다. 
2. 2.0의 장점으로는 멀티플렉싱, HPACK 방식의 헤더 압축, 서버 푸시 등이 있습니다.
  * 멀티플렉싱은 여러 개의 스트림을 사용해 패킷을 송수신 하는 것입니다.
  * HPACK은 허프만 코딩 알고리즘으로 헤더를 압축한 것을 말하는데, 중복되는 헤더를 그 횟수에 따라 다른 비트로 표현하는 것입니다.
  * 서버 푸시란, 한 번의 요청으로 관련된 자료를 같이 응답으로 주는 것입니다.
3. 단점으로는, TCP 상의 HOL Blocking을 해결하지 못했다는 것과, TCP와 TLS 사용으로 초기 연결 시간이 길다는 것입니다.

</details>

---

* HTTP/3.0에 대해 설명해주세요.

<details>
<summary>답변</summary>

1. HTTP/3.0은 2.0까지 사용되어 왔던 TCP가 아닌, UDP 기반의 QUIC 프로토콜을 이용해 통신하는 버전입니다.
2. 3.0의 장점으로는 1-RTT 연결 설정, 스트림 식별자 사용, Connection ID 사용 등이 있습니다.
  * 3.0는 TLS 인증서를 내포한 QUIC 프로토콜로 연결을 설정하므로, 한 번의 RTT로 연결 설정이 완료됩니다.
  * 또, 스트림 식별자를 사용해 패킷 유실시에도 해당 패킷과 관련된 스트림만 멈춰 HOL Blocking을 해결합니다.
  * 그리고 IP 대신 Connection ID로 연결을 설정해, 네트워크 변경시에도 연결을 유지할 수 있습니다.
3. 단점으로는, QUIC은 패킷별로 암호화를 해 리소스 소모가 크다는 점, QUIC 폭주나 반사 공격 등 보안 취약점이 아직 존재한다는 점이 있습니다.

</details>

---

## 참고 자료
***

* [[Network] HTTP 1.0 vs HTTP 1.1](https://it-mesung.tistory.com/146)
* [HTTP/3는 왜 UDP를 선택한 것일까?](https://evan-moon.github.io/2019/10/08/what-is-http3/)
* [HTTP3, 사실 진짜로 바뀐건 TCP 였다.](https://www.hamadevelop.me/http3/)
* [QUIC 폭주 DDoS 공격이란? | QUIC 폭주와 UDP 폭주](https://www.cloudflare.com/ko-kr/learning/ddos/what-is-a-quic-flood/)