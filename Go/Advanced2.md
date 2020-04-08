# Advanced2

***

**Golang并发**

1. Go 语言通过编译器运行时（runtime），从语言上支持了并发的特性，由goroutine特性完成，**goroutine由语言运行时调度完成，而线程是由操作系统调度完成**，此外，Go 语言还提供 **channel** 在多个 goroutine 间进行通信。goroutine和channel是golang秉承并发模式的重要实现基础。

2. go从语言层面就支持并发，同时实现了自动垃圾回收机制

   进程和线程：

   进程是程序在操作系统中的一次执行过程，系统进行资源分配和调度的一个独立单位

   线程是进程的一个执行实体，是CPU调度和分派的基本单位

   并发和并行：

   多线程在单核CPU上运行，称为并发（主要通过切换时间片来实现“同时”运行）

   多线程程序在多核CPU上运行，称为并行

   协程和线程：

   协程：独立的栈空间，共享堆空间，调度由用户自己控制

   线程：一个线程上可以跑多个协程，协程是轻量级的线程

   **goroutine:**

   说到底就是线程，但是比线程小，十几个 goroutine 可能体现在底层就是五六个线程，而且Go语言内部也实现了 goroutine 之间的内存共享。

   **channel**（联想Unix管道）

   是go在语言级别提供的goroutine的通信方式，channel是进程内的通信方式，因此channel传递对象的方式和函数的传参方式基本是一致的，（可以传递指针等）

   如果需要跨进程通信，可以使用分布式系统的方法来解决，如使用HTTP/Socket，go对网络方面也有非常完善的支持

   并发程序的优点：

   - 客观表现问题模型
   - 充分利用CPU多核优势
   - 充分利用CPU与硬件固有的异步特性

3. goroutine创建

   - 通过普通函数创建
   - 通过匿名函数创建

   所有goroutine在main函数结束时才会一同结束

   Goroutine类似于线程概念，但是没有线程细致，细致程度取决于go程序的goroutine的调度器实现和运行环境

   终止 goroutine 的最好方法就是自然返回 goroutine 对应的函数。

4. 并发通信

   两种最常见的**并发通信模型：**共享数据和信息

   go提供另一种通信模型：即以消息机制而非共享内存作为通信方式

   **消息机制认为每个共享单元都是自包含的、独立的个体，他们有自己的变量，但在不同并发单元中这些变量不共享，每个单元的输入和输出只有一种，即是消息。类似于进程，不同进程间各自负责自己的任务就OK，不共享内存。**

   共享数据是指多个并发单元保持对同一个数据的引用，实现对该数据的共享，共享的数据可能有多种格式，如内存数据块、网络数据、磁盘文件等，最常用的是共享内存。

   并发编程难点在于协调调度，协调就要通过交流，因此从某种程度上来说，并发单元间的通信是最大的问题

5. 并发中的资源竞争

   有并发，就有资源竞争。比如同时对一个资源进行读写，这时候就会产生资源竞争（对于同一个资源的读写必须是原子化的，也就是说，同一时间只能允许有一个goroutine对共享资源进行读写操作。）

   `go build -race`运行生成的可执行文件，可以看到打印出的检测信息，可以检测共享资源竞争的问题

   go提供了传统的同步groutine机制，就是对共享资源加锁。

   `atomic 和 sync`原子函数和互斥锁

6. 调整运行时的并发性能

   `runtime.GOMAXPROCS(runtime.NumCPU())`

   Go 在GOMAXPROCS数量和任务数量相等时，可以做到并行执行，大多数情况下都是并发执行

7. goroutine和coroutine（Lua）的区别

   Coroutine运行机制属于协作式任务处理，需要程序主动交出CPU的使用权，而这在开发者未在程序完成时设置交出使用权的时候就容易出现死机或失去响应。

   goroutine属于抢占式任务处理，执行权交由操作系统，当某个任务处理时间过长，长时间占用大量CPU时，操作系统有权终止该任务。

