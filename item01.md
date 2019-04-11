# 아이템 1. 생성자 대신 정적 팩터리 메서드를 고려하라

클라이언트가 클래스의 인스턴스를 얻는 전통적인 수단은 ```public 생성자```이다. 클래스는 생성자와 별도로 ```정적 팩터리 메서드(static factory method)```를 제공할 수 있다.

다음 코드는 Boolan 클래스에서 발췌한 예이다. ```valueOf``` 메서드는 기본 타입 ```boolean``` 값을 받아서 ```Boolean``` 객체 참조를 반환한다.

```java
public static Boolean valueOf(boolean b) {
    return b ? Boolean.TRUE : Boolean.FALSE;
}
```

## 정적 팩터리 메서드가 생성자보다 좋은 장점은?

### 장점 1. 이름을 가질 수 있다
- 생성자에 넘기는 매개변수와 생성자 자체만으로는 반환될 객체의 특성을 제대로 설명하지 못한다.
- 정적 팩터리 메서드는 네이밍을 잘하면 반환될 객체의 특성을 쉽게 묘사할 수 있다.
- 하나의 시그니처로는 생성자를 하나만 만들 수 있지만, 정적 팩터리 메서드는 제한이 없다.

<br/>

생성자와 정적 팩터리 메서드에서 값이 소수인 ```BigInteger```를 반환하는 방법은 다음과 같다.

어느 코드가 본질적인 의미를 잘 전달하고 있는가? 명시적으로 소수의 값을 반환할 것이라고 예상되는 정적 팩터리 메서드가
API를 사용하는 클라이언트에게 의미를 잘 전달할 수 있다.

| 생성자    | 정적 팩터리 메서드 |
|--------|---------|
| BigInteger(int, int, Random) | BigInteger.probablePrime |

<br/>

### 장점 2. 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다
- 인스턴스를 미리 생성해놓거나 또는 새로 생성한 인스턴스를 캐싱해서 불필요한 객체 생성을 피한다.
- ```Boolean.valueOf(boolean)``` 메서드는 객체를 아예 생성하지 않는다. 이전에 설명한 코드를 다시 살펴보자. 이미 만들어둔 ```Boolean.TRUE```와 ```Boolean.FALSE```를 반환하고 있다.
- 플라이웨이트 패턴(Flyweight pattern)과 비슷한 기법이다.

<br/>

### 장점 3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다
- 반환할 객체의 클래스를 자유롭게 선택할 수 있는 유연함이 있다.
- 정적 팩터리 메서드의 반환 타입으로 인터페이스로 지정하면, 해당 인터페이스의 구현 클래스를 외부에 노출하지 않고 전달할 수 있어서 API를 작게 유할 수 있다.
- API가 작아진 것은 개발자가 API를 익히기에 필요한 개념의 수와 난이도를 낮출 수 있다. 개발자는 인터페이스를 알고 있기 때문에 굳이 구현체 클래스를 알 필요가 없기 때문이다.

<br/>

### 장점 4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다
- 반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체를 반환하더라도 상관없다.
- 다음 릴리즈에서는 또 다른 클래스의 객체를 반환해도 된다.
- ```EnumSet``` 클래스는 원소의 수에 따라 두 가지 하위 클래스 중 하나의 인스턴스를 반환한다. API를 사용하는 클라이언트 입장에서는 ```EnumSet``` 하위 클래스라는 것만 알면 된다.

<br/>

### 장점 5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다
- 이러한 유연함은 서비스 프로바이더 프레임워크(service provider framework)를 만드는 근간이 된다.
- 대표적인 서비스 프로바이더 프레임워크로는 JDBC(Java Database Connectivity)가 있다.

<br/>

