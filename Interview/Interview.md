# Interview

#### Across-origin

同源策略是一种约定，它是浏览器最核心也最基本的安全功能，如果缺少了同源策略，浏览器很容易受到 `XSS`（跨站脚本攻击）、`CSFR`（跨站请求伪造） 等攻击。所谓同源是指"协议+域名+端口"三者相同，即便两个不同的域名指向同一个 ip 地址，也非同源
同源策略限制以下内容：
​	Cookie，localstroage,sessionStorage等存储性内容
​	DOM节点
​	Ajax请求发送后，结果被浏览器拦截

以下三个标签是允许跨域加载资源：

```javascript
<img src=XXX>
<link href=XXX>
<script src=XXX>
```

不同域之间（协议，域名，端口号任意一个不相同时）相互请求资源，即算作跨域

跨域并不是请求发不出去，请求能发出去，服务端能收到请求并正常返回结果，只是结果被浏览器拦截了。你可能会疑问明明通过表单的方式可以发起跨域请求，为什么 Ajax 就不会?因为归根结底，跨域是为了阻止用户读取到另一个域名下的内容，Ajax 可以获取响应，浏览器认为这不安全，所以拦截了响应。但是表单并不会获取新的内容，所以可以发起跨域请求。同时也说明了跨域并不能完全阻止 CSRF，因为请求毕竟是发出去了

##### 总结：

1. CORS 支持所有类型的 HTTP 请求，是跨域 HTTP 请求的根本解决方案
2. JSONP 只支持 GET 请求，JSONP 的优势在于支持老式浏览器，以及可以向不支持 CORS 的网站请求数据。
3. 不管是 Node 中间件代理还是 nginx 反向代理，主要是通过同源策略对服务器不加限制。
4. 日常工作中，用得比较多的跨域方案是 cors 和 nginx 反向代理

#### JS

1. call,apply和bind的区别

   1. 三者均为`Function.prototype`下的方法，很好地体现了js函数式语言特性，都是用于改变函数运行时的上下文，最终返回的是调用的方法的返回值，若该方法没有返回值，则返回`undefined`

      ```js
      var max3 = Math.max.call(null,1,2,3);//3 
      //在非严格模式下，第一个参数为null或者undefined时会自动替换为指向全局对象
      var max4 = Math.max.apply(null,[1,2,3,4,5])//5 apply第二个参数为数组或类数组
      var max5 = Math.max.bind(null,[1,2,3])()//NaN
      var max6 = Math.max.bind(null,1,2,3,4,56)()//56
      //可以发现，call和apply的区别仅仅是传参的不同，而bind的区别是不会自动立即执行，需调用时才会执行
      ```

   2. `call()`是`apply()`的语法糖，三者均可实现继承；

   3. 在选用时：如果不需要考虑具体多少参数被传入函数时选`apply()`；如果确定函数可以接收参数个数， 并且想表达形参和实参的对应关系，用`call()`；如果不需要立即调用执行，按需调用使用`bind`

   4. 看以下实例：

      ```js
      var name = 'bigBoss', age = 25;
      let obj = {
        name: 'xiaosan',
        objAge: this.age,
        myFun: function() {
          console.log(this.name + ', age: ' + this.age);
        }
      }
      obj.objAge;//25
      obj.myFun();//xiaosan, age: undefined 因为这里this指向obj
      
      let dm = {
        name: 'dema',
        age: 100
      }
      
      obj.myFun.call(dm);
      obj.myFun.apply(dm);
      obj.myFun.bind(dm)();
      //均为 dema, age: 100
      ```

      ```js
      var name = 'bigBoss', age = 25;
      let obj = {
        name: 'xiaosan',
        objAge: this.age,
        myFun: function(from, to) {
          console.log(this.name + ', age: ' + this.age + ' from:' + from + ' to:' + to);
        }
      }
      obj.objAge;//25
      obj.myFun();//xiaosan, age: undefined 因为这里this指向obj
      
      let dm = {
        name: 'dema',
        age: 100
      }
      obj.myFun.call(dm,'成都','上海');//dema, age: 100 from:成都 to:上海
      obj.myFun.apply(dm,['成都','上海']);//dema, age: 100 from:成都 to:上海
      obj.myFun.bind(dm,'成都','上海')();//dema, age: 100 from:成都 to:上海
      obj.myFun.bind(dm,['成都','上海'])();//dema, age: 100 from:成都,上海 to:undefined
      ```

