# 아이템 9. try-finally보다는 try-with-resources를 사용하라

자바 라이브러리에는 close 메서드를 호출해서 직접 닫아줘야 하는 자원이 많다. ```InputStream```, ```OutputStream```, ```java.sql.Connection``` 등이 있다. 자원을 닫는 것을 잊어버리면 예상치 못한 문제로 이어질 수 있다. [아이템 8](item8.md)에서 살펴봤지만 finalizer, cleaner는 안전망 역할을 할뿐이지 즉시 해제 된다는 것을 보장 받지 못한다.

## 1. try-finally
자바 7 이전에는 ```try-finally```을 통해 자원을 해제하는 로직을 작성했다. finally 블록은 반드시 호출되기 때문에 해당 블록에 자원을 해제하는 작업을 작성했다.

다음 코드는 ```try-finally```를 사용한 예제이다.

```java
public static String firstLineOfFile(String path) throw IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
        return br.readLine();
    } finally {
        br.close();
    }
}
```

finally 블록에서 자원을 해제하는 close 메서드를 호출할 때도 예외가 발생할 수 있다. finally 블록 내에서 다시 ```try-finally```을 사용하게 되면, 코드는 길어지고 가독성도 떨어진다. 자바7부터는 이러한 문제를 해결하기 위해서 ```try-with-resources```를 지원한다.

## 2. try-with-resources
```try-with-resources```를 사용하면 자원을 생성하고 해제하는 작업을 코드로 작성하는 것이 간편해졌다. 단, 이 구조를 사용하기 위해서는 해당 자원이 AutoCloseable 인터페이스를 구현해야 한다. 우리가 자주 사용하는 라이브러리에 있는 자원 클래스들은 대부분 AutoCloseable 인터페이스를 이미 구현하고 있다.

다음 코드는 ```try-finally```에서 작성한 코드를 ```try-with-resources```로 변경한 예제이다.

```java
public static String firstLineOfFile(String path) throw IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    } catch (Exception e) {
        return "";
    }
}
```

try-finally 보다 코드가 간편해지고 읽기 수월해진다. 그리고 예외가 발생한 경우에 문제를 진단하기도 좋다. try 블록에서 자원을 생성한다. try 블록이 끝나면 자원은 자동으로 해제 되기 때문에 개발자가 직접 close 메서드를 호출하지 않아도 된다.

예제에서는 하나의 자원만 사용했지만, 여러 개의 자원을 선언할 수 있다.

다음 코드는 try 블록에서 ```InputStream```, ```OutputStream``` 자원 객체를 생성하는 예제이다.

```java
try (InputStream in = new FileInputStream(src);
    OutputStream out = new FileOutputStream(dst)) {
        // 생략 ..
}
```