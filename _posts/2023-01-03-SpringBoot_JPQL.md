---
title: "[백엔드|스프링부트] JPQL과 JPA Repository 쿼리 메서드"
author: TaemHam
date: 2023-01-03 10:00:00 +0900
categories: [Backend, SpringBoot]
tags: [Tutorial, Backend, Spring, SpringBoot, Java, JPQL]
---

<img src="https://cdn.educba.com/academy/wp-content/uploads/2020/02/JPQL.jpg.webp"/>

---

## JPQL

* JPQL이란?

    * JPA에서 사용할 수 있는 쿼리를 말한다.

    * String으로 쿼리문을 작성, EntityManager을 통해 쿼리 실행한다.
    ```java
    String jpql = "select product "
            + "from Product product "
            + "where product.name = '펜' "
            + "order by product.price asc, product.stock desc";
    ```

    * SQL은 데이터베이스 테이블을 대상으로 쿼리하는데 반해, JPQL은 엔티티 객체를 대상으로 쿼리를 실행한다.

* EntityManager란?

    * EntityManager란, 엔티티를 저장하고, 수정하고, 삭제하고 조회하는 등 엔티티와 관련된 모든 일을 처리하는 Bean.

    * 스프링부트는 EntityManager가 자동으로 등록되어있어서, 아래와 같이 간단하게 불러올 수 있.
    ```java
    @PersistenceContext
    private EntityManager em;
    ```

    * 작성한 JPQL 문자열을 다음과 같이 메소드를 호출해 쿼리를 실행할 수 있다.
    ```java
    List<Product> jpqlResult = em.createQuery(jpql, Product.class).getResultList();
    ```

## JpaRepository의 쿼리 메서드

리포지토리는 JPA Repository를 상속 받는 것만으로 기본적인 CRUD 메서드를 제공하지만, 이런 메서드들은 PK 기반으로 생성되기 때문에, 다른 조건으로 쿼리하려면 별도의 메서드를 정의해서 사용해야 한다. 이 때 간단한 쿼리문을 작성하기 위해 사용되는 것이 쿼리 메서드이다.

쿼리 메서드는 다음과 같은 키워드들을 조합해 생성할 수 있다.

```java

public interface ProductRepository extends JpaRepository<Product, Long> {

    //  ----- 주제 키워드 -----
    // 무엇을 찾을 지 정함. 반환 타입이 달라짐.

    /*
    조회
    find / read / get / query / search / stream
    */
    List<Product> findByName(String name, Sort sort);

    /*
    개수 제한 (단건 조회 가능)
    First<number> / Top<number>
     */
    List<Product> findFirst5ByName(String name, Sort sort);
    Product findFirstProduct(String name); // Sort를 createdAt 기준으로 주면, 가장 최근에 생성된 데이터 조회 가능.

    /*
    존재 유무
    exists
    */
    boolean existsProductByPrice(Integer price);

    /*
    개수
    count
    */
    Integer countProductByStock(Integer stock);

    /*
    삭제 (삭제된 갯수 반환 가능)
    delete / remove
     */
    void deleteProductByName(String name);

    // ----- 조건자 키워드 -----
    // 모든 조건자 앞에 Is를 넣을 수도 있다.

    /*
    일치
    (생략) / Equals
    */
    List<Product> findByProductId(Long productId, Sort sort);

    /*
    불일치
    Not
    */
    List<Product> findProductByProductIdNot(Long productId, Sort sort);

    /*
    일부 일치
    StartingWith / EndingWith / Containing / Like
    */
    List<Product> findProductByNameStartingWith(String name, Sort sort);

    /*
    null 확인
    Null / NotNull
    */
    List<Product> findProductByNameNull(String name, Sort sort);

    /*
    boolean 확인
    True / False
    */

//    List<Product> find___ByTrue(String name, Sort sort); // boolean 컬럼이 없기 때문에 주석처리

    /*
    대소 비교
    GreaterThan / LessThan / Between
    뒤 Equal 추가로 경곗값 포함 가능
    */
    List<Product> findProductByPriceGreaterThan(Integer price);

    /*
    전후 비교 (시간)
    After / Before
     */
    List<Product> findByCreatedAtBefore(LocalDateTime time);

}

```