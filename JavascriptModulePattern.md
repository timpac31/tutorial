# Javascript Module Pattern
1.
~~~
var myUtil = (function(myUtil, $, undefined) {
    var name = 0;
    
    function add() {    //private
        i++;
    };
    
    myUtil.getName = function() {    //public
        console.log(i++);
    };
 
    return myUtil;
})(window.myUtil || {}, jQuery);
~~~

2.
~~~
var myUtil = (function() {
    var privateKey = 0; 
 
    function privateMethod() { 
        return ++privateKey; 
    } 
 
    return { 
        publickey : privateKey,
        publicMethod : function () { 
            return privateMethod(); 
        } 
    }
})();
 
myUtil.privateKey //undefined
myUtil.publickey //0
myUtil.publicMethod()    //1
~~~

3.
~~~
(function(){
    function _myUtil(){
        var api={};
        
        api.param= {q:null,fromDt:null,toDt:null,pageNo:null,display:null,sort:null,nstCd:null};
 
        api.getList = function(callback){
            if(성공) callback({result: "" .... }, null);
            else callback(null, error);
        };
        api.getDetail = function(callback){
            callback(success, error);
        };
 
        return api;
    }
})();
 
//사용
_myUtil.getList(function(success, error){
    if(error) {
        console.log(error);
        return;
    }
 
    cosole.log(success);
});
~~~
