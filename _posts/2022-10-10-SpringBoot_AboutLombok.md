---
title: "[백엔드|스프링부트] Lombok으로 코드 다이어트"
author: TaemHam
date: 2022-10-10 19:00:00 +0900
categories: [Backend, SpringBoot]
tags: [Tutorial, Backend, Spring, SpringBoot, Java, Lombok]
---

<img src="https://user-images.githubusercontent.com/95671168/194983085-f56b0018-8547-4bc9-bc87-68ce91f0d4f5.png" width="20%" height="20%"/><em>Lombok 로고[^1]</em>

자바로 DTO나 Entity 같은 객체를 만들면, Getter/Setter나 toString 등 비즈니스 로직이 아님에도 코드가 길어지도록 하는 메서드들이 있다. 이런 반복되는 코드들을 없애고, 어노테이션 하나로 간단하게 만들 수 있도록 도와주는 플러그인이 바로 이번에 소개할 롬복 이다. 

이렇게 편리한 기능을 제공해주는 롬복을 프로젝트에 어떻게 적용하고 사용할 수 있는지, 또 주의사항에는 어떤 것이 있는지 알아보도록 하자.

## 적용 방법
***

프로젝트에 Lombok을 적용 시키는 방법은 다음과 같다.

1. 의존성 추가
2. 어노테이션 프로세서 설정
3. 어노테이션 추가

### 예제 환경

* IntelliJ IDEA
* Spring boot 2.5.6
* Maven

### 의존성 추가

IntelliJ 환경이라면 프로젝트 생성 단계에서 다음의 그림과 같이 Lombok을 포함시킬 수 있다.

<img src="https://user-images.githubusercontent.com/95671168/194991720-98a0b9ee-b690-45ed-8274-7a8373b27d5a.PNG"/><em>IntelliJ에서 Lombok 의존성 추가 방법</em>

그렇지 않더라도 `pom.xml`에 롬복 의존성을 추가해주면, 다른 설정 없이 바로 사용 가능하다.

```xml
  <dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
  </dependency>
```

### 어노테이션 프로세서 설정

다음은 어노테이션 설정이다. `Ctrl + Alt + S`로 Setting 창을 연 후, 다음과 같이 진입한다.

* Build, Execution, Deployment
  * Compiler
    * Annotation Processors

이 후 Enable annotation processing 이라는 항목에 체크하고, OK를 눌러 적용시킨다.

<img src="https://user-images.githubusercontent.com/95671168/194993243-973e25db-f1b4-4bd3-9801-88d042815635.PNG"/><em>IntelliJ에서 어노테이션 프로세서 설정 방법</em>

### 어노테이션 추가

마지막으로 롬복에서 제공하는 어노테이션을 DTO나 Entity 클래스에 추가하면 된다. 롬복에서 제공하는 기능은 다음과 같다.

#### @Getter / @Setter

클래스에 선언돼 있는 필드에 대한 getter나 setter 메서드를 생성해준다. 아래의 코드에서 주석 처리 된 부분들이 Getter와 Setter 부분이다.

```java
@Getter
@Setter
public class ProductDto {

    private String name;
    private int price;
    private int stock;

    public ProductDto (String name, int price, int stock) {
        this.name = name;
        this.price = price;
        this.stock = stock;
    }

// 아래의 주석 처리된 코드와 같다.
//    public String getName() {
//        return this.name;
//    }
//
//    public int getPrice() {
//        return this.price;
//    }
//
//    public int getStock() {
//        return this.stock;
//    }
//
//    public void setName(String name) {
//        this.name = name;
//    }
//
//    public void setPrice(int price) {
//        this.price = price;
//    }
//
//    public void setStock(int stock) {
//        this.stock = stock;
//    }
}
```

만약 id 처럼 한 번 지정하고 나면 바뀌면 안되어서 Setter 메서드가 없어야하는 필드는 final 키워드를 붙이거나 AccessLevel.None을 붙여주면 된다.

