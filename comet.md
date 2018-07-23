# COMET
> push server 방법
1. polling
> 주기적으로 서버에 요청
~~~
setInterval(function(){
    $.ajax({ url: "server", success: function(data){
        //TODO 
    }, dataType: "json"});
}, 30000);
~~~
2. long polling
> 요청을 서버에서 가지고 이벤트를 발생할때까지 일정시간 가지고 있는다
~~~
(function poll() {
   setTimeout(function() {
       $.ajax({ 
            url: "server",
            success: function(data) {
              //TODO
            }, dataType: "json", complete: poll });
    }, 30000);
})();
~~~
3. websocket
> 양방향통신  
+ Support : java1.6+, tomcat7+, jeus 7 fix3+, IE10+, HTML5
4. Server Sent Event
~~~
if(typeof(EventSource) !== "undefined") {
    var source = new EventSource("demo_sse.php");
    source.onmessage = function(event) {
        document.getElementById("result").innerHTML += event.data + "<br>";
    };
} else {
    document.getElementById("result").innerHTML = "Sorry, your browser does not support server-sent events...";
}
~~~
~~~
<?php
header('Content-Type: text/event-stream');
header('Cache-Control: no-cache');

$time = date('r');
echo "data: The server time is: {$time}\n\n";
flush();
?>
~~~
