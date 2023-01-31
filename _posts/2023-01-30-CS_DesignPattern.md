---
title: "[CS] 디자인패턴이란? Part 01"
author: TaemHam
date: 2023-01-30 09:00:00 +0900
categories: [CS, Design Pattern]
tags: [CS, Design Pattern]
---

## 디자인 패턴이란?

디자인 패턴이란, **프로그램을 설계할 때 발생했던 문제점들을 객체 간의 상호 관계 등을 이용해 해결할 수 있도록 하나의 '규약' 형태로 만들어 놓은 것**을 의미한다. 

#### 종류

'GoF 디자인 패턴' 에 따라 분류하면 디자인 패턴은 3가지 종류로 나뉜다: 생성(Creational), 구조(Structural), 행위(Behavioral)

생성 패턴은 객체 생성에 사용되는 패턴으로, 의존성을 낮추도록(객체를 수정해도 호출부가 영향을 받지 않도록) 개발된 패턴들이다.
구조 패턴은 객체를 조합해서 더 큰 구조를 만드는 패턴이다.
행위 패턴은 객체 간의 알고리즘이나 책임 분배에 관한 패턴으로, 객체 하나로 수행할 수 없는 작업을 여러 객체를 이용해 작업을 분배하는 패턴이다.

### 싱글톤 패턴

#### 개념

**생성 패턴** 중 하나인 싱글톤 패턴(Singleton Pattern)은 **하나의 클래스에 오직 하나의 인스턴스만 가지는 패턴**이다. 인스턴스를 여러 개 만들게 되면 자원을 낭비하게 되거나 버그를 발생시킬 수 있는 모듈에 사용된다.

#### 사용

**DB 연결 모듈**, 커넥션 풀, 스레드 풀 등에 사용.

#### 구현

싱글톤 패턴을 구현하는 방법에는 여러가지가 있고, 그 중 가장 최적화된 패턴을 상황에 맞게 사용하는 게 가장 좋다.
이 글에서는 검증된 싱글톤 패턴 코드 두 가지를 다룰 것이다.

1. Bill Pugh (성능)
    * 멀티스레드 환경에서 안전하고 Lazy Loading(나중에 객체 생성)도 가능
    * 클래스 안에 내부 클래스를 두어 클래스가 로드되는 시점에 만들어지도록 함
    * 다만 클라이언트가 임의로 싱글톤을 파괴할 수 있다는 단점을 지님 (Reflection API, 직렬화/역직렬화를 통해)

```java
class Singleton {
 
    private Singleton() {}
 
    // static 내부 클래스를 이용
    // Holder로 만들어, 클래스가 메모리에 로드되지 않고 getInstance 메서드가 호출되어야 로드됨
    private static class SingleInstanceHolder {
        private static final Singleton INSTANCE = new Singleton();
    }
 
    public static Singleton getInstance() {
        return SingleInstanceHolder.INSTANCE;
    }
}
```

2. Enum 클래스
    * 초기화가 한 번만 이루어져 멀티스레드 환경에서 안전하고, 위의 싱글톤 파괴 공격에도 안전
    * 다시 일반적인 클래스로 변환해야 할 경우, 코드를 처음부터 다시 짜야 함
    * 클래스 상속이 불가

```java
enum SingletonEnum {
    INSTANCE;
 
    private final Client dbClient;
	
    SingletonEnum() {
        dbClient = Database.getClient();
    }
 
    public static SingletonEnum getInstance() {
        return INSTANCE;
    }
 
    public Client getClient() {
        return dbClient;
    }
}
 
public class Main {
    public static void main(String[] args) {
        SingletonEnum singleton = SingletonEnum.getInstance();
        singleton.getClient();
    }
}
```

#### 문제점

1. TDD 단위 테스트가 힘듦

싱글톤 패턴은 TDD(Test Driven Development)를 할 때 걸림돌이 된다. TDD를 할 때 주로 단위 테스트를 하는데, 단위 테스트는 테스트가 서로 독립적이어야 하며 테스트를 어떤 순서로든 실행할 수 있어야 한다.
하지만 싱글톤 패턴은 미리 생성된 하나의 인스턴스를 기반으로 구현하는 패턴이므로 각 테스트마다 '독립적인' 인스턴스를 만들기 어렵다.

2. 모듈간 의존성이 높아짐

싱글톤 패턴의 대부분은 인터페이스가 아닌 클래스의 객체를 미리 생성해 놓고 정적 메소드를 이용하는 방식을 사용하기 때문에, 클래스 사이에 강한 의존성과 높은 결합이 생기게 된다. 의존성이 높다는 것은, 하나의 모듈을 수정함으로 그 모듈을 참조하는 다른 모듈들도 수정이 필요하게 된다는 것이다.

