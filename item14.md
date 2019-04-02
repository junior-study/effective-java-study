# 아이템 14. Comparable을 구현할지 고려하라

## Comparable 인터페이스

```Comparable```은 동치성 비교와 더불어 순서까지 비교하는 인터페이스이다. 인터페이스에는 ```compareTo``` 메서드가  정의되어 있다. Object 클래스의 equals와 성격이 비슷하지만 차이점으로는 <b>순서를 비교할 수 있고, 제네릭이다.</b> 

```java
public interface Comparable<T> {

    public int compareTo(T o);
}
```

Comparable 인터페이스를 구현한 클래스는 인스턴스들간에 자연적인 순서가 있음을 뜻한다. String 클래스는 Comparable 인터페이스를 구현하고 있기 때문에 다음과 같이 쉽게 정렬할 수 있다.

```java
String[] names = new String[3];
names[0] = "Jayden";
names[1] = "Kelly";
names[2] = "Andrew";

Arrays.sort(names);
```

자바 플랫폼 라이브러리에서 모든 값 클래스와 열거 타입은 Comaprable을 구현한다. 그리고 제네릭 하기 때문에 다양하게 사용할 수 있으며, 컬렉션 구현체와도 연동해서 사용할 수 있는 장점이 있다.

## compareTo 메서드 규약
compareTo 메서드 역할은 현재 객체와 주어진 객체의 순서를 비교한다. 객체를 비교하고 음의 정수, 0, 양의 정수를 반환한다. 비교할 수 없는 타입의 객체가 주어지면 ClassCastException 예외를 던진다.

- 현재 객체 < 주어진 객체 : 음의 정수
- 현재 객체 == 주어진 객체 : 0
- 현재 객체 > 주어진 객체 : 양의 정수

<br/>

compareTo 메서드의 일반 규약은 equals와 유사하다.

1. sgn(x.compareTo(y)) == -sgn(y.compareTo(x))
2. (x.compareTo(y) > 0 && y.compareTo(z) > 0)이면, x.compareTo(z) > 0
3. x.compareTo(y) == 0이면, sgn(x.compareTo(z)) == sgn(y.compareTo(z))
4. (x.compareTo(y) == 0) == (x.equals(y))

> 특정 컬렉션들이 동치성을 비교할 때, equals가 아닌 compareTo 메서드를 사용한다. 4번째 규약을 지키지 않으면 특정 컬렉션에서 예상치 않은 결과를 초래할 수 있기 때문에 주의해야 한다.

## compareTo 메서드 작성 방법
Comparable은 타입을 인수로 받는 제네릭 인터페이스이므로 compareTo 메서드의 인수 타입은 컴파일 타임에 결정된다. 이 점은 제네릭의 장점이다.

compareTo 메서드는 <b>인스턴스의 필드가 동치인지 비교하는 것이 아닌 순서를 비교한다.</b> 객체 참조를 비교하려면 compareTo 메서드를 재귀적으로 호출한다. 그리고 Comparable을 구현하지 않았거나 표준 순서가 아닌 방법으로 비교해야 한다면 ```Comparator```를 사용한다.

```java
public class User {
    private String name;

    public User(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                '}';
    }
}


List<User> users = Arrays.asList(new User("jayden"),
    new User("kelly"), 
    new User("andrew"));

users.stream().sorted(Comparator.comparing(User::getName))
        .forEach(System.out::println);
```

기존에는 기본 타입 비교시에 >, < 연산자를 많이 사용했다. 하지만, 이제는 박싱된 기본 타입 클래스에 compare 정적 메서드가 추가되었기 때문에 해당 메서드를 이용하도록 하자.

```java
int result = Integer.compare(5, 10);
System.out.println("Compare 5, 10 : " + result);
```

클래스에 필드가 많다면 어느 것을 먼저 비교하느냐에 따라 성능이 달라질 수 있다. 핵심적인 필드부터 비교하면서, 비교 결과 시에 동일하지 않다면 그 결과를 바로 반환하도록 하자.

## 정리
순서를 고려해야 하는 값 클래스라면 꼭 Comparable 인터페이스를 구현해야 한다. 기존에 구현된 컬렉션에서는 compareTo 메서드가 구현되어 있다고 생각하고 정렬, 검색, 비교 기능을 구현하고 있다. 필드의 값을 비교할 때는 박싱된 기본 타입 클래스가 제공하는 compare 메서드 또는 Comparator 인터페이스가 제공하는 기능을 사용하도록 하자.