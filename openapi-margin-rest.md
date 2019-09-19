# REST行情与交易接口

[TOC]

## 基本信息
- 本篇列出REST接口的baseurl:  http://openapi-exchange.coinbene.com 或 https://openapi-exchange.coinbene.com
- 建议创建完API后，修改添加上自己服务器出口IP，进一步增强API安全性校验
- 所有接口的响应都是JSON格式
- 所有时间、时间戳均为UNIX时间，单位为毫秒
- HTTP 4XX 错误码用于指示错误的请求内容、行为、格式。
- HTTP 429 错误码表示警告访问频次超限，即将被封IP
- HTTP 5XX 错误码用于指示Coinbene服务侧的问题。
- HTTP 504 表示API服务端已经向业务核心提交了请求但未能获取响应，特别需要注意的是504代码不代表请求失败，而是未知。很可能已经得到了执行，也有可能执行失败，需要做进一步确认。
- 每个接口都有可能抛出异常，异常响应格式如下：

```
{
  "code": 10001,
  "msg": "Invalid Paramater."
}
```
- 具体的错误码及其解释在错误代码汇总。
- GET方法的接口, 参数必须在query string中发送。
- POST方法的接口, 参数在 request body中发送(content type application/json)。
- 对参数的顺序不做要求。
## 访问限制
- 当访问接口超过频率限制时，将返回429状态：请求太频繁。
- 限制规则，如果传入有效的API key则用user id限速；如果没有则拿用户的公网IP限速。各个接口上有单独的说明。
## 接口类型
- 主要为两类接口，公共接口和私有接口。
- 公共接口无需认证即可调用。
- 私有接口用户订单和账户。每个私有请求必须使用规范的验证形式进行签名。私有接口需要使用您的API key进行验证。
## 签名方式
所有接口请求头必须包含以下内容：
- ACCESS-KEY 字符串类型的API key
- ACCESS-SIGN 使用Hex生成字符串返回，具体代码参考下面Java版本和Python版本代码
- ACCESS-TIMESTAMP 发起请求的时间戳
- 所有请求都应该含有application/json类型内容，并且是有效的JSON。

ACCESS-SIGN的值生成规则：
- 按照timestamp + method + requestPath + body字符串（+表示字符串连接），以及secret，使用HMAC SHA256方法加密，最后把加密串的字节数组转成字符串返回。
- 其中，timestamp的值与ACCESS-TIMESTAMP请求头相同，必须是UTC时区Unix时间戳的十进制秒数或ISO8601标准的时间格式，精确到毫秒。
- Method是请求方法，字母全部大写：GET/POST
- requestPath是请求接口路径，例如：/api/exchange/v2/market/orderBook
- body是指请求主体的字符串。GET请求没有body信息可省略；POST请求有body信息JSON串，例如{"symbol":"BTCUSDT","order_id":"7440"}
- secket为用户申请API时所生成的。

接口请求样例：
- GET协议接口两种情况: 
```
1. 不带参数：
签名串preHash: 2019-09-11T08:45:47.881ZGET/api/margin/v1/account/list
2. 带参数：
签名串preHash: 2019-09-11T08:48:22.673ZGET/api/margin/v1/account/one?symbol=BTC%2FUSDT
```

```
Url: http://域名/api/margin/v1/account/one?symbol=BTC%2FUSDT
Method: GET
Headers: 
	Accept: application/json
	ACCESS-KEY: 978672ddedbd1c5340a83a277b2ac654
	ACCESS-SIGN: 2f4d7cc66e7e7654f0c38ad7533f32e72c6e78b0be7c74804f0eae6027f1a25d
	ACCESS-TIMESTAMP: 2019-09-11T08:48:22.673Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-09-11T08:48:22.673ZGET/api/margin/v1/account/one?symbol=BTC%2FUSDT
		
```

```
Url: http://域名/api/margin/v1/account/list
Method: GET
Headers: 
	Accept: application/json
	ACCESS-KEY: 978672ddedbd1c5340a83a277b2ac654
	ACCESS-SIGN: 46a5839e90239efafbecc1881f9d0c59a03ec5b7a1755495770a4e6b8d0eaf75
	ACCESS-TIMESTAMP: 2019-09-11T08:45:47.881Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-09-11T08:45:47.881ZGET/api/margin/v1/account/list

```


- POST协议接口情况：
```
签名串preHash: 2019-09-11T09:10:08.238ZPOST/api/margin/v1/account/borrow{"symbol":"BTC/USDT","asset":"BTC","quantity":"1"}
```


