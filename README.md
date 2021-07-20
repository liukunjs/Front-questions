* debounce 和 thrrow

#####原理
1.throttle 比如电梯。保证如果电梯第一个人进来后，15秒后准时运送一次，不等待。如果没有人，则待机。
2.debounce 如果电梯里有人进来，等待15秒。如果又人进来，15秒等待重新计时，直到15秒超时，开始运送。

```
// input 输入的时候
function debounce (fun,time){
    let timer = null
    return function(...arg){
        // 下次来的清理到上次的
        clearTimeout(timer)
        timer = setTimeout(()=>{fun.apply(this,arg)},time)
    }
}
const input = document.getElementById("input")
function inputFun(){
    console.log(1111)

}
input.addEventListener("input",thrrow2(inputFun,2000))
function thrrow(fun,dely){
    let canrun = true
    return function(...arg){
        if(canrun){
            canrun = false
            fun.apply(this,arg)
            // 利用 setTimeout 做开关
            setTimeout(()=>{
                canrun = true
            },dely)
        }
    }
}
// 利用时间差计算
function thrrow2(fun,dely){
    let time = new Date()
    return function(...arg){
        let newDate = new Date()
        if(newDate-time>dely){
            time = new Date()
            fun.apply(this,arg)
        }
    }
}
```
* Set  Map weakSet  weakMap
#### Set
Set 更像数组，但是值是唯一的，且无序的
Set(iterator)  构造函数入参只支持 iterator 可迭代的对象,
```
  let set = new Set([1,2,3,4])
  let set2 = new Set("1234")
````
![image.png](https://upload-images.jianshu.io/upload_images/25862823-509af6da3e5e7a8a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
###### set 对象如果建立set
 利用iterator 实现
```
let obj = {name:"小明",age:"11"}
obj[Symbol.iterator] = function(){
    let list = []
    Object.keys(obj).forEach((item)=>{
         list.push([item,obj[item]])
    }
    )
    return {
        list,
        index:0,
        len:list.length,
        next(){
            if(this.index<this.len){
                return{
                    value:this.list[this.index++],
                    done:false
                }
            }
            return {
              done:true,
              value:undefined  
            }
        }
    }
}
for(var i of obj){
    console.log(i)
}
let m = new Set(obj)
```
![image.png](https://upload-images.jianshu.io/upload_images/25862823-e22420355326c8ef.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
######  实例方法
```
// has
// get 
// delete
// forEach
// keys
// values
// entries
// keys 和 values 一样
// entries 返回的{key,value}一致
```
###### Map 
 共同点：集合、字典 可以储存不重复的值
不同点：集合 是以 [value, value]的形式储存元素，字典 是以 [key, value] 的形式储存
```
let john = { name: "John" };

let map = new Map();
map.set(john, "...");

john = null; // 覆盖引用

// john 被存储在了 map 中，
// 我们可以使用 map.keys() 来获取它
```
WeakMap 在这方面有着根本上的不同。它不会阻止垃圾回收机制对作为键的对象（key object）的回收。

让我们通过例子来看看这指的到底是什么。

`WeakMap` 和 `Map` 的第一个不同点就是，`WeakMap` 的键必须是对象，不能是原始值：


```
let john = { name: "John" };

let weakMap = new WeakMap();
weakMap.set(john, "...");

john = null; // 覆盖引用

// john 被从内存中删除了

```
##### new 的实现
```
function _new (...s){
  let obj=Object.create({})
  let fun = s.shift()
  let res = fun.apply(obj,s)
  obj=__proto__ = fun.prototype
  if(Object.prototype.toString.call(res)=="[object Object]"){
      return res
  }
return obj
}
```
### 浏览器解析url的过程 
* 1 用户输入url ：scheme/host.doman:popt/path/files .scheme：协议。如http，host.主机名称www  .doman url port 默认80端口 path 服务器地址。files服务器地址下的文件
* 2 通过url解析出主机名称
* 3 通过主机名获得ip地址，这个过程如果本地缓存列表有，则会的在本地获取，如果没有则需要在dns上获取到ip地址，然后缓存(在dns本地缓存的顺序是：一.浏览器缓存 二 操作系统缓存 三 路由缓存 四 ips 和dns 服务器缓存 五 根服务器缓存，遵循 递归查找)
* 4 浏览器解析出 端口号
* 5 客户端与服务端建立tcp 的三次握手
 第一次握手：建立连接时，客户端发送 SYN 包（seq=x）到服务器，并进入 SYN_SENT 状态， 等待服务器确认。（SYN：同步序列编号，Synchronize Sequence Numbers）；
第二次握手：服务器收到 SYN 包，必须确认客户的 SYN（ack=x+1），同时自己也发送一个 SYN 包（seq=y），即 SYN + ACK 包，此时服务器进入 SYN_RECEVED 状态；
第三次握手：客户端收到服务器的 SYN + ACK 包，向服务器发送确认包 ACK（ack=y+1），此包发送完毕，客户端和服务器端进入 ESTABLISHED（TCP 连接成功）状态，完成三次握手。
![三次握手.png](https://upload-images.jianshu.io/upload_images/25862823-6826be9670c91af7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
CP 有6中标志位：
   - SYN（synchronous 建立连接）
  -  ACK（acknowledgement 确认）
   - PSH（push 传送）
   - FIN（finish 结束）
   - RST（reset 重置）
   - URG（urgent 紧急）
另外数据包：
Sequence number（顺序号码）
Acknowledge number（确认号码）
第一次握手中 SYN = 1，代表客户端 Client 要建立连接，同时随机产生 seq（Sequence number）= x 的数据包，然后传输到 Server；
Server 收到 Client 的信息，通过 SYN = 1 知道是要建立连接，然后向 Client 发送信息，ACK = 1 代表确认，ack（Acknowledge number）= x + 1，SYN = 1 代表建立连接，随机产生 seq = y 的数据包；
Clien 收到 Server 的信息，通过 SYN = 1 知道是要建立连接，然后检查 ack 是否正确，即第一次发送的 seq + 1，同时检查 ACK 是否为 1，若正确，Client 再发送 ack = y + 1，ACK = 1 到 Server，Server 收到确认 seq 值和 ACK = 1 后则连接建立成功。

* 6 客户端发出请求报文，请求报文包括：一 请行：http:liuliukun.com，二 请求头:get、post请求方式、cash、cookie、path、origin 请求报文：请求加入的接口数据
* 7 服务端返回报文，一 包括相应码，报文头：请求的跨越、时间、content-type，报文主题：返回请求后的数据
```
1.. 代表是 
    准备请求http
2..请求成功的状态
    200 请求成功 
    204 请求成功并放回空 
    206 请求大文件时候，如下载功能，部分文件成功
3.. 重定向 
    301 永久重定向 
    302 临时重定向，后续还是访问老的地址 304：访问文件未作修改，客户端在缓存中获取
4.. 客户端出错 
    400：语法出错、
    401表示http协议有问题  
    403表示：服务器拒绝
    404：未找到资源
5.. 服务器
    500 ：服务器运行出错
    503：服务器碟机
```
* 8 获取文档并解析，
* 9 如果文档中有其他请求则，在执行678
结束时，执行四次挥手，1 浏览器说，我的报文发送完成了，准备关闭了 2服务器发发请求，表示，这边消息，已经接受完毕，3 服务发送消息，表示，消息已经发送完成 4 浏览器告诉服务端。我消息已经接受完准备关闭
##### 浏览器解析
```
html-->html parse -->domtree            layout
                        |               |
(html 和 css 的一个attch过程)attach-->padding tree--paining
                        |
css-->css parse-->css rule
```
###### css tree
```
    css rules


selectors   declaration
 div       margin    3px
```
###### 计算的权重规则
```
 *             {}  /* a=0 b=0 c=0 d=0 -> specificity = 0,0,0,0 */
 li            {}  /* a=0 b=0 c=0 d=1 -> specificity = 0,0,0,1 */
 li:first-line {}  /* a=0 b=0 c=0 d=2 -> specificity = 0,0,0,2 */
 ul li         {}  /* a=0 b=0 c=0 d=2 -> specificity = 0,0,0,2 */
 ul ol+li      {}  /* a=0 b=0 c=0 d=3 -> specificity = 0,0,0,3 */
 h1 + *[rel=up]{}  /* a=0 b=0 c=1 d=1 -> specificity = 0,0,1,1 */
 ul ol li.red  {}  /* a=0 b=0 c=1 d=3 -> specificity = 0,0,1,3 */
 li.red.level  {}  /* a=0 b=0 c=2 d=1 -> specificity = 0,0,2,1 */
 #x34y         {}  /* a=0 b=1 c=0 d=0 -> specificity = 0,1,0,0 */
 style=""          /* a=1 b=0 c=0 d=0 -> specificity = 1,0,0,0 */
