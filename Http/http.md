# Http

#### UDP

1. `UDP`是面向无连接的，即不需要在式传递数据前连接双方，传递的报文不保证有序也不保证不丢失，并且没有控制流量的算法，相对`TCP`来说也更轻便。

2. 不需要和`TCP`一样通过三次握手建立联系，传递报文时也不会对其进行任何拆分和拼接操作。即在发送端，应用层将数据传递给传输层，使用的`UDP`协议只会给数据添加一个`UDP`头部标识，表示是使用的`UDP`协议，就传给网络层了。在接收端，网络层将数据传给传输层，`UDP`只去掉报文头就直接传给应用层，而不会做任何拼接操作。

3. 不可靠性。没有拥塞控制，网络不好可能会丢包，但对于**电话会议**这种使用场景还是有其优势。

4. `UDP`的头部开销小，只有8字节，相比于`TCP`的至少20字节少得多，因此传输是**高效**的。

5. ![img](https://user-gold-cdn.xitu.io/2018/5/1/163195b245ceb89c?imageslim)

   `UDP`头部包含：

   - 前16位包含源端口（可选）和目标端口
   - 整个数据报文的长度
   - 整个数据报文的检验和`IPV4`（可选），用于发现头部信息和数据中的错误

6. `UDP`还支持一对多。适用于**直播，实时性高的游戏（王者荣耀？）**

#### TCP

1. 全双工，建立和断开连接都需要握手，通过各种算法保证了数据的可靠性，相对来说也就没那么高效。

2. ![img](https://user-gold-cdn.xitu.io/2018/5/1/1631be45b084e4bc?imageslim)

   相对来说，`TCP`头部复杂的多：

   - `Sequence number`保证报文有序传输，对端也可以通过序号顺序拼接报文；

   - `Acknowledgment number`表示的是接收端希望接收的下一个字节编号是多少，同时也标示着上一个序号的数据已经收到；

   - `Window Size`表示能接收的字节数据的多少，便于流量控制。

   - 标识符：

     `URG = 1`：紧急信息

     `ACK = 1`：表示确认号字段有效

     `PSH = 1`：表示接收端应立即将该数据push到应用层，而不是等缓冲区满后再提交

     `RST = 1`：表示连接出现问题

     `SYN = 1`：当`SYN = 1， ACK = 0`时，表示当前报文段是连接请求报文，均为1时则是表示同意连接的应答报文

     `FIN = 1`：表示释放连接的请求报文

3. 状态机

   `RTT`：该指标表示发送端发送数据到接收端接收到数据所需的往返时间

4. 为什么 TCP 建立连接需要三次握手，明明两次就可以建立起连接？

   因为这是为了防止出现失效的连接请求报文段被服务端接收的情况，从而产生错误。

5. `ARQ`协议

   即超时重传机制，已保证数据的正确送达。其中又包括`停止等待ARQ`和`连续ARQ`协议两种。

6. 滑动窗口

   发送窗口和接收窗口大小是动态的，发送窗口会随着接收窗口的大小而调整，以达到一个流量控制的目的。发送端口包含已发送但没收到应答的数据和可以发送但还未发送的数据。此外还可能出现`Zero窗口`。

7. 拥塞处理

   拥塞处理和流量控制不同，后者是作用于接收方，保证接收方来得及接收到数据。而前者是作用于网络，防止过多的数据拥塞网络，避免出现网络负载过大的情况。

   拥塞处理包含四个算法：**慢开始、拥塞避免、快速重传、快速恢复**

   **慢开始**即是说一开始不会将带宽拉满，就像下载的时候速度是慢慢升起来的。

   **拥塞避免**每过一个 `RTT` 窗口大小只加一，这样能够避免指数级增长导致网络拥塞，慢慢将大小调整到最佳值

   **快速重传**一般和**快速恢复**一起出现。一旦接收端收到的报文出现失序的情况，接收端只会回复最后一个顺序正确的报文序号。如果发送端收到三个重复的` ACK`，无需等待定时器超时而是直接启动快速重传算法。包括`TCP Taho`和`TCP Reno`

**总结：**

> 建立连接需要三次握手，断开连接需要四次握手
>
> 滑动窗口解决了数据的丢包、顺序不对和流量控制问题
>
> 拥塞窗口实现了对流量的控制，保证在全天候环境下最优的传递数据

#### HTTP&TLS

1. `Http`请求由三部分组成：请求行（大概长这样`Get /images/logo.png HTTP/1.1`，基本由请求方式，URL和协议版本组成）、首部、实体。

2. `Get`和`Post`的区别

   首先引入**副作用**和**幂等**的概念，副作用是指对服务器上资源的修改，如搜索是无副作用的，注册是有副作用的。

   而幂等是说注册10个账号和注册11个账号是不幂等的，对文章更改10次和11次则是幂等的，前者是增加了一个资源，后者是更新了一个资源。

   所以，在规范的应用场景上说，`Get`多用于无副作用，幂等的场景，例如搜索，`Post`多用于副作用，不幂等的场景，如注册账号等。

   再者，`Get`请求可以缓存，而`Post`不能；

   `Post`相对于`Get`安全一点，因为`Get`请求多包含在`URL`中，当然也可以写在`body`中，且会被浏览器保存历史记录，而`Post`不会；

   浏览器对`URL`的长度有限制，会影响到`Get`请求；

   `Post`支持更多的编码类型且不对数据类型进行限制。

3. 首部

   | 通用字段          | 作用                         | 请求首部            | 作用 |
   | ----------------- | ---------------------------- | ------------------- | ---- |
   | Cache-Control     | 控制缓存行为                 | Accept              |      |
   | Connection        | 浏览器想要优先使用的连接类型 | Accept-Charst       |      |
   | Date              | 创建报文的时间               | Accept-Encoding     |      |
   | `Pragma`          | 报文指令                     | Accept-Language     |      |
   | Via               | 代理服务器相关信息           | Expect              |      |
   | Transfer-Encoding | 传输编码方式                 | From                |      |
   | Upgrade           | 要求客户端升级协议           | Host                |      |
   | Warning           | 内容中可能存在错误           | If-Match            |      |
   |                   |                              | If-Modified-Since   | 时间 |
   |                   |                              | If-None-Match       | 标记 |
   |                   |                              | `Referer`           |      |
   |                   |                              | Range               |      |
   |                   |                              | Proxy-Authorization |      |

   |                    | 作用             | 实体头部         | 作用                           |
   | ------------------ | ---------------- | ---------------- | ------------------------------ |
   | Accept-Ranges      |                  | Allow            | 资源的正确请求方式             |
   | `ETag`             | 资源标识         | Content-Encoding |                                |
   | Location           | 重定向地址       | Content-Language |                                |
   | Proxy-Authenticate | 代理验证         | Content-Length   |                                |
   | Server             |                  | Content-Location | 返回数据的备用地址             |
   | WWW-Authenticate   | 资源验证         | `Content-MD5`    | Base64加密格式的内容 MD5检验值 |
   | Age                | 代理缓存存在时间 | Content-Range    | 内容的位置范围                 |
   |                    |                  | Expires          | 内容过期时间                   |
   |                    |                  | Last_modified    |                                |

4. `TLS`

   `HTTPS`是通过`HTTP`传输信息，但是使用了`TLS`协议进行了加密。位于传输层之上，应用层之下，其中使用到了两种加密技术，即对称加密和非对称加密。

#### HTTP/2&HTTP/3

1. HTTP/2很好的解决了当前常用的`HTTP/1`的一些性能问题，但兼容等问题使得它并不普及。相较来说，`HTTP/2`大幅度提高了网页的性能。

2. `HTTP/2`中使用了**多路复用**的技术，这个技术可以只通过一个`TCP`连接就传输所有的请求数据。该技术很好的解决了浏览器限制同一个域名下的请求数量被限制的问题，同时也间接更容易实现全速传输，因为新开连接后的传输速度是慢慢提升的。

   

   **多路复用**：帧代表着最小的数据单位，每个帧会标识出该帧属于哪个流，流也即是多个帧组成的数据流。在一个`TCP`连接中，可以存在多条流。换句话说，就是可以发送多个请求，对端可以通过帧中的标识知道其属于哪个请求，从而避免了**队头阻塞**的问题，极大提高性能。通过 [该链接](<https://http2.akamai.com/demo>) 感受下 HTTP/2 比 HTTP/1 到底快了多少。

   

   **Header压缩**：如果`header`中携带了`cookie`那么每次请求都需要重复传输几百到几千字节。而在`HTTP/2`中，使用了`HPACK`压缩格式对其进行编码，减少了`header`的大小，并在两端维护了一个索引表，记录出现过的`header`，后面在传输中就可以传输已经记录过的`header`的键名，对端接收到数据后通过键名找到对应的值。

   

   **服务端PUSH**：在` HTTP/2` 中，服务端可以在客户端某个请求后，主动推送其他资源。减少了延迟，在浏览器兼容的情况下也可考虑`prefetch`

   

3. 相比于`HTTP/1`的文本方式传输，`HTTP/2`引入的新的编码机制，使得传输数据被分割，并采用二进制编码格式，成了加强其性能的核心点。

4. `HTTP/3`：Google搞了一个基于`UDP`协议的`QUIC`协议，使用在了`HTTP/3`上。从2到3是由于2的多路复用在一个域名下只需要使用一个`TCP`连接，当这个连接出现了丢包的情况，整个`TCP`都要进行重传，后面的数据都将被阻塞，其表现将不如1，这样就有人会想去修改`TCP`协议，但是这个协议是基于操作系统实现的，充斥在各种设备中，修改不易，所以有了谷歌的另起炉灶。

5. `QUIC`新增了多路复用、`0-RTT`、`TLS1.3`加密、流量控制、有序交付、重传等功能，

   **多路复用**：因为`TCP`是基于**`IP`和端口**去识别连接的，在多边的移动网络环境下就会变得很脆弱。而`QUIC`是通过`ID`的方式去识别一个连接，只要`ID`不变，就能连上。所以在移动端，`QUIC`的表现比`TCP`好。且其原生就实现了多路复用，传输的单个数据流可以保证有序交付，不会影响其他的数据流，从而解决了`TCP`存在的问题。

   **`0-RTT`**：通过使用类似`TCP`快速打开的技术，缓存当前会话的上下文，在下次会话恢复的时候，只需要将之前的缓存发给服务器验证通过就可以继续传输。

   **纠错机制**：丢失一个包的情况下适用，但是丢失多个包就需要使用重传了。

总结：

> HTTP/2 通过多路复用、二进制流、Header 压缩等等技术，极大地提高了性能，但是还是存在着问题的
>
> `QUIC` 基于 `UDP` 实现，是 HTTP/3 中的底层支撑协议，该协议基于 `UDP`，又取了`TCP` 中的精华，实现了即快又可靠的协议

#### RabbitMQ

1. 它是一个由`erlang`（有着和原生socket一样低延迟）语言开发的基于`AMQP`协议的开源消息队列系统，
2. 类似的消息中间件还有`RocketMQ`(阿里开源)，`ActiveMQ`(Apache)，`Kafka`(Apache，开源流处理平台，`Scala和Java`编写，高吞吐量的分布式发布订阅系统，支持单机每秒百万并发，不支持事物)
3. 应用场景：
   - 同步转异步：如短信发送，没必要的同步处理的可使用`MQ`异步处理，
   - 应用解耦：如商城业务中的订单系统和库存系统，A出现问题而不会影响B
   - 限流：通过设置阈值限制短时间的流量过大

4. `MQ`的时间解耦和空间解耦

   消息的生产者不用去关心什么时候消息被消费，消费者亦是，两者之间不会强依赖，从而实现了空间上的解耦；

   消息的生产者只负责将消息放入队列，而不用关心消费者什么时间去消费，消费者又可以根据自己需要（缓冲区）来选择即时消费还是延迟消费，即都有了自己的生命周期，实现了时间上的解耦。

#### Axios

1. 请求格式

   ```javascript
   axios.get('../static/data.json', {
         params: {id: 12}
       }).then(res => {
         console.log(res)
       })
   
       axios({
         methods: 'get',
         url: '../static/data.json',
         params: {id: 12}
       }).then(res => {
         console.log('methods')
       })
   
       let formdata = new FormData()
       axios.post('./post', formdata).then(res => {
         console.log(res)
       })
   
       axios.delete('/delete', {
         params: {id: 12}
       }).then(res => {
         console.log(res)
       })
   
       axios.delete('/delete', {
         data: 'fas'
       }).then(res => {
         console.log(res)
       })
   
       // 并发请求
       axios.all([
         axios.get('../static/data.json'),
         axios.get('../static/city.json')
       ]).then(
         axios.spread((dataRes, cityRes) => {
           console.log(dataRes, cityRes)
         })
       )
   ```

2. 基本配置和拦截器

   ```javascript
   let instance = axios.create({
         baseUrl: 'http://localhost:8080',
         timeout: 1000,
         methods: 'get,post, put， delete, patch',
         url: '../static/data.json',
         params: {},
         data: {},
         headers: {
           token: ''
         }
       })
   
       // axios全局配置
       axios.defaults.timeout = 1000
       axios.defaults.baseURL = 'http://localhost:8080'
   
       // axios实例配置
       let instance1 = axios.create()
       instance1.defaults.timeout = 3000
   
       // 请求配置
       instance.get('../static/data.json', {
         timeout: 5000
       }).then(res => {
         console.log(res)
       })
   
       // 优先级：请求配置>实例配置>全局配置
   
       // 实际开发
       let instance2 = axios.create({
         baseURL: '',
         timeout: 4000
       })
       let instance3 = axios.create({
         baseURL: '',
         timeout: 3000
       })
   
       instance2.get('/xxx').then(res => {
         params: {
           id: 12
         }
       })
   
       instance3.get('/xxx').then(res => {
         timeout: 4000
       })
   
   
       // 拦截器: 请求或响应被处理前拦截，分两种
       // 请求拦截器
       axios.interceptors.request.use(config => {
         config.headers = {
           auth: 'xiaopp'
         }
         return config
       }, err => {
         return Promise.reject(err)
       })
   
       // 响应拦截器
       axios.interceptors.response.use(res => {
         return res
       }, err => {
         return Promise.reject(err)
       })
   
       // 取消拦截器，声明，然后eject该拦截器
       let interceptors = axios.interceptors.request.use()
       axios.interceptors.request.eject(interceptors)
   
       axios.get('../static/data.json', {
         timeout: 5000
       }).then(res => {
         console.log(res)
       })
   
   
       // 实际开发，需要登录接口和不需要登录接口
       let instance4 = axios.create({})
       instance4.interceptors.request.use(config => {
         config.headers.token = 'sss'
         // 不能使用
         // config.headers = {
         //   token: ''
         // }
         return config
       })
   
       // 移动端开发
       let instance_phone = axios.create({})
       instance_phone.interceptors.request.use(config => {
         // 请求前展示请求状态框
         console.log('show')
         return config
       })
       instance_phone.interceptors.response.use(res => {
         // 隐藏请求状态框
         console.log("hide")
         return res
       })
   
       instance_phone.get('../static/data.json', {
         timeout: 5000
       }).then(res => {
         console.log(res)
       })
   ```

3. 错误处理和取消请求

   ```javascript
   axios.interceptors.request.use(config => {
         return config
       }, err => {
         // 请求错误401超时， 404
         return Promise.reject(err)
       })
   
       axios.interceptors.response.use(res => {
         return res
       }, err => {
         // 500系统错误 502系统重启
         return Promise.reject(`${err}>>>>`)
       })
   
       axios.get('/xxx').then(res => {
         console.log(res)
       }).catch(err => {
         console.log('error:', err)
       })
   
   
       // 取消请求
       let source = axios.CancelToken.source()
       axios.get('/xxx', {
         CancelToken: source.token
       }).then(res => {
         console.log(">>>", res)
       }).catch(err => {
         console.log("err>>>", err)
       })
   
       source.cancel('cancel http message')
     }
   ```

   

4. 封装

***

**TCP/IP协议族**

1. 分层管理

   TCP/IP协议族里重要的一点就是分层，好处是解藕，协议族以下有四层：

   - 应用层：
   - 传输层：
   - 网络层：
   - 数据链路层：

2. **与HTTP关系密切的协议：IP、TCP和DNS**

3. http1.1浏览器缓存

   第一次请求，浏览器通过请求头部header，附带expires, cache-control, Last-Modified/Etag向服务器发起请求，此时服务器记录第一次请求的Last-Modified/Etag。

   浏览器再次请求时，依然附带如上信息向服务器发起请求

   服务器根据第一次记录的Last-Modified/Etag和再次请求的If-Modified-Since/Etag判断资源是否更新，如果未更新，则返回304，不需要重新下载

   做缓存的优点：

   - 服务器响应时间更快
   - 减少网络带宽消耗