```
Url: http://域名/api/margin/v1/account/borrow
Method: POST
Headers: 
	Accept: application/json
	ACCESS-KEY: 978672ddedbd1c5340a83a277b2ac654
	ACCESS-SIGN: cb42134e3542b9507883d779199293e818c63478570af494ab130e44e76ec7a1
	ACCESS-TIMESTAMP: 2019-09-11T09:10:08.238Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: {"symbol":"BTC/USDT","asset":"BTC","quantity":"1"}
preHash: 2019-09-11T09:10:08.238ZPOST/api/margin/v1/account/borrow{"symbol":"BTC/USDT","asset":"BTC","quantity":"1"}

```
- 签名算法验证：


```
源串：2019-09-11T09:11:12.141ZGET/api/margin/v1/account/list
secret：978672ddedbd1c5340a83a277b2ac654
生成sign串：b7c941f76626bc1bf6cfbfd8789d5aed47b221829c25fd6729b7f969abd18718

样例代码（Java版本）：
**
   * 生成签名
   *
   * @param timeStamp   时间戳
   * @param method      请求方法：POST或者GET
   * @param requestUrl  url
   * @param requestBody 请求内容，没有传null
   * @param secret      密钥
   */
  private String signForContractOpenApi(String timeStamp, String method, String requestUrl, String requestBody, String secret) {
    String shaResource = timeStamp + method + requestUrl + (requestBody == null ? "" : requestBody);
    System.out.println(shaResource);
    String signStr = sha256_HMAC(shaResource, secret);
    return signStr;
  }

  /**
   * sha256_HMAC加密
   *
   * @param resource 签名源字符串
   * @param secret   秘钥
   * @return 加密后字符串
   */
  private static String sha256_HMAC(String resource, String secret) {
    String hash = "";
    try {
      Mac sha256_HMAC = Mac.getInstance("HmacSHA256");
      SecretKeySpec secret_key = new SecretKeySpec(secret.getBytes("UTF-8"), "HmacSHA256");
      sha256_HMAC.init(secret_key);
      byte[] bytes = sha256_HMAC.doFinal(resource.getBytes("UTF-8"));
      hash = byteArrayToHexString(bytes);
    } catch (Exception e) {
      System.out.println("Error HmacSHA256 ===========" + e.getMessage());
    }
    return hash;
  }

  /**
   * 将加密后的字节数组转换成字符串
   *
   * @param bytes 字节数组
   * @return 字符串
   */
  private static String byteArrayToHexString(byte[] bytes) {
    StringBuffer buffer = new StringBuffer();
    String stmp;
    for (int index = 0; bytes != null && index < bytes.length; index++) {
      stmp = Integer.toHexString(bytes[index] & 0XFF);
      if (stmp.length() == 1) {
        buffer.append('0');
      }
      buffer.append(stmp);
    }
    return buffer.toString().toLowerCase();
  }

样例代码（Python版本）：

import hashlib
import hmac
import unittest

def sign(message, secret):
    """
    gen sign
    :param message: message wait sign
    :param secret:  secret key
    :return:
    """
    secret = secret.encode('utf-8')
    message = message.encode('utf-8')
    sign = hmac.new(secret, message, digestmod=hashlib.sha256).hexdigest()
    return sign

class TestUtil(unittest.TestCase):
    def test_sign(self):
        sn = sign("2019-05-25T03:20:30.362ZGET/api/spot/v2/account/info", "9daf13ebd76c4f358fc885ca6ede5e27")
        self.assertEqual(sn, "a02a6428bb44ad338d020c55acee9dd40bbcb3d96cbe3e48dd6185e51e232aa2")


```

## 接口规范

### 公共接口-获取全部交易配置信息
```
获取交易所全部币对配置列表
限速规则：5次/2秒
HTTP GET /api/margin/v1/tradePair/list
```
请求参数：
无

返回字段说明：

名称   | 类型  | 说明
---|---|--- 
symbol   | string | 币对名称, 如BTC/USDT
base   | string | 计价货币 BTC
quote   | string | 交易货币 USDT
leverage   | string | 杠杆倍数
pricePrecision   | string | 价格精度
volumePrecision   | string | 数量精度
takerFee   | string | taker手续费
makeFee   | string | maker手续费
minAmount   | string | 最新成交数量
priceChangeScale   | string | 价格波动限制
sellDisabled   | string | 禁止卖单操作，0允许，1禁止
initialPrice   | string | 初始价格
minVolume   | string | 最小数量限制

