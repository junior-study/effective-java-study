# 아이템 11. equals를 재정의하려거든 hashCode도 재정의하라

[아이템 10](item10.md)에서 equals 메서드 재정의 규약에 대해서 알아봤다. 주의사항으로 equals 메서드를 재정의하는 경우 ```hashCode``` 메서드도 재정의해야 한다고 했다.

<b>equals를 재정의한 클래스에서는 hashCode도 재정의해야 한다.</b> 이를 어길 경우에 ```HashMap```, ```HashSet```, ```HashTable```와 같은 해시 기반의 컬렉션에서 오동작이 발생한다.

<br/>

다음은 Object 명세에서 발췌한 규약이다.

- 응용프로그램 실행 중에 같은 객체의 hashCode를 여러 번 호출하는 경우에 equals가 사용하는 정보들이 변경되지 않았다면, 언제나 동일한 해쉬값을 반환한다. 다만 프로그램이 재시작한 경우에는 동일한 값이 나올 필요는 없다.
- equals 메서드가 같다고 판단한 두 객체의 hashCode 값은 같아야 한다.
- equals 메서드가 다르다고 판단한 두 객체의 hashCode 값은 꼭 다를 필요는 없다.

<br/>

equals를 재정의했지만, hashCode를 재정의하지 않으면 일반 규약에서 2번째에 해당하는 규약을 위반하는 것이다.

## hashCode를 재정의하지 않으면 발생하는 버그
사용자가 특정 클래스를 생성했다고 가정한다. 이 클래스에서는 논리적 동일성 검사를 하는 equals 메서드를 재정의를 했다. 하지만, hashCode 메서드는 재정의 하지 않았다. 해당 객체를 생성해서 HashMap 컬렉션에 우선 저장한다. 이전에 저장한 객체와 동일하게 객체를 생성해서 HashMap에서 get 메서드를 호출하면 null 값이 반환된다.

다음 코드는 equals만 재정의하고 hashCode를 재정의하지 않는 경우에 오작동이 발생하는 예제입니다. 예제에서 사용되는 클래스는 PhoneNumber로 아이템 10에서 살펴봤습니다.

```java
Map<PhoneNumber, String> map = new HashMap<>();
map.put(new PhoneNumber(031, 112, 1192), "jayden-lee");

// null 값을 반환
String name = phoneNumberMap.get(new PhoneNumber(031, 112, 1192));
```

## hashCode 메서드를 재정의하는 방법
좋은 hashCode는 다른 객체인 경우에 다른 hashCode 값을 반환하는 것이다. 다른 객체인데 동일한 해시 코드를 반환하게 되면 같은 공간의 해시 버킷 공간을 사용하게 된다. 되도록이면 다른 해시 코드가 반환되도록 해서 균등하게 배분 할 수 있도록 해야 한다.

hashCode를 작성하는 방법은 책에 자세히 설명되어 있다. equals에서 사용된 필드는 hashCode를 생성할 때 반드시 사용해야 한다. 그리고 자바에서는 기본 필드의 hashCode 값을 계산하기 위해서 Type.hashCode API를 제공한다. 배열의 경우에는 Arrays.hashCode를 사용하면 된다. 파생 필드는 hashCode를 계산에서 제외해도 된다.

다음 코드는 아이템 10에서 살펴봤던 PhoneNumber 클래스의 hashCode를 구현한 예제이다. result 마다 31을 곱한 이유는 곱셈을 시프트 연산과 뺼셈으로 대체해 최적화 할 수 있다. 또한, 31은 소수이다.

```java
@Override
public int hashCode() {
    int result = Short.hashCode(areaCode);
    result = 31 * result + Short.hashCode(prefix);
    result = 31 * result + Short.hashCode(lineNum);
    return result;
}
```

> 해싱 충돌이 더욱 적은 방법을 써야 한다면 guava의 [Hashing](https://google.github.io/guava/releases/23.0/api/docs/com/google/common/hash/Hashing.html)을 참고하자.

<br/>

자바 7 이후부터는 Objects 클래스에서 임의의 개수만큼 객체를 받아서 해시코드를 계산해주는 hash 메서드를 제공한다.

다음 코드는 Objects 클래스의 정적 메서드 hash를 사용한 예제이다.

```java
@Override
public int hashCode() {
    return Objects.hash(lineNum, prefix, areaCode);
}
```

hashCode를 생성할 때 성능을 높이기 위해서 핵심 필드 계산을 제외하면 안된다. 그리고 hashCode 생성 규칙을 API 외부로 자세히 공표할 필요는 없다. 만약 다른 클라이언트가 해당 방법을 확인하고 의존하도록 코드를 작성할 수 있기 때문이다. AutoValue 프레임워크를 이용하면 equals, hashCode를 자동으로 만들어준다. 프레임워크에 의존적이면 안되지만, 자신이 직접 만드는 것보다 안정성을 보장하기 떄문에 이용하는 것도 나쁘지 않다.

## 이해가 안된다면 아래 자료를 추가적으로 살펴보자
- [java.lang.Object.hashCode 메소드](https://johngrib.github.io/wiki/Object-hashCode/)