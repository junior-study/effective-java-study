# clon 재정의는 주의해서 진행하라

Cloneable은 복제해도 되는 클래스 임을 명시하는 용도의 믹스인 인터페이스지만, 의도한 목적을 제대로 이루지 못함.

가장 큰 문제는 clone메서드가 선언된 곳이 Cloneable이 아닌 Object이고, protected로 되어있어서, Cloneable를 구현하는 것만으로도 외부 객체에서 clone 메서드 호출 x

목적 

- clone 메서드 구현 방법 
- 언제 그렇게 해야 하는지
- 가능한 다른 선택지 



Cloneable 인터페이스는 메서드가 하나도 없다. 

```java
public interface Cloneable {
}
```

이 인터페이스는 Object의 protected 멤서드인 clone의 동작 방식을 결정한다. 

Cloneable을 구현한 클래스의 인스턴스에서 clone()을 호출하면, 그 객체의 필드들을 하나하나 복사한 객체를 반환하며, 그렇지 않은 클래스의 인터페이스를 호출하면 CloneNotSupportedException을 던진다. 