8. **channel**（FIFO队列）

   go提倡用通信的方法代替共享内存，当一个资源需要在多个goroutine之间共享时，通道在goroutine之间架起一条管道，并确保数据能同步交换，声明通道（引用类型make）时，需要指定将要被共享的数据的类型。

   在任何时候，只能有一个goroutine访问通道进行发送和获取数据，goroutine间通过通道就可以通信。

   ```go
   ch := make(chan interface{})
   ch <- 0 // 发送0到接口通道
   ch <- "hello" // 发送字符串到接口通道
   data := <-ch // 执行到这里会阻塞，直到接收到数据并赋给data为止
   data, ok := <-ch // 非阻塞接收数据
   <- ch // 执行时发生阻塞，直到接收到数据，但会忽略接收的数据
   
   func main() {
     ch := make(chan int)
    // 开启一个匿名函数并发
     go func() {
       fmt.Println("start")
       ch <- 0
       fm.Println("end")
     }()
     fmt.Println("wait goroutine") // 匿名函数结束时通知另一个goroutine
     <- ch // 等待匿名goroutine
     fmt.Println("all done")
   }
   // wait goroutine
   // start
   // end
   // all done
   ```

   **使用通道接收数据**

   - 通道的收发操作需要在不同的两个goroutine中进行

     如果发送方一直发送，而没有接收方处理，会造成阻塞，因此通道的接收方需要在另一个goroutine中进行

   - 接收方将持续阻塞直到发送方发送数据

     如果接收数据时，发送方一直没有发送数据，接收方也会发生阻塞，直到发送方发送数据为止

   - 每次只接收一个数据

     阻塞接收数据

     非阻塞接收数据

     接收任意数据，但忽略接收的数据

     循环接收// 通道ch是可以进行遍历的

   **单向通道**

   通道本身是支持同时读写的，否则根本没法用

   声明方式：

   >var 通道实例 chan <- 元素类型 // 只能发送
   >
   >var 通道实例 <- chan 元素类型 // 只能接收

   ```go
   var ch = make(chan int)
   var sendOnly chan <- int = ch
   var receiveOnly <- chan int = ch
   // 在声明时即创建单向通道
   var ch1 = make(chan <- int)
   var readOnly chan <- int = ch1
   chan <-
   ```

   time包中的单向通道

   关闭通道：`close(ch)`

   `x, ok := <- ch`通过ok这个布尔值判断，为false则已关闭

   **无缓冲通道**

   要求收发双方都要准备好，才可以执行收发操作，因为其在接收前没有任何能力保存值。如果没有同时准备好，都可能发生阻塞。

   无缓冲的通道保证进行发送和接收的 goroutine 会在同一时间进行数据交换。

   模拟一个🎾网球比赛的场景，两个选手即是两个goroutine，而球是消息，选手要么在接球的状态，要么在发球的状态，球（消息）不可能说打到空中暂停了，即是说没有有缓冲。

   **带缓冲通道**

   在通道中没有要接收的值时，接收操作才会阻塞，通道中没有可缓冲区容纳被发送的值时，发送动作会阻塞（即，取不到值，和缓冲区满发不进去）

   `var 通道实例 := make(chan 通道类型, 缓冲大小)`

   无缓冲通道可以看作是缓冲大小永远为0的带缓冲通道

   为什么要设置缓冲大小？而不是无限制？

   因为当提供方的数据发送速度大于接收方的处理速度，通道不限制长度，会导致内存膨胀到应用崩溃。

   （供给量需要和消费方+缓冲大小之间做一个平衡）

   **通道的超时机制**

   go并没有提供直接的超时处理机制，所谓的超时可以理解为网站登录时间过长时会要求你重新登录

   这时可以使用select来处理超时

   select 的特点是只要其中有一个 case 已经完成，程序就会继续往下执行，而不会考虑其他 case 的情况

   ```go
   select {
     case <- chan1: 
     // 处理 ，每个case语句里必须有一个I/O操作，防止通道阻塞
     case chan2 <-1: 
     //
     default: // 保证了至少有一个可以进行下去
     //
   }
   ```

9. 多核并行化

   虽然goroutine简化了我们写并行代码的过程，但是实际运行效率可能并不真正高于单线程程序，你可以运行多个goroutine，从运行状态看也的确在并行运行，但是实际上这些goroutine都运行在一个CPU上。

