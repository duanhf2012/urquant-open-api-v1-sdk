# WebSocket API for URQUANT
## 开始使用 
WebSocket是HTML5一种新的协议(Protocol)。它实现了客户端与服务器全双工通信，使得数据可以快速地双向传播。通过一次简单的握手就可以建立客户端和服务器连接，服务器根据业务规则可以主动推送信息给客户端。其优点如下：   
- 客户端和服务器进行数据传输时，请求头信息比较小，大概2个字节；   
- 客户端和服务器皆可以主动地发送数据给对方；   
- 不需要多次创建TCP请求和销毁，节约宽带和服务器的资源。  
目前平台只支持WebSocket API与URQUANT平台接入。
## 请求交互
**URQ平台**WebSocket服务连接地址：`wss://urquant.urquant.net:10440/wsv1`  
#### 发送请求    
请求数据格式为：  
`{'op':'optype','param':{'key1':'value1','key2':'value2'}} `  
其中  
op: 操作类型   
param: 参数为选填参数，根据赶快件名称来决定参数体内容，内容是一个json结构 。
    



## API接口说明  
### 登录 
####1. Ping
```
请求方向：Client->APIServer
# Request 
{
 "op":"ping",
 "timestamp": 1558335839000
}
```
请求说明
op:ping为操作类型
timestamp:为时间戳

```
# Response
{
	"op":"pong",
	"timestamp": 1558335839000
}
```
注：返回的数据里面的 "pong" 的timestamp值为收到的 "ping" 的timestamp值 
注：WebSocket Client 和 WebSocket Server 建立连接之后，WebSocket Client 每隔 5s（这个频率可能会变化） 会向 WebSocket Server 发起一次心跳，WebSocket Client 忽略心跳2次后，WebSocket Server 将会主动断开连接；WebSocket Client发送最近2次心跳message中的其中一个ping的值，WebSocket Server都会保持WebSocket连接。


####2. 登录到urquant API平台

```
请求方向：Client->APIServer
# Request 
{
	"id":1,
	"op": "login",
	"param": {
		"serverid": 1000,
		"timestamp": 1558335839000,
		"apikey": "985d5b66-57ce-40fb-b714-afc0b9787083",
		"sign": "7L+zFQ+CEgGu5rzCj4+BdV2/uUHGqddA9pI6ztsRRPs=",
		"statuslist": [{
			"pid": 100001,
			"maxoverload": 10
		}]
	}
}
```
请求说明
id(int64):客户端带过来id
op(string):login表示登录
serverid(int32):  量化策略程序编号id，不允许重复
timestamp(int64): 时间戳毫秒
apikey(string):URQUANT后台申请获取apikey信息
sign(string):参照签名部分
statuslist(json):同步当前策略程序支持的产品id与最大负载等信息，同时将正在运行的apikey同步到 URQUANT API平台
	pid: 支持的产品id,发布产品时获取id
	maxoverload:最大允许负载的apikey数量
	
```
# Response
{
	"id":1,
	"op": "login",
	"code":100,
	"data":"invalid sign"
}
```

返回值说明	
如果在一定时间内没有登录成功，会被断开连接
id:客户端请求带过来的id
op:操作类型
code:错误代码
data:如果错误，返回错误信息


####3. 同步账号
```
请求方向：Client->APIServer
# Request 
{
	"id":1,
	"op": "accountstate",
	"param": {
		"serverid": 1000,
		"accountlist": [{
			"pid": 100001,
			"apikey":"985d5b66-57ce-40fb-b714-afc0b9787083",
			"state":1,
			"msg":""
		}]
	}
}
```
请求说明
id:客户端带来id
op:同步账号状态
serverid:服务器id
pid:正在运行的产品id
apikey:交易所apikey
state: 0 停止  1 运行
msg:附带停止和运行原因内容

```
# Response
{
	"id":1,
	"op": "accountstate",
	"code":0,
	"data":"ok"
}
```
id: 客户端带来id
op:同步账号状态
code:错误代码
data:如果错误，返回错误信息


####4. 策略控制
```
请求方向：APIServer->Client
# Request 
{
	"id":1,
	"op": "accountcontrol",
	"param": {
			"pid": 100001,
			"apikey":"985d5b66-57ce-40fb-b714-afc0b9787083",
			"optype":1
			}
}
```

请求说明
id:服务器带来id
op:策略控制
pid:产品id
apikey:正在控制的apikey
optype:0表示停止策略   1表示启动策略


```
# Response
{
	"id":1,
	"op": "accountcontrol",
	"param": {
		"accountlist": [{
			"pid": 100001,
			"apikey":"985d5b66-57ce-40fb-b714-afc0b9787083",
			"optype":1,
			"code":0,
			"data":"ok"
		}]
		}
}

```
id: 服务器带来id
op:同步账号状态
code:错误代码
data:如果错误，返回错误信息

####5. 同步负载情况
```
请求方向：Client->APIServer
# Request 
{
	"id": 1,
	"op": "serverinfo",
	"param": {
		"pid": 100001,
		"currentload": 1,
		"maxoverload": 10
	}
}
```

请求说明
op:量化程序同步负载状态
pid:产品id
currentload:当前负载api数
maxoverload:最大允许负载api数量
注意：如果量化程序需要暂停提供新api服务，可以传入maxoverload为0


```
# Response
{
	"id": 1,
	"op": "serverinfo",
	"code":0,
	"data":"ok"
}
```






