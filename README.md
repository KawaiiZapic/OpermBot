# OpermBot
`Open + Permission + Bot = OpermBot`  
Largely inspried by [Shizuku](https://github.com/RikkaApps/Shizuku).  
`Shizuku`是一个安卓的超级权限管理器,通过开放系统的Java API使得普通应用不需要`Root`权限也能执行某些操作.

受到启发, `OpermBot`就此诞生.  
`OpermBot`意在解决直接给Bot赋予管理员权限带来的一系列问题, 如意外滥权, 或者管理员人数过多超过限制等等.  
使用`OpermBot`, 群内只需拥有一个机器人管理员, 其他所有需要管理权限的Bot通过与`OpermBot API`交互完成所需操作, 群主无需再为其余任何Bot赋予管理员权限.  

通过`OpermBot`的权限管理, 群主只需赋予机器人必要的权限, 而不用给予完整的管理员权限, 极大的优化了机器人的权限结构, 大幅降低了意外操作的可能性, 特别是在群内有多个不同功能的Bot时效果显著.  
例如, 某个Bot需要修改群名称以实现Tag功能, 通过`OpermBot API`, 这个Bot不再需要完整的管理员权限, 仅需要赋予修改群信息的权限即可, 这个Bot除了能修改群名片以外, 权限都与普通群成员一致.  

`OpermBot API`基于`HTTP`/`Websocket`/`QQ私聊`实现,使用`JSON`作为基本结构,保证其易用性和简易性.

# 协议设计
 ## 1.1 入群注册
 入群注册分为两类, 一类是普通Bot入群时进行注册, 一类是`OpermBot`进群时进行注册.  
 ### 1.1.1 普通Bot进群注册
 #### Period 1 判断状况
 普通Bot进群后,首先判断当前有无权限发送消息, 若不能发送消息, 应当立即终止注册进程, 防止意料以外的情况发生.  
 #### Period 2 发送注册消息
 构造如下数据结构:
 ```json
 {
     "BotName": "TestBot",
     "Action": "Handshake",
     "Group": 299990001,
     "OpermBotAPI": "1.0",
     "Timestamp": 1322394441,
     "RequiredPermissions": [
         "Admin.Group.Name.*",
         "Creator.Member.SpecialTag.*",
         "Any.Permission.You.Wanted"
     ]
 }
 ```
将其进行`Base64`编码, 得到注册消息主体:
```
ewogICAgICJCb3ROYW1lIjogIlRlc3RCb3QiLAogICAgICJBY3Rpb24iOiAiSGFuZHNoYWtlIiwKICAgICAiT3Blcm1Cb3RBUEkiOiAiMS4wIiwKICAgICAiVGltZXN0YW1wIjogMTMyMjM5NDQ0MSwKICAgICAiUmVxdWlyZWRQZXJtaXNzaW9uIjogWwogICAgICAgICAiQWRtaW4uR3JvdXAuTmFtZS4qIiwKICAgICAgICAgIkNyZWF0b3IuTWVtYmVyLlNwZWNpYWxUYWcuKiIsCiAgICAgICAgICJBbnkuUGVybWlzc2lvbi5Zb3UuV2FudGVkIgogICAgIF0KIH0K
```
添加头部, 便于`OpermBot`进行辨识与处理:
```
OpermBotewogICAgICJCb3ROYW1lIjogIlRlc3RCb3QiLAogICAgICJBY3Rpb24iOiAiSGFuZHNoYWtlIiwKICAgICAiT3Blcm1Cb3RBUEkiOiAiMS4wIiwKICAgICAiVGltZXN0YW1wIjogMTMyMjM5NDQ0MSwKICAgICAiUmVxdWlyZWRQZXJtaXNzaW9uIjogWwogICAgICAgICAiQWRtaW4uR3JvdXAuTmFtZS4qIiwKICAgICAgICAgIkNyZWF0b3IuTWVtYmVyLlNwZWNpYWxUYWcuKiIsCiAgICAgICAgICJBbnkuUGVybWlzc2lvbi5Zb3UuV2FudGVkIgogICAgIF0KIH0K
```
 #### Period 3 等待响应
 响应应在`15s`内发送,若在`15s`内没有接受到任何响应,则判断无`OpermBot`,终止注册进程,进行下一步处理.  
 若存在`OpermBot`,`OpermBot`会在响应内告知本群`OpermBot`的权限状况, 以及注册成功与否.
 响应数据示例如下:
 ```json
 {
     "RequestID": "97f938503cc2c5077b56ef6e8945bc74",
     "OpermBotAPI": "1.0",
     "Status": 1,
     "Timestamp": 1322394445,
     "AcceptedMethod": [
         "Websocket": {
             "Host": "localhost",
             "Port": 8000,
             "SSL": true
         },
         "HTTP": {
             "Host": "localhost",
             "Port": 8000,
             "SSL": true
         },
         "FriendMessage": true
     ]
 }
 ```
 
 其中:
 |属性|含义|
 |-|-|
 |RequestID|注册请求消息主体`md5`哈希结果|
 |Status|注册结果状态码,状态码下有详细说明|
 
状态码含义:
|值|含义|
|-|-|
|负数|出现错误,待定|
|0|注册已经完成,API交互途径早已确定并建立,所需权限已经被本群默认授予|
|1|注册已经完成,API交互途径早已确定并建立,`OpermBot`管理员尚未授予所需全部权限|
|2|注册尚未完成,因为API交互方式尚未确定,需要进一步处理,但是所需权限已经被本群默认授予|
|3|注册尚未完成,因为API交互方式尚未确定,需要进一步处理,且`OpermBot`管理员尚未授予所需全部权限|

响应将以与注册消息同样的结构发送,需要自行判断头部,对`base64`进行解码并解析`JSON`.

 #### Period 4 回复响应
 非`2`与`3`的状态码都不需要进行响应,因为此时`OpermBot`已经准备好与你的机器人进行交互,或者根本无法使用.  
 当状态码为`2`或`3`时,需要确定并建立交互途径,响应内容内已经包含了可用的交互途径,向这些方式中的任意一个建立连接,并在连接时告知自己的身份,即可完成注册.  
 告知身份的方式由交互途径具体决定.  
 交互途径可以任意切换,可以同时使用,需要新的交互方式只需直接再次通过同样的方法再次连接即可.  
 
---------
### 题外话
目前来说, 这个项目只是一个构想, 但这很明显, 对于Bot特别多的群是一种刚需.  
集中并粒度化的管理权限, 弥补了QQ本身权限机制的不足.  
即使没有很多机器人,对于某些强迫症来说也是福音.  