2. `localStorage`和`sessionStorage`的区别

   1. 两者的区别主要在于作用域和有效期，两者均不与服务器通信
   2. `localStorage`在相同文档源(协议，域名，端口号相同)下，相同浏览器的不同页面间可以共享数据，可以互相读取，甚至覆盖；但不同标签页或不同页面间无法共享`sessionStorage`数据，这里的标签页和页面指的是`顶级窗口`，如果一个标签页下包含多个`iframe`且属于同源页面，则`sessionStorage`数据是可以共享的
   3. 有效期：`localStorage`的有效期是永久的，除非通过浏览器`UI`去人为删除，而`sesionStorage`则是在关闭浏览器或标签页后便被删除，与目前情况不同的是，部分现代浏览器可以`Ctrl+shift+T`实现历史记录的重开，这时`sessionStorage`的有效期可能延长。

3. JS原型链

4. 深拷贝和浅拷贝

5. 手写一个Promise

6. 节流和防抖实现

7. == 和 ===的区别

   1. 可以这么理解，双等号比较时，会先进行数据类型比较，如果不同，则进行类型转换再比较，而三等号不做类型转换，类型不同一定不等。

   2. ```js
      let a;
      let b = undefined;
      let c = null;
      a == c;
      b == c;
      b == a;
      a === c;//false
      b === c;//false
      b === a;//其他均为true
      typeof(c) === "object";//true,其他两者typeof均为"undefined"
      ```

   3. ```js
      function Person(){};
      var p =new Person();
      console.log(p instanceof Person);//true
      var p1 = new Person();
      var p2 = new Person();
      
      p1 == p;//false
      p1 === p;//虽然typeof(p)和typeof(p1)都为“object",但是new对象只是引用地址改变(栈内存和堆内存)
      
      p2 = p;
      p2 == p;//true
      p2 === p;
      ```

   4. 

8. 实现一个`instanceof`

   1. `instanceof`用来判断一个构造函数的`prototype`属性所指向的对象是否存在另一个要检测对象的原型链上

   2. ```js
      obj instanceof Object;//true 实例obj是否存在Object构造函数中
      ```

   3. ```js
      function _instanceof(left, right) {
          var prototype = right.prototype;// 取right的显示原型
          proto = left.__proto__;// 取left的隐式原型
          while (true) {
              //Object.prototype.__proto__ === null
              if (proto === null)
                  return false;
              if (prototype === proto)// 这里重点：当 prototype严格等于 proto 时，返回 true
                  return true;
              proto = proto.__proto__;
          }
      }
      //检验
      function Person(){};
      var p =new Person();
      p instanceof Person;//true
      _instanceof(p, Person);//true
      ```

   4. 

#### layout

1. 两列自适应布局

   1. float + overflow: hidden

      ```html
      <div class="parent" style="background-color: #aaaaaa;">
         <div class="left"  style="background-color: #cccccc;">
           <p>left</p>
         </div>  
         <div class="right" style="background-color: #bbbbbb;">
           <p>right</p>
           <p>right</p>
         </div>
        </div>
      ```

      ```css
         .parent {
           /* 注意这里最好给一个整体overflow,而不是直接body给overflow属性 */
           overflow: hidden;
           /* 由于设置overflow:hidden并不会触发IE6-浏览器的haslayout属性，所以需要设置zoom:1来兼容IE6-浏览器 */
           zoom: 1;
         }
         .left {
           float: left;
           margin-right: 30px; 
         }
         .right {
           overflow: hidden;
           zoom: 1;
         }
      ```

   2. Flex

      ```css
      //html同上
      .parent {
        display:flex;
      }  
      .right {
        margin-left:20px; 
        flex:1;
      }
      ```

   3. Grid

      ```css
      .parent {
         display: grid;
         grid-template-columns:  auto 1fr;//fr相等时实现等宽两列布局
         grid-gap: 1px
      } 
      ```

> 当侧边栏在右边时，需注意渲染顺序，即HTML中，需先写侧边栏再写主内容