서비스 프로바이더 프레임워크는 구현체의 동작의 정의하는 ```서비스 인터페이스```, 구현체를 등록할 때 사용하는 ```프로바이더 등록 API ```, 클라이언트가 서비스의 인스턴스를 얻을 때 사용하는 ```서비스 액세스 API``` 3개의 핵심 컴포넌트가 필요하다. 부가적으로 서비스 인터페이스를 생성하는 ```서비스 프로바이더 인터페이스```를 사용하기도 한다. 서비스 프로바이더 인터페이스가 없다면 각 구현체를 인스턴스로 만들 때 리플렉션을 사용해야 한다.

JDBC에서는 ```Connection```이 서비스 인터페이스 역할, ```DriverManager.registerDriver```가 프로바이더 등록 API 역할, ```DriverManager.getConnection```이 서비스 접근 API 역할, ```Driver```가 서비스 제공자 인터페이스 역할을 각각 수행한다.

```DriverManager.getConnection``` 메서드를 호출해서 반환되는 Connection 인터페이스의 구현체는 데이터베이스마다 다르다.

다음 코드는 MySQL JDBC를 이용해서 Connection 객체를 얻어오는 예제이다.

```java
// DriverManager에 연결하고자 하는 데이터베이스의 JDBC 드라이버를 지정한다.
Class.forName("com.mysql.cj.jdbc.Driver").newInstance();

// DriverManager.getConnection로 접근해서 Connection을 가져온다.
Connection conn = DriverManager.getConnection("jdbc:mysql://localhost/test?" +
        "user=minty&password=greatsqldb");
```

다음은 MySQL JDBC [Driver 클래스](https://github.com/mysql/mysql-connector-j/blob/66459e9d39c8fd09767992bc592acd2053279be6/src/main/user-impl/java/com/mysql/cj/jdbc/Driver.java)에서 가져온 코드이다.

```java
public class Driver extends NonRegisteringDriver implements java.sql.Driver {
    //
    // Register ourselves with the DriverManager
    //
    static {
        try {
            // DriverManager에 Driver를 등록한다.
            java.sql.DriverManager.registerDriver(new Driver());
        } catch (SQLException E) {
            throw new RuntimeException("Can't register driver!");
        }
    }

    /**
     * Construct a new driver and register it with DriverManager
     * 
     * @throws SQLException
     *             if a database error occurs.
     */
    public Driver() throws SQLException {
        // Required for Class.forName().newInstance()
    }
}
```

## 정적 팩터리 메서드가 생성자와 비교하여 단점은?

### 단점 1. 상속을 하려면 생성자가 필요한데, 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다
- 컬렉션 프레임워크 유틸리티 구현 클래스들은 상속할 수 없다.
- 이러한 제약은 상속보다 구성을 사용하도록 유도하고 불변 타입으로 만들도록 하므로 오히려 장점일 수 있다.

<br/>

### 단점 2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다
- 생성자처럼 API 설명에 명확히 드러나지 않으므로 클라이언트는 클래스의 인스턴스를 얻을 수 있는 방법을 알아내야 하는 번거로움이 있다.
- 해결 방안으로는 API 문서에 인스턴스를 얻는 방법을 설명하고, 메서드 이름도 널리 알려진 규약을 따라 짓는 식으로 문제를 완화해야 한다.

## 정적 팩터리 메서드 명명 방식
정적 팩터리 메서드에서 흔힌 사용되는 명명 방식들에 대해 알아보자. 지키지 않으면 에러가 발생하는 것은 아니지만, 우리는 혼자 개발하는 것이 아닌 동료들과 함께하므로 이러한 명명 방식을 지키도록 노력해야 한다.

| 메서드 | 설명 |
|------|------|
| from | 매개변수를 하나 받아서 해당 타입의 인스턴스를 반환하는 형변환 메서드 |
| of | 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드 |
| valueOf | from과 of의 더 자세한 버전 |
| instance, getInstance | 인스턴스를 반환하지만, 같은 인스턴스임을 보장하지 않는 메서드 |
| create, newInstance | 매번 새로운 인스턴스를 생성해 반환함을 보장하는 메서드 |
| getType | 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓰는 메서드 |
| newType | 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓰는 메서드 |
| type | getType과 newType의 간결한 버전 |
