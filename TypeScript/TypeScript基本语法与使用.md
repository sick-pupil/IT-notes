## 1. 下载安装
1. 使用`NVM`安装多个`nodejs`版本，如`18.18.2`、`16.14.0`、`8.9.4`
2. 设置每个版本`nodejs`的全局`prefix`与`cache`：`npm config set prefix`和`npm config set cache`为同一目录
3. `npm install -g typescript`全局安装`typescript`，使用命令`tsc -v`查看`typescript`是否正常安装
4. 设置`vscode`启动时都以管理员身份启动
5. 在`vscode`终端中执行：`get-ExecutionPolicy`，如果显示`Restricted`则再执行`set-ExecutionPolicy RemoteSigned`，直至正确显示`RemoteSigned`
6. 根据`typescript`语法写一个`.ts`文件，使用命令`tsc xxx.ts`将`ts`文件编译`js`文件，再执行`node xxx.js`文件
## 2. 基础
`TypeScript`是`JavaScript`的超集，扩展了`JavaScript`的语法，因此现有的`JavaScript`代码可与`TypeScript`一起工作无需任何修改，`TypeScript`通过类型注解提供编译时的静态类型检查
`TypeScript`可处理已有的`JavaScript`代码，并只对其中的`TypeScript`代码进行编译
## 3. 基础类型

|   |   |   |
|---|---|---|
| 任意类型 | `any` | 声明为 `any` 的变量可以赋予任意类型的值 |
| 数字类型 | `number` | 双精度 64 位浮点值。它可以用来表示整数和分数 |
| 字符串类型 | `string` | 一个字符系列，使用单引号（'）或双引号（"）来表示字符串类型。反引号（\`）来定义多行文本和内嵌表达式 |
| 布尔类型 | `boolean` | 表示逻辑值：`true` 和 `false` |
| 数组类型 | 无 | 声明变量为数组 |
| 元组 | 无 | 元组类型用来表示已知元素数量和类型的数组，各元素的类型不必相同，对应位置的类型需要相同 |
| 枚举 | `enum` | 枚举类型用于定义数值集合 |
| `void` | `void` | 用于标识方法返回值的类型，表示该方法没有返回值 |
| `null` | `null` | 表示对象值缺失 |
| `undefined` | `undefined` | 用于初始化变量为一个未定义的值 |
| `never` | `never` | `never` 是其它类型（包括 `null` 和 `undefined`）的子类型，代表从不会出现的值 |

