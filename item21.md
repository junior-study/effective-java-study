# 아이템 21. 인터페이스는 구현하는 쪽을 생각해 설계하라

자바 8에 들어서며 기존 인터페이스에 디폴드 메서드를 선언하여 메서드를 추가하는 길이 열렸지만 (Even Though), 모든 기존 구현체들과 매끄럽게 연동되리라는 보장은 없다.

## removeIf

```java
 // 자바 8의 Collection 인터페이스에 추가된 디폴트 메서드
 // 불리언 함수 predicate가 true를 반환하는 모든 원소를 제거
 default boolean removeIf(Predicate<? super E> filter) {
        Objects.requireNonNull(filter);
        boolean removed = false;
        final Iterator<E> each = iterator();
        while (each.hasNext()) {
            if (filter.test(each.next())) {
                each.remove();
                removed = true;
            }
        }
        return removed;
    }
```

## [apache.commons.collections4.collection.SynchronizedCollection](https://github.com/apache/commons-collections/blob/master/src/main/java/org/apache/commons/collections4/collection/SynchronizedCollection.java)

모든 메서드에서 주어진 락 객체로 동기화한 후 내부 컬렉션 객체에 기능을 위임하는 래퍼 클래스다.

removeIf 메서드를 재정의하지 않고 있기 때문에 removeIf의 default 구현을 물려받게 된다면 락 객체를 사용하여 동기화를 할 수 없다.

> 멀티 스레드 환경에서 ConcurrentModificationException이 발생하거나 다른 예기치 못한 결과로 이어질 수 있다.

## 정리

인터페이스의 디폴트 메서드를 잘 쓰자.

_어떻게, 왜?_

- 디폴트 메서드는 (컴파일에 성공하더라도) 기존 구현체에 런타임 오류를 일으킬 수 있다.
- 디폴트 메서드는 인터페이스로부터 메서드를 제거하거나 기존 메서드의 시그니처를 수정하는 용도가 아님을 명심해야 한다. -> 기존 클라이언트를 망가뜨리게 됨
- 기존 인터페이스에 디폴트 메서드로 새 메서드를 추가하는 일은 꼭 필요한 경우가 아니라면 피해야 한다.