```
###### layout
负责计算页面的高度，位置信息的 --改变了css的样式，如文字大小等高度，位置，都需要重新布局（会影响其他元素，损耗巨大）
##### painting
绘制，从新改了页面的颜色，背景图（损耗小）
### cookie、session、sessionStorage和localSorage
###### cookie
* 1 是放在客户端的
* 体积较小4k,每个浏览器运行最多20个，
* 有失效时间，靠设置的。如果为负数这关闭页面失效。如果为正数则看看设置时间，如果为0则认为是清除 cookie
* cookie是会通过http 放在请求报文头部传给服务端的
* 生成的原因是，用于保持用户登录数据，让服务端接收到请求时，先判断是否sessionId 如果没有则生产sessionid 给到cookie,然后浏览器保持起来放在硬盘中
* cookie 是字符串
* cookie 不安全，容易被窃取
###### session
* session 是放在服务端的，用来处理用户身份的
* session 是一个对象，并且没有访问限制
###### sessionStorage
* 不支持跨域  当页面消失时,数据也会丢失，不同页面也会消失
* 不会携带http请求
* 最大支持 5m
* 有成熟的api 
###### localStorage
* 不支持跨域  当页面消失时,数据不会丢失，不同页面也会消失
* 不会携带http请求
* 最大支持 5m
* 有成熟的api 
* 可以在相同的域名，端口号下 信息共享，而session只能在相同的页面下
```
URL	                    说明	                  是否允许通信
http://www.a.com/a.js
http://www.a.com/b.js	同一域名下	允许
http://www.a.com/lab/a.js
http://www.a.com/script/b.js	同一域名下不同文件夹	允许
http://www.a.com:8000/a.js
http://www.a.com/b.js	同一域名，不同端口	不允许
http://www.a.com/a.js
https://www.a.com/b.js	同一域名，不同协议	不允许
http://www.a.com/a.js
http://70.32.92.74/b.js	域名和域名对应ip	不允许
http://www.a.com/a.js
http://script.a.com/b.js	主域相同，子域不同	不允许
http://www.a.com/a.js
http://file.a.com/b.js	同一域名，不同二级域名（同上）	不允许（cookie这种情况下也不允许访问）
http://www.cnblogs.com/a.js
http://www.a.com/b.js	不同域名	不允许
```
###### 防止cookie 窃取
* 设置httpOnly 为true,无法通过js脚本获取cookie
* 合理的时间内修改seeionid
* 使用token (服务端返回的加密字符)
* 校验头部信息User-Agent验 等
##### 如果用户关闭了cookie怎么办
通过url带cookieid 但很不安全，淘宝关闭了cookie 直接跳转到了登录页。
#### GC 算法
* GC是一种机制，垃圾回收器完成的具体工作
* 具体工作就是，查找垃圾，释放空间（obj=null），回收空间
* 算法是，工作时，时时查找和回收所遵循的原则
具体算法有
* 引用计数
* 标记清除
* 标记整理
* 分代回收

###### 引用计数
* 设置引用数，判断当前引用数是否为0
* 引用计数器
* 引用关系修改时，修改引用数
```
let obj1 ={
    name:1
}
let obj2 ={
    name:2
}
let obj3 ={
    name:3
}
let arrlist = [obj1.name,obj2.name,obj3.name]
function fn(){
    let name1=1
    let name2 =2 
}
fn()
// 其中 name1 和name2 在 fn执行后，引用为o被清楚
// 而obj被arrlist 引用1不会被清楚
```
优点
* 发现垃圾立即处理
* 最大程度减少程序暂停 // 应为是时时处理

缺点
* 无法清楚相互引用的状态 
* 时间开销大，因为如果涉及大量数据删除时候
##### 标记清除
```
let a:{b:c{d}} //标记
function (){
    name.next =name2 // 没有被标记
    name2.pre = name1 gf 
} 
```
* 核心思想：分标记和清除两个阶段
* 遍历所有可达对象 a.b.c.d 标记
* 清楚所有没有标记的列表（并且清除标记的部分，把所有回收空间直接放在空闲列表上）

优点

可以解决上面函数内部相互引用的 引用计数无法清楚的问题

缺点

会照成空间碎片的问题
```
         global
          |
          V
a头域1域2  b头 域1   c头域1
  |                   |
  |—————空闲列表----|


```
只有 b被标记，ac会被清除，但是由于清除的a,有两个字节
c有一个字节。而且abc是连续地址，假设空闲列表只有这么多了，在生成1.5个字节时，会照成一个多，一个不够用
##### 标记整理
于标记清除唯一不同的是，在标记清除的空间回收前，会做一个整理阶段，会把清除，和使用的内存空间整理开来，防止出现空间碎片
### V8
什么是v8
* v8是javascript 主流的执行引擎
* v8 采用及时编译
* v8 有内存上线，64g电脑 1.5g，32g电脑800m,为了配合垃圾回收机制，过大的内存，清理垃圾更加耗时，影响浏览器速度
#### 分代回收
v 8内存分配
```
新生代                老生代

from   to            老生代存档区
```
###### 新生代
* 小空间存储新生代（32|16）
* 新生代是存储时间交端的内存，如在函数里面的变量

新生代处理逻辑
* 使用标记整理和复制算法
* 新生代将内存分配为2个大小相等的空间，
* 使用空间为from,限制空间为to
* 在使用的时候，把内存分到 from
* 当使用的空间达到一定程度时，使用标记整理，使得内存空间连续化
* 整理完成后，讲form 拷贝到to
* 在把from的内容清除掉

###### 内存晋升
在拷贝的时候发现，当前清除的空间发现，在老生代也存在的时候

场景
* 当经历了一轮gc 操作发现空间还在的时候
* 如果to 使用到了 25%的时候，将内存分到老生代里面为了 下次to空间的存储

###### 老生代
* 在内存中存活较长的数据，比如全局作用下的数据，和闭包数据
* 老生代内存大小为1.4g/700m
原理实现
* 主要采用，标记清楚，标记整理，增量标记算法
* 当程序运行的时候是标签清楚
* 当新生代的晋升数据时，并且老生代空间不足的情况
* 执行的过程会出现切片情况，停止js运行，分段的去垃圾清除
###### 新老对比
* 由于新生代是用空间换时间，基于内存小，老生代一份为2 比较浪费， 不适合复制算法
* 老生代内容大，采用分量标记，处理
##### 内存问题的外在表现
* 加载延迟
* 操作不顺，内存过大导致 设备开启更多大的空间
* 性能伴随时间越长，越差

##### 监控内存的方式
存储问题的总监
* 内存泄露 --> 内存持续走高没有下降
* 内存膨胀  --> 设备为了性能开辟了更大的空间
* 平凡的垃圾回收 --->内存变化图
##### 快照查找分离dom
分离dom 占用了空间
```
   let tempDom 
    function  fn () {
        let ul = document.createElement("ul")
        for(let i=0 ;i<10;i++){
            let li = document.createElement("li")
            ul.appendChild(li)
        }
        tempDom = ul
    }
    let add = document.getElementById("add")
    add.addEventListener("click",fn)
```
在 chrom 浏览器，中的memery中查找deta，会发现有分离dom，虽然没有使用，但没js引用
#### 优化方式
1 慎用全局变量
* 会导致垃圾回收到结束时才使用
* 是作用域的顶端，查找使用时，
* 会照成污染

2 全局缓存系统的方法
```
// 例子1 
 document.getElementById("ll")
  document.getElementById("ll")
   document.getElementById("ll") document.getElementById("ll")
   
例子2 
let doc =  document
    doc.getElementById("ll")
   doc.getElementById("ll") doc.getElementById("ll")
```
在jsperf 上性能会提升很多

3通过原型链的方法替代构造函数本身的方法
```
function Person(){
    this.mame = funciton (){}
}
unction Person1(){
}
Person1.prototype.name=funciton (){}
```
4 解决闭包的内存泄漏
```

funciton fn (){
    let m  = document.getElementByid("lk")
    m.onclick=funciotn(){
        console.log(m.id) //调用了外部的值形成了闭包
    }
    m = null
}
fn()
```
5 避免访问方法的使用
```
function Person(){
    this.name  = "lk"
    this.getName=function (){
        return this.name
    }
}
let person = new Person()
// person.name 的方式速度优于 person.getName()
```
6 最快的循环方式
```
forEach >for>for in
```
7 减少作用域的查找
```
let name = 1
funtion f1(){
 
    return mame
    
}
funtion f2(){
 let name =2 
    return mame
    
}
// f2 会减少内存查找，但会开辟跟多的空间
```
8 数据的创建方式
* 引用类型的时候，字面量和new的速度差不多
* 基本数据类型的时候，字面量快，因为new的是一个对象，而a 是个字符串，但a还是调用字符串的方法，是因为隐式转换的
```
let a = "111"
let b = new String("1111")
```
9 减少在for循环里面的数据操作，while 会快于for
#### 正则
```
[1234] 表示其中一个
(1234) 表示一个整体
 -  除了 
 + 1 到任何个
 d 数字
 D 除了数字
 s 空格换行符
 S 除了空格换行符
 w 字母数字下划线
 W  除了字母数字下划线
 . 除了换行符
```
### http问题 
###### http 缓存
* 强缓存
* 协商缓存
```
                    开始
                     |
                     V                     符合时间内，和hash相同
    |----------->---浏览器——————304————————————————|
    |                 |                         服务器请求
    |                 V                     ---------|-------           
    ^             发起请求                  |               |
    |                 |         请求头：(hash)if-no-match   if-modify-since(时间)
    |                 V                     |               |
     ------<-----是否有缓存(本地)---->返回头Etags---no--头last-modify
                    