**例**：
```ts
let binaryLiteral: number = 0b1010; // 二进制
let octalLiteral: number = 0o744;    // 八进制
let decLiteral: number = 6;    // 十进制
let hexLiteral: number = 0xf00d;    // 十六进制

let name: string = "Runoob";
let years: number = 5;
let words: string = `您好，今年是 ${ name } 发布 ${ years + 1} 周年`;

let flag: boolean = true;

// 在元素类型后面加上[]
let arr: number[] = [1, 2];
// 或者使用数组泛型
let arr: Array<number> = [1, 2];

let x: [string, number];
x = ['Runoob', 1];    // 运行正常
x = [1, 'Runoob'];    // 报错
console.log(x[0]);    // 输出 Runoob

enum Color {Red, Green, Blue};
let c: Color = Color.Blue;
console.log(c);    // 输出 2

function hello(): void {
    alert("Hello Runoob");
}

//任意值是 TypeScript 针对编程时类型不明确的变量使用的一种数据类型
let x: any = 1;    // 数字类型
x = 'I am who I am';    // 字符串类型
x = false;    // 布尔类型

//undefined 是一个没有设置值的变量
//Null 和 Undefined 是其他任何类型（包括 void）的子类型，可以赋值给其它类型，
//赋值后的类型会变成 null 或 undefined
//而在TypeScript中启用严格的空校验（--strictNullChecks）特性，就可以使得null 和 undefined 只能被赋值给 void 或本身对应的类型
// 启用 --strictNullChecks
let x: number;
x = 1; // 编译正确
x = undefined;    // 编译错误
x = null;    // 编译错误
// 启用 --strictNullChecks
let x: number | null | undefined;
x = 1; // 编译正确
x = undefined;    // 编译正确
x = null;    // 编译正确

//never 是其它类型（包括 null 和 undefined）的子类型，代表从不会出现的值。这意味着声明为 never 类型的变量只能被 never 类型所赋值，在函数中它通常表现为抛出异常或无法执行到终止点
let x: never;
let y: number;
// 编译错误，数字类型不能转为 never 类型
x = 123;
// 运行正确，never 类型可以赋值给 never类型
x = (()=>{ throw new Error('exception')})();
// 运行正确，never 类型可以赋值给 数字类型
y = (()=>{ throw new Error('exception')})();
// 返回值为 never 的函数可以是抛出异常的情况
function error(message: string): never {
    throw new Error(message);
}
// 返回值为 never 的函数可以是无法被执行到的终止点的情况
function loop(): never {
    while (true) {}
}
```
## 4. 变量声明
格式：
1. `var | let | const [变量名] : [类型] = 值`，初始化并赋予类型与值
2. `var | let | const [变量名] : [类型]`，初始化并赋予类型，但没有赋值
3. `var | let | const [变量名] = 值`，初始化变量并赋值，但没有赋予类型
4. `var | let | const [变量名]`，初始化变量，并没有赋值与赋予类型

变量存在三个作用域：
1. 全局作用域，声明在方法与类，块外部
2. 类作用域，声明在类中，可以是静态或者实例变量
3. 局部作用域，声明在一个代码块（方法）内
## 5. 运算符
算术运算符：

|   |   |   |   |   |
|---|---|---|---|---|
|+|加法|x=y+2|7|5|
|-|减法|x=y-2|3|5|
|*|乘法|x=y*2|10|5|
|/|除法|x=y/2|2.5|5|
|%|取模（余数）|x=y%2|1|5|
|++|自增|x=++y|6|6|
|x=y++|5|6|
|--|自减|x=--y|4|4|
|x=y--|5|4|

关系运算符：

|   |   |   |
|---|---|---|
|\=\=|等于|x\=\=8|
|!=|不等于|x!=8|
|>|大于|x>8|
|<|小于|x<8|
|>=|大于或等于|x>=8|
|<=|小于或等于|x<=8|

逻辑运算符

|   |   |   |
|---|---|---|
|&&|and|(x < 10 && y > 1) 为 true|
|\||or|(x\=\=5 \| y\=\=5) 为 false|
|!|not|!(x\=\=y) 为 true|

赋值运算符：

|   |   |   |   |
|---|---|---|---|
|= (赋值)|x = y|x = y|x = 5|
|+= (先进行加运算后赋值)|x += y|x = x + y|x = 15|
|-= (先进行减运算后赋值)|x -= y|x = x - y|x = 5|
|\*= (先进行乘运算后赋值)|x \*= y|x = x \* y|x = 50|
|\/\= (先进行除运算后赋值)|x /= y|x = x / y|x = 2|

三元运算符：`Test ? expr1 : expr2`