```
Request:
Url: http://域名/api/margin/v1/tradePair/list
Method: GET
Headers: 
	Accept: application/json
	ACCESS-KEY: 978672ddedbd1c5340a83a277b2ac654
	ACCESS-SIGN: 77e056e11c1ccec7b472bc424df380648f7021e9a9a35260bbb78ca9ce5e33a1
	ACCESS-TIMESTAMP: 2019-09-11T09:11:59.924Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-09-11T09:11:59.924ZGET/api/margin/v1/tradePair/list


Response:
{
    "code":200,
    "data":[
        {
            "symbol":"BTC/USDT",
            "base":"BTC",
            "quote":"USDT",
            "leverage":"5",
            "pricePrecision":"8",
            "makeFee":"0.001",
            "takeFee":"0.001",
            "sellDisabled":"0",
            "minVolume":"0.0001",
            "volumePrecision":"5",
            "initialPrice":"6525.92",
            "baseInterestRate":"0.0002",
            "quoteInterestRate":"0.0002",
            "priceChangeScale":"0.5"
        },
        {
            "symbol":"ETH/USDT",
            "base":"ETH",
            "quote":"USDT",
            "leverage":"5",
            "pricePrecision":"2",
            "makeFee":"0.001",
            "takeFee":"0.001",
            "sellDisabled":"0",
            "minVolume":"0.01",
            "volumePrecision":"2",
            "initialPrice":"328.85",
            "baseInterestRate":"0.0002",
            "quoteInterestRate":"0.0002",
            "priceChangeScale":"0.5"
        }
    ]
}
```


### 公共接口-获取指定交易币对配置信息
```
获取交易所全部币对配置列表
限速规则：5次/2秒
HTTP GET /api/margin/v1/tradePair/one
```
请求参数：

名称|类型|说明
---|---|---
symbol|string|币对名称, 如BTC/USDT

返回字段说明：

名称   | 类型  | 说明
---|---|--- 
symbol   | string | 币对名称
base   | string | 计价货币 BTC
quote   | string | 交易货币 USDT
leverage   | string | 杠杆倍数
pricePrecision   | string | 价格精度
volumePrecision   | string | 数量精度
takerFee   | string | taker手续费
makeFee   | string | maker手续费
minAmount   | string | 最新成交数量
priceChangeScale   | string | 价格波动限制
sellDisabled   | string | 禁止卖单操作，0允许，1禁止
initialPrice   | string | 初始价格
minVolume   | string | 最小数量限制

```
Request:
Url: http://域名/api/margin/v1/tradePair/one
Method: GET
Headers: 
	Accept: application/json
	ACCESS-KEY: 978672ddedbd1c5340a83a277b2ac654
	ACCESS-SIGN: 8e830f05c02cefaffd57a6d64123873302b79c49a0c189b934a264e2374842ff
	ACCESS-TIMESTAMP: 2019-09-11T09:23:59.620Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-09-11T09:23:59.620ZGET/api/margin/v1/tradePair/one?symbol=BTC%2FUSDT


Response:
{
    "code":200,
    "data":{
        "symbol":"BTC/USDT",
        "base":"BTC",
        "quote":"USDT",
        "leverage":"5",
        "pricePrecision":"8",
        "makeFee":"0.001",
        "takeFee":"0.001",
        "sellDisabled":"0",
        "minVolume":"0.0001",
        "volumePrecision":"5",
        "initialPrice":"6525.92",
        "baseInterestRate":"0.0002",
        "quoteInterestRate":"0.0002",
        "priceChangeScale":"0.5"
    }
}
```



### 私有接口-查询全部账户信息

```
获取杠杆用户资产的全部账户信息
限速次数：3次/1秒
HTTP GET /api/margin/v1/account/list
```

请求参数
无

返回结果参数

名称   | 类型  | 说明
---|---|---
symbol   | string | 币对名称
forceClosePrice   | string | 爆仓价格
riskRate   | string | 风险率
asset   | string | 资产名称
available   | string | 可用余额
borrow   | string | 借币数量
frozen   | string | 冻结数量
interest | string | 利息数量

```
Request:
Url: http://域名/api/margin/v1/account/list
Method: GET
Headers: 
	Accept: application/json
	ACCESS-KEY: 978672ddedbd1c5340a83a277b2ac654
	ACCESS-SIGN: f21fff7fa66d41e32016ae0bfaeafacf35f0e5503ef2702859257162c86118ca
	ACCESS-TIMESTAMP: 2019-09-11T09:25:59.362Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-09-11T09:25:59.362ZGET/api/margin/v1/account/list


Response:
{
    "code":200,
    "data":[
        {
            "symbol":"BTC/USDT",
            "forceClosePrice":"0",
            "riskRate":"0",
            "assetList":[
                {
                    "available":"0.00000000",
                    "borrow":"0.00000000",
                    "asset":"BTC",
                    "frozen":"0.00000000",
                    "interest":"0.00000000"
                },
                {
                    "available":"0.00000000",
                    "borrow":"0.00000000",
                    "asset":"USDT",
                    "frozen":"0.00000000",
                    "interest":"0.00000000"
                }
            ]
        },
        {
            "symbol":"ETH/USDT",
            "forceClosePrice":"0",
            "riskRate":"0",
            "assetList":[
                {
                    "available":"0.00000000",
                    "borrow":"0.00000000",
                    "asset":"ETH",
                    "frozen":"0.00000000",
                    "interest":"0.00000000"
                },
                {
                    "available":"0.00000000",
                    "borrow":"0.00000000",
                    "asset":"USDT",
                    "frozen":"0.00000000",
                    "interest":"0.00000000"
                }
            ]
        }
    ]
}
```

