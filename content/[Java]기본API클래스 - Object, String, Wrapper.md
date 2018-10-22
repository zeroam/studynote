## Object 클래스

-   자바의 모든 클래스는 Object 클래스를 상속 받는다.
-   클래스를 선언할 때 extends 키워드로 다른 클래스를 상속하지 않으면 암시적으로 java.lang.Object 클래스를 상속하게 된다
-   멤버 메소드
	-   equals() - 객체 비교
    
<br>
<br>
<br>
 
## String 클래스

-   String 문자열 = 문자 + 배열   
-   new -> 새로 객체 생성
-   리터럴 방식으로 저장 -> 같은 문자열이면 같은 주솟값 공유, 같은 객체 참조
-   멤버 메소드
	- equals() - 객체의 문자열 값 비교
	-   length() - 문자 갯수
	-   substring(start, end) - 문자열 자르기 start 부터 end-1까지
	-   charAt(index) - index의 문자 추출
	-   indexOf("문자") - 문자열의 첫번째 인덱스값
	-   lastIndexOf("문자") - 문자열의 마지막 인덱스 값
	-   replace("old", "new") - 문자열 교체
	-   trim() - 앞뒤 공백 제거
	-   String.valueOf(값) - 기본 타입의 값을 문자열로 변환
	-   "" + 변수 - 기본 타입의 값을 문자열로 변환
    
<br>
<br>
<br>
  
## Wrapper 클래스

-   기본 타입의 값을 갖는 객체 생성 - 포장 객체
-   기본 타입의 값을 stack 영역이 아닌 heap 영역에 저장
-   포장하고 있는 기본 타입의 값은 외부에서 변경할 수 없다
-   박싱(Boxing) - 기본 타입의 값을 포장 객체로 만드는 과정
```java
Integer obj = new Integer(5); // 생성자 이용

Integer obj = Integer.valueOf(5); // valueOf 메서드 이용
```
 - -  자동 박싱

```java
Integer obj = 100; // 자동 박싱
```
-   언박싱(Unboxing) - 포장 객체에서 기본 타입의 값을 얻어내는 과정
    

	-   자동 언박싱
    
```java
Integer obj = new Integer(200);

int value1 = obj; //자동 언박싱 - int 타입에 대입

int value2 = obj + 100; //자동 언박싱 - Integer 객체와 int타입 값을 연산
```
  
  <br>

-   Wrappter 클래스의 용도
	
1. 문자열을 기본 타입으로 변환(parse~)
```java
int num = Integer.parseInt("1000");
```

2. 문자열을 wrapper 타입으로 변환(valueOf)
```java
Integer obj = Integer.valueOf("1000");
```
3.   wrapper 타입을 문자열로 변환(toString)
```java
String str = obj.toString();
```

<br>


기본타입 | 포장클래스
-------- | -----
byte | Byte
char | Character
short | Short
int | Integer
long | Long
float | Float
double | Double
boolean | Boolean

<br>

<hr>

### 참고문헌

- 신용권(2015).「이것이 자바다」한빛미디어