이는 의존성 주입(DI, Dependency Injection)을 통해 모듈간의 결합을 조금 더 느슨하게 만들어 해결할 수 있다. 의존성 주입이란, 클래스가 필요한 객체를 클래스 외부에서 생성하여 클래스 내부로 파라미터를 통해 내부로 주입 시키는 것이다.

### 팩토리 패턴

#### 개념

**생성 패턴** 중 하나인 팩토리 패턴(Factory pattern)은 객체를 사용하는 코드에서 **객체 생성 부분을 떼어내 추상화한 패턴**으로, 상속 관계에 있는 두 클래스에서 **상위 클래스가 중요한 뼈대를 결정하고 하위 클래스에서 객체 생성에 관한 구체적인 내용을 결정하는 패턴**이다. 즉, 상위 클래스의 인스턴스를 하위 클래스가 생성하는 것이다.

#### 구현

커피 주문을 받아 각기 다른 커피를 생산하는 커피 팩토리 코드를 구축해보자.

```java
abstract class Coffee {
    public abstract int getPrice();

    @Override
    public String toString() {
        return this.getPrice() + "원 입니다."
    }
}

class CoffeeFactory { 
    public static Coffee getCoffee(String type, int price){
        if ("Latte".equalsIgnoreCase(type)) return new Latte(price);
        else if ("Americano".equalsIgnoreCase(type)) return new Americano(price);
        else {
            return new DefaultCoffee();
        } 
    }
}

class DefaultCoffee extends Coffee {
    private int price;

    public DefaultCoffee() {
        this.price = -1;
    }

    @Override
    public int getPrice() {
        return this.price;
    }
}

class Latte extends Coffee { 
    private int price; 
    
    public Latte(int price){
        this.price = price; 
    }

    @Override
    public int getPrice() {
        return this.price;
    } 
}

class Americano extends Coffee { 
    private int price; 
    
    public Americano(int price){
        this.price = price; 
    }

    @Override
    public int getPrice() {
        return this.price;
    } 
} 

public class CoffeeTest{ 
    public static void main(String [] args){ 
        Coffee latte = CoffeeFactory.getCoffee("Latte", 4000);
        Coffee americano = CoffeeFactory.getCoffee("Americano", 3000); 
        System.out.println("라떼는 " + latte);
        System.out.println("아메리카노는 " + americano); 
    }
} 
/*
라떼는 4000원 입니다.
아메리카노는 3000원 입니다.
*/
```

위 코드를 보면, 커피 객체를 메인 메서드가 아닌 CoffeeFactory 클래스에서 생성하는 것을 알 수 있다. 이러면 SOLID 원칙 중 의존성 역전 원칙(DIP, Dependency Inversion Principle)을 지킬 수 있다.

### 전략 패턴

#### 개념

행위 패턴 중 하나인 전략 패턴(Strategy pattern)은 실행(런타임) 중에 알고리즘 전략(기능, 동작)을 선택하여 객체 동작을 실시간으로 바뀌도록 할 수 있게 하는 행위 디자인 패턴이다. 같은 문제를 해결하는 여러 알고리즘을 클래스별로 캡슐화 해 놓고, 이들이 필요할 때 교체할 수 있도록 하는 방식으로 구현한다.

#### 사용

**인증 모듈**, 결제 시스템

#### 구현

다음은 쇼핑 카트에 아이템을 담아 LUNA 신용카드 또는 KAKAO 신용카드라는 두 개의 전략을 이용해 상황에 따라 결제를 진행하는 예제이다.

```java
class Item {
    public String name;
    public int price;
 
    public Item(String name, int cost) {
        this.name = name;
        this.price = cost;
    }
}

// 전략 - 추상화된 알고리즘
interface PaymentStrategy {
    void pay(int amount);
}
 
class KAKAOCardStrategy implements PaymentStrategy {
    private String name;
    private String cardNumber;
    private String cvv;
    private String dateOfExpiry;
 
    public KAKAOCardStrategy(String nm, String ccNum, String cvv, String expiryDate) {
        this.name = nm;
        this.cardNumber = ccNum;
        this.cvv = cvv;
        this.dateOfExpiry = expiryDate;
    }
 
    @Override
    public void pay(int amount) {
        System.out.println("카카오 카드를 이용해 " + amount + "원을 결제했습니다.");
    }
}
 
class LUNACardStrategy implements PaymentStrategy {
    private String emailId;
    private String password;
 
    public LUNACardStrategy(String email, String pwd) {
        this.emailId = email;
        this.password = pwd;
    }
 
    @Override
    public void pay(int amount) {
        System.out.println("루나 카드를 이용해 " + amount + "원을 결제했습니다.");
    }
}
 
// 컨텍스트 - 전략을 등록하고 실행
class ShoppingCart {
    List<Item> items;
 
    public ShoppingCart() {
        this.items = new ArrayList<Item>();
    }
 
    public void addItem(Item item) {
        this.items.add(item);
    }
	
    // 전략을 매개변수로 받아서 실행
    public void pay(PaymentStrategy paymentMethod) {
        int amount = 0;
        for (Item item : items) {
            amount += item.price;
        }
        paymentMethod.pay(amount);
    }
}
// 클라이언트 - 전략 제공/설정
class User {

    private final ShoppingCart cart = new ShoppingCart();

    // 물건 담기
    public void choose(Item item) {
        cart.addItem(item);
    }

    // 결제 전략 실행
    public void pay(PaymentStrategy paymentMethod) {
        cart.pay(paymentMethod); 
    }
}
```