### 私有接口-查询指定账户资产信息

```
获取杠杆用户指定资产的账户信息
限速次数：10次/2秒
HTTP GET /api/margin/v1/account/one
```

请求参数

名称   | 类型  | 是否必填  | 说明
---|---|---|---
symbol   | string | 是 | 资产名称/缩写，如BTC

返回结果参数

名称   | 类型  | 说明
---|---|---
symbol   | string | 币对名称
forceClosePrice   | string | 爆仓价格
riskRate   | string | 风险率
asset   | string | 资产名称
available   | string | 可用余额
borrow   | string | 借币数量
frozen   | string | 冻结数量
interest | string | 利息数量

```
Request:
Url: http://域名/api/margin/v1/account/one?symbol=BTC%2FUSDT
Method: GET
Headers: 
	Accept: application/json
	ACCESS-KEY: 978672ddedbd1c5340a83a277b2ac654
	ACCESS-SIGN: eaaf368996c4d6c4afc24f9c64f3d21a52421b13941f01b3e703d3b8c874c198
	ACCESS-TIMESTAMP: 2019-09-11T09:29:07.082Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-09-11T09:29:07.082ZGET/api/margin/v1/account/one?symbol=BTC%2FUSDT


Response:
{
    "code":200,
    "data":{
        "symbol":"BTC/USDT",
        "forceClosePrice":"0",
        "riskRate":"0",
        "assetList":[
            {
                "available":"0.00000000",
                "borrow":"0.00000000",
                "asset":"BTC",
                "frozen":"0.00000000",
                "interest":"0.00000000",
                "interestRate":"0.0002"
            },
            {
                "available":"0.00",
                "borrow":"0.00",
                "asset":"USDT",
                "frozen":"0.00",
                "interest":"0.00",
                "interestRate":"0.0002"
            }
        ]
    }
}
```




### 私有接口-查询指定账户资产最大可借数量

```
获取查询指定账户资产最大可借数量
限速次数：10次/2秒
HTTP GET /api/margin/v1/account/max-borrow
```

请求参数

名称   | 类型  | 是否必填  | 说明
---|---|---|---
symbol   | string | 是 | 币对名称/缩写，如BTC/USDT
asset   | string | 是 | 资产名称/缩写，如BTC

返回结果参数

名称   | 类型  | 说明
---|---|---
max   | string | 最大可借数量

```
Request:
Url: http://域名/api/margin/v1/account/max-borrow?symbol=BTC%2FUSDT&asset=BTC
Method: GET
Headers: 
	Accept: application/json
	ACCESS-KEY: 978672ddedbd1c5340a83a277b2ac654
	ACCESS-SIGN: 87cc372a37fdfa87400aee112c937d8d2fba5e6e0cc75b5695b842fa9f087286
	ACCESS-TIMESTAMP: 2019-09-11T09:32:29.878Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-09-11T09:32:29.878ZGET/api/margin/v1/account/max-borrow?symbol=BTC%2FUSDT&asset=BTC


Response:
{
    "code":200,
    "data":{
        "max":"1"
    }
}
```


### 私有接口-借币

```
获取查询指定账户资产最大可借数量
限速次数：10次/2秒
HTTP POST /api/margin/v1/account/borrow
```

请求参数

名称   | 类型  | 是否必填  | 说明
---|---|---|---
symbol   | string | 是 | 币对名称/缩写，如BTC/USDT
asset   | string | 是 | 资产名称/缩写，如BTC
quantity   | string | 是 | 借币数量

返回结果参数

名称   | 类型  | 说明
---|---|---
data   | boolean| 返回true借币成功，其它失败返回异常code

```
Request:
Url: http://域名/api/margin/v1/account/borrow
Method: POST
Headers: 
	Accept: application/json
	ACCESS-KEY: 978672ddedbd1c5340a83a277b2ac654
	ACCESS-SIGN: ecafb948e5169e9ea76f4dea9b6a571ac99d4a603709d4d1df693d2d5048f4aa
	ACCESS-TIMESTAMP: 2019-09-11T09:33:49.930Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: {"symbol":"BTC/USDT","asset":"BTC","quantity":"1"}
preHash: 2019-09-11T09:33:49.930ZPOST/api/margin/v1/account/borrow{"symbol":"BTC/USDT","asset":"BTC","quantity":"1"}


Response:
{
    "code":200,
    "data":true
}
```