10. 等待组（sync.WaitGroup)

    Go除了可以使用通道和互斥锁进行两个并发程序同步外，还可以使用等待组进行多个任务的同步

11. **死锁，活锁，饥饿锁**

    死锁是指两个或两个以上进程（或线程）在执行过程中，因为争夺资源而造成的一种相互等待的现象。若无外力作用，它们都将无法推动进行下去，这种一直卡着的相互等待的进程即为死锁进程。

    死锁发生的条件：

    - 互斥条件：一个线程占用了某个资源，其他的线程都处于等待状态，直到资源被释放
    - 请求和保持条件：T1占用了R1，还想占用R2，但是R2被T2占用了，于是T1继续等待，等待过程中还不放开R1
    - 不剥夺条件：说白了就是资源的使用权在自己手里，别人不能抢，只能用完后自己释放
    - 环路等待条件：T1在等待T2占用的资源，T2在等待T1占用的资源

    死锁解决办法：

    - 并发查询多个表时约定顺序
    - 同一个事务中，尽可能做到一次锁定需要获取的资源
    - 对容易产生死锁的业务场景，升级颗粒度，如使用表级锁
    - 使用分布式事务锁或者乐观锁

    活锁：即是指线程一直在重复同样的操作，如T1，T2，T3等线程都需要资源R1 ，但都让来让去，最后谁都没用上，通常发生在处理事务消息中，活锁可能自动解开，死锁却不能。

    饥饿：是指一个线程本可以继续执行，但是被调度器无限期的忽视而不能执行

    活锁与饥饿无关，活锁最终是没有完成任务的，而饥饿通常意味着一个或多个贪婪的进程，妨碍了其他线程或进程的继续执行。

    **总结：**

    - 死锁的原因是因为用锁不当
    - 活锁，是逻辑上感觉对，实际上可能在重复做某个操作
    - 与锁使用的粒度有关，通过计数取样，可以判断进程的工作效率

    只要是共享资源，都必定要使起其逻辑上进行顺序化和原子化，这避不开锁的使用。

12. 通信顺序进程

    并发编程中，对共享资源的精确控制访问，在绝大多数语言中，都是通过加锁等线程同步方案解决，而go是将共享值通过通道传递。

    go实现了两个并发形式，一种是类似java c++的多线程共享内存，一种是CSP（communicating sequential process)并发模型（未完全实现）（process -> grouting -> channel)

    **并发编程的核心概念是同步通信。**

***

**接口**

1. 本身是调用方和实现方都需要遵守的一种协议（protocol)

   非侵入式设计是 Go 语言设计师经过多年的大项目经验总结出来的设计之道，实现真正地解耦，提高编译速度，

   **接口实现是隐式的，无须让实现接口的类型写出实现了哪些接口。这个设计被称为非侵入式设计**

2. 接口定义：

   go不是一种传统的面向对象的编程语言，没有类和继承等概念

   ```go
   type Writer struct {
     Write(p []byte) (n int, err error)
   }
   ```

3. 接口实现的条件：

   接口的方法和实现接口的类型方法格式一致

   接口中所有的方法均被实现

   开发者唯一需要关注的就是**“我需要什么？”，以及“我能实现什么？”。**

   一个类型可以实现多个接口，多个类型可以实现相同接口

4. 类型断言

   `value, ok := x.(T)`

5. 接口嵌套组合

   系统包中的嵌套组合

   ```go
   type Write interface {
     Write(p []byte) (n int, err error)
   }
   type Closer interface {
     Close() error
   }
   type WriteCloser interface {
     Write // 注意没有括号
     Closer
   }
   // 声明一个设备结构，然后实现写和关闭方法
   type device struct {
     
   }
   func (d *device) Write(p []byte) (n int, err error) {
     return 0, nil
   }
   func (d *device) Close() error {
     return nil
   }
   func main{
     var wc io.WriteCloser = new(device)
     wc.Write()
     wc.Closer()
     var writeonly io.Write = new(device)
     writeonly.Write(nil) // 只有写方法
   }
   ```