类型运算符：`typeof variable`
## 6. 条件、循环
```ts
var num:number = 2 
if(num > 0) { 
    console.log(num+" 是正数") 
} else if(num < 0) { 
    console.log(num+" 是负数") 
} else { 
    console.log(num+" 不是正数也不是负数") 
}

var grade:string = "A"; 
switch(grade) { 
    case "A": { 
        console.log("优"); 
        break; 
    } 
    case "B": { 
        console.log("良"); 
        break; 
    } 
    case "C": {
        console.log("及格"); 
        break;    
    } 
    case "D": { 
        console.log("不及格"); 
        break; 
    }  
    default: { 
        console.log("非法输入"); 
        break;              
    } 
}


var num:number = 5; 
var i:number; 
var factorial = 1; 
for(i = num;i>=1;i--) {
   factorial *= i;
}
console.log(factorial)


var j:any; 
var n:any = "a b c" 
for(j in n) {
    console.log(n[j])  
}


let list = [4, 5, 6];
list.forEach((val, idx, array) => {
    // val: 当前值
    // idx：当前index
    // array: Array
});


let list = [4, 5, 6];
list.every((val, idx, array) => {
    // val: 当前值
    // idx：当前index
    // array: Array
    return true; // Continues
    // Return false will quit the iteration
});


var num:number = 5; 
var factorial:number = 1; 
while(num >=1) { 
    factorial = factorial * num; 
    num--; 
} 
console.log("5 的阶乘为："+factorial);


var n:number = 10;
do { 
    console.log(n); 
    n--; 
} while(n>=0);


var i:number = 1 
while(i<=10) { 
    if (i % 5 == 0) {   
        console.log ("在 1~10 之间第一个被 5 整除的数为 : "+i) 
        break     // 找到一个后退出循环
    } 
    i++ 
}  // 输出 5 然后程序执行结束
```
## 7. 函数
```ts
function test() {   // 函数定义
    console.log("调用函数") 
} 
test()              // 调用函数


// 函数定义
function greet():string { // 返回一个字符串
    return "Hello World" 
}
function caller() { 
    var msg = greet() // 调用 greet() 函数 
    console.log(msg) 
} 
// 调用函数
caller()


function add(x: number, y: number): number {
    return x + y;
}
console.log(add(1,2))


//可选参数
function buildName(firstName: string, lastName?: string) {
    if (lastName)
        return firstName + " " + lastName;
    else
        return firstName;
}
 
let result1 = buildName("Bob");  // 正确
let result2 = buildName("Bob", "Adams", "Sr.");  // 错误，参数太多了
let result3 = buildName("Bob", "Adams");  // 正确


//默认参数
function calculate_discount(price:number,rate:number = 0.50) { 
    var discount = price * rate; 
    console.log("计算结果: ",discount); 
} 
calculate_discount(1000) 
calculate_discount(1000,0.30)


//剩余参数
function addNumbers(...nums:number[]) {  
    var i;   
    var sum:number = 0; 
    
    for(i = 0;i<nums.length;i++) { 
       sum = sum + nums[i]; 
    } 
    console.log("和为：",sum) 
 } 
 addNumbers(1,2,3) 
 addNumbers(10,10,10,10,10)


//匿名参数
var res = function(a:number,b:number) { 
    return a*b;  
}; 
console.log(res(12,2))


//主动构造一个方法
var myFunction = new Function("a", "b", "return a * b"); 
var x = myFunction(4, 3); 
console.log(x);


//箭头函数
var foo = (x:number)=> {    
    x = 10 + x 
    console.log(x)  
} 
foo(100)
```
## 8. 封装类型
1. 数字封装类型
	`var num = new Number(value)`
	类静态属性：`MAX_VALUE、MIN_VALUE、NaN、NEGATIVE_INFINITY、POSITIVE_INFINITY`
	实例方法：
	- `toExponential`对象的值转换为指数计数法
	-  `toFixed`把数字转换为字符串，并对小数点指定位数
	-  `toLocaleString`把数字转换为字符串，使用本地数字格式顺序
	-  `toPrecision`把数字格式化为指定的长度
	-  `toString`把数字转换为字符串，使用指定的基数
	-  `valueOf`返回一个 Number 对象的原始数字值