### 私有接口-还币

```
获取查询指定账户资产最大可借数量
限速次数：10次/2秒
HTTP POST /api/margin/v1/account/borrow
```

请求参数

名称   | 类型  | 是否必填  | 说明
---|---|---|---
symbol   | string | 是 | 币对名称/缩写，如BTC/USDT
asset   | string | 是 | 资产名称/缩写，如BTC
quantity   | string | 是 | 借币数量
orderId   | string | 否 | 借币数量

返回结果参数

名称   | 类型  | 说明
---|---|---
data   | boolean| 返回true还币成功，其它失败返回异常code

```
Request:
Url: http://域名/api/margin/v1/account/repayment
Method: POST
Headers: 
	Accept: application/json
	ACCESS-KEY: 978672ddedbd1c5340a83a277b2ac654
	ACCESS-SIGN: 67dd4936e8f8dd4b8e2fd6559e6d393be31aa8d69b14055df88b3811d83b89c3
	ACCESS-TIMESTAMP: 2019-09-11T09:47:40.830Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: {"symbol":"BTC/USDT","asset":"BTC","orderId":"","quantity":"5"}
preHash: 2019-09-11T09:47:40.830ZPOST/api/margin/v1/account/repayment{"symbol":"BTC/USDT","asset":"BTC","orderId":"","quantity":"5"}


Response:
{
    "code":200,
    "data":true
}
```



### 私有接口-下单

```
按用户输入进行下单操作
限速规则：10次/2秒
HTTP POST /api/margin/v1/order/place
```
请求参数：
名称  | 类型  | 是否必填  | 说明
---|---|---|---
symbol      | string | 是 | 币对名称，如BTC/USDT，用"/"分割
accountType      | string | 是 | 固定值 margin
direction      | string | 是 | 方向，buy:买 sell:卖
price      | string | 是 | 下单价格
orderType      | string | 是 | 下单类型，只支持 限价单 limit 
quantity      | string | 是 | 数量
clientId      | string | 否 | 用户请求id，透传返回给用户


```
1.只支持限价单类型
```

返回字段说明：

名称   | 类型  | 说明
---|---|---
orderId   | string | 生成的订单id
clientId   | string | 用户请求的clientId


```
Request:
Url: http://域名/api/margin/v1/order/place
Method: POST
Headers: 
	Accept: application/json
	ACCESS-KEY: 978672ddedbd1c5340a83a277b2ac654
	ACCESS-SIGN: c3c8352fa6c3ce38bbc4b0f83cf5f47b245012919ccdfca2dbc122754b22c491
	ACCESS-TIMESTAMP: 2019-09-11T09:49:30.209Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: {"symbol":"BTC/USDT","price":"18526.0","quantity":"0.01","direction":"sell","clientId":"1568195370181","accountType":"margin","orderType":"limit"}
preHash: 2019-09-11T09:49:30.209ZPOST/api/margin/v1/order/place{"symbol":"BTC/USDT","price":"18526.0","quantity":"0.01","direction":"sell","clientId":"1568195370181","accountType":"margin","orderType":"limit"}



Response:
{
  "code": 200, 
  "data": {
    "orderId": "1911862608898764800", 
    "clientId": "1560332366247"
  }
}
```


### 私有接口-查询当前委托挂单列表

```
按用户请求进行订单列表查询，
限速规则：5次/2秒
HTTP GET /api/margin/v1/order/openOrders
```
请求参数：

名称  | 类型  | 是否必填  | 说明
---|---|---|---
symbol      | string | 否 | 币对名称，如BTC/USDT
latestOrderId      | string | 否 | 订单id，分页使用，默认值为空，返回最新20条数据，按订单id倒排显示。获取最后一个订单id-1，取下一页数据


```
说明：
分页查询，每页返回20条
```


返回字段说明：

名称   | 类型  | 说明
---|---|---
orderId   | string | 订单Id
baseAsset   | string | 基础货币，如BTC
quoteAsset   | string | 交易货币，如USDT
orderDirection   | string | 方向
quantity   | string | 订单数量
filledQuantity   | string | 已成交数量
amount   | string | 订单金额
filledAmount   | string | 已成交金额
avgPrice   | string | 平均价格
orderStatus   | string | 订单状态，未成交：open 完全成交：filled 取消：cancelled 部分成交：partiallyCancelled
orderTime   | string | 下单时间
fee   | string | 手续费