2. 三栏布局

   1. 圣杯布局

      ```css
      .container {
          padding-left: 220px;
          padding-right: 220px;
        }
        .left {
          /* 三部分都设为左浮动 */
          float: left;
          width: 200px;
          height: 100vh;
          background: #aaaaaa;
          /* 通过设置margin-left为负值让left和right部分回到与center部分同一行
          左边的-100% 对应左边宽度，右边只能写右边宽度的负值 */
          margin-left: -100%;
          /* 两侧position均为relative */
          position: relative;
          /* 两侧对应给左右属性负值padding-left+left = padding-right+right = 0 */
          left: -220px;
        }
        .center {
          float: left;
          /* 中间宽度100% */
          width: 100%;
          height: 100vh;
          background: #bbbbbb;
        }
        .right {
          float: left;
          width: 200px;
          height: 100vh;
          background: #cccccc;
          margin-left: -200px;
          position: relative;
          right: -220px;
        }
      ```

      ```html
        <article class="container">
          <div class="center">
            <h2>圣杯布局</h2>//先写中间列，以实现优先加载
          </div>
          <div class="left"></div>
          <div class="right"></div>
        </article>
      ```

   2. 双飞翼布局

      ```css
      //解决圣杯布局错乱的问题，实现内容和布局的分离，而且任一栏都可以是最高栏
          .container {
              min-width: 600px;//确保中间内容可以显示出来,必须要大于left宽+right宽
          }
          .left {
              float: left;
              width: 200px;
              height: 400px;
              background: #aaaaaa;//margin同样实现三者同行的作用
              margin-left: -100%;
          }
          .center {
              float: left;
              width: 100%;
              height: 500px;
              background: #bbbbbb;
          }
          .center .inner {
              margin: 0 200px; //新增部分，左右两翼各200px
          }
          .right {
              float: left;
              width: 200px;
              height: 400px;
              background: #cccccc;
              margin-left: -200px;
          }
      
      ```

      ```html
      <article class="container">
          <div class="center">
              <div class="inner">双飞翼布局</div>
          </div>
          <div class="left"></div>
          <div class="right"></div>
      </article>
      ```

      **圣杯布局是利用父容器的左、右内边距+两个从列相对定位； 双飞翼布局是把主列嵌套在一个新的父级块中利用主列的左、右外边距进行布局调整**

3. 等高布局

   1. 正padding + 负margin

4. 粘连布局

   1. footer设计

