# Explanation of coinbene-margin-rest lever OpenAPI rest interface
[中文版本](https://github.com/Coinbene/API-Margin-Documents/blob/master/openapi-margin-rest.md)


   * [Basic Information](#basic-information)
   * [restriction of visit](#restriction-of-visit)
   * [Interface Type](#interface-type)
   * [Signature](#signature)
   * [Interface Specification](#interface-specification)
      * [Public Interface - Get all transaction configuration information](#public-interface---get-all-transaction-configuration-information)
      * [Public interface - Get the specified transaction currency pair configuration information](#public-interface---get-the-specified-transaction-currency-pair-configuration-information)
      * [Private Interface - Query all account information](#private-interface---query-all-account-information)
      * [Private Interface - Query specified account asset information](#private-interface---query-specified-account-asset-information)
      * [Private Interface - Query the maximum loanable amount of the specified account asset](#private-interface---query-the-maximum-loanable-amount-of-the-specified-account-asset)
      * [Private Interface - Borrowing Coins](#private-interface---borrowing-coins)
      * [Private Interface - Coin repayment](#private-interface---coin-repayment)
      * [Private Interface - Order](#private-interface---order)
      * [Private Interface - Query the current list of delegate orders](#private-interface---query-the-current-list-of-delegate-orders)
      * [Private Interface - Query History Order List](#private-interface---query-history-order-list)
      * [Private Interface - Query specified order information](#private-interface---query-specified-order-information)
      * [Private Interface - Query Order Transactions List](#private-interface---query-order-transactions-list)
      * [Private Interface - Undo the specified order](#private-interface---undo-the-specified-order)
      * [Private Interface - Unpaid Loan List](#private-interface---unpaid-loan-list)
      * [Private Interface - Also pay off the list of bills](#private-interface---also-pay-off-the-list-of-bills)
         
* [Error Code Summary](#error-code-summary)

## Basic Information
- This section lists the baseurl for the REST interface: https://openapi-exchange.coinbene.com
- It is recommended to add the own server export IP after modifying the API to further enhance the API security check.
- The response of all interfaces is in JSON format
- All time and timestamp are UNIX time in milliseconds
- The HTTP 4XX error code is used to indicate the content, behavior, and format of the error.
- HTTP 429 error code indicates warning access frequency is exceeded, IP will be blocked
- The HTTP 5XX error code is used to indicate problems on the Coinbene service side.
- HTTP 504 indicates that the API server has submitted a request to the business core but failed to get a response. It is important to note that the 504 code does not represent a request failure, but is unknown. It is very likely that it has already been executed, and it is possible that the execution will fail and further confirmation is needed.
- Each interface may throw an exception. The exception response format is as follows:

```
{
  "code": 10001,
  "msg": "Invalid Paramater."
}
```
- The specific error code and its explanation are summarized in the error code.
- The interface of the GET method, the parameter must be sent in the query string.
- The interface of the POST method, the parameter is sent in the request body (content type application/json).
- No requirements are required for the order of the parameters.
## restriction of visit
- When the access interface exceeds the frequency limit, it will return a 429 status: the request is too frequent.
- Restrict the rule. If a valid API key is passed, the user id is used to limit the rate; if not, the user's public IP address is used to limit the speed. There are separate instructions on each interface.
## Interface Type
- Mainly two types of interfaces, public and private.
- The public interface can be called without authentication.
- Private interface user orders and accounts. Each private request must be signed using a canonical form of authentication. The private interface needs to be verified with your API key.
## Signature
All interface request headers must contain the following:
- ACCESS-KEY string type API key
- ACCESS-SIGN uses Hex to generate a string return. The specific code refers to the following Java version and Python version code.
- ACCESS-TIMESTAMP timestamp of the request
- All requests should contain application/json type content and be valid JSON.

ACCESS-SIGN value generation rules:
- Follow the timestamp + method + requestPath + body string (+ for string concatenation), and secret, encrypt using the HMAC SHA256 method, and finally return the byte array of the encrypted string to a string.
- The value of timestamp is the same as the ACCESS-TIMESTAMP request header. It must be the decimal time of the UTC time zone Unix timestamp or the ISO8601 standard time format, accurate to the millisecond.
- Method is the request method, all letters are capitalized: GET/POST
- requestPath is the request interface path, for example: /api/exchange/v2/market/orderBook
- body is the string of the request body. The GET request has no body information to omit; the POST request has a body information JSON string, such as {"symbol": "BTCUSDT", "order_id": "7440"}
- secket is generated when the user applies for the API.

Sample interface request:
- GET protocol interface in two cases:
```
1. Without parameters:
Signature string preHash: 2019-09-11T08:45:47.881ZGET/api/margin/v1/account/list
2. With parameters:
Signature string preHash: 2019-09-11T08:48:22.673ZGET/api/margin/v1/account/one?symbol=BTC%2FUSDT
```

```
Url: http://domain/api/margin/v1/account/one?symbol=BTC%2FUSDT
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
Url: http://domain/api/margin/v1/account/list
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


- POST protocol interface situation:
```
Signature string preHash: 2019-09-11T09:10:08.238ZPOST/api/margin/v1/account/borrow{"symbol":"BTC/USDT","asset":"BTC","quantity":"1" }
```


```
Url: http://domain/api/margin/v1/account/borrow
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
- Signature algorithm verification:


```
Source string: 2019-09-11T09:11:12.141ZGET/api/margin/v1/account/list
Secret:978672ddedbd1c5340a83a277b2ac654
Generate a string of characters: 5aa0c737234f837f06cfc15a42dad7b64d781ce468f85c8b3e7deb24054b8715

Sample code (Java version):
**
   * Generate signature
   *
   * @param timeStamp timestamp
   * @param method Request method: POST or GET
   * @param requestUrl url
   * @param requestBody request content, no null passed
   * @param secret key
   */
  Private String signForContractOpenApi(String timeStamp, String method, String requestUrl, String requestBody, String secret) {
    String shaResource = timeStamp + method + requestUrl + (requestBody == null ? "" : requestBody);
    System.out.println(shaResource);
    String signStr = sha256_HMAC(shaResource, secret);
    Return signStr;
  }

  /**
   * sha256_HMAC encryption
   *
   * @param resource signature source string
   * @param secret key
   * @return Encrypted string
   */
  Private static String sha256_HMAC(String resource, String secret) {
    String hash = "";
    Try {
      Mac sha256_HMAC = Mac.getInstance("HmacSHA256");
      SecretKeySpec secret_key = new SecretKeySpec(secret.getBytes("UTF-8"), "HmacSHA256");
      sha256_HMAC.init(secret_key);
      Byte[] bytes = sha256_HMAC.doFinal(resource.getBytes("UTF-8"));
      Hash = byteArrayToHexString(bytes);
    } catch (Exception e) {
      System.out.println("Error HmacSHA256 ===========" + e.getMessage());
    }
    Return hash;
  }

  /**
   * Convert the encrypted byte array to a string
   *
   * @param bytes byte array
   * @return string
   */
  Private static String byteArrayToHexString(byte[] bytes) {
    StringBuffer buffer = new StringBuffer();
    String stmp;
    For (int index = 0; bytes != null && index < bytes.length; index++) {
      Stmp = Integer.toHexString(bytes[index] & 0XFF);
      If (stmp.length() == 1) {
        Buffer.append('0');
      }
      Buffer.append(stmp);
    }
    Return buffer.toString().toLowerCase();
  }

Sample code (Python version):

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
        sn = sign("2019-05-25T03:20:30.362ZGET/api/swap/v2/account/info", "9daf13ebd76c4f358fc885ca6ede5e27")
        self.assertEqual(sn, "a02a6428bb44ad338d020c55acee9dd40bbcb3d96cbe3e48dd6185e51e232aa2")


```

## Interface Specification

### Public Interface - Get all transaction configuration information
```
Get the exchange currency pair configuration list
Speed ​​limit rule: 2 times / 1 seconds
HTTP GET /api/margin/v1/tradePair/list
```
Request parameters:
no

Return field description:

Name | Type | Description
---|---|---
symbol | string | currency pair name, such as BTC/USDT
base | string | Trading currency BTC
quote | string | pricing Currency USDT
leverage | string | leverage multiple
pricePrecision | string | Price accuracy
volumePrecision | string | quantity accuracy
takerFee | string | taker fee
makeFee | string | maker fee
minAmount | string | Latest Lots
priceChangeScale | string | Price fluctuation limit
sellDisabled | string | prohibit sell order operation, 0 allowed, 1 prohibited
initialPrice | string | Initial price
minVolume | string | minimum number limit

```
Request:
Url: http://domain/api/margin/v1/tradePair/list
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
    "code": 200,
    "data":[
        {
            "symbol": "BTC/USDT",
            "base": "BTC",
            "quote": "USDT",
            "leverage": "5",
            "pricePrecision":"8",
            "makeFee": "0.001",
            "takeFee": "0.001",
            "sellDisabled": "0",
            "minVolume": "0.0001",
            "volumePrecision": "5",
            "initialPrice": "6525.92",
            "baseInterestRate": "0.0002",
            "quoteInterestRate":"0.0002",
            "priceChangeScale": "0.5"
        },
        {
            "symbol": "ETH/USDT",
            "base": "ETH",
            "quote": "USDT",
            "leverage": "5",
            "pricePrecision":"2",
            "makeFee": "0.001",
            "takeFee": "0.001",
            "sellDisabled": "0",
            "minVolume": "0.01",
            "volumePrecision":"2",
            "initialPrice": "328.85",
            "baseInterestRate": "0.0002",
            "quoteInterestRate":"0.0002",
            "priceChangeScale": "0.5"
        }
    ]
}
```


### Public interface - Get the specified transaction currency pair configuration information
```
Get the exchange currency pair configuration list
Speed ​​limit rule: 3 times / 2 seconds
HTTP GET /api/margin/v1/tradePair/one
```
Request parameters:

Name|type|description
---|---|---
symbol|string|coin pair name, such as BTC/USDT

Return field description:

Name | Type | Description
---|---|---
symbol | string | currency pair name
base | string | Trading currency BTC
quote | string | pricing Currency USDT
leverage | string | leverage multiple
pricePrecision | string | Price accuracy
volumePrecision | string | quantity accuracy
takerFee | string | taker fee
makeFee | string | maker fee
minAmount | string | Latest Lots
priceChangeScale | string | Price fluctuation limit
sellDisabled | string | prohibit sell order operation, 0 allowed, 1 prohibited
initialPrice | string | Initial price
minVolume | string | minimum number limit

```
Request:
Url: http://domain/api/margin/v1/tradePair/one
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
    "code": 200,
    "data":{
        "symbol": "BTC/USDT",
        "base": "BTC",
        "quote": "USDT",
        "leverage": "5",
        "pricePrecision":"8",
        "makeFee": "0.001",
        "takeFee": "0.001",
        "sellDisabled": "0",
        "minVolume": "0.0001",
        "volumePrecision": "5",
        "initialPrice": "6525.92",
        "baseInterestRate": "0.0002",
        "quoteInterestRate":"0.0002",
        "priceChangeScale": "0.5"
    }
}
```

### Private Interface - Query all account information

```
Get all account information for leveraged user assets
Speed ​​limit: 3 times / 1 second
HTTP GET /api/margin/v1/account/list
```

Request parameter
no

Return result parameter

Name | Type | Description
---|---|---
symbol | string | currency pair name
forceClosePrice | string |
riskRate | string | risk ratio
asset | string | asset name
available | string | available balance
borrow | string |
frozen | string | frozen quantity
interest | string | interest amount

```
Request:
Url: http://domain/api/margin/v1/account/list
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
    "code": 200,
    "data":[
        {
            "symbol": "BTC/USDT",
            "forceClosePrice": "0",
            "riskRate": "0",
            "assetList":[
                {
                    "available":"0.00000000",
                    "borrow": "0.00000000",
                    "asset": "BTC",
                    "frozen": "0.00000000",
                    "interest": "0.00000000"
                },
                {
                    "available":"0.00000000",
                    "borrow": "0.00000000",
                    "asset": "USDT",
                    "frozen": "0.00000000",
                    "interest": "0.00000000"
                }
            ]
        },
        {
            "symbol": "ETH/USDT",
            "forceClosePrice": "0",
            "riskRate": "0",
            "assetList":[
                {
                    "available":"0.00000000",
                    "borrow": "0.00000000",
                    "asset": "ETH",
                    "frozen": "0.00000000",
                    "interest": "0.00000000"
                },
                {
                    "available":"0.00000000",
                    "borrow": "0.00000000",
                    "asset": "USDT",
                    "frozen": "0.00000000",
                    "interest": "0.00000000"
                }
            ]
        }
    ]
}
```

### Private Interface - Query specified account asset information

```
Obtain account information for leveraged user-specified assets
Speed ​​limit: 3 times / 1 seconds
HTTP GET /api/margin/v1/account/one
```

Request parameter

Name | Type | Required | Description
---|---|---|---
Symbol | string | yes | asset name/abbreviation, such as BTC


Return result parameter

Name | Type | Description
---|---|---
symbol | string | currency pair name
forceClosePrice | string |
riskRate | string | risk ratio
asset | string | asset name
available | string | available balance
borrow | string |
frozen | string | frozen quantity
interest | string | interest amount

```
Request:
Url: http://domain/api/margin/v1/account/one?symbol=BTC%2FUSDT
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
    "code": 200,
    "data":{
        "symbol": "BTC/USDT",
        "forceClosePrice": "0",
        "riskRate": "0",
        "assetList":[
            {
                "available":"0.00000000",
                "borrow": "0.00000000",
                "asset": "BTC",
                "frozen": "0.00000000",
                "interest": "0.00000000",
                "interestRate":"0.0002"
            },
            {
                "available": "0.00",
                "borrow": "0.00",
                "asset": "USDT",
                "frozen": "0.00",
                "interest": "0.00",
                "interestRate":"0.0002"
            }
        ]
    }
}
```

### Private Interface - Query the maximum loanable amount of the specified account asset

```
Get the maximum loanable amount of the specified account asset
Speed ​​limit: 3 times / 1 seconds
HTTP GET /api/margin/v1/account/max-borrow
```

Request parameter

Name | Type | Required | Description
---|---|---|---
symbol | string | yes | currency pair name/abbreviation, such as BTC/USDT
asset | string | yes | asset name/abbreviation, such as BTC

Return result parameter

Name | Type | Description
---|---|---
max | string | maximum borrowable quantity

```
Request:
Url: http://domain/api/margin/v1/account/max-borrow?symbol=BTC%2FUSDT&asset=BTC
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
    "code": 200,
    "data":{
        "max": "1"
    }
}
```


### Private Interface - Borrowing Coins

```
Get the maximum loanable amount of the specified account asset
Speed ​​limit: 3 times / 1 seconds
HTTP POST /api/margin/v1/account/borrow
```

Request parameter

Name | Type | Required | Description
---|---|---|---
symbol | string | yes | currency pair name/abbreviation, such as BTC/USDT
asset | string | yes | asset name/abbreviation, such as BTC
quantity | string | yes |

Return result parameter

Name | Type | Description
---|---|---
data | boolean| Returns true to borrow success, other failures return exception code

```
Request:
Url: http://domain/api/margin/v1/account/borrow
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
    "code": 200,
    "data": true
}
```


### Private Interface - Coin repayment

```
Get the maximum loanable amount of the specified account asset
Speed ​​limit: 5 times / 1 seconds
HTTP POST /api/margin/v1/account/repayment
```

Request parameter

Name | Type | Required | Description
---|---|---|---
symbol | string | yes | currency pair name/abbreviation, such as BTC/USDT
asset | string | yes | asset name/abbreviation, such as BTC
quantity | string | yes |
orderId | string | no |

Return result parameter

Name | Type | Description
---|---|---
data | boolean| Returns true coin return success, other failures return exception code

```
Request:
Url: http://domain/api/margin/v1/account/repayment
Method: POST
Headers:
Accept: application/json
ACCESS-KEY: 978672ddedbd1c5340a83a277b2ac654
ACCESS-SIGN: 67dd4936e8f8dd4b8e2fd6559e6d393be31aa8d69b14055df88b3811d83b89c3
ACCESS-TIMESTAMP: 2019-09-11T09:47:40.830Z
Content-Type: application/json; charset=UTF-8
Cookie: locale=zh_CN
Body: {"symbol":"BTC/USDT","asset":"BTC","orderId":"","quantity":"5"}
preHash: 2019-09-11T09:47:40.830ZPOST/api/margin/v1/account/repayment{"symbol":"BTC/USDT","asset":"BTC","orderId":"","quantity ":"5"}


Response:
{
    "code": 200,
    "data": true
}
```

### Private Interface - Order

```
Order by user input
Speed ​​limit rule: 5 times / 1 seconds
HTTP POST /api/margin/v1/order/place
```
Request parameters:

Name | Type | Required | Description
---|---|---|---
symbol | string | yes | currency pair name, such as BTC/USDT, separated by "/"
accountType | string | yes | fixed value margin
direction | string | yes | direction, buy: buy sell: sell
price | string | yes | order price
orderType | string | yes | order type, only limit order limit
quantity | string | yes | quantity
clientId | string | no | user request id, transparently returned to the user


```
1. Only limit order types are supported.
```

Return field description:

Name | Type | Description
---|---|---
orderId | string | generated order id
clientId | string | clientId requested by client


```
Request:
Url: http://domain/api/margin/v1/order/place
Method: POST
Headers:
Accept: application/json
ACCESS-KEY: 978672ddedbd1c5340a83a277b2ac654
ACCESS-SIGN: c3c8352fa6c3ce38bbc4b0f83cf5f47b245012919ccdfca2dbc122754b22c491
ACCESS-TIMESTAMP: 2019-09-11T09:49:30.209Z
Content-Type: application/json; charset=UTF-8
Cookie: locale=zh_CN
Body: {"symbol":"BTC/USDT","price":"18526.0","quantity":"0.01","direction":"sell","clientId":"1568195370181","accountType":" Margin","orderType":"limit"}
preHash: 2019-09-11T09:49:30.209ZPOST/api/margin/v1/order/place{"symbol":"BTC/USDT","price":"18526.0","quantity":"0.01"," Direction":"sell","clientId":"1568195370181","accountType":"margin","orderType":"limit"}



Response:
{
  "code": 200,
  "data": {
    "orderId": "1911862608898764800",
    "clientId": "1560332366247"
  }
}
```


### Private Interface - Query the current list of delegate orders

```
Order list query by user request,
Speed ​​limit rule: 2 times / 1 seconds
HTTP GET /api/margin/v1/order/openOrders
```
Request parameters:

Name | Type | Required | Description
---|---|---|---
symbol | string | no | currency pair name, such as BTC/USDT
latestOrderId | string | No | Order id, used by page, the default value is empty, return the latest 20 data, displayed in reverse order by order id. Get the last order id-1, take the next page of data


```
Description:
Paging query, return 20 per page
```


Return field description:

Name | Type | Description
---|---|---
orderId | string | Order Id
baseAsset | string | Trading currency, such as BTC
quoteAsset | string | pricing currency, such as USDT
orderDirection | string | direction
quantity | string | order quantity
fillQuantity | string | Number of transactions
amount | string | order amount
filledAmount | string |
avgPrice | string | Average price
orderStatus | string | Order status, unfilled: open Completed: filled Cancel: canceled Partial deal: partialCancelled
orderTime | string | Order time
fee | string | handling fee


```
Request:
Url: http:// domain/api/margin/v1/order/openOrders?symbol=BTC%2FUSDT
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

### Private Interface - Query History Order List

```
Order list query by user request,
Speed ​​limit rule: 2 times / 1 seconds
HTTP GET /api/margin/v1/order/closedOrders
```
Request parameters:

Name | Type | Required | Description
---|---|---|---
Symbol | string | no | currency pair name, such as BTC/USDT
latestOrderId | string | No | Order id, used by page, the default value is empty, return the latest 20 data, displayed in reverse order by order id. Get the last order id-1, take the next page of data

```
Description:
Paging query, return 20 per page
```


Return field description:

Name | Type | Description
---|---|---
orderId | string | Order Id
base | string | Trading currency BTC
quote | string | pricing Currency USDT
orderDirection | string | direction
quantity | string | order quantity
amount | string | order amount
filledAmount | string |
takerFeeRate | string | taker rate
makerFeeRate | string | maker rate
avgPrice | string | Average price
orderStatus | string | Order status, unfilled: open Completed: filled Cancel: canceled Partial deal: partially cancelled
orderTime | string | Order time
totalFee | string | handling fee


```
Request:
Url: http://domain/api/margin/v1/order/closedOrders
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

### Private Interface - Query specified order information

```
Order list query by user request,
Speed ​​limit rule: 3 times / 1 seconds
HTTP GET /api/margin/v1/order/info
```
Request parameters:

Name | Type | Required | Description
---|---|---|---
orderId | string | yes | order ID


Return field description:

Name | Type | Description
---|---|---
orderId | string | Order Id
baseAsset | string | base currency, such as BTC
quoteAsset | string | Trading currency, such as USDT
orderDirection | string | Direction
quantity | string | order quantity
amount | string | order amount
filledAmount | string |
takerFeeRate | string | taker rate
makerFeeRate | string | maker rate
avgPrice | string | Average price
orderStatus | string | Order status, unfilled: open Completed: filled Cancel: canceled Partial deal: partially cancelled
orderTime | string | Order time
totalFee | string | handling fee

```
Request:
Url: http://domain/api/margin/v1/order/info
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
    "orderDirection": "buy",
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

### Private Interface - Query Order Transactions List

```
Order list query by user request,
Speed ​​limit rule: 3 times / 1 seconds
HTTP GET /api/margin/v1/order/trade/fills
```
Request parameters:

Name | Type | Required | Description
---|---|---|---
orderId | string | yes | order ID


Return field description:

Name | Type | Description
---|---|---
price | string | transaction price
quantity | string | Number of transactions
amount | string | transaction amount
fee | string | handling fee
direction | string | direction
tradeTime | string | Order trading time, international time
feeByConi | string | coni deduction

```
Request:
Url: http://domain/api/margin/v1/order/trade/fills?orderId=1938306197706977280
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




### Private Interface - Undo the specified order

```
Order list query by user request,
Speed ​​limit rule: 3 times / 1 seconds
HTTP POST /api/margin/v1/order/cancel
```
Request parameters:

Name | Type | Required | Description
---|---|---|---
orderId | string | yes | order ID

Return field description:

Name | Type | Description
---|---|---
data | string | Undo Order Id

```
Request:
Url: http://domain/api/margin/v1/order/cancel
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

### Private Interface - Unpaid Loan List

```
Order list query by user request,
Speed ​​limit rule: 3 times / 1 seconds
HTTP GET /api/margin/v1/account/unRepayOrderList
```
Request parameters:

Name | Type | Required | Description
---|---|---|---
symbol | string | no | currency pair name such as BTC/USDT
asset | string | no | asset name such as BTC
latestBorrowId | string | no | loan order ID

Return field description:

Name | Type | Description
---|---|---
borrowId | string | Borrower Id
asset | string | asset name
borrowQuantity | string |
interest | string |
repayQuantity | string | paid principal amount
repayInterest | string |

```
Request:
Url: http://domain/api/margin/v1/account/unRepayOrderList
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
    "code": 200,
    "data":[
        {
            "borrowId": "10898",
            "asset": "BTC",
            "borrowQuantity": "0.01",
            "interest": "0.00000009",
            "repayQuantity": "0",
            "repayInterest": "0"
        },
        {
            "borrowId": "10897",
            "asset": "BTC",
            "borrowQuantity": "0.01",
            "interest": "0.00000009",
            "repayQuantity": "0",
            "repayInterest": "0"
        },
        {
            "borrowId": "10885",
            "asset": "BTC",
            "borrowQuantity": "1",
            "interest": "0.00000834",
            "repayQuantity":"0.99999166",
            "repayInterest":"0.00000833"
        }
    ]
}
```

### Private Interface - Also pay off the list of bills

```
Order list query by user request,
Speed ​​limit rule: 3 times / 1 seconds
HTTP GET /api/margin/v1/account/finishRepayOrderList
```
Request parameters:

Name | Type | Required | Description
---|---|---|---
symbol | string | no | currency pair name such as BTC/USDT
asset | string | no | asset name such as BTC
latestBorrowId | string | no | loan order ID

Return field description:

Name | Type | Description
---|---|---
borrowId | string | Borrower Id
asset | string | asset name
borrowQuantity | string |
interest | string |
repayQuantity | string | paid principal amount
repayInterest | string |

```
Request:
Url: http://domain/api/margin/v1/account/finishRepayOrderList
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
    "code": 200,
    "data":[
        {
            "borrowId": "10884",
            "asset": "BTC",
            "borrowQuantity": "1",
            "interest": "0.00000833",
            "repayQuantity": "1",
            "repayInterest":"0.00000833"
        }
    ]
}
```

## Error Code Summary

Error code | message
---|:---
429 | Requests are too frequent
10001 | "ACCESS_KEY" cannot be empty
10002 | "ACCESS_SIGN" cannot be empty
10003 | "ACCESS_TIMESTAMP" cannot be empty
10005 | Invalid ACCESS_TIMESTAMP
10006 | Invalid ACCESS_KEY
10007 | Invalid Content_Type, please use "application / json" format
10008 | Request timestamp expired
10009 | System Error
10010 | API authentication failed
11000 | Required parameter cannot be empty
11001 | Parameter value error
11002 | The parameter value exceeds the maximum limit
11003 | No data returned by third-party interface
11004 | Order price accuracy does not match
11005 | The currency pair has not yet opened leverage
11007 | Coin pair does not match asset