```
Request:
Url: http://域名/api/margin/v1/order/openOrders?symbol=BTC%2FUSDT
Method: GET
Headers: 
	Accept: application/json
	ACCESS-KEY: 978672ddedbd1c5340a83a277b2ac654
	ACCESS-SIGN: 1f893649c758a5c147e44938e7d44a8efc6c7b1af3c3643cea20ade02a37b085
	ACCESS-TIMESTAMP: 2019-09-11T09:59:44.520Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-09-11T09:59:44.520ZGET/api/margin/v1/order/openOrders?symbol=BTC%2FUSDT


Response:
{
  "code": 200, 
  "data": [
    {
      "orderId": "1912119792282841088", 
      "baseAsset": "BTC", 
      "quoteAsset": "USDT", 
      "orderDirection": "buy", 
      "quantity": "53", 
      "amount": "3426.45", 
      "filledAmount": "0", 
      "takerFeeRate": "0.001", 
      "makerFeeRate": "0.001", 
      "avgPrice": "0", 
      "orderStatus": "Open", 
      "orderTime": "2019-06-13T02:41:24.0Z", 
      "totalFee": "0"
    }, 
    {
      "orderId": "1911862608898764800", 
      "baseAsset": "BTC", 
      "quoteAsset": "USDT", 
      "orderDirection": "buy", 
      "quantity": "54", 
      "amount": "3645", 
      "filledAmount": "0", 
      "takerFeeRate": "0.001", 
      "makerFeeRate": "0.001", 
      "avgPrice": "0", 
      "orderStatus": "Open", 
      "orderTime": "2019-06-12T09:39:27.0Z", 
      "totalFee": "0"
    }, 
    {
      "orderId": "1911862547074723840", 
      "baseAsset": "BTC", 
      "quoteAsset": "USDT", 
      "orderDirection": "buy", 
      "quantity": "15", 
      "amount": "966.75", 
      "filledAmount": "0", 
      "takerFeeRate": "0.001", 
      "makerFeeRate": "0.001", 
      "avgPrice": "0", 
      "orderStatus": "Open", 
      "orderTime": "2019-06-12T09:39:12.0Z", 
      "totalFee": "0"
    }
  ]
}
```

### 私有接口-查询历史委托单列表

```
按用户请求进行订单列表查询，
限速规则：5次/2秒
HTTP GET /api/margin/v1/order/closedOrders
```
请求参数：

名称  | 类型  | 是否必填  | 说明
---|---|---|---
symbol      | string | 否 | 币对名称，如BTC/USDT
latestOrderId      | string | 否 | 订单id，分页使用，默认值为空，返回最新20条数据，按订单id倒排显示。获取最后一个订单id-1，取下一页数据

```
说明：
分页查询，每页返回20条
```


返回字段说明：

名称   | 类型  | 说明
---|---|---
orderId   | string | 订单Id
baseAsset   | string | 基础货币，如BTC
quoteAsset   | string | 交易货币，如USDT
orderDirection   | string | 方向
quantity   | string | 订单数量
amount   | string | 订单金额
filledAmount   | string | 已成交金额
takerFeeRate   | string | taker费率
makerFeeRate   | string | maker费率
avgPrice   | string | 平均价格
orderStatus   | string | 订单状态，未成交：open 完全成交：filled 取消：cancelled 部分成交：partially cancelled
orderTime   | string | 下单时间
totalFee   | string | 手续费


```
Request:
Url: http://域名/api/margin/v1/order/closedOrders
Method: GET
Headers: 
	Accept: application/json
	ACCESS-KEY: 978672ddedbd1c5340a83a277b2ac654
	ACCESS-SIGN: d0c6e8cfe818eb263e3860eb0d7261b588a8dc13ebaa5d0799bc2e154d49877b
	ACCESS-TIMESTAMP: 2019-06-13T11:39:45.024Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-06-13T11:39:45.024ZGET/api/exchange/v2/order/closedOrders

Response:
{
  "code": 200, 
  "data": [
    {
      "orderId": "1912131427156307968", 
      "baseAsset": "BTC", 
      "quoteAsset": "USDT", 
      "orderDirection": "buy", 
      "quantity": "63", 
      "amount": "4205.25", 
      "filledAmount": "3979.557", 
      "takerFeeRate": "0.001", 
      "makerFeeRate": "0.001", 
      "avgPrice": "63.1675714285714285714286", 
      "orderStatus": "Filled", 
      "orderTime": "2019-06-13T03:27:38.0Z", 
      "totalFee": "3.979557"
    }, 
    {
      "orderId": "1911862608898764800", 
      "baseAsset": "BTC", 
      "quoteAsset": "USDT", 
      "orderDirection": "sell", 
      "quantity": "54", 
      "amount": "3645", 
      "filledAmount": "0", 
      "takerFeeRate": "0.001", 
      "makerFeeRate": "0.001", 
      "avgPrice": "0", 
      "orderStatus": "Cancelled", 
      "orderTime": "2019-06-12T09:39:27.0Z", 
      "totalFee": "0"
    }
  ]
}
```