```java

@Getter
@Setter
public class ProductDto {

    @Setter(AccessLevel.None) // 이걸 설정하거나
    private final Long id;    // final을 붙이거나
    private String name;
    private int price;
    private int stock;

    public ProductDto (Long id, String name, int price, int stock) {
        this.id = id;
        this.name = name;
        this.price = price;
        this.stock = stock;
    }

```

#### @NoArgsConstructor / @RequiredArgsConstructor / @AllArgsConstructor

데이터 클래스의 초기화를 위한 생성자를 자동으로 만들어주는 어노테이션들이다.

* `@NoArgsConstructor`: 매개변수가 없는 생성자를 생성한다. 필드에 final 키워드가 붙어 있다면 초기화할 수 없기 때문에 오류가 난다.

  ```java
  @NoArgsConstructor
  public class ProductDto {

      private final Long id;    // 이것 때문에 오류가 난다.
      @NonNull
      private String name;
      private int price;
      private int stock;

  //     아래의 주석 처리된 코드와 같다.
  //     public ProductDto () {
  //     }
  }
  ```

* `@RequiredArgsConstructor`: final 이나 @NonNull 을 가지는 필드를 매개변수로 받는 생성자를 생성한다.

  ```java
  @RequiredArgsConstructor
  public class ProductDto {

      private final Long id;
      @NonNull
      private String name;
      private int price;
      private int stock;

  //     아래의 주석 처리된 코드와 같다.
  //     public ProductDto(Long id, @NonNull String name) {
  //         if (name == Null) {
  //             throw new NullPointerException("name is marked NonNull but is null.");
  //         } else {
  //             this.id = id;
  //             this.name = name;
  //         }
  //     }
  }
  ```

* `@AllArgsConstructor`: 모든 필드를 매개변수로 받는 생성자를 생성한다.

  ```java
  @AllArgsConstructor
  public class ProductDto {

      private final Long id;
      @NonNull
      private String name;
      private int price;
      private int stock;

  //     아래의 주석 처리된 코드와 같다.
  //     public ProductDto(Long id, @NonNull String name, int price, int stock) {
  //         if (name == Null) {
  //             throw new NullPointerException("name is marked NonNull but is null.");
  //         } else {
  //             this.id = id;
  //             this.name = name;
  //             this.price = price;
  //             this.stock = stock;
  //         }
  //     }
  }
  ```

#### @ToString

필드의 값을 문자열로 조합해 리턴하는 `.toString()`메서드를 생성하는 어노테이션이다. 

```java
@ToString
public class ProductDto {

    private String name;
    private int price;
    private int stock;

//     아래의 주석 처리된 코드와 같다.
//     public String toString() {
//          return "ProductDto(name=" + this.name + ", price=" + this.price + ", stock=" + this.stock + ")";
//     }
}
```

만약 `@ToString`으로 표현하고 싶지 않은 필드가 있으면, exclude 속성으로 생성에서 제외할 수 있다.

```java
@ToString(exclude = "stock")
public class ProductDto {

    private String name;
    private int price;
    private int stock;

//     아래의 주석 처리된 코드와 같다.
//     public String toString() {
//          return "ProductDto(name=" + this.name + ", price=" + this.price + " + ")";
//     }
}
```

@ EqualsAndHashCode

객체의 동등성과 동일성을 비교하는 연산 메서드를 생성한다. 이름에 걸맞게 하나의 어노테이션으로 `.equals()` 와 `.hashCode()` 메서드 둘 모두 생성해준다.

두 메서드는 각각 다음과 같은 연산을 수행한다.
* equals: 두 객체의 내용이 같은지 동등성을 비교한다.
* hashCode: 두 객체가 같은 객체인지 동일성을 비교한다.

하지만 이렇게 설명해서는 도통 무슨 말인지 모를 것 같다. 다음의 예시 코드를 보자. 여기서 ProductDto와 ProductDto2 는 동일한 코드의 다른 클래스다.

