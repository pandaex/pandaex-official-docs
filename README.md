PandaEX交易平台官方API文档
==================================================
[PandaEX][]交易平台开发者文档([English Docs][])。

<!-- TOC -->

- [介绍](#介绍)
- [开始使用](#开始使用)
- [API接口加密验证](#api接口加密验证)
    - [生成API Key](#生成api-key)
    - [发起请求](#发起请求)
    - [签名](#签名)
    - [选择时间戳](#选择时间戳)
    - [请求交互](#请求交互)
        - [请求](#请求)
        - [分页](#分页)
    - [标准规范](#标准规范)
        - [时间戳](#时间戳)
        - [例子](#例子)
        - [数字](#数字)
        - [限流](#限流)
                - [REST API](#rest-api)
- [现货(Spot)业务API参考](#现货spot业务api参考)
    - [币币行情API](#币币行情api)
        - [1. 获取所有币对列表](#1-获取所有币对列表)
        - [2. 获取币对交易深度列表](#2-获取某币对交易深度列表)
        - [3. 获取所有币对的深度信息](#3-获取所有币对的深度信息)
        - [4. 获取指定币对的深度信息](#4-获取指定币对的深度信息)
        - [5. 获取币对Ticker](#5-获取币对ticker)
        - [6. 获取币对历史成交记录](#6-获取币对历史成交记录)
        - [7. 获取K线数据](#7-获取k线数据)
        - [8. 获取服务器时间](#8-获取服务器时间)
        - [9. 获取币种代号](#9-获取币种代号)
    - [币币账户API](#币币账户api)
        - [1. 获取账户的资产信息](#1-获取账户的资产信息)
        - [2. 获取账户单个币种的资产信息](#2-获取账户单个币种的资产信息)
        - [3. 交易委托](#3-交易委托)
        - [4. 批量交易委托](#4-批量交易委托)
        - [5. 按订单撤销委托](#5-按订单撤销委托)
        - [6. 批量撤销委托](#6-批量撤销委托)
        - [7. 撤销所有委托](#7-撤销所有委托)
        - [8. 根据价格撤销委托](#8-根据价格撤销委托)
        - [9. 查询订单](#9-查询订单)
        - [10. 查询已完成的委托订单](#10-查询已完成的委托订单)
        - [11. 获取账单信息](#11-获取账单信息)
        - [12. 获取所有账单类型](#12-获取所有账单类型)
        - [13. 获取关注的币对列表](#13-获取关注的币对列表)

<!-- /TOC -->

# 介绍

欢迎使用[PandaEX][]开发者文档。

本文档提供了PandaEX交易平台的币币交易(Spot)业务的行情查询、交易、账户管理等API使用方法的介绍。
行情API是公开接口，提供币币交易市场的行情数据；交易和账户API需要身份验证，提供下单、撤单、查询订单和帐户信息、提现等功能。

# 开始使用    
REST，即Representational State Transfer的缩写，是一种流行的互联网传输架构。它具有结构清晰、符合标准、易于理解、扩展方便的，正得到越来越多网站的采用。其优点如下：

+ 在RESTful架构中，每一个URL代表一种资源；
+ 客户端和服务器之间，传递这种资源的某种表现层；
+ 客户端通过四个HTTP指令，对服务器端资源进行操作，实现“表现层状态转化”。

建议开发者使用REST API进行行情查询、币币交易和账户管理等操作。

# API接口加密验证
## 生成API KEY

在对任何请求进行签名之前，您必须通过 PandaEX 网站【用户中心】-【API】创建一个API KEY。 创建API KEY后，您将获得3个必须记住的信息：

* API Key

* Secret Key

* Passphrase

API Key和Secret Key将随机生成，Passphrase由用户自己设定。

## 发起请求

所有REST请求都必须包含以下标题：

* ACCESS-KEY API Key作为一个字符串
* ACCESS-SIGN 使用base64编码签名（请参阅签名消息）
* ACCESS-TIMESTAMP 作为您的请求的时间戳
* ACCESS-PASSPHRASE 您在创建API KEY时设置的Passphrase
* 所有请求都应该含有application/json类型内容，并且是有效的JSON

## 签名
ACCESS-SIGN的请求头是对 **timestamp + method + requestPath + "?" + queryString + body** 字符串(+表示字符串连接)使用**HMAC SHA256**方法加密，通过**BASE64**编码输出而得到的。其中，timestamp的值与ACCESS-TIMESTAMP请求头相同。

* method是请求方法(POST/GET/PUT/DELETE)，字母全部大写
* requestPath是请求接口路径
* queryString是GET请求中的查询字符串
* body是指请求主体的字符串，如果请求没有主体(通常为GET请求)，则body可省略

**例如：对于如下的请求参数进行签名**

* 获取深度信息，以ETH_USDT为例

```java
Timestamp = 1540286290170 
Method = "GET"
requestPath = "/openapi/exchange/public/ETH_USDT/orderbook"
queryString= "?size=100"

```

生成待签名的字符串

```java
Message = '1540286290170GET/openapi/exchange/public/ETH_USDT/orderbook?size=100'  
```
* 下单，以ETH_USDT为例

```java
Timestamp = 1540286476248 
Method = "POST"
requestPath = "/openapi/exchange/orders"
body = {"code":"ETH_USDT","side":"buy","type":"limit","size":"1","price":"1.001"}

```

生成待签名的字符串

```java
Message = '1540286476248POST/openapi/exchange/orders{"code":"ETH_USDT","side":"buy","type":"limit","size":"1","price":"1.001"}'  
```

然后，将待签名字符串添加私钥参数生成最终待签名字符串

```java
hmac = hmac(secretkey, Message, SHA256)
```

在使用前需要对于hmac进行base64编码

```java
Signature = base64.encode(hmac.digest())
```

## 请求交互  

REST访问的根URL：`https://www.pandaex.pro`

### 请求

所有请求基于Https协议，请求头信息中Content-Type需要统一设置为:'application/json’。

**请求交互说明**

1、请求参数：根据接口请求参数规定进行参数封装。

2、提交请求参数：将封装好的请求参数通过POST/GET/DELETE等方式提交至服务器。

3、服务器响应：服务器首先对用户请求数据进行参数安全校验，通过校验后根据业务逻辑将响应数据以JSON格式返回给用户。

4、数据处理：对服务器响应数据进行处理。

**成功**

HTTP状态码200表示成功响应，并可能包含内容。如果响应含有内容，则将显示在相应的返回内容里面。

**常见错误码**

* 400 Bad Request – Invalid request forma 请求格式无效

* 401 Unauthorized – Invalid API Key 无效的API Key

* 403 Forbidden – You do not have access to the requested resource 请求无权限

* 404 Not Found – 没有找到请求

* 429 Too Many Requests – 请求太频繁被系统限流

* 500 Internal Server Error – We had a problem with our server 服务器内部错误

如果失败，Response body带有错误描述信息

### 分页

部分返回数据集的REST请求支持使用游标分页。    

游标分页允许在结果的当前页面之前和之后获取结果，并且非常适合于实时数据。根据当前的返回结果，后续请求可以在此基础之上指定请求数据的方向，可以请求在这之前和之后的数据。before和after游标可通过响应头PD-BEFORE和PD-AFTER使用。

**例子**

`GET /orders?before=2&limit=30`

## 标准规范

### 时间戳

除非另外指定，API中的所有时间戳均以微秒为单位返回。

请求签名中的ACCESS-TIMESTAMP的单位是秒，允许用小数表示更精确的时间。请求的时间戳必须在API服务时间的30秒内，否则请求将被视为过期并被拒绝。如果本地服务器时间和API服务器时间之间存在较大的偏差，那么我们建议您使用通过查询API服务器时间来更新Http Header。

### 例子

`1524801032573`

### 数字

为了保持跨平台时精度的完整性，十进制数字作为字符串返回。建议您在发起请求时也将数字转换为字符串以避免截断和精度错误。 

整数（如交易编号和顺序）不加引号。

### 限流

如果请求过于频繁系统将自动限制请求，并在Http Header中返回429 too many requests状态码。

##### REST API

* 公共接口：我们通过IP限制公共接口的调用：每2秒最多6个请求。

* 私人接口：我们通过用户ID限制私人接口的调用：每2秒最多6个请求。

* 某些接口的特殊限制在具体的接口上注明

# 币币交易(Spot)API参考

## 币币行情API

### 1. 获取所有币对列表

**请求**

```http
    # Request
    GET /openapi/exchange/public/currencies
```

**命令举例**

```shell
    # shell
    curl -i -X GET 'https://www.pandaex.pro/openapi/exchange/public/currencies' \
    -H 'Content-Type: application/json; charset=utf-8' \
    -H 'ACCESS-KEY: 9f4a7317921ba663bb80a93c69d717af' \
    -H 'ACCESS-SIGN: 9SlGnF0795bSvOnFK6uKz79jxRPG7Yev303ZAPM6dO8=' \ 
    -H 'ACCESS-TIMESTAMP: 1569805597000' \
    -H 'ACCESS-PASSPHRASE: 123456'
```

**响应**

```javascript
    # Response
    [
        {
            "baseIncrement":0,
            "baseSymbol":"BTC",
            "id":0,
            "lastPrice":0,
            "makerFeesRate":"-0.001",
            "maxPrice":4,
            "maxVolume":4,
            "minTrade":0.001,
            "online":0,
            "pairCode":"BTC_USDT",
            "quoteIncrement":0,
            "quotePrecision":0,
            "quoteSymbol":"USDT",
            "sort":1,
            "tickerFeesRate":"-0.001"
        },
        {
            "baseIncrement":0,
            "baseSymbol":"ETH",
            "id":0,
            "lastPrice":0,
            "makerFeesRate":"-0.001",
            "maxPrice":4,
            "maxVolume":4,
            "minTrade":0.01,
            "online":0,
            "pairCode":"ETH_USDT",
            "quoteIncrement":0,
            "quotePrecision":0,
            "quoteSymbol":"USDT",
            "sort":2,
            "tickerFeesRate":"-0.001"
        },
        ...
    ]
```

**返回值说明**  


|返回字段 | 字段说明|
| ----------|:-------:|
| id            | 币对序号|
| baseSymbol    | 基准币，如：BTC |
| baseIncrement   | 基准币的最小交易单位 |
| quoteSymbol | 计价币，如：USDT |
| quoteIncrement | 计价币的最小交易单位 |
| quotePrecision | 计价币数量单位精度 |
| pairCode | 基准币和计价币的组合，如：BTC_USDT |
| lastPrice | 最新价格 |
| makerFeesRate  | maker 费率 |
| tickerFeesRate | ticker 费率 |
| maxPrice | 最高价格 |
| maxVolume | 最高交易量 |
| minTrade | 最小委托量 |
| online | 是否上线 |
| sort | 币对排序号 |


### 2. 获取某币对交易深度列表

**请求**

```http
    # Request
    GET /openapi/exchange/public/{pairCode}/orderBook
```

**命令举例**

```shell
    # shell
    curl -i -X GET 'https://www.pandaex.pro/openapi/exchange/public/BTC_USDT/orderBook' \
    -H 'Content-Type: application/json; charset=utf-8' \
    -H 'ACCESS-KEY: 9f4a7317921ba663bb80a93c69d717af' \
    -H 'ACCESS-SIGN: 9SlGnF0795bSvOnFK6uKz79jxRPG7Yev303ZAPM6dO8=' \ 
    -H 'ACCESS-TIMESTAMP: 1569805597000' \
    -H 'ACCESS-PASSPHRASE: 123456'
```

**响应**

```javascript
    # Response
    {
        "asks":[
            [
                "10463.3399",
                "0.0025"
            ],
            ...
        ],
        "bids":[
            [
                "7300.2456",
                "0.0022"
            ],
            ...
        ]
    }
```
**返回值说明**  


|返回字段|字段说明|  
| ------------- |----|
| asks | 卖方深度 |
| bids | 买方深度 |

**请求参数**  


| 参数名 | 参数类型  | 必填 | 描述 |
| ------------- |----|----|----|
| pairCode | String | 是 | 币对，如 BTC_USDT |

### 3. 获取所有币对的深度信息

获取所有币对的深度信息的接口。

**请求**

```http
    # Request
    
    GET /openapi/exchange/public/pairDepth
```

**命令举例**

```shell
    # shell
    curl -i -X GET 'https://www.pandaex.pro/openapi/exchange/public/pairDepth' \
    -H 'Content-Type: application/json; charset=utf-8' \
    -H 'ACCESS-KEY: 9f4a7317921ba663bb80a93c69d717af' \
    -H 'ACCESS-SIGN: 9SlGnF0795bSvOnFK6uKz79jxRPG7Yev303ZAPM6dO8=' \ 
    -H 'ACCESS-TIMESTAMP: 1569805597000' \
    -H 'ACCESS-PASSPHRASE: 123456'
```

**响应**
    
```javascript
    # Reponse

    [
        {
            "createOn":1556100483000,
            "depthLevel":"[0.0001,0.001,0.01,0.1]",
            "id":1,
            "legalCoin":"USD",
            "pairCode":"BTC_USDT",
            "updateOn":1556100483000
        },
        {
            "createOn":1556100483000,
            "depthLevel":"[0.0001,0.001,0.01,0.1]",
            "id":2,
            "legalCoin":"USD",
            "pairCode":"ETH_USDT",
            "updateOn":1556100483000
        },
        ...
    ]
```
    
**返回值说明**
    
|返回字段|字段说明|
|-----|----|
|id| 编号 |
|pairCode| 币对，如 BTC_USDT |
|legalCoin| 对应计价法币名称，USD、CNY |
|depthLevel| 档位 |
|createOn| 该条信息创建的时间戳，单位为毫秒 |
|updateOn| 该条信息更新的时间戳，单位为毫秒 |

### 4. 获取指定币对的深度信息

获取指定币对的深度信息的接口。

**请求**

```http
    # Request
    
    GET /openapi/exchange/public/{pairCode}/pairDepth
```

**命令举例**

```shell
    # shell
    curl -i -X GET 'https://www.pandaex.pro/openapi/exchange/public/BTC_USDT/pairDepth' \
    -H 'Content-Type: application/json; charset=utf-8' \
    -H 'ACCESS-KEY: 9f4a7317921ba663bb80a93c69d717af' \
    -H 'ACCESS-SIGN: 9SlGnF0795bSvOnFK6uKz79jxRPG7Yev303ZAPM6dO8=' \ 
    -H 'ACCESS-TIMESTAMP: 1569805597000' \
    -H 'ACCESS-PASSPHRASE: 123456'
```

**响应**
    
```javascript
    # Reponse

    [
        {
            "depthLevel":"[0.0001,0.001,0.01,0.1]",
            "id":2,
            "legalCoin":"USD",
            "pairCode":"ETH_USDT"
        }
    ]
```
    
**返回值说明**
    
|返回字段|字段说明|
|-----|----|
|id| 编号 |
|pairCode| 币对，如 BTC_USDT |
|legalCoin| 对应计价法币名称，USD、CNY |
|depthLevel| 档位 |

**请求参数**

|参数名|参数类型|必填|描述|
|------|----|:---:|:---:|
|pairCode|String|是|币对，如 BTC_USDT|

### 5. 获取币对Ticker

最新成交价、买一价、卖一价、24h最高价、24h最低价、24h开盘价和24h成交量的快照信息。

**请求**

```http
    # Request
    GET /openapi/exchange/public/{pairCode}/ticker
```

**命令举例**

```shell
    # shell
    curl -i -X GET 'https://www.pandaex.pro/openapi/exchange/public/BTC_USDT/ticker' \
    -H 'Content-Type: application/json; charset=utf-8' \
    -H 'ACCESS-KEY: 9f4a7317921ba663bb80a93c69d717af' \
    -H 'ACCESS-SIGN: 9SlGnF0795bSvOnFK6uKz79jxRPG7Yev303ZAPM6dO8=' \ 
    -H 'ACCESS-TIMESTAMP: 1569805597000' \
    -H 'ACCESS-PASSPHRASE: 123456'
```

**响应**

```javascript
    # Response
    {
        "buy":"218.1929",
        "change24":"0.11750000",
        "changePercentage":"",
        "changeRate24":"0.0005",
        "close":"",
        "createOn":1569807457106,
        "high":"218.34980000",
        "high24":"218.34980000",
        "last":"218.1983",
        "low":"218.04780000",
        "low24":"218.04780000",
        "open":"218.08080000",
        "pairCode":"ETH_USDT",
        "quoteVolume":"2579441.4163840000000000",
        "sell":"218.1987",
        "volume":"11830.00000000"
    }
```

**返回值说明**
 
|返回字段|字段说明|
|--------| :-------: |
|pairCode| 币对，如 BTC_USDT |
|createOn| 该条行情信息创建的时间戳，单位为毫秒 |
|buy|最新买入价|
|sell|最新卖出价|
|last|最新成交价|
|volume|基准币的成交量|
|quoteVolume|计价币的成交量|
|change24|24小时变化值|
|changeRate24|24小时涨跌比例|
|changePercentage|变化百分比|
|high|最高成交价|
|high24|24小时最高成交价|
|low|最低成交价|
|low24|24小时最低成交价|
|open|24小时的开盘价|
|close|24小时的收盘价|
    
**请求参数**

|参数名|参数类型|必填|描述|
|------|----|:---:|:---:|
|pairCode|String|是|币对，如 BTC_USDT|
    
### 6. 获取币对历史成交记录，支持分页查询

获取所请求交易对的历史成交信息，该请求支持分页。

**请求**

```http
    # Request
    GET /openapi/exchange/public/{pairCode}/fills?limit=10
```

**命令举例**

```shell
    # shell
    curl -i -X GET 'https://www.pandaex.pro/openapi/exchange/public/BTC_USDT/fills?limit=10' \
    -H 'Content-Type: application/json; charset=utf-8' \
    -H 'ACCESS-KEY: 9f4a7317921ba663bb80a93c69d717af' \
    -H 'ACCESS-SIGN: 9SlGnF0795bSvOnFK6uKz79jxRPG7Yev303ZAPM6dO8=' \ 
    -H 'ACCESS-TIMESTAMP: 1569805597000' \
    -H 'ACCESS-PASSPHRASE: 123456'
```

**响应**

```javascript
    # Response
    [
        [
            "218.1642",
            "2",
            "sell",
            1569808694511
        ],
        [
            "218.1896",
            "2",
            "sell",
            1569808692297
        ]
    ]
```

**返回值说明**


|返回字段|字段说明|
|--------|----|
| "218.1642" |成交价格|
| "2" |成交量|
| "sell" |Maker成交方向|
| 1569808694511 |成交时间戳|


**请求参数**

|参数名|参数类型|必填|描述|
|-----|:---:|----|----|
|code|String|是|币对，如 BTC_USDT|
|limit|Integer|否|请求返回数据量，默认值/最大值为100|

**解释说明**

+ 交易方向 side 表示每一笔成交订单中 maker 下单方向,maker 是指将订单挂在订单深度列表上的交易用户，即被动成交方。

+ buy 代表行情下跌，因为 maker 是买单，maker 的买单被成交，所以价格下跌；相反的情况下，sell代表行情上涨，因为此时maker是卖单，卖单被成交，表示上涨。

### 7. 获取K线数据

**请求**

```http
    # Request
    GET /openapi/exchange/public/{pairCode}/candles?interval=1min&start=<timestamp>&end=<timestamp>
```

**命令举例**

```shell
    # shell
    curl -i -X GET 'https://www.pandaex.pro/openapi/exchange/public/BTC_USDT/candles?interval=1min&start=1569804887031&end=1569808692297' \
    -H 'Content-Type: application/json; charset=utf-8' \
    -H 'ACCESS-KEY: 9f4a7317921ba663bb80a93c69d717af' \
    -H 'ACCESS-SIGN: 9SlGnF0795bSvOnFK6uKz79jxRPG7Yev303ZAPM6dO8=' \ 
    -H 'ACCESS-TIMESTAMP: 1569805597000' \
    -H 'ACCESS-PASSPHRASE: 123456'
```

**响应**
    
```javascript
    # Response
    [
        [ 1569808692297, 0.32, 0.42, 0.36, 0.41, 12.3 ],
        ...
    ]
```

**返回值说明（按顺序）**  
    
|返回字段|字段说明|
|-----|----|
|1569808692297|K线开始时间戳，单位为毫秒|
|0.32|最低价|
|0.42|最高价|
|0.36|开盘价（第一笔交易）|
|0.41|收盘价（最后一笔交易）|
|12.3|交易量（按交易币统计）|

**请求参数**
    
|参数名|参数类型|必填|描述|
|-----|----|----|----|
|pairCode|String|是|币对，如 BTC_USDT|
|interval|String|是|K线周期类型，如：1min/1hour/day/week|
|start|String|是|开始时间戳，单位为毫秒|
|end|String|是|结束时间戳，单位为毫秒|

### 8. 获取服务器时间

获取API服务器的时间的接口。

**请求**

```http
    # Request
    
    GET /openapi/exchange/public/time
```

**命令举例**

```shell
    # shell
    curl -i -X GET 'https://www.pandaex.pro/openapi/exchange/public/time' \
    -H 'Content-Type: application/json; charset=utf-8' \
    -H 'ACCESS-KEY: 9f4a7317921ba663bb80a93c69d717af' \
    -H 'ACCESS-SIGN: 9SlGnF0795bSvOnFK6uKz79jxRPG7Yev303ZAPM6dO8=' \ 
    -H 'ACCESS-TIMESTAMP: 1569805597000' \
    -H 'ACCESS-PASSPHRASE: 123456'
```

**响应**
    
```javascript
    # Reponse

    {
        "epoch":"1569811049.398",
        "iso":"2019-09-30T02:37:29.398Z",
        "timestamp":1569811049398
    }
```
    
**返回值说明**
    
|返回字段|字段说明|
|-----|----|
|epoch|以秒为时间戳形式表达的服务器时间|
|iso|为ISO 8061标准的时间字符串表达的服务器时间|
|timestamp|以毫秒为时间戳形式表达的服务器时间|

### 9. 获取币种代号

获取币种代号的接口。

**请求**

```http
    # Request
    
    GET /openapi/exchange/public/symbol
```

**命令举例**

```shell
    # shell
    curl -i -X GET 'https://www.pandaex.pro/openapi/exchange/public/symbol' \
    -H 'Content-Type: application/json; charset=utf-8' \
    -H 'ACCESS-KEY: 9f4a7317921ba663bb80a93c69d717af' \
    -H 'ACCESS-SIGN: 9SlGnF0795bSvOnFK6uKz79jxRPG7Yev303ZAPM6dO8=' \ 
    -H 'ACCESS-TIMESTAMP: 1569805597000' \
    -H 'ACCESS-PASSPHRASE: 123456'
```

**响应**
    
```javascript
    # Reponse

    ["BTC","USDT","ETH","LTC","ETC","DMA","SBS","NWD"]
```
    
**返回值说明**
    
|返回字段|字段说明|
|-----|----|
|"BTC"|币种代号|


## 币币账户API

### 1. 获取账户的资产信息

获取币币交易账户余额列表，查询各币种的余额，冻结和可用情况。

**请求**

```
    # Request
    GET /openapi/exchange/assets
```

**命令举例**

```shell
    # shell
    curl -i -X GET 'https://www.pandaex.pro/openapi/exchange/assets' \
    -H 'Content-Type: application/json; charset=utf-8' \
    -H 'ACCESS-KEY: 9f4a7317921ba663bb80a93c69d717af' \
    -H 'ACCESS-SIGN: 9SlGnF0795bSvOnFK6uKz79jxRPG7Yev303ZAPM6dO8=' \ 
    -H 'ACCESS-TIMESTAMP: 1569805597000' \
    -H 'ACCESS-PASSPHRASE: 123456'
```

**响应**

```
    # Response
    [
        {
            "available":"10",
            "baseBTC":"1",
            "brokerId":0,
            "hold":"0",
            "sort":5,
            "symbol":"BTC",
            "transfer":true,
            "userId":0,
            "withdrawLimit":"-1"
        },
        {
            "available":"50.1964034",
            "baseBTC":"0.021",
            "brokerId":0,
            "hold":"0",
            "sort":6,
            "symbol":"ETH",
            "transfer":true,
            "userId":0,
            "withdrawLimit":"-1"
        },
        ...
    ]
```

**返回值说明**

|返回字段|字段说明|
|----|----|
|userId|用户ID|
|symbol|币种代号|
|available|余额|
|baseBTC|折合成 BTC 的估值|
|brokerId|业务方ID|
|hold|是否冻结|
|sort|币种序号|
|transfer|是否可划转，true - 可划转，false - 不可划转|
|withdrawLimit|提币限额|

### 2. 获取账户单个币种的资产信息

获取币币交易账户单个币种的余额列表，查询各币种的余额，冻结和可用情况。

**请求**

```
    # Request
    GET /openapi/exchange/{symbol}/assets
```

**命令举例**

```shell
    # shell
    curl -i -X GET 'https://www.pandaex.pro/openapi/exchange/public/BTC/assets' \
    -H 'Content-Type: application/json; charset=utf-8' \
    -H 'ACCESS-KEY: 9f4a7317921ba663bb80a93c69d717af' \
    -H 'ACCESS-SIGN: 9SlGnF0795bSvOnFK6uKz79jxRPG7Yev303ZAPM6dO8=' \ 
    -H 'ACCESS-TIMESTAMP: 1569805597000' \
    -H 'ACCESS-PASSPHRASE: 123456'
```

**响应**

```
    # Response
    {
        "available":"50.1964034",
        "baseBTC":"0.021",
        "brokerId":0,
        "hold":"0",
        "sort":6,
        "symbol":"ETH",
        "transfer":true,
        "userId":0,
        "withdrawLimit":"-1"
    }
```

**返回值说明**

|返回字段|字段说明|
|----|----|
|userId|用户编号|
|symbol|币种代号|
|available|余额|
|baseBTC|折合成 BTC 的估值|
|brokerId|介绍人编号|
|hold|是否冻结|
|sort|币种序号|
|transfer|是否可划转，true - 可划转，false - 不可划转|
|withdrawLimit|提币限额|

**请求参数**
    
|参数名|参数类型|必填|描述|
|-----|----|----|----|
|symbol|String|是|币对，如 BTC、ETH|


### 3. 交易委托

PandaEX提供限价和市价两种订单类型。

**请求**

```
    # Request
    POST /openapi/exchange/{pairCode}/orders
```

**命令举例**

```shell
    # shell
    curl -i -X POST 'https://www.pandaex.pro/openapi/exchange/BTC_USDT/orders' \
    -H 'Content-Type: application/json; charset=utf-8' \
    -H 'ACCESS-KEY: 9f4a7317921ba663bb80a93c69d717af' \
    -H 'ACCESS-SIGN: 9SlGnF0795bSvOnFK6uKz79jxRPG7Yev303ZAPM6dO8=' \ 
    -H 'ACCESS-TIMESTAMP: 1569805597000' \
    -H 'ACCESS-PASSPHRASE: 123456' \
    -d '{"side": "buy", "systemOrderType": "limit", "volume": 1.00, "price": 218.56, "source": "openapi"}'
```

**响应**

```javascript
    # Response
    12345678
```
    
**返回值说明**

|返回字段|字段说明|
|----|----|
| 12345678 |订单ID|

**请求参数**

|参数名| 参数类型 |必填|描述|
|:----:|:----:|:---:|----|
|pairCode|String|是|币对，如：BTC_USDT|
|side|String|是|买入为buy，卖出为sell|
|systemOrderType|String|是|限价委托为limit，市价委托为market|
|volume|Decimal|否|限价委托以及市价卖出时传递，代表基准币的数量|
|price|Decimal|否|限价委托时传递，代表交易价格|
|quoteVolume|Decimal|否|市价买入时传递，代表计价币的数量|
|source|String|是|客户端来源类型，可以为：web, app, android, ios, openapi|

### 4. 批量交易委托

PandaEX提供限价和市价两种订单类型。

**请求**

```
    # Request
    POST /openapi/exchange/{pairCode}/bulkOrders
```

**命令举例**

```shell
    # shell
    curl -i -X POST 'https://www.pandaex.pro/openapi/exchange/ETH_USDT/bulkOrders' \
    -H 'Content-Type: application/json; charset=utf-8' \
    -H 'ACCESS-KEY: 9f4a7317921ba663bb80a93c69d717af' \
    -H 'ACCESS-SIGN: 9SlGnF0795bSvOnFK6uKz79jxRPG7Yev303ZAPM6dO8=' \ 
    -H 'ACCESS-TIMESTAMP: 1569805597000' \
    -H 'ACCESS-PASSPHRASE: 123456' \
    -d '[{"side": "buy", "systemOrderType": "limit", "volume": 1.00, "price": 218.56, "source": "openapi"}, {"side": "sell", "systemOrderType": "market", "volume": 1.00, "quoteVolume": 219.00, "source": "openapi"}]'
```

**响应**

```javascript
    # Response
    [
        {

        },
        ...
    ]
```
    
**返回值说明**

|返回字段|字段说明|
|----|----|
| id |订单id|
| pairCode |币对，如：BTC_USDT|
| userId |用户id|
| brokerId |业务方ID|
| side |交易方向，买入为buy，卖出为sell|
| entrustPrice |价格|
| amount |委托数量|
| dealAmount |已成交数量|
| quoteAmount |基准币数量，只有在市价买的情况下会用到|
| dealAmount |已成交数量|
| dealQuoteAmount |基准币已成交数量|
| systemOrderType |10:限价委托，11:市价委托|
| status |成交状态，0:未成交 1:部分成交 2:完全成交 3:撤单中 -1:已撤单数量|
| sourceInfo |客户端来源类型，可以为：web, app, android, ios, openapi|
| createOn |创建时间戳，单位为毫秒|
| updateOn |修改时间戳，单位为毫秒|
| symbol |币种代号|
| trunOver |成交金额|
| notStrike |尚未成交的数量|
| averagePrice |平均成交价|
| openAmount |尚未成交的数量|

**请求参数**

|参数名| 参数类型 |必填|描述|
|:----:|:----:|:---:|----|
|pairCode|String|是|币对，如：BTC_USDT|
|side|String|是|买入为buy，卖出为sell|
|systemOrderType|String|是|限价委托为limit，市价委托为market|
|volume|Decimal|否|限价委托以及市价卖出时传递，代表基准币的数量|
|price|Decimal|否|限价委托时传递，代表交易价格|
|quoteVolume|Decimal|否|市价买入时传递，代表计价币的数量|
|source|String|是|客户端来源类型，可以为：web, app, android, ios, openapi|

### 5. 按订单撤销委托

按照订单id撤销指定订单。由于是异步撤单，所以该接口没有返回值。

**请求**

```http
    # Request
    DELETE /openapi/exchange/{pairCode}/orders/{id}
```

**命令举例**

```shell
    # shell
    curl -i -X DELETE 'https://www.pandaex.pro/openapi/exchange/BTC_USDT/orders/20001' \
    -H 'Content-Type: application/json; charset=utf-8' \
    -H 'ACCESS-KEY: 9f4a7317921ba663bb80a93c69d717af' \
    -H 'ACCESS-SIGN: 9SlGnF0795bSvOnFK6uKz79jxRPG7Yev303ZAPM6dO8=' \ 
    -H 'ACCESS-TIMESTAMP: 1569805597000' \
    -H 'ACCESS-PASSPHRASE: 123456' 
```

**响应**

```javascript
    # Response
    {...}
```

**返回值说明**

没有返回

**请求参数**

|参数名|参数类型|必填|描述|
|---|----|----|----|
|pairCode|String|是|币对，如：BTC_USDT|
|id|Integer|是|需要撤销的未成交委托的id|

### 6. 批量撤销委托

撤销指定币对下多条未成交委托。由于是异步撤单，所以该接口没有返回值。

**请求**

```
    # Request
    DELETE /openapi/exchange/{pairCode}/orders
```

**命令举例**

```shell
    # shell
    curl -i -X DELETE 'https://www.pandaex.pro/openapi/exchange/BTC_USDT/orders' \
    -H 'Content-Type: application/json; charset=utf-8' \
    -H 'ACCESS-KEY: 9f4a7317921ba663bb80a93c69d717af' \
    -H 'ACCESS-SIGN: 9SlGnF0795bSvOnFK6uKz79jxRPG7Yev303ZAPM6dO8=' \ 
    -H 'ACCESS-TIMESTAMP: 1569805597000' \
    -H 'ACCESS-PASSPHRASE: 123456' \
    -d '{"ids": [10010,10011,10012]}'
```

**响应**

```javascript
    # Response
    {...}
```

**返回值说明**

没有返回

**请求参数**

|参数名|参数类型|必填|描述|
|----|----| ----| ----|
|pairCode|String|是|币对，如：BTC_USDT|
|ids|Long[]|是|订单id数组, 如[10010L,10011L,10012L]，目前只支持最多撤销50条订单，如果不填则撤销50条未完成订单|

### 7. 撤销所有委托

撤销指定币对下所有未成交委托。由于是异步撤单，所以该接口没有返回值。

**请求**

```
    # Request
    DELETE /openapi/exchange/{pairCode}/cancel-all
```

**命令举例**

```shell
    # shell
    curl -i -X DELETE 'https://www.pandaex.pro/openapi/exchange/BTC_USDT/cancel-all' \
    -H 'Content-Type: application/json; charset=utf-8' \
    -H 'ACCESS-KEY: 9f4a7317921ba663bb80a93c69d717af' \
    -H 'ACCESS-SIGN: 9SlGnF0795bSvOnFK6uKz79jxRPG7Yev303ZAPM6dO8=' \ 
    -H 'ACCESS-TIMESTAMP: 1569805597000' \
    -H 'ACCESS-PASSPHRASE: 123456'
```

**响应**

```javascript
    # Response
    {...}
```

**返回值说明**

没有返回

**请求参数**

|参数名|参数类型|必填|描述|
|----|----| ----| ----|
|pairCode|String|是|币对，如：BTC_USDT|

### 8. 根据价格撤销委托

根据价格撤销指定币对下未成交委托。由于是异步撤单，所以该接口没有返回值。

**请求**

```
    # Request
    DELETE /openapi/exchange/{pairCode}/cancel-price
```

**命令举例**

```shell
    # shell
    curl -i -X DELETE 'https://www.pandaex.pro/openapi/exchange/ETH_USDT/cancel-price?minPrice=218.50&maxPrice=219.00&side=buy' \
    -H 'Content-Type: application/json; charset=utf-8' \
    -H 'ACCESS-KEY: 9f4a7317921ba663bb80a93c69d717af' \
    -H 'ACCESS-SIGN: 9SlGnF0795bSvOnFK6uKz79jxRPG7Yev303ZAPM6dO8=' \ 
    -H 'ACCESS-TIMESTAMP: 1569805597000' \
    -H 'ACCESS-PASSPHRASE: 123456'
```

**响应**

```javascript
    # Response
    {...}
```

**返回值说明**

没有返回

**请求参数**

|参数名|参数类型|必填|描述|
|----|----| ----| ----|
|pairCode|String|是|币对，如：BTC_USDT|
|minPrice|Decimal|否|价格下限|
|maxPrice|Decimal|否|价格上限|
|side|String|否|买入为buy，卖出为sell|

### 9. 查询订单

按照一定参数查询订单，支持分页查询。
    
**请求**

```http   
    # Request
    GET /openapi/exchange/orders?pairCode=BTC_USDT&startDate=1556100483000&endDate=1569342299000&price=218.32&amount=1.215&systemOrderType=10&source=openapi&page=1&pageSize=10
```

**命令举例**

```shell
    # shell
    curl -i -X GET 'https://www.pandaex.pro/openapi/exchange/orders?pairCode=BTC_USDT&startDate=1556100483000&endDate=1569342299000&price=218.32&amount=1.215&systemOrderType=10&source=openapi&page=1&pageSize=10' \
    -H 'Content-Type: application/json; charset=utf-8' \
    -H 'ACCESS-KEY: 9f4a7317921ba663bb80a93c69d717af' \
    -H 'ACCESS-SIGN: 9SlGnF0795bSvOnFK6uKz79jxRPG7Yev303ZAPM6dO8=' \ 
    -H 'ACCESS-TIMESTAMP: 1569805597000' \
    -H 'ACCESS-PASSPHRASE: 123456'
```

**响应**

```javascript
    # Response
    {

    }
```

**返回值说明**

|返回字段|字段说明|
|----|----|
| id |订单id|
| pairCode |币对，如：BTC_USDT|
| userId |用户id|
| brokerId |业务方ID|
| side |交易方向，买入为buy，卖出为sell|
| entrustPrice |价格|
| amount |委托数量|
| dealAmount |已成交数量|
| quoteAmount |基准币数量，只有在市价买的情况下会用到|
| dealAmount |已成交数量|
| dealQuoteAmount |基准币已成交数量|
| systemOrderType |10:限价委托，11:市价委托|
| status |成交状态，0:未成交 1:部分成交 2:完全成交 3:撤单中 -1:已撤单数量|
| sourceInfo |客户端来源类型，可以为：web, app, android, ios, openapi|
| createOn |创建时间戳，单位为毫秒|
| updateOn |修改时间戳，单位为毫秒|
| symbol |币种代号|
| trunOver |成交金额|
| notStrike |尚未成交的数量|
| averagePrice |平均成交价|
| openAmount |尚未成交的数量|

**请求参数**

|参数名 | 参数类型 | 必填 | 描述 |
|---|----|----|----|
|pairCode|String|是|币对，如：BTC_USDT|
|startDate|Long|否|起始时间戳，单位为毫秒|
|endDate|Long|否|结束时间戳，单位为毫秒|
|price|Decimal|否|价格|
|amount|Decimal|否|数量|
|systemOrderType|Integer|否|10:限价委托，11:市价委托|
|source|String|否|客户端来源类型，可以为：web, app, android, ios, openapi|
|page|Integer|否|页号，不指定则返回第1页|
|pageSize|Integer|否|每页数量，如果不指定则最多返回300条|

### 10. 查询已完成的委托订单

按照一定参数查询已完成的订单，支持分页查询。

**请求**

```http
    # Request
    GET /openapi/exchange/{pairCode}/fulfillment?startDate=1556100483000&endDate=1569342299000&price=218.32&amount=1.215&systemOrderType=10&source=openapi&isHistory=1&page=1&pageSize=10
```

**命令举例**

```shell
    # shell
    curl -i -X GET 'https://www.pandaex.pro/openapi/exchange/BTC_USDT/fulfillment?startDate=1556100483000&endDate=1569342299000&price=218.32&amount=1.215&systemOrderType=10&source=openapi&isHistory=1&page=1&pageSize=10' \
    -H 'Content-Type: application/json; charset=utf-8' \
    -H 'ACCESS-KEY: 9f4a7317921ba663bb80a93c69d717af' \
    -H 'ACCESS-SIGN: 9SlGnF0795bSvOnFK6uKz79jxRPG7Yev303ZAPM6dO8=' \ 
    -H 'ACCESS-TIMESTAMP: 1569805597000' \
    -H 'ACCESS-PASSPHRASE: 123456'
```

**响应**

```javascript
    # Response 
    {

    }
```

**返回值说明**

|返回字段|字段说明|
|----|----|
| id |订单id|
| pairCode |币对，如：BTC_USDT|
| userId |用户id|
| brokerId |业务方ID|
| side |交易方向，买入为buy，卖出为sell|
| entrustPrice |价格|
| amount |委托数量|
| dealAmount |已成交数量|
| quoteAmount |基准币数量，只有在市价买的情况下会用到|
| dealAmount |已成交数量|
| dealQuoteAmount |基准币已成交数量|
| systemOrderType |10:限价委托，11:市价委托|
| status |成交状态，0:未成交 1:部分成交 2:完全成交 3:撤单中 -1:已撤单数量|
| sourceInfo |客户端来源类型，可以为：web, app, android, ios, openapi|
| createOn |创建时间戳，单位为毫秒|
| updateOn |修改时间戳，单位为毫秒|
| symbol |币种代号|
| trunOver |成交金额|
| notStrike |尚未成交的数量|
| averagePrice |平均成交价|
| openAmount |尚未成交的数量|

**请求参数**

|参数名 | 参数类型 | 必填 | 描述 |
|---|----|----|----|
|pairCode|String|是|币对，如：BTC_USDT|
|startDate|Long|否|起始时间戳，单位为毫秒|
|endDate|Long|否|结束时间戳，单位为毫秒|
|price|Decimal|否|价格|
|amount|Decimal|否|数量|
|systemOrderType|Integer|否|10:限价委托，11:市价委托|
|source|String|否|客户端来源类型，可以为：web, app, android, ios, openapi|
|isHistory|Boolean|否|是否为历史完成订单，0 - 不是，1 - 是|
|page|Integer|否|页号，不指定则返回第1页|
|pageSize|Integer|否|每页数量，如果不指定则最多返回300条|

### 11. 获取账单信息

获取币币交易账单。

**请求**

```http
    # Request
    GET /openapi/exchange/bills?startDate=1556100483000&endDate=1569342299000&symbol=ETH&type=10&isHistory=1&page=1&pageSize=10
```

**命令举例**

```shell
    # shell
    curl -i -X GET 'https://www.pandaex.pro/openapi/exchange/bills?startDate=1556100483000&endDate=1569342299000&symbol=ETH&type=10&isHistory=1&page=1&pageSize=10' \
    -H 'Content-Type: application/json; charset=utf-8' \
    -H 'ACCESS-KEY: 9f4a7317921ba663bb80a93c69d717af' \
    -H 'ACCESS-SIGN: 9SlGnF0795bSvOnFK6uKz79jxRPG7Yev303ZAPM6dO8=' \ 
    -H 'ACCESS-TIMESTAMP: 1569805597000' \
    -H 'ACCESS-PASSPHRASE: 123456'
```

**响应**

```javascript
    # Response
    {
        "bills":[

        ],
        "paginate":{
            "page":1,
            "pageSize":10,
            "total":0
        }
    }
```

**返回值说明**

|返回字段 | 字段说明 |
|----|----|
|id|订单ID|
|userId|用户ID|
|brokerId|业务方ID|
|symbol|币种代号|
|type|委托类型，10:限价委托，11:市价委托|
|amount|变换金额|
|assets|币数量，可记入正负两种情况|
|makerTaker|当前用户在交易中的角色，maker或taker|
|fee|手续费|
|referId|账单关联记录ID|
|tradeNo|交易号|
|createOn|创建时间戳，单位为毫秒|
|updateOn|修改时间戳，单位为毫秒|
|page|页号|
|pageSize|每页数量|
|total|条目总数|

**请求参数**  
    
|参数名|参数类型|必填|描述|
|----|---|---|---|
|startDate|Long|否|起始时间戳，单位为毫秒|
|endDate|Long|否|结束时间戳，单位为毫秒|
|symbol|String|否|货币代号|
|type|Integer|否|10:限价委托，11:市价委托|
|isHistory|Boolean|是|是否为历史完成订单，0 - 不是，1 - 是|
|page|Integer|是|页号，不指定则返回第1页|
|pageSize|Integer|是|每页数量，如果不指定则最多返回300条|

### 12. 获取所有账单类型

获取所有账单类型的接口。

**请求**

```http
    # Request
    
    GET /openapi/exchange/billTypes
```

**命令举例**

```shell
    # shell
    curl -i -X GET 'https://www.pandaex.pro/openapi/exchange/billTypes' \
    -H 'Content-Type: application/json; charset=utf-8' \
    -H 'ACCESS-KEY: 9f4a7317921ba663bb80a93c69d717af' \
    -H 'ACCESS-SIGN: 9SlGnF0795bSvOnFK6uKz79jxRPG7Yev303ZAPM6dO8=' \ 
    -H 'ACCESS-TIMESTAMP: 1569805597000' \
    -H 'ACCESS-PASSPHRASE: 123456'
```

**响应**
    
```javascript
    # Reponse

    [
        {
            "code":"BUY",
            "name":"",
            "id":7
        },
        {
            "code":"SELL",
            "name":"",
            "id":8
        },
        ...
    ]
```
    
**返回值说明**
    
|返回字段|字段说明|
|-----|----|
|code|账单类型代号|
|name|账单类型说明|
|id|账单类型编号|

### 13. 获取关注的币对列表

获取关注的币对列表的接口。

**请求**

```http
    # Request
    GET /openapi/exchange/favorite/list
```

**命令举例**

```shell
    # shell
    curl -i -X GET 'https://www.pandaex.pro/openapi/exchange/favorite/list' \
    -H 'Content-Type: application/json; charset=utf-8' \
    -H 'ACCESS-KEY: 9f4a7317921ba663bb80a93c69d717af' \
    -H 'ACCESS-SIGN: 9SlGnF0795bSvOnFK6uKz79jxRPG7Yev303ZAPM6dO8=' \ 
    -H 'ACCESS-TIMESTAMP: 1569805597000' \
    -H 'ACCESS-PASSPHRASE: 123456'
```

**响应**
   
```javascript
    # Response
    [
        "BTC_USDT", 
        "ETH_USDT", 
        ...
    ]
```

**返回值说明**
    
|返回字段|字段说明|
|-----|----|
|BTC_USDT|币对，如：BTC_USDT|
  

[PandaEX]: https://www.pandaex.pro/ 
[English Docs]: https://github.com/pandaex/pandaex-official-docs/blob/master/README_EN.md
[Unix Epoch]: https://en.wikipedia.org/wiki/Unix_time
