# 아이템 10. equals는 일반 규약을 지켜 재정의하라
```
결론: 꼭 필요한 경우가 아니면 equals를 재정의하지 말자. 재정의 할 때는 5가지 규약을 지키며 비교한다. 
IDE, Google AutoValue로 도움 받는걸 추천함.
```

## equals는 언제 정의 하나?
- 객체 식별성이 아니라, 논리적 동치성(값 비교)을 확인해야 하는데, 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의 되지 않았을 때


## equals 재정의 할 때 따르는 일반 규약 5가지
1. reflexivity: x.equals(x) -> true
2. symmetry: x.equals(y) 가 true이면 -> y.equals(x)
3. transitivity: x.equals(y) , y.equals(z) -> z.equals(x)
4. consistency: x.equals(y) 반복해서 호출하면 항상 true 이거나 항상 false를 반환
5. not-null: x.euqals(null) -> false

## equals 메서드 구현 방법 
1. == 연산자 사용해 입력이 자기 자신의 참조 인지 확인
2. instanceof 연산자로 입력이 올바른 타입인지 확인 (null 검사 굳이 하지 않아도 됨) 
3. 입력을 올바른 타입으로 형변환
4. 입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사


### 참조 
- [리스코프 치환 원칙](http://wonwoo.ml/index.php/post/1780)
- [google AutoValue](https://github.com/google/auto/blob/master/value/userguide/index.md)
- [why autovalue?](https://www.baeldung.com/introduction-to-autovalue)
- [autovalue plugin youtube](https://www.youtube.com/watch?v=sMX9PT3ecu8)
- [lombok - @EqualsAndHashCode](https://projectlombok.org/features/EqualsAndHashCode)

### 총평
- 그럼에도 불구하고, 특별하게 equals()가 중요하게 여겨지는 비즈니스 로직을 직접 경험하지 못해서 와닿지는 않음. 큰 문제가 직접 발생하면 식겁할 듯 
