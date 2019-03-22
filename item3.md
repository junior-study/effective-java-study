# 아이템 3. private 생성자나 열거 타입으로 싱글턴임을 보증하라

```싱글턴(singleton)```이란 <b>인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다.</b> 싱글턴은 함수와 같은 무상태(stateless) 객체나 설계상 유일해야 하는 시스템 컴포넌트에서 사용할 수 있다.

<b>클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기가 어려울 수 있다.</b> 인터페이스를 정의하고 구현하는 경우가 아니라면 mock 구현으로 교체하기가 어렵기 때문이다.

싱글턴을 만들 때 생성자는 ```private```으로 감춰두고, 유일한 인스턴스에 접근할 수 있는 수단으로 ```public static``` 멤버를 제공한다.

## public static final 필드 방식
private 생성자는 public static final 필드인 인스턴스를 초기화할 때 딱 한번만 호출된다. public 또는 protected 생성자가 없으므로 Elvis 클래스가 초기화될 때 만들어진 인스턴스가 전체 시스템에서 하나뿐임을 보장한다.

다음 코드는 public static final 필드 방식의 싱글턴 예제이다.

```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
 
    private Elvis() {
    }
}
```

예외로 권한이 있는 클라이언트가 리플렉션 API ```AccessibleObject.setAccessible```를 사용해서 private 생성자를 호출할 수 있다. 이러한 공격을 방어하기 위해서는 두 번째 객체가 생성되는 경우에 예외를 발생시켜서 막는 방법이 있다.

## 정적 팩터리 메서드 방식
정적 팩터리 메서드를 public static 멤버로 제공한다. Elvis.getInstance는 항상 같은 객체의 참조를 반환하므로 시스템 내에 하나뿐인 객체가 된다. 이 방식에서도 리플렉션을 사용하면 예외가 동일하게 발생한다.

다음 코드는 정적 팩터리 메서드 방식의 싱글턴 예제이다.

```java
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
 
    private Elvis() {
    }
     
    public static Elvis getInstance() {
        return INSTANCE;
    }
}
```

두 가지 방식으로 만든 싱글턴 클래스를 직렬화하기 위해서는 단순히 ```Serializable```를 구현하는 것만으로는 부족하다. 모든 인스턴스 필드를 transient이라고 선언하고 readResolve 메서드를 제공해야 한다. 이렇게 하지 않으면 직렬화된 인스턴스를 역직렬화 하는 경우에 매번 새로운 인스턴스가 생성된다.

```java
private Object readResolve() {
    return INSTANCE;
}
```

## 열거 타입 방식
pulbic 필드 방식과 비슷하지만 더 간결하다. 직렬화하는 상황과 리플렉션 공격에도 오직 하나의 인스턴스가 생성됨을 보장한다. <b>원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다.</b>

다음 코드는 열거 타입 방식의 싱글턴 예제이다.

```java
public enum Elvis {
    INSTANCE

    // 메서드 선언 생략 ...
}
```

## 싱글턴 객체를 만드는 다른 방법은?
싱글턴을 객체를 생성할 수 있는 다른 방법에 대해 더 알아보도록 한다. Double Checked Locking과 Lazyholder 두 가지 방식이 있다.

## Double Checked Locking
```java
public class Singleton {
  private volatile static Singleton instance;
  private Singeton() {}
  
  public static Singleton getInstance() {
    if (instance == null)  {
        synchronized(Singleton.class) {
          if (instance == null) {
              instance == new Singleton();  
          }
        }
    }
    
    return instance;
  }
}
```

## LazyHolder
```java
public class Singleton {
  private Singleton() {}
  public static Singleton getInstance() {
    return LazyHolder.INSTANCE;
  }
  
  private static class LazyHolder {
    private static final Singleton INSTANCE = new Singleton();  
  }
}
```

## 참고자료
- [Multi Thread 환경에서의 올바른 Singleton](https://medium.com/@joongwon/multi-thread-%ED%99%98%EA%B2%BD%EC%97%90%EC%84%9C%EC%9D%98-%EC%98%AC%EB%B0%94%EB%A5%B8-singleton-578d9511fd42)