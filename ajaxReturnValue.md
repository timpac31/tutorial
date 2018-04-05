# AJAX에서 반환한 값 활용하는 방법
> ajax 는 비동기 통신이기 때문에 값을 반환하여 다음 작업하는 것이 어렵다.
~~~
var f1 = function() {
    var a;
    $.ajax({
        url:aaa.jsp,
        type:'post',
        success: function(data) {
            a = data.value;
        }
    });
    return a;
}

var a = f1();
console.log(a);
~~~
+ 위와같이하면 비동기 특성 때문에 a값이 undefined가 찍힌다.
<br/><br/>
# A방안 - async 속성 false를 이용
~~~
var f1 = function() {
    var a;
    $.ajax({
        url:aaa.jsp,
        type:'post',
        async:false,
        success: function(data) {
            a = data.value;
        }
    });
    return a;
}

var a = f1();
console.log(a);
~~~
+ 값을 받아올 수 있지만 권장하지 않는다.동기식으로 요청하면 메인 쓰레드가 같이 멈춰서 서버 응답이 늦게 오면 화면 전체가 얼어버린다
+ jquery에서도 deprecated 되었다. deffered를 쓰자
<br/><br/>
# B방안 - callback과 then이용
~~~
var f1 = function(callback) {
    $.ajax({
        url:aaa.jsp,
        type:'post'
    }).then(function(data){
        callback(data.value);
    });
}

f1(function(val) {
  console.log(val);
});
~~~
+함수를 인자로 넘겨서 ajax 통신이 끝난 뒤에 callback 함수를 실행한다.
