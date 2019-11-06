# 아이템 27. 비검사 경고를 제거하라

제네릭을 잘못 사용한 경우에 컴파일러는 경고를 나타낸다. 경고에는 무엇이 잘못 되었다는 것을 알려주며, 컴파일러 설명에 맞춰서 코드를 수정하면 경고는 사라지게 된다.

아래 코드는 unchecked conversion 경고가 발생한다.

```java
Set<Lark> exaltation = new HashSet();
```

이 경고는 Java 7부터 추가된 다이아몬드 연산자 '<>'를 사용해서 해결할 수 있다. <code>new HashSet<>()</code> 코드로 변경하면 컴파일러는 타입 매개변수를 추론할 수 있기 때문에 경고가 사라진다.

이처럼 쉽게 해결할 수 있는 비검사 경고가 있기도 하지만 어려운 경고도 있다. <b>할 수 있는 한 코드에서 모든 비검사 경고를 제거</b>해야 한다. 경고는 당장은 문제를 일으키지 않지만, 런타임(실행 중)에 문제를 발생 시킬 수 있기 때문이다.

경고를 제거할 수 없지만 타입이 확실한 경우에는 <code>@SuppressWarnings("unchecked")</code> 어노테이션을 추가함으로서 경고를 숨길 수 있다. 이 어노테이션은 변수, 메서드, 클래스 등 여러 범위에서 사용할 수 있으므로 가능한 좁은 범위에 적용해야 한다.

아래 코드는 <code>@SuppressWarnings("unchecked")</code> 사용 예제이다.

```java
@SuppressWarnings("unchecked") T[] result = (T[]) Arrays.copyOf(elements, size, a.getClass());
```