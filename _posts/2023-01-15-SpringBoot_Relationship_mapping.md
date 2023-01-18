---
title: "[백엔드|스프링부트] 연관 관계 매핑과 영속성 전이"
author: TaemHam
date: 2023-01-15 10:00:00 +0900
categories: [Backend, SpringBoot]
tags: [Tutorial, Backend, Spring, SpringBoot, Java, Relationship Mapping]
---

## 연관관계 매핑

* 연관관계 매핑이란?

    ```java
    @Entity
    public class Post { // 게시글과 댓글의 경우

        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long postId;

        @Column(nullable = false)
        private String content;

        @OneToMany
        private Comment comment; // 댓글 엔티티를 그대로 넣어도 된다!
    }
    ```

    * 하나의 엔티티에서 다른 엔티티를 참조할 수 있도록, 엔티티의 필드에 다른 엔티티를 대응시켜주는 것.

* 연관관계 매핑 사용 이유

    * 테이블의 외래키를 그대로 사용할 수도 있지만, 외래키 식별자를 직접 다루어야 하기 때문에 객체지향적이지 않음.

* 연관관계 매핑의 종류

    * 참조의 방향에 따라 **단방향**, **양방향**

    * 엔티티의 관계에 따라 **일대일**, **일대다**, **다대일**, **다대다**

### 엔티티 작성 방법

1. 엔티티의 관계를 정의하고, 관계에 따라 `@OneToOne`(일대일), `@OneToMany`(일대다), `@ManyToOne`(다대일), `@ManyToMany`(다대다) 어노테이션을 넣는다.
2. `@JoinColumn` 어노테이션을 사용해 매핑할 외래키를 설정한다.

|속성|기능|기본값|
|------|---|---|
|name|매핑할 외래키의 이름 설정|[필드 명]_[참조하는 테이블의 기본 키 컬럼명]|
|referencedColumnName|외래키가 참조할 상대 테이블의 컬럼명|[참조하는 테이블의 기본 키 컬럼명]|
|foreignKey|외래키를 생성하면서 지정할 제약 조건||
    
### 단방향과 양방향 매핑

* 단방향 매핑

    * 만약 한 쪽 엔티티에서 다른 엔티티를 참조할 필요는 있는데, 다른 쪽에서는 그럴 필요가 없을 때 맺음.

    * Eg) 게시글과 회원 관계를 생각해보자.
    
    ```java
    @Entity
    public class Order { // 주문 정보과 주문 상품의 경우

        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long orderId;

        @OneToMany
        private List<Product> products = new List<>(); // 주문 정보에서는 주문 상품을 조회할 필요는 있지만,
    }

    // -----

    @Entity
    public class Product { // 상품에선 주문을 몰라도 된다.

        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long productId;

        @Column(nullable = false)
        private Integer price;
    }
    ```


* 양방향 매핑

    * 단방향과 반대로, 서로가 서로를 참조할 수 있어야 할 경우 맺음.

    * Eg) 게시글과 댓글의 관계를 생각해보자.

    ```java
    @Entity
    public class Post { // 게시글과 회원의 경우

        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long postId;

        @OneToMany
        private User user; // 게시글을 어떤 회원이 작성했는지도 알아야 하지만,
    }

    // -----

    @Entity
    public class User { 

        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long userId;

        @ManyToOne
        private List<Post> posts = new ArrayList<>(); // 회원이 어떤 글을 작성했는지도 알아야 한다.
    }
    ```

### 일대일 매핑

    * 다른 엔티티 객체를 필드로 정의했을 때

### 다대일, 일대다 매핑

### 다대다 매핑

* Eg) 회원은 여러 게시글을 좋아요 할 수 있고, 게시글도 여러 회원에게 좋아요 받을 수 있다.

```java
@Entity
public class Post {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long postId;

    // ...

    @ManyToMany
    private List<User> likedUsers = new ArrayList<>();
}

// -----

@Entity
public class User { 

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long userId;

    // ...

    @ManyToMany
    private List<Post> likedPosts = new ArrayList<>();
}
```

* 위처럼 다대다(M:N) 연관관계는 어노테이션 `@ManyToMany`로 구현 가능하나, 실무에서 거의 사용되지 않는 구조이다. 이유는 다음과 같다:

    관계형 데이터베이스는 정규화된 테이블 2개로 다대다를 표현할 수 없기 때문에 중간 테이블을 생성해주긴 하지만 묵시적으로 생성해주기 때문에,
    1. 자기도 모르는 복잡한 조인의 쿼리가 발생하는 경우가 생길 수 있고,
    2. 필요한 추가 컬럼을 사용할 수 없다.

* 따라서, 이 중간 테이블을 엔티티로 새로 만들어 일대다, 다대일 관계로 풀어 직접 사용하는 것이 보통이다.

```java
@Entity
public class Post {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long postId;
}
// -----
@Entity
public class User { 

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long userId;
}
// -----
@Entity
public class PostLike {
    @Id 
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long likeId;

    @ManyToOne
    private User user;

    @ManyToOne
    private Post post;

    private LocalDateTime likedTime; // 필요한 메타데이터도 추가 가능
}
```

## 영속성 전이

* 영속성 전이란?

    * 영속성 전이란, 특정 엔티티의 영속성 상태를 변경할 때, 그 엔티티와 연관된 엔티티의 영속성에도 영향을 미쳐 영속성 상태를 같이 변경하는 것을 의미한다.

* 영속성 전이 사용 방법

`@OneToOne` 과 같은 연관관계를 매핑한 후, `cascade = `요소를 추가해주면 된다. 사용 가능한 전이 타입은 다음과 같다:


|종류|설명|
|----|---|
|PERSIST|영속화할 때 같이 영속화|
|MERGE|영속성 컨텍스트에 변합할 때 같이 병합|
|REMOVE|엔티티를 제거할 때 같이 제거|
|REFRESH|엔티티를 새로고침할 때 같이 새로고침|
|DETACH|영속성 컨텍스트에서 제외하면 같이 제외|
|ALL|위의 모든 상태 변경을 같이 적용|


```java
@Entity
public class Post { // 게시글과 댓글의 경우

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long postId;

    @OneToMany(cascade = CascadeType.ALL) // 게시글이 삭제되면 댓글도 같이 삭제되어야 하지만,
    private List<Comment> comments = new ArrayList<>();
}

// -----

@Entity
public class Comment { 

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long userId;

    @ManyToOne // 댓글이 삭제되어도 게시글은 여전히 존재해야 한다.
    private Post post; 
}
```