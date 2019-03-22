# 아이템 4. 인스턴스화를 막으려거든 private 생성자를 사용하라

<b>단순히 정적 메서드와 정적 필드만을 담은 클래스</b>를 만들어야 하는 경우가 있다. 이 방식은 객체 지향적으로 좋은 않은 클래스라고 여길 수 있지만, 이러한 클래스도 나름의 쓰임새가 있다.

예를 들어 ```java.lang.Math```와 ```java.util.Arrays``` 클래스처럼 기본 타입 값이나 배열 관련 메서드를 모아 놓을 수 있다. 또한, ```java.util.Collections```처럼 특정 인터페이스를 구현하는 객체를 생성해주는 정적 메서드를 가지고 있는 클래스가 있을 수 있다.

다음 코드는 Math 클래스의 일부분이다.

```java
public final class Math {

    private Math() {}

    public static final double E = 2.7182818284590452354;

    public static final double PI = 3.14159265358979323846;


    public static double sin(double a) {
        return StrictMath.sin(a); // default impl. delegates to StrictMath
    }

    public static double cos(double a) {
        return StrictMath.cos(a); // default impl. delegates to StrictMath
    }

    public static double tan(double a) {
        return StrictMath.tan(a); // default impl. delegates to StrictMath
    }
    
    // 생략 ...
}
```

<br/>

<b>정적 멤버만 담은 유틸리티 클래스는 사실 인스턴스로 만들어 쓰려고 설계한 것이 아니다.</b> Math 클래스는 ```final``` 클래스이기 때문에 상속해서 하위 클래스에 메서드를 넣을 수도 없다. 그리고 private 생성자를 만들었기 때문에 컴파일러가 자동으로 기본 생성자를 만들지도 않는다.

유틸리티 클래스를 추상 클래스로 만들면 인스턴스 생성하는 것을 막을 수 없다. 이러한 종류의 클래스는 애초에 다른 클래스에서 상속 받아 사용하도록 설계한 것이 아니기 때문에 인스턴스가 생성되는 것을 막는 것이 좋다.

인스턴스 생성을 막는 방법은 간단한다. 개발자가 클래스의 기본 생성자를 빼먹으면 컴파일러가 기본 생성자를 추가해준다. 그렇기 때문에 명시적으로 <b>private 생성자</b>를 클래스에 추가하면 클래스의 인스턴스 생성을 막을 수 있다.

다음 코드는 Utils 클래스에 private 생성자를 넣는 예제이다.

```java
public class Utils {

    private Utils() {
        throw new Exception("Utils 클래스는 인스턴스화를 할 수 없습니다.");
    }

    public static int plus(int a, int b) { 
        return a + b
    }

    // 정적 멤버 선언...
}
```

<br/>

만약 Utils 클래스를 상속 받아서 사용하는 코드를 작성하면, 하위 클래스 생성자에서는 에러가 발생한다. 하위 클래스에서는 상위 클래스의 생성자를 반드시 호출해야 하는데, 이를 private 접근자로 선언했으므로 상위 클래스의 생성자에 접근할 수 없다.