6. 类型分支判断基本类型/接口类型

   ```go
   // 伪代码
   struct => func () a()
   type b nterface {
   	a()
   }
   // 分支判断
   case b:
   ```

7. 空接口类型

   ```go
   // 空接口保存值
   var any interface{}
   any = 1
   any = "hello"
   // 从空接口取值
   var a int = 1
   var i interface{} = a
   var b int = i.(int) // 这里若不使用类型断言会报错，因为i之前还是interface{}类型
   ```

   空接口可以比较，但是不能比较空接口中的动态值，如函数，切片，map，通道，数组，结构体

8. 接口和类型之间的转换

   go中接口可以转换成另外的接口，也可以转换为另外的类型

   interface里面放函数，通过struct完成函数

   接口在转换为其他类型时，接口内保存的实例对应的类型指针，必须是要转换的对应的类型指针

***

**文件处理**

1. encoding/json, encoding/xml

   os.Create()，os.Open()

2. encoding/binary, bytes

3. archive/zip   archive/tar

4. Go文件的写入、追加、删除、复制操作

   - O_RDONLY
   - O_WRONLY
   - O_RDWR
   - O_APPEND：写操作时追加到文件尾部
   - O_CREATE：如果不存在则创建
   - O_EXCL：和create一起使用，文件不存在时必须返回一个错误
   - O_SYNC：执行一系列写操作时，每次都要等待上次的 I/O 操作完成再进行
   - O_TRUNC：如果可能，在打开时清空文件

5. 文件锁flock

   多个进程同时操作同一份文件的情况，这很容易导致文件中的数据混乱

   文件锁的常见例子如Nginx，进程运行起来后就会把当前的 PID 写入这个文件，如果文件已经存在，就是说前一个进程还没有退出，那么Nginx就不会重启。所以锁还可以用来检测进程是否存在。

***

**反射**

1. 定义

   反射是指在程序运行期间对程序本身进行访问和修改的能力。程序在编译时，变量转换为内存地址，变量名不会被编译到可执行部分。在程序运行时，程序也无法获取自身的信息。

   Lua，JavaScript 类动态语言，由于其本身的语法特性就可以让代码在运行期访问程序自身的值和类型信息，因此不需要反射系统

   反射功能强大但代码可读性并不理想，若非必要并不推荐使用反射

   **reflect包**

   ```go
   const (
   	Zero Enum = 0
   )
   type cat struct {
   }
   func main() {
   	var a int
     typeOfA := reflect.TypeOf(a)
     typeOfB := reflect.TypeOf(Zero)
     typeOfcat := reflect.TypeOf(cat{})
     ptr := &cat{}
     typeOfPtr := reflect.TypeOf(ptr)
     fmt.Println(typeOfPtr.Name(), typeOfPtr.Kind()) // 空 ptr
     fmt.Println(typeOfA.Name(), typeOfA.Kind()) // cat struct
     fmt.Println(typeOfA.Name(), typeOfA.Kind()) // Enum int
     fmt.Println(typeOfA.Name(), typeOfA.Kind()) // 类型名和种类名，均为int
   }
   ```

   对指针获取反射对象时，可以通过 reflect.Elem() 方法获取这个指针指向的元素类型，这个获取过程被称为取元素，等效于对指针类型变量做了一个`*`操作。