5. 其他布局

   1. 参考[实现三栏布局的6种方式](https://www.jianshu.com/p/3046eb050664)
   2. [div块元素垂直水平居中](https://www.cnblogs.com/Youngly/p/6796922.html)

#### 前端

1. 原始类型：`boolean, null, undefined, number, symbol, string`原始类型存储的都是值（对比对象类型存储的是指针地址），没有函数可以调用。

   - string类型是不可变的，无论调用何种方法，值都不会改变；

   - null是个对象类型其实是个bug，JS最初版本使用的是32位系统，为了性能考虑使用的是低位存储变量的类型信息，`000`开头代表是对象，而`null`表示的是全零，故此误判为`object`。

2. 对象类型：

   ```javascript
   const a = []
   const b = a
   b.push(1) // a = [1], b = [1]
   // 当变量赋值时，复制的是原本变量的地址，也就是说两个变量的存放的地址（指针）是一样的，所以两个值都会
   //发生改变
   ```

   ```javascript
   // 函数参数为对象
   // 记住函数传参为对象时传的是对象指针的副本，进入函数内部后会修改属性所以p1的值也会被修改
   function test(person) {
     person.age = 26
     person = {
       name: 'yyy',
       age: 30
     }
     // person.name = "xiaohei" 若添加此行，p1不变，p2{name:"xiaohei",age:30}
     return person
   }
   const p1 = {
     name: 'yck',
     age: 25
   }
   const p2 = test(p1)
   // p1{name:"yck", age:26}
   // p2{name:"yyy", age:30}
   ```

3. `typeof & instanceof`

   `typeof`对于原始类型来说，除了`null`都可以显示正确的类型

   `typeof`对于对象类型，除了函数都会显示为`object`，所以并不准确

   ```javascript
   //注意symbol
   typeof Symbol() // "symbol"
   typeof null // "object"
   
   typeof console.log // "function"
   typeof console // "object"
   ```

   如果想要正确判断一个对象的正确类型，这里就要使用`intanceof`，因为其内部机制是通过原型链来判断的。如果想要使用`instanceof`来判断原始类型也是可以的，可以使用`Symbol.hasInstance`来自定义`instanceof`的行为。

   ```javascript
   class PrimitiveString() {
       static [Symbol.hasInstance](x) { // 静态属性
           return typeof x === "string"
       }
   }
   console.log("aaa" instanceof PrimitiveString) // true // 这里说明，instanceof也不是一定就是对的
   ```

4. 类型转换

| 原始值           | 转换目标 |                      结果                      |
| ---------------- | -------- | :--------------------------------------------: |
| number           | boolean  |             除了0，-0，NaN都为true             |
| string           | boolean  |                除了空串都是true                |
| undefined/null   | boolean  |                     false                      |
| 引用类型         | boolean  |                      true                      |
| number           | string   |                   10 -> '10'                   |
| Array            | string   |               [1,2,4] -> '1,2,4'               |
| object           | string   |                     string                     |
| string           | number   |              "1" -> 1, 'a' -> NaN              |
| Array            | number   | 空数组为0，存在一个元素且为数字转数字，其他NaN |
| null             | number   |                       0                        |
| 除数组的引用类型 | number   |                      NaN                       |
| Symbol           | number   |                      报错                      |

  在条件判断时，除了`undefined, null, '', 0, false, NaN, -0`其他所有值都转为`true`，注意这里空数组也是转为`true`

  **对象转换为原始类型**：`toString() , valueOf`, 对象在转换类型的时候，会调用内置的`[[toprimitive]]`函数，该函数优先级最高，也可以重写

  ```javascript
  let a = {
      valueOf() {
          return 0
      }
      toString() {
          return 1
      }
  	[Symbol.toPrimitive]() {
      	return 2
  	}
  }
  1 + a; // 3
  ```

  **四则运算符**：

  运算中一方是字符串，结果会被转为字符串；

  一方中不是字符串或数字，结果会被转换为字符串或数字；

  对于除加法之外的运算来说，只要其中一方是数字，则会转换为数字

  ```javascript
  true + true // 2
  true + false // 1
  1 + [2,3,5] // "12,3,5"
  'a'+ +'b' // -> "aNaN" 注意空格
  2 * [1,2] // NaN
  2 * [] // 0 //上面表格有讲空数组转数字时为0
  ```

  **比较运算符**：对象--`toPrimitive`;字符串--通过`unicode`字符索引比较

  **this**

  ![img](https://user-gold-cdn.xitu.io/2018/11/15/16717eaf3383aae8?imageslim)

  箭头函数其实是没有`this`的，它只取决于包裹箭头函数的第一个普通函数的`this`，另外对箭头函数使用`bind`这类函数是无效的。

  ```javascript
  let a = {age: 20}
  function foo() {
      console.log(this.age)
  }
  foo.bind(a)() // 20
  ```

  在上图的`this`规则中，`new`的方式优先级最高，然后`bind()`，然后是`obj.foo()`最后是`foo`，而且，箭头函数的`this`一旦绑定，是不会再改变的。

  ```javascript
  function foo() {
      console.log(this.age)
  }
  var age = 10
  foo() // 10
  
  const obj = {
      age: 20,
      foo: foo
  }
  obj.foo() // 20
  
  let c = new foo()
  let d = {age: 30}
  foo.bind(d)() // 30
  ```

####   `== vs ===`

双等号是针对于如果对比双方的类型不一样，就会进行类型转换。`===`即判断两者类型和值是否相同。

![img](https://user-gold-cdn.xitu.io/2018/12/19/167c4a2627fe55f1?imageslim)

#### 闭包

#### 深浅拷贝

#### 原型

#### var、let及const

#### 原型继承和Class继承

1. `Class`只是语法糖，本质还是函数

   ```javascript
   class Person {}
   Person instanceof Function // true
   ```

2. 组合继承

3. 寄生组合继承

4. `Class`继承

#### 模块化

1. 好处：解决命名冲突；提供复用性；提高代码可维护行
2. `CommonJS`
3. `ES Module`
4. `Proxy`

#### map、filter、reduce

5. **并发与并行的区别？**

   并发是宏观概念，假设我有A和B两个任务需要执行，在一段时间内通过任务间的切换完成了这两个任务，这种情况就可称之为并发；

   并行是微观概念，假设CPU中有两个核心，我想同时跑任务A和B，同时完成多个任务的情况就可称之为并行。

6. `Generator`

   ```javascript
   function * bar(x) {
         let y = yield(x + 1);
         let z = yield(y/3);
         return (x + y + z)
       }
       let it = bar(5)
       //
   undefined
   it.next()
   {value: 6, done: false}
   it.next(9)
   {value: 3, done: false}
   it.next(2)
   {value: 16, done: true}
   it.next(2)
   {value: undefined, done: true}
   ```

7. `Promise`

   ```javascript
   let promise1 = new Promise((resolve, reject) => {
         console.log("new Promise")
         resolve("success")
       })
   
       let promise2 = new Promise((resolve, reject) => {
         console.log("error")
         reject("error")
       })
   
       Promise.all([promise1, promise2]).then(res => {
         console.log("ok")
       })
   //
   VM1052:2 new Promise
   VM1052:7 error
   ```

8. `async`及`await`

9. 定时器函数

   `setInterval, setTimeout, requestAnimationFrame`，如果有循环定时器的需求，可以通过	`requestAnimationFrame`实现，其自带函数节流功能，基本可以保证在16.6ms内只执行一次（不掉帧的情况下）。相比其他定时器时间不准的问题，该函数的延时效果是精确的。

   ```javascript
   function setInterval(callback, interval) {
         let timer
         const now = Date.now
         let startTime = now()
         let endTime = startTime
         const loop = () => {
           timer = window.requestAnimationFrame(loop)
           endTime = now()
           if (endTime - startTime >= interval) {
             startTime = endTime = now()
             callback(timer)
           }
         }
         timer = window.requestAnimationFrame(loop)
         return timer
       }
   
       let a = 0
       setInterval(timer => {
         console.log(1)
         a++
         while(a === 5) cancelAnimationFrame(timer)
       }, 1000)
   ```

10. Promise

    1. 简易Promise

       ```javascript
       //1.function myPromise(fn)
       //2. function resolve()
       //3. function reject()
       //4. try{fn(resolve, reject)} catch(e){reject(e)}
       //myPromise.prototype.then
       ```

11. Event Loop

    1. Js单线程的好处
       - 如果是多线程，那么就有可能出现多个线程操作同一个DOM，
       - 单线程的执行效率比较高，省去了比较耗时的线程切换以保证程序执行上下文。

12. `instanceof`的原理

    1. 原理是其内部机制是通过判断对象的原型链中是否能够找到类型的`prototype`，以此来正确判断对象的类型。

    2. 实现`instanceof`只针对对象有效

       ```javascript
       function myInstanceof(left, right) {
       	left = left.__proto__ // 获取对象原型
           let prototype = right.prototype// 获取类型原型
           while(true) { // 一直判断对象的原型是否等于类型的原型，知道对象原型为null，因原型链最终为null
               if(left === null || left === undefined)
                   return false
               if(left === prototype)
                   return true
               left = left.__proto__
           }
       }
       ```

13. `0.1 + 0.2 != 0.3`的问题

    1. 原因：JS采用IEEE754双精度版本（64位），其中使用二进制存储，二进制表示小数位很多是无限循环的，JS采用的浮点数标准会裁剪掉部分数字，导致小数不准确。

    2. 解决方法：`parseFloat((0.1 + 0.2).toFixed(10)) === 0.3 //true`或`parseFloat((0.1 + 0.2).toFixed(10)) === 0.3`

14. V8引擎下的垃圾回收机制

    GC算法

    标记清除算法和标记压缩算法

    并发标记（2018）：GC扫描和标记对象的同时，允许JS运行

    清除对象后会造成堆内存出现碎片的情况，当碎片超过一定限制后会启动清除压缩算法。压缩过程中，将活的对象向一端移动，直到所有对象都移动完成然后清理掉不需要的内存。

15. `call, apply, bind`函数

    - 不传入第一个参数，那么上下文默认为`window`
    - 改变了`this`的指向，让新的对象可以执行该函数，并且能接受函数

    ```javascript
    //实现call
    Function.prototype.myCall = function(context) {
      if(typeof this !== 'function') {
        throw new TypeError('Error')
      }
      context = context || window
      context.fn = this //给context创建一个fn属性，值设置为需要调用的函数
      const args = [...arguments].slice(1) // 剥离第一个参数
      //调用函数并将对象上的函数删除
      const result = context.fn(...args)
      delete context.fn
    
      return result
    }
    
    //实现apply
    Function.prototype.myApply = function(context) {
      if(typeof this !== 'function') {
        throw new TypeError('Error')
      }
      context = context || window
      context.fn = this
      let result
      // 和call的区别
      if(arguments[1]) {
        result = context.fn(...arguments[1])
      }else {
        result = context.fn()
      }
      delete context.fn
    
      return result
    }
    
    // bind与前两者的区别是返回的是一个函数
    ```

    

16. `new`操作符

    1. `new`的原理：生成一个新的对象，链接到原型，绑定了`this`，返回新对象

       ```javascript
       function create() {
       // 创建一个空对象
       let obj = {}
       // 获取构造函数
       let Con = [].shift.call(arguments)
       // 链接到原型（instanceof的实现？对象原型和类型原型）
       obj.__proto__ = Con.prototype
       // 绑定到this并执行构造函数
       let result = Con.apply(obj, arguments)
       // 确保返回值为函数
       return result instanceof Object ? result : obj
       }
       ```

       

    2. new一个对象和直接使用对象字面量创建对象有什么区别？

       字面量创建的对象可读性和性能都比较好，new的对象需要通过作用域链找到`Object`

17. DevTools Tips

    1. 元素快速定位：需要快速定位到某个滚动页面里的元素时，可以F12找到该元素代码，然后右键选择`Scroll into view`
    2. 给DOM打断点可以查看该DOM是如何通过JS更改的：元素=>右键=>`Break on`=>`subtree modifications`
    3. 找到之前查看过的DOM元素：console页面菜单下使用`$0,$1`等等，右键该元素可定位到该元素，也可以更改
    4. 浏览器中打断点时使用`Edit breakpoint...`在弹框中输入条件，使得只有满足该条件时断点才会生效

18. 事件机制

    1. window=>冒泡=>捕获

    2. 使用`addEventListener` 注册事件：

       ```javascript
       // 注册事件的传参
       node.addEventListener(
         'click',
         event => {
           console.log('冒泡')
         },
         false
       )
       ```

       有三个参数，第三个参数可以是`boolean`值，也可以是对象。布尔值`useCapture`决定了注册的事件是捕获事件还是冒泡事件。对象参数如下：

       `capture`: 和`useCapture`一样；

       `once`: 为`true`表示只会该回调只会调用一次，调用完就不在监听

       `passive`: 表示永远不会调用`preventDefault`

    3. `stopPropagation`可阻止事件冒泡，当然也可以阻止事件捕获

    4. `stopImmediatePropagation`同样能阻止事件，还能阻止该事件目标执行别的注册事件

       ```javascript
       node.addEventListener(
         'click',
         event => {
           event.stopImmediatePropagation()
           console.log('冒泡')
         },
       false)
       // 以下将不会执行
       node.addEventListener(
         'click',
         event => {
           console.log('捕获')
         },
         false
       )
       ```

    5. 事件代理

       1. 如果一个节点中的子节点是动态生成的，那么要注册事件则应该注册在父节点上。

       2. ```javascript
          let ul = document.querySelector('#ul')
          ul.addEventListener('click', (event) => {
              console.log(event.target)
          })
          ```

       3. 优点： 节省内存；不需要给子节点注销事件

19. 跨域

    1. 主要用来防止`CSRF`攻击（利用用户的登录状态发起恶意请求），只是防止，并不能完全阻止，因为请求毕竟是发出去了。

    2. `JSONP`利用`<script>`标签没有跨域限制的漏洞，

       优点：兼容性好

       缺点：只支持get请求

       ```javascript
       <script src="http://domain/api?param=aa&param1=bb&callback=jsonp"></script>
       <script>
           function jsonp(data) {
           	console.log(data)
       	}
       </script>
       ```

       封装多个请求回调函数相同的`JSONP`的简单实现

       ```javascript
       function jsonp(url, callback, success) {
           let script = document.createElement('script')// 创建一个script标签，包含url,type,async
           script.url = url
           script.async = true
           script.type = 'text/javascript'
           window[callback] = function(data) {
               success & success(data)
           }
           document.body.appendChild(script)
       }
       jsonp('http://abc', 'callback', function(value)) {
             console.log(value)
       }
       ```

    3. `CORS`需要浏览器端和后端同时支持，`IE8和IE9`需要通过`XDomainRequest`来实现

       `XDomainRequest`[参考](<https://developer.mozilla.org/zh-CN/docs/Web/API/XDomainRequest>)，`IE10`中被包含`CORS`的`XMLHttpRequest`取代了，其他浏览器不兼容

       通过`CORS`解决跨域问题，会在发送请求时出现两种情况，分别为**简单请求和复杂请求**

       **简单请求**：`get head post`；`Content-Type`仅限以下三者之一：`text/plain multipart/form-data application/x-www-form-urlencoded`

       请求中的任意`XMLHttpRequestUpload`对象均没有注册任何事件监听器；但是可以使用`XMLHttpRequest.upload`属性访问

       **复杂请求**：除简单请求之外的即是复杂请求

       复杂请求首先会发起一个预检请求，`option`方法，通过该请求确定服务端是否允许跨域请求。

       `tips`：注意使用`Node`配置`CORS`可能会踩到`Authorization`字段验证的坑，解决方法：

       ```javascript
       // 在回调中过滤option方法
       res.statusCode = 204
       res.setHeader('Content-Length', 0)
       res.send()
       ```

       

       **document.domian：**此方式只适用于二级域名相同的情况下的跨域，如`a.test.com和b.test.com`，只要在页面中添加`document.domain='test.com'`表示二级域名相同即可实现跨域

       **postMessage：**通常用于获取的嵌入页面的第三方页面的数据，一个页面发送消息，另一个页面判断来源并接收信息

       ```javascript
       // 发送信息端
       window.parent.postMessage('message', 'http://test.com')
       // 接收信息端
       var mc = new MessageChannel()
       mc.addEventListener('message', event => {
           var origin = event.origin || event.originalEvent.origin
           if(origin === 'http://test.com') {
               console.log('验证通过')
           }
       })
       ```

20. **存储**

    

    | 特性         | cookie                                   | localStorage             | sessionStorage   | indexDB                  |
    | ------------ | ---------------------------------------- | ------------------------ | ---------------- | ------------------------ |
    | 数据生命周期 | 一般由服务器端生成，可设置过期时间       | 除非被清理，否则一直存在 | 页面关闭就被清理 | 除非被清理，否则一直存在 |
    | 数据存储大小 | 4K                                       | 5M                       | 5M               | 不限                     |
    | 与服务端通信 | 每次都会携带在header中，对请求性能有影响 | 不参与                   | 不参与           | 不参与                   |
    | 适用场景     | 不建议用于存储                           | 不经常改变的数据         | 经常改变的数据   | 存储数据量大             |

    `cookie安全`：`value`属性如果用于保存登录状态，应该加密而不是适用明文；

    `http-only`设置不能通过`Js`访问，防止`XSS`攻击

    `secure`只能在`https`请求中携带

    `same-site`设置浏览器不能再跨域请求中携带`cookie`，防止`CSRF`攻击

    **Service Worker**

    其为运行在浏览器背后的`独立线程`，可以用来实现缓存功能，因为涉及到请求拦截，所以必须使用`https`协议保障安全。

    实现缓存功能的三个步骤：

    1. 注册`Service Worker`
    2. 监听`install`事件后即可缓存需要缓存的文件
    3. 下次用户访问的时候就可以通过拦截请求的方式查询是否存在缓存，存在则直接读取，否则去请求数据

    Devtools => application => Service Worker中查看

    **Web Worker**

    

21. #### 浏览器的渲染原理

    1. 了解这个是为了解决性能问题，执行JS文件有一个JS引擎，那么渲染也有一个渲染引擎（DOM）
    2. 浏览器接收到`html`文件到DOM树：字节数据 => 字符串 => Token(标记化) => Node => DOM
    3. CSSDOM和HTML类似
    4. 俩DOM合并生成渲染树，然后根据渲染树布局（或者叫回流），然后调用GPU绘制，合成图层，渲染在屏幕上

    **页面中插入大量的DOM，如何不卡顿？**

    - 方法一：使用`requestAnimationFrame`的方式循环插入DOM，
    - 方法二：使用**虚拟滚动**，只渲染可视区域的内容，不可见区域完全不渲染，当用户滚动鼠标的时候实时替换渲染的内容

    **如何减少渲染阻塞**

    1. 要想渲染快，可以减少渲染文件大小，并扁平层级，优化选择器
    2. 将`script`标签放在`body`标签底部，因为在遇到`script`标签时，浏览器会暂停构建`DOM`，完成后再继续渲染，所以若想首屏加载快，不应该首屏就加载JS文件。若不放在底部，可以考虑给`script`标签添加`defer`或`async`属性，前者表示该JS文件会并行下载，但是会放到HTML解析完成后顺序执行，后者表示JS文件和解析不会阻塞渲染，适合用于没有任何依赖的JS文件。

    **回流与重绘**

    重绘：当前节点需要改变外观而不会影响布局的，比如颜色

    回流：布局或几何属性需要发生改变

    回流必定重绘，重绘不一定导致回流

    重绘和回流其实也和 Eventloop 有关，参考[HTML文档](<https://html.spec.whatwg.org/multipage/webappapis.html#event-loop-processing-model>)

    **减少回流与重绘**

    - 使用`transform`代替`top`后者会引起回流

    - 使用`visibility`替换`display: none`，因为前者只会引起重绘，后者会引起回流

    - 不要把节点属性值放入循环中当成变量

      ```javascript
      for(let i = 0; i < 1000; i++) {
          // 获取 offsetTop 会导致回流，因为需要去获取正确的值
          console.log(document.querySelector('.test').style.offsetTop)
      }
      ```

    - 尽量不要使用`table`布局，因为其小改动会造成整个`table`重新布局
    - 使用动画的速度跟回流次数成正比，尽量降低频率，或者使用`requestAnimationFrame`
    - CSS 选择符**从右往左**匹配查找，避免节点层级过多
    - 将频繁重绘或者回流的节点设置为图层，图层能够阻止该节点的渲染行为影响别的节点。比如对于 `video` 标签来说，浏览器会自动将该节点变为图层。可使用`will-change`和`video`,`iframe`生成新图层

22. **安全防范**

    1. **XSS攻击**：说白了就是想尽办法将可以执行的代码（比如说脚本代码）注入到网页中，又分为**持久型和非持久型**

    2. 持久型是攻击代码写入到了数据库中，可想而知，攻击范围会变大

    3. 非持久型则是一般通过**修改URL参数**的方式注入攻击代码，诱导用户访问链接从而进行攻击。一般`Chrome`浏览器可以帮用户自动防御此种攻击。

    4. 对于`XSS`攻击来说，一般有以下两种方法防御：

       - 转义字符，即转义输入输出的内容

       - CSP，建立白名单（推荐），即明确告诉浏览器哪些资源可以加载和执行，兼容性不错

       - 通常可用以下两种方式开启CSP：

         1. 设置HTTP header中的`Content-Security-Policy` [参考](<https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy>)
         2. 设置`meta`标签为`<meta http-equiv="Content-Security-Policy>"`

         ```http
         // 以http header 为例
         Content-Security-Policy: default-src 'self' // 只允许加载本站资源
         Content-Security-Policy: img-src https://* // 只允许加载https协议图片
         Content-Security-Policy: child-src 'none'  // 允许加载任何来源框架
         ```

    5. **CSRF攻击**，跨站请求伪造，原理是攻击者构造一个后端请求地址，诱导用户点击或通过某些途径自动发起请求，一般针对登录态。
    6. 防范：
       - get请求不对数据进行修改
       - 不让第三方网站访问到用户cookie
       - 阻止第三方请求接口
       - 请求时附带token或其他验证
    7. 实际操作
       - `same-site` 对cookie设置该属性可以表示cookie不随跨域请求发送，但是并不是所有浏览器兼容
       - 验证`Referer`判断请求是否是第三方网站发起
       - `token`，服务器下发一个token，每次请求都携带上，服务器验证token有效性

    8. **点击劫持**：利用`iframe`诱导用户点击

    9. 防范：`X-FRAME-OPTIONS`专为防御用`iframe`嵌套的点击劫持效应。

       ```http
       X-FRAME-OPTIONS: DENY，//表示页面不允许通过 iframe 的方式展示
       X-FRAME-OPTIONS: SAMEORIGIN //表示页面可以在相同域名下通过 iframe 的方式展示
       X-FRAME-OPTIONS: ALLOW-FROM //表示页面可以在指定来源的 iframe 中展示
       ```

    10. **中间人攻击**
        1. 攻击者同时与客户端和服务端建立了连接，并让对方都认为连接是安全的，但实际上整个过程都被攻击者控制了，攻击者可以获得和修改通信信息。如`公共Wifi`，`https`可以防中间人攻击，但前提是完全关闭了`http`访问，否则`https`可能被降级为`http`从而有可能遭受中间人攻击。



