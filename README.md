# intercept

## 功能

- 在目标函数调用前可以修改参数
- 在目标函数调用完成之后可以修改返回值
- 支持异步的函数的拦截
- 支持 before、after、exception 类型的拦截

## 思路

使用 Object.create 来新建一个新的对象来继承 原有的对象。在子对象里面实现对父对象的方法与属性进行拦截。对于拦截器的组织是先搜集所有的拦截器在一起，然后在执行的时候展开，这样可以自定义拦截器的顺序。

## 使用

```
var Http = {
	send : function(data, cb){
		setTimeout(function(){
			console.log(data);
			cb && cb();
		}, 10);
	},
	variable : 10
}

Http = ObjectDecorator.getObjectDecorator(Http).proxy;

var od = ObjectDecorator.getObjectDecorator(Http);

od.interception('variable', { after : function(inv, next){
	inv.result = 11;
	console.log('拦截 variable 属性');
	next();

},index : 1, desc : 'callback2' });

Http.variable = 1;
// console.log(Http.variable);

od.interception('send', function(inv, next){
		console.log('begin1');
		next();
	},
	function(inv, next){
		console.log('end1');
		next(true); // true end2 不执行
	},
	null, 1, 'wo de 1'
);

od.interception('send', function(inv, next){
		console.log('begin2');
		next();
	},
	function(inv, next){
		console.log('end2');
		next();
	},
	null, 2, 'wo de 2'
);

od.interception_args('send', 1, { before : function(inv, next){

	console.log('拦截1');
	next();

}, 
index : 2,
desc : 'callback1' });

od.interception_args('send', 1, { after : function(inv, next){

	console.log('拦截2');
	next();

},index : 1, desc : 'callback2' });

Http.send({actid:123}, function(a,b){ console.log('callback1') });
```
