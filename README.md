Bihuex交易所官方API文档
==================================================
[Bihuex][]交易所开发者文档([English Docs][])。

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
- [币币业务API参考](#币币业务api参考)
    - [币币行情API](#币币行情api)
        - [1. 获取所有币对列表](#1-获取所有币对列表)
        - [2. 获取币对交易深度](#2-获取币对交易深度)
    - [币币账户API](#币币账户api)
        - [1. 获取账户信息](#1-获取账户信息)
        - [2. 交易委托](#2-交易委托)
        - [3. 撤销所有委托](#3-撤销所有委托)
        - [4. 按订单撤销委托](#4-按订单撤销委托)
        - [5. 查询所有订单](#5-查询所有订单)
        - [6. 按id查询订单](#6-按id查询订单)
        - [7. 获取账单](#7-获取账单)

<!-- /TOC -->

# 介绍

欢迎使用[Bihuex][]开发者文档。

本文档提供了币币交易业务的账户管理、行情查询、交易功能等相关API的使用方法介绍。
行情API提供市场的公开的行情数据接口，账户和交易API需要身份验证，提供下单、撤单，查询订单和帐户信息等功能。

# 开始使用    
REST，即Representational State Transfer的缩写，是一种流行的互联网传输架构。它具有结构清晰、符合标准、易于理解、扩展方便的，正得到越来越多网站的采用。其优点如下：

+ 在RESTful架构中，每一个URL代表一种资源；
+ 客户端和服务器之间，传递这种资源的某种表现层；
+ 客户端通过四个HTTP指令，对服务器端资源进行操作，实现“表现层状态转化”。

建议开发者使用REST API进行币币交易或者资产提现等操作。

# API接口加密验证
## 生成API Key

在对任何请求进行签名之前，您必须通过 Bihuex 网站【用户中心】-【API】创建一个API key。 创建key后，您将获得2个必须记住的信息：
* access Key

* Secret Key


API Key 和 Secret Key将由随机生成和提供。

## 发起请求

所有REST请求都必须包含以下参数：

* accessKey API KEY作为一个字符串。
* sign 使用一定算法得出的签名（请参阅签名信息）。
* timestamp 作为您的请求的时间戳。
* 所有请求都应该含有application/json类型内容，并且是有效的JSON。

## 签名
sign 参数是对 **所有参数(含timestamp)按照字典排序之后，按照key1=value1+ key2=value2 ... + Secret Key** 字符串(+表示字符串连接)使用 **HMAC SHA256** 方法加密而得到的。

* method 是请求方法(POST/GET/PUT/DELETE)，字母全部大写。


**例如：对于如下的请求参数进行签名**

```bash
curl "https://bihuex.com/api-web/api/v2/user/getAllBalance" 
      
```
* 获取获取用户余额信息，以 accessKey= abckey 为例
```java
timestamp = 1540286290170 
accessKey= ABC
```

生成待签名的字符串

```
originString = 'accessKey=ABCtimestamp=1540286290170'
  
```

然后，将待签名字符串添加私钥参数生成最终待签名字符串。



例如：
```
Signature = HmacSHA256(secretkey, originString)

```

## 请求交互  

REST访问的根URL：`https://www.bihuex.com`

### 请求

所有请求基于Https协议，请求头信息中Content-Type 需要统一设置为:'application/json’。

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

* 404 Not Found 没有找到请求

* 429 Too Many Requests 请求太频繁被系统限流

* 500 Internal Server Error – We had a problem with our server 服务器内部错误

* 如果失败，response body 带有错误描述信息
### 分页

部分返回数据集的REST请求支持使用游标分页。
游标分页允许在结果的当前页面之前和之后获取结果，并且非常适合于实时数据。根据当前的返回结果，后续请求可以在此基础之上指定请求数据的方向，可以请求在这之前和之后的数据。before和after游标可通过响应头CB_BEFORE和CB_AFTER使用。

**例子**

`GET /orders?before=2&limit=30`

## 标准规范

### 时间戳

除非另外指定，API中的所有时间戳均以微秒为单位返回。

请求的时间戳必须在API服务时间的30秒内，否则请求将被视为过期并被拒绝。如果本地服务器时间和API服务器时间之间存在较大的偏差，那么我们建议您使用通过查询API服务器时间来更新http header。

### 例子

1524801032573

### 数字

为了保持跨平台时精度的完整性，十进制数字作为字符串返回。建议您在发起请求时也将数字转换为字符串以避免截断和精度错误。 

整数（如交易编号和顺序）不加引号。

### 限流

如果请求过于频繁系统将自动限制请求，//  TODO。

##### REST API

* 公共接口：我们通过IP限制公共接口的调用：每2秒最多6个请求。

* 私人接口：我们通过用户ID限制私人接口的调用：每2秒最多6个请求。

* 某些接口的特殊限制在具体的接口上注明

# 币币业务API参考

## 币币行情API

### 1. 获取所有币对信息

**HTTP请求**

```http
    # Request
    GET api-web/api/v2/market
    
    example： https://api.bihuex.com/api-web/api/v2/market
```
```javascript
    # Response
   {
   	"error": 0,
   	"msg": "",
   	"data": [{
   		"market": "LTC_ETH",
   		"amountPrecision": 5,
   		"pricePrecision": 8,
   		"feeRate": "0.00200",
   		"tradeMinLimit": "0.01000"
   	}, {
   		"market": "ETH_BTC",
   		"amountPrecision": 8,
   		"pricePrecision": 8,
   		"feeRate": "0.00200000",
   		"tradeMinLimit": "0.00100000"
   	}],
   	
   	...
   	
   } 
```



**返回值说明**  


|返回字段 | 字段说明|
| ----------|:-------:|
| error            | 是否有错误信息，0为正常，1为有错误|
| market   |交易市场，以A_B的形式返回  |
| amountPrecision  | 数量精度，即A的小数点位数 |
| pricePrecision   | 价格精度，即B的小数点位数 |
| feeRate   | 交易手续费率 |
| tradeMinLimit | 最小报价单位 |

### 2. 获取币对交易深度

    获取币对盘口深度的请求列表。

**HTTP请求**

```http
    # Request
    GET api/v2/ticker/getDepth
```

**请求参数**  


| 参数名 | 参数类型  | 必填 | 描述 |
| ------------- |----|----|----|
| market | String | 是 | 币对, 如 btc_usdt |


```javascript
    # Response
  {
  	"error": 0,
  	"msg": "",
  	"data": {
  		"btc_usdt": {
  			"sell": [
  				[5319.94, 0.05483456],
  				[5320.19, 1.05734545],
  				[5320.39, 1.16307999],
  				[5320.64, 1.27938799],
  				[5320.88, 1.40732679],
  				[5321.08, 1.54805947],
  				[5321.31, 1.70286542],
  				[5321.53, 1.87315196],
  				[5321.73, 2.06046716],
  				[5321.96, 2.26651388]
  			],
  			"buy": [
  				[5319.46, 1.02423587],
  				[5319.23, 1.12665946],
  				[5318.99, 1.23932541],
  				[5318.79, 1.36325795],
  				[5318.54, 1.49958375],
  				[5318.29, 1.64954213],
  				[5318.08, 1.81449634],
  				[5317.87, 0.0012],
  				[5317.86, 1.99594597],
  				[5317.61, 2.19554057]
  			]
  		}
  	}
  } 
```
**返回值说明**  


|返回字段|字段说明|  
| ------------- |----|
| sell | 卖方深度 |
| buy | 买方深度 |


  

[Bihuex]: https://www.bihuex.com 
[English Docs]: https://www.bihuex.com
[Unix Epoch]: https://en.wikipedia.org/wiki/Unix_time
