
```
sequenceDiagram

participant vm客户端
participant msgGateWay
participant msgCenter
participant redis数据库
participant pms


vm客户端->>pms:初始化配置请求
pms-->>pms:根据售货机id 分配gateway接入信息
pms-->>vm客户端:  
vm客户端->>msgGateWay:建立连接,发送au请求
msgGateWay->>msgCenter:发送给msgCenter处理au事件请求
msgCenter->>pms:请求验证售货机合法性
pms->pms: 验证售货机合法性并返回
pms-->msgCenter: 
msgCenter->>redis数据库:设置设备在线状态
redis数据库-->>msgCenter:  
msgCenter-->>msgGateWay:  
msgGateWay-->>vm客户端:  返回认证结果

msgGateWay-->>msgGateWay:轮询检测与售货机的session 状态
msgGateWay->>msgCenter:上报在线售货机心跳
msgCenter->>redis数据库:设置最后在线时间戳

vm客户端-->msgGateWay:售货机客户端断开连接
msgGateWay-->>msgGateWay:轮询检测与售货机的session 状态
msgGateWay->>msgCenter:发送售货机断连事件
msgCenter->>redis数据库:设置设备在线状态

pms-->>pms:执行轮询任务
pms->>redis数据库: 检测设备状态及在线时间戳判断设备是否在线

msgCenter->>msgCenter:执行调度任务
msgCenter->>msgGateWay:心跳检测msgGateWay 状态
msgGateWay-->>msgCenter:返回心跳结果异常
msgCenter->>redis数据库:将挂载到msgGateWay的设备踢下线
redis数据库-->>msgCenter: 返回

```
