
### 가상콘솔
- 가상의 모니터
- 우분투는 7개의 가상 콘솔을 제공
- 단축키는 ctrl + alt + f1~f7(f7은 x 윈도우 모드)
- logout(로그아웃)

<br>


### 런 레벨(Runlevel)
- init 명령어 뒤에 붙는 숫자
- `init 0` - Power Off - 종료 모드
- `init 1` - Rescue - 시스템 복구 모드, 단일 사용자 모드
- `init 3` - Multi-User - 텍스트 모드의 다중 사용자 모드
- `init 5` - Graphic - 그래픽 모드의 다중 사용자 모드
- `init 6` - Reboot - 재부팅
- run 레벨 확인
`ls -l /lib/systemd/system/default.target`

<br>

### 자동 완성과 히스토리
- 파일명의 일부만 입력한 후에 Tab키를 눌러서 나머지 파일명을 자동으로 완성하는 기능
- `history`
	- 지금까지 썻던 명령어들을 전부 확인하는 명령어
- `history -c`
	- history 기록들 모두 삭제
<br/>

<hr>

### 참조문서
- [우재남(2017) [이것이 우분투 리눅스다] 한빛 미디어](http://www.hanbit.co.kr/store/books/look.php?p_code=B6815940952)
