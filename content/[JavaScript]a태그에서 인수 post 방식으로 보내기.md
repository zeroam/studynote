## a태그의 함수 호출 방식
#### `<a href="#" onclick="함수();">`
- 함수에 리턴값이 있던 없던 상관 없음
- 그러나 클릭하는 순간 링그 "#" 때문에 페이지의 최상위로 이동됨

#### `<a href="javascript:함수();">`
- 브라우저에 리턴값이 출력되므로 리턴값이 있으면 안됨

#### `<a href="javascript:void(0);" onclick="함수();">`
- 함수에 리턴값이 있던 없던 상관 없음
- 클릭해도 페이지의 최상위로 이동하지 않음

<br/>
a
## a태그에서 인수를 post 방식으로 보내는 방법
```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>a태그 form 전송</title>
    <script>
    // content, cate, index를 인수로 받아 form 태그로 전송하는 함수
    function goPage(content, cate, index = 0) {
      // name이 paging인 태그
      var f = document.paging;

      // form 태그의 하위 태그 값 매개 변수로 대입
      f.content.value = content;
      f.cate.value = cate;
      f.index.value = index;

      // input태그의 값들을 전송하는 주소
      f.action = "./content.do"

      // 전송 방식 : post
      f.method = "post"
      f.submit();
    };
    </script>
  </head>
  <body>
    <!-- 페이지 전송 폼 -->
    <form name="paging">
    	<input type="hidden" name="content"/>
    	<input type="hidden" name="index"/>
    	<input type="hidden" name="cate"/>
    </form>
    <!-- a태그로 인수 전달 -->
    <a href="javascript:goPage('look', 'observation', 1);">관람</a>
  </body>
</html>
```