2. 字符串封装类型
	`var txt = new String("string")`
	类静态属性：`length`
	实例方法：
	- `charAt`返回在指定位置的字符
	- `charCodeAt`返回在指定的位置的字符的 Unicode 编码
	- `concat`连接两个或更多字符串，并返回新的字符串
	- `indexOf`返回某个指定的字符串值在字符串中首次出现的位置
	- `lastIndexOf`从后向前搜索字符串，并从起始位置（0）开始计算返回字符串最后出现的位置
	- `slice`提取字符串的片断，并在新的字符串中返回被提取的部分
	- `split`把字符串分割为子字符串数组
	- `substr`从起始索引号提取字符串中指定数目的字符
	- `substring`提取字符串中两个指定的索引号之间的字符
3. 数组封装类型
	`var numlist:number[] = [2,4,6,8]`
	实例方法：
	- `concat`连接两个或更多的数组，并返回结果
	- `every`检测数值元素的每个元素是否都符合条件
	- `filter`检测数值元素，并返回符合条件所有元素的数组
	- `forEach`数组每个元素都执行一次回调函数
	- `indexOf`搜索数组中的元素，并返回它所在的位置。如果搜索不到，返回值 -1
	- `join`把数组的所有元素放入一个字符串
	- `lastIndexOf`返回一个指定的字符串值最后出现的位置，在一个字符串中的指定位置从后向前搜索
	- `map`通过指定函数处理数组的每个元素，并返回处理后的数组
	- `pop`删除数组的最后一个元素并返回删除的元素
	- `push`向数组的末尾添加一个或更多元素，并返回新的长度
	- `reduce`将数组元素计算为一个值（从左到右）
	- `reverse`反转数组的元素顺序
	- `shift`删除并返回数组的第一个元素
	- `slice`选取数组的的一部分，并返回一个新数组
4. 字典封装类型
	`let myMap = new Map([ ["key1", "value1"], ["key2", "value2"] ])`
	实例方法：
	- `map.clear()`移除 Map 对象的所有键/值对
	- `map.set()`设置键值对，返回该 Map 对象
	- `map.get()`返回键对应的值，如果不存在，则返回 undefined
	- `map.has()`返回一个布尔值，用于判断 Map 中是否包含键对应的值
	- `map.delete()`删除 Map 中的元素，删除成功返回 true，失败返回 false
	- `map.size` 返回 Map 对象键/值对的数量
	- `map.keys()`返回一个 Iterator 对象， 包含了 Map 对象中每个元素的键
	- `map.values()`返回一个新的Iterator对象，包含了Map对象中每个元素的值
5. 元组封装类型
	`var mytuple = [10,"Runoob"];`
	实例方法：
	- `push()`向元组添加元素，添加在最后面
	- `pop()`从元组中移除元素（最后一个），并返回移除的元素
## 9. 面向对象
### 1. 接口
普通使用：
```ts
interface IPerson { 
    firstName:string, 
    lastName:string, 
    sayHi: ()=>string 
} 
 
var customer:IPerson = { 
    firstName:"Tom",
    lastName:"Hanks", 
    sayHi: ():string =>{return "Hi there"} 
} 
 
console.log("Customer 对象 ") 
console.log(customer.firstName) 
console.log(customer.lastName) 
console.log(customer.sayHi())  
 
var employee:IPerson = { 
    firstName:"Jim",
    lastName:"Blakes", 
    sayHi: ():string =>{return "Hello!!!"} 
} 
 
console.log("Employee  对象 ") 
console.log(employee.firstName) 
console.log(employee.lastName)
```

接口继承
```ts
interface Person { 
   age:number 
} 
 
interface Musician extends Person { 
   instrument:string 
} 
 
var drummer = <Musician>{}; 
drummer.age = 27 
drummer.instrument = "Drums" 
console.log("年龄:  "+drummer.age)
console.log("喜欢的乐器:  "+drummer.instrument)
```

多个接口继承
```ts
interface IParent1 { 
    v1:number 
} 
 
interface IParent2 { 
    v2:number 
} 
 
interface Child extends IParent1, IParent2 { } 
var Iobj:Child = { v1:12, v2:23} 
console.log("value 1: "+Iobj.v1+" value 2: "+Iobj.v2)
```

