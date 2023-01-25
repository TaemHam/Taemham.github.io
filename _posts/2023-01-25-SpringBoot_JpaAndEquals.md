---
title: "[백엔드|스프링부트] 프록시 객체때문에 발생할 수 있는 equals 예외"
author: TaemHam
date: 2023-01-25 09:00:00 +0900
categories: [Backend, SpringBoot]
tags: [Tutorial, Backend, Spring, SpringBoot, Java, JPA, Hibernate, Proxy]
---

## 개요

JPA를 통해 객체를 불러오면서도 모든 연관된 엔티티를 전부 불러오고 싶지는 않다면, JPA가 지원하는 지연로딩 방식을 사용하면 된다. 그러면 JPA는 하이버네이트 구현체가 만든 프록시 객체로 불러와 데이터의 자리를 메워준다. 이 프록시 객체는 실제 데이터를 DB에서 불러오지 않고도 데이터가 존재하는 것처럼 해줘서, 해당 데이터를 실제로 접근하기 전까지 불러오는 걸 미뤄둘 수 있도록 해주는 유용한 객체이다. 하지만 그런 차이점 때문에 **프록시 객체를 실제 객체처럼 대하다가는 오류가 발생할 수 있다**. 이번 글에서는 그렇게 발생할 수 있는 오류에 대해 알아보고자 한다.

### 프록시의 equals

자바에서 객체 비교를 하기 위해 보통 인텔리제이의 기능을 빌려 equals 메서드를 오버라이딩 한다. 그렇게 작성된 엔티티는 다음과 같다.

```java
@Getter
@Entity
public class Post {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Setter
    @JoinColumn(name = "userId")
    @ManyToOne(fetch = FetchType.LAZY, optional = false)
    private UserAccount userAccount;

    // 기타 필드 생략...
}

@Getter
@Entity
public class UserAccount {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long userId;

    //... 생략

    @Override
    public boolean equals(Object o) {
        if (this == o) {
            return true;
        }
        if (o == null || getClass() != o.getClass()) {
            return false;
        }
        UserAccount userAccount = (UserAccount) o;
        return Objects.equals(id, userAccount.id);
    }
}
```

이 때, 다음과 같이 **하나의 Post에서 동일한 UserAccount를 호출해 비교**하는 테스트 코드를 실행시켜보자.

```java
@Test
void name() {
    Post post = postRepository.findById(1L).orElseThrow();
    UserAccount userAccount1 = post.getUserAccount();
    UserAccount userAccount2 = post.getUserAccount();
    assertThat(userAccount1).isEqualTo(userAccount2);
}
```

하나의 엔티티에서 가져온 동일한 엔티티여야 하지만, 테스트 코드는 실패를 반환한다.

```bash
org.opentest4j.AssertionFailedError: 
expected: "UserAccount(userId=1) (UserAccount$HibernateProxy$luj7HgwZ@5f8d4b51)"
 but was: "UserAccount(userId=1) (UserAccount$HibernateProxy$luj7HgwZ@5f8d4b51)"
```

메시지를 보면 필드도 주솟값도 같은 객체인데도, 테스트에 실패한다는 메시지를 보여준다. 어디에서 문제가 있는 걸까?

원인은 `getClass()` 에 있다. 
`equals()`를 호출하는 객체를 A라 하고, 그 인자로 받은 객체는 B라 하자. A는 확실히 UserAccount 클래스가 맞다. 하지만 B의 클래스는 UserAccount가 아닌, 하이버네이트가 만든 프록시 객체인 것. 고치려면 조건문을 `if (!(o instanceof UserAccount))` 로 바꾸면 된다.

그렇다면 클래스 비교만이 문제일까? 코드를 고치고 다시 테스트 코드를 실행해보면 똑같은 오류가 또 다시 나온다. `Objects.equals(id, userAccount.id)` 에도 문제가 있다는 뜻이다.

하이버네이트가 만들어주는 프록시 객체는 **실제 객체의 상속본**이며, 실제 객체와 다르게 **필드가 존재하지 않고 메서드로만 값을 조회할 수 있다.** 따라서 인자로 받은 객체 B의 id를 `userAccount.id` 로 접근하면 id 필드가 존재하지 않아 null을 반환하게 되는 것이다. 고치려면 getter 메서드를 이용해 필드를 조회하도록 바꾸면 된다.

위 두 오류를 고쳐 다음과 같이 작성하면 드디어 테스트 코드를 통과하게 된다.

```java
    @Override
    public boolean equals(Object o) {
        if (this == o) {
            return true;
        }
        if (!(o instanceof UserAccount that)) { // that을 패턴 변수로 변환해 한 줄을 줄였다.
            return false;
        }
        return this.getUserId() != null && this.getUserId().equals(that.getUserId());
    }
```

## 참고 자료
***
* [JPA Hibernate 프록시 제대로 알고 쓰기](https://tecoble.techcourse.co.kr/post/2022-10-17-jpa-hibernate-proxy/)
* [엔티티 관련 질문글](https://www.inflearn.com/questions/20180/orderitem-%EA%B4%80%EB%A0%A8)