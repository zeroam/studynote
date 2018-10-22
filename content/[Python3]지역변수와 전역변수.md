## 지역변수와 전역변수란

코드를 작성하다 보면 코드의 줄이 늘어나게 되고 변수의 이름과 같은 것은 중복이 발생할 수 있다. 이 때 잘못 정의하거나 초기화를 하여 프로그램 전체에 지대한 영향을 미칠 수 있는데 이를 방지하기 위해 기본적으로 함수내에서는 함수 내에서만 처리하고 함수 밖에 영향을 미치지 않는 변수인 *지역변수*와 함수 밖에서 선언되어 자유롭게 사용할 수 있는 *전역변수*로 나누었다.

이를 간단히 정리하면 다음과 같다.
- 전역변수(global variable) : 함수 밖에서 정의되어 프로그램 어디서든 부를 수 있는 변수
- 지역변수(local variable) : 함수 내에서 정의되어 그 함수 내에서만 부를 수 있는 변수

지역변수와 전역변수는 사용에 있어서 그 차이점이 존재하는데 구체적으로 살펴보도록 하겠다.

- #### 전역변수의 참조
```python
x = 5 # 전역변수

def example():
	print(x)
	print(x+5) # 다음과 같이 전역변수 참조 가능
	z = 5 # 지역변수
	print(z)
```

- #### 함수 내에서 전역변수의 수정
```python
x = 5

def example():
	print(x)
	print(x+5)

	#x += 6  불가능
	global x; # global 선언을 통해 수정가능
	x += 6
	print(x) #결과값 11
print(x)  #결과값 11
```
 전역변수는 함수 내에서 수정이 불가능하며 global선언을 통해 수정이 가능하며 수정된 값은 함수를 빠져나오더라도 유지된다

- #### 함수 밖에서 지역변수의 참조
```python
x = 5
def example():
	x = 10 # 지역변수
	y = 5 # 지역변수
print(x) # 결과값 5
print(y) # 불가능
```
함수 안에서 x를 재정의 하더라도 x는 지역변수로 취급되며 함수 밖에서 정의된 전역변수 x에 영향을 주지 않는다. y또한 함수를 빠져 나가면 사용할 수 없기 때문에 y를 참조하는 print문은 에러를 발생시킨다.

- #### global문의 사용
 global함수로 함수내에서 전역변수를 수정하는 것은 좋은 방법이 아니다. 그보다는 함수의 반환값을 이용해서 수정하는 것이 낫다.
```python
x = 6

def example(modify):
	print(modify) # 결과값 6
	modify += 5 # 수정가능
	print(modify) # 결과값 11
	return modify
x = example(x)
print(x) # 결과값 11
```


### 참조자료
[Global and Local Variables Python Tutorial](https://pythonprogramming.net/global-local-variables/)
[파이썬 프로그래밍 입문서 - 3.4 전역변수와 지역변수](https://python.bakyeono.net/chapter-3-4.html)
