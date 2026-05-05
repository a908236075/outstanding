# TypeScript

## JavaScript VS  TypeScript

### javaScript的缺点

1. 不清楚的数据类型

   ~~~html
   let welcom='hello'
   welcome()// 此处报 welcome is not a  function.
   ~~~

2. 漏洞的逻辑

   ~~~html
   const str=1
   if(str!='奇数'){}// 这样的判断也不会暴露出来
   ~~~

3. 访问不存在的属性

   ~~~html
   const obj={width: 10, height:15};
   const area=obj.wid*2; // 即使wid拼接错了 不会飘红.
   ~~~

总结:有一些静态类型检查 运行之前进行检查.

### ts转换js

~~~shell
tsc index.ts 
tsc --watch // 监控文件变化 自动转为js.
~~~



## 浏览器打开文件报错

1. ~~~text
   npx serve .                                                                            然后浏览器访问 http://localhost:3000/demo1.html。                                     或者如果你用的是 VS Code，安装 Live Server 插件，右键 demo1.html → Open with Live Server，一键解决。
   ~~~


## 常见类型

### any

1. 不做类型检查

2. ~~~ts
   let c:any;
   c=9;
   c="hello world";
   console.log(c);
   let x:string;
   x=c;
   console.log(x); 
   ~~~

3. 

### unknown

1. ~~~ts
   let y:unknown;
   y="yyyy"
   let x:string
   x=y; // 此处不被允许
   ~~~

2. 比any更加的安全 需要做类型检查

   ~~~ts
   let y:unknown;
   y="yyyy"
   if (typeof y === "string") {
     x = y;
   }
   // or
   x= y as string;
   // or
   x=<string>y
   ~~~

### never

1. 限制函数的返回类型 不能有返回值 undefined也不能有

2. ~~~ts
   function test1():never{
     throw new Error('error');
   }
   
   ~~~

### void

1. 函数的返回值为空   但是可以接受undefined和return;
2. 不能用返回值做文章

### object和Object

1. object只能放包装对象 不能放原始类型 例如number.

2. Object除了null和undefined 都可以存

3. ~~~ts
   let person:{name:string,age?:number,[key:string]:any}; // 索引签名 完全不用在乎key的名字
   person={
     name:'zhangsan'
   }
   age后面要加? 表示可以选
   let count:(a:number,b:number) => number // count 变量只能接收两个参数，返回一个number的函数值
   count=(x,y)=>{
     return x+y
   }
   ~~~

4. =>  符号 在ts中在函数类型声明时表示函数类型,描述器参数类型和返回类型.js中表示函数的具体实现.

### tuple

1.  元组是一种特殊的数组类型,可以存储固定数量的元素,且每个元素的类型是已知的且可以不同.元组用于精确的描述

2. ~~~ts
   let arr1: [string, number,number]
   arr1=['hello',123,123]
   ~~~

###   enum

1. 枚举

2. ~~~ts
   // 字符串枚举  
   enum Direction{
     Up,
     Down,
     Left,
     Right
   }
   console.log(Direction.Up)
   console.log(Direction[0])
   function walk(data:Direction){
     console.log(data == Direction.Down);
   }
   ~~~

### type

1. 能够为任意类型设置别名

2. ~~~ts
   type shuzi=number
   let b:shuzi;
   b=100
   
   type Status=number | string
   
   function printStatus(status:Status){
     console.log(status);
   }
   printStatus(100)
   printStatus('hello')
   ~~~


### Interface

1. interface与type的区别
   1. 都可以用于定义对象结构,两者在许多场景中可以互换.
   2. type:可以定义类型别名,联合类型,交叉类型,但不支持继承和自动合并.

### 泛型

1. 有any为什么还有泛型  泛型可以保有类型检查

2. ~~~ty
   // 用 any
   function identity_any(arg: any): any {
       return arg;
   }
   
   const result1 = identity_any("hello");
   // result1 的类型是 any，类型信息丢失了
   result1.toFixed(2);  // ❌ 编译不报错，但运行时崩溃！
   
   
   // 用泛型
   function identity<T>(arg: T): T {
       return arg;
   }
   
   const result2 = identity("hello");
   // result2 的类型是 string，类型信息保留了
   result2.toFixed(2);  // ✅ 编译直接报错，string 没有 toFixed
   ~~~

3. 