### 2. 类
普通使用
```ts
class Car { 
    // 字段 
    engine:string; 
 
    // 构造函数 
    constructor(engine:string) { 
        this.engine = engine 
    }  
 
    // 方法 
    disp():void { 
        console.log("发动机为 :   "+this.engine) 
    } 
}

var obj = new Car("Engine 1")
```

类（接口）继承
```ts
class Shape { 
   Area:number 
   
   constructor(a:number) { 
      this.Area = a 
   } 
}
class Circle extends Shape { 
   disp():void { 
      console.log("圆的面积:  "+this.Area) 
   } 
}
var obj = new Circle(223); 
obj.disp()


interface ILoan { 
   interest:number 
}
class AgriLoan implements ILoan { 
   interest:number 
   rebate:number 
   
   constructor(interest:number,rebate:number) { 
      this.interest = interest 
      this.rebate = rebate 
   } 
}
var obj = new AgriLoan(10,1) 
console.log("利润为 : "+obj.interest+"，抽成为 : "+obj.rebate )
```

方法重写
```ts
class PrinterClass { 
   doPrint():void {
      console.log("父类的 doPrint() 方法。") 
   } 
} 
 
class StringPrinter extends PrinterClass { 
   doPrint():void { 
      super.doPrint() // 调用父类的函数
      console.log("子类的 doPrint()方法。")
   } 
}
```

类静态成员
```ts
class StaticMem {  
   static num:number; 
   
   static disp():void { 
      console.log("num 值为 "+ StaticMem.num) 
   } 
} 
 
StaticMem.num = 12     // 初始化静态变量
StaticMem.disp()       // 调用静态方法
```

`instanceof`运算符
```ts
class Person{ } 
var obj = new Person() 
var isPerson = obj instanceof Person; 
console.log("obj 对象是 Person 类实例化来的吗？ " + isPerson);
```

访问控制修饰符
- `public`：公有，可以在任何地方被访问
- `protected`：受保护，可以被其自身以及其子类访问
- `private`：私有，只能被其定义所在的类访问
```ts
class Encapsulate { 
   str1:string = "hello" 
   private str2:string = "world" 
}
 
var obj = new Encapsulate() 
console.log(obj.str1)     // 可访问 
console.log(obj.str2)   // 编译错误， str2 是私有的
```
### 3. 泛型
泛型函数
```ts
function identity<T>(arg: T): T {
    return arg;
}
// 使用泛型函数
let result = identity<string>("Hello");
console.log(result); // 输出: Hello

let numberResult = identity<number>(42);
console.log(numberResult); // 输出: 42
```

泛型接口
```ts
// 基本语法
interface Pair<T, U> {
    first: T;
    second: U;
}
// 使用泛型接口
let pair: Pair<string, number> = { first: "hello", second: 42 };
console.log(pair); // 输出: { first: 'hello', second: 42 }
```

泛型类
```ts
// 基本语法
class Box<T> {
    private value: T;

    constructor(value: T) {
        this.value = value;
    }

    getValue(): T {
        return this.value;
    }
}
// 使用泛型类
let stringBox = new Box<string>("TypeScript");
console.log(stringBox.getValue()); // 输出: TypeScript
```

泛型约束
```ts
// 基本语法
interface Lengthwise {
    length: number;
}
function logLength<T extends Lengthwise>(arg: T): void {
    console.log(arg.length);
}
// 正确的使用
logLength("hello"); // 输出: 5
// 错误的使用，因为数字没有 length 属性
logLength(42); // 错误
```

泛型默认值
```ts
// 基本语法
function defaultValue<T = string>(arg: T): T {
    return arg;
}
// 使用带默认值的泛型函数
let result1 = defaultValue("hello"); // 推断为 string 类型
let result2 = defaultValue(42);      // 推断为 number 类型
```