## 1. let、const
相较于`ES5`的`var`，`ES6`新增了两个重要的变量关键字：`let`与`const`，块级作用域变量声明

`ES5`的`var`：在全局作用域或者局部作用域声明`var`变量，变量都会被提升至该作用域的最顶层
- 在全局作用域内：`var`声明的全局作用域中的变量会被挂载到`window`对象上，可能会覆盖`window`对象中的某个属性
- 在局部作用域内：`var`变量声明语句前也能被访问，只不过值为`undefined`

`ES6`的`let`：`let`声明的变量只在声明语句所在的代码块有效，不能重复声明，且不会出现变量提升
`ES6`的`const`：`const`声明的变量为一个只读变量，声明必须初始化且不允许改变

经典例子：`for`循环中的闭包作用域
```js
//此时的i在每次迭代时都是使用的全局i
for(var i = 0; i < 5; i++){
	setTimeout(function(){
    	console.log(i)
    })
}

//每次循环产生一个闭包作用域，并把每次迭代产生的i值赋给j
for(var i = 0; i < 5; i++){
	(function(j){
      setTimeout(function(){
          console.log(j)
      })
    })(i)
}

//每次循环产生子作用域，let变量的每次迭代值存入每个子作用域中
for(let i = 0; i < 5; i++){
    setTimeout(function(){
        console.log(i)
    })
}
```

## 2. 解构赋值
```js
//数组解构
let [a, b, c] = [1, 2, 3]
let [a, [[b], c]] = [1, [[2], 3]]
let [a, , b] = [1, 2, 3]
let [a, ...b] = [1, 2, 3]

//对象解构
let { foo, bar } = { foo: 'aaa', bar: 'bbb' }
let { baz : foo } = { baz : 'ddd' }

let obj = {p: ['hello', {y: 'world'}] }
let {p: [x, { y }] } = obj

let obj = {p: ['hello', {y: 'world'}] }
let {p: [x, { }] } = obj;
```

## 3. 模板字符串
使用反撇号定义
```js
// 普通字符串
`In JavaScript '\n' is a line-feed.`

// 多行字符串
`In JavaScript this is
 not legal.`
 
console.log(`string text line 1
string text line 2`);

// 字符串中嵌入变量
let name = "Bob", time = "today";
`Hello ${name}, how are you ${time}?`
```

## 4. 对象简化写法
```js
let name = 'abc'
let change = function() {
	console.log('change')
}

const obj = {
	name : name
	change : change
	customFunction : function() {...}
}
//简化写法
const obj = {
	name
	change
	customFunction() {...}
}
```

## 5. 箭头函数
```js
let fn1 = function(a, b) {return a + b}
let fn2 = (a, b) => {return a + b}

//箭头函数调用中的this永远是函数定义所在作用域中的this
fn1.call(1, 2)
fn2.call(1, 2)//fn2.call()无法改变上下文this

//箭头函数不能作为类的构造函数
let Obj = (a, b) => {
	a = a
	b = b
}//报错

//箭头函数不能使用arguments
let fn = () => {
	console.log(arguments)
}//报错

//简写
let functionArrow = n => n*n
```

## 6. 函数参数默认值
```js
function add(a ,b, c = 10) {
	return a + b + c
}

function multiply({a = 10, b, c}) {
	return a * b * c
}
```

## 7. rest函数参数
```js
function log(...args) {
	console.log(args)
}
```

## 8. Symbol
新的原始数据类型，表示独一无二，类似字符串
1. 值唯一，用来解决命名冲突的问题
2. 值不能与其他数据进行运算
3. 定义的对象属性不能使用`for...in`循环遍历

```js
let s1 = Symbol()

let s2 = Symbol('a')
let s3 = Symbol('b')

let s4 = Symbol.for('c')
let s5 = Symbol.for('c')
```

*原始数据类型：undefined string symbol object null number boolean*

