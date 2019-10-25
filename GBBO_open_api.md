# 55 Open API 文档 

> ```
> Copyright © 2019   v1.2.1
> ```

[TOC]


# 入门指南

* 以下所有接口 BaseURL 均为： **https://api.55.com/broker/spot/openApi**
* 所有接口响应均为JSON格式
* 所有时间、时间戳均为UNIX时间，单位为毫秒



## 快速开始

1. 申请机构账户，并获取 `accountID` 和 `secretKey`
2. 使用 `JWT` 对每个请求进行签名
3. 调用创建订单ID接口 `order.generateOrderId`
4. 创建订单 `order.insertOrder`
5. 使用 `SSE` 协议客户端或 `WebSocket` 获取回报信息



## 请求方式
使用 POST 方式请求，请求示例：
```assembly
URL：
https://api.55.com/broker/spot/openApi?signature={signature}

HTTP：
POST /broker/spot/openApi HTTP/1.1
Host: api.55.com
Content-Type: application/json
cache-control: no-cache
{
    "accountId": "ST-00000001"
}
```

请求的接口在签名 `signature` 中指定。



## 签名说明

接口中权限均使用 JWT 作为签名 `signature`，详细内容请参考： [jwt.io](https://jwt.io/)

每次请求时，需要使用 JWT 工具并用申请的 secretKey 进行签名，得到 signature 并作为URL的参数：

```js
signature = JWT(payload={
    "accountId": "accountId",
    "secretKeyId": "secretKeyId",
    "digest": hash256(RequestBody),
    "method": "module.methodName",
    "exp": 1535526941
}, key=secretKey, algorithm='HS256')
```

payload 的签名信息中包含如下内容：

- digest：请求内容body的sha256编码hash值
- method：对应的方法名
- exp：过期时间，即token的过期时间



## 标准响应说明

发起API调用时，服务端会返回 JSON 对象，基本结构如下：

```json
{
  "result": null,
  "error": {
    "code": 131114,
    "message": "account not found"
  }
}
```
其中：

* result：请求成功时包含此key，返回响应结果，当调用方法错误时为null
* error：请求失败时包含此key，返回错误对象，无错误时为null
* code：响应的错误码，具体参见错误码说明
* message：错误描述



# 接口说明

**注：**以下标注的响应内容均为标准响应中的 result 部分



## 订单接口

### 创建订单ID

创建订单前需要先获取订单ID，然后使用该ID创建订单：

```assembly
method:	order.generateOrderId
```
**参数:**

```json
{
  //账号ID
  "accountId": "ST-0000001"
}
```

**响应:**

```json
{
  //订单ID
  "value": "0faeaf4433ec4aa591c90560a1aedb80", 
  //订单ID过期时间
  "expiredAt": 1552114241319
}
```



### 创建新订单

发起创建新订单请求：

```assembly
method:	order.insertOrder
```
**参数:**

```json
{
  //账号ID
  "accountId": "ST-0000001",
  //通过 orderId.create 创建的有效订单ID
  "orderId": "c825b00059b4479b81e9a5420284b664",
  //订单信息
  "orderInfo": {
    //订单交易对
    "symbol": "BTCUSD",
    //订单类型
    "orderType": "LIMIT",
    //订单买卖方向
    "orderSide": "BUY",
    //限价单，挂单价格
   	"limitPrice": "100",
    //订单数量
    "quantity": "1.12"
  }
}
```

**响应:**

```json
{
  //账号ID
  "accountId": "ST-0000001", 
  //订单ID
  "orderId": "c825b00059b4479b81e9a5420284b664", 
  //订单交易对
  "symbol": "BTCUSD", 
  //订单类型
  "orderType": "LIMIT",
  //订单买卖方向
  "orderSide": "BUY", 
  //订单限价价格
  "limitPrice": 100, 
  //订单挂单数量
  "quantity": 1.12, 
  //订单平均成交价，单位为当前交易对的计价资产
  "filledAveragePrice": 0, 
  //订单累计成交的数量
  "filledCumulativeQuantity": 0, 
  //订单待成交的数量
  "openQuantity": 1.12, 
  //订单状态
  "orderStatus": "PENDING_SUBMIT", 
  //订单创建时间
  "createdAt": 1552113774280, 
  //订单更新时间
  "updatedAt": 1552113774280, 
  //如果有撤单，撤单的操作时间或撤单完成的时间
  "cancelledUpdatedAt": null, 
  //最后一次成交的时间
  "filledUpdatedAt": null
}
```

**订单种类:**

- LIMIT 限价单

**订单方向:**

- BUY 买入
- SELL 卖出

**订单状态:**

| 订单状态       | 状态说明                                         |
| -------------- | ------------------------------------------------ |
| PENDING_SUBMIT | 新建订单（已提交到订单队列中，待确认）           |
| SUBMITTED      | 已提交，部分成交，撤单申请在队列中，均为当前状态 |
| FILLED         | 全部成交                                         |
| CANCELLED      | 已撤单                                           |
| FAILED         | 提交订单失败                                     |
| REJECTED       | 订单被拒绝                                       |



### 撤销订单

发起撤单请求，发起撤单后 cancelledUpdatedAt 将被更新。

**注：只有当订单状态为SUBMITTED，并且cancelledUpdatedAt为null的时候才可以进行撤单**

```assembly
method:		order.cancelOrder
```
**参数:**

```json
{
  "accountId": "ST-0000001", 
  "orderId": "c825b00059b4479b81e9a5420284b664"
}
```

**响应:**

```json
{
  //与【创建新订单】响应相同
}
```



### 查询单个订单信息

```assembly
method:	order.queryOrderInfo
```

**参数:**

```json
{
  "accountId": "STA-01102061",
  "orderId": "c825b00059b4479b81e9a5420284b664"
}
```

**响应:**

```json
{
  //与【创建新订单】响应相同
}
```



### 查询多个订单信息

```assembly
method:	order.listOrderInfo
```

**参数:**

```json
{
  "accountId": "STA-01102061",
	//个数限制[0,500]
	"orderIdList": ["c825b00059b4479b81e9a5420284b664","c825b00059b4479b81e9a5420284b665"]
}
```

**响应:**

```json
[
  {
     //与【创建新订单】响应相同
  }，
  {
     //与【创建新订单】响应相同
  }
]
```



###查询待成交订单

查询所有未成交的订单。
```assembly
method:	order.listOpenOrder
```
**参数:**
```json
{
  "accountId": "ST-0000001"
}
```

**响应:**

```json
[
  {
     //与【创建新订单】响应相同
  }，
  {
     //与【创建新订单】响应相同
  }
]
```



### 查询近期已完成订单

查询近期已成交订单、撤销订单、失败订单（已过期订单请调用 `listCompletedOrder` ）
```assembly
method:	order.listCompleted
```
**参数:**

```json
{
    "accountId": "ST-00000001",
    "page": "1",
    "limit": "10",
}
```

**响应:**
```json
{
  "data": [
    {
      //与【创建新订单】响应相同
    }
  ], 
  "page": 1, 
  "pages": 500, 
  "limit": 10, 
  "count": 5000
}
```



### 查询所有已完成订单

查询所有已成交订单、撤销订单、失败订单

```assembly
method:	order.listCompletedOrder
```

**参数:**

```json
{
  //账号ID
	"accountId": "ST-0000001", 
  //开始时间
  "startTime": "1568104459953", 
  //结束时间
  "endTime": "1568104459953", 
  //[0,500]，数量
  "limit": 100, 
  //Boolean 正向，反向 true，false，可选
  "forward": null, 
  //交易资产，例如交易对BTCUSDT，该为BTC，可选：填 null
  "baseAsset": null, 
  //计价资产，例如交易对BTCUSDT，该为USDT，可选
  "quoteAsset": null, 
  //OrderSide 枚举  BUY, SELL，可选
  "orderSide": null, 
  //枚举：FILLED， CANCELLED，可选
  "orderStatus": null 
}
```

**响应:**

```json
[
  {
     //与【创建新订单】响应相同
  }，
  {
     //与【创建新订单】响应相同
  }
]
```



### 查询单个订单日志

```assembly
method:	order.listOrderLog
```

**参数:**

```json
{
  //账号ID
  "accountId": "ST-0000001", 
  //订单ID
  "orderId": "c825b00059b4479b81e9a5420284b664"
}
```

**响应:**

```json
{
  //账号ID
  "accountId": "STA-01102061", 
  //订单ID
  "orderId": "c825b00059b4479b81e9a5420284b664", 
  //订单交易对
  "symbol": "BTCUSDT", 
  //订单类型
  "orderType": "LIMIT",
  //订单买卖方向
  "orderSide": "BUY", 
  //订单限价价格
  "limitPrice": 11, 
  //订单挂单数量
  "quantity": 1.12, 
  //订单平均成交价，单位为当前交易对的计价资产
  "filledAveragePrice": 0, 
  //累计成交额
  "filledCumulativeAmount": 0, 
  //订单累计成交的数量
  "filledCumulativeQuantity": 0, 
  //订单待成交的数量
  "openQuantity": 1.12, 
  //订单状态
  "orderStatus": "PENDING_SUBMIT", 
  //订单创建时间
  "createdAt": 1552113774280, 
  //订单更新时间
  "updatedAt": 1552113774280, 
  //如果有撤单，撤单的操作时间或撤单完成的时间
  "cancelledUpdatedAt": null, 
  //最后一次成交的时间
  "filledUpdatedAt": null,
  //日志ID
  "logId": null,
  //日志最后的修改的时候，在开通手续费调整的时候会发生变化
  "logTimestamp": null,
  //手续费现在的状态，在开通手续费调整的时候会发生变化
  "lastCommission": null,
  //手续费现在的币种，在开通手续费调整的时候会发生变化
  "lastCommissionCurrency": null,
  //如果有成交那么这次日志成交的数量
  "lastFilledQuantity": null,
  //如果有成交那么这次日志成交的价格
  "lastFilledPrice": null
}
```



## 订单变动通知推送

###变动通知接口

变动通知可以使用基于 http 的 SSE 协议，也可以使用WebSocket。

```assembly
method:	order.stream
```
**参数:**

```json
{
    "accountId": "ST-00000001"
}
```

**响应:**
```json
{
 "logTimestamp" : 1552116459512,
 "payload" : {
   "accountId" : "ST-00000001",
   "orderId" : "b967a71ff8924560986a88ff5e0b0c0e",
   "symbol" : "BTCUSDT",
   "orderType" : "LIMIT",
   "orderSide" : "SELL",
   "limitPrice" : 0.01,
   "quantity" : 0.001,
   "filledAveragePrice" : 0,
   "filledCumulativeQuantity" : 0,
   "openQuantity" : 0.001,
   "orderStatus" : "SUBMITTED",
   "createdAt" : 1552116459432,
   "updatedAt" : 1552116459512,
   "cancelledUpdatedAt" : null,
   "filledUpdatedAt" : null,
   "lastFilledQuantity" : null,
   "lastFilledPrice" : null,
   "lastFilledCreatedAt" : null
 }
}
```
```json
{
 "logTimestamp" : 1552116459536,
 "payload" : {
   "accountId" : "ST-00000001",
   "orderId" : "b967a71ff8924560986a88ff5e0b0c0e",
   "symbol" : "BTCUSDT",
   "orderType" : "LIMIT",
   "orderSide" : "SELL",
   "limitPrice" : 0.01,
   "quantity" : 0.001,
   "filledAveragePrice" : 3928.4,
   "filledCumulativeQuantity" : 0.001,
   "openQuantity" : 0,
   "orderStatus" : "FILLED",
   "createdAt" : 1552116459432,
   "updatedAt" : 1552116459536,
   "cancelledUpdatedAt" : null,
   "filledUpdatedAt" : 1552116459467,
   //订单逐笔：成交数量
   "lastFilledQuantity" : 0.001,
   //订单逐笔：成交数价格
   "lastFilledPrice" : 3928.4,
   //订单逐笔：成交时间
   "lastFilledCreatedAt" : 1552116459467
 }
}
```



### SSE 协议

推送通知使用基于 http 的 SSE 协议，详细内容请参考：[Server-Sent Events](https://www.w3.org/TR/eventsource/)

可下载推荐的客户端：  

- [java](https://github.com/heremaps/oksse)  
- [php](https://github.com/obsh/sse-client)  
- [python](https://github.com/mpetazzoni/sseclient)
- [go](https://github.com/r3labs/sse)

订单提交成功返回示例：

```assembly
http[s]://api.55.com/broker/spot/openApi?signature={signature}&body={body}
method:	order.stream

GET /broker/spot/openApi HTTP/1.1
Host: api.55.com

event:_RESULT
data:{
data:  "logTimestamp" : 1552101711879,
data:  "payload" : {
data:    "accountId" : "ST-00000001",
data:    "orderId" : "a260db7d7f964504bcc58d84b9624840",
data:    "symbol" : "BTCUSDT",
data:    "orderType" : "LIMIT",
data:    "orderSide" : "BUY",
data:    "limitPrice" : 100,
data:    "quantity" : 1.12,
data:    "filledAveragePrice" : 0,
data:    "filledCumulativeQuantity" : 0,
data:    "openQuantity" : 1.12,
data:    "orderStatus" : "SUBMITTED",
data:    "createdAt" : 1552101711813,
data:    "updatedAt" : 1552101711879
data:  }
data:}
```

内容为空的是心跳信息，心跳周期为15秒。



### WebSocket 协议

推送通知可以使用 WebSocket 协议：

```assembly
# URL中需要增加参数：&p=webSocket
ws[s]://api.55.com/broker/spot/openApi?signature={signature}&body={body}&p=webSocket
method:	order.stream

RESULT:
{
	"logTimestamp" : 1552116459512,
	"payload" : {
      "accountId" : "STA-01102061",
      "orderId" : "b967a71ff8924560986a88ff5e0b0c0e",
      "symbol" : "BTCUSDT",
      "orderType" : "LIMIT",
      "orderSide" : "SELL",
      "limitPrice" : 0.01,
      "quantity" : 0.001,
      "filledAveragePrice" : 0,
      "filledCumulativeQuantity" : 0,
      "openQuantity" : 0.001,
      "orderStatus" : "SUBMITTED",
      "createdAt" : 1552116459432,
      "updatedAt" : 1552116459512,
      "cancelledUpdatedAt" : null,
      "filledUpdatedAt" : null,
      "lastFilledQuantity" : null,
      "lastFilledPrice" : null,
      "lastFilledCreatedAt" : null
   }
}
```



## 账户部分

### 查询账户信息

```assembly
method:	account.queryAccountInfo
```

**参数:**

```json
{
  //账号ID
  "accountId": "ST-0000001",
}
```

**响应:**

```json
{
  //账号ID
  "accountId": "STA-0000001", 
  //账号状态
  "accountStatus": "OPENED", 
  //手续费折扣
  "orderCommissionRate": "1", 
  //是否开启调整订单手续费功能
  "orderCommissionAdjust": "false", 
  //创建时间
  "createdAt": 1537857271784, 
  //更新时间
  "updatedAt": 1551956476878, 
}
```

**账户状态:**

| 账户状态   | 状态说明                                                     |
| ---------- | ------------------------------------------------------------ |
| OPENED     | 正常                                                         |
| RESTRICTED | 限制，不能使用 提交订单，撤单等写操作，但是可以进行查询订单等读取操作 |
| SUSPENDED  | 暂停，禁用任何操作                                           |



## 资产部分

### 查询账户资产列表

```assembly
method:	asset.listBalance
```

**参数:**

```json
{
  //账号ID
  "accountId": "ST-0000001",
}
```

**响应:**

```json
[
  {
    //账号ID
    "accountId": "ST-0000001", 
    //资产代码
    "currency": "BTC", 
    //可用余额
    "available": 51.95,
    //已经冻结资产额
    "frozen": 0,
    //资产总额 
    "amount": 51.95,
    //更新时间 
    "updatedAt": 1547208404061
  }
]
```



## 工具接口

### 查询交易对列表

```assembly
method:	utils.listSymbol
```

**参数:**
无

**响应:**

```json
[
    {
      //交易对
      "symbol": "BTCUSDT",
      //现在的交易状态
      "status": "TRADING",
      //下一个交易状态
      "nextStatus": null,
      //下一个交易状态时间
      "nextStatusAt": null,
      //交易资产，交易对左侧币种
      "baseAsset": "BTC",
      //交易资产的小数后精度
      "baseAssetPrecision": 8,
      //计价资产，交易对右侧币种
      "quoteAsset": "USDT",
      //计价资产的小数后精度
      "quotePrecision": 8,
      //最小价格
      "minPrice": 0.00000001,
      //最大价格
      "maxPrice": 99999,
      //价格的步长，price % priceTickSize == 0
      "priceTickSize": 0.000001,
      //最小交易数量
      "minQuantity": 0.01,
      //最大交易数量
      "maxQuantity": 99999,
      //交易数量步长，quantity % quantityStepSize == 0
      "quantityStepSize": 0.01,
      //该交易对手续费
      "commissionRate": 0.01
    }
  ]
```

**交易对状态:**

| 交易对状态 | 状态说明   |
| ---------- | ---------- |
| TRADING    | 正常交易中 |
| HALT       | 暂停交易   |



### 查询资产币种列表

```assembly
method:	utils.listCurrency
```

**参数:**
无

**响应:**

```json
[
  {
    //资产名称
    "currency": "BTC",
    //资产小数后精度
    "currencyPrecision": 8,
    //提币的最大值
    "withdrawMaxAmount": 1000,
    //提币的最小值
    "withdrawMinAmount": 0.01,
    //提币的最少手续费
    "withdrawMinFee": 0.0001,
    //当前币种充提状态
    "status": "DEPOSIT_WITHDRAW",
    //下一个币种充提状态
    "nextStatus": null,
    //下一个币种充提状态更新时间
    "nextStatusAt": null
  }
]
```

**资产币种充提状态:**

|充提状态|状态说明|
| -- | -- |
| DEPOSIT_WITHDRAW| 可充，可提|
| NOT_DEPOSIT_WITHDRAW| 不可充，可提|
| DEPOSIT_NOT_WITHDRAW| 可充，不可提|
| NOT_DEPOSIT_NOT_WITHDRAW| 不可充，不可提|



### 查询当前服务端时间戳

```assembly
method:	utils.currentTimeMillis
```

**参数:**
无

**响应:**

```json
//当前服务端时间戳
1552112542263
```



# 其他说明

##HTTP错误码

- HTTP *200* 正常响应，如有异常会在响应体中说明。
- HTTP *4XX* 错误码用于指示错误的请求内容、行为、格式
- HTTP *403* 错误码表示错误次数过多，已封IP
- HTTP *5XX* 错误码用于指示服务侧的问题
- HTTP *502* 表示服务暂时不可用



## 响应错误码

错误状态码在接口响应中表示为整数，一般用16进制表示。

最后一位为A表示客户端错误，B表示服务端错误。

出现错误请不要频繁进行重试，客户端错误请修改提交参数后进行重新调用，服务端错误可以进行低频的重试，如多次出现此情况请联系客服。

重试中触发错误次数过多，过频繁都会导致请求IP被封。

| 错误码   | 说明|
| ------ | ------------------- |
| 0x01001A | 通用请求错误，需确认请求参数是否正确                         |
| 0x01002B | 通用服务端错误，需重试或联系客服                            |
| 0x02002A | 账号不存在，需确认账号ID                                     |
| 0x02005A | 账号状态不满足当前请求，需确认账号状态                       |
| 0x05001A | 订单ID不存在，需重新创建订单ID并重试                         |
| 0x05002B | 订单创建失败，需重试或联系客服                               |
| 0x05003A | 订单不存在，需修改订单ID                                     |
| 0x05004B | 撤单错误，需联系客服                                         |
| 0x05005A | 创建订单操作等待超时，需等待并重试    |
| 0x05006A | 当前待成交订单过多，无法创建新订单，需放弃或撤销待成交订单再重试 |
| 0x05007A | 当前订单状态无法满足撤单操作，需确认订单状态再重试或放弃请求 |
| 0x05008A | 撤销订单操作等待超时，需等待并重试      |
| 0x05011A | 创建订单操作次数超过24小时内允许的最大限制，需联系客服  |
| 0x05014A | 正在撤单，需确认订单状态                                     |
| 0x05015A | 订单为Pending_Submit状态不可撤单，需等待订单为Submitted状态后再操作撤单（如超过10分钟仍为Pending_Submit需联系客服） |
| 0x06002A | 资产可用额度不足，需确认资产可用额度是否充足                 |