```
###### 强缓存
 不需要去服务器获取数据，从本地获取，serviceWorker 、memery cashe、dask cache。
 这些缓存得设置由 Expires、 cache control、pragma三个决定pragma的级别最高 
 
 
###### expires 
 expires 设置时间，如果系统时间大于了 expires的时间，缓存失效，由于时间放在前端，导致了时间的不准确，这个优先级最低
###### cache-control 
* max-age 超过时间缓存失效
* no-cache 没有强缓存，会遵循协商缓存
* no-store 没有缓存包括协商缓存
* private 专用于前端缓存，不包括中间cdn，中间代理
* public 缓存过程中使用，后期必须服务区校验
###### pragma
唯一一个 no-cache 优先级最高
###### 协商缓存
用于和服务器校验的
* Etag if-no-match,用于判断文件的hash 值是否改变，如果没改变。返回304
* last-modify if-modify-since 用于判断修改时间，服务器会根据最后一次修改时间左判断，如果没有修改返回304
 
#### http1.0和http1.x的区别
* http默认打开了keep-alive
* 增加了 Etags等If-Unmodified-Since, If-Match, If-None-Match控制缓存的属性
* 支持了断点传出，返回206
* 增加了host 头处理--应对一个服务器对接多个虚拟机的情况
#### http2.0和http1.x的区别
* 原来的tcp 传输只能支持6-8个。单个tcp 会出现堵塞，二http 可以实现多路复用，原因是，http2 使用了二进制分流的概念，而http使用的报文传输，无法区分不同的流
* 对header 经行了压缩
* 服务器可发起推送
#### http3.0与http2.0
解决单个 tcp 堵塞问题，使用udp+加密+控流+丢包重新穿的技术
#### https 和http 的差别
```
    http应用层              https应用层
        |                       |
        V                       V
     Tcp传输层             ssl/tls 安全层
        |                       |     
        V                       V
     ip网咯层                 tcp-->ip
       
```
* 客户端向服务器请求 ssl连接
* 服务器返回安全证书，带有公钥
* 客户端随机产生密钥，在用公钥来加密，发送给服务端
* 服务端使用自己的私钥去解密
* 最后客服端和服务端使用公钥去建立了连接
##### get 和 post的区别
* get 会默认把请求的数据缓存下来，post 不会
* get 会对把请求的参数放在url中，post放在header 中
* get 会把headr 和body 内容一次性的tcp ，post会两次
##### tcp 和udp 
tcp 有序可靠，udp 会出现丢包情况 

##### 总结缓存
本地缓存有
* serviceWorder
```
//注册
navigator.serviceWorker.registor("ws.js",{scope:"/"}).then

//ws.js
//install的时候存储所需文件
self.addEventLister("install",()=>{
    waitUntil(()=>{
        cache.open("key").then(()=>{
            cache.addAll('')
        })
    })
})
// actrated 销毁缓存
self.addEventLister("actrated",()=>{
    watiUnitl(()=>{
        cache.delect
    })
})
// fetch时给到cche
self.addEventLister("fetch",()=>{
    cache.match()
})
```
* memery cache 如get 的一些img ,cdn 
* dick cache 存储了 url 的ip地址，cookie。
* Etag/if-no-match 、Last modify/if-modify-since
##### 跨域
* Access-control-Allow-origin * 
* Access-control-Allow-Gredentials = true 携带cookies-服务器
* Access-control-export-headers 该字段可选。CORS请求时，XMLHttpRequest对象的getResponseHeader()方法只能拿到6个基本字段：Cache-Control、Content-Language、Content-Type、Expires、Last-Modified、Pragma。如果想拿到其他字段，就必须在Access-Control-Expose-Headers里面指定。上面的例子指定 
* withCredentials 客户端发送cookie的开关
* document.domain + iframe跨域  location.hash + iframe
### css部分
##### 伪类和伪元素
* 伪类 1状态： hover active visit focus 2 结构话：nth-child first child 3表单：checked default disable readOnly
* 伪元素 before after selection
##### rem 和 em
rem 是根据根元素的计算，em 是根据当前父元素计算 下面是利用rem 的移动端布局
```
var dpr = 1
if (isIPhone) {
                  // iOS下，对于2和3的屏，用2倍的方案，其余的用1倍方案
                  if (devicePixelRatio >= 3 && (!dpr || dpr >= 3)) {
                      dpr = 3;
                  } else if (devicePixelRatio >= 2 && (!dpr || dpr >= 2)){
                      dpr = 2;
                  } else {
                      dpr = 1;
                  }
              }else if(isAndroid && devicePixelRatio > 2){
                  dpr = 3;
              }else {
                  // 其他设备下，仍旧使用1倍的方案
                dpr = 1;
              }
              var width = documentElement.getBoundingClientRect().width;
              if (width / dpr > 540) {
                  width = 540 * dpr;
              }
              var rem = width / 10;
              docEl.style.fontSize = parseInt(rem) + 'px';