### 私有接口-查询指定订单信息

```
按用户请求进行订单列表查询，
限速规则：10次/2秒
HTTP GET /api/margin/v1/order/info
```
请求参数：

名称  | 类型  | 是否必填  | 说明
---|---|---|---
orderId      | string | 是 | 委托单ID


返回字段说明：

名称   | 类型  | 说明
---|---|---
orderId   | string | 订单Id
baseAsset   | string | 基础货币，如BTC
quoteAsset   | string | 交易货币，如USDT
orderDirection   | string | 方向，1：买 2：买
quantity   | string | 订单数量
amount   | string | 订单金额
filledAmount   | string | 已成交金额
takerFeeRate   | string | taker费率
makerFeeRate   | string | maker费率
avgPrice   | string | 平均价格
orderStatus   | string | 订单状态，未成交：open 完全成交：filled 取消：cancelled 部分成交：partially cancelled
orderTime   | string | 下单时间
totalFee   | string | 手续费

```
Request:
Url: http://域名/api/margin/v1/order/info
Method: GET
Headers: 
	Accept: application/json
	ACCESS-KEY: 978672ddedbd1c5340a83a277b2ac654
	ACCESS-SIGN: 6ff0c1912a00f188f92189c66817b20c731324ea560e7aed63971292b5af89d7
	ACCESS-TIMESTAMP: 2019-09-11T09:58:45.540Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-09-11T09:58:45.540ZGET/api/margin/v1/order/info?orderId=1911862608898764800

Response:
{
  "code": 200, 
  "data": {
    "orderId": "1911862608898764800", 
    "baseAsset": "BTC", 
    "quoteAsset": "USDT", 
    "orderDirection": "2", 
    "quantity": "54", 
    "amount": "3645", 
    "filledAmount": "0", 
    "takerFeeRate": "0.001", 
    "makerFeeRate": "0.001", 
    "orderStatus": "Cancelled", 
    "orderTime": "2019-06-12T09:39:27.0Z", 
    "totalFee": "0"
  }
}

```


### 私有接口-查询订单成交明细列表

```
按用户请求进行订单列表查询，
限速规则：5次/2秒
HTTP GET /api/margin/v1/order/trade/fills
```
请求参数：

名称  | 类型  | 是否必填  | 说明
---|---|---|---
orderId      | string | 是 | 委托单ID


返回字段说明：

名称   | 类型  | 说明
---|---|---
price   | string | 交易价格
quantity   | string | 交易数量
amount   | string | 交易金额
fee   | string | 手续费
direction   | string | 方向
tradeTime   | string | 订单交易时间，国际时间
feeByConi   | string | coni抵扣手续

```
Request:
Url: http://域名/api/margin/v1/order/trade/fills?orderId=1938306197706977280
Method: GET
Headers: 
	Accept: application/json
	ACCESS-KEY: 978672ddedbd1c5340a83a277b2ac654
	ACCESS-SIGN: c20a4cbc3b8ff17709d653f27ea46d28755612f50e4e76e87c2497a78bcc7b32
	ACCESS-TIMESTAMP: 2019-09-11T09:57:06.065Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-09-11T09:57:06.065ZGET/api/margin/v1/order/trade/fills?orderId=1938306197706977280

Response:
{
  "code": 200, 
  "data": [
    {
      "price": "63.16", 
      "quantity": "1.2", 
      "amount": "75.792", 
      "fee": "0.075792", 
      "direction": "buy", 
      "tradeTime": "2019-06-13T03:27:38.0Z", 
      "feeByConi": ""
    }, 
    {
      "price": "63.16", 
      "quantity": "1.2", 
      "amount": "75.792", 
      "fee": "0.075792", 
      "direction": "buy", 
      "tradeTime": "2019-06-13T03:27:38.0Z", 
      "feeByConi": ""
    }, 
    {
      "price": "63.16", 
      "quantity": "1.2", 
      "amount": "75.792", 
      "fee": "0.075792", 
      "direction": "buy", 
      "tradeTime": "2019-06-13T03:27:38.0Z", 
      "feeByConi": ""
    }, 
    {
      "price": "63.16", 
      "quantity": "1.1", 
      "amount": "69.476", 
      "fee": "0.069476", 
      "direction": "buy", 
      "tradeTime": "2019-06-13T03:27:38.0Z", 
      "feeByConi": ""
    }
  ]
}
```




### 私有接口-撤销指定委托单

```
按用户请求进行订单列表查询，
限速规则：10次/2秒
HTTP POST /api/margin/v1/order/cancel
```
请求参数：

名称  | 类型  | 是否必填  | 说明
---|---|---|---
orderId      | string | 是 | 委托单ID

返回字段说明：

