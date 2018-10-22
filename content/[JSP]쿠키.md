
## 쿠키란
----
-   웹 브라우저가 보관하는 데이터
-   웹 브라우저는 웹 서버에 요청을 보낼 때 쿠키를 함께 전송
-   웹 서버는 웹브라우저가 전송한 쿠키를 사용해서 필요한 데이터를 읽음
-   쿠키는 웹 서버와 웹 브라우저 양쪽에서 생성할 수 있음
-   JSP에서 생성하는 쿠키는 웹 서버에서 생성하는 쿠키

<br>
<br>
  
### 쿠키 동작 방식

-   쿠키 생성 단계

-   쿠키 저장 단계(웹브라우저 - 쿠키 저장소에 보관)

-   쿠키 전송 단계(요청이 있을 때마다 웹 서버에 전송)


  <br>
  <br>

### JSP 쿠키 생성 코드
```JSP
<%

// 쿠키 객체 생성
Cookie cookie = new Cookie("cookieName", "cookieValue");  

// 쿠키 추가 - response 객체가 웹 브라우저에 쿠키 정보를 전송
response.addCookie(cookie);  

%>
```

 <br>

### 쿠키 값 읽어오기

```
Cookie[] cookies = request.getCookies();
```

  <br>

### 쿠키 값 변경하기

```
Cookie cookie = new Cookie("name", "새로운 값")

response.addCookie(cookie);
```
-   같은 이름의 쿠키를 새로 생성해서 응답 데이터로 보낸다.

    <br>

### 쿠키 삭제하기
```
Cookie cookie = new Cookie(name, value);

cookie.setMaxAge(0);

response.addCookie(cookie);
```
-   같은 이름의 쿠키를 새로 생성한 후

-   유효 시간을 0으로 지정해주고 응답 헤더에 추가

<br>
<hr>

### 참조문헌

최범균(2015). [[최범균의 JSP 2.3 웹 프로그래밍 기초부터 중급까지]](https://book.naver.com/bookdb/book_detail.nhn?bid=9789206) 한빛미디어