```
利用babel postcss-pxtorem 的把当前文件px 转换成 rem
##### flex
flex 是 disable position float 的语法糖
flex 1 的含义
```
flex 1 === flex.grow===1 flex.shrink===1 flex.basis ==auto
```
* grow 是判断当前子元素的扩展属性，场景：只有当前子元素的宽度中和小于总宽度才会有效,默认值0，即使小于于总宽度，也不放大
* shrink 是判断当前子元素的总宽度是否大于总宽度，如果大于则缩小，默认值为1，默认缩小父盒子的宽度
* basis，在缩放前，判断父盒子的总宽度。默认auto
##### BFC(块格式化上下文)
 css 执行上下文，规范内部元素的渲染位置，BFC 之间不相互影响。

开启BFC : 
* body元素
* 设置disable inline-block flex
* 设置 float 除了设置为none 外
* 设置 overflow 除了visiable
#### XSS CFRS
* XSS 跨站脚本攻击，比如在input 输入一些带有script的标签，这些标签通过document.cookie获取cookie 信息。
 
解决办法：1 通过设置 httpOnly=true,无法从本地获取cookie 2 通过校验输入框中的内容来判断'<'闭合标签的，做转换。
* CSRF 跨站请求伪造：都过劫持当期登录的网站cookie <img src="facebook.com" style="visibility:hidden;"> 这个图片，如果你点击，就会把你的当前cookie 发给facebook，然后第三方网站会利用你的cookie 去请求 当前网站的接口伪造信息。

解决办法1 cookie 有个叫做SameSite 的属性，strict：最为严格，告诉的是 禁止第三方的cookie。lax:稍微宽泛，指的是大多数情况允许，只有get的导航地址时会带上。none:不管
#### websocket
* 长连接通讯，轮询访问
* 双向通信协议
* 需要先建立连接后在不断的发送数据
#### 柯里化
```
function add(){
    const _arg = [...arguments]
      function addFun(){
        _arg.push(...arguments)
        return addFun
    }
    addFun.toString=function(){
        return _arg.reduce((a,b)=>a+b)
    }
    return addFun
}
console.log(add(1)(2)(4,1))
```
利用高级函数做回调调用
#### compose
```
function compose(f,g){
    return (x){
        return(f(g(x))
    }
}
```
实现了数据流有右往左的过程
### 模块化
* commonJs 是同步加载模块，用于nodeJs ，因为node 文件加载都在本地进行
* AMD 是requireJs的推广的模块化思想，异步加载数据
* CMD 是sea.js推广过程的模块化思想
* UMD 是AMD he CommonJS 的合并产物
```
// AMD
define(key,[a.js,b.js],funcion()=>{
    console.log(111)
})
require(['foo', 'bar'], function(foo, bar) {
    foo.func();
    bar.func();
} );
// CMD
define(()=>{
    cont a = require("./a")
    cont a = require("./a")
})
// comonJs 
module.export
requre()
ES6
export
import
```
###### AMD 和CMD 的区别
* amd有全局的api 支持，而cmd 没有，只能通过 sea.use 
* amd 是提前执行,cmd 是延迟执行，但到后面amd 也执行延迟执行
###### commonJs 和 es6 module
* commonJs 是同步获取文件，适合服务端。
* commonJs 是讲字符串，对象，函数复制了一份给到另外一个变量，而es6module是一个解构的过程
* reruire 可以放在任何位置 而import 只能放在开头
##### for in  for of
* for in 适合遍历对象，也可以数组，但是遍历的是 key ,而 of 遍历的是value
* for of 本质是 迭代器，map set 实现了 迭代接口的，每次next 执行到下个值### oss cnd 的区别
* oss 是储存功能，只要存储静态资源
* cnd 是分发作用，把oss 的文件分发 并缓存到cdn 节点
#### js 函数思想
###函数

##### [](#函数是一等公民)函数是一等公民

*   函数可以存储在变量
*   函数可以作为入参
*   函数可做返回值

##### [](#纯函数)纯函数

每次输入输出的值都是一样的 例子：

```
let arr = [1,2,3,4,5]
console.log(arr.slice(1,3))
console.log(arr.slice(1,3))
console.log(arr.slice(1,3))
// 每次输出是一样的结果

console.log(arr.slice(1,3))
console.log(arr.slice(1,3))
console.log(arr.slice(1,3))
// 每次输出是的结果不一样

```

###### [](#纯函数的好处)纯函数的好处

1 缓存值，应为相同的输出。可以做缓存

```
function sum (a,b){
    console.log(1111,a,b)
    return a+b
}
function memoize(fn){
    let cache = {}
    return function(){
        let key = JSON.stringify(arguments)
        cache[key] = cache[key]? cache[key]:fn.apply(fn,[...arguments])
        return cache[key]

    }
}
let _m = memoize(sum)
console.log(_m(1,3))
console.log(_m(1,3))
console.log(_m(1,3))
console.log(_m(1,4))

```

2 可以做单元测试，由于纯函数的输出是可靠的 3 在多线程时，操作公共数据，会导致公共数据不可靠，但是纯函数不会，相同的输出会有相同的输出

##### [](#副作用)副作用

```
// 副作用使我的的函数变得不纯洁
let num = 8
function getVal(val){
    return val>num
}
// 相同的输入会得到相同的输出，但是由于nums的存在，导致了变数，以至于函数的不纯，所以称为副作用

```

副作用的来源

*   配置文件
*   数据库
*   用户输入
*   。。。。

##### [](#柯里化)柯里化

```
function sum (a,b,c){
    return a+b+c
}
function curry(fn){
    return function num1(...arg){
        if(fn.length>arg.length){
            return function(){
                let arr = [...arguments].concat(arg)
                //1 注意return 值 2 arr是数组传进 num1时候要换成单个形参 
                return num1(...arr)
            }
        }else{
            return fn(...arg)
        }
    }
}
let sumFn = curry(sum)
console.log(sumFn(1)(1,3))
console.log(sumFn(1,2)(3))
console.log(sumFn(1,2,3))

```

解决重复问题的，生成过程 假如 要计算 1+3+x的问题，那就let all=sumFn(1,2)，all(1),all(2),不用在关注1，3的问题 柯里化意义：

*   可以缓存函数的参数
*   可以让我传递较少的参数得到返回记忆的，新函数
*   让函数变得更加灵活，颗粒度更小
*   把多元函数换成一元函数，使其功能更强大

#### [](#compose-实现)compose 实现

```
// 洋葱代码
fes1(fes2(fes3(2)))

```

去除洋葱代码，利用纯函数，让数据从右往左传递

```
function reduce(arr,fn,acc){

    for(var i = 0;i<arr.length;i++){
        let val = arr[i]
        acc=fn(acc,val)
    }
return acc
}
function compose(...arg){
    return function(val){
        return reduce(arg.reverse(),(acc,fn)=>fn(acc),val)
    }
}
function getRevece(arr){
    return arr.reverse()
}
function getFirst(arr){
   return arr[0]
}
function toUpert(val){
    return val.toLocaleUpperCase()
 }
let getHF =  compose(toUpert,getFirst,getRevece)
console.log(getHF(["l","lk","love"]))
// let getHF =  compose(toUpert,getFirst,getRevece)
// let getHF =  //compose(toUpert,compose(getFirst,getRevece))
//let getHF =  compose(compose(toUpert,getFirst),getRevece)
// 任意组合顺序得到的结果都是相同的

```

#### [](#compose-函数调试)compose 函数调试

```
function log(val){
    console.log(val)
    return val
}
let getHF =  compose(toUpert,getFirst,log,getRevece)

```

### [](#point-free)point free

*   不需要关系数据
*   只需要组合函数
*   定义使用的基本函数

```
compose(toUpert,getFirst,log,getRevece)

```

### [](#函子)函子

```

// maybe 函子 如果当前函子的值为nuLl,导致下一步报错时候

class Maybe {
    constructor(val){
        this._val = val
    }
    static of(val){
       return new Maybe(val)
    }
    map(fn){
        console.log(this.isNoting(),"dd",this._val)
       return  this.isNoting()?new Maybe(null):new Maybe(fn(this._val))
    }
    isNoting(){
        return this._val===null||this._val===undefined
    }
}
console.log(Maybe.of(1).map(x=>undefined).map(x=>undefined).map(x=>undefined))

// maybe 函子虽然知道是解决了 null的不报错问题，但导致后续流程走不通 Either 函子
class  Left {
    static of(val){
        return new Left(val)
    }
    constructor (val){
        this._val = val
    }
    map(){
        return this
    }
}

class  Right {
    static of(val){
        return new Left(val)
    }
    constructor (val){
        this._val = val
    }
    map(fn){
        return new Right(fn(this._val))
    }
}
try {
    Right.of("1111")
} catch (error) {
    left.of("1111")
}
// io 函子，让副作用 到执行的时候去操作变量
class IO {
    static of(val){
        return new IO(function(){
            return val
        })
    }
    constructor(fn){
        this._val = fn
    }
    map(fn){
        return new IO(compose(fn,this._val))
    }
}
let res = IO.of(process).map(p=>p.execPath)
console.log(res._val())

```

###### [](#pointed-函子)pointed 函子

```
上面的函子使用了 of 方法，是的变量_val 包含于在一个盒子里，也是就是new后的一个变量，point 函子是一个概念

```

#### [](#monad)monad

```
// io 函子，让副作用 到执行的时候去操作变量
class IO {
    static of(val){
        return new IO(function(){
            return val
        })
    }
    constructor(fn){
        this._val = fn
    }
    map(fn){
        return new IO(compose(fn,this._val))
    }
}
let res = IO.of(process).map(p=>p.execPath)
// console.log(res._val())
// 
// console.log(fs,"fs")
function readFiles(str){
    return new IO(()=>{
        return fs.readFileSync(str,"utf-8")
    })
}
function log(x){
    return new IO(()=>{
        console.log(x,"xx")
        return x
    })
}
let r = compose(log,readFiles)("index.js")
// console.log(r._val()._val())
 // 链式调用太过繁琐
 class Monad {
     constructor(val){
        this._val = val
     }
     map(fn){
        return new Monad(compose(fn,this._val))
     }
     join(){
         return this._val()
     }
     firstMap(fn){
        return this.map(fn).join()
     }
 }
 function read2(sr){
        return new Monad(()=>{
           return fs.readFileSync(sr,"utf-8")
         })
 }
 function log2(x){
         return new Monad(()=>{
             console.log(x)
             return x
         })
 }
 // 等于永jion 方法执行了两次替代了_val
console.log(read2("index.js").firstMap(log2).join())

```

#### [](#es6)es6

###### [](#字符串标签)字符串标签

```
const name =1
sdf`sd${name}`

```

#### [](#proxy)proxy

```
const {log:log1} = console
const Objtext = {
    name:"lk",
    age:18,
    isMerry:false
}
let ObjectProxy = new Proxy(Objtext,{
    get(target,name,receiver){
        // console.log(...arg,"arge111")
        console.log(target,"target111")
        console.log(name,"name111")
        console.log(receiver,"who111")

    },
    set(target,name,val){
        log1(111111)
        target[name] =val
        log1(target,name,val)
    }
})
console.log(ObjectProxy.name="lp")
log1(Objtext,"origin val")

```

proxy 和 object.defineProtity的区别

*   proxy 还可以监听 数组，
*   proxy 不是侵入式的，不需要改写原型方法
*   proxy 更加强大可以监听数据删除，遍历等
#### [](#es2017)es2017

##### [](#objectentries)Object.entries

```
let obj2 = {name:"lk",age:18}
console.log(Object.entries(obj2))
console.log(new Map(Object.entries(obj2)))
// log:[ [ 'name', 'lk' ], [ 'age', 18 ] ]
//     Map { 'name' => 'lk', 'age' => 18 }

```

可以将对象的键值对完美的变成二维数组形式，而这种形式刚好是map想要的形式

##### [](#objectgetownpropertydescriptors)Object.getOwnPropertyDescriptors

```
let p1 = {
    firstName:"liu",
    lastname:"kun",
    get fulllName(){
        return this.firstName+"**"+this.lastname
    }
}
// let p2 = Object.assign({},p1)
// p2.firstName = "lp"
// console.log(p2,"p2")
/*
    Object.defineProperty(p1, 'property1', {
        value: 42,
        writable: false
        });

    Object.defineProperties(obj, {  
        'property1': {    value: true,    writable: true  },  
        'property2': {    value: 'Hello',    writable: false  }
    });
    // 只是改不对象本身，和一些getter 和setter 方法，不是修改property的属性，不牵扯原型
*/ 
let getOwnPropertyDescriptors =  Object.getOwnPropertyDescriptors(p1)
 let p2 = Object.defineProperties(p1,getOwnPropertyDescriptors)
console.log(p2,"p2")
p2.firstName = "li"
console.log(p2.fulllName)

```
##### 浏览器解析
```
html-->html parse -->domtree            layout
                        |               |
(html 和 css 的一个attch过程)attach-->padding tree--paining
                        |
css-->css parse-->css rule
```
###### css tree
```
    css rules


selectors   declaration
 div       margin    3px
```
###### 计算的权重规则
```
 *             {}  /* a=0 b=0 c=0 d=0 -> specificity = 0,0,0,0 */
 li            {}  /* a=0 b=0 c=0 d=1 -> specificity = 0,0,0,1 */
 li:first-line {}  /* a=0 b=0 c=0 d=2 -> specificity = 0,0,0,2 */
 ul li         {}  /* a=0 b=0 c=0 d=2 -> specificity = 0,0,0,2 */
 ul ol+li      {}  /* a=0 b=0 c=0 d=3 -> specificity = 0,0,0,3 */
 h1 + *[rel=up]{}  /* a=0 b=0 c=1 d=1 -> specificity = 0,0,1,1 */
 ul ol li.red  {}  /* a=0 b=0 c=1 d=3 -> specificity = 0,0,1,3 */
 li.red.level  {}  /* a=0 b=0 c=2 d=1 -> specificity = 0,0,2,1 */
 #x34y         {}  /* a=0 b=1 c=0 d=0 -> specificity = 0,1,0,0 */
 style=""          /* a=1 b=0 c=0 d=0 -> specificity = 1,0,0,0 */
```
###### layout
负责计算页面的高度，位置信息的 --改变了css的样式，如文字大小等高度，位置，都需要重新布局（会影响其他元素，损耗巨大）
##### painting
绘制，从新改了页面的颜色，背景图（损耗小）
##### 总结缓存
本地缓存有
* serviceWorder
```
//注册
navigator.serviceWorker.registor("ws.js",{scope:"/"}).then

//ws.js
//install的时候存储所需文件
self.addEventLister("install",()=>{
    waitUntil(()=>{
        cache.open("key").then(()=>{
            cache.addAll('')
        })
    })
})
// actrated 销毁缓存
self.addEventLister("actrated",()=>{
    watiUnitl(()=>{
        cache.delect
    })
})
// fetch时给到cche
self.addEventLister("fetch",()=>{
    cache.match()
})
```
* memery cache 如get 的一些img ,cdn 
* dick cache 存储了 url 的ip地址，cookie。
* Etag/if-no-match 、Last modify/if-modify-since
#### websocket
* 长连接通讯，轮询访问
* 双向通信协议
* 需要先建立连接后在不断的发送数据
实现了数据流有右往左的过程
### 性能优化
*  雪碧图
*  代码压缩，从个人代码压缩：1使用table 代替div 、2精简代码、3 网页Gzip 压缩
```
// gizip 压缩，看是否浏览器支持，如果responseHeader 中有 content-Encoding="gizp"时
// node 使用expression 中间件
express.use(compression())
// webpack compressionWebpackPlugin
const CompressionWebpackPlugin = require('compression-webpack-plugin');
new CompressionWebbapckPlugin({
    asset:"[path].gz[qurey]"
    algorithm:"gzip",
    text:/\.(js|css)/,
    mixRatio:8 //压缩比最小为8时候压缩
    threShold:1024 // 大于1024时压缩
})
```
* 代码封装库，提高js 效率。
* 使用cdn 加速
```
// 动静分离
动态资源：多次访问会发生变化的
静态资源：多次访问不会发生变化的，如 html,js css


把静态资源放在 nginx 代理服务器上
动态资源放在 tomcat 上

// cdn 回溯
是 cdn 跳过了 缓存（本地，nginx） 访问的原始服务器上了
```
* 使用json作为前后端 交换
* 控制cookie大小，
### ssr 
服务端 使用react 的 
* renderToString 将react组件转换为字符串随浏览器返回
* 如果遇到 o'nclick 事件，客户端需要独立打包重新注释服务端
* 路由控制 服务端使用react-router-dom/StaticRouter 来实现路由跳转
```
    app.get('*',(req,res) => {
        const content=renderToString(
            <StaticRouter context={{}} location={req.path}>
                {routes}
            </StaticRouter>
        );
        res.send(`
        <!DOCTYPE html>
        <html lang="en">
            <head>
                <meta charset="UTF-8">
                <meta name="viewport" content="width=device-width, initial-scale=1.0">
                <meta http-equiv="X-UA-Compatible" content="ie=edge">
                <title></title>
            </head>
            <body>
                <div id="root">${content}</div>
                <script src="/index.js"></script>
            </body>
        </html>
        `);
    });
    app.listen(9090);

```
### deepclone
```
    function deepClone(obj,hash=new WeakMap()){
    if(hash.has(obj))return obj.get(obj)
    const type = [Map,Date,Set,RegExp,WeakMap,WeakSet]
    if(type.includes(obj.constructor))return new obj.constructor(obj)
    const clone = Object.create(Object.getPrototypeOf(obj),Object.getOwnPropertyDescriptors(obj))
    hash.set(obj,clone)
    for(let k of Reflect.ownKeys(obj)){
        clone[k] = typeof obj[k]==="object"&&typeof obj[k]!=null?deepClone(obj[k]):obj[k]
    }
}
```
```

// 示例
// 第二个参数为null时，抛出TypeError
// const throwErr = Object.myCreate({a: 'aa'}, null)  // Uncaught TypeError
// 构建一个以
const obj1 = Object.myCreate({a: 'aa'})
console.log(obj1)  // {}, obj1的构造函数的原型对象是{a: 'aa'}
const obj2 = Object.myCreate({a: 'aa'}, {
  b: {
    value: 'bb',
    enumerable: true
  }
})
console.log(obj2)  // {b: 'bb'}, obj2的构造函数的原型对象是{a: 'aa'}
```
###  原型 
##### 1 构造函数
构造函数模式就是创建一个自定义的类，并且可以创造类的实例  
```
 function Person(name, age, gender) {
        this.name = name
        this.age = age
        this.gender = gender
        this.sayName = function () {
            alert(this.name);
        }
    }
    var per = new Person("孙悟空", 18, "男");
    function Dog(name, age, gender) {
        this.name = name
        this.age = age
        this.gender = gender
    }
    var dog = new Dog("旺财", 4, "雄")
    console.log(per);//当我们直接在页面中打印一个对象时，事件上是输出的对象的toString()方法的返回值
    console.log(dog);
```
上面的 new 方法每调用一次都生产新的name 属性和 sayName 方法，这样的工作是重复的，如何解决了？---prototype 
##### 2原型链
 对象之间纯在继承关系，可用通过prototype 访问父类的对象。直到Object 为止 
 ```
         Person -->--prototype---->Person.protype---->
           |  ^                            |  ^
           |  |--<--constructor--<---------|  |      |
           v                                  ^      |
           new                                |      |
           |                                  ^      |
           person------>----__proto__->------—|   __proto__
                                                     |
                                                     |
                                                     v
                                                     |
         Object()----->proto----------->Object.prototype
                                                    |
                                                __proto__
                                                   null
                                                   
                                                   
                                                   
  other:  __proto__ === Object.getprototypeof   b=Object.create(a),b.__proto__=a
```
![关系链.png](https://upload-images.jianshu.io/upload_images/25862823-e521b808459e22d7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 事件冒泡、事件捕获和事件委托

  
    // 事件监听，第三个参数是布尔值，默认false，false是事件冒泡，true是事件捕获
            document.getElementById('box3').addEventListener('click', sayBox3, false);
            document.getElementById('box2').addEventListener('click', sayBox2, false);
            document.getElementById('box1').addEventListener('click', sayBox1, false);

###### 1 先捕获后冒泡 直到document位置
   
        document-box3->box2->box1->box2->box3-doucment
   
###### 2阻止冒泡事件 
```
    if(window.event.stopPropagation){widnow.event.stopPropagation()}else{window.event.cancelBubble=true}
```
##### 3关于IE
```
 事件注册 window.attachEvent(name,fun)
 事件删除 window.detachEvent(name,fun)
 chorme:window.addEventListener window.removeEventListener
 获取事件源id chorme:e.target.id    ie:e.srcElement.id
 关于js的浏览器兼容问题，一般用能力检测来解决，if(){}else{}
```
### js事件循环eventLoop
#####  js 是单线程，但js纯在同步任务和异步任务，js处理同步和异步任务的机制叫做eventLoop
 * 同步任务 ：所有立即执行的任务都是同步任务如申明变量，函数执行(无依赖)
 * 异步任务 ：时间类的setTimeOut 、setInterval，ajax请求、promise等
 ##### 异步任务
 * 宏观任务（Macro Task）:时间类，ajax、dom 
 * 微观任务（Micro Task）promise
 ##### 执行顺序
 * 1 先执按顺序执行同步队列，遇到异步任务统一放到异步等待队列中，至到从上到下代码执行完成。
 * 2 执行异步队列，遇到宏观任务先按顺序放入等待队列中，直到所有宏观任务放入队列，在执行微观任务，如遇在微观任务中遇到到宏观任务，排到上面宏观队列的最后面
 * 执行所有的宏观任务
 ```
    console.log('开始111');

    setTimeout(function () {
    
      console.log('timeout111');
    
    });
    
    new Promise(resolve => {
    
      console.log('promise111');
    
      resolve();
    
      setTimeout(() => console.log('timeout222'));
    
    }).then(function () {
    
      console.log('promise222');setTimeout(() => console.log('timeout333'));
    
    })
    console.log('开始222');
 ```
结果
```
 开始111
 promise111
 开始222
 promise222
 undefined
 timeout111
 timeout222
```
```
console.log('开始111');

    setTimeout(function () {
    
      console.log('timeout111');
    
    });
    
    new Promise(resolve => {
    
      console.log('promise111');
    
      resolve();
    
      setTimeout(() => console.log('timeout222'));
    
    }).then(function () {
    
      console.log('promise222');setTimeout((function(){console.log("time33333")})());
    
    })
    console.log('开始222');
```
结果  
```
    开始111
    promise111
    开始222
    promise222
    time33333
    undefined
    timeout111
    ()()会立即执行跳出setTimteOut
```
### js中call、apply、bind方法的作用和区别
```
    function a (){console.log(this)}
    a() // window
    a.cell({},111) // {}
    a.apply({},[1]) //{}
    a.apply({},[1]) // function
    a.apply({},[1])() // {}
```
### 闭包
```
var fun1 = addCount();
fun1(); //1
fun1(); //2
var fun2 = addCount();
fun2(); //1
fun2(); //2
```
##### 特殊的for 循环
```
    for (var i = 0; i < 4; i++) {
  setTimeout(function() {
    console.log(i);
  }, 300);
}
```
结果： 4个4 ，等同步任务完成后拿到的i 为4 

* 解决方法1
```
    //方法一:
for (var i = 0; i < 4; i++) {
  setTimeout(
    (function(i) {
      return function() {
        console.log(i);
      };
    })(i),
    300
  );
}
// 或者
for (var i = 0; i < 4; i++) {
  setTimeout(
    (function() {
      var temp = i;
      return function() {
        console.log(temp);
      };
    })(),
    300
  );
}
//这个是通过自执行函数返回一个函数,然后在调用返回的函数去获取自执行函数内部的变量,此为闭包

//方法发二:
for (var i = 0; i < 4; i++) {
  (function(i) {
    setTimeout(function() {
      console.log(i);
    }, 300);
  })(i);
}
```
* 解决方法2
```
 for(let i= 0;i<4;i++){
     setTimeout(()=>{console.log(i)},300)
 }
 //错位方式
 let i= 0;
  for(;i<4;i++){
     setTimeout(()=>{console.log(i)},300)
 }
```
这种原因是由于 let 和 var,let 有作用域，每次在for 循环的时候都会i=3生成新的作用域，所以导致了，在var 的时候最后一次覆盖了前面，而let作用域的存在每次都是在自己的作用域下生成了新的 i,i =0 ...i=3

### Promise
##### 例子
```
new Promise((resolve)=>{ setTimeout(()=>{resolve()},300)})
    .then(res=>{
        return new Promise((resolve)=>setTimeout(()=>{resolve(222)},300))})
    .then((res)=>{
        console.log(res)}
    ).then(res=>{
        console.log("上个为null")
        
    }).then((res)=>{
    throw new Eror()}).
    then(null,(res)=>{
        console.log("error1")
    })
    .then((res)=>{
        console.log(res+"eror hou") ;
        return 111
    }).then(res=>console.log(res+"last"))
```
返回
```
    222
    上个为null
    error1
    undefinederor hou
    1 111last
```
##### Promise.all 和Promise.race
```
//切菜
    function cutUp(){
        console.log('开始切菜。');
        var p = new Promise(function(resolve, reject){        //做一些异步操作
            setTimeout(function(){
                console.log('切菜完毕！');
                resolve('切好的菜');
            }, 1000);
        });
        return p;
    }

    //烧水
    function boil(){
        console.log('开始烧水。');
        var p = new Promise(function(resolve, reject){        //做一些异步操作
            setTimeout(function(){
                console.log('烧水完毕！');
                resolve('烧好的水');
            }, 1000);
        });
        return p;
    }

    Promise.all([cutUp(), boil()])
        .then((result) => {
            console.log('准备工作完毕');
            console.log(result);
        })
```
返回结果： 
```
    开始切菜。
    开始烧水。
    切菜完毕！
    烧水完毕！
    准备工作完毕
    ['切好的菜','烧好的水']
```
```
    let p1 = new Promise(resolve => {
        setTimeout(() => {
            resolve('I\`m p1 ')
        }, 1000)
    });
    let p2 = new Promise(resolve => {
        setTimeout(() => {
            resolve('I\`m p2 ')
        }, 2000)
    });
    Promise.race([p1, p2])
        .then(value => {
            console.log(value)
        })
```
返回结果：
```
    I`m p1
```
* then 返回的如果是promise 后面的.then 等待状态完成后在执行，如果返回普通值这不用等待，可直接在then中获取。不返回也可调用后面.then()
* Promise.all() 和Promise.race()都是批量处理异步。但是all是有完成才能成功，有一个失败则返回第一个失败。Promise.race()成功只返回第一个成功。
### 迭代器
```
let a = [1,2,3,4]
for(var i =0;i<a.length;i++){
    console.log(a[i])
}
```
迭代器要解决的问题就是，像上面在循环的时候，不考虑标记位i的数据，只关注遍历数据的value值
##### 手写promise
```

class MyPromise {
    static PENDING = "pending"
    static FULFILLED = "fulfilled"
    static REJECTED = "rejected"
    constructor(executer){
        this.state = MyPromise.PENDING
        this.value = null
        this.callbacks = []
        // 使用try catch 保证函数体内得报错
        try {
        // bind this 保证this 永远指向MyPromise
            executer(this.resolve.bind(this),this.reject.bind(this))
        } catch (err) {
            this.reject(err)
        }
    }
    resolve(value){
        // 只有在 peding 状态时才让改变 state。防止状态倒流
        if(this.state===MyPromise.PENDING){
            this.state = MyPromise.FULFILLED;
            this.value = value
           setTimeout(()=>{
            this.callbacks.forEach(({onFulfilled})=>{
                try {
                    onFulfilled(this.value)
                } catch (error) {
                    this.reject(error)
                }
            })
           })
        }
    }
    reject(reason){
        if(this.state===MyPromise.PENDING){
            this.state = MyPromise.REJECTED
            this.value  = reason
            setTimeout(()=>{
                this.callbacks.forEach(({onReject})=>{
                    onReject(this.value)
                   
                })
            })
        }
    }
    /*
        let p = new MyPromise((resolve,reject)=>{
            setTimeout(()=>{
                resolve("解决")
            },1000)
            // reject("失败")
        })
        像这种异步调用，因为resolve 被滞后调用，但是then 方法是同步执行得，所有虽然 reslove被1秒后调用成功，
        但是then 已经提前调用，并且没有获取到状态和值
    */ 
    then(onFulfilled,onReject){
        // onFulfilled,onReject
        //  他俩存在是为了和value 产生一个  result ,这个result 为了后续的 新 的 
        // pmromise resolve,和reject 传值并改变状态 
        // onFulfilled 执行是为了完成当前的第一个then 的函数体的值
        if(typeof onFulfilled!="function"){
            // then() 解决这种空得then时候得穿透问题
            onFulfilled = ()=> {this.value;this.state=MyPromise.FULFILLED}
        }
        if(typeof onReject!="function"){
            onReject = ()=>{this.value;this.state=MyPromise.REJECTED}
        }
        // 返回一个新的 promise 重复上面操作
      let promise = new MyPromise ((resolve,reject)=>{
                if(this.state===MyPromise.PENDING){
                    this.callbacks.push({
                        onFulfilled:(valve)=>{
                           this.parse(promise,onFulfilled(valve),resolve,reject)
                        },
                        onReject:(valve)=>{
                            this.parse(promise,onFulfilled(valve),resolve,reject)
                        }
                    })
                }
                if(this.state===MyPromise.FULFILLED){
                    // 保证到异步队列
                    setTimeout(()=>{
                        this.parse(promise,onFulfilled(this.value),resolve,reject)
                    })
                }
                if(this.state===MyPromise.REJECTED){
                    setTimeout(()=>{
                        this.parse(promise,onReject(this.value),resolve,reject,true)
                    })
                   
                }
       })
       return promise
    }
    parse(promise,result,reject,resolve,mark){
        // 不允许返回自己
        if(promise===result){
            throw new TypeError("Chaning cycle detected for promise")
        }
        try {
            if(result instanceof MyPromise ){
                result.then(resolve,reject)
            }else{  
               if(mark){
                   reject(result)
               }else resolve(result)
            }
        } catch (error) {
            reject(error)
        }
    }
    static resolve (value){
        return new MyPromise((resolve,reject)=>{
            if(value instanceof MyPromise){
                value.then(resolve,reject)
            }else{
                resolve(value)
            }
        })
    }
    static reject (value){
        return new MyPromise((resolve,reject)=>{
            if(value instanceof MyPromise){
                value.then(resolve,reject)
            }else{
                reject(value)
            }
        })
    }
    static all(promises){
        const value = []
        return new MyPromise((resolve,reject)=>{
            promises.forEach((promise)=>{
                promise.then(res=>{
                    value.push(res)
                    if(value.length===promises.length){
                        resolve(value)
                    }
                },(reson)=>{
                    reject(reson)
                })
            })
            
        })
    }
    static race(promises){
       return new MyPromise((resove,reject)=>{
        promises.forEach(promise=>{
            promise.then(value=>{
                resove(value,1111)
                },reason=>{
                    reject(reason)
                })
            })
        })
    }
}
```
##### 自定义迭代器
```
function iterator(arr){ 
    var index=0 ;
    return {
        next(){     
        if(index>=arr.length){         
            return{done:true,value:undefined }
            
        }; 
        return {done:false,value:arr[index++]}  
            
        }
    }
    
}
```
##### generator 生成器
```
function *genarator (arr){ 
    for(var i=0 ;i<arr.length;i++){ 
        yield arr[i]
    } 
    
}
const gen = genarator([1,2,3,4])
gen.next()
```
* yield 可中断函数执行，到下个.next（）时候才能继续执行下面函数体
* yield 不可在*函数外调用
##### 可迭代对象
* 数组，set,map
*  Symbol.iterator 函数可以返回一个迭代器
```
a = [1,2,4][Symbol.iterator]
a.next()

qu  = new Set([1,2,3])
mm = qu.[Symbol.iterator]()
mm.next

uu = new Map()
uu.set(1,1)
uu.set(2,2)
yy = uu[Symbol.iterator]()
yy.next()
{
    done: false
    value: [1, 1]
}
```
for of 可以替代迭代器
```
a = [1,2,3,4]
for(var i of a){console.log(i)}
1 2 3 4
```
for of 为何可以遍历数据，遍历数据具备的属性是什么？
```
 let col = {
            list:[],
            *[Symbol.iterator](){
                for(var i =0;i<this.list.length;i++){     yield this.list[i] 
                    
                }
            }
 }
 col.list.push(1)
 col.list.push(2)
 col.list.push(3)
 col.list.push(4)
 for(let item of col){console.log(item)}

```

是因为对象内部有一个Symbol.iterator属性，所生成的迭代器。
##### 内建迭代器
* entries() 返回[key,value ] set数据key和value值相同
* values() 只返回 value的值
* keys() 只返回 key的值
```
    var arr1 = [1,2,3,4]
    var set1 = new Set([1,2,3,4,5])
    var map1 = new Map()
    map1.set(11,11)
    map1.set(22,22)
    for (let i of arr1.entries()){console.log(i)}
    // [0,1]  [1,2]  [2,3]  [3,4]  [4,5]
    arr1.entries().next()
    // {value:[0,1],done:fase}
    for (let m of set1.entries()){console.log(m)}
    // [1,1] [2,2] [3,3] [4,4] [5,5]
    set1.entries().next()
    // {value:[1,1],done:false}
    for(let q of map1.entries()){console.log(q)}
    // [11,11] [22,22]
    map1.entries().next()
    // {value:[11,11],done:false}
    // value() 和keys() 同理，都是可以返回可迭代的数据
```
 可以看出 entries 和values keys 都是可以使用.next与Symbol.iterator有相同功能
 ##### 高级迭代器
 ```
 function *createIterator() {
    let first = yield 1;
    let second = yield first + 2; // 4 + 2
    yield first + 3; // 5 + 3
} 
next() next(2) next()
// next 结果{value: 1, done: false} {value: 4, done: false} {value: 5, done: false}
 ```
 第一次什么都没传值，则为1 。第二次传了2.yield+2 为4，此时first 被赋值为2。所以在第三次的时候会返回5，如果第二次不穿2的话，无法获取到后面的值。所以在函数体内无法获取前面的first 值，只能通过next 后的返回值做为下个next 的入参获取值
 ##### 迭代器抛错
 ```
 function *createIterator() {
    let first = yield 1;
    let second;
    try {
        second = yield first + 2; // yield 4 + 2 ，然后抛出错误
    } catch (ex) {
        second = 6; // 当出错时，给变量另外赋值
    }
    yield second + 3;
}
let iterator = createIterator();
console.log(iterator.next()); // "{ value: 1, done: false }"
console.log(iterator.next(4)); // "{ value: 6, done: false }"
console.log(iterator.throw(new Error("Boom"))); // "{ value: 9, done: false }"
console.log(iterator.next()); // "{ value: undefined, done: true }"

 ```
 注意这个错误处理得放上一次
 ##### return 可阻断generator
 ```
    function *cf(){ yield 1 ;return 2 ; yield 2}
    mm = cf()
    mm.next()
    mm.next()
    mm.next()
    //  {value: 1, done: false} {value: 2, done: true} {value: undefined, done: true}
 ```
 
##### 异步函数回调，并获取其值
```
const async11 = (data)=>{
    return (cb)=>{
        setTimeout(()=>{
            cb&&cb(11111)
        },2000)
    }
}
const async22 = (data)=>{
    return(cb)=>{
        setTimeout(()=>{
            // cb&&cb("async22")
            // console.log("chuanzhi")
            // 下一次想做什么 ？1 ff.next(data) ,按照这个步骤在走一次
            console.log(data)
           cb&&cb(data +"--2222")
    },2000)
    }
}
const async33=(data)=>{
 return (cb)=>{
     cb&&cb(data)
     console.log(data)
    setTimeout(()=>{
        cb&&cb()
    },2000)
 }
}
function *generator(){
    let data1 = yield async11()
    let data2 = yield async22(data1)
        yield async33(data2)

}
let i = 0
function run (generantor){
    const gen = generantor()
    let result = gen.next()
    // const { value,done } = result
    console.log(result,"result")
    function  step() {
        i++
        if(!result.done){
            result.value((data)=>{  // 需要一个中间函数的原因是，除了要执行 gen.next(data)，执行回调完成后要在 step 后面继续调用函数
                result = gen.next(data)
                if(i>10){
                    console.log("死循环了")
                }else{
                    step()
                }
            })
        }
    }
    step()
}
run(generator);

// 11111   11111--2222
```
##### async await
```
var snyca = () => {
    return new Promise((res) => {
        setTimeout(() => {
            res(11111)
        }, 2000)
    })
};
var snyca2 = (data) => {
    setTimeout(() => {
        console.log("console.log(222)");
        console.log(data, "data");
        return data + "2222"
    }, 3000)
};
async function genr() {
    let data1 = await snyca();
    let data2 = await snyca2(data1)
};
genr()
```
* 如果返回的是promise，会自动执行 resolve函数，并返回resolve入参。
* 如果没有await的话，在synca2函数中得到的是一个promise对象
* 没有await的话，右边的函数体不会做等待，执行下面函数
### 继承 很多继承
* 原型链继承
```
function Person (){
    this.name = "kk"
};// 父类
function Women (){
    // 子类
};
Women.prototype = new Person();
Women.prototype.qq = 111;
Women.prototype.constructor = Women ; // 上面prototype 被污染了，赋值成整个Person的类，导致了Women.prototype.constructor 的对象指针指错
let son = new Women();
Person.prototype.mm = 'fff'
son.mm ==="fff"
```
优点，简单。父级添加元素，子级同步 

缺点：1 调用父类函数的时候不能传值。2不能批量给子类函数继承 3 new Person()时会导致产生实例
* 构造函数继承
```
function Person(){this.name="kk"}
Person.prototype.say = ()=>{console.log("say")}
function Women(){Person.call(this)}
var son = new Women()
son.name // kk
son.say //underfined

```
优点：能够实现多个类继承，缺点：由于没有new 只是执行了类函数，所以不能继承prototype的属性  

* 实例继承
```
function Person (){this.name="kk"}
Person.prototype.say = ()=>{console.log("say")}
function Women (){var instants = new Person() ; return instants}
var son  = Women()
son.name // kk
son.say() // say
```

```
function Person (){this.name="kk"}
Person.prototype.say = ()=>{console.log("say")}
function Person2 (){this.age = 13}
function Women(){var instans1 = new Person() ;var instans2 = new Person2();return{...instans1,...instans2}}
var son = new Women()
son.age // 13
son.say //underfined
```
结构会导致 prototype 丢失 

实例继承优点，可以继承父类原型，简单。缺点：无法实现多类继承
* 组合模式
```
function Person (){
    this.name = 'kk'
    
} ;
Person.prototype.say = ()=>{
    console.log("say")
    
};
function Person2 (){
    this.age = 13
    
};
function Women(){
    Person.call(this);
    Person2.call(this)
};
Women.prototype = {...new Person,...new Person2};
Women.prototype.constructor = Women;
var son = new Women()
son.name// kk
son.say // underfined
```
由于...多个原型prototyp导致原型链丢失
```
function Person() { 
    this.name = 'kk'
    };
 Person.prototype.say = () => { 
     console.log("say") 
     }; 
function Person2() {
     this.age = 13 
     }; 
function Women() { 
    Person.call(this); 
    Person2.call(this) 
    }; 
Women.prototype = new Person; 
Women.prototype.constructor = Women; 
var son = new Women()
console.log(son.name) // kk
console.log(son.say()) // say
```
优点：组件模式可以实现多个继承，（原型链自能继承一个，所有这种模式，必须把prototype属性放在一个类的原型链上）缺点：会有两次原型调用，两次Person() 执行，两次实例
* 寄生组合模式
```
function Person() { 
    this.name = 'kk'
    };
Person.prototype.say = () => { 
     console.log("say") 
}; 
function Women() { 
    Person.call(this)
}; 
function Super(){}
Super.prototype = Person.prototype
Women.prototype = new Super(); 
Women.prototype.constructor = Women; 
var son = new Women()
console.log(son.name)
console.log(son.say())
```
Super.prototype = Person.prototype ，使用了一个空的类作为中转，减少实例的次数
* es6 estends继承 
 ###### constructor
一个构造函数必须有一个 constrcutor 方法，在使用new时候会自动调用，如果没有会默认添加一个新的空的constructor
 ```
 class me{
    name =111
}
class ff extends me{
    // name = "fff"
    constructor(){
        // super()
        // this.name="fff"
    }
}
let qq =new ff()
console.log(qq.name) 
 ```
 然后就报错了：Uncaught ReferenceError: Must call super constructor in derived class before accessing 'this' or returning from derived  
 
 说明写了 constructor 必须要些super()，super 指的是父级的constrctor 和父级的类对象，所以想要在子类中使用this 必须先super，在继承过程中es6的模式和es5完全不同，es6的子类，是没有this的，是通过super 获取父级的this 然后在改造而来的
 ##### __proto__
 ```
 class A {

}
class B extends A{}
console.log(B.__proto__==A)
console.log(B.prototype.__proto__==A.prototype)
// true true
 ```
 因为：
 ```
 Object.setPrototypeOf(B,A)
Object.setPrototypeOf(B.prototype,A.prototype)
/*
    原理：setPrototypeOf(obj1,obj2){
        obj1.__proto__==obj2
        return obj1
    }
*/
 ```
 ###### 特俗的3中继承方式
 ```
 class A extends Object{}
console.log(A.__proto__===Object)
console.log(A.prototype.__proto__===Object.prototype)
class  B {}
console.log(B.__proto__==Function.prototype)
console.log(B.prototype.__proto__==Object.prototype)
class C extends null{}
console.log(C.__proto__===Function.prototype)
console.log(C.prototype.__proto__===undefined)
// 继承了null 之后导致了 C.prototype 的__proto__ 断裂，不在执行Object
 ```
 ###### super
 调用子集的时候super()，会指向当前子类
 ```
    class C {constructor(){this.name=1}}
    console.log(C.__proto__===Function.prototype)
    console.log(C.prototype.__proto__.__proto__)
 ```
 super还可以作为对象使用,作为调用父级的方法
 ```
     class A {
        name=1
        p(){
            console.log(this.name)
        }
    }
    class B extends A{
        constructor(){
            super()
            this.name = 'ffff'
        }
        say(){
            super.p()
        }
    }
    const b = new B()
    b.say() // ffff
    // 此时调用super的时候就等于 A.prototype.p.call(this)
 ```
 但是支能调用父级的prototype 的方法，父级的this方法无法使用super.获取
 ```
class A {
    constructor(){
        this.qqq="12345234"
    }
    p(){
        console.log(this.name)
    }
}
A.prototype.xx =111
class B extends A{
    say(){
        console.log(super.qqq)
        console.log(super.p)
        console.log(super.xx)
    }
}
const b = new B()
b.say() // undefined funtion 111

 ```
 super 的绑定场景
 ```
 class A {
    constructor(){
        this.qqq="12345234"
    }
}
class B extends A{
  constructor(){
      super()
       super.qqq =9999999
        console.log(this.qqq)
        console.log(super.qqq)
  }
}

new B() // 9999999 undefined  
// 原因，在调用new 的时候会把super绑定this,上，所有super.qqq =this.qqq
//。但在找super.qqq的时候，会去A 的 prototype上找
 ```
 修改原生构造函数
 ```
 function Myclass (){
    Array.call(this)
}
Myclass.prototype = Object.create(Array.prototype)
const arr1=new Myclass()
arr1.push(1)
console.log(arr1.length)
arr1[1] =33
console.log(arr1.length)
console.log(new Array())
 ```
 这种或多或少会有问题，无法因为，这种Array.call(this)这种Array的内部是无法拿到this的
 ```
 class MyArray extends Array{
    constructor(){
        super()
    }
    getNamelk(){
        console.log("lkkkk")
    }
}
let lk = new MyArray();
console.log(lk)
lk.getNamelk()
lk.push(111)
console.log(lk)
 ```
 这种使用了es6的方法可以很好的定义自己的数组的方法，完美的继承了数组的属性
 get set
 ```
 class A {
    get value(){
       return this.name
    }
    set value(value){
        this.name = value
    }
}
let a = new A();
a.value=1111
console.log(a.value)
 ```
 class的 generator
 ```
 class A{
    constructor(...arr){
        this.arr=arr
    }
    *[Symbol.iterator](){
        for(let i of this.arr){
            yield i
        }
    }
}
let a = new A(1,2,3,4,5,6)
for(let m of a){
    console.log(m)
}
 ```
混合模式
```
 function mix (...max){
     class mix {}
     for(let m of max){
         copeProtos(mix,m)
         copeProtos(mix.prototype,m.prototype)
     }
     return mix
 }
 function copeProtos(a,b){
     for(let i of Reflect.ownKeys(b)){
     if(i!="prototype"&&i!="constructor"&&i!="name"){
          let dec = Object.getOwnPropertyDescriptor(b,i);
        Object.defineProperty(a,i,dec)

    }
}
}
class A{
    name=11
}
class B{
    name=2323;
    say(){
        console.log("mmm")
    }

}
const MaxAll = mix(A,B)
aa = new MaxAll()
console.log(aa.say())

```
### differ算法
differ算法就是找到jsx和current Fiber的差异 

###### 规则有三 
* 1整个树时，前后跨越层级的时候去的重建
* 2 相同组件有相识的结构，如果发生了div 变成p这种情况重建，
* 使用key 暗示react元素的不变
###### 根据newchildren 的类型，有object（elementTpye）、number、string为单节点，当为array 时候为多节点。规则有两种，
单个节点
```
<div key="1"><div>
<div key="2"><div>
// key 不同重建

<div key="1"></div>
<p key="1"></div>
// type 不同重建

<div key="1">1</div>
<div key="1">2</div>
// 复用
```
2 一对多时
```
// 老的
ul>li*3
// 新的
ul>p
```
会先判断 key,如果key 不同，就删除掉后面的li,表示p我们已经找到了对应的li
。如果key不同，type相对，给当前元素标记删除。
源码
```
function reconcileSingleElement(
  returnFiber: Fiber,
  currentFirstChild: Fiber | null,
  element: ReactElement
): Fiber {
  const key = element.key;
  let child = currentFirstChild;
  
  // 首先判断是否存在对应DOM节点
  while (child !== null) {
    // 上一次更新存在DOM节点，接下来判断是否可复用

    // 首先比较key是否相同
    if (child.key === key) {

      // key相同，接下来比较type是否相同

      switch (child.tag) {
        // ...省略case
        
        default: {
          if (child.elementType === element.type) {
            // type相同则表示可以复用
            // 返回复用的fiber
            return existing;
          }
          
          // type不同则跳出switch
          break;
        }
      }
      // 代码执行到这里代表：key相同但是type不同
      // 将该fiber及其兄弟fiber标记为删除
      deleteRemainingChildren(returnFiber, child);
      break;
    } else {
      // key不同，将该fiber标记为删除
      deleteChild(returnFiber, child);
    }
    child = child.sibling;
  }

  // 创建新Fiber，并返回 ...省略
}
```
###### 多个节点的differ
1 存在在多种情况，更新，删除和增加，由于react官方认为，更新的频率更高，所以选择跟新优先级更高
####### 遍历方式，由于返回的jsx的是这样的
```
{
  $$typeof: Symbol(react.element),
  key: null,
  props: {
    children: [
      {$$typeof: Symbol(react.element), type: "li", key: "0", ref: null, props: {…}, …}
      {$$typeof: Symbol(react.element), type: "li", key: "1", ref: null, props: {…}, …}
      {$$typeof: Symbol(react.element), type: "li", key: "2", ref: null, props: {…}, …}
      {$$typeof: Symbol(react.element), type: "li", key: "3", ref: null, props: {…}, …}
    ]
  },
  ref: null,
  type: "ul"
}
```
而，fiber，是单链表sibling获取，无法双指针遍历
###### 遍历第一次
```
// 之前
<li key="0">0</li>
<li key="1">1</li>
<li key="2">2</li>
            
// 之后
<li key="0">0</li>
<li key="2">1</li>
<li key="1">2</li>
```
* 遍历children[0],olderfiber 判断是否可用。
* 如果可用继续遍历。i++
* 如果不可用，判断不用的场景，如果是key 不同直接跳出遍历。如果type不同在标记delect
* 直到所有遍历完成

###### 第二次遍历
情况如下
* oldfiber遍历完了，childer没遍历完，把剩下的newchiled生成的workinprogress.fiber 以此标记为placement
* oldfiber 没遍历完，chiderb遍历完了，这是标记为deletion
* 两者都没遍历完成
#### 处理移动的节点
会初始化一个lastindex的值，如果oldefiber.index<=lastindex 就往后移动(往后添加元素的开销最小)，然后跟新当前oldefiber.index 和lastindex ,lastindex为当前两者最大值。以此类推，继续往下比
```
// 之前
abcd

// 之后
acdb

===第一轮遍历开始===
a（之后）vs a（之前）  
key不变，可复用
此时 a 对应的oldFiber（之前的a）在之前的数组（abcd）中索引为0
所以 lastPlacedIndex = 0;

继续第一轮遍历...

c（之后）vs b（之前）  
key改变，不能复用，跳出第一轮遍历
此时 lastPlacedIndex === 0;
===第一轮遍历结束===

===第二轮遍历开始===
newChildren === cdb，没用完，不需要执行删除旧节点
oldFiber === bcd，没用完，不需要执行插入新节点

将剩余oldFiber（bcd）保存为map

// 当前oldFiber：bcd
// 当前newChildren：cdb

继续遍历剩余newChildren

key === c 在 oldFiber中存在
const oldIndex = c（之前）.index;
此时 oldIndex === 2;  // 之前节点为 abcd，所以c.index === 2
比较 oldIndex 与 lastPlacedIndex;

如果 oldIndex >= lastPlacedIndex 代表该可复用节点不需要移动
并将 lastPlacedIndex = oldIndex;
如果 oldIndex < lastplacedIndex 该可复用节点之前插入的位置索引小于这次更新需要插入的位置索引，代表该节点需要向右移动

在例子中，oldIndex 2 > lastPlacedIndex 0，
则 lastPlacedIndex = 2;
c节点位置不变

继续遍历剩余newChildren

// 当前oldFiber：bd
// 当前newChildren：db

key === d 在 oldFiber中存在
const oldIndex = d（之前）.index;
oldIndex 3 > lastPlacedIndex 2 // 之前节点为 abcd，所以d.index === 3
则 lastPlacedIndex = 3;
d节点位置不变

继续遍历剩余newChildren

// 当前oldFiber：b
// 当前newChildren：b

key === b 在 oldFiber中存在
const oldIndex = b（之前）.index;
oldIndex 1 < lastPlacedIndex 3 // 之前节点为 abcd，所以b.index === 1
则 b节点需要向右移动
===第二轮遍历结束===

最终acd 3个节点都没有移动，b节点被标记为移动
```
```
// 之前
abcd

// 之后
dabc

===第一轮遍历开始===
d（之后）vs a（之前）  
key改变，不能复用，跳出遍历
===第一轮遍历结束===

===第二轮遍历开始===
newChildren === dabc，没用完，不需要执行删除旧节点
oldFiber === abcd，没用完，不需要执行插入新节点

将剩余oldFiber（abcd）保存为map

继续遍历剩余newChildren

// 当前oldFiber：abcd
// 当前newChildren dabc

key === d 在 oldFiber中存在
const oldIndex = d（之前）.index;
此时 oldIndex === 3; // 之前节点为 abcd，所以d.index === 3
比较 oldIndex 与 lastPlacedIndex;
oldIndex 3 > lastPlacedIndex 0
则 lastPlacedIndex = 3;
d节点位置不变

继续遍历剩余newChildren

// 当前oldFiber：abc
// 当前newChildren abc

key === a 在 oldFiber中存在
const oldIndex = a（之前）.index; // 之前节点为 abcd，所以a.index === 0
此时 oldIndex === 0;
比较 oldIndex 与 lastPlacedIndex;
oldIndex 0 < lastPlacedIndex 3
则 a节点需要向右移动

继续遍历剩余newChildren

// 当前oldFiber：bc
// 当前newChildren bc

key === b 在 oldFiber中存在
const oldIndex = b（之前）.index; // 之前节点为 abcd，所以b.index === 1
此时 oldIndex === 1;
比较 oldIndex 与 lastPlacedIndex;
oldIndex 1 < lastPlacedIndex 3
则 b节点需要向右移动

继续遍历剩余newChildren

// 当前oldFiber：c
// 当前newChildren c

key === c 在 oldFiber中存在
const oldIndex = c（之前）.index; // 之前节点为 abcd，所以c.index === 2
此时 oldIndex === 2;
比较 oldIndex 与 lastPlacedIndex;
oldIndex 2 < lastPlacedIndex 3
则 c节点需要向右移动

===第二轮遍历结束===
```
