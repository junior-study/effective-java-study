> 아이템78. 공유중인 가변 데이터는 동기화해 사용하라 



synchronized 키워드는 해당 메서드나 블록을 한번에 한 스레드씩 수행하도록 보장한다. 



## 동기화의 2가지 특징

**동기화의 배타적실행**

한 객체가 일관된 상태를 가지고 생성되고, 이 객체에 접근하는 메서드는 그 객체에 락을 건다. 락을 건 메서드는 개체의 상태를 확인하고 필요하면 수정한다. 즉, 객체를 하나의 일관된 상태에서 다른 일관된 상태로 변화시킨다. 동기화를 제대로 사용하면 어떤 메서드도 이 객체의 상태가 일관되지 않은 순간을 볼 수 없을 것이다.



**스레드간 통신**

동기화에 중요한 기능이 하나 더 있다. 동기화 없이는 한 스레드가 만든 변화를 다른 스레드에서 확인하지 못할 수 있다. 동기화는 일관성이 깨진 상태를 볼 수 없게 하는 것은 물론, 동기화된 메서드나 블록에 들어간 스레드가 같은 락의 보호하에 수행된 모든 이전 수정의 최종 결과를 보게 해준다. 



언어 명세상 long과 double 외의 변수를 읽고 쓰는 동작은 원자적이다. 자바 언어 명세는 스레드가 필드를 읽을 때 항상 ’수정이 완전히 반영된’ 값을 얻는다고 보장하지만, 한 스레드가 저장한 값이 다른 스레드에게 ‘보이는가’는 보장하지 않는다. **동기화는 배타적 실행뿐 아니라 스레드 사이의 안정적인 통신에 꼭 필요하다.** 



## 예제 코드

```java
public class StopThread {
  private static boolean stopRequested;
  
  public static void main(String[] args) throws InterruptedException {
    Thread backgroundThread = new Thread(()-> {
      int i = 0;
      while (!stopRequested)
        i++;
    });
    backgroundThread.start();
    
    TimeUnit.SECONDS.sleep(1);
    stopRequested = true;
  }
}
```

main스레드외에 background 스레드가 돌면서 1초 후에 종료되리라 생각하는가?  하지만 컴퓨터는 도통 끝날 줄 모르고 영원히 수행된다. 원인은 동기화에 있다. **동기화를 하지 않으면 메인스레드가 수정한 값 (여기서는 stopRequested)을 백그라운드 스레드가 언제쯤 보게 될지 보증할 수가 없다.** 이문제는 stopRequested 필드를 동기화해 접근하면 이 문제를 해결 할 수 있다. 그래서 다음 처럼 바꾸면 기대한 대로 1초후에 종료된다. 





```java
public class StopThread {
  private static boolean stopRequested;
  
  //쓰기 메서드에 동기화
  private static synchronized void requestStop(){
    stopRequested = true;
  }
  
  // 읽기 메서드에 동기화
  private static synchronized boolean stopRequested(){
    return stopRequested;
  }
  
  public static void main(String[] args) throws InterruptedException {
    Thread backgroundThread = new Thread(()-> {
      int i = 0;
      while (!stopRequested())
        i++;
    });
    backgroundThread.start();
    
    TimeUnit.SECONDS.sleep(1);
		requestStop();
  }
}
```

쓰기와 읽기 메서드 모두가 동기화 되지 않으면 동작을 보장하지 않는다. 어떤 기기에서는 둘 중 하나만 동기화해도 동작하는 듯 보이지만, 겉모습에 속아서는 안된다. 

**다른 해결책**

반복문에서 매번 옹기화하는 비용이 크진 않지만 속도가 더 빠른 대안을 소개하겠다. stopRequested 필드를 volatile로 선언하면 동기화를 생략해도 된다. **<u>volatile 한정자는 배타적 수행과는 상관없지만 항상 가장 최근에 기록된 값을 읽게 됨을 보장한다. (그 이유가, 메인 메모리에 올리기 때문에)</u>**

```java
public class StopThread {
  private static volatile boolean stopRequested;
  
  public static void main(String[] args) throws InterruptedException {
    Thread backgroundThread = new Thread(()-> {
      int i = 0;
      while (!stopRequested)
        i++;
    });
    backgroundThread.start();
    
    TimeUnit.SECONDS.sleep(1);
    stopRequested = true;
  }
}
```

> 📕 volatile는 자바 변수를 main 메모리에 저장하겠다는 것을 명시한다. 매번 변수의 값을 읽고 / 쓸때 메인 메모리를 통해서 읽고/쓰는 동작을 수행한다. (Main Memory에 read & write를 보장하는 키워드)
>
> - 언제 사용할까? 멀티스레드 환경에서 스레드 변수값을 읽어 올때 각각의 CPU cache에 저장된 값이 다르기 때문에 변수값 불일치  문제가 발생한다. 
>
> - 항상 옳을까? 그렇지 않다. 여러 thread가 write 하는 상황이면, synchronized를 통해 read & write 원자성을 보장해야 한다. 
>
> - 성능은?  CPU cache보다 메인 메모리가 비용이 크게 발생한다. (필요한 경우에는 volatile를 사용하는 것이 좋다)
>
> https://nesoy.github.io/articles/2018-06/Java-volatile



## volatile은 주의해서 사용해야한다.

```java
private static volatile int nextSerialNumber = 0;

public static int generateSerialNumber(){
  return nextSerialNumber++;
}
```

다음은 일련번호를 생성할 의도로 작성한 메서드다. 문제는 증가 연산자(++)이다. 이 연산자는 코드상으로 하나지만 실제로는 nextSerialNumber 필드에 두번 접근한다. 먼저 값을 읽고 그런다음 1증가한 새로운 값을 저장하는 것이다. 만약 두 번째 스레드가 이 두 접근 사이를 비집고 들어와 값을 읽어가면 첫 번째 스레드와 똑같은 값을 돌려받게 된다. 프로그램이 잘못된 결과를 계산해 내는 오류를 안전실패(safety failure)라고 한다. synchronized 한정자를 붙이면 이 문제가 해결된다. 더 견고하게 하기위해서 int대신 long타입으로 바꾸고, 최댓값에 도달하면 예외를 던지게 하자. 

**더 나은 대안**

`java.utill.concurrent.atomic` 패키지의 AtomicLong을 사용해보자. 이 패키지에는 락 없이도 스레드 안전한 프로그래밍을 지원하는 클래스들이 담겨 있다. volatile은 동기화의 두 효과 중 통신 쪽만 지원하지만 이 패키지는 원자성(배타적 실행)까지 지원한다. 더구나 성능도 동기화 버전 보다 우수하다. 

```java
private static final AtomicLong nextSerialNum = new AtomicLong();

public static long generatedSerialNumber(){
  return nextSerialNum.getAndIncrement();
}
```





아이템에서 언급한 문제들을 피하는 가장 좋은 방법은 가변 데이터를 공유하지 않는 것이다. 다시 말해 가변 데이터는 단일 스레드에서만 쓰도록 하자. 이 정책을 받아들였다면, 그 사실을 문서에 남겨 유지보수 과정에서도 정책이 계속 지켜지도록 하는게 중요하다. 



## 핵심정리

여러 스레드가 가변 데이터를 공유한다면 그 데이터를 읽고 쓰는 동작은 반드시 동기화해야 한다. 배타적 실행은 필요없고 스레드끼리의 통신만 필요하다면 volatile 한정자만으로 동기화 할 수 있다. 다만 올바로 사용하기가 까다롭다.