예제 코드를 보면 알 수 있듯이, 전략 패턴을 이용하면 카드사 정책에 따라 필요한 정보를 다르게 받을 수 있다. 또, 만약 다른 카드를 추가하고 싶으면 `PaymentStrategy`를 상속받아 쉽게 추가 가능하다.

### 옵저버 패턴

#### 개념

옵저버 패턴(Observer Pattern)은 특정 주체가 다른 객체의 상태 변화를 관찰하다가, **상태 변화가 생기면 옵저버들에게 변화를 알려주는 디자인 패턴**이다. 

#### 사용

알림 기능, MVC 패턴

#### 구현

```java
// 주체의 기능 정의
public interface Subject {
    public void register(Observer obj);
    public void unregister(Observer obj);
    public void notify();
    public Object getUpdate();
}

// 객체의 기능 정의
public interface Observer {
    public void update();
}

// 특정 주제에 대한 옵저버들을 관리
class Topic implements Subject {

    private List<Observer> observers;
    private String message;

    public Topic() {
        this.observers = new ArrayList<>();
        this.message = "";
    }

    // 옵저버 추가
    @Override
    public void register(Observer obj) {
        if (!observers.contains(obj)) {
            observers.add(obj);
        }
    }

    // 옵저버 제거
    @Override
    public void unregister(Observer obj) {
        observers.remove(obj);
    }

    // 옵저버에게 알림
    @Override
    public void notify() {
        this.observers.forEach(Observer::update);
    }

    // 업데이트 내역 받기
    @Override
    public String getUpdate() {
        return this.message;
    }

    // 업데이트 하기
    public void postMessage(String msg) {
        this.message = msg;
        notify();
    }
}

// 
class TopicSubscriber implements Observer {
    private String name;
    private Subject topic;

    public TopicSubscriber(String name, Subject topic) {
        this.name = name;
        this.topic = topic;
    }

    // 위의 notify 에서 호출되면, topic으로부터 업데이트 내역을 받는다.
    @Override
    public void update() {
        String message = topic.getUpdate();
        System.out.println(name + " 유저가 받은 메시지: " + message)
    }
}

public class Test {
    public static void main(String[] args) {
        Topic topic = new Topic();
        Observer a = new TopicSubscriber("A", topic);
        Observer b = new TopicSubscriber("B", topic);
        Observer c = new TopicSubscriber("C", topic);

        topic.register(a);
        topic.register(b);
        topic.register(c);

        topic.postMessage("Hello world!")
    }
}
```

특정 Topic에 대한 업데이트를 감지하고 전달하는 옵저버 패턴이다. `class Topic implements Subject` 를 통해 Subject interface를 구현했고, `Observer a = new TopicSubscriber("A", topic);` 으로 옵저버를 선언할 때 해당 이름과 어떤 토픽의 옵저버가 될 것인지 정했다.

## 참고 자료

***
* [[기술 면접 질문] 기술 면접 예상 질문 대비하기 - 디자인패턴(Design Pattern)편](https://gmlwjd9405.github.io/2017/10/01/basic-concepts-of-development-designpattern.html)
* [[Design Pattern] 싱글톤 패턴](https://devjeong.com/design%20pattern/DesignPattern-1.1.1/)
* [디자인패턴 - 팩토리 패턴 (factory pattern)](https://jusungpark.tistory.com/14)
* [[GOF] 💠 전략(Strategy) 패턴 - 제대로 배워보자](https://inpa.tistory.com/entry/GOF-%F0%9F%92%A0-%EC%A0%84%EB%9E%B5Strategy-%ED%8C%A8%ED%84%B4-%EC%A0%9C%EB%8C%80%EB%A1%9C-%EB%B0%B0%EC%9B%8C%EB%B3%B4%EC%9E%90)