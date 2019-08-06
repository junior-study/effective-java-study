> 아이템83. 지연 초기화는 신중히 사용하라 

## 지연초기화 ?

지연초기화는 필드의 초기화 시점을 그 값이 처음 필요할 때까지 늦추는 기법이다. 지연 초기화는 주로 최적화 용도로 쓰이지만, 클래스와 인스턴스 초기화 때 발생하는 위험한 순환 문제를 해결하는 효과도 있다.

다른 모든 최적화와 마찬가지로 지연초기화는 ‘필요할 때까지는 하지말라’. 클래스 혹은 인스턴스 생성 시의 초기화 비용은 줄어들지만 그 대신 지연 초기화 하는 필드에 접근하는 비용은 커진다. 



## 지연 초기화 언제 필요한가? 

그럼에도 지연 초기화가 필요할 때가 있다. **해당 클래스의 인스턴스 중 그 필드를 사용하는 인스턴스의 비율이 낮은 반면, 그 필드를 초기화 하는 비용이 크다면 지연 초기화가 제 역할을 해줄 것이다.** 하지말 실제 알 수 있는 유일한 방법은 지연 초기화 적용 전/후로 성능을 측정해 보는 것이다.

멀티 스레드 환경에서는 지연 초기화를 하기가 까다롭다. 지연 초기화하는 필드를 둘 이상의 스레드가 공유한다면 어떤 형태로든 반드시 동기화해야 한다. 그렇지 않으면 심각한 버그로 이어질 것이다. 

대부분의 상황에서 **일반적인 초기화가 지연 초기화 보다 낫다**. 인스턴스 필드를 선언할 때 수행하는 일반적인 초기화의 모습이다.

```java
// 인스턴스 필드를 초기화 하는 일반적인 방법
private final FieldType field = computerFieldValue();
```

지연 초기화가 초기화 순환성을 깨드릴 것 같으면 synchronized를 단 접근자를 사용하자. 이 방법이 가장 간단하고 명확한 대안이다. 

```java
private FieldType field;

// 인스턴스 필드의 지연 초기화 -synchronized 접근자 방식
private synchronized FieldType getField(){
  if(field == null){
    field = computeFieldValue();
    return field;
  }
}
```

성능 때문에 정적 필드를 지연 초기화해야 한다면 지연 초기화 홀더 클래스 관용구를 사용하자. 클래스는 클래스가 처음 쓰일 때 비로소 초기화 된다는 특성을 이용한 관용구다. 

```java
// 정적 필드용 지연 초기화 홀더 클래스 관용구
private static class FieldHolder {
  static final FieldType field = computeFieldValue();
}

private static FieldType getField(){
  return FieldHolder.field;
}
```

getField가 처음 호출되는 순간 FieldHolder.field가 처음 읽히면서, 비로소 FieldHolder 클래스 초기화를 촉발한다. 이 관용구의 멋진 점은 getField메서드가 필드에 접근하면서 동기화를 전혀 하지 않으니 성능이 느려질 거리가 전혀 없다는 것이다. 일반적인 VM은 오직 클래스를 초기화 할때만 필드 접근을 동기화 할 것이다. 



성능때문에 인스턴스 필드를 지연 초기화해야 한다면 이중검사 관용구를 사용하라. 이 관용구는 초기화된 필드에 접근할 때의 동기화 비용을 없애준다. 이름에서도 알 수 있듯이 필드의 값을 두 번 검사하는 방식으로, 한 번은 동기화 없이 검사하고, (필드가 아직 초기화 되지 않았다면) 두번째는 동기화하여 검사한다. 두번째 검사에서도 필드가 초기화 되지 않았을 때만 필드를 초기화 한다. 필드가 초기화된 후로는 동기화 하지 않으므로 해당 필드는 반드시 volatile로 선언해야 한다. 

```java
//인스턴스 필드 지연 초기화용 이중검사 관용구
private volatile FieldType field;

private FieldType getField(){
  FieldType result = field;
  if(result != null){ // 첫번째 검사 (락 사용 안함)
    return result;
  }
  synchronized(this){
    if (field == null){ // 두번째 검사 (락 사용)
      field = computeFieldValue();
      return field;
    }
  }
}
```

result 변수가 필요한 이유는 뭘까? 이 변수는 필드가 이미 초기화된 상황에서는 그 필드를 딱 한 번만 읽도록 보장하는 역할을 한다. 반드시 필요하지는 않지만 성능을 높여주고, 저수준 동시성 프로그래밍에 표준적으로 적용되는 우아한 방법이다. 



```java
//단일 검사 관용구 - 초기화가 중복해서 일어날 수 있다
private volatile FieldType field;

private FieldType getField(){
  FieldType result = field;
  if(result == null){
    field = result = computeFieldValue();
    return result;
  }
}
```

이번 아이템에서 모든 초기화 기법은 기본 타입 필드와 객체 참조 필드 모두에 적용할 수 있다. 이중 검사와 단일 검사 관용구를 수치 기본 타입 필드에 적용한다면 필드의 값을 null대신 (숫자 기본 타입변수의 기본값인) 0과 비교하면 된다.



## 핵심정리

대부분의 필드는 지연시키지 말고 곧바로 초기화해야 한다. 성능 때문에 혹은 위험한 초기화 순환을 막기 위해 꼭 지연 초기화를 써야 한다면 올바른 지연 초기화 기법을 사용하자. 인스턴스필드에는 이중검사 관용구를, 정적 필드에는 지연 초기화 홀더 클래스 관용구를 사용하자. 반복해 초기화해도 괜찮은 인스턴스 필드에는 단일 검사 관용구도 고려 대상이다.
