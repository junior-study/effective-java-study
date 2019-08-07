> 아이템 72. 표준 예외를 사용하라

## 표준 예외를 사용했을 때 장점

예외도 재사용하는 것이 좋다. 자바 라이브러리는 대부분 API에서 쓰기에 충분한 수의 예외를 제공한다. 표준 예외를 재사용하면 사용하기 쉽고, 낯설지 않아 읽기 쉽고,  마지막으로 예외 클래스 수가 적어 메모리 사용량도 줄고 클래스를 적재하는 시간도 적게 걸린다. 





가장많이 사용하는 예외 **IllegalArgumentException**. 호출자 인수로 부적절한 값을 넘길 때 던지는 예외이다. 

**IllegalStateException**. 대상 객체의 상태가 호출된 메서드를 수행하기에 적합하지 않을 때 주로 던진다. 예컨대 제대로 초기화 되지 않은 객체를 사요하려 할 때 ㅈ던질 수 있다.

>  null 값을 허용하지 않는 메서드에 null을 건네면 관례상 IllegalArgumentException이 아닌 NullPointerException을 던진다. 비슷하게 어떤 시퀀스의 허용범위를 넘는 값을 건넬 때도 IllegalArgumentException보다는 IndexOutOfBoundsException을 던진다. 



**ConcurrentModificationException은** 단일 스레드에서 사용하려고 설계한 객체를 여러 스레드가 동시에 수정하려 할때 던진다. 

**UnsupportedOperationException는** 클라이언트가 요청한 동작을 대상 객체가 지원하지 않을 때 던진다. 대부분 객체는 자신이 정의한 메서드를 모두 지원하니 흔히 쓰이는 예외는 아니다. 

**Exception, RuntimeException, Throwable, Error는 직접 재사용하지 말자.**

상황에 부합한다면 항상 표준 예외를 재사용하자.더 많은 정보를 제공하길 원한다면 표준예외를 확장해도 된다. 단, 예외를 직렬화 할 수 있다는 사실을 기억하자. (직렬화에는 많은 부담이 따르니) 이 사실만으로도 나만의 예외를 새로 만들지 않아야 할 근거로 충분할 수 있다. 