```java
public class EqualsAndHashCodeTester {

    public static void main(String[] args) {

        ProductDto prod1 = new ProductDto("빵", 1000, 10);
        ProductDto prod2 = new ProductDto("빵", 1000, 10);
        ProductDto2 prod3 = new ProductDto2("빵", 1000, 10);

        System.out.println(prod1 == prod2);

        System.out.println(prod1.equals(prod2));
        System.out.println(prod1.hashCode() == prod2.hashCode());

        System.out.println(prod1.equals(prod3));
        System.out.println(prod1.hashCode() == prod3.hashCode());
    }
}
```

prod1 과 prod2는 클래스와 내용은 같지만, 다른 주솟값을 갖는 다른 인스턴스이고,

prod1 과 prod3은 내용만 같고 클래스도, 주솟값도 다른 인스턴스이다.

이러한 두 쌍에 대해 equals 와 hashCode 메서드로 비교하면 어떤 결과가 나올까?

위의 코드를 실행하면 다음과 같은 결과를 얻는다.

<img src="https://user-images.githubusercontent.com/95671168/195016275-7c28e507-3151-486e-ab2a-bf40a97e01db.PNG"/><em>EqualsAndHashCode 실험 결과</em>

첫 줄의 결과는 prod1과 prod2가 **다른 주솟값을 가지는 다른 인스턴스**이기 때문에 false가 나왔다.

다음 두 줄의 결과는 prod1과 prod2가 다른 인스턴스 일지라도, **같은 클래스로 생성된 같은 내용을 가진 인스턴스**이기 때문에 모두 true가 나왔다.

마지막 두 줄은 prod1과 prod3가 **다른 클래스로 생성된 인스턴스** 이므로 equals의 결과가 false가 나왔지만, **두 클래스의 내용이 같으므로** hashCode의 결과는 true가 나왔다.

즉, 객체의 동일성을 비교해야하는 `hashCode`가, 서로 다른 객체임에도 true를 반환하고 있다. 이로 인해 생기는 버그들이 있지만, 이는 후에 유의사항에서 알아보도록 하자.

#### @ Data

앞서 설명한 네 개의 어노테이션 모두를 포괄하는(생성자의 경우는 Required로) 어노테이션이다. 만약 모두 사용하고 싶으면 간단히 `@Data`만 붙여주면 되고, 그 중 사용하고 싶지 않은 것이 있다면 AccessLevel 을 설정해주면 된다.

```java
@Data
public class ProductDto {

    @Setter(AccessLevel.None)
    private Long id;
    private String name;
    private int price;
    private int stock;
```

#### @Value

위의 `@Data`와 비슷하지만, `@Setter`를 빼고 필드를 final로 정의한, 즉 불변 클래스인 VO(Value Object)를 만들 때 사용하는 어노테이션이다.

```java
@Value
public class ProductVo {

    private Long id;
    private String name;
    private int price;
    private int stock;
```

#### @Builder

생성자 대신 빌더 패턴으로도 필드를 정의할 수 있게 해주는 코드를 생성하는 어노테이션이다.

```java

@Builder
public class ProductDto {

    private String name;
    private int price;
    private int stock;

//     ProductDto(String name, int price, int stock) {
//         this.name = name;
//         this.price = price;
//         this.stock = stock;
//     }
// 
//     public static ProductDtoBuilder builder() {
//         return new ProductDtoBuilder();
//     }
// 
//     public static class ProductDtoBuilder {
//         private String name;
//         private int price;
//         private int stock;
// 
//         ProductDtoBuilder() {
//         }
// 
//         public ProductDtoBuilder name(String name) {
//             this.name = name;
//             return this;
//         }
// 
//         public ProductDtoBuilder price(int price) {
//             this.price = price;
//             return this;
//         }
// 
//         public ProductDtoBuilder stock(int stock) {
//             this.stock = stock;
//             return this;
//         }
// 
//         public ProductDto build() {
//             return new ProductDto(name, price, stock);
//         }
// 
//         public String toString() {
//             return "ProductDto.ProductDtoBuilder(name=" + this.name + ", price=" + this.price + ", stock=" + this.stock + ")";
//         }
//     }
}
```


