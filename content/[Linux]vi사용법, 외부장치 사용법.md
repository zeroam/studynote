### 에디터
---
`gedit` - 윈도우의 메모장과 같은 편집기, 초심자가 문서를 편집할 때 보기 편하다

`vi` - 리눅스의 편집기 중 하나, 처음에 배우기 어렵지만 익숙해지면 다른 편집기 보다 더 편하다.
-  명령모드
    - `:q` - 끝내기
    -  `:wq` - 저장 후 종료
    - `:q!` - 저장 안하고 종료
    - `i 또는 a` - 입력모드
      - I (맨앞), A(맨뒤)
    - `Esc` - 명령모드 복귀
	- `x` - 한글자 삭제(del)
	- `X` - 한글자 삭제(backspace)
	- `dd` - 현재 행 삭제 + 숫자(숫자만큼)
	- `yy` - 현재 행 복사 + 숫자
	- `p` - 붙여넣기
	- `:set number` - 행번호 표시
  - 비정상 종료시
    - ``.파일명`` 파일 삭제

<br/>
<br/>

### 마운트
---
- CD/DVD 마운트(/dev/cdrom or /dev/sr0 이 /media/cdrom 으로 올라감)
	* `mount` - 현재 연결된 장치 목록 확인
	* `umount /dev/cdrom` - cdrom 연결 끊기
	* `mount   /dev/cdrom    /media/cdrom`

<br/>

- USB(/dev/sdb1 이 /media/~ 으로 올라감)
  * sda는 하드 디스크
  * `mount    /dev/sdb1    /media/usb`

<br>


- 리눅스에서 ISO 파일을 제작, 사용
	* genisoimage 패키지가 설치 되어있는지 확인
  - 설치되어 있지 않으면 `apt-get install genisoimage`로 설치
  - `genisoimage -r -J -o boot.iso /boot` - boot 폴더 밑의 모든 파일 boot.iso파일로 생성
  - `mount -o loop boot.iso /media/iso/` -boot.iso 파일을 /media/iso 파일에 마운트

  <hr>

  ### 참조문서
  - [우재남(2017) [이것이 우분투 리눅스다] 한빛 미디어](http://www.hanbit.co.kr/store/books/look.php?p_code=B6815940952)
