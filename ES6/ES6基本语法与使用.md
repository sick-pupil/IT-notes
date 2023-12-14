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