名称   | 类型  | 说明
---|---|---
data   | string | 撤销的订单Id

```
Request:
Url: http://域名/api/margin/v1/order/cancel
Method: POST
Headers: 
	Accept: application/json
	ACCESS-KEY: 978672ddedbd1c5340a83a277b2ac654
	ACCESS-SIGN: 6f6dcb795ec22168f6c7c6047051eef2c899746e1f34b93f64e6cd9a4faa5a5a
	ACCESS-TIMESTAMP: 2019-09-11T09:55:13.973Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: {"orderId":"1938306197706977280"}
preHash: 2019-09-11T09:55:13.973ZPOST/api/margin/v1/order/cancel{"orderId":"1938306197706977280"}


Response:
{
  "code": 200, 
  "data": "1938306197706977280"
}
```


### 私有接口-未还清借币单列表

```
按用户请求进行订单列表查询，
限速规则：5次/2秒
HTTP GET /api/margin/v1/account/unRepayOrderList
```
请求参数：

名称  | 类型  | 是否必填  | 说明
---|---|---|---
symbol      | string | 否 | 币对名称 如BTC/USDT
asset      | string | 否 | 资产名称 如BTC
latestBorrowId      | string | 否 | 借币订单ID

返回字段说明：

名称   | 类型  | 说明
---|---|---
borrowId   | string | 借币单Id
asset   | string | 资产名
borrowQuantity   | string | 借币数量
interest   | string | 借币利息
repayQuantity   | string | 已还本金数量
repayInterest   | string | 已还利息数量

```
Request:
Url: http://域名/api/margin/v1/account/unRepayOrderList
		Method: GET
		Headers: 
			Accept: application/json
			ACCESS-KEY: 97013a9d43cad8f31d5b385a788402de
			ACCESS-SIGN: ac4a457cb7ae1dc24219c6bbf514b77d019364eed9414481f1e6ca693c2ff371
			ACCESS-TIMESTAMP: 2019-09-15T09:55:19.380Z
			Content-Type: application/json; charset=UTF-8
			Cookie: locale=zh_CN
		Body: 
		preHash: 2019-09-15T09:55:19.380ZGET/api/margin/v1/account/unRepayOrderList?symbol=BTC%2FUSDT&asset=BTC&latestBorrowId=10899


Response:
{
    "code":200,
    "data":[
        {
            "borrowId":"10898",
            "asset":"BTC",
            "borrowQuantity":"0.01",
            "interest":"0.00000009",
            "repayQuantity":"0",
            "repayInterest":"0"
        },
        {
            "borrowId":"10897",
            "asset":"BTC",
            "borrowQuantity":"0.01",
            "interest":"0.00000009",
            "repayQuantity":"0",
            "repayInterest":"0"
        },
        {
            "borrowId":"10885",
            "asset":"BTC",
            "borrowQuantity":"1",
            "interest":"0.00000834",
            "repayQuantity":"0.99999166",
            "repayInterest":"0.00000833"
        }
    ]
}
```

### 私有接口-还还清借币单列表

```
按用户请求进行订单列表查询，
限速规则：5次/2秒
HTTP GET /api/margin/v1/account/finishRepayOrderList
```
请求参数：

名称  | 类型  | 是否必填  | 说明
---|---|---|---
symbol      | string | 否 | 币对名称 如BTC/USDT
asset      | string | 否 | 资产名称 如BTC
latestBorrowId      | string | 否 | 借币订单ID

返回字段说明：

名称   | 类型  | 说明
---|---|---
borrowId   | string | 借币单Id
asset   | string | 资产名
borrowQuantity   | string | 借币数量
interest   | string | 借币利息
repayQuantity   | string | 已还本金数量
repayInterest   | string | 已还利息数量

```
Request:
Url: http://域名/api/margin/v1/account/finishRepayOrderList
Method: GET
Headers: 
	Accept: application/json
	ACCESS-KEY: 97013a9d43cad8f31d5b385a788402de
	ACCESS-SIGN: c4b0e9cb04141d51acae1e8d2a17c8b1a7a105146124fc01677168aa6b29d6c4
	ACCESS-TIMESTAMP: 2019-09-15T10:00:19.829Z
	Content-Type: application/json; charset=UTF-8
	Cookie: locale=zh_CN
Body: 
preHash: 2019-09-15T10:00:19.829ZGET/api/margin/v1/account/finishRepayOrderList?symbol=BTC%2FUSDT&asset=BTC&latestBorrowId=


Response:
{
    "code":200,
    "data":[
        {
            "borrowId":"10884",
            "asset":"BTC",
            "borrowQuantity":"1",
            "interest":"0.00000833",
            "repayQuantity":"1",
            "repayInterest":"0.00000833"
        }
    ]
}
```
