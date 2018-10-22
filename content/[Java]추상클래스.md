## 추상클래스

추상(abstract)란 사물들 간의 공통된 특성을 추출한 것을 말한다. 어떠한 실체들의 공통되는 특성을 추출하여 정의해 놓는 것이다.
 추상 클래스 또한 클래스들의 공통된 특성을 추출해서 선언한 클래스이다. 객체를 직접 생성할 수 있는 실체 클래스와는 달리 추상클래스는 객체를 직접 생성할 수 없다.

<br>

### 추상 클래스의 선언
```java
public abstract class 클래스 {... }
```
다음과 같이 클래스 앞에 abstract 키워드를 붙인다.

<br>

### 추상 메서드
멤버 메서드가 구현이 되어 있지 않고 선언만 되어 있는 형태
```java
[public|protected] abstract 리턴타입 메소드명(매개변수, ...);
```
리턴 타입 앞에 abstract 키워드를 붙인다.

<br>

### 추상 메서드의 오버라이딩
```java
public abstract class Animal {
	public abstract void sound() // 추상메서드 
}
```
```java
public class Dog extends Animal { // 추상 클래스의 상속
	public Dog() {
		this.kind = "포유류";
	}

	@Override
	public void sound() { // 추상메서드의 오버라이딩
		System.out.println("멍멍"); 
	}
}
```
Animal 클래스는 추상 클래스이고, sound라는 메서드는 Animal객체에서 선언한 추상 메서드이다. Animal 클래스를 상속하는 Dog 클래스는 추상 메서드를 위와 같이 오버라이딩 함으로서 사용가능하다.

<br>
<br>

### 추상클래스의 특징

- 최소 1개 이상의 추상 메서드를 갖는다
- 상속을 위한 클래스로서 부모클래스이다.
- 클래스 설계의 표준 가이드 역할을 제공한다
- 추상 클래스는 객체 생성이 불가능 하다


