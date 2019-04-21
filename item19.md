> 아이템 19. 상속을 위한 설계와 문서를 갖추거나, 그럴수 없다면 상속을 금지하라 



### 재정의 가능 메서드를 내부적으로 어떻게 사용하는지 반드시 문서로 남겨라 

- AbstractiCollection의 remove()

```java
/**
     * {@inheritDoc}
     *
     * <p>This implementation iterates over the collection looking for the
     * specified element.  If it finds the element, it removes the element
     * from the collection using the iterator's remove method.
     *
     * <p>Note that this implementation throws an
     * <tt>UnsupportedOperationException</tt> if the iterator returned by this
     * collection's iterator method does not implement the <tt>remove</tt>
     * method and this collection contains the specified object.
     
   		콜렉션의 반복자 메소드에 의해 리턴 된 반복자가 remove 메소드를 구현하지 않고,이 콜렉션에 지정된 오브젝트가 포함   
   		되어있는 경우에 UnsupportedOperationException이 발생

     
     * @throws UnsupportedOperationException {@inheritDoc}
     * @throws ClassCastException            {@inheritDoc}
     * @throws NullPointerException          {@inheritDoc}
     */
    public boolean remove(Object o) {
        Iterator<E> it = iterator();
        if (o==null) {
            while (it.hasNext()) {
                if (it.next()==null) {
                    it.remove();
                    return true;
                }
            }
        } else {
            while (it.hasNext()) {
                if (o.equals(it.next())) {
                    it.remove();
                    return true;
                }
            }
        }
        return false;
    }
```

Iterator메서드를 재정의하면 remove가 영향을 받는 다는 사실을 알 수 있다. 게다가 iterator 메서드가 반환하는 iterator 객체가 remove 메서드에 어떤 영향을 주는지 정확하게 명시되어 있다. 



- AbstractList의 removeRange()



### 상속을 고려하여 클래스를 설계할 때 protected로 선언할 멤버는 어떻게 정하나? 

- 실제로 하위클래스를 만들어 보면서 테스트 하는 것 
  - 중요한 멤버를 protected로 선언하는 것을 잊었다면, 하위 클래스를 만들때 영향을 받음
  - 반대로, 하위 클래스를 몇 개 만들어 봐도 사용할 일이 없었던 protected 멤버는 private으로 선언 해야 함





### 상속을 허용하려면 반드시 따라야할 제약사항 

1. 생성자는 직/간접적으로 재정의 가능 메서드를 호출해서는 안된다. 



```java
public class Super {
  // 생성자가 재정의가능 메서드를 호출하는 잘못된 사례
  public Super(){
    overrideMe();
  }
  public void overrideMe();
}
```

```java
public final class Sub extends Super {
  private final Date date; 
  
  Sub(){
    // 상위클래스 생성자 먼저 호출
    date = new Date();
  }
  
  // 상위 클래스 생성자가 호출하게 되는 재정의 메서드
  @Override
  public void overrideMe(){
    System.out.println(date);
  }
  
  public static void main(String[] args){
    Sub sub = new Sub();
    sub.overrideMe();
  }
}
```

두번 date가 찍힐것을 예상했지만, 첫번째는 null, 두번째만 date값이 찍힌다. 

Main메서드의 `new Sub()` 를 하게되면 하위클래스의 생성자가 호출 -> 상위 클래스의 생성자 호출 -> overrideMe()를 호출 (하지만 date값이 초기화만 되었고 아무 값이 없다.) 그리고 -> sub생성자의 `new Date()`가 실행되고 `date` 필드에 날짜 값이 할당된다. -> `sub.overrideMe()`를 호출하면 -> date값이 찍힌다. 



> NullPointerException이 발생하지 않은 이유? 
>
> System.out.println() 메서드가 null인자에 대해서 잘 대처하도록 구현되어 있기 때문



- Cloneable과 Serializable 인터페이스를 사용할 경우 상속용 클래스를 설계하는데 더 까다롭다.
  - 상속을 위한 클래스를 설계하면 클래스에 상당한 제약이 가해진다. 