2. 反射三定律

   第一定律：反射可以将“接口类型变量”转换成“反射类型对象”

   反射类型指 reflect.Type 和 reflect.Value

   ```go
   func main() {
       var x float64 = 3.4
       fmt.Println("type:", reflect.TypeOf(x))
       fmt.Println("value:", reflect.ValueOf(x))
       v := reflect.ValueOf(x)
       v.CanSet() // false 对应第三定律
       p := reflect.ValueOf(&x)
       e := p.Elem() // 取值指向的元素值，类似于语言层*操作
       e.CanSet() // true
       e.SetFloat(2.0)
       fmt.Println(v.Interface()) // 2.0
       fmt.Println(x) // 2.0
   }
   ```

   reflect.TypeOf(x) 时，x 被存储在一个空接口变量中被传递过去，然后 reflect.TypeOf 对空接口变量进行拆解，恢复其类型信息。

   第二定律：反射可以将“反射类型对象”转换为“接口类型变量”

   第三定律：如果要修改“反射类型对象”其值必须是“可写的”

   如果想要改变，参考函数的传参，参数若要改变，需要传参数的地址，f(&x)，反射机制与此相同，要把想要修改的变量的指针传递给反射库。

   使用反射修改结构体的字段，只要有结构体的指针，我们就可以修改它的字段，如下

   ```go
   type T struct {
     A int
     B string
   }
   t := { 20, "2oops"}
   e := reflect.ValueOf(&t).Elem()
   typeOfT := e.Type()
   for i := 0; i < e.NumField(); i ++ {
     f := e.Field(i)
     fmt.Println(i, typeOfT.Field(i).Name, f.Interface()) // 0 A int 20.  1 B string 2oops
   }
   e.Field(0).SetInt(21)
   e.Field(1).SetString("oops")
   fmt.Println(t) // {21 oops}
   ```

3. 通过反射获取结构体成员类型

4. 结构体标签

   ```go
   func main() {
       type cat struct {
           Name string
           Type int `json:"type" id:"100"` // 注意json:后面不能有空格
       }
       typeOfCat := reflect.TypeOf(cat{})
       if catType, ok := typeOfCat.FieldByName("Type"); ok {
           fmt.Println(catType.Tag.Get("json"))
       }
   }
   ```

5. 从反射值获取被包装的值

   ```go
   func main() {
       // 声明整型变量a并赋初值
       var a int = 1024
       // 获取变量a的反射值对象
       valueOfA := reflect.ValueOf(a)
       // 获取interface{}类型的值, 通过类型断言转换
       var getA int = valueOfA.Interface().(int)
       // 获取64位的值, 强制类型转换为int类型
       var getA2 int = int(valueOfA.Int())
       fmt.Println(getA, getA2)
   }
   ```

6. IsNil()和IsValid()——判断反射值的空和有效性

   IsNil()如果值类型不是通道（channel）、函数、接口、map、指针或 切片时发生 panic

   ```go
   var a *int
   fmt.Println(reflect.ValueOf(a).IsNil()) // true
   fmt.Println(reflect.ValueOf(nil).IsValid()) // false
   ```

7. 通过类型信息创建实例

   ```go
   func main() {
       var a int
       // 取变量a的反射类型对象
       typeOfA := reflect.TypeOf(a)
       // 根据反射类型对象创建类型实例
       aIns := reflect.New(typeOfA)
       // 输出Value的类型和种类
       fmt.Println(aIns.Type(), aIns.Kind()) // *int ptr
   }
   ```

8. 通过反射调用函数

   ```go
   func add(a, b int) int {
       return a + b
   }
   func main() {
       // 将函数包装为反射值对象
       funcValue := reflect.ValueOf(add)
       // 构造函数参数
       paramList := []reflect.Value{reflect.ValueOf(10), reflect.ValueOf(20)}
       // 反射调用函数
       retList := funcValue.Call(paramList)
       // 获取第一个返回值, 取整数值
       fmt.Println(retList[0].Int()) // 30
   }
   ```

9. Inject库 依赖注入

***

**编译与工具**

1. go程序编写基本与源码的方式，GOPATH 作为工作目录和一套完整的工程目录规则，因此几乎不用额外配置

   go build -o main exe main.go

   go build -p 8 开启8核并发编译

   go build -x 打印编译时会用到的所有命令

   go build -v 编译时显示包名

   go build -race 开启竞态检测

   go build -a 强制重新构建

2. go clean -i -n（java maven clean)

   移除当前源码包和关联源码包里面编译生成的文件，包括以下几种：

   - go build 生成的可执行文件及临时目录中的文件（`.o`,`.5`……）
   - go test生成的`.test`， windows -> `.test.exe`
   - go install 安装当前代码包时产生的结果文件/ pkg/ bin

   参数解释：

   - -i 清除安装的包和可运行文件，即go install 安装的文件
   - -n 打印清除时用到的执行（rm rf等等）,但不执行清除操作
   - -x 打印清除时的指令并执行清除操作
   - -r 循环清除import引入的包
   - -cache 删除go build时产生的缓存
   - -testchace 删除当前包所有的测试结果

