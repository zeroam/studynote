```html
  <script>
  // 함수 안에 재귀함수 형식으로 setTimeout 메서드 실행 - 1초 마다 함수 실행
  function printClock() {
    // id가 clock인 객체 얻기
    var clock = document.getElementById("clock");
    var date = new Date();
    var hours = addZeros(date.getHours(), 2);
    var minutes = addZeros(date.getMinutes(), 2);
    var seconds = addZeros(date.getSeconds(), 2);

    // id가 clock 객체에 텍스트 바꾸기
    clock.innerHTML = hours+":"+minutes+":"+seconds;
    setTimeout("printClock();", 1000)
  };

  // 시간이 한자리 수일 때 0을 붙여 길이 맞춤
  function addZeros(num, digit) {
    var zero = "";
    num = num.toString();
    if(num.length < digit) {
      zero += "0";
    }
    return zero + num;
  };

  // html 문서 로드가 끝날을 때 함수 호출
  window.onload = function() {
    printClock();
  };
  </script>
  
  <body>
    <div id=clock></div>
  </body>
```
