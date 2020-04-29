# 0423 | 表达式，类型转换



浮点数解析的  Vue  demo



1. 符号位(正负数)
2. 指数位(整数部分--参考科学计数法)
3. 有效数字部分(小数部分)



在二进制中，一直乘以2 永远不会变成整数的 就是除不尽的，

在十进制中， 一直乘以10永远不会变成整数的 就除不尽的



```
1.3+1.1 === 2.4  // false
0.1 + 0.2  ===0.3 // false
1.3+1.2 ===2.5  // true
```

不同的运算，精度丢失的程度不一样







## Expression(表达式)

> 注: EMCA -262 ： 12.3 Left-Hand-Side Expressions 

 

一定要学会看标准文档了.....



优先级，是用表达式生成树的方式来实现的。

![image.png](https://cdn.nlark.com/yuque/0/2020/png/382504/1588002810991-70e57fa2-944a-4ae9-b1af-d381d5e8e19c.png)

### 运算符优先级第一个:MemberExpression--成员访问表达式

- a.b
- a[b]
- foo `string`
- super.b(只能在构造函数中使用)
- super['b'](只能在构造函数中使用)
- new.target(只能在构造函数中使用)
- new Foo()



**判断  fakeObjet 是否是通过 new 生成的 -- 结果：只能在函数中通过new.target判断**

```
        function foo() {
            console.log('new.target :', new.target);

        }
        console.log('foo() :', foo());

        console.log('new foo() :', new foo());


        const fakeObjet = {};

        Object.setPrototypeOf(fakeObjet, foo.prototype)

        fakeObjet.constructor = foo;

        foo.apply(fakeObjet)
        // 判断  fakeObjet 是否是通过 new 生成的 -- 结果：只能在函数中通过new.target判断

        // 通过原型链检查
        const res1 = fakeObjet instanceof foo;
```

 **继承父类的东西--  super();**

```
    class Parent {
        constructor() {
            this.a = 1
        }
    }

    class Child extends Parent {
        constructor() {
            super();
            console.log(this.a)
        }
    }

    new Child;
        function fooA() {
            console.log(arguments);
        }
        const name = '小明'
        fooA`hello ${name} !`
  // 结果是  被分段解析了
```

**new Foo, 与 new Foo(), 带括号的优先级更高**

```
        function class1(s) {
            console.log('s: ', s);

        }

        function class2(s2) {
            console.log('s2: ', s2);
            return class1
        }
        console.log('new class2: ', new class2);

        console.log('class2(): ', class2());
        // class2的实例， 返回了class1
        console.log('new class2(): ', new class2());
        // 返回 class1的对象
        console.log('new (class2()): ', new (class2()));
        // 1. 先执行了 new class2() 然后再执行前面一个new
        console.log('new new class2(): ', new new class2('good'));

        console.log('new (new class2)(): ', new (new class2)());

        console.log('new (new class2()): ', new (new class2()));

        // 总结:
        // new Foo, 与 new Foo(), 带括号的优先级更高
```

###  

### 运算符优先级第二个:NewExpression

-  new Foo



> 注: EMCA -262 ： 搜索  NewExpression

```
new a()()
new new a()
```



#### Reference

```
        let o = { x: 1 }
            // 相等， 为什么, 因为加法的时候会去解析 o.x Reference类型的值
        console.log(' o.x  +2 ===3 :', o.x + 2 == 3);
        // 不相等， 为什么
        console.log(' delete o.x ===1 :', delete o.x === 1);

        // delete o.x 返回的是一个Reference类型
```

##### 组成部分

- Object
- Key



// 类似下面的代码

```
        class Reference {
            constructor(object, property) {
                this.object = object
                this.property = property
            }
        }
```



##### 案例

- delete
- assign



### 运算符优先级第三个:CallExpression



- foo()
- super()
- foo()['b']
- foo().b
- foo()`abc`



案例: new a()['b']



让 new的结果符合人类的思维预期

```
        class foo {
            constructor() {
                this.b = 1;
            }
        }

        console.log("new foo()['b']:", new foo()['b']);
        // 报错
        // console.log(" new (foo()['b']):", new (foo()['b']));

        foo['c'] = function () { }

        console.log("new foo['c']: ", new foo['c']);

        console.log(" new (foo['c']):", new (foo['c']));
```

### Left hand side(等号左边) & Right hand side(等号右边表达式)

```
a.b =c
a+b = c
```

等号左边必须是  Reference 类型

####  

### Right hand side(等号右边表达式)



##### Update(更新运算符--不能换行,必须是number)

- a++ 
- a--
- --a
- ++a

> ECMA-262 搜索  UpdateExpression 



```
a
++
b
++c

// 执行结果是  b,c自增了
```

 

##### Unary(单目运算符)

- delete a.b
- void foo()
- typeof a
- +a
- -a
- ~a(必须是整数number)
- !a (转换为boolean)
- await a(必须是promise)



```
// 在JavaScript中 void 是运算符，不管后面接什么都会返回 undefined
```

**立即执行函数**

```
        function test() {
            for (var i = 0; i < 7; i++) {
                const button = document.createElement('button');
                document.body.appendChild(button)
                button.addEventListener('click', function () {
                    console.log(i)
                })
            }
        }
        // test()
        // 一种解决方式
        function test2() {
            for (var i = 0; i < 7; i++) {
                const button = document.createElement('button');
                button.innerHTML = i;
                document.body.appendChild(button)
                void function (i) {
                    button.addEventListener('click', function () {
                        console.log(i)
                    })
                }(i)
            }
        }
        test2()
```



```
        // 没法区分同一个对象里面的不同的class
        console.log(typeof null)

        // 没法区分 包装类型和原型类型
        console.log(Object.prototype.toString.call(null));

typeof function(){}  // function
```

### Exponental --肯定是右手运算符

-  ** (必须是number)

```
 3 **2 **3 === 3 ** (2 ** 3)
```

### Muliplicative

-  \* / %

####  

### Additive(number或string)

-   \+ -

####  

### Shift -- 位运算（必须是整数number）

-  << ,>>, >>>

####  

### Relationship--关系比较运算

-  <,>, <=,>=,  （必须是number）
- instanceof， in



### Equlity

- ===
- !==



### Bitwise （必须是整数number）

- & ,|,^
   

### Logical--逻辑运算

- &&
-  || 

```
// 有一个短路的逻辑， 可以模拟 if else

如: foo()  && foo2()

不会把所有的函数都计算出来，先算左边的，如果有真，后面就不会执行
```



### Conditional--三目运算

- ? :

```
// 有一个短路的逻辑， 可以模拟 if else

如: a? foo()  : foo2()

不会把所有的函数都计算出来，只会执行其中一个
```

#### JavaScript中有几种加法

- 数字加法
- 字符串加法



其他类型的加法都会被转换为 数字或字符串

![image.png](https://cdn.nlark.com/yuque/0/2020/png/382504/1588008238437-d19bc0a9-8248-48bc-8aec-5490859dad6a.png)

### 装箱， 拆箱转换



**以下四种类型可以装箱**

- number类型 对应 Number类
- string类型对应 String类
- symbol类型对应Symbol类--无法new
- boolean类型对应 Boolean类

#### string&number的转换

```
// object  使用new会转换为对象
console.log('new Number(1) :', typeof new Number(1));
// number   不使用new 会被转换为普通类型
console.log('typeof 1 :', typeof 1);
const a = Symbol('1')
// 通过Object装箱
console.log('Object(a) :', Object(a));

console.log('Object(a).constructor :', Object(a).constructor);

console.log(' Object.getPrototypeOf(Object(a)) === Symbol.prototype: ', Object.getPrototypeOf(Object(a)) === Symbol.prototype);

// 使用 this 装箱
console.log(' (function(){return this}).apply(a) :', (function () { return this }).apply(a));
```

##  

**拆箱**

```
        console.log('1+{} :', 1 + {});

        console.log('1+{valueOf(){return 2}} :', 1 + { valueOf() { return 2 } });

        console.log('1+{toString(){return 2}} :', 1 + { toString() { return 2 } });

        console.log('1+{toString(){return 2}} :', 1 + { toString() { return '4' } });

        // 优先使用 valueOf的值
        const a = 1 + { toString() { return 4 }, valueOf() { return '4' } }
        console.log(a);

        const b = 'b' + { toString() { return 4 }, valueOf() { return '4' } }
        console.log(b);

        // Symbol(Symbol.toPrimitive)
        // 只会使用 Symbol.toPrimitive的值
        const c = 1 + { [Symbol.toPrimitive]() { return 6 }, toString() { return 4 }, valueOf() { return '4' } }
        console.log(c);
```

##  

##  

## 答疑



1.  convertStringToNumber(str)，convertNumberToString(num) 与 Number(num), String(str)的区别



​     convertStringToNumber(str)，convertNumberToString(num) 可以看做  Number(num), String(str)具体内部原理实现

1.  Set类型的意义

js中缺少哈希的结构，比如给你一个新的对象，判断是否是已有对象其中的一个，怎么做？

以前只能循环比较是否相等，但计算机中有一个可哈希的概念，直接转成字符串进行比较就可以了。



# 0425 | 语句，对象



## Atom

### Grammar

- 简单语句
- 组合语句
- 声明



### Runtime

#### Completion Record(完成的记录)

语句执行完成的结果



1. [[type]]--语句完成的类型如:四种条件语句: `normal(正常语句的完成)`，`break`,`continue`,`return`,`throw`
2. [[value]]--如:七种语言类型:Types
3. [[target]]:label







#### Lexical  Environment

运行的执行过程

在JavaScript中除了null，undefined，number，string，boolean,Symbol,object，bugint这七种类型外，在代码运行时还有一些其他内部类型, 主要用来解释JavaScript机制，不会在实际编程中用到





### 简单语句

- ExpressionStatement(表达式语句，一个表达式就可以构成一个语句，相当于告诉计算机进行一个运算)
- EmptyStatement(空语句，)
- DebuggerStatement(从运行时的角度不产生任何作用，做代码调试用的)
- ThrowStatement( throw 后面可以跟一个表达式)
- ContinueStatement
- BreakStatment
- ReturnState



```
        // 表达式语句
        a=1+2;

        // 空语句
        ;
        debugger;

        throw a;

        continue;

        break;

        return 1+2;
```

### 复合语句

- BlockStatment
- IfStatement
- SwitchStatement
- IterationStatement
- WIthStatement
- LabelledStatement
- TryStatement



#### Block

当我们需要用一条语句的地方，我们就可以放一个BlockStatement,变成多条语句,使用方式， 用一对大括号括起来



- [[type]]:normal
- [[value]]:--
- [[target]]:--

```
{


}
```