3. go run

   其不会在运行目录下生成任何文件，之所以能看到结果是执行的放在临时文件中的可执行文件，且后边添加的参数会以命令行输入提供给程序

   不能使用`go run +包`的形式，如果需要这种效果，可以先build，在运行可执行文件

4. gofmt

   cli(Command language interpreter)命令语言解释程序，优先读取标准输入。

   gofmt用tab表示缩进，不限制宽度，不会强制合并已换行的代码

   传入文件路径，则格式化路径文件，传入目录，格式化目录下所有.go文件，不传，则格式化命令目录下所有.go文件

   `go fmt`是gofmt的简单封装 -n（打印出内部要执行的`go fmt`的命令） -x （打印且执行）

   - -l 仅把那些需要改写的源文件的绝对路径打印到标准输出，而不是改写后的内容
   - -w 把改写后的内容直接写到文件中，而不是作为结果打印到标准输出
   - -r rewrite重写规则，形如： gofmt -w -r "a + b -> b + a" main.go

   - -d 只把diff后的对比信息打印到标准输出
   - -e 打印所有语法错误，如果不使用此标记，只会打印每行的第一个错误，且只打印前十个错误
   - -comments 是否保留注释，默认为true
   - -tabwidth 默认为8，使标记生效要设置成false

   - -tabs 是否使用 tab（'\t'）来代替空格表示缩进，默认为true
   - -cpuprofile 是否记录CPU使用情况，并将记录内容保存在此标记值所指的文件中

5. go install

   将编译的中间文件放到GOPATH的pkg目录下，以及固定地将编译结果放在 GOPATH 的 bin 目录下

   实际上完成了两步操作：生成执行文件，移动文件到目标目录下

   该命令基于GOPATH，所以独立的目录是无法使用该命令

   bin目录下的可执行文件的名称来自于编译时的包名

   bin目录是固定的，无法使用-o来更改目标目录

6. go get

   远程拉取或更新代码包及其依赖包，并自动完成编译和安装

   也是分了两步：下载源码包，执行go install

   - -d downlod只下载不安装
   - -u 强制使用网络更新包和依赖，下载丢失的包，但不会更新已经存在的包
   - -f 必须包含-u，阻止-u去校验每个import中的包都已经获取了，适合fork项目的包、依赖更新
   - -t 下载测试所需的包
   - -fix 获取源码之后先fix 
   - -v 显示执行的命令
   - -insecure 允许http方式下载

   `$ go get github.com/2oops/go-test`获取源码并编译

7. go generate 在编译前自动生成某类代码

   Unicode， HTML

   - run 仅执行匹配了正则的命令
   - -n -x -v

   ```go
   package main
   import "fmt"
   //go:generate go run main.go
   //go:generate go version
   func main() {
       fmt.Println("http://c.biancheng.net/golang/")
   }
   ```

   运行`go generate -x`

   输出：go run main.go

   http://c.biancheng.net/golang/

   go version

   go version go1.13.7 darwin/amd64

8. go test

   go有单元和性能测试系统

   单元测试：

   ​	标记单元测试结果：

   ​	t.FailNow() 标记并终止当前用例

   ​	t.Fail()标记但不终止当前用例

   ​	t.Log()  t.Logf()  t.Error()  t.Errorf() 

   ​	t.Fatal()打印致命错误并结束测试

   ​	t.Fatalf()

   基准测试：

   ​	测试代码需要保证函数可重入性及无状态，即测试代码不使用全局变量等带有记忆性质的数据结构

   原理：框架对一个测试用例的默认时间是1s，开始时，用例函数返回时不到1s，那么N值将不断增加，1，2，5，10，20，50...，同时递增后的值重新调用基准测试用例函数

   控制计时器

   b.ResetTimer()  StopTimer()   StartTimer()   计数器内部不仅包含耗时数据，还包括内存分配的数据

9. gopprof

   graphviz

   github.com/pkg/profile 