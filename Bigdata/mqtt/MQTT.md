# MQTT

MQTT（Message Queuing Telemetry Transport，消息队列遥测传输协议），是一种基于发布/订阅（publish/subscribe）模式的"轻量级"通讯协议，该协议构建于TCP/IP协议上，由IBM在1999年发布。  
MQTT最大优点在于，可以以极少的代码和有限的带宽，为连接远程设备提供实时可靠的消息服务。作为一种低开销、低带宽占用的即时通讯协议，使其在物联网、小型设备、移动应用等方面有较广泛的应用。  
MQTT本身并不是基于WebSocket的，但MQTT可以通过WebSocket进行传输。  
paho-mqtt 是目前 Python 中使用较为广泛的 MQTT 客户端库，它在 Python 2.7 及 3.x 上为客户端提供了对 MQTT v5.0、v3.1.1 和 v3.1 的支持。  
paho-mqtt 库：  

```

pip install paho-mqtt  

```

# 7 种 实现 web 实时消息推送的方案  

原文：[我有 7种 实现web实时消息推送的方案 - 掘金](https://juejin.cn/post/7122014462181113887#heading-5)  
[Web 实时消息推送详解 | JavaGuide](https://javaguide.cn/system-design/web-real-time-message-push.html)  
我有一个朋友做了一个小破站，现在要实现一个站内信 web 消息推送的功能，对，就是下图这个小红点，一个很常用的功能。  
不过他还没想好用什么方式做，这里我帮他整理了一下几种方案，并简单做了实现。  

![clearoff (5).png](https://cdn.nlark.com/yuque/0/2024/png/40959717/1733637749832-e317bfe4-7ccb-4332-817b-2dfc492b568e.png)

[案例下载](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fchengxy-nds%2FSpringboot-Notebook%2Ftree%2Fmaster%2Fspringboot-realtime-data)  
### 什么是消息推送（push）  

推送的场景比较多，比如有人关注我的公众号，这时我就会收到一条推送消息，以此来吸引我点击打开应用。  
消息推送 (push) 通常是指网站的运营工作等人员，通过某种工具对用户当前网页或移动设备 APP 进行的主动消息推送。  
消息推送一般又分为web端消息推送和移动端消息推送。  

![图片](https://cdn.nlark.com/yuque/0/2024/webp/40959717/1733637823237-502b01d8-b19f-4b35-a647-130a034a4894.webp)

上边的这种属于移动端消息推送，web 端消息推送常见的诸如站内信、未读邮件数量、监控报警数量等，应用的也非常广泛。  

![图片](https://cdn.nlark.com/yuque/0/2024/webp/40959717/1733637823208-01a21e77-1e83-48bc-90d6-4cf84625d89e.webp)

在具体实现之前，咱们再来分析一下前边的需求，其实功能很简单，只要触发某个事件（主动分享了资源或者后台主动推送消息），web 页面的通知小红点就会实时的+1就可以了。  
通常在服务端会有若干张消息推送表，用来记录用户触发不同事件所推送不同类型的消息，前端主动查询（拉）或者被动接收（推）用户所有未读的消息数。  

![图片](https://cdn.nlark.com/yuque/0/2024/webp/40959717/1733637823209-44efe757-2268-4999-baac-704a09c4b76e.webp)

消息推送无非是推（push）和拉（pull）两种形式，下边我们逐个了解下。  
### 短轮询  

轮询 (polling) 应该是实现消息推送方案中最简单的一种，这里我们暂且将轮询分为短轮询和长轮询。  
短轮询很好理解，指定的时间间隔，由浏览器向服务器发出HTTP请求，服务器实时返回未读消息数据给客户端，浏览器再做渲染显示。  
一个简单的 JS 定时器就可以搞定，每秒钟请求一次未读消息数接口，返回的数据展示即可。  

```

setInterval(() => {  
  // 方法请求  
  messageCount().then((res) => {  
    if (res.code === 200) {  
      this.messageCount = res.data  
    }  
  })  
}, 1000);  

```

效果还是可以的，短轮询实现固然简单，缺点也是显而易见，由于推送数据并不会频繁变更，无论后端此时是否有新的消息产生，客户端都会进行请求，势必会对服务端造成很大压力，浪费带宽和服务器资源。  

![图片](https://cdn.nlark.com/yuque/0/2024/webp/40959717/1733637823493-cee89ff4-fce7-46bd-9b43-9404601f6762.webp?date=1779972681459)

### 长轮询  

长轮询是对上边短轮询的一种改进版本，在尽可能减少对服务器资源浪费的同时，保证消息的相对实时性。长轮询在中间件中应用的很广泛，比如Nacos和apollo配置中心，消息队列kafka、RocketMQ中都有用到长轮询。  
[Nacos 配置中心交互模型是 push 还是 pull？](https://link.juejin.cn/?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2F94ftESkDoZI9gAGflLiGwg)一文中我详细介绍过Nacos长轮询的实现原理，感兴趣的小伙伴可以瞅瞅。  
这次我使用apollo配置中心实现长轮询的方式，应用了一个类DeferredResult，它是在servelet3.0后经过 Spring 封装提供的一种异步请求机制，直意就是延迟结果。  

![图片](https://cdn.nlark.com/yuque/0/2024/webp/40959717/1733637823502-69045636-0fa5-4083-86ed-284d008390da.webp)

DeferredResult可以允许容器线程快速释放占用的资源，不阻塞请求线程，以此接受更多的请求提升系统的吞吐量，然后启动异步工作线程处理真正的业务逻辑，处理完成调用DeferredResult.setResult(200)提交响应结果。  
下边我们用长轮询来实现消息推送。  
因为一个 ID 可能会被多个长轮询请求监听，所以我采用了guava包提供的Multimap结构存放长轮询，一个 key 可以对应多个 value。一旦监听到 key 发生变化，对应的所有长轮询都会响应。前端得到非请求超时的状态码，知晓数据变更，主动查询未读消息数接口，更新页面数据。  

```

@Controller  
@RequestMapping("/polling")  
public class PollingController {  

    // 存放监听某个Id的长轮询集合  
    // 线程同步结构  
    public static Multimap\> watchRequests = Multimaps.synchronizedMultimap(HashMultimap.create());  

    /\*\*  
\* 设置监听  
\*/  
    @GetMapping(path = "watch/{id}")  
    @ResponseBody  
    public DeferredResult watch(@PathVariable String id) {  
        // 延迟对象设置超时时间  
        DeferredResult deferredResult = new DeferredResult<>(TIME_OUT);  
        // 异步请求完成时移除 key，防止内存溢出  
        deferredResult.onCompletion(() -> {  
            watchRequests.remove(id, deferredResult);  
        });  
        // 注册长轮询请求  
        watchRequests.put(id, deferredResult);  
        return deferredResult;  
    }  

    /\*\*  
\* 变更数据  
\*/  
    @GetMapping(path = "publish/{id}")  
    @ResponseBody  
    public String publish(@PathVariable String id) {  
        // 数据变更 取出监听ID的所有长轮询请求，并一一响应处理  
        if (watchRequests.containsKey(id)) {  
            Collection\> deferredResults = watchRequests.get(id);  
            for (DeferredResult deferredResult : deferredResults) {  
                deferredResult.setResult("我更新了" + new Date());  
            }  
        }  
        return "success";  
    }

```

当请求超过设置的超时时间，会抛出AsyncRequestTimeoutException异常，这里直接用@ControllerAdvice全局捕获统一返回即可，前端获取约定好的状态码后再次发起长轮询请求，如此往复调用。  

```

@ControllerAdvice  
public class AsyncRequestTimeoutHandler {  

    @ResponseStatus(HttpStatus.NOT_MODIFIED)  
    @ResponseBody  
    @ExceptionHandler(AsyncRequestTimeoutException.class)  
    public String asyncRequestTimeoutHandler(AsyncRequestTimeoutException e) {  
        System.out.println("异步请求超时");  
        return "304";  
    }  
}  

```

我们来测试一下，首先页面发起长轮询请求/polling/watch/10086监听消息更变，请求被挂起，不变更数据直至超时，再次发起了长轮询请求；紧接着手动变更数据/polling/publish/10086，长轮询得到响应，前端处理业务逻辑完成后再次发起请求，如此循环往复。  

![图片](https://cdn.nlark.com/yuque/0/2024/webp/40959717/1733637823745-fc28bce5-df66-4711-aa71-382c38a93021.webp?date=1779972687695)

长轮询相比于短轮询在性能上提升了很多，但依然会产生较多的请求，这是它的一点不完美的地方。  
### iframe 流  

iframe 流就是在页面中插入一个隐藏的\<iframe>标签，通过在src中请求消息数量 API 接口，由此在服务端和客户端之间创建一条长连接，服务端持续向iframe传输数据。  
传输的数据通常是HTML、或是内嵌的javascript脚本，来达到实时更新页面的效果。  

![图片](https://cdn.nlark.com/yuque/0/2024/webp/40959717/1733637823815-3037d977-67ab-47aa-9b86-fcb3031deed3.webp)

这种方式实现简单，前端只要一个\<iframe>标签搞定了  

```

```

服务端直接组装 html、js 脚本数据向response写入就行了  

```

@Controller  
@RequestMapping("/iframe")  
public class IframeController {  
    @GetMapping(path = "message")  
    public void message(HttpServletResponse response) throws IOException, InterruptedException {  
        while (true) {  
            response.setHeader("Pragma", "no-cache");  
            response.setDateHeader("Expires", 0);  
            response.setHeader("Cache-Control", "no-cache,no-store");  
            response.setStatus(HttpServletResponse.SC_OK);  
            response.getWriter().print(" \\n"&nbsp+\<br/>&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp"parent.document.getElementById('clock').innerHTML = \\""&nbsp+&nbspcount.get()&nbsp+&nbsp"\\";"&nbsp+\<br/>&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp"parent.document.getElementById('count').innerHTML = \\""&nbsp+&nbspcount.get()&nbsp+&nbsp"\\";"&nbsp+\<br/>&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp"");  
        }  
    }  
}  

```

但我个人不推荐，因为它在浏览器上会显示请求未加载完，图标会不停旋转，简直是强迫症杀手。  

![图片](https://cdn.nlark.com/yuque/0/2024/webp/40959717/1733637823839-04fd9f90-9e24-47ae-89f3-45bda0909b1c.webp?date=1779972690031)

### SSE (我的方式)  

很多人可能不知道，服务端向客户端推送消息，其实除了可以用WebSocket这种耳熟能详的机制外，还有一种服务器发送事件 (Server-sent events)，简称SSE。  
SSE它是基于HTTP协议的，我们知道一般意义上的 HTTP 协议是无法做到服务端主动向客户端推送消息的，但 SSE 是个例外，它变换了一种思路。  

![图片](https://cdn.nlark.com/yuque/0/2024/webp/40959717/1733637824188-04b7cda5-8d5e-4e94-a2f9-ba7f6d8ec337.webp)

SSE 在服务器和客户端之间打开一个单向通道，服务端响应的不再是一次性的数据包而是text/event-stream类型的数据流信息，在有数据变更时从服务器流式传输到客户端。  
整体的实现思路有点类似于在线视频，视频流会连续不断的推送到浏览器，你也可以理解成，客户端在完成一次用时很长（网络不畅）的下载。  

![图片](https://cdn.nlark.com/yuque/0/2024/webp/40959717/1733637824044-ba02fd42-89c5-4c2b-b98e-4d50436809ac.webp)

SSE与WebSocket作用相似，都可以建立服务端与浏览器之间的通信，实现服务端向客户端推送消息，但还是有些许不同：  
●SSE 是基于 HTTP 协议的，它们不需要特殊的协议或服务器实现即可工作；WebSocket需单独服务器来处理协议。  
●SSE 单向通信，只能由服务端向客户端单向通信；webSocket 全双工通信，即通信的双方可以同时发送和接受信息。  
●SSE 实现简单开发成本低，无需引入其他组件；WebSocket 传输数据需做二次解析，开发门槛高一些。  
●SSE 默认支持断线重连；WebSocket 则需要自己实现。  
●SSE 只能传送文本消息，二进制数据需要经过编码后传送；WebSocket 默认支持传送二进制数据。  
SSE 与 WebSocket 该如何选择？  
技术并没有好坏之分，只有哪个更合适  
SSE 好像一直不被大家所熟知，一部分原因是出现了 WebSockets，这个提供了更丰富的协议来执行双向、全双工通信。对于游戏、即时通信以及需要双向近乎实时更新的场景，拥有双向通道更具吸引力。  
但是，在某些情况下，不需要从客户端发送数据。而你只需要一些服务器操作的更新。比如：站内信、未读消息数、状态更新、股票行情、监控数量等场景，SEE不管是从实现的难易和成本上都更加有优势。此外，SSE 具有WebSockets在设计上缺乏的多种功能，例如：自动重新连接、事件ID和发送任意事件的能力。  
前端只需进行一次 HTTP 请求，带上唯一 ID，打开事件流，监听服务端推送的事件就可以了  

```

\<br/>&nbsp&nbsplet&nbspsource&nbsp=&nbspnull;\<br/>&nbsp&nbsplet&nbspuserId&nbsp=&nbsp7777\<br/>&nbsp&nbspif&nbsp(window.EventSource)&nbsp{\<br/>&nbsp&nbsp&nbsp&nbsp// 建立连接\<br/>&nbsp&nbsp&nbsp&nbspsource&nbsp=&nbspnew&nbspEventSource('http://localhost:7777/sse/sub/'+userId);\<br/>&nbsp&nbsp&nbsp&nbspsetMessageInnerHTML("连接用户="&nbsp+&nbspuserId);\<br/>&nbsp&nbsp&nbsp&nbsp/\*\*\<br/> \* 连接一旦建立，就会触发open事件\<br/> \* 另一种写法：source.onopen = function (event) {}\<br/> \*/\<br/>&nbsp&nbsp&nbsp&nbspsource.addEventListener('open',&nbspfunction&nbsp(e)&nbsp{\<br/>&nbsp&nbsp&nbsp&nbsp&nbsp&nbspsetMessageInnerHTML("建立连接。。。");\<br/>&nbsp&nbsp&nbsp&nbsp},&nbspfalse);\<br/>&nbsp&nbsp&nbsp&nbsp/\*\*\<br/> \* 客户端收到服务器发来的数据\<br/> \* 另一种写法：source.onmessage = function (event) {}\<br/> \*/\<br/>&nbsp&nbsp&nbsp&nbspsource.addEventListener('message',&nbspfunction&nbsp(e)&nbsp{\<br/>&nbsp&nbsp&nbsp&nbsp&nbsp&nbspsetMessageInnerHTML(e.data);\<br/>&nbsp&nbsp&nbsp&nbsp});\<br/>&nbsp&nbsp}&nbspelse&nbsp{\<br/>&nbsp&nbsp&nbsp&nbspsetMessageInnerHTML("你的浏览器不支持SSE");\<br/>&nbsp&nbsp}\<br/>  

```

服务端的实现更简单，创建一个SseEmitter对象放入sseEmitterMap进行管理  

```

private static Map sseEmitterMap = new ConcurrentHashMap<>();  

/\*\*  
\* 创建连接  
\*/  
public static SseEmitter connect(String userId) {  
    try {  
        // 设置超时时间，0表示不过期。默认30秒  
        SseEmitter sseEmitter = new SseEmitter(0L);  
        // 注册回调  
        sseEmitter.onCompletion(completionCallBack(userId));  
        sseEmitter.onError(errorCallBack(userId));  
        sseEmitter.onTimeout(timeoutCallBack(userId));  
        sseEmitterMap.put(userId, sseEmitter);  
        count.getAndIncrement();  
        return sseEmitter;  
    } catch (Exception e) {  
        log.info("创建新的sse连接异常，当前用户：{}", userId);  
    }  
    return null;  
}  

/\*\*  
\* 给指定用户发送消息  
\*/  
public static void sendMessage(String userId, String message) {  

    if (sseEmitterMap.containsKey(userId)) {  
        try {  
            sseEmitterMap.get(userId).send(message);  
        } catch (IOException e) {  
            log.error("用户[{}]推送异常:{}", userId, e.getMessage());  
            removeUser(userId);  
        }  
    }  
}

```

我们模拟服务端推送消息，看下客户端收到了消息，和我们预期的效果一致。  

![图片](https://cdn.nlark.com/yuque/0/2024/webp/40959717/1733637824310-b0555f92-7188-4121-88c5-5926639e8176.webp?date=1779972693779)

注意： SSE 不支持IE浏览器，对其他主流浏览器兼容性做的还不错。  

![图片](https://cdn.nlark.com/yuque/0/2024/webp/40959717/1733637824402-d55d4804-c732-4284-8b42-2d0708362618.webp)

### MQTT  

什么是 MQTT 协议？  
MQTT 全称 (Message Queue Telemetry Transport)：一种基于发布 / 订阅（publish/subscribe）模式的轻量级通讯协议，通过订阅相应的主题来获取消息，是物联网（Internet of Thing）中的一个标准传输协议。  
该协议将消息的发布者（publisher）与订阅者（subscriber）进行分离，因此可以在不可靠的网络环境中，为远程连接的设备提供可靠的消息服务，使用方式与传统的 MQ 有点类似。  

![图片](https://cdn.nlark.com/yuque/0/2024/webp/40959717/1733637824410-82bfbdde-eedc-4818-8d2e-8c3c83bffa85.webp)

TCP协议位于传输层，MQTT协议位于应用层，MQTT 协议构建于TCP/IP协议上，也就是说只要支持TCP/IP协议栈的地方，都可以使用MQTT协议。  
为什么要用 MQTT 协议？  
MQTT协议为什么在物联网（IOT）中如此受偏爱？而不是其它协议，比如我们更为熟悉的 HTTP协议呢？  
●首先HTTP协议它是一种同步协议，客户端请求后需要等待服务器的响应。而在物联网（IOT）环境中，设备会很受制于环境的影响，比如带宽低、网络延迟高、网络通信不稳定等，显然异步消息协议更为适合IOT应用程序。  
●HTTP是单向的，如果要获取消息客户端必须发起连接，而在物联网（IOT）应用程序中，设备或传感器往往都是客户端，这意味着它们无法被动地接收来自网络的命令。  
●通常需要将一条命令或者消息，发送到网络上的所有设备上。HTTP要实现这样的功能不但很困难，而且成本极高。  
具体的 MQTT 协议介绍和实践，这里我就不再赘述了，大家可以参考我之前的两篇文章，里边写的也都很详细了。  
MQTT 协议的介绍  
[我也没想到 springboot + rabbitmq 做智能家居，会这么简单](https://link.juejin.cn/?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2FudFE6k9pPetIWsa6KeErrA)  
MQTT 实现消息推送  
[未读消息（小红点），前端 与 RabbitMQ 实时消息推送实践，贼简单~](https://link.juejin.cn/?target=https%3A%2F%2Fmp.weixin.qq.com%2Fs%2FU-fUGr9i1MVa4PoVyiDFCg)  
### Websocket  

websocket应该是大家都比较熟悉的一种实现消息推送的方式，上边我们在讲 SSE 的时候也和 websocket 进行过比较。  
WebSocket 是一种在TCP连接上进行全双工通信的协议，建立客户端和服务器之间的通信渠道。浏览器和服务器仅需一次握手，两者之间就直接可以创建持久性的连接，并进行双向数据传输。  

springboot 整合 websocket，先引入websocket相关的工具包，和 SSE 相比额外的开发成本。  

```

[空代码块]  

```

服务端使用@ServerEndpoint注解标注当前类为一个 websocket 服务器，客户端可以通过ws://localhost:7777/webSocket/10086来连接到 WebSocket 服务器端。  

```

[空代码块]  

```

前端初始化打开 WebSocket 连接，并监听连接状态，接收服务端数据或向服务端发送数据。  

```

[空代码块]  

```

页面初始化建立 websocket 连接，之后就可以进行双向通信了，效果还不错  

![图片](https://cdn.nlark.com/yuque/0/2024/webp/40959717/1733637824929-15dbb27c-2b0c-4b91-89a4-cd310fa1a1dc.webp?date=1779972699497)

### 自定义推送  

上边我们给我出了 6 种方案的原理和代码实现，但在实际业务开发过程中，不能盲目的直接拿过来用，还是要结合自身系统业务的特点和实际场景来选择合适的方案。  
推送最直接的方式就是使用第三推送平台，毕竟钱能解决的需求都不是问题，无需复杂的开发运维，直接可以使用，省时、省力、省心，像 goEasy、极光推送都是很不错的三方服务商。  
一般大型公司都有自研的消息推送平台，像我们本次实现的 web 站内信只是平台上的一个触点而已，短信、邮件、微信公众号、小程序凡是可以触达到用户的渠道都可以接入进来。  

![图片](https://cdn.nlark.com/yuque/0/2024/webp/40959717/1733637825055-a2454b8a-2e91-4591-abc8-59908a07e1c4.webp)

消息推送系统内部是相当复杂的，诸如消息内容的维护审核、圈定推送人群、触达过滤拦截（推送的规则频次、时段、数量、黑白名单、关键词等等）、推送失败补偿非常多的模块，技术上涉及到大数据量、高并发的场景也很多。所以我们今天的实现方式在这个庞大的系统面前只是小打小闹。  
### Github 地址  

文中所提到的案例我都一一的做了实现，整理放在了Github上，觉得有用就 Star 一下吧！  
传送门：[https://github.com/chengxy-nds/Springboot-Notebook](https://github.com/chengxy-nds/Springboot-Notebook)  
# MQTT简介  

●MQTT 协议 3.1.1：[https://www.runoob.com/manual/mqtt/protocol/MQTT-3.1.1-CN.html](https://www.runoob.com/manual/mqtt/protocol/MQTT-3.1.1-CN.html)  
●MQTT 协议 3.1.1 PDF：[https://www.runoob.com/manual/mqtt/protocol/MQTT-3.1.1-CN.pdf](https://www.runoob.com/manual/mqtt/protocol/MQTT-3.1.1-CN.pdf)  
## mqtt 简介  

MQTT（Message Queuing Telemetry Transport，消息队列遥测传输协议），是一种基于发布 / 订阅（publish/subscribe）模式的 "轻量级" 通讯协议，该协议构建于 TCP/IP 协议上，由 IBM 在 1999 年发布  
由于物联网的环境是非常特别的，所以 MQTT 遵循以下设计原则：  
●（1）精简，不添加可有可无的功能；  
●（2）发布 / 订阅（Pub/Sub）模式，方便消息在传感器之间传递；  
●（3）允许用户动态创建主题，零运维成本；  
●（4）把传输量降到最低以提高传输效率；  
●（5）把低带宽、高延迟、不稳定的网络等因素考虑在内；  
●（6）支持连续的会话控制；  
●（7）理解客户端计算能力可能很低；  
●（8）提供服务质量管理；  
●（9）假设数据不可知，不强求传输数据的类型与格式，保持灵活性。  
## mqtt基本概念  

●Publisher（发布者）：消息的发出者，负责发送消息。  
●Subscriber（订阅者）：消息的订阅者，负责接收并处理消息。  
●Broker（代理）：消息代理，位于消息发布者和订阅者之间，各类支持 MQTT 协议的消息中间件都可以充当。  
●Topic（主题）：可以理解为消息队列中的路由，订阅者订阅了主题之后，就可以收到发送到该主题的消息。  
●Payload（负载）；可以理解为发送消息的内容。  
●QoS（消息质量）：全称 Quality of Service，即消息的发送质量，主要有QoS 0、QoS 1、QoS 2三个等级，下面分别介绍下：  
○QoS 0（Almost Once）：至多一次，只发送一次，会发生消息丢失或重复；  
○QoS 1（Atleast Once）：至少一次，确保消息到达，但消息重复可能会发生；  
○QoS 2（Exactly Once）：只有一次，确保消息只到达一次。  
## mqtt 的实现  

实现 MQTT 协议需要客户端和服务器端通讯完成，在通讯过程中，MQTT 协议中有三种身份：发布者（Publish）、代理（Broker）（服务器）、订阅者（Subscribe）。其中，消息的发布者和订阅者都是客户端，消息代理是服务器，消息发布者可以同时是订阅者。  
MQTT 传输的消息分为：主题（Topic）和负载（payload）两部分：  
1Topic，可以理解为消息的类型，订阅者订阅（Subscribe）后，就会收到该主题的消息内容（payload）；  
2payload，可以理解为消息的内容，是指订阅者具体要使用的内容。  
当应用数据通过 MQTT 网络发送时，MQTT 会把与之相关的服务质量（QoS）和主题名（Topic）相关连。  
说明：  
11个客户端既可以是订阅者也可以是发布者  
2可以通过通配符一次订阅多个topic  
## mqtt 和 websocket 结合  

MQTT 原本是基于 TCP/IP 的协议，但为了适应 Web 应用的需求，MQTT 也可以通过 WebSocket 传输。  
优点：使用 websocket 的 mqtt 与直接使用 websocket 通信相比，前后台开发工作量能够有效减少，主要有以下几点：  
1在 Web 环境中的适用性：WebSocket 是 Web 环境中的标准双向通信协议，可以很容易地在浏览器中使用。结合 MQTT 和 WebSocket，浏览器客户端可以直接与 MQTT 代理（Broker）进行通信。  
2实时通信：MQTT 和 WebSocket 都支持实时通信，结合使用可以在 Web 应用中实现高效、低延迟的数据传输。  
3跨平台：使用 WebSocket，MQTT 可以在各种平台（包括浏览器、移动设备和桌面应用）上使用，无需额外的网络库。  
4mqtt 自带鉴权，后台 socket 不需要编写登陆验证等逻辑  
5mqtt 自带 topic，可以实现消息分发，不需要编写群发等复杂程序逻辑  
6mqtt 自带消息确认功能（qos 1 或者 2），不需要编写消息回传逻辑  
缺点：  
1需要单独在后台独立运行 mqtt broker  
2需要增加应用程序与 broker 之间的通信逻辑  

![image.png](https://cdn.nlark.com/yuque/0/2024/png/40959717/1734447155771-59da401e-411a-4ad6-b148-8a48f34afbad.png)

![图片](https://cdn.nlark.com/yuque/0/2024/png/40959717/1734447161979-34d66524-0229-44b9-a5b0-604f1c5f778a.png)

## 可用 mqtt broker  

目前可用 broker 有 mosquitto、emqx、activemq 等，下表列出几种对比  

![图片](https://cdn.nlark.com/yuque/0/2024/png/40959717/1733643442499-10ff8cbc-5867-4b82-afe6-37daa3d5c270.png)

## mqtt错误码  

[MQTT 客户端错误码 | EMQX Platform 文档](https://docs.emqx.com/zh/cloud/latest/connect_to_deployments/mqtt_client_error_codes.html)  
## 公共MQTT Broker服务器  

1Eclipse Mosquitto Broker  
○地址：[test.mosquitto.org](http://test.mosquitto.org)  
○端口：  
■1883：MQTT，未加密，未认证。  
■1884：MQTT，未加密，已认证。  
■8883：MQTT，加密，未认证。  
■8884：MQTT，加密，需要客户端证书。  
■8885：MQTT，加密，已认证。  
■8886：MQTT，加密，未认证（与8883类似，但可能有不同的配置或用途）。  
■8887：MQTT，加密，服务器证书故意过期（用于测试证书过期的情况）。  
■8080：MQTT over WebSockets，未加密，未认证。  
■8081：MQTT over WebSockets，加密，未认证。  
■8090：MQTT over WebSockets，未加密，已认证。  
■8091：MQTT over WebSockets，加密，已认证。  
2mqtt.eclipseprojects.io  
○端口  
■1883 : MQTT over unencrypted TCP  
■8883 : MQTT over encrypted TCP  
■80 : MQTT over unencrypted WebSockets (note: URL must be /mqtt )  
■443 : MQTT over encrypted WebSockets (note: URL must be /mqtt )  
3HiveMQ Public MQTT Broker  
○地址：[broker.hivemq.com](http://broker.hivemq.com)  
○端口：  
■TCP：1883  
■WebSocket：8000  
○示例连接字符串：ws://broker.hivemq.com:8000/mqtt  
4EMQX Public MQTT Broker  
○地址：[broker.emqx.io](http://broker.emqx.io)  
○端口：  
■TCP：1883  
■WebSocket：8083  
■SSL/TLS：8883  
■SSL/TLS WebSocket：8084  
○示例连接字符串：ws://broker.emqx.io:8083/mqtt  
5免费开放的MQTT服务器（由个人或组织提供，可能存在一定的不稳定性和安全风险）  
○地址：[mqtt://47.94.220.165:1833](http://mqtt://47.94.220.165:1833)  
使用这些公共MQTT服务器时，请注意以下几点：  
●安全性：由于这些服务器是公共的，因此可能存在安全风险。在使用之前，请确保了解并接受这些风险。  
●稳定性：公共服务器的稳定性可能不如自己搭建的服务器。在生产环境中使用时，请务必进行充分的测试。  
●带宽和流量限制：一些公共服务器可能对带宽和流量进行限制。如果发送的数据量较大，请确保了解这些限制并考虑其他选项。  
另外，如果需要搭建自己的MQTT服务器，可以选择使用Eclipse Mosquitto、HiveMQ、EMQX等开源或商业MQTT服务器软件。这些软件提供了丰富的功能和配置选项，可以满足不同场景下的需求。  
使用方法：  
协议//主机名.域名:端口/路径（路径是可选的）  
例如：mqtt://test.mosquitto.org:1883/mqtt  
ws://test.mosquitto.org:8080/mqtt  
协议：  
WebSocket 是一种在客户端和服务器之间进行双向通信的协议，允许实时数据传输。它有两种常见的连接方式：  
1ws：未加密的 WebSocket 连接  
2wss：加密的 WebSocket 连接  
MQTT 是一种轻量级的发布/订阅消息传输协议，通常用于物联网应用程序。MQTT 也可以通过不同的连接方式进行通信：  
1mqtt：未加密的 TCP 连接，默认使用端口号 1883。  
2mqtts：加密的 TCP 连接。  
## keepalive机制  

[【MQTT学习】lesson9：Keep Alive 和连接保活_mqtt keepalive-CSDN博客](https://blog.csdn.net/qq997758497/article/details/90677479)  
## 保留消息  

原文地址 [www.cnblogs.com](https://www.cnblogs.com/emqx/p/16833311.html)  
[MQTT 保留消息是什么？如何使用？ - MQTT中文站](https://www.mqtt.cn/1881.html)  
### 什么是 MQTT 保留消息？  

发布者发布消息时，如果 Retained 标记被设置为 true，则该消息即是 MQTT 中的保留消息（Retained Message）。MQTT 服务器会为每个主题存储最新一条保留消息，以方便消息发布后才上线的客户端在订阅主题时仍可以接收到该消息。  
如下图，当客户端订阅主题时，如果服务端存在该主题匹配的保留消息，则该保留消息将被立即发送给该客户端。

![图片](https://www.yuque.com/api/filetransfer/images?url=https%3A%2F%2Fimg2022.cnblogs.com%2Fblog%2F2F2F1775668-20221027183227763-1475238120.png%3Fx-oss-process%3Dimage%252Fwatermark%252Ctype_d3F5LW1pY3JvaGVp%252Csize_252Ctext_QWFyb242Zmw%253D%252Ccolor_FFFFFF%252Cshadow_252Ct_252Cg_se%252Cx_252Cy_10&sign=63b2b72da85021a3ffe3a6513f6cf3a956769e8ba4646bbdaf3dfdb0ca3c248a)

### 何时使用 MQTT 保留消息？  

发布订阅模式虽然能让消息的发布者与订阅者充分解耦，但也存在一个缺点，即订阅者无法主动向发布者请求消息。订阅者何时收到消息完全依赖于发布者何时发布消息，这在某些场景中就产生了不便。  
借助保留消息，新的订阅者能够立即获取最近的状态，而不需要等待无法预期的时间，例如：  
●智能家居设备的状态只有在变更时才会上报，但是控制端需要在上线后就能获取到设备的状态；  
●传感器上报数据的间隔太长，但是订阅者需要在订阅后立即获取到最新的数据；  
●传感器的版本号、序列号等不会经常变更的属性，可在上线后发布一条保留消息告知后续的所有订阅者。  
应用场景  
1重新连接MQTT服务时，不需要接收该主题最新消息，设置retained为false;  
2重新连接MQTT服务时，需要接收该主题最新消息，设置retained为true;  
#### 场景分析  

某个mqtt客户端A每小时向某个特定的topic发布一条消息，所有订阅这个topic的客户端将会收到该消息，这是正常流程，如果客户端A刚刚发布消息， 此时有一个新的客户端B订阅该topic，也就是“订阅”是在“发布”后，这个时候客户端B将接收不到该消息。  
Retain 功能就是为了解决这一问题，当客户端A发布小时时，将 retain标志置1，那么broker就会保存该消息，当有新的客户端订阅该topic时，会立刻将该条消息推送给客户端B。  
所以官方的协议中是这样介绍该功能：“如果客户端发给服务端的 PUBLISH 报文的保留（RETAIN）标志被设置为 1，服务端 必须存储这个应用消息和它的服务质量等级（QoS），以便它可以被分发给未来的主题名匹配的订阅者”  
#### Retain 功能特点  

1一个topic只能有1条Retain消息，新的Retain消息会覆盖旧的。  
2每当MQTT客户端连接到MQTT服务器并订阅了某个topic，如果该topic下有Retained消息，那么MQTT服务器会立即向客户端推送该条Retained消息。  
3删除一个Retain消息，可以向这个topic发布一个长度为0的 Retain 消息即可。  
### MQTT 保留消息的使用  

测试离线消息：  
1客户端 A 订阅主题 test 后断开连接。  
2客户端 B 发布消息到 test（QoS 1）。  
3客户端 A 重新连接后，会收到离线期间的消息。  
若要使用 MQTT 保留消息，只需在消息发布时将 Retained 状态设置为 true 即可。接下来我们以开源的跨平台 [MQTT 5.0 桌面客户端工具 - MQTT X](https://mqttx.app/zh) 为例，演示如何使用 MQTT 保留消息。  
打开 MQTT X 后如下所示，需点击 New Connection 按钮创建一个 MQTT 连接。  

![图片](https://www.yuque.com/api/filetransfer/images?url=https%3A%2F%2Fimg2022.cnblogs.com%2Fblog%2F2F2F1775668-20221027183214946-1374495249.png%3Fx-oss-process%3Dimage%252Fwatermark%252Ctype_d3F5LW1pY3JvaGVp%252Csize_252Ctext_QWFyb242Zmw%253D%252Ccolor_FFFFFF%252Cshadow_252Ct_252Cg_se%252Cx_252Cy_10&sign=433da98df6f1b072872833cf04748603245d9b4b153b261dc20f5f612c05fed2)

创建页面如下，我们只需填写一个连接名称（Name），其他参数保持默认。Host 将默认为 EMQX Cloud 提供的[公共 MQTT 服务器](https://www.emqx.com/zh/mqtt/public-mqtt5-broker)。连接参数填写完成后，点击右上角的 Connect 按钮创建 MQTT 连接。  

![图片](https://www.yuque.com/api/filetransfer/images?url=https%3A%2F%2Fimg2022.cnblogs.com%2Fblog%2F2F2F1775668-20221027183202051-1382208943.png%3Fx-oss-process%3Dimage%252Fwatermark%252Ctype_d3F5LW1pY3JvaGVp%252Csize_252Ctext_QWFyb242Zmw%253D%252Ccolor_FFFFFF%252Cshadow_252Ct_252Cg_se%252Cx_252Cy_10&sign=6fbe3ea90a01e6df98cc5d14fc2697238c063be37051b6ce58cbf14c1f6e35a8)

连接成功后将会看到连接名称旁边的状态为绿色。然后我们在右下角消息输入框向主题 sensor/t1 发送一条普通的消息。  

![图片](https://www.yuque.com/api/filetransfer/images?url=https%3A%2F%2Fimg2022.cnblogs.com%2Fblog%2F2F2F1775668-20221027183149694-1462078252.png%3Fx-oss-process%3Dimage%252Fwatermark%252Ctype_d3F5LW1pY3JvaGVp%252Csize_252Ctext_QWFyb242Zmw%253D%252Ccolor_FFFFFF%252Cshadow_252Ct_252Cg_se%252Cx_252Cy_10&sign=080e25485c32c07e0e80b7c49905b7ad2d5726c1a145be5b6b989a8c83e7ee4e)

接下来我们选中右下角的 Retain 标记，并向主题 sensor/t2 发送两条保留消息。  

![图片](https://www.yuque.com/api/filetransfer/images?url=https%3A%2F%2Fimg2022.cnblogs.com%2Fblog%2F2F2F1775668-20221027183138675-424218693.png%3Fx-oss-process%3Dimage%252Fwatermark%252Ctype_d3F5LW1pY3JvaGVp%252Csize_252Ctext_QWFyb242Zmw%253D%252Ccolor_FFFFFF%252Cshadow_252Ct_252Cg_se%252Cx_252Cy_10&sign=f3ba3b5d5d417f6252ff149b4621fd4e43f86dc258d4ff018c6b3104a3a69e9c)

然后点击页面中间的 New Subscription 按钮创建订阅。  

![图片](https://www.yuque.com/api/filetransfer/images?url=https%3A%2F%2Fimg2022.cnblogs.com%2Fblog%2F2F2F1775668-20221027183128287-608978292.png%3Fx-oss-process%3Dimage%252Fwatermark%252Ctype_d3F5LW1pY3JvaGVp%252Csize_252Ctext_QWFyb242Zmw%253D%252Ccolor_FFFFFF%252Cshadow_252Ct_252Cg_se%252Cx_252Cy_10&sign=380400b2f3d36b3c924181926fc96b5b78c5463a626424f08e0366d39fb3bbf3)

如下，我们订阅通配符主题 sensor/+，该通配符主题将会匹配主题 sensor/t1 及 sensor/t2。  
关于通配符主题的更多细节，请查看博客[通过案例理解 MQTT 主题与通配符](https://www.emqx.com/zh/blog/advanced-features-of-mqtt-topics)。  

![图片](https://www.yuque.com/api/filetransfer/images?url=https%3A%2F%2Fimg2022.cnblogs.com%2Fblog%2F2F2F1775668-20221027183114984-289573161.png%3Fx-oss-process%3Dimage%252Fwatermark%252Ctype_d3F5LW1pY3JvaGVp%252Csize_252Ctext_QWFyb242Zmw%253D%252Ccolor_FFFFFF%252Cshadow_252Ct_252Cg_se%252Cx_252Cy_10&sign=7207e674d4075f73a0399b4f76a64850217aed549ef03dfb9c9639a1f972ca83)

最后，我们将会看到该订阅能成功收到第二条保留消息，sensor/t1 的普通消息及 sensor/t2 的第一条保留消息都未收到。可见 MQTT 服务器只会为每个主题存储最新一条保留消息。  

![图片](https://www.yuque.com/api/filetransfer/images?url=https%3A%2F%2Fimg2022.cnblogs.com%2Fblog%2F2F2F1775668-20221027182606543-56023830.png%3Fx-oss-process%3Dimage%252Fwatermark%252Ctype_d3F5LW1pY3JvaGVp%252Csize_252Ctext_QWFyb242Zmw%253D%252Ccolor_FFFFFF%252Cshadow_252Ct_252Cg_se%252Cx_252Cy_10&sign=da517bdb4e50b94454f2ea0f285c4001b121d9573037af966f67e79bad430abd)

### 关于 MQTT 保留消息的 Q&A  

#### 如何判断一条消息是否是保留消息？  

当客户端订阅了有保留消息的主题后，即会收到该主题的保留消息，可通过消息中的保留标志位判断是否是保留消息。需要注意的是，在保留消息发布前订阅主题，将不会收到保留消息。需要待保留消息发布后，重新订阅该主题，才会收到保留消息。  
如下图，我们先订阅主题 sensor/t2，然后向该主题发布一条保留消息，该订阅会立即收到一条消息，但是该消息并不是保留消息。当我们删除该订阅，再次重新订阅 sensor/t2 主题时，立即收到了刚刚发布的保留消息。  

![图片](https://www.yuque.com/api/filetransfer/images?url=https%3A%2F%2Fimg2022.cnblogs.com%2Fblog%2F2F2F1775668-20221027182051754-329463287.png%3Fx-oss-process%3Dimage%252Fwatermark%252Ctype_d3F5LW1pY3JvaGVp%252Csize_252Ctext_QWFyb242Zmw%253D%252Ccolor_FFFFFF%252Cshadow_252Ct_252Cg_se%252Cx_252Cy_10&sign=3fbf6a2683a2e0e99752d867014639467117c3a120a746441f2b1d837fc77402)

#### 保留消息将保存多久？如何删除？  

服务器只会为每个主题保存最新一条保留消息，保留消息的保存时间与服务器的设置有关。若服务器设置保留消息存储在内存，则 MQTT 服务器重启后消息即会丢失；若存储在磁盘，则服务器重启后保留消息仍然存在。  
保留消息虽然存储在服务端中，但它并不属于会话的一部分。也就是说，即便发布这个保留消息的会话已结束，保留消息也不会被删除。删除保留消息有以下几种方式：  
●客户端往某个主题发送一个 Payload 为空的保留消息，服务端就会删除这个主题下的保留消息；  
●在 MQTT 服务器上删除，比如 EMQX MQTT 服务器提供了在 Dashboard 上删除保留消息的功能；  
●MQTT 5.0 新增了消息过期间隔属性，发布时可使用该属性设置消息的过期时间，不管消息是否为保留消息，都将会在过期时间后自动被删除。  
### EMQX 中的 MQTT 保留消息  

[EMQX](https://www.emqx.com/zh/products/emqx) 是一款全球下载量超千万的大规模分布式物联网 MQTT 服务器，于 2013 年在 GitHub 发布开源版本。  
不久前，[EMQX 发布了 5.0 版本](https://www.emqx.com/zh/blog/emqx-v-5-0-released)，该版本通过一个 23 节点的集群达成了 1 亿 MQTT 连接 + 每秒 100 万消息吞吐，这使得 EMQX 5.0 成为目前为止全球最具扩展性的 MQTT 服务器。  
EMQX 5.0 支持在内置的 [Dashboard](https://www.emqx.com/zh/blog/an-easy-to-use-and-observable-mqtt-dashboard) 中查看、设置保留消息。感兴趣的读者可通过如下 Docker 命令安装 EMQX 5.0 开源版进行体验。  

```

docker run -d --name emqx -p 1883:1883 -p 8083:8083 -p 8084:8084 -p 8883:8883 -p 18083:18083 emqx/emqx:latest  

```

EMQX 安装成功后，使用浏览器访问 http://127.0.0.1:18083/ 即可体验 EMQX 5.0 全新 Dashboard。  
默认用户名为 admin，密码为 public  
登录成功后，可在左侧菜单 System -> Settings 中修改显示语言为中文。如下图，可点击功能配置->MQTT 菜单查看已保留的消息列表，同时也可以查看保留消息的 Payload 或者删除某条保留消息。  

![图片](https://www.yuque.com/api/filetransfer/images?url=https%3A%2F%2Fimg2022.cnblogs.com%2Fblog%2F2F2F1775668-20221027181723009-1171465454.png%3Fx-oss-process%3Dimage%252Fwatermark%252Ctype_d3F5LW1pY3JvaGVp%252Csize_252Ctext_QWFyb242Zmw%253D%252Ccolor_FFFFFF%252Cshadow_252Ct_252Cg_se%252Cx_252Cy_10&sign=79b61d4e77880044f209495d52de49ffc655244e2abd236a2858558079b37d3d)

点击保留消息下的设置菜单，可看到 EMQX 支持在 Dashboard 中设置保留消息的存储类型（内存或磁盘）、最大保留消息数、保留消息有效期等参数，点击保存后所有更改将会立即生效。  

![图片](https://www.yuque.com/api/filetransfer/images?url=https%3A%2F%2Fimg2022.cnblogs.com%2Fblog%2F2F2F1775668-20221027181702523-690727721.png%3Fx-oss-process%3Dimage%252Fwatermark%252Ctype_d3F5LW1pY3JvaGVp%252Csize_252Ctext_QWFyb242Zmw%253D%252Ccolor_FFFFFF%252Cshadow_252Ct_252Cg_se%252Cx_252Cy_10&sign=64f6302fb90df0089131297cb8770d48e6ad48f43c76aa9f737b329671e37916)

### 结语  

本文对 MQTT 保留消息进行了介绍及使用演示，用户可以参考本文更好地利用 MQTT 保留消息解决订阅后无法立即获取最近数据的问题。除此之外，MQTT 协议还具备更多实用特性，读者可查看 EMQ 提供的 [MQTT 入门与进阶](https://www.emqx.com/zh/mqtt)系列文章进行深入了解，探索 MQTT 的更多高级应用，开启 MQTT 应用及服务开发。  
版权声明： 本文为 EMQ 原创，转载请注明出处。  
原文链接：[https://www.emqx.com/zh/blog/mqtt5-features-retain-message](https://www.emqx.com/zh/blog/mqtt5-features-retain-message)  
## 持久会话（clean_session）  

[MQTT 保留消息是什么？如何使用？ | EMQ](https://www.emqx.com/zh/blog/mqtt5-features-retain-message)  
[MQTT 持久会话与 Clean Session 详解## MQTT 持久会话 不稳定的网络及有限的硬件资源是物联网应用 - 掘金](https://juejin.cn/post/7197687219581730877)  
[MQTT 持久会话 vs. Clean Session内幕一网打尽 - 简书](https://www.jianshu.com/p/b86bfe168d1a)  
不稳定的网络及有限的硬件资源是物联网应用需要面对的两大难题，MQTT 客户端与服务器的连接可能随时会因为网络波动及资源限制而异常断开。为了解决网络连接断开对通信造成的影响，MQTT 协议提供了持久会话功能。  
MQTT 客户端在发起到服务器的连接时，可以设置是否创建一个持久会话。持久会话会保存一些重要的数据，以使会话能在多个网络连接中继续。  
持久会话主要有以下三个作用：  
●避免因网络中断导致需要反复订阅带来的额外开销。  
●避免错过离线期间的消息。  
●确保 QoS 1 和 QoS 2 的消息质量保证不被网络中断影响。  
### 简介  

#### 1.持久会话（cleanSession=false）的核心作用  

●保存客户端与 Broker 的会话状态（订阅关系、未确认的 QoS 1/2 消息）；  
●设备重连后，Broker 会推送离线期间的消息，确保数据不丢失。  
#### 2.持久会话的弊端  

●设备集中上线时，持久会话会触发「会话恢复流程」（Broker 查询会话状态、推送离线消息），增加 Broker 的 CPU 和内存开销，可能加剧连接拥堵；  
●若设备无离线消息需求，持久会话的额外开销完全无用。  
#### 3.推荐的 MQTT 会话配置  

●设备端：cleanSession=true（清洁会话），减少 Broker 会话存储压力，连接建立更快；  
●数据可靠性保障：通过「QoS 1」（至少一次送达）+「Broker 消息持久化」替代持久会话，既保证数据不丢失，又避免会话恢复开销；  
●离线消息场景：仅对关键设备启用持久会话，并限制离线消息缓存时长（如 1 小时），避免 Broker 存储溢出。  
### MQTT Clean Session 的使用  

Clean Session 是用来控制会话状态生命周期的标志位，为 true 时表示创建一个新的会话，在客户端断开连接时，会话将自动销毁。为 false 时表示创建一个持久会话，在客户端断开连接后会话仍然保持，直到会话超时注销。  
注意： 持久会话能被恢复的前提是客户端使用固定的 Client ID 再次连接，如果 Client ID 是动态的，那么连接成功后将会创建一个新的持久会话。  
如下为[开源 MQTT 服务器 EMQX](https://link.juejin.cn?target=https%3A%2F%2Fwww.emqx.io%2Fzh) 的 Dashboard，可以看到图中的连接虽然是断开状态，但是因为它是持久会话，所以仍然能被查看到，并且可以在 Dashboard 中手动清除该会话。  

同时，EMQX 也支持在 Dashboard 中设置 Session 相关参数。  

MQTT 3.1.1 没有规定持久会话应该在什么时候过期，如果仅从协议层面理解的话，这个持久会话应该永久存在。但在实际场景中这并不现实，因为它会非常占用服务端的资源，所以服务端通常不会完全遵循协议来实现，而是向用户提供一个全局配置来限制会话的过期时间。  
比如 EMQ 提供的 [免费的公共 MQTT 服务器](https://link.juejin.cn?target=https%3A%2F%2Fwww.emqx.com%2Fzh%2Fmqtt%2Fpublic-mqtt5-broker) 设置的会话过期时间为 5 分钟，最大消息数为 1000 条，且不保存 QoS 0 消息。  
接下来我们使用开源的跨平台 [MQTT 5.0 桌面客户端工具 - MQTT X](https://link.juejin.cn?target=https%3A%2F%2Fmqttx.app%2Fzh) 演示 Clean Session 的使用。  
打开 MQTT X 后如下所示，点击 New Connection 按钮创建一个 [MQTT 连接](https://link.juejin.cn?target=https%3A%2F%2Fwww.emqx.com%2Fzh%2Fblog%2Fhow-to-set-parameters-when-establishing-an-mqtt-connection)。  

创建一个名为 MQTT_V3 的连接，Clean Session 为关闭状态（即为 false），MQTT 版本选择 3.1.1，然后点击右上角的 Connect 按钮。  
连接的服务器默认为 EMQ 提供的 [免费的公共 MQTT 服务器](https://link.juejin.cn?target=https%3A%2F%2Fwww.emqx.com%2Fzh%2Fmqtt%2Fpublic-mqtt5-broker)。  

连接成功后订阅 clean_session_false 主题，且 QoS 设置为 1。  

订阅成功后，点击右上角的断开连接按钮。然后，创建一个名为 MQTT_V3_Publish 的连接，MQTT 版本同样设置为 3.1.1，连接成功后向 clean_session_false 主题发布两条 QoS 1 消息。  

然后选中 MQTT_V3 连接，点击连接按钮连接至服务器，将会成功接收到两条离线期间的消息。  

### MQTT 5.0 中的会话改进  

MQTT 5.0 中将 Clean Session 拆分成了 Clean Start 与 Session Expiry Interval。Clean Start 用于指定连接时是创建一个全新的会话还是尝试复用一个已存在的会话，Session Expiry Interval 用于指定网络连接断开后会话的过期时间。  
Clean Start 为 true 时表示必须丢弃任何已存在的会话，并创建一个全新的会话；为 false 时表示必须使用与 Client ID 关联的会话来恢复与客户端的通信（除非会话不存在）。  
Session Expiry Interval 解决了 MQTT 3.1.1 中持久会话永久存在造成的服务器资源浪费问题。设置为 0 或未设置，表示断开连接时会话即到期；设置为大于 0 的数值，则表示会话在网络连接关闭后会保持多少秒；设置为 0xFFFFFFFF 表示会话永远不会过期。  
更多细节可查看博客：[Clean Start 与 Session Expiry Interval](https://link.juejin.cn?target=https%3A%2F%2Fwww.emqx.com%2Fzh%2Fblog%2Fmqtt5-new-feature-clean-start-and-session-expiry-interval)。  
### 关于 MQTT 会话的 Q&A  

#### 当会话结束后，保留消息还存在么？  

[MQTT 保留消息](https://link.juejin.cn?target=https%3A%2F%2Fwww.emqx.com%2Fzh%2Fblog%2Fmqtt5-features-retain-message)不是会话状态的一部分，它们不会在会话结束时被删除。  
#### 客户端如何知道当前会话是被恢复的会话？  

MQTT 协议从 v3.1.1 开始，就为 CONNACK 报文设计了 Session Present 字段。当服务器返回的该字段值为 1 时，表示当前连接将会复用服务器保存的会话。客户端可通过该字段值决定在连接成功后是否需要重新订阅。  
#### 使用持久会话时有哪些建议？  

●不能使用动态 Client ID，需要保证客户端每次连接的 Client ID 都是固定的。  
●根据服务器性能、网络状况、客户端类型等合理评估会话过期时间。设置过长会占用更多的服务端资源，设置过短会导致未重连成功会话就失效。  
●当客户端确定不再需要会话时，可使用 Clean Session 为 true 进行重连，重连成功后再断开连接。如果是 MQTT 5.0 则可在断开连接时直接设置 Session Expiry Interval 为 0，表示连接断开后会话即失效。  
## MQTT 5.0 的消息过期间隔特性  

原文地址 [如何用 MQTT 5.0 的消息过期间隔特性提升应用性能 - MQTT中文站](https://www.mqtt.cn/1887.html)  
在实时通讯协议 MQTT 的最新版本 5.0 中，引入了一个称为 “消息过期间隔”（Message Expiry Interval）的新特性。这一功能允许消息发布者为每条消息设置一个有效期限。如果消息在服务器上停留的时间超过了这个期限，它就不会被分发给任何订阅者。这意味着，当消息的内容不再相关或有用时，它会被自动清理，从而优化了网络资源的使用和提高了通信的效率。  
### 什么是消息过期间隔？  

消息过期间隔定义了一个消息从被发布起，到它变得不再可分发为止的时间长度。在 MQTT 协议中，这是通过在消息中设置一个特定的 “过期间隔” 值来实现的。默认情况下，消息不设置过期间隔，即视为永不过期。但在某些场景下，设定一个合理的过期时间可以显著提高数据传输的效率和实时性。  
### 应用场景  

#### 时间敏感的信息  

例如，一个促销活动仅在接下来的两小时内有效。如果消费者在活动结束后才收到这个消息，那么这个消息就失去了它的价值。  
#### 状态更新  

考虑到实时交通信息的更新，如道路拥堵情况。随着时间的推移和交通流量的变化，过时的拥堵信息就不再有参考价值。  
#### 自动管理保留消息  

在 MQTT 中，保留消息功能允许新的订阅者接收到他们订阅主题的最后一条消息。通过设置消息的过期间隔，可以避免过时的保留消息长时间占用服务器资源。  
### 实例演示  

假设有一个联网汽车应用场景，其中包括发送实时交通状况和路口信号灯配时建议。通过设置消息的过期间隔，可以确保车辆只接收到当前位置附近的、时效性强的信息。  
#### 设置消息过期间隔的操作步骤  

要演示消息过期间隔的代码示例，我们可以使用 Python 和 paho-[mqtt](https://www.mqtt.cn/tag/29 "View all posts in mqtt") 库，这是一个常用于 MQTT 客户端开发的库。下面的示例将分为两部分：发布者（Publisher）和订阅者（Subscriber）。  
### 安装 paho-[mqtt](https://www.mqtt.cn/tag/29)  

首先，确保你已经安装了 paho-[mqtt](https://www.mqtt.cn/tag/29 "View all posts in mqtt")。如果没有安装，可以通过 pip 安装：  

```

[空代码块]  

```

### 发布者（Publisher）  

发布者将发送两条消息，一条设置了 5 秒的过期间隔，另一条设置了 60 秒。  

```

[空代码块]  

```

### 订阅者（Subscriber）  

订阅者订阅同一个主题，并处理接收到的消息。  

```

[空代码块]  

```

### 运行示例  

1运行订阅者代码：首先启动订阅者客户端，让它在后台运行并监听指定主题的消息。  
2运行发布者代码：然后运行发布者代码，发送两条带有不同过期间隔的消息。  
你将看到订阅者只接收到在其过期间隔内的消息。如果你在发布后立刻运行订阅者，两条消息都可能被接收。但如果在 5 秒后再启动订阅者，只有设置了 60 秒过期间隔的消息会被接收，因为另一条消息已经过期。  
这个演示说明了如何在 MQTT 5.0 中使用消息过期间隔功能，以及它如何影响消息的传递和接收。  
通过这个例子，我们可以清楚地看到消息过期间隔如何在实际应用中起作用，以确保信息的实时性和相关性。  
### 总结  

消息过期间隔是 MQTT 5.0 提供的一个非常实用的特性，它帮助开发者和企业确保消息的时效性，同时减轻服务器的负担，并优化资源使用。无论是在实时交通信息、促销活动通知，还是在智能家居和工业物联网应用中，适当使用消息过期间隔都能带来显著的效益。  
在设计和实现基于 MQTT 协议的通信系统时，合理利用消息过期间隔不仅可以提高信息传输的效率，还能增强用户体验，确保用户及时获取最重要和最相关的信息。同时，它还减少了网络带宽的浪费，优化了服务器存储资源的使用，使得系统更加高效和可靠。  
## 遗嘱消息（Lass Will and Testament）  

原文地址 [什么是 MQTT 遗嘱消息？MQTT 遗嘱消息如何运作？ - MQTT中文站](https://www.mqtt.cn/1883.html)  
[MQTT 遗嘱消息（Will Message）的使用 - 知乎](https://zhuanlan.zhihu.com/p/459926792)  
[从断开连接到实时通知：探索MQTT遗嘱消息的功能 - MQTT中文站](https://www.mqtt.cn/629.html)  
[MQTT的遗嘱机制 - vx_guanchaoguo0 - 博客园](https://www.cnblogs.com/guanchaoguo/p/18678034)  
MQTT 遗嘱消息（Last Will and Testament，简称 LWT）是 MQTT 协议中的一个特性，允许客户端在建立连接时向服务器注册一个遗嘱消息。这个消息将在客户端异常断开连接时由服务器自动发布，通知其他客户端该客户端已离线。这个功能对于监控设备状态、实现设备断线通知等场景非常有用。  
### MQTT 遗嘱消息的工作原理  

1客户端连接时指定遗嘱消息： 客户端在与 MQTT 服务器（Broker）建立连接时，可以指定一个遗嘱消息。这包括遗嘱消息的主题（Will Topic）、消息内容（Will Message）、消息质量（QoS）和是否保留（Retain）等信息。  
2服务器存储遗嘱消息： MQTT 服务器接收到遗嘱消息后，会将其存储，但不立即发布。只有在满足特定条件时，服务器才会发布这个遗嘱消息。  
3客户端异常断开连接： 如果客户端因网络故障、设备故障或其他原因异常断开连接（不包括正常发送 DISCONNECT 报文的情况），MQTT 服务器将判断客户端 “死亡”，并自动发布之前存储的遗嘱消息。  
4其他客户端接收遗嘱消息： 其他订阅了遗嘱消息主题的客户端将接收到这个遗嘱消息，从而得知特定客户端已经断开连接。  

先说原理  
1MQTT协议中的遗嘱机制允许客户端在连接时指定一条遗嘱消息，  
2当客户端意外断线时，服务端会发布这条遗嘱消息只要对订阅这个主题都可以收到这个消息  
3遗嘱机制的目的是在客户端意外断线时通知其他设备客户端的状态变化。  
### 概念  

#### lastWillTopic – 遗嘱主题  

●遗嘱消息和普通MQTT消息很相似，也有主题和正文内容。  
●lastWillTopic的作用正是告知服务端，本客户端的遗嘱主题是什么。  
●只有那些订阅了这一遗嘱主题的客户端才会收到本客户端的遗嘱消息。  
●以上图为例，此遗嘱主题为”hans/will”。也就是说，只有订阅了主题”hans/will”的客户端，才会收到这台客户端的遗嘱消息。  
#### lastWillMessage – 遗嘱消息  

●遗嘱消息定义了遗嘱消息内容。在本示例中，那些订阅了主题”hans/will”的客户端会在客户端意外断线时，收到服务端发布的“unexpected exit”。  
#### lastWillQoS – 遗嘱QoS  

●对于遗嘱消息来说，同样可以使用服务质量来控制遗嘱消息的传递和接收。  
●这里的服务质量与普通MQTT消息的服务质量是一样的概念。  
●也可以设置为0、1、2。对于不同的服务质量级别，服务端会使用不同的服务质量来发布遗嘱消息。  
#### lastWillRetain – 遗嘱保留  

●QoS 0【At most once】：最多一次，快速但不可靠。  
●QoS 1【At least once】：至少一次，保证消息可靠传递，但可能重复。  
●QoS 2【Exactly once】：只有一次，确保消息准确且仅一次传递，最可靠但效率最低。  
●遗嘱消息也可以设置为保留消息，遗嘱保留用于设置遗嘱消息是否需要进行保留处理。服务端会根据此处内容，对遗嘱消息进行相应的保留与否处理。  
### 具体操作流程  

●在使用MQTT遗嘱时，我们建议您通过以下方法让设备的MQTT遗嘱机制可以更好的发挥作用。  
●假设我们现在有一台MQTT客户端。它的client id是 client-1。它的遗嘱主题是“client-1-will”  
●当client-1连接服务端时，CONNECT报文中的遗嘱消息是“offline”。并且它的遗嘱保留设置为true。  
●当client-1成功连接服务端后，立即向遗嘱主题“client-1-will”发布一条消息“online”。同时在发布此消息时，保留标志设置为true。这样，只要client-1在线，那么任何设备一订阅“client-1-will”就能收到设备在线的消息“online”。  
●如果client-1发生意外离线。那么任何设备一订阅“client-1-will”就会收到设备离线的消息”offline”。  
●如果client-1恢复连接，那么它会将遗嘱主题“client-1-will”的保留消息更改为“online”，这样任何设备一订阅“client-1-will”就能收到设备在线的消息“online”。  
### 使用遗嘱消息的场景  

●设备状态监控： 在物联网应用中，可以利用遗嘱消息监控设备的在线状态。如果设备异常断开，遗嘱消息可以通知监控系统或其他设备，采取相应措施。  
●用户在线状态通知： 在即时通讯应用中，遗嘱消息可以用来通知其他用户某个用户已经离线。  
### 示例代码  

以下是使用paho-[mqtt](https://www.mqtt.cn/tag/29 "View all posts in mqtt")客户端库（Python）设置遗嘱消息的示例：  
首先，确保安装了paho-[mqtt](https://www.mqtt.cn/tag/29 "View all posts in mqtt")：  

```

[空代码块]  

```

然后，创建一个 MQTT 客户端，设置遗嘱消息，并连接到 MQTT 服务器：  

```

[空代码块]  

```

在这个示例中，如果客户端异常断开连接，MQTT 服务器将自动发布遗嘱消息到device/status主题，消息内容为Device offline。其他订阅了这个主题的客户端将接收到这个消息，知道该设备已离线。  
### 小结  

MQTT 遗嘱消息是一个强大的功能，它为客户端提供了一种机制，在无法正常断开连接时通知其他客户端或系统。这在需要高可靠性的物联网应用中尤其有用，可以帮助系统及时响应设备故障或网络问题。  
# python之paho-mqtt库  

Paho 是一个开源的 MQTT 客户端项目，提供多种语言的 MQTT 客户端实现，包括 C、C++、C#、Java、Python、JavaScript 等，完全支持 MQTT v3.1 和 v3.1.1 。Paho Python Client 是它的 Python 语言版本，支持 Python 2.7 和 3.x 。  
更多特性可以查看 [http://www.eclipse.org/paho/clients/python/](http://www.eclipse.org/paho/clients/python/)  
源码和文档在 [https://github.com/eclipse/paho.mqtt.python](https://github.com/eclipse/paho.mqtt.python)  
## 安装paho-mqtt  

python 下载 mqtt 包：pip install paho-mqtt  
或者下载源码：  

```

[空代码块]  

```

下面是一个简单的例子，连接一个 borker ，订阅系统默认话题，获取 broker 的版本号：  

```

[空代码块]  

```

## mqtt 简单应用 (实例)  

### 发布端（publish）  

```

[空代码块]  

```

此处也可以将 mqtt 方法封装为一个类，使用会更方便一些  
参数解释:  
●keepalive \=> 心跳间隔，单位是秒，如果 broker 和 client 在这段时间内没有任何通讯，client 会给 broker 发送一个 ping 消息  
●retain => 如果设为 Ture ，这条消息会被设为保留消息  
●payload => 消息内容，字符串类型，如果设为 None ，会发送一条长度为 0 消息。如果设置了 int 或者 3. float 类型的值，会当做字符串发送，如果你想发送真正的 int 或者 float 值，需要用 struct.pack() 生成消息, mqtt 的 publish 只支持 None, string, int, float 类型的数据, 如果需要发送 json 类型数据可以通过 json.dumps() 将数据进行转换后在发送, 接收端在 on_message() 回调函数中通过 json.loads() 将数据解析就可以了  
●topic \=> 这条消息所属的话题  
●qos => 消息的安全等级 [Qos 详细介绍](https://blog.csdn.net/programguo/article/details/100125177)  
○qos=0 QoS0，At most once，至多一次；  
■QoS0 代表，Sender 发送的一条消息，Receiver 最多能收到一次，也就是说 Sender 尽力向 Receiver 发送消息，如果发送失败，也就算了；  
○qos=1 QoS1，At least once，至少一次；  
■QoS1 代表，Sender 发送的一条消息，Receiver 至少能收到一次，也就是说 Sender 向 Receiver 发送消息，如果发送失败，会继续重试，直到 Receiver 收到消息为止，但是因为重传的原因，Receiver 有可能会收到重复的消息；  
○qos=2 QoS2，Exactly once，确保只有一次  
■QoS1 代表，Sender 发送的一条消息，Receiver 至少能收到一次，也就是说 Sender 向 Receiver 发送消息，如果发送失败，会继续重试，直到 Receiver 收到消息为止，但是因为重传的原因，Receiver 有可能会收到重复的消息  
qos 安全等级需要注意的点：  
1python 中下载的 paho-mqtt 包中，默认 qos=0。  
2不论是 sub 还是 pub 都需要指定 qos 安全等级。  
apub 指定的 qos 是服务器肯定按此规则接收，但是最终订阅者不一定。  
bsub 指定的 qos 表示订阅者可以接收的最高消息等级，也就是可能收到更低等级的消息  
c也就是服务器只会按 pub 和 sub 两者 qos 等级最小的那个 qos 规则来发送消息  
### 订阅端（subscribe）  

```

[空代码块]  

```

订阅服务 client.subscribe("mqtt11", qos=0) 也可以将改订阅放在 on_connect() 回调函数中，程序在建立连接成功后首先后执行 on_connect() , 可将整个订阅端封装为一个类使用  
参数解释:  
●keepalive => 心跳间隔，单位是秒，如果 broker 和 client 在这段时间内没有任何通讯，client 会给 broker 发送一个 ping 消息  
●loop_forever() => 该函数用于保持永久连接，阻塞式，可结合多线程或多进程的方式使用  
注意： 同一个 mqtt客户端即可以是发布端，也可以是订阅端，也可以订阅自己发布的内容，这里之所以分开是为了看起来更直观一些  
mqtt 方法可以封装为如下类  

```

[空代码块]  

```

一个简单的发布订阅就完成了, 通过回调函数, 可以对相应的值进行操作  
## API详解  

[Python paho-mqtt 模块使用（转） - 乖乖楠 - 博客园](https://www.cnblogs.com/lnn123/p/10837754.html)  
[Python 客户端类库之paho-mqtt学习总结 - 授客 - 博客园](https://www.cnblogs.com/shouke/p/18417803)  
### 回调函数(Callbacks)  

回调函数的应用是非常有必要的，回调函数有很多种，我们可能根据不同的业务场景采用不同的回调函数进行数据的处理，可以达到代码的高可用，减少代码的冗余。  
回调函数只是业务程序的中转站  
在这里有一个特别要注意的点，回调函数可以获取到请求的响应数据，但回调函数并不是适合作为对响应结果进行处理的地方  
举个例子：  
发布端发起请求，通过 topic 将请求传入 mqtt，订阅端通过同一个 topic 订阅到该发布端的 data 数据，然后再 on_message() 中做业务处理，假定该业务处理需要 10 秒，在第 1 秒的时候，发布端又发布了新的内容，这是订阅端依旧通过 topic 订阅到了该内容，想要在 on_message() 中做业务处理，但是上一个业务并没有处理完成，程序处于堵塞的状态，直到 10 秒结束后，新订阅到的内容才会被处理。长期以此的话，就会在业务上造成延迟，如果业务需求对消息的实时性要求很高的话，那这样的处理方式就不可取了。  
所以说，回调函数并不适合做业务处理，正确的做法应该是将订阅到的数据通过回调函数转给别的处理程序去执行，做到分模块分节点的执行。或者根据不同的业务需求，分类分节点的订阅，通过多个线程多个订阅的方式，做到业务上的互不干扰。  
个人理解：回调函数类似于中间件的效果，mqtt 在运行的过程中，会依次访问回调函数，将当前回调函数所需的一些参数信息传给回调函数，前提是你的回调函数在建立连接时被引用注册了才行。  
回调函数用于处理从MQTT代理返回的数据  
要使用回调需要先定义回调函数然后将其指派给客户端实例（client）。  
例如：  

```

[空代码块]  

```

所有的回调函数都有client和userdata参数。  
●client是调用回调的客户端实例；  
●userdata可以是任何类型的用户数据，可以在创建新客户端实例时设置或者使用user_data_set(userdata)设置。  
#### 连接回调 on_connect  

连接主题 (成功, 失败) 都会调用此函数  
on_connect(client, userdata, flags, rc, reasonCode, properties)  
参数解释: (回调中重复参数不做重复解释)  
●client：此回调的客户机实例  
●userdata：在 Client() 或 userdata_set() 中设置的私有用户数据  
●flags：代理发送的响应标志（字典类型）  
●rc：连接结果（表示是否连接成功）  
○0: 连接成功  
○1: 连接被拒绝 - 协议版本不正确  
○2: 连接被拒绝 - 客户端标识符无效  
○3: 连接被拒绝 - 服务器不可用  
○4: 连接被拒绝 - 用户名或密码错误  
○5: 连接被拒绝 - 未授权  
○6-255: 当前未使用。  
●reasonCode：mqttv5.0 原因码：reasonCode 类的实例。  
●properties：从代理返回的 mqttv5.0 属性  
(对于 MQTT v3.1 和 v3.1.1，未提供属性，但用于兼容性；对于 mqttv5.0，官方建议添加 properties=None)  
#### 订阅回调 on_subscribe  

当代理响应订阅请求时被调用。  
on_subscribe(client，userdata，mid，grated_qos，properties=None)  
参数解释: (回调中重复参数不做重复解释)  
●mid：匹配从相应的 subscribe() 调用  
●grated_qos：一个整数列表，它提供了代理为每个不同的订阅请求授予的QoS级别。  
#### 消息回调 on_message  

on_message(client, userdata, message)  
参数解释: (回调中重复参数不做重复解释)  
●message：MQTTMessage 的实例。这是一个包含成员主题、负载、qos 和保留的类  
●使用 message_callback_add() 定义将调用的多个回调, 用于特定主题筛选器  
注意：mqtt 中 on_message 可以返回订阅到的信息，on_message 是系统的默认订阅回调，如果没有自定义消息回调 message_callback_add(sub, callback) ，则所有的订阅接收到的数据都会被 on_message 回调函数接收  
如果数据量过大，或者解析数据耗时时建议使用 message_callback_add(sub, callback)方法单独处理  
#### 注册特定主题消息回调 message_callback_add() --> 主题筛选器  

message_callback_add(sub, callback)  
参数解释: (回调中重复参数不做重复解释)  
●sub：sub 即 subscribe， 也就是 client.subscribe() 方法中的订阅的 topic，可以是通配符匹配订阅  
●callback：自定义回调函数，回调参数与 on_message() 相同即可，参数的意义也是一样的。举个例子如下  

```

[空代码块]  

```

注意： callback 一定要初始化绑定一下，即 client.username_message = username_message  
在消息回调中，message_callback_add() 的优先级要高于 on_message 默认回调的，匹配 "sub" 的 message 将传递给 "callback", 任何不匹配的 message 将传递到默认的 on_message 回调。  
这里建议多次调用不同的 "sub" 来定义多个主题特定回调  
若订阅的主题很多，且数据传输的频率很快，如果不使用message_callback_add() 实现特定主题回调，进行单独处理数据的话，则所有的数据都会进入 on_message 系统默认回调中，则可能会产生数据拥堵，on_message 处理不过来的现象，最严重的后果将造成消息堵塞，消息延迟  
#### 删除注册的特定回调 message_callback_remove()  

message_callback_remove(sub)  
删除以前注册过的回调，与 message_callback_add() 相对应  
#### 消息发布回调 on_publish  

当使用使用publish()发送的消息已经传输到代理时被调用。  
on_publish(client, userdata, mid)  
参数解释: (回调中重复参数不做重复解释)  
●mid：mid变量与从相应的publish()返回的mid变量匹配，以允许跟踪传出的消息。  
○对于 QoS 级别为 1 和 2 的消息，这意味着已经完成了与代理的握手。  
○对于 QoS 0，这只意味着消息离开了客户端。  
这个回调很重要，因为即使 publish() 调用返回成功，并不总是意味着消息已发送  
#### 取消订阅 on_unsubscribe  

当代理响应取消订阅请求时调用。  
on_unsubscribe(client, userdata, mid, properties, reasonCodes)  
参数解释: (回调中重复参数不做重复解释)  
●mid：匹配从相应的 unsubscribe() 调用调用返回的中间变量。  
#### 断开连接回调 ondisconnect  

当与代理断开连接时调用  
on_disconnect(client, userdata, rc, reasonCode, properties)  
参数解释: (回调中重复参数不做重复解释)  
●rc：断开连接的结果  
○disconnect()被调用时rc=MQTT_ERR_SUCCESS（0）；rc等于其他值表示连接被意外断开，例如可能出现网络错误。  
#### 套接子打开回调 on_scoket_open  

on_socket_open(client, userdata, socket)  
参数解释: (回调中重复参数不做重复解释)  
●socket：刚打开的 socket  
#### 套接子关闭回调 on_socket_close  

on_socket_close(client, userdata, socket)  
参数解释: (回调中重复参数不做重复解释)  
#### 套接子写入回调 on_socket_register  

on_socket_register_write(client, userdata, socket)  
参数解释: (回调中重复参数不做重复解释)  
●socket：写入的套接字  
#### 套接子注销写入回调 on_socket_unregister  

on_socket_unregister_write(client, userdata, socket)  
参数解释: (回调中重复参数不做重复解释)  
还有一些系统默认的回调函数，就不在次一一列出了，感兴趣的可以去看看 paho.mqtt.client 包的源码 (paho.mqtt 包中的 client 类)  
#### on_log()  

当客户端有日志信息时调用  
on_log(client, userdata, level, buf):  
●level变量给出了消息的严重性，并且将是MQTT_LOG_INFO，MQTT_LOG_NOTICE，MQTT_LOG_WARNING，MQTT_LOG_ERR和MQTT_LOG_DEBUG中的一个。  
●buf变量用于存储信息。  
### 客户端方法  

#### 1\. 构造函数 Client（）  

```

[空代码块]  

```

| 参数 | 含义 |
| --- | --- |
| 参数 | 含义 |
| client_id | 连接到代理时使用的唯一客户端 ID 字符串。 如果 client_id 长度为零或无，则会随机生成一个。 在这种情况下，clean_session 参数必须为 True。 |
| clean_session | 一个决定客户端类型的布尔值。 如果为 True，那么代理将在其断开连接时删除有关此客户端的所有信息；如果为 False，则客户端是持久客户端，当客户端断开连接时，订阅信息和排队消息将被保留。 |
| userdata | 用户定义的任何类型的数据作为 userdata 参数传递给回调函数。 它可能会在稍后使用 user_data_set() 函数进行更新。 |
| protocol | 用于此客户端的 MQTT 协议的版本。 可以是 MQTTv3.1 或 MQTTv3.11。 |
| transport | 设置为 "websockets" 通过 WebSockets 发送 MQTT。 保留默认的 "tcp" 使用原始 TCP。 |

示例：  

```

[空代码块]  

```

#### 2.reinitialise()  

```

[空代码块]  

```

reinitialise() 函数将客户端重置为其开始状态，就像它刚刚创建一样。 它采用与 Client() 构造函数相同的参数。  
示例：  

```

[空代码块]  

```

#### 3\. 选项函数  

这些函数表示可以在客户端上设置以修改其行为的选项。 在大多数情况下，这必须在连接到代理之前完成。  
##### （1）max_inflight_messages_set()  

```

[空代码块]  

```

设置 QoS> 0 的消息的最大数量，该消息一次可以部分通过其网络流量。默认为 20. 增加此值将消耗更多内存，但可以增加吞吐量。  
##### （2）max_queued_messages_set()  

```

[空代码块]  

```

设置传出消息队列中可等待处理的具有 QoS> 0 的传出消息的最大数量。默认为 0 表示无限制。 当队列已满时，任何其他传出的消息都将被丢弃。  
##### （3）message_retry_set(）  

```

[空代码块]  

```

如果代理没有响应，设置在重发 QoS> 0 的消息之前以秒为单位的时间。默认设置为 5 秒，通常不需要更改。  
##### （4）ws_set_options()  

```

[空代码块]  

```

设置 websocket 连接选项。 只有在 transport="websockets" 被传入Client()构造函数时才会使用这些选项。  
| 参数 | 含义 |
| --- | --- |
| 参数 | 含义 |
| path | 代理使用的 mqtt 路径 |
| headers | 可以是一个字典，指定应该附加到标准 websocket 头部的额外头部列表，也可以是可调用的正常 websocket 头部并返回带有一组头部以连接到代理的新字典。 |

必须在调用 connect() 之前调用。  
##### （4）tls_set()  

```

[空代码块]  

```

配置网络加密和身份验证选项。 启用 SSL / TLS 支持。  
| 参数 | 含义 |
| --- | --- |
| 参数 | 含义 |
| ca_certs | 证书颁发机构证书文件的字符串路径，该证书文件将被视为受此客户端信任。 |
| certfile, keyfile | 分别指向 PEM 编码的客户端证书和私钥的字符串。 如果这些参数不是 None，那么它们将用作基于 TLS 的身份验证的客户端信息。 对此功能的支持取决于代理。 |
| cert_reqs | 定义客户对经纪人施加的证书要求。 默认情况下，这是 ssl.CERT_REQUIRED，这意味着代理必须提供证书。 有关此参数的更多信息，请参阅 ssl pydoc。 |
| tls_version | 指定要使用的 SSL / TLS 协议的版本。 默认情况下（如果 python 版本支持它），检测到最高的 TLS 版本。 |
| ciphers | 指定哪些加密密码可供此连接使用的字符串，或者使用 None 来使用默认值。 有关更多信息，请参阅 ssl pydoc。 |

必须在调用 connect() 之前调用。  
##### （5）tls_set_context()  

配置网络加密和认证上下文。 启用 SSL / TLS 支持。  

```

[空代码块]  

```

| 参数 | 含义 |
| --- | --- |
| 参数 | 含义 |
| context | 一个 ssl.SSLContext 对象。 |

必须在调用 connect() 之前调用。  
##### （6）tls_insecure_set()  

配置服务器证书中服务器主机名的验证。  

```

[空代码块]  

```

如果 value 设置为 True，则不可能保证您连接的主机不模拟您的服务器。 这在初始服务器测试中可能很有用，但是，恶意的第三方通过可以 DNS 欺骗模拟您的服务器。  
●请勿在真实系统中使用此功能。 将值设置为 True 意味着使用加密没有意义。  
●必须在 connect() 之前和 tls_set() 或 tls_set_context() 之后调用。  
##### （7）enable_logger()  

使用标准的 Python 日志包启用日志记录。 这可以与 on_log 回调方法同时使用  

```

[空代码块]  

```

如果指定了记录器，那么将使用该 logging.Logger 对象，否则将自动创建一个。 按照以下映射将 Paho 日志记录级别转换为标准日志级别：  
| Paho | logging |
| --- | --- |
| Paho | logging |
| MQTT_LOG_ERR | ligging.ERROR |
| MQTT_LOG_WARNING | logging.WARNING |
| MQTT_LOG_NOTICE | logging.INFO (no direct equivalent) |
| MQTT_LOG_INFO | logging.INFO |
| MQTT_LOG_DEBUG | logging.DEBUG |

##### （8）disable_logger()  

使用标准 python 日志包禁用日志记录。 这对 on_log 回调没有影响。  

```

[空代码块]  

```

##### （9）username_pw_set()  

为代理认证设置一个用户名和一个可选的密码。必须在 connect() 之前调用。  

```

[空代码块]  

```

##### （10）user_data_set()  

设置在生成事件时将传递给回调的私人用户数据。 将其用于您自己的目的以支持您的应用程序。  

```

[空代码块]  

```

##### （11）will_set()  

设置要发送给代理的遗嘱。 如果客户端断开而没有调用 disconnect()，代理将代表它发布消息。  

```

[空代码块]  

```

| 参数 | 含义 |
| --- | --- |
| 参数 | 含义 |
| topic | 该遗嘱消息发布的主题 |
| payload | 该消息将作为遗嘱发送 |
| qos | 用于遗嘱的服务质量等级 |
| retain | 如果设置为 True，遗嘱消息将被设置为该主题的 “最后已知良好”/ 保留消息。 |

如果 qos 不是 0,1 或 2，或者主题为 None 或字符串长度为零，则引发 ValueError。  
##### （11）reconnect_delay_set（）  

客户端将自动重试连接。 在每次尝试之间，它会在 min_delay 和 max_delay 之间等待几秒钟。  

```

[空代码块]  

```

当连接丢失时，最初重新连接尝试延迟 min_delay 秒。 延迟在随后的尝试到中增加一倍。当连接完成时（例如收到 CONNACK，而不仅仅是 TCP 连接建立），延迟重置为 min_delay。  
#### 4.connect()  

connect()函数将客户端连接到代理。 这是一个阻塞函数。  

```

[空代码块]  

```

| 参数 | 含义 |
| --- | --- |
| 参数 | 含义 |
| host | 远程代理的主机名或 IP 地址 |
| port | 要连接的服务器主机的网络端口。 默认为 1883 |
| keepalive | 与代理通信之间允许的最长时间段（以秒为单位）。 如果没有其他消息正在交换，则它将控制客户端向代理发送 ping 消息的速率 |
| bind_address | 假设存在多个接口，将绑定此客户端的本地网络接口的 IP 地址 |

#### 5.connect_async()  

与 loop_start() 一起使用以非阻塞方式连接。 直到调用 loop_start() 之前，连接才会完成。  

```

[空代码块]  

```

#### 6.connect_srv()  

使用 SRV DNS 查找连接到代理以获取代理地址。  

```

[空代码块]  

```

| 参数 | 含义 |
| --- | --- |
| 参数 | 含义 |
| domain | 该 DNS 域搜索 SRV 记录。 如果无，请尝试确定本地域名。 |

#### 7.reconnect()  

使用先前提供的详细信息重新连接到经纪商。 在调用此函数之前，您必须先调用 connect()。  

```

[空代码块]  

```

#### 8.disconnect()  

干净地从代理断开连接。 使用 disconnect() 不会导致代理发送遗嘱消息。  

```

[空代码块]  

```

#### 9.loop()  

定期调用处理网络事件。  

```

[空代码块]  

```

此调用在 select() 中等待，直到网络套接字可用于读取或写入（如果适用），然后处理传入 / 传出数据。  
| 参数 | 含义 |
| --- | --- |
| 参数 | 含义 |
| timeout | 此方法最多可阻塞 timeout 秒 |
| max_packets | max_packets 参数已过时，应保留为未设置状态。 |

示例：  

```

[空代码块]  

```

#### 10.loop_start() / loop_stop()  

这些功能实现了到网络循环的线程接口。  

```

[空代码块]  

```

在 connect() 之前或之后调用 loop_start() 一次，会在后台运行一个线程来自动调用 loop()。这释放了可能阻塞的其他工作的主线程。这个调用也处理重新连接到代理。  
调用 loop_stop() 来停止后台线程；force 参数目前被忽略。  
示例：  

```

[空代码块]  

```

#### 11.loop_forever()  

这是网络循环的阻塞形式，直到客户端调用 disconnect() 时才会返回。它会自动处理重新连接。  

```

[空代码块]  

```

除了使用 connect_async 时的第一次连接尝试以外，请使用 retry_first_connection = True 使其重试第一个连接。这可能会导致客户端连接到一个不存在的主机的情况。  
#### 12.publish()  

从客户端发送消息给代理。  

```

[空代码块]  

```

消息将会发送给代理，并随后从代理发送到订阅匹配主题的任何客户端。  
| 参数 | 含义 |
| --- | --- |
| 参数 | 含义 |
| topic | 该消息发布的主题 |
| payload | 要发送的实际消息。如果没有给出，或设置为无，则将使用零长度消息。 传递 int 或 float 将导致有效负载转换为表示该数字的字符串。 如果你想发送一个真正的 int / float，使用 struct.pack（）来创建你需要的负载 |
| qos | 服务的质量级别 |
| retain | 如果设置为 True，则该消息将被设置为该主题的 “最后已知良好”/ 保留的消息 |

返回以下属性和方法的 MQTTMessageInfo:  
1rc: 发布的结果。  
| 内容 | 含义 |
| --- | --- |
| 内容 | 含义 |
| MQTT_ERR_SUCCESS | 成功 |
| MQTT_ERR_NO_CONN | 客户端当前未连接 |
| MQTT_ERR_QUEUE_SIZE | 当使用 max_queued_messages_set 来指示消息既不排队也不发送。 |

2mid: 发布请求的消息 ID。  
如果 mid 已定义，则可以通过检查 on_publish（）回调中的 mid 来跟踪发布请求。  
○wait_for_publish(): 函数将阻塞，直到消息发布。 如果消息未排队（rc == MQTT_ERR_QUEUE_SIZE），它将引发 ValueError。  
○is_published: 如果消息已发布，is_published 返回 True。 如果消息未排队（rc == MQTT_ERR_QUEUE_SIZE），它将引发 ValueError。  
如果主题为无，长度为零或无效（包含通配符），qos 不是 0、1 或 2 之一，或者有效负载长度大于 268435455 字节，则会引发 ValueError。  
#### 13.subscribe()  

```

[空代码块]  

```

订阅一个或多个主题。  
这个函数可以用三种不同的方式调用：  
##### （1）简单的字符串和整数  

```

[空代码块]  

```

| 参数 | 值 |
| --- | --- |
| 参数 | 值 |
| topic | 一个字符串，指定要订阅的订阅主题 |
| qos | 期望的服务质量等级。 默认为 0。 |

##### （2）字符串和整数元组  

```

[空代码块]  

```

| 参数 | 值 |
| --- | --- |
| 参数 | 值 |
| topic | （topic，qos）的元组。 主题和 qos 都必须存在于元组中。 |
| qos | 没有使用 |

##### （3）字符串和整数元组的列表  

这允许在单个 SUBSCRIPTION 命令中使用多个主题订阅，这比使用多个订阅 subscribe() 更有效。  

```

[空代码块]  

```

| 参数 | 值 |
| --- | --- |
| 参数 | 值 |
| topic | 格式元组列表（topic，qos）。 topic 和 qos 都必须出现在所有的元组中。 |
| qos | 没有使用 |

该函数返回一个元组(result，mid)。  
如果 qos 不是 0,1 或 2，或者主题为 None 或字符串长度为零，或者 topic 不是字符串，元组或列表，则引发 ValueError。  
#### 14.unsubscribe()  

取消订阅一个或多个主题。  

```

[空代码块]  

```

| 参数 | 含义 |
| --- | --- |
| 参数 | 含义 |
| topic | 主题的单个字符串或者字符串列表 |

返回一个元组 (result, mid)  
#### 15\. 外部事件循环支持  

##### （1）loop_read()  

```

[空代码块]  

```

当套接字准备好读取时调用。 max_packets 已过时，应保持未设置状态。  
##### （2）loop_write()  

```

[空代码块]  

```

当套接字准备好写入时调用。 max_packets 已过时，应保持未设置状态。  
##### （3）loop_misc()  

```

[空代码块]  

```

每隔几秒呼叫一次以处理消息重试和 ping。  
##### （4）socket()  

```

[空代码块]  

```

返回客户端中使用的套接字对象，以允许与其他事件循环进行交互。  
##### （5）want_write()  

```

[空代码块]  

```

如果有数据等待写入，则返回 true，以允许将客户端与其他事件循环连接。  
#### 16\. 全局辅助函数  

client 模块还提供了一些全局帮助函数。  
1topic_matches_sub（sub，topic）可用于检查主题是否与预订匹配。  
2connack_string（connack_code）返回与 CONNACK 结果关联的错误字符串。  
3error_string（mqtt_errno）返回与 Paho MQTT 错误号关联的错误字符串。  
### Publish 模块  

该模块提供了一些帮助功能，可以以一次性方式直接发布消息。换句话说，它们对于您想要发布给代理的单个 / 多个消息然后断开与其他任何必需的连接的情况非常有用。  
#### 1.Single  

将一条消息发布给代理，然后彻底断开连接。  

```

[空代码块]  

```

| 参数 | 含义 |
| --- | --- |
| 参数 | 含义 |
| topic | 唯一必需的参数必须是负载将发布到的主题字符串。 |
| payload | 要发布的有效载荷。 如果 “” 或 None，零长度的有效载荷将被发布 |
| qos | 发布时使用的 qos 默认为 0 |
| retain | 设置消息保留（True）或不（False） |
| hostname | 一个包含要连接的代理地址的字符串。 默认为 localhost |
| port | 要连接到代理的端口。 默认为 1883 |
| client_id | 要使用的 MQTT 客户端 ID。 如果 “” 或 None，Paho 库会自动生成客户端 ID |
| keepalive | 客户端的存活超时值。 默认为 60 秒 |
| will | 一个包含客户端遗嘱参数的字典,will = {‘topic’: “\<topic>”, ‘payload’:”\<payload”>, ‘qos’:\<qos>, ‘retain’:\<retain>}. |
| auth | 一个包含客户端验证参数的字典,auth = {‘username’:”\<username>”, ‘password’:”\<password>”} |
| tls | 一个包含客户端的 TLS 配置参数的字典,dict = {‘ca_certs’:”\<ca_certs>”, ‘certfile’:”\<certfile>”, ‘keyfile’:”\<keyfile>”, ‘tls_version’:”\<tls_version>”, ‘ciphers’:”\<ciphers”>} |
| protocol | 选择要使用的 MQTT 协议的版本。 使用 MQTTv31 或 MQTTv311。 |
| transport | 设置为 “websockets” 通过 WebSockets 发送 MQTT。 保留默认的 “tcp” 使用原始 TCP。 |

示例：  

```

[空代码块]  

```

#### 2.Multiple  

将多条消息发布给代理，然后干净地断开连接。  

```

[空代码块]  

```

| 参数 | 含义 |
| --- | --- |
| 参数 | 含义 |
| msgs | 要发布的消息列表。 每条消息是一个字典或一个元组。msg = {‘topic’:”\<topic>”, ‘payload’:”\<payload>”, ‘qos’:\<qos>, ‘retain’:\<retain>}或(“\<topic>”, “\<payload>”, qos, retain) |

有关 hostname，port，client_id，keepalive，will，auth，tls，protocol，transport 的描述，请参阅 single（）。  
示例：  

```

[空代码块]  

```

### Subscribe 模块  

该模块提供了一些帮助功能，以允许直接订阅和处理消息。  
#### 1.Simple  

订阅一组主题并返回收到的消息。 这是一个阻塞函数。  

```

[空代码块]  

```

| 参数 | 含义 |
| --- | --- |
| 参数 | 含义 |
| topics | 唯一需要的参数是客户端将订阅的主题字符串。 如果需要订阅多个主题，这可以是字符串或字符串列表 |
| qos | 订阅时使用的 qos 默认为 0 |
| msg_count | 从代理检索的消息数量。 默认为 1. 如果为 1，则返回一个 MQTTMessage 对象。 如果 > 1，则返回 MQTTMessages 列表 |
| retained | 设置为 True 以考虑保留的消息，将其设置为 False 以忽略具有保留标志设置的消息 |
| hostname | 一个包含要连接的代理地址的字符串。 默认为 localhost |
| port | 要连接到代理的端口。 默认为 1883 |
| client_id | 要使用的 MQTT 客户端 ID。 如果 “” 或 None，Paho 库会自动生成客户端 ID |
| keepalive | 客户端的存活超时值。 默认为 60 秒。 |
| will | 一个包含客户端遗嘱参数的字典,will = {‘topic’: “\<topic>”, ‘payload’:”\<payload”>, ‘qos’:\<qos>, ‘retain’:\<retain>}. |
| auth | 一个包含客户端验证参数的字典,auth = {‘username’:”\<username>”, ‘password’:”\<password>”} |
| tls | 一个包含客户端的 TLS 配置参数的字典,dict = {‘ca_certs’:”\<ca_certs>”, ‘certfile’:”\<certfile>”, ‘keyfile’:”\<keyfile>”, ‘tls_version’:”\<tls_version>”, ‘ciphers’:”\<ciphers”>} |
| protocol | 选择要使用的 MQTT 协议的版本。 使用 MQTTv31 或 MQTTv311。 |

#### 2.Callback  

订阅一组主题并使用用户提供的回叫处理收到的消息。  

```

[空代码块]  

```

| 参数 | 含义 |
| --- | --- |
| 参数 | 含义 |
| callback | 一个 “on_message” 回调将被用于每个收到的消息 |
| topics | 客户端将订阅的主题字符串。 如果需要订阅多个主题，这可以是字符串或字符串列表。 |
| qos | 订阅时使用的 qos 默认为 0 |
| userdata | 用户提供的对象将在收到消息时传递给 on_message 回调函数 |

有关 hostname，port，client_id，keepalive，will，auth，tls，protocol 的描述，请参阅 simple（）。  
示例：  

```

[空代码块]  

```

## 使用EMQ 提供的免费公共 MQTT 服务器  

本文将使用 EMQ 提供的[免费公共 MQTT 服务器](https://www.emqx.com/zh/mqtt/public-mqtt5-broker)，该服务基于 MQTT 云服务 - EMQX Cloud 创建。服务器接入信息如下：  
●Broker: broker.emqx.io  
●TCP Port: 1883  
●Websocket Port: 8083  
#### 导入 paho-mqtt  

```

[空代码块]  

```

#### 编写连接回调函数  

可以在该回调函数中对 MQTT 连接成功或失败的情况进行处理，本示例将在连接成功后订阅 django/mqtt 主题。  

```

[空代码块]  

```

#### 编写消息回调函数  

该函数将打印 django/mqtt 主题接收到的消息。  

```

[空代码块]  

```

#### 增加 Django 配置项  

在 settings.py 中增加 MQTT 服务器的配置项。  
读者如果对如下配置项及本文中提到的 MQTT 相关概念有疑问，可查看博客 [MQTT 协议快速体验](https://www.emqx.com/zh/blog/the-easiest-guide-to-getting-started-with-mqtt)。  
本示例使用匿名认证，所以用户名与密码设置为空。  

```

[空代码块]  

```

#### 配置 MQTT 客户端  

```

[空代码块]  

```

#### 创建发布消息接口  

我们创建一个简单的 POST 接口实现 MQTT 消息发布。  
在实际应用中该接口可能需要进行一些更复杂的业务逻辑处理。  
在 views.py 中增加如下代码。  

```

[空代码块]  

```

在 urls.py 中增加如下代码。  

```

[空代码块]  

```

#### 启动 MQTT 客户端  

在 __init__.py 中增加如下代码。  

```

[空代码块]  

```

至此我们已完成了所有代码的编写，查看[完整代码](https://github.com/emqx/MQTT-Client-Examples/tree/master/mqtt-client-Django)。  
最后，执行如下命令运行 Django 项目。  

```

[空代码块]  

```

当 Django 应用启动后，MQTT 客户端将会连接到 MQTT 服务器，并且订阅主题 django/mqtt。  
### 测试  

接下来我们使用[开源的跨平台 MQTT 客户端 - MQTT X](https://mqttx.app/zh) 进行连接、订阅、发布测试。  
#### 测试消息接收  

1在 MQTT X 中创建 MQTT 连接，输入连接名称，其他参数保持默认，并点击右上角的 Connect 按钮连接至服务器。  
2在 MQTT X 底部的消息发布框里向 django/mqtt 主题发布消息 Hello from MQTT X。  
3在 Django 运行窗口中将能看到 MQTT X 发送的消息。  

#### 测试消息发布接口  

1在 MQTT X 中订阅 django/mqtt 主题。  

2使用 Postman 调用 /publish 接口：发送消息 Hello from Django 至 django/mqtt 主题。  

3在 MQTT X 中将能看到 Django 发送过来的消息。  

### 总结  

至此，我们使用 paho-mqtt 完成了 MQTT 客户端的开发，实现了在 Django 应用中使用 MQTT 进行通信。在实际应用中，我们可以根据业务需求对 MQTT 客户端进行扩展，实现更复杂的业务逻辑。接下来，读者可查看 EMQ 提供的 [MQTT 入门与进阶](https://www.emqx.com/zh/mqtt) 系列文章了解 MQTT 协议特性，探索 MQTT 的更多高级应用，开始 MQTT 应用及服务开发。  
## paho-mqtt测试  

[Python paho-mqtt 模块使用（转） - 乖乖楠 - 博客园](https://www.cnblogs.com/lnn123/p/10837754.html)  

```

[空代码块]  

```

## 工具类  

```

[空代码块]  

```

## paho-mqtt实现多客户端订阅一个主题，并保证消息只被接收一次  

[paho-mqtt实现多客户端订阅一个主题，并保证消息只被接收一次-CSDN博客](https://blog.csdn.net/qq_41034780/article/details/129087592)  
## mqtt_cli对象stop之后再loop_start  

# 前端实现  

前端一般使用 websocket 协议实现长链接通信，为了简化 websocket 协议编程，可以直接使用建立在 websocket 上的 mqtt 协议，能够直接得到登陆鉴权、消息分发的功能，免去手动编程解析。  
注意: 前端使用的话，要求服务端开通ws的使用方式，因为前端不能使用mqtt://，这是TCP的方式。而我们前端只能使用ws的方式，ws://开头。  
## mqttws31.js库  

该库不推荐使用  
[javascript - mosquitto 与 websocket 的结合 - 个人文章 - SegmentFault 思否](https://segmentfault.com/a/1190000016633438)  
前端 websocket 的 mqtt 协议依赖 mqttws31.js 库实现。  
react 项目中，在public/index.html中引入  

```

[空代码块]  

```

组件中调用：  

```

[空代码块]  

```

### 主要 API  

引入依赖文件后，能够得到一个全局的 Paho.MQTT 对象，主要 API 是该对象下的方法，包括主动连接、断开、发送数据以及收到数据的回掉函数接口  
1生成 mqtt 客户端  

```

[空代码块]  

```

2建立连接  

```

[空代码块]  

```

option 为配置参数，包括以下内容  

```

[空代码块]  

```

3订阅主题  

```

[空代码块]  

```

Topic name 为订阅的主题  
4接收数据  

```

[空代码块]  

```

接受数据通过回调函数进行处理，回调函数参数是信息体，包括收到的主题名称，数据，Qos 等数据  
5发送数据  

```

[空代码块]  

```

message 为要发送的数据  
6连接断开，以及重新连接  

```

[空代码块]  

```

连接断开也是通过回调函数来处理，入参数是失败的原因。在这里可以进行重连，如下。  

```

[空代码块]  

```

注意：重连时使用的链接 option 对象不能直接使用上一次链接成功后的 option 因为该对象已经被修改。  

一、能否控制页面不刷新（消息防丢失处理）  
控制页面不刷新的目的是防止实时消息的丢失，在用此用户实际浏览网页过程中，刷新只是一个短暂而又快速的过程，而前端进行 mqtt 连接时将连接配置项 clean 设置为 false，Qos 设置为 1 或者 2，能够实现即使恰巧用户在刷新网页时 后台通过 mqtt 推送了消息，错过了此次推送，待到刷新完毕 mqtt 进行再次连接时仍然能够接收到刚才错过的消息。  
该配置项含义如下：  
1clean：false，上文提到，无论是 tcp 实现的 mqtt 还是 websocket 实现的 mqtt，只要客户端与 broker 建立连接后 broker 就维护了一个 session 会话，包含了这个链接的所有信息，消息的收发也是通过该 session 实现的，session 的 key 时客户端链接时的 clientId。当 clean 设置为 false 时，意味着通知 broker，当该客户端掉线后不要清除客户端消息，下次重联客户端仍然采用与之前相同的 clientId 进行链接。  
2Qos 为消息服务质量，当设置为 1 或者 2 时，服务端向客户端订阅的某一个主题推送数据时，客户端处在刷新掉线情况下，没有对服务端进行回复，服务端会存储消息到 senssion 中，待到客户端恢复链接后，重新推送给订阅此主题的客户端。（这里客户端的恢复是有 mqtt 协议底层实现的，一般在库中已经实现，使用时不用关心）  
消息服务质量 Qos 分别如下：  
aQoS0，最多一次送达。  
bQoS1，至少一次送达。  
cQoS2，准确一次送达。  

二、消息去重  
当设置为 qos1 或者 qos2 时可能会出现收到重复消息，此时可以在消息数据中增加应用序号，每次收到后对比，如果重复则丢弃  
## mqtt.js  

Github：[https://github.com/mqttjs/MQTT.js](https://github.com/mqttjs/MQTT.js)  
[MQTT.js - MQTT Client Library Encyclopedia](https://www.hivemq.com/blog/mqtt-client-library-mqtt-js/)  
[使用 WebSocket 连接 MQTT 服务器 | EMQ](https://www.emqx.com/zh/blog/connect-to-mqtt-broker-with-websocket)  
# MQTT和WebSocket的关系  

MQTT（Message Queuing Telemetry Transport）本身并不是基于WebSocket的，但MQTT可以通过WebSocket进行传输（mqtt over websocket）。以下是对MQTT和WebSocket关系的详细解释：  
### 一、MQTT协议  

1定义：MQTT是一种轻量级的发布/订阅消息传输协议，广泛应用于物联网（IoT）设备之间的通信。  
2设计目标：MQTT在设计时考虑了低带宽、不可靠网络环境下的高效数据传输。  
3传输方式：MQTT原本是基于TCP/IP的协议，用于在客户端和服务器之间传输消息。  
### 二、WebSocket协议  

1定义：WebSocket是一种基于TCP的网络协议，用于在客户端和服务器之间建立持久连接，实现全双工通信。  
2特点：  
○允许服务器主动向客户端推送数据，同时也允许客户端向服务器发送数据。  
○使用更少的头部信息和保持连接的机制，减少了数据传输的开销。  
○提供了实时的、双向的通信机制，可以立即将数据从服务器推送到客户端，实现即时更新。  
### 三、MQTT与WebSocket的结合  

1原因：为了适应Web应用的需求，MQTT可以通过WebSocket进行传输。  
2好处：  
○在Web环境中的适用性：WebSocket是Web环境中的标准双向通信协议，可以很容易地在浏览器中使用。结合MQTT和WebSocket，浏览器客户端可以直接与MQTT代理（Broker）进行通信。  
○高效、低延迟的数据传输：MQTT和WebSocket都支持实时通信，结合使用可以在Web应用中实现高效、低延迟的数据传输。  
○跨平台使用：使用WebSocket，MQTT可以在各种平台（包括浏览器、移动设备和桌面应用）上使用，无需额外的网络库。  
# mosquitto  

简介： Eclipse Mosquitto是一个开源消息代理，实现了MQTT协议版本3.1和3.1.1、5.0。Mosquitto轻量，适用于低功耗单板计算机到完整服务器的所有设备。  
Mosquitto 的特点：  
1轻量级：Mosquitto 是一个轻量级的 MQTT 代理服务器，它的设计目标是提供高效、快速和可靠的消息传递，适用于各种规模的应用。  
2容易部署：Mosquitto 的安装和部署非常简单，可以在多个平台上运行，包括 Linux、Windows、macOS 等。  
3安全性：Mosquitto 支持基于 TLS/SSL 的加密通信，可以保护消息的安全性和机密性。同时，它还支持基于用户名和密码的身份验证，以及访问控制列表（ACL）来限制访问权限。  
4可扩展性：Mosquitto 支持多个客户端连接和多个主题的订阅，可以满足大规模应用的需求。  
●其他服务器代理实现：[https://github.com/mqtt/mqtt.github.io/wiki/servers](https://github.com/mqtt/mqtt.github.io/wiki/servers)  
●各操作系统安装指引：[https://mosquitto.org/download/](https://mosquitto.org/download/)  
## 安装mosquitto  

### Linux 安装mosquitto  

原文：[Linux搭建MQTT服务器（mosquitto）并使用-CSDN博客](https://blog.csdn.net/tswc_byy/article/details/130766747)  
#### 一、Linux 搭建 MQTT 服务器（mosquitto）并使用  

##### 1、安装依赖  

```

[空代码块]  

```

##### 2、下载 mosquitto  

官网：[https://mosquitto.org/](https://mosquitto.org/)  

```

[空代码块]  

```

##### 3、解压 编译 安装  

```

[空代码块]  

```

之后会碰到找不到 libmosquitto.so.1 这个问题，修改链接路径，重新加载动态链接库  

```

[空代码块]  

```

##### 4、创建配置文件 mosquitto.conf  

```

[空代码块]  

```

配置文件中默认使用 user mosquitto。 如果不想创建此用户，可以修改成 root  

```

[空代码块]  

```

##### 5、启动、查看、关闭程序  

```

[空代码块]  

```

##### 6、设置用户名密码  

```

[空代码块]  

```

编辑配置文件  

```

[空代码块]  

```

增加下面内容  

```

[空代码块]  

```

然后重启即可  
##### 7、查看版本  

```

[空代码块]  

```

##### 8、本地简单测试  

```

[空代码块]  

```

###### 打开一个订阅者  

```

[空代码块]  

```

###### 打开一个发布者  

```

[空代码块]  

```

相同 topic 的双方，发布者 pub 发送 “发布内容” 给订阅者 sub  
#### 二、安装过程中报错解决  

##### 1.mosquitto_ctrl.h:21:25: 致命错误：cjson/cJSON.h：没有那个文件或目录  

解决方法：缺少 cJSON 库，安装 cJSON 库即可。  
cJSON 下载地址：[https://github.com/arnoldlu/cJSON](https://github.com/arnoldlu/cJSON)  
我用的是这个版本：[https://codeload.github.com/arnoldlu/cJSON/tar.gz/refs/tags/v1.3.2](https://codeload.github.com/arnoldlu/cJSON/tar.gz/refs/tags/v1.3.2)  
将下载的 cJSON-x.x.x.tar.gz 压缩包上传到 / home 文件夹中，并解压到 cJSON 文件夹中  

```

[空代码块]  

```

参考文章：  
[https://blog.csdn.net/doujingwei0825/article/details/129111730](https://blog.csdn.net/doujingwei0825/article/details/129111730)  
## mosquitto.conf  

```

[空代码块]  

```

### port和listener  

这里的配置文件有如下不同：  

默认的1883端口和扩展的1884端口都没有配置protocol参数，因此，mosquitto会使用默认的mqtt协议；  
为扩展端口1885、1886配置了protocol参数为mqtt，则mosquitto在这两个端口上使用mqtt协议；  
扩展端口9001配置了protocol参数为websocket，则mosquitto在该端口上使用websocket协议；  
### 无密码配置  

为了在 MQTT Broker Mosquitto 中实现无认证（匿名）访问，我们可以使用allow_anonymous选项进行设置。下面是如何配置匿名访问的具体步骤：  

```

[空代码块]  

```

在这段配置中，我们为监听在 1883 端口的 Broker 启用了匿名访问权限。这意味着任何连接到此端口的客户端无需提供用户名和密码即可进行通信。  
值得注意的是，在同一台 MQTT Broker 上同时允许匿名访问和经过身份验证的访问是完全可行的。特别是对于动态安全插件而言，它支持对匿名用户和已认证用户提供不同的权限设定，这在某些场景下非常实用。  
### 密码文件配置  

密码文件是存储用户名和密码的一种简单机制，特别适用于拥有少量且相对固定用户的场景。通过将用户名与加密后的密码记录在一个单独的文件中，可以方便地对 MQTT Broker 进行用户身份验证。  
更新密码文件 当您对密码文件进行更改时，需要向 Broker 发送 SIGHUP 信号以触发其重新加载文件内容：  

```

[空代码块]  

```

创建密码文件 利用mosquitto_passwd工具可以创建并填充密码文件。若要新建一个密码文件并添加用户，请执行以下命令，系统会提示您输入密码。注意，这里的-c选项表示如果文件已存在则会被覆盖：  

```

[空代码块]  

```

若要向已存在的密码文件中添加更多用户或修改现有用户的密码，只需省略 -c 参数：  

```

[空代码块]  

```

从密码文件中移除用户 如需从密码文件中删除某个用户，请使用以下命令：  

```

[空代码块]  

```

另外，您也可以在单行命令中添加 / 更新用户名及其密码，但请注意，这种方式会导致密码明文出现在命令行及命令历史记录中：  

```

[空代码块]  

```

配置 Broker 开始使用密码文件之前，您需要在 Broker 的配置文件中添加password_file选项，并指向您的密码文件位置：  

```

[空代码块]  

```

确保运行 Mosquitto 服务的用户有权限读取该密码文件。在 Linux/POSIX 系统上，通常这个用户是mosquitto，而 /etc/mosquitto/password_file 是存放此文件的一个常见路径。  
针对不同监听器设置独立的安全设置 如果您启用了per_listener_settings true选项，以便为每个监听器设置不同的安全参数，则必须在相关监听器配置之后指定相应的密码文件：  

```

[空代码块]  

```

这样，连接到 1883 端口的客户端就需要通过密码文件中的凭证进行身份验证。  
[mosquitto启动命令以及配置文件使用_mosquitto 启动-CSDN博客](https://blog.csdn.net/weixin_45459266/article/details/136903981)  
#### 示例  

默认示例配置文件：  
1pwfile.example (保存用户名与密码)  
2aclfile.example (保存权限配置)  
首先我们来新增两个用户 1： admin/admin 2: mosquitto/mosquitto  
具体步骤：  
1打开 mosquitto.conf 文件，找到 allow_anonymous 节点，这个节点作用是，是否开启匿名用户登录，默认是 true。打开此项配置（将前面的 # 号去掉）之后将其值改为 true  
修改前：#allow_anonymous  
修改后：allow_anonymous false  
2找到 password_file 节点，这个节点是告诉服务器你要配置的用户将存放在哪里。打开此配置并指定 pwfile.example 文件路劲（注意是绝对路劲）  
修改前：#password_file  
修改后：password_file /etc/mosquitto/pwfile.example （这里的地址根据自己文件实际位置填写）  
3创建用户名和密码、打开命令窗口 键入如下命令：  

```

[空代码块]  

```

提示连续两次输入密码、创建成功。命令解释： -c 创建一个用户、/etc/mosquitto/pwfile.example 是将用户创建到 pwfile.example 文件中、admin 是用户名。  
至此两个用户创建成功，此时如果查看 pwfile.example 文件会发现其中多了两个用户。  

此时所有客户端连接 Mosquitto 服务都需要输入用户名密码  
## mosquitto命令  

[MQTT mosquitto 订阅 、发布常用命令及示例_mosquitto 命令-CSDN博客](https://blog.csdn.net/laoweieda/article/details/132735223)  
### 1\. mosquitto  

1\. 启动 Mosquitto 代理服务器  
这个命令将启动 Mosquitto MQTT 代理服务器，默认监听 1883 端口。  
2\. 指定监听端口启动 Mosquitto 代理服务器：  
mosquitto -p 1884  
这个命令将启动 Mosquitto MQTT 代理服务器，并监听在 1884 端口。  
3\. 指定配置文件启动 Mosquitto 代理服务器：  
mosquitto -c mosquitto.conf  
这个命令将使用指定的配置文件（例如 mosquitto.conf）启动 Mosquitto MQTT 代理服务器。  
4\. 查看 Mosquitto 代理服务器的日志输出：  
mosquitto -v  
这个命令将启动 Mosquitto MQTT 代理服务器，并打印详细的日志输出信息。  
5\. 发布消息到指定主题：  
mosquitto_pub -h broker.hivemq.com -t test/topic -m "Hello, MQTT!"  
这个命令将消息 "Hello, MQTT!" 发布到名为 "test/topic" 的主题。  
6\. 订阅指定主题并接收消息：  
mosquitto_sub -h broker.hivemq.com -t test/topic  
这个命令将订阅名为 "test/topic" 的主题，并接收该主题下的所有消息。  
7\. 指定用户名和密码订阅主题：  
mosquitto_sub -h broker.hivemq.com -t test/topic -u your_username -P your_password  
这个命令将在连接到 MQTT 代理服务器时使用指定的用户名和密码，并订阅 "test/topic" 主题。  
### 2\. mosquitto_pub  

mosquitto_pub 是 Mosquitto MQTT 客户端工具的一部分，它用于发布（publish）MQTT 消息到指定的主题。以下是一些常用的 mosquitto_pub 命令及示例：  
1\. 发布消息到指定主题：  
mosquitto_pub -h broker.hivemq.com -t test/topic -m "Hello, MQTT!"  
这个命令将消息 "Hello, MQTT!" 发布到名为 "test/topic" 的主题。  
2\. 指定用户名和密码发布消息：  
mosquitto_pub -h broker.hivemq.com -t test/topic -m "Hello, MQTT!" -u your_username -P your_password  
这个命令将在连接到 MQTT 代理服务器时使用指定的用户名和密码，并发布消息到 "test/topic" 主题。  
3\. 指定消息的 QoS 级别：  
mosquitto_pub -h broker.hivemq.com -t test/topic -m "Hello, MQTT!" -q 1  
这个命令将消息发布到 "test/topic" 主题，并将消息的 QoS 级别设置为 1。  
4\. 指定消息的保留标志：  
mosquitto_pub -h broker.hivemq.com -t test/topic -m "Hello, MQTT!" -r  
这个命令将消息发布到 "test/topic" 主题，并设置消息的保留标志为 true。  
5\. 从文件中读取消息内容：  
mosquitto_pub -h broker.hivemq.com -t test/topic -f message.txt  
这个命令将从名为 "message.txt" 的文件中读取消息内容，并将其发布到 "test/topic" 主题。  
参数：  
●-h 服务器主机，默认localhost  
●-t 指定主题  
●-u 用户名  
●-P 密码  
●-i 客户端id，唯一  
●-m 发布的消息内容  
### 3\. mosquitto_sub  

以下是一些常用的 mosquitto_sub 命令及示例：  
1\. 订阅指定主题并接收消息：  
mosquitto_sub -h broker.hivemq.com -t test/topic  
这个命令将订阅名为 "test/topic" 的主题，并接收该主题下的所有消息。  
2\. 指定用户名和密码订阅主题：  
mosquitto_sub -h broker.hivemq.com -t test/topic -u your_username -P your_password  
这个命令将在连接到 MQTT 代理服务器时使用指定的用户名和密码，并订阅 "test/topic" 主题。  
3\. 指定消息的 QoS 级别：  
mosquitto_sub -h broker.hivemq.com -t test/topic -q 1  
这个命令将订阅 "test/topic" 主题，并将消息的 QoS 级别设置为 1。  
4\. 打印接收到的消息内容和主题：  
mosquitto_sub -h broker.hivemq.com -t test/topic -v  
这个命令将订阅 "test/topic" 主题，并打印接收到的消息内容和主题。  
5\. 指定客户端标识订阅主题：  
mosquitto_sub -h broker.hivemq.com -t test/topic -i myclient  
这个命令将使用 "myclient" 作为客户端标识连接到 MQTT 代理服务器，并订阅 "test/topic" 主题。  
注意：这些是一些常用的 Mosquitto 命令及示例，你可以根据自己的需求进行调整和使用。更多关于 Mosquitto 命令和选项，请参考 Mosquitto 官方文档。  
以下是 Mosquitto 常用命令的详细示例：  
1mosquitto_sub 订阅主题并接收消息：  
●订阅主题 mytopic 并接收消息：mosquitto_sub -t mytopic  
●订阅主题 mytopic ，指定 MQTT 代理服务器的主机名为 localhost ，端口号为 1883 ，并显示接收到的消息的详细信息：mosquitto_sub -t mytopic -h localhost -p 1883 -v  
●订阅主题 mytopic ，连接到 MQTT 代理服务器时使用用户名和密码进行身份验证：mosquitto_sub -t mytopic -u myusername -P mypassword  
2mosquitto_pub 发布消息到指定主题：  
●发布消息 Hello, MQTT! 到主题 mytopic ：mosquitto_pub -t mytopic -m "Hello, MQTT!"  
●发布消息 Hello, MQTT! 到主题 mytopic ，指定 MQTT 代理服务器的主机名为 localhost ，端口号为 1883 ：mosquitto_pub -t mytopic -m "Hello, MQTT!" -h localhost -p 1883  
●发布消息 Hello, MQTT! 到主题 mytopic ，连接到 MQTT 代理服务器时使用用户名和密码进行身份验证：mosquitto_pub -t mytopic -m "Hello, MQTT!" -u myusername -P mypassword  
3mosquitto_passwd 管理 Mosquitto 的用户密码：  
●创建新用户 myuser 并设置密码，将其保存到名为 passwordfile 的密码文件中：mosquitto_passwd -c passwordfile myuser  
●为现有用户 myuser 设置密码，将密码保存到名为 passwordfile 的密码文件中：mosquitto_passwd -b passwordfile myuser mypassword  
●删除名为 myuser 的用户，从名为 passwordfile 的密码文件中：mosquitto_passwd -D passwordfile myuser  
4mosquitto_ctrl 控制 Mosquitto 代理程序的运行：  
●显示 Mosquitto 代理程序的运行状态：mosquitto_ctrl status  
●启动 Mosquitto 代理程序：mosquitto_ctrl start  
●停止 Mosquitto 代理程序：mosquitto_ctrl stop    ●重启 Mosquitto 代理程序：mosquitto_ctrl restart  
这些示例可以帮助您更好地理解如何使用 Mosquitto 的常用命令。根据您的需求，您可以根据这些示例进行相应的修改和调整。  
## Nginx 代理Websocket  

Nginx 通过在客户端和后端服务器之间建立隧道来支持 WebSockets 通信。为了让 Nginx 可以将来自客户端的 Upgrade 请求发送到后端服务器，Upgrade 和 Connection 的头信息必须被显式的设置。  

```

[空代码块]  

```

一旦我们完成以上设置，Nginx 就可以处理 WebSocket 连接了  
### Nginx 代理 WebSocket 保持长连接的方案  

现象描述：用 nginx 反代代理某个业务，发现平均 1 分钟左右，就会出现 webSocket 连接中断，然后查看了一下，是 nginx 出现的问题。 产生原因：nginx 等待第一次通讯和第二次通讯的时间差，超过了它设定的最大等待时间，简单来说就是超时！  
1解决方法 1  
其实只要配置 nginx.conf 的对应 localhost 里面的这几个参数就好  

```

[空代码块]  

```

2解决方法 2：发心跳包，原理就是在有效地再读时间内进行通讯，重新刷新再读时间  

```

[空代码块]  

```

[Nginx反向代理WebSocket（WSS） - JZLZLZL - 博客园](https://www.cnblogs.com/ybyqjzl/p/10350732.html)  
# MQTT客户端工具  

客户端：  
●MQTTX（测试工具，支持web端）：[https://mqttx.app/](https://mqttx.app/)  
●MQTT.fx：[https://www.softblade.de/download/](https://www.softblade.de/download/)  
●MqttInsight：[https://gitee.com/ptma/mqtt-insight](https://gitee.com/ptma/mqtt-insight)  
网页版：  
●[https://mqttx.app/web-client](https://mqttx.app/web-client)  
●[https://mqtt.p2hp.com/websocket/](https://mqtt.p2hp.com/websocket/)  
●[https://www.hivemq.com/demos/websocket-client/](https://www.hivemq.com/demos/websocket-client/)  
●Websocket在线模拟请求工具：[http://www.jsons.cn/websocket/](http://www.jsons.cn/websocket/)  
# Reference  

●mqtt资源：[https://mqtt.iot01.com/](https://mqtt.iot01.com/)  
●EMQ官方文档：[MQTT 协议快速入门：基础知识和实用教程 | EMQ](https://www.emqx.com/zh/blog/the-easiest-guide-to-getting-started-with-mqtt)  
●EMQX官方示例（各种语言）：[https://github.com/emqx/MQTT-Client-Examples](https://github.com/emqx/MQTT-Client-Examples)  
●MQTT客户端：[https://github.com/mqtt/mqtt.org/wiki/servers](https://github.com/mqtt/mqtt.org/wiki/servers)  
●[还在用WebSocket实现实时消息推送？试试MQTT吧，真香！_websocket mqtt-CSDN博客](https://blog.csdn.net/lxy1290439047/article/details/139782804)  
●[MQTT在Python中的使用mqtt-paho(简单实例, 回调函数,回调参数，qos安全等级)详解及回调函数的正确用法_paho-mqtt-CSDN博客](https://blog.csdn.net/XC_SunnyBoy/article/details/115790445)  
●[如何在 Django 项目中使用 MQTT_mqtt django-CSDN博客](https://blog.csdn.net/emqx_broker/article/details/127494232)  
●[MQTT在Python的使用整理(2024.11.06)_python mqtt-CSDN博客](https://blog.csdn.net/ChenLing666/article/details/143576928)  
●[前端mqtt使用总结-CSDN博客](https://blog.csdn.net/sinat_34605342/article/details/121488413)