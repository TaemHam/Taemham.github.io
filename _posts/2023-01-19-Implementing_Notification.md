---
title: "[백엔드|스프링부트] 알림 기능은 어떻게 구현하는게 좋을까?"
author: TaemHam
date: 2023-01-19 09:00:00 +0900
categories: [Backend, SpringBoot]
tags: [Tutorial, Backend, Spring, SpringBoot, Java, SSE]
---

![Redis](https://upload.wikimedia.org/wikipedia/en/thumb/6/6b/Redis_Logo.svg/1200px-Redis_Logo.svg.png)

## 개요

댓글이나 좋아요 알림은 유저의 요청 없이도 실시간으로 서버의 변경 사항을 웹 브라우저에 갱신해줘야 하는 서비스이다. 하지만 전통적인 Client-Server 모델의 HTTP 통신에서는 이런 기능을 구현하기가 어렵다. 클라이언트의 요청이 있어야만 서버가 응답을 할 수 있기 때문이다.
HTTP 기반으로 해당 문제를 해결하려면 다음과 같은 방식들이 있다.

## 실시간 통신의 방법

### Polling

![Polling](https://user-images.githubusercontent.com/95671168/213355195-79a05f17-9a34-4b78-ae31-8efce811488e.png)

일정 주기를 가지고 서버의 API를 호출하는 방법이다. 예를 들어, 클라이언트에서 5초마다 한 번씩 알림 목록을 호출한다면, 업데이트 내역이 5초마다 갱신되며 변경 사항을 적용할 수 있다. 이 방식은 기본적인 HTTP 통신을 기반으로 하기 때문에 호환성이 좋다는 장점이 있다.

하지만 해당 방식은 업데이트 주기가 길다면 실시간으로 데이터가 갱신 되지 않고, 또 짧다면 갱신 사항이 없음에도 서버에 요청이 들어와 불필요한 서버 부하가 발생한다는 것이다. 

### Long-Polling

![LongPolling](https://user-images.githubusercontent.com/95671168/213355200-6fe5431c-a2f3-4d52-807b-bb17f313fdf5.png)

Polling과 비슷하나, 업데이트 발생시에만 응답을 보내는 방식이다. 서버로 요청이 들어올 경우, 일정 시간동안 대기하였다가 요청한 데이터가 업데이트 되면 웹 브라우저에게 응답을 보낸다. Polling에서 불필요한 응답을 주는 경우를 줄이기 위해 사용할 수 있는 방법이다. 따라서 연결이 된 경우엔 실시간으로 데이터가 들어올 수 있다는 장점이 있다.

하지만 이 방식 또한 데이터 업데이트가 빈번하게 일어난다면 연결을 위한 요청도 똑같이 발생하므로, Polling과 유사하게 서버에 부하가 일어날 수 있다.

### SSE (Server-Sent Event)

![SSE](https://user-images.githubusercontent.com/95671168/213355206-3e4c3f0c-2247-406e-8d11-32b192652d11.png)

웹 브라우저에서 서버쪽으로 특정 이벤트를 구독하면, 서버에서는 해당 이벤트 발생시 웹브라우저 쪽으로 이벤트를 보내주는 방식이다. 따라서 한 번만 연결 요청을 보내면, 연결이 종료될 때까지 재연결 과정 없이 서버에서 웹 브라우저로 데이터를 계속해서 보낼 수 있다.

다만, 서버에서 웹 브라우저로만 데이터 전송이 가능하고, 그 반대는 불가능하다는 단점이 있다. 또, 최대 동시 접속 횟수가 제한되어 있다.

### Web Socket

![WebSocket](https://user-images.githubusercontent.com/95671168/213355212-00343881-ecea-44b9-8a9d-db722baff5f6.png)

서버와 웹브라우저 사이 양방향 통신이 가능한 방법이다. 변경 사항에 빠르게 반응해야하는 채팅이나, 리소스 상태에 대한 지속적 업데이트가 필요한 문서 동시 편집과 같은 서비스에 많이 사용되는 방식이다.

이 방식은 양방향 통신이 지속적으로 이루어질 수는 있으나, 연결을 유지하는 것 자체가 비용이 들기 때문에 트래픽 양이 많아진다면 서버에 큰 부담이 된다는 단점이 있다.

## 결론

Polling 방식은 실시간성을 높이려면 그 주기를 짧게 해야 하는데, 트래픽이 많아질 경우 서버에 걸리는 부하가 커지기 때문에 알림 서비스에는 부적합하고 할 수 있다. Long-Polling 역시 마찬가지로, 트래픽이 많아지면 요청도 그만큼 많아지므로 부적합하다.

그렇다면 HTTP 연결 방식에 대한 부담이 적은 SSE와 WebSocket 방식이 남는데, 알림 서비스의 경우 클라이언트에서 서버로 데이터를 전송하지 않아도 되어서 단방향 통신만으로도 구현할 수 있으므로, SSE 방식을 택하는 것이 좋겠다.

## 구현

spring framework 4.2부터 SSE 통신을 지원하는 SseEmitter 클래스를 이용해 구현할 계획이다.

### 연결 생성

* 컨트롤러

클라이언트에서 구독하는 요청을 보내면, 컨트롤러는 SseEmitter를 만들어주는 서비스 레이어를 통해 전달 받은 SseEmitter를 반환한다.

```java
@RestController
@RequestMapping("/api/v1/users/notification")
@RequiredArgsConstructor
public class NotificationController {

    private final NotificationService notificationService;
    
    @GetMapping("/subscribe")
    public SseEmitter subscribe(Authentication authentication) {
        // Authentication을 UserDto로 업캐스팅
        UserDto userDto = ClassUtils.getCastInstance(authentication.getPrincipal(), UserDto.class)
                .orElseThrow(() -> new ApplicationException(ErrorCode.INTERNAL_SERVER_ERROR,
                        "Casting to UserDto class failed"));
        
        // 서비스를 통해 생성된 SseEmitter를 반환
        return notificationService.connectNotification(userDto.getId());
    }
}
```

* 서비스

새로운 연결을 생성할 때에는 유저의 ID를 받아 SSE Emitter를 리포지토리에 저장하도록 했다. 이후, SSE 응답을 할 때 아무런 이벤트도 보내지 않으면 재연결 요청을 보낼때나, 아니면 연결 요청 자체에서 오류가 발생하기 때문에, 첫 응답을 보내주었다.

```java
@Slf4j
@Service
@RequiredArgsConstructor
public class NotificationService {
    private final static Long DEFAULT_TIMEOUT = 3600000L;
    private final static String NOTIFICATION_NAME = "notify";

    private final EmitterRepository emitterRepository;

    public SseEmitter connectNotification(Long userId) {
        // 새로운 SseEmitter를 만든다
        SseEmitter sseEmitter = new SseEmitter(DEFAULT_TIMEOUT);

        // 유저 ID로 SseEmitter를 저장한다.
        emitterRepository.save(userId, sseEmitter);

        // 세션이 종료될 경우 저장한 SseEmitter를 삭제한다.
        sseEmitter.onCompletion(() -> emitterRepository.delete(userId));
        sseEmitter.onTimeout(() -> emitterRepository.delete(userId));

        // 503 Service Unavailable 오류가 발생하지 않도록 첫 데이터를 보낸다.
        try {
            sseEmitter.send(SseEmitter.event().id("").name(NOTIFICATION_NAME).data("Connection completed"));
        } catch (IOException exception) {
            throw new ApplicationException(ErrorCode.NOTIFICATION_CONNECTION_ERROR);
        }
        return sseEmitter;
    }
}
```

* 리포지토리

유저 ID로 저장하고 불러올 수 있도록 간단하게 HashMap으로 구현했다. 불러올 SSE Emitter가 없을 경우를 대비해 Optional.ofNullable로 반환하도록 했다.

```java
@Slf4j
@Repository
public class EmitterRepository {

    // 유저ID를 키로 SseEmitter를 해시맵에 저장할 수 있도록 구현했다.
    private Map<String, SseEmitter> emitterMap = new HashMap<>();

    public SseEmitter save(Long userId, SseEmitter sseEmitter) {
        emitterMap.put(getKey(userId), sseEmitter);
        log.info("Saved SseEmitter for {}", userId);
        return sseEmitter;
    }

    public Optional<SseEmitter> get(Long userId) {
        log.info("Got SseEmitter for {}", userId);
        return Optional.ofNullable(emitterMap.get(getKey(userId)));
    }

    public void delete(Long userId) {
        emitterMap.remove(getKey(userId));
        log.info("Deleted SseEmitter for {}", userId);
    }

    private String getKey(Long userId) {
        return "Emitter:UID:" + userId;
    }
}
```

### 알림 전송

* 서비스

알림 서비스에 메서드를 추가하고, 알림이 발생할 때마다 해당 메서드를 호출하도록 구현했다.

```java
@Slf4j
@Service
@RequiredArgsConstructor
public class NotificationService {
    private final static Long DEFAULT_TIMEOUT = 3600000L;
    private final static String NOTIFICATION_NAME = "notify";

    private final EmitterRepository emitterRepository;

    public void send(Long userId, Long notificationId) {
        // 유저 ID로 SseEmitter를 찾아 이벤트를 발생 시킨다.
        emitterRepository.get(userId).ifPresentOrElse(sseEmitter -> {
            try {
                sseEmitter.send(SseEmitter.event().id(notificationId.toString()).name(NOTIFICATION_NAME).data("New notification"));
            } catch (IOException exception) {
        // IOException이 발생하면 저장된 SseEmitter를 삭제하고 예외를 발생시킨다.
                emitterRepository.delete(userId);
                throw new ApplicationException(ErrorCode.NOTIFICATION_CONNECTION_ERROR);
            }
        }, () -> log.info("No emitter found"));
    }
}
```


## 참고 자료
***
* [Spring에서 Server-Sent-Events 구현하기](https://tecoble.techcourse.co.kr/post/2022-10-11-server-sent-events/)
* [[HTTP vs Socket] HTTP와 소켓 통신의 장단점](https://velog.io/@jinh2352/HTTP-vs-Socket-HTTP%EC%99%80-%EC%86%8C%EC%BC%93-%ED%86%B5%EC%8B%A0%EC%9D%98-%EC%9E%A5%EB%8B%A8%EC%A0%90)
* [[Spring + SSE] Server-Sent Events를 이용한 실시간 알림](https://velog.io/@max9106/Spring-SSE-Server-Sent-Events%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EC%8B%A4%EC%8B%9C%EA%B0%84-%EC%95%8C%EB%A6%BC)