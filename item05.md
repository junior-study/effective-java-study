# 아이템 5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

많은 클래스들은 협력하기 위해서 하나 이상의 자원에 의존한다. 예를 들어 맞춤법 검사기는 사전에 의존하는데, 이러한 맞춤법 검사기를 정적 유틸리티 클래스로 구현한 것을 드물지 않게 볼 수 있다. 이와 유사하게 싱글턴으로 구현하는 경우도 흔한다.

다음 코드는 맞춤법 검사기를 싱글턴으로 구현한 예제이다.

```java
public class SpellChecker {
    private final Lexicon dictionary = ...;

    private SpellChecker() {
    }

    public static SpellChecker INSTANCE = new SpellChecker();

    public boolean isValid(String word) {...}
    
    public List<String> suggestions(String typo) {...}

    // 생략...
}
```

<br/>

이러한 방식으로 클래스를 만들게 되면 사전을 단 한가지만을 사용한다고 가정하게 된다. 하지만 사전은 한가지만 있는게 아닌 언어별로 따로 있을 수 있고 여러 경우가 많기 때문에 한가지 사전만 사용하도록 클래스를 설계하는 것은 좋지 않다.

```dictioanry``` 필드의 final 키워드를 빼고 전략패턴(Strategy Pattern)을 이용해서 동적으로 사전을 바꾸게 되면, 중간에 사전이 교체될 수 있기 때문에 멀티스레드 환경에서는 더욱 조심해야 한다. <b>사용하는 자원에 따라 동작이 달라지는 클래스의 경우에는 정적 유틸클래스나 싱글턴 방식은 적합하지 않다.</b>

맞춤법 검사기 클래스가 여러 인스턴스를 지원하며, 클라이언트가 원하는 자원(dictionary)를 사용하도록 하기 위해서는 <b>인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식</b>이다.

다음 코드는 자원(dictionary)를 생성자에 주입 받아서 사용하는 예제이다.

```java
public class SpellChecker {
    private final Lexicon dictionary;

    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requrieNonNull(dictionary);
    }

    public boolean isValid(String word) {...}
    
    public List<String> suggestions(String typo) {...}

    // 생략...
}
```

<br/>

의존 객체 주입이 여러모로 도움을 주지만, 의존성이 많은 큰 프로젝트에서는 코드를 어지럽게 만들 수 있다. 이러한 경우에는 Dagger, Guice, Spring와 같은 의존 객체 주입(DI)프레임워크를 사용하는 것이 좋다.

> 클래스 내에서 사용되는 자원을 클래스가 직접 생성하는 것은 좋지 않다. 대신에 필요한 자원을 클래스에 넘겨줘야 한다. 의존 객체 주입은 클래스의 유연성, 재사용성, 테스트 용이성을 개선해준다.
