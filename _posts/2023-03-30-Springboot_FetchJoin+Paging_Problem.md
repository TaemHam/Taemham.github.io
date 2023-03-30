---
title: "[Springboot] 페이징 + Fetch Join 쿼리로 OneToMany 할 때, 모든 엔티티를 불러오는 문제를 해결해보자"
author: TaemHam
date: 2023-03-30 12:00:00 +0900
categories: [Springboot, DB]
tags: [CS, DB, N+1, Fetch Join, OneToMany]
---

## 문제 상황

댓글이 일대다 매핑 된 게시글들을 페이지 단위로 불러오는 쿼리를 실행하면 OOM(메모리 초과) 문제가 발생하는 것을 확인했다.

문제가 되는 엔티티와 쿼리문은 다음과 같다.

<details>
<summary>게시글 엔티티</summary>

    ```java
    @Builder
    @Getter
    @AllArgsConstructor
    @NoArgsConstructor
    @Entity
    public class Post {

        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private long id;

        @Column
        private String title;

        @Column
        private String content;

        @OneToMany(mappedBy = "post")
        private final List<Comment> comments = new ArrayList<>();

        public void addComment(Comment comment) {
            comments.add(comment);
        }
    }
    ```

</details>


<details>
<summary>댓글 엔티티</summary>

    ```java
    @Builder
    @Getter
    @AllArgsConstructor
    @NoArgsConstructor
    @Entity
    public class Comment {

        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private long id;

        @Column
        private String content;

        @ManyToOne
        @JoinColumn(name = "post_id")
        private Post post;
    }
    ```

</details>


<details>
<summary>게시글 리포지토리</summary>

    ```java
    public interface PostRepository extends JpaRepository<Post, Long> {

        @Query(value = "SELECT DISTINCT p FROM Post p JOIN FETCH p.comments WHERE p.content LIKE %:content%",
                countQuery = "SELECT COUNT(DISTINCT p) FROM Post p INNER JOIN p.comments WHERE p.content LIKE %:content%")
        Page<Post> findByContentWithFetchJoin(String content, Pageable pageable);
    }
    ```

</details>


## 문제 원인

문제는 아래의 상황에서 발생한다고 한다.

1. OneToMany 매핑이 있는 엔티티
2. N+1 문제의 해결 방법인 Fetch Join을 사용 
3. 페이징 처리를 위해 Pageable 인터페이스를 사용

이유는 다음과 같다.

컬렉션을 조인할 때 생기는 같은 식별자의 엔티티들을 distinct 쿼리로 제거해주면, 불러온 엔티티의 개수가 달라져 페이지에 담을 엔티티가 부족해지게 된다.

이런 상황이 발생하는 걸 막고자, JPA는 모든 엔티티를 불러와서 중복 제거를 하고, 그 후 페이지에 필요한 만큼 잘라내어 준다.

이렇게 모든 엔티티를 불러오는 과정때문에 메모리 초과 문제가 발생한 것.

출력된 쿼리문을 찾아보니, 엔티티를 모두 메모리에 올린다는 로그와 함께, 쿼리문에는 limit 같은 제한 조건이 걸려있지 않은 걸 확인했다.

![쿼리로그1](https://user-images.githubusercontent.com/95671168/228720611-256372e9-6318-4d7a-9b6d-1c3258e09a9b.PNG)

## 해결 방법

Fetch Join이 아닌 Batch Size 를 설정해 N+1 문제를 해결하고, 쿼리문에서는 Fetch Join을 지워주면 된다.

<details>
<summary>application.yaml</summary>

    ```yml
    spring:
    jpa:
        properties:
        hibernate:
            default_batch_fetch_size: 1000
    ```

</details>

<details>
<summary>게시글 리포지토리</summary>

    ```java
    public interface PostRepository extends JpaRepository<Post, Long> {

        @Query(value = "SELECT p FROM Post p WHERE p.content LIKE %:content%",
                countQuery = "SELECT COUNT(p) FROM Post p WHERE p.content LIKE %:content%")
        Page<Post> findByContentLike(String content, Pageable pageable);
    }
    ```

</details>

위와 같이 수정 한 후 쿼리문을 찾아보니, WARN 로그 없이 `fetch first ? rows only` 가 출력되는 것을 확인했다.

![쿼리로그2](https://user-images.githubusercontent.com/95671168/228720614-45cf2f19-55c4-4c9e-81da-bdd41f801b1a.PNG)

## 참고 자료

* [The best way to fix the Hibernate “firstResult/maxResults specified with collection fetch; applying in memory!” warning message](https://vladmihalcea.com/fix-hibernate-hhh000104-entity-fetch-pagination-warning-message/)
* [JPA Fetch 조인(join)과 페이징(paging) 처리](https://junhyunny.github.io/spring-boot/jpa/jpa-fetch-join-paging-problem/)
* [JPA에서 Fetch Join과 Pagination을 함께 사용할때 주의하자](https://tecoble.techcourse.co.kr/post/2020-10-21-jpa-fetch-join-paging/)