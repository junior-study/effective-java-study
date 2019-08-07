> 아이템 77. 예외를 무시하지 말라 

예외를 무시하기 쉽다.  try문으로 감싸고 catch 블록에서 아무일도 하지 않으면 끝이다. catch 블록을 비워두면 예외가 존재할 이유가 없어진다. 물론 예외를 무시해야 할 때도 있다. FileinputStream을 닫을 때가 그렇다. (입력 전용 스트림이므로) 파일의 상태를 변경하지 않았으니 복구할 것이 없으며, (스트림을 닫는다는 건) 필요한 정보는 이미 다 읽었다는 뜻이니 남은 작업을 중단할 이유도 없다. 어쨋든 **예외를 무시하기로 했다면 catch 블록 안에 그렇게 결정한 이유를 주석으로 남기고 예외 변수의 이름도 ignored로 바꾸자** 



```java
int numColors = 4;
try {
  numColors= f.get(1L, TimeUnit.SECONDS);
}catch (TimeoutExcepiton | ExecutionException ignored){
  // 기본값을 사용한다(색상수를 최소화하면 좋지만, 필수는 아니다)
  
}
```

이번 절의 내용은  빈 catch 블록을 못본척 지나치면 나중에 프로그램은 오류를 내재한 채 동작하다가 죽게된다. 