## 9. 迭代器
```js
const arr = ['a', 'b', 'c']
for(let i of arr) {
	console.log(i)
}
//原理与链表一样
let iter = arr[Symbol.iterator]()
iter.next()

//自定义迭代器
const obj = {
	name : 'obj',
	arr : [
		'a',
		'b',
		'c',
		'd'
	],
	[Symbol.iterator]() {
		let index = 0
		let _this = this
		return {
			next: function() {
				if(index < _this.arr.length) {
					const result = {value: _this.arr[i], done: false}
					index++
					return result
				} else {
					return {value: undefined, done: true}
				}
			}
		}
	}
}
```

## 10. 生成器
```js
function * createIter() {
	console.log('iterator next')
	//yield 'iterator next1'
	//yield 'iterator next2'
	//yield 'iterator next3'
}
let iter = createIter()
iter.next()//每一次执行next，就会执行一次yield
```

## 11. Promise
`ES6`引入的异步编程，`Promise`本质上是一个构造函数，用来封装异步操作获取成功或失败的结果
```js
//resolve与reject是两个函数，调用resolve后Promise对象为成功状态，调用reject后Promise对象为失败状态
const p = new Promise(function(resolve, reject) {
	//异步操作
	setTimeout(function() {
		let data = ''
		resolve(data)
		//reject(err)
	}, 1000)
})

p.then(function(value) {
	//该回调函数为调用resolve函数后的成功回调
	//return '123'如果回调存在返回，则返回值会被封装进Promise对象中
	//Promise的resolve与reject函数返回值都是Promise对象
	//return new Promise((resolve, reject) => {
		//resolve('ok')
		//reject('fail')
	//})
	console.log(value)
}, function(err) {
	//该回调函数为调用reject函数后的失败回调
	console.log(err)
})

//链式调用
p.then(value => {
	//...
}).then(value => {
	//...
})

//catch
p.catch(err => {
	console.warn(err)
})
```

## 12. Set与Map
```js
//与Java的set类似，非重复
let s = new Set(['a', 'b', 'c', 'd'])
console.log(s)

s.size
s.add('abc')
s.delete('a')
s.has('b')
s.clear()

for(let v of s) {
	console.log(v)
}
```

```js
//与Java的map类似，key-value
let m = new Map()
m.set('1', 'a')
m.set('2', 'b')
m.set('3', function() {
	console.log('function')
})

let key = {a : 'abc'}
m.set(key, ['1', '2', '3'])

m.size
m.delete('1')
m.get('2')
m.clear()

for(let v of m) {
	console.log(v)
}
```

## 13. Class
```js
//ES5
function ClassInstance(arg1, arg2) {
	this.arg1 = arg1
	this.arg2 = arg2
}

Clazz.prototype.func1 = function() {
	console.log('执行方法')
}

let classInstance = new ClassInstance('a', 'b')
classInstance.func1()
console.log(classInstance)

//ES6
class Class1 {
	//类中的静态属性与方法
	static name = 'abc'
	static func2() {
		console.log('abc')
	}

	constructor(arg1, arg2) {
		this.arg1 = arg1
		this.arg2 = arg2
	}

	func1() {
		console.log('执行方法')
	}
}

class Class2 extends Class1 {

	constructor(arg1, arg2) {
		super(arg1, arg2)
	}

	func11() {
		console.log('执行方法')
	}

	//get set实例属性方法
	get field() {
		return 'abcd'
	}

	set field(newVal) {
		console.log('set')
	}
}

let class2Instance = new Class2()
s.field = 'abcdefg'

let classInstance1 = new Class1('1', '2')
console.log(classInstance1)
```

## 14. 模块化
模块化指一个大程序文件，拆分成多个小文件，再将多个小文件组合起来
主要通过命令`export`暴露模块对外接口，命令`import`引入其他模块暴露的接口

```js
export let field = 'abc'

export function func() {
	console.log('abcdefg')
}

export default {
	field1 : 'abc'
	func1 : function() {
		console.log('abcdefg')
	}
}
```

```js
<script type='module'>
	import * as m1 from '../src/m1.js'
	//import {func1, field1} from '../src/m1.js'
	console.log(m1)
</script>
```

## 15. Babel
将ES6语法转为浏览器适配的js语法