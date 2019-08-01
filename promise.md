# promise, async, awiat 을 사용하여 동기화하기

> async 함수는 항상 Promise를 리턴한다
~~~
async function f() {
	return 1;		// return Promise.resolve(1); 과 같다
}

f().then(console.log); //1
~~~  
<br/>  

> promise test
~~~
async function awaitTest() {
	let promise = new Promise(function(resolve, reject) {
		return setTimeout(function(){
			return resolve("done");
		}, 1000);
	});

	let result = await promise;	//wait till the promise resolves()

	console.log(result);
}
awaitTest();
~~~
<br/>  

> 1.nomal function
~~~
function doubleAfter2Seconds(x) {
	setTimeout(function(){
		return x * 2;
	}, 2000);
}

console.log('doubleAfter2Seconds:' + doubleAfter2Seconds(1));	//undefined
~~~
<br/>

> 2.promise function
~~~
function promiseDoubleAfter2Seconds(x) {
	return new Promise(function(resolve, reject) {	
		setTimeout(function(){
			return resolve(x * 2);
		}, 2000);
	});
}

promiseDoubleAfter2Seconds(1).then(function(r) {
	console.log('promiseDoubleAfter2Seconds:' + r);
});
~~~
<br/>

> promise chainning hell~
~~~
promiseDoubleAfter2Seconds(1).then(function(a) {
	promiseDoubleAfter2Seconds(2).then(function(b) {
		promiseDoubleAfter2Seconds(3).then(function(c){
			console.log(a+b+c);
		})
	})
})
~~~
<br/>

> avoid chainning hell using async
~~~
async function addAsync() {
	const a = await promiseDoubleAfter2Seconds(1);
	const b = await promiseDoubleAfter2Seconds(2);
	const c = await promiseDoubleAfter2Seconds(3);
	return a + b + c;
}

addAsync().then(function(r) {
	console.log(r);
});
~~~