## 유의 사항
***

Lombok 은 편리한 기능을 많이 제공하지만, 때때로는 그 편리함이 독이 될 수 있다. [^2]

### @AllArgsConstructor, @RequiredArgsConstructor 관련

두 어노테이션은 필드가 정의된 순서대로 매개변수로 받도록 생성자를 만든다. 즉, 필드를 추가나 제거 하면서 필드의 순서가 뒤바뀌게 되는 경우가 생기면, 그대로 뒤바뀐 채로 생성하게 된다. 심지어 만약 그 뒤바뀐 필드가 같은 타입이면 오류로 멈추지도 않아 더욱 큰일이 나게 되는 것이다. 다음의 코드를 보자.

```java

// DTO 클래스

@AllArgsConstructor
@ToString
public class ProductDto {

    private String name;
//  private int price; // 여기서 stock과 price의 위치가 바뀌었다고 하자.
    private int stock; 
    private int price;
}

// DTO를 생성하는 클래스

class Tester {
    @Test
    void test() {

        ProductDto prod1 = new ProductDto("빵", 1000, 10);

        System.out.println(prod1.toString()); // ProductDto(name=빵, stock=1000, price=10)
    }
}

```

원래는 (name, price, stock) 순서로 들어갔다고 생각하던게, (name, stock, price)로 들어가서 10원짜리 빵 1000개가 생겨버리는 불상사가 생겨버렸다.

이런 오류를 범하지 않기 위해서는
1. @AllArgsConstructor 와 @RequiredArgsConstructor를 처음부터 사용을 하지 않거나,
2. @Builder 를 사용해 파라미터 값이 잘못 들어가지 않도록 코딩하거나,
하는 방법을 사용하면 된다.

### @EqualsAndHashCode 관련

위에 설명했던 대로, HashCode가 값들만 비교하기 때문에 에러가 생긴다. 

```java
@SpringBootTest
class AppTest {

    private ProductDto prod1 = new ProductDto("빵", 1000, 10);

    @Test
    void test() {
        Set<ProductDto> prodSet = new HashSet<>();
        prodSet.add(prod1);

        boolean contains1 = prodSet.contains(prod1);
        System.out.println("contains1 = " + contains1); // true

        prod1.setStock(9);
        boolean contains2 = prodSet.contains(prod1);
        System.out.println("contains2 = " + contains2); // false
    }
}
```

위의 코드는 prod1을 해시셋에 넣고 해시셋에 존재하는지 확인, prod1의 필드값 하나만 바꾸고 다시 해시셋에 존재하는지 확인하는 코드이다.

앞에서 비교한 prod1과 뒤에서 비교한 prod1은 분명히 같은 객체이지만, 필드가 바뀌어 다른 해시값을 만들기 때문에 서로 다른 버켓을 참조하게 되면서, 해시셋 안에 존재하지 않는 것처럼 판별하는 것이다. 만약 `@EqualsAndHashCode` 가 없었다면 Object에 기본으로 구현된 `.hashCode()`가 주솟값으로 해시값을 만들어 내, 같은 버켓을 참조하게 만들었을 것이다.

이런 오류를 범하지 않기 위해서는
1. @EqualsAndHashCode 를 사용하지 않거나,
2. @EqualsAndHashCode(of={"[필드]"}) 로 고유한 값을 가지는 필드(ID 등)만으로 해시값을 만들도록 하거나,
하는 방법을 사용하면 된다.

### @ToString 관련

A 라는 객체의 멤버로 B 라는 객체가 있고, B 라는 객체의 멤버로 A 라는 객체가 있다면, 둘은 무한으로 순환하여 참조할 수 있다.

이 경우, `@ToString(exclude = "[필드]")` 해당 객체를 가지는 필드를 참조하지 않게 해 예방하는 방법이 있다.


## 출처
***

[^1]: <https://projectlombok.org/>
[^2]: <https://jake-seo-dev.tistory.com/70>