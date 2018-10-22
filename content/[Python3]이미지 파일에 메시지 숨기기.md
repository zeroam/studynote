# 들어가며
<hr>
파이썬에서는 다양한 기능을 제공하는 패키지들이 많다. 그 중 Pillow라는 패키지는 이미지 파일을 다루는 것을 쉽게 만들어 주며, 이미지 파일을 수정하거나, 비트 파일을 수정하며 드러나지 않게 메시지를 숨길 수도 있다. 이번에는 이 패키지를 이용하여 어떻게 비밀메시지를 이미지 파일 속에 숨기며, 확인 할 수 있는지 알아보도록 하겠다.

<br/>
<br/>

- [들어가며](#들어가며)
  - [메시지 숨기기](#메시지-숨기기)
    - [유니코드 메시지를 비트의 시퀀스로 만들기](#유니코드-메시지를-비트의-시퀀스로-만들기)
    - [메시지 인코딩](#메시지-인코딩)
    - [이미지 변경하기](#이미지-변경하기)
  - [메시지 디코딩](#메시지-디코딩)
    - [이미지의 적색 채널에서 비트들을 뽑아오는 추출기 함수](#이미지의-적색-채널에서-비트들을-뽑아오는-추출기-함수)
    - [비트 값을 바이트로 바꾸어 주는 함수들](#비트-값을-바이트로-바꾸어-주는-함수들)
    - [메시지 얻기](#메시지-얻기)

<br/>
<br/>


> [스테가노그래피](https://ko.wikipedia.org/wiki/%EC%8A%A4%ED%85%8C%EA%B0%80%EB%85%B8%EA%B7%B8%EB%9E%98%ED%94%BC)
>
> 스테가노그래피는 데이터 은폐 기술 중 하나이며, 데이터를 다른 데이터에 삽입하는 기술 혹은 그 연구를 가리킨다.


```python
from PIL import Image
import urllib.request
from zipfile import ZipFile
import hmac
import os
```


```python
image_source = "https://cdn.pixabay.com/photo/2018/01/21/19/57/tree-3097419_1280.jpg"
image = Image.open(urllib.request.urlopen(image_source))
```

- 다음과 같이 인터넷 상의 url 주소를 가지고 이미지 파일을 불러올 수 있다.


```python
image
```


![output_5_0](https://i.imgur.com/fDMfgKH.jpg)



본격적으로 이미지를 수정하기 전에 이미지 파일의 정보를 한번 살펴 보도록 하겠다.


```python
#exif(교환가능 이미지 파일 형식) 키 확인
import PIL.ExifTags
try:
    for key, value in image._getexif().items():
        print(PIL.ExifTags.TAGS[key], value)
except:
    print("파일정보가 없습니다.")
```

    Model ILCE-7
    ExifOffset 82
    ExposureTime (1, 20)
    Make SONY
    DateTimeOriginal 2016:01:01 13:31:23
    FocalLength (200, 10)
    LensModel DT 0mm F0 SAM
    ISOSpeedRatings (100, 0)
    Flash (16, 0)
    FNumber (8, 1)


다음과 같이 카메라 모델, 찍은 날짜, 렌즈 등 여러가지 정보를 살펴볼 수 있다. 만약, GPS 데이터를 제공할 수 있는 카메라로 찍은 사진이라면 EXIF에 추가 정보가 들어 있을 것이다.

## 메시지 숨기기
<hr/>
<br>
 이미지 파일에 메시지를 숨기는 방법은 여러가지가 있다. 여기서는 이미지 픽셀의 값 rgb에서 적색채널의 비트를 수정하여 비밀 메시지를 이미지 파일에 숨기도록 할 것이다.

- 이미지 픽셀 얻기 : (x, y) 튜플을 이용해서 해당 좌표 값의 이미지 픽셀을 볼 수 있다.


```python
y = 0
for x in range(5):
    print(image.getpixel((x,y)))
```

    (55, 47, 68)
    (17, 13, 27)
    (53, 51, 56)
    (50, 51, 46)
    (72, 75, 66)


- 어떤 색의 채널인지 확인하기 : 다음과 같이 이미지의 픽셀은 RGB 속성을 사용하는 것을 알 수 있다.


```python
image.getbands()
```




    ('R', 'G', 'B')



### 유니코드 메시지를 비트의 시퀀스로 만들기
 위에서 보았듯이 이미지 파일은 픽셀에 RGB 바이트 값이 들어있는 파일이다. 때문에 우리가 원하는 메시지를 이미지 파일에 삽입하기 위해서는 메시지를 바이트로 바꾸는 과정이 필요하며, 더 나아가 비트 값으로 변환하여 특별한 해독과정 없이는 메시지가 드러나지 않도록 해야한다.


```python
#어떤 수에서 최하위 8비트 뽑아내기
def to_bits(v):
    b = []
    for i in range(8):
        #비트 연산
        b.append(v & 1)
        #v를 1비트 오른쪽으로 시프트
        v = v>>1
    #거꾸로 되어있는 순서 뒤집어서 8-튜플 객체 만들기
    return tuple(reversed(b))
```

위 함수는 1바이트의 값을 받아서 비트로 변환하는 함수이다.
비트 연산을 통해 각각의 비트 값을 얻어서 튜플의 형태로 저장한다.


```python
print(to_bits(255))
```

    (1, 1, 1, 1, 1, 1, 1, 1)


1바이트의 메모리에서는 최대 255까지의 숫자를 저장할 수 있으므로 255를 입력하면 11111111의 값이 튜플 형태로 나온다


```python
#튜플리스트를 받아서 각 요소를 리턴하는 제너레이터 함수
def bit_sequence(tuple_list):
    for t8 in tuple_list:
        for b in t8:
            yield b
```

다음은 8-튜플의 리스트를 받아서 비트의 리스트로 반환하는 제너레이터 함수이다. 제너레이터 함수는 일반 함수와는 달리 값을 메모리에 저장하지 않으므로 처리 속도가 빠르다.

### 메시지 인코딩
위의 함수를 이용해서 임의의 문자열을 비트로 바꿀 수 있다. 이 비트를 통해 메시지를 이미지에 인코딩 할 수 있을 것이다. 하지만 인코딩된 메시지를 디코딩(해독)하는 과정에서 디코딩을 언제 그만 둘 수 있을지 결정할 수 있어야 한다.
여러가지 기법들이 있겠지만, 문자열 맨 앞에 2바이트 크기의 메시지 길이를 넣는 방식을 사용하도록 하겠다. 위와 같은 방식을 통해 최대 256x256 비트 크기(한글의 경우 대략 4000자)의 메시지를 넣을 수 있다.


```python
#삽입할 문자열
message = "hello, world"

#바이트로 인코딩(UFT-8 형식)
message_bytes = message.encode("UTF-8")
#비트로 인코딩
bits_list = list(to_bits(byte) for byte in message_bytes)

#메시지 길이 구하기
len_h, len_l = divmod(len(message_bytes), 256)
#비트로 인코딩
size_list = [to_bits(len_h), to_bits(len_l)]

#최종 비트
total_list = size_list + bits_list
```

### 이미지 변경하기


```python
width, height = image.size
for idx, bit in enumerate(bit_sequence(total_list)):
    y, x = divmod(idx, width)
    r, g, b = image.getpixel((x, y))
    #비트조작 - R값 조작
    r_new = (r & 0xfe) | bit
    image.putpixel((x,y), (r_new, g, b))
```

- 이미지 저장
    - 이미지를 저장할 때, jpeg형식은 파일을 압축하면서 데이터 손실이 발생할 수 있기 때문에 tiff확장자로 저장한다.


```python
image.save("image.tiff")
```

## 메시지 디코딩
<hr/>

메시지 디코딩은 2단계에 걸쳐서 진행된다.
- 메시지 복구에 필요한 2바이트의 길이 정보 디코딩
- 필요한 만큼의 비트 복구

### 이미지의 적색 채널에서 비트들을 뽑아오는 추출기 함수


```python
def get_bits(image, offset=0, size=16):
    width, height = image.size
    for pos in range(offset, offset+size):
        y, x = divmod(pos, width)
        r, g, b = image.getpixel((x,y))
        # R값의 최하위 1비트 추출
        yield r & 0x01
```

### 비트 값을 바이트로 바꾸어 주는 함수들


```python
#8-튜플에서 원래의 바이트를 복구
def to_byte(bits):
    byte = 0
    for bit in bits:
        byte = (byte<<1) | bit
    return byte


#비트의 시퀀스를 8-튜플로 모으고 원래의 바이트 복구
def byte_sequence(bits):
    byte = []
    for idx, bit in enumerate(bits):
        #맨 첫비트 고려 후 8비트씩 끊기
        if idx%8 == 0 and idx != 0:
            yield to_byte(byte)
            byte = []
        byte.append(bit)
    #맨 마지막 8비트를 바이트로
    yield to_byte(byte)
```

### 메시지 얻기


```python
#이미지 불러오기
image = Image.open("image.tiff")

#메시지 크기 - 첫 2바이트
size_h, size_l = byte_sequence(get_bits(image, 0, 16))
size = 256*size_h + size_l

#메시지
message_byte_generator = byte_sequence(get_bits(image, 16, size*8))
message_get = bytes(message_byte_generator).decode("UTF-8")

print(message_get)
```

    hello, world


- 다음과 같이 숨겨놓은 메시지를 확인할 수 있다.
