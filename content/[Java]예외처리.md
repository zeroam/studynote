## 예외 처리문의 기본 구조

```java
try { 
	... 
} catch(예외1) { 
	... 
} catch(예외2) { 
	... ... 
}
```
- 다음와 같이 에러가 발생할만한 영역을 try로 감싸고 예외 처리는 catch영역에서 처리한다.
- catch는 여러개가 올 수 있으며, 위에서 부터 아래로 에러처리를 수행한후 예외 처리문을 빠져 나온다

<br>
<br>

### finally
- 예외가 발생하더라도 무조건 실행되어야 하는 블록이 있다면 finally 블록을 활용한다.
```java
try {

} catch (예외) {

} finally {

}
```

<br>
<br>

### 발생하기 쉬운 예외

<b>[ArrayIndexOutOfBoundsException]</b>

-   배열의 인덱스를 잘못 지정 했을때
    
```java
int  arr[] = {1, 2, 3};

int  num  =  arr[3];
```
  
<br>

<b>[NullPointerException]</b>

-   객체를 생성하지 않고 멤버메서드를 호출했을때
    
```java
Calc  c  =  null;

c.add();
```

<br>
<br>

### 예외 발생시키기(throw)
- 연산자 new를 이용해서 예외 클래스의 객체를 만들고
- 키워드 throw를 이용해서 예외를 발생시킨다.
```java
Exception e = new Expception("메시지");
throw e;
```

<br>
<br>

### 예외 던지기(throws)
- 예외가 되는 메서드를 호출한 곳에서 처리할 수록 떠넘김
- 만약 메인메서드에서 throws로 예외를 떠넘기면 자바 가상머신(JVM)에서 예외처리함
```java
public void errorMethod() throws Exception {

}
``` 
