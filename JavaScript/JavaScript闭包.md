# JavaScript闭包

闭包是指有权访问另一个函数作用域中变量的函数，闭包可以作为一个参数传入函数中，或者作为一个函数返回值进行返回

```javascript
function fn1(fn) {
	fn()
}

function fn2() {
	return function fn() {}
}
```

### 闭包的特点

- 可以让外部访问函数内部的变量
- 变量会常驻内存中
- 可以避免使用全局变量，防止全局变量污染

### 闭包优缺点

好处：可以读取其他函数的变量，并一直保存在内存中

坏处：闭包会导致变量不能被垃圾回收机制回收，滥用闭包可能会导致内存泄漏或者溢出

### 闭包用处

1. 创建私有变量

   ```javascript
   function fn() {
   	const a = 'hello'
   	return function() {
   		console.log(a)
   	} 
   }
   
   const f = fn()
   ```

2. 间接访问函数内部的变量

   ```javascript
   for(var i = 0; i < 10; i++) {
       (function(j) {
           setTimeout(function(){
               console.log(j)
           })
       })(i)
   }
   ```

   
