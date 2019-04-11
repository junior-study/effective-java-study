# 아이템 16  public 클래스 안에는 public 필드를 두지 말고 접근자 메서드를 사용하라



```java
class Point {
    public double x;
    public double y;
}
```

데이터의 필드를 직접 조작 할 수 있다. (캡슐화 X )

- 캡상추다!



```java
class Point {
    private double x;
    private double y;
    
    public Point(double x, double y){
        this.x = x;
        this.y = y;
    }
    
    public double getX(){
        return x;
    }
    public double getY(){
        return y;
    }
    public void setX(double x){
        this.x = x;
    }
    public void setY(double y){
        this.y = y;
    }
}
```



- public 클래스는 필드를 외부에 직접 공개 하지 말아야 한다.
  - 1 )따르지 않은 경우가 있어 엄청 심각함. java.awk 패키지 Point, Dimension 
  - 2) 그나마 덜 한건, 변경 불가능 필드



### 1번 케이스

```java
public class Point extends Point2D implements java.io.Serializable {
    /**
     * The X coordinate of this {@code Point}.
     * If no X coordinate is set it will default to 0.
     *
     * @serial
     * @see #getLocation()
     * @see #move(int, int)
     * @since 1.0
     */
    public int x;

    /**
     * The Y coordinate of this {@code Point}.
     * If no Y coordinate is set it will default to 0.
     *
     * @serial
     * @see #getLocation()
     * @see #move(int, int)
     * @since 1.0
     */
    public int y;
```



### 2번 케이스

```java
public final class Time {
    private static final int HOURS_PER_DAY = 24;
    private static final int MINUTE_PER_HOUR = 60;
    
    // public 으로 공개했으나, 변경 불가능 필드임
    public final int hour;
    public final int minute;
    
    // 생략
}
```





## 정리

public 클래스는 변경 가능 필드를 외부로 공개하면 안된다. 2번 케이스처럼 변경 불가능 필드로 외부에 공개 할 순 있지만, 정말 그렇게 해야하는지 의문임. (하지말라는 거) 

