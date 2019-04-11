# 아이템 2. 생성자에 매개변수가 많다면 빌더를 고려하라

```정적 팩터리```와 ```생성자```는 선택적 매개변수가 많을 경우에 적절하게 대응하게 어렵다는 제약이 있다.

특정 클래스에 선택 항목이 많은 경우가 있다. 예를 들어 식품 포장의 영양정보를 표현하는 클래스를 생각해보자.

영양정보 클래스 ```NutritionFacts```는 1회 내용량, 트랜스지방, 포화지방, 콜레스테롤 등 20개가 넘는 선택 항목을 갖고 있다. 대부분의 제품은 이 선택 항목 중 대다수의 값은 기본값인 0이다. 각각의 제품은 항목의 모든 값을 대부분 사용하지 않는다.

## 점층적 생성자 패턴
이러한 클래스의 경우 대부분의 개발자들이 ```점층적 생성자 패턴```(telescoping constructor pattern)을 즐겨 사용했다. 필수 매개변수만 받는 생성자를 기본으로 하고, 필수 매개변수와 선택 매개변수 1개를 받는 생성자, 필수 매개변수와 선택 매개변수 2개를 받는 생성자 등 점층적으로 선택 매개변수를 전부 받는 생성자를 만들어서 사용했다.

다음 코드가 점층적 생성자 패턴의 예제이다.

```java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }

    // 매개변수를 더 받을 수 있는 생성자 생략....
}
```

점층적 생성자 패턴을 사용한 클래스의 인스턴스를 생성하기 위해서는 필요하지 않은 매개변수 값을 넘겨야 하는 경우도 있다. <b>매개변수가 많아지면 클라이언트 코드를 작성하거나 읽기 어렵다.</b> 개발자가 실수로 동일한 타입의 매개변수의 순서를 바꿔 넣어도 컴파일러는 알아채지 못하고 런타임에 엉뚱한 동작을 할 가능성이 있으므로 주의해야 한다.

## 자바빈즈 패턴
매개변수가 없는 기본 생성자 객체를 먼저 만든 후, ```setter``` 메서드를 통해서 원하는 매개변수의 값을 설정하는 방식이다.

다음 코드가 자바 빈즈 패턴의 예제이다.

```java
NutritionFacts cocaCola = new NutritionFacts();
cocaCola.setServingSize(240);
cocaCola.setServings(8);
// 필요한 setter 메서드 호출 생략 ...
```

객체를 생성하는 방법이 간편해지고 필요한 매개변수를 setter 메서드를 호출하기 때문에 이전에 살펴본 점층적 생성자 패턴보다 읽기 쉬운 코드가 되었다.

자바빈즈 패턴에서는 심각한 단점이 있다. <b>객체 하나를 생성하기 위해서 여러 개 메서드를 호출해야 하며, 완전히 생성되기 전까지는 일관성이 무너진 상태에 놓이게 된다.</b>

자바빈즈 패턴에서는 클래스를 불변으로 만들 수 없으며 스레드 안정성을 얻으려면 개발자가 추가 작업을 해줘야 한다. 사용하고자 하는 완전한 객체를 만들기 위해서 setter 메서드를 호출하기 때문이다.

## 빌더 패턴
점층적 생성자 패턴의 안전성과 자바빈즈 패턴의 가독성을 겸비한 빌더 패턴이 있다. 빌더 객체를 통해서 필요한 매개변수 값을 설정하고 ```build``` 메서드를 호출해서 필요한 객체를 얻는다. 빌더 클래스는 생성하고자 하는 클래스 안에 정적 멤버 클래스로 만드는 경우가 많다.

다음 코드가 빌더 패턴의 예제이다.

```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
    .calories(100)
    .sodium(35)
    .carbohydrate(27)
    .build();
```

빌더의 setter 메서드들은 빌더 자신을 반환하기 때문에 연쇄적으로 호출할 수 있다. 메서드 호출이 흐르듯 연결된다는 뜻으로 플루언트 API(fluent API) 혹은 메서드 연쇄(method chaining)라 한다.

빌더 패턴을 사용해서 객체를 생성하면 개발자가 코드 쓰기도 쉽고 가독성도 높아진다. <b>빌더 패턴은 마치 파이썬과 스칼라에 있는 명명된 선택적 매개변수(named optional parameters)를 흉내 낸 것이다.</b>

Java 개발에서 많이 사용하는 ```Lombok```라이브러리가 있다. Lombok을 이용하면 어노테이션을 사용해서 빌더 패턴을 쉽게 사용할 수 있다.

다음 코드는 [Lombok을 사용해서 빌더 패턴](https://projectlombok.org/features/Builder)을 사용하는 예제이다.

```java
@Builder
public class BuilderExample {
    @Builer.Default
    private long created = System.currentTimeMillis();
    private String name;
    private int age;
}
```
