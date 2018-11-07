# Hermes 开发者接口 
```当前版本：1.4```

| 版本历史      | 备注               |时间                                          |
| ---------- | -------------------           |           ---|
| 1.0 | 增加Hermes签名与基本接口                |  2018/10/22  |
| 1.1 | 增加版本修改历史，增加申请提现接口提示信息  |  2018/10/23  |
| 1.2 | 增加错误码对照表，修改返回参数的错误码类型  |  2018/10/24  |
| 1.3 | 增加回调地址接口                        |  2018/10/25  |
| 1.4 | 金额将double改成string                    |  2018/10/29  |

测试地址TEST_URL:`https://hermes.ocx.im`

测试APP_KEY: ``


测试APP_SECRET:
``


## 1.签名


### 如何签名 (验证)

在给一个API请求签名之前, 你必须准备好你的`app_key/app_secret`。

在注册并认证通过后之后，只需访问API密钥页面就可以得到您的密钥。

所有的API都需要用这2个参数生成的`X-Hermes-Key`与`X-Hermes-Signature`用于身份验证。实际代码操作中这两个参数需要被添加到`http.Header`（具体可见下方完整代码示例）中。

| 参数       | 解释                                                         |
| ---------- | ---------------------------------------------  |
| X-Hermes-Key |你的`app_key`                                   |
| X-Hermes-Signature| 使用`http_query拼接成的字符串`与`app_secret`生成的签名 |

#### 签名步骤

1.通过组合HTTP方法, 请求地址和请求参数得到 `payload`。

~~~
# canonical_verb 是HTTP方法，例如 POST
# canonical_uri 是请求地址， 例如 /v1beta/addresses
# canonical_query 是请求参数通过&连接而成的字符串，参数包括http请求方式请求路径与请求参数, 参数必须按照字母序排列，例如 currency_code=eth&sn=1
# 最后再把这三个字符串通过'|'字符连接起来，看起来就像这样:

# POST|/v1beta/addresses|currency_code=eth&sn=1
#
def payload
  "#{canonical_verb}|#{canonical_uri}|#{canonical_query}"
end

~~~

2. 对上述字符串使用`HMAC-SHA256`加密算法进行SHA256计算:

~~~
 X-Hermes-Signature = HMAC-SHA256(payload, app_secret).to_hex
 处理得到：
 X-Hermes-Signature = bec05b722f04e2a4b9fa447c76c64ffb3f6899b004dce60d28c461929d31d2b8
~~~



#### 一个完整的`Golang`范例

~~~
  
package main

import (
	"crypto/hmac"
	"crypto/sha256"
	"encoding/hex"
	"errors"
	"fmt"
	"io/ioutil"
	"net/http"
	"net/url"
	"sort"
	"strings"
	"unsafe"
)

func hashMAC(message, key []byte) string {
	mac := hmac.New(sha256.New, key)
	mac.Write(message)
	expected := mac.Sum(nil)
	return hex.EncodeToString(expected)
}

func getPayload(reqMethod, urlStr string) (string, error) {
	u, err := url.Parse(urlStr)
	if err != nil {
		return "", errors.New("Resolveed the URL failed," + err.Error())
	}
	return strings.ToUpper(reqMethod) + "|" + u.Path + "|" + u.RawQuery, nil
}

func prepareQueryString(params url.Values) string {
	var keys []string
	for key := range params {
		keys = append(keys, strings.ToLower(key))
	}
	sort.Strings(keys)
	var pieces []string
	for _, key := range keys {
		pieces = append(pieces, key+"="+params.Get(key))
	}
	return strings.Join(pieces, "&")
}

type Client struct{}

func (this *Client) ClientGet(grpcHttpUrl string) {
	appKey := "88b4d0fe76dcfce6420d59c454eb560e692a6f67c035a8fb336ef36420f4333f"
	appSecret := "7ca2b1c29b22c7e45caaefca88c6235ab7cf0d8f45a5ad2031f8ade56d319dd8"
	reqMethod := "GET"
	payload, _ := getPayload(reqMethod, grpcHttpUrl)
	preToken := hashMAC([]byte(payload), []byte(appSecret))
	client := &http.Client{}
	req, err := http.NewRequest("GET", grpcHttpUrl, nil)
	if err != nil {
		panic("http newreq error :" + err.Error())
	}
	req.Header.Set("X-Hermes-Key", appKey)
	req.Header.Set("X-Hermes-Signature", preToken)
	resp, err := client.Do(req)
	defer resp.Body.Close()
	body, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		panic(err)
	}
	str := (*string)(unsafe.Pointer(&body))
	fmt.Println(*str)
}

func main() {
	var s Client
	BASE_URL := "https://hermes.ocx.im"
	s.ClientGet(BASE_URL + "/v1beta/deposits")
}


~~~
成功返回：

~~~

{
    "data":[
        {
            "uuid":"dbc4a2a6-bb5a-4129-a7a8-0f8bc4dda281",
            "address":"0xe37b4d97344f770309f1866bf93a7e3461441f6a",
            "amount":"1",
            "currency_code":"eth"
        },
        {
            "uuid":"b46c45b2-500a-4daa-8718-925c07ab0b1d",
            "address":"0xe37b4d97344f770309f1866bf93a7e3461441f6a",
            "amount":"0.5",
            "currency_code":"eth"
        },
        ......
    ]
}

~~~

## 2.API

**返回结果格式:`application/json`**


需要说明的是：所有接口的出参格式都是一样的，如下：

~~~
{
   "error":"...",
   "data":"..."
}
~~~
当接口请求成功的时候，`error`不返回，只返回`data`部分；同理，请求接口失败的`data `不返回，只返回`error`部分。
`error`部分参数格式都是相同的，只有`data`部分各不相同。下面是`error`的参数格式：

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |
| code       | string | 返回结果码 |
| message    | string | 返回结果信息  |

错误码对照表见文档底部。

下面在介绍各个接口出参的参数格式的时候，默认是`data`的参数格式。



### API 请求范例

 获取一个新的充值地址:

```
curl -i -H 'Content-Type: application/json' -H 'X-Hermes-Key: b645fe6588bfc8754ad4db35c967a7539ece14c781821565ff08ae082a1ff7f5' -H 'X-Hermes-Signature: 26ecd3c692bb2aa64d1e882493630f0fd2e4ecfa721470921033b3e772932d2a' -XPOST  https://hermes.ocx.im/v1beta/addresses -d '{ "currency_code": "eth", "sn": "1" }
```

成功返回：

~~~

{
    "data":{
        "currency_code":"eth",
        "address":"0xe82403d26E52415466461c74486f66b4f56674d5",
        "sn":"1"
    }
}

~~~

如果API调用失败，返回的请求会使用对应的HTTP status code, 同时返回包含了详细错误信息的JSON数据, 比如:

~~~json
{ "error": { "code": "4006", "message": "创建新地址失败。"}}
~~~

所有错误都遵循上面例子的格式，只是`code`和`message`不同。

`code`是Hermes自定义的一个错误代码, 表明此错误的类别, message是具体的出错信息。



### API列表

#### 公有接口

获取待充值地址
`POST /v1beta/addresses`

校验充值地址是否可用
`POST /v1beta/addresses/validate`

申请提现
`POST /v1beta/withdraws`

#### 私有接口

获取单个账户余额
`GET /v1beta/accounts/{currency_code}/balance`

获取单个账户信息
`GET /v1beta/accounts/{currency_code}`

获取所有充值记录
`GET /v1beta/deposits`

获取单个充值记录
`GET /v1beta/deposits/{uuid}`

获取所有提现记录
`GET /v1beta/withdraws`

获取单个提现记录
`GET /v1beta/withdraws/{uuid}`

提现回调

提现回调


#### `POST /v1beta/addresses `  获取待充值地址

* URL 

`{{TEST_URL}}/v1beta/addresses`

* 入参
 
| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |
| currency_code  | string | 币种类型 |
| sn             | string | 用户ID  |

* 出参

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |
| currency_code  | string | 币种类型 |
| sn             | string | 用户ID  |
| address        | string | 地址    |

* 请求示例

```json
# Request
POST {{TEST_URL}}/v1beta/addresses

curl -i -H 'Content-Type: application/json' -H 'X-Hermes-Key: b645fe6588bfc8754ad4db35c967a7539ece14c781821565ff08ae082a1ff7f5' -H 'X-Hermes-Signature: 26ecd3c692bb2aa64d1e882493630f0fd2e4ecfa721470921033b3e772932d2a' -XPOST  https://hermes.ocx.im/v1beta/addresses -d '{ "currency_code": "eth", "sn": "1" }

# Response

{
    "data":{
        "currency_code":"eth",
        "address":"0xe82403d26E52415466461c74486f66b4f56674d5",
        "sn":"1"
    }
}

```


#### `POST /v1beta/addresses/validate`  校验充值地址是否可用

* URL 

`{{TEST_URL}}/v1beta/addresses/validate`

* 入参

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |
| currency_code  | string | 币种类型 |
| address        | string | 地址    |

* 出参

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |
| is_valid  | bool | 是否可用 |

* 请求示例

```json
# Request
POST {{TEST_URL}}/v1beta/addresses/validate

# Response
 {
     "data" : {
         "is_valid" : true
     }
 }
```



####  `POST /v1beta/withdraws ` 申请提现

 注意：调用此接口之前，请先调用地址验证接口以保证地址（address）可用

* URL 

`{{TEST_URL}}/v1beta/withdraws`

* 入参

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |
| currency_code  | string | 币种类型 |
| address        | string | 地址    |
| amount         | string | 金额    |
| memo           | string | 备注    |
| external_uuid    | string | 客户端提现请求唯一标示 |


* 出参

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |
| uuid  | string | Hermes提现唯一标示 |
| txid  | string | 公链transaction_id |
| currency_code  | string | 币种code |
| state  | string | 状态 |
| from   | string | 提现资金流出方addr |
| to     | string | 提现资金流入方addr |
| value  | string | 提现金额 |
| memo   | string | 备注 |
| external_uuid   | string | 客户端提现请求唯一标示 |


* 请求示例

```json
# Request
POST {{TEST_URL}}/v1beta/withdraws

# Response
 {
    "data":{
        "uuid":"5de0cf14-758b-451c-8e9f-344d0a6d9f9a",
        "currency_code":"eth",
        "state":"submitted",
        "to":"rngnb",
        "value":"0.002",
        "memo":"gg",
        "external_uuid":"........"
    }
}
```


#### `GET /v1beta/accounts/{currency_code}/balance`  获取单个账户余额

* URL 

`{{TEST_URL}}/v1beta/accounts/{currency_code}/balance`

* 入参

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |
| currency_code  | string | 币种类型 |

* 出参

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |
| currency_code  | string | 币种类型 |
| balance  | string | 金额 |

* 请求示例

```json
# Request
GET {{TEST_URL}}/v1beta/accounts/eth/balance

# Response
 {
    "data":{
        "balance":"0.8749999999999999",
        "currency_code":"eth"
    }
}
```


#### `GET /v1beta/accounts/{currency_code}`  获取单个账户信息

* URL

 `{{TEST_URL}}/v1beta/accounts/{currency_code}`
 
* 入参

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |
| currency_code  | string | 币种类型 |

* 出参

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |
| currency_code  | string | 币种类型 |
| balance  | string | 金额 |
| locked   | string | 锁定金额 |


* 请求示例

```json
# Request
GET {{TEST_URL}}/v1beta/accounts/insur

# Response
{
    "data":{
        "currency_code":"insur",
        "balance":"100"
    }
}
```


#### `GET /v1beta/deposits`  获取所有充值记录
* URL 

`{{TEST_URL}}/v1beta/deposits`

* 入参

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |
| uuid  | string | 充值唯一标示 |
| page_number  | int | 页码 |
| page_size  | int | 每页显示数量 |
| start_time  | string | 开始时间 |
| end_time  | string | 结束时间 |
| state  | string | 状态。depositing/done=充值中/完成 |
| int  | cursor_limit | 单页限制 |


* 出参

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |
| currency_code  | string | 币种类型 |
| uuid  | string | 订单唯一标示 |
| amount   | string | 金额 |
| address  | string | 地址 |


* 请求示例

```json
# Request
GET {{TEST_URL}}/v1beta/deposits

# Response
{
    "data":[
        {
            "uuid":"f2501ce0-141c-4972-8ad5-025d9f1fb50e",
            "address":"0xfb1daa0934cfd2e6d1928c8613298d7a5747c793",
            "amount":"1",
            "currency_code":"eth"
        },
        {
            "uuid":"5d548c8e-7ae6-4b7c-b604-8f25b0fdcb34",
            "address":"0xa97ecd78da3bec89bd5bbd77132d3e0041bd110f",
            "amount":"100",
            "currency_code":"insur"
        }
    ]
}

```


#### `GET /v1beta/deposits/{uuid}`  获取单个充值记录
* URL 

`{{TEST_URL}}/v1beta/deposits/{uuid}`

* 入参

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |
| uuid  | string | 充值唯一标示 |


* 出参

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |
| currency_code  | string | 币种类型 |
| uuid  | string | 订单唯一标示 |
| amount   | string | 金额 |
| address  | string | 地址 |



* 请求示例

```json
# Request
GET {{TEST_URL}}/v1beta/deposits/5d548c8e-7ae6-4b7c-b604-8f25b0fdcb34

# Response
{
    "data":{
        "uuid":"5d548c8e-7ae6-4b7c-b604-8f25b0fdcb34",
        "address":"0xa97ecd78da3bec89bd5bbd77132d3e0041bd110f",
        "amount":"100",
        "currency_code":"insur"
    }
}
```



#### `GET /v1beta/withdraws`  获取所有提现记录
* URL 

`{{TEST_URL}}/v1beta/withdraws`

* 入参

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |
| uuid  | string | 充值唯一标示 |
| page_number  | int | 页码 |
| page_size  | int | 每页显示数量 |
| start_time  | string | 开始时间 |
| end_time  | string | 结束时间 |
| state  | string | 状态。withdrawing/done=提现中/完成 |
| int  | cursor_limit | 单页限制 |


* 出参

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |
| currency_code  | string | 币种类型 |
| uuid  | string | 订单唯一标示 |
| amount   | string | 金额 |
| address  | string | 地址 |



* 请求示例

```json
# Request
GET {{TEST_URL}}/v1beta/withdraws
# Response
{
    "data":[
        {
            "uuid":"90f72080-00ba-4c3e-ad6c-b2b45a1ec0fb",
            "address":"0xfb1daa0934cfd2e6d1928c8613298d7a5747c793",
            "amount":"0.007",
            "currency_code":"eth"
        },
        {
            "uuid":"25be73fa-a3a5-44fe-bbf8-42cf2180b760",
            "address":"0xfb1daa0934cfd2e6d1928c8613298d7a5747c793",
            "amount":"0.007",
            "currency_code":"eth"
        },
        {
            "uuid":"29cb14ee-1b35-4b8c-8825-e65044afdac5",
            "address":"rng nb",
            "amount":"0.002",
            "currency_code":"eth"
        },
        {
            "uuid":"bf02d0b7-8bf7-4d90-beb7-970327cefab9",
            "address":"rng nb",
            "amount":"0.002",
            "currency_code":"eth"
        },
        {
            "uuid":"5de0cf14-758b-451c-8e9f-344d0a6d9f9a",
            "address":"rngnb",
            "amount":"0.002",
            "currency_code":"eth"
        }
    ]
}
```



#### `GET /v1beta/withdraws/{uuid}`  获取单个提现记录
* URL

 `{{TEST_URL}}/v1beta/withdraws/{uuid}`
 
* 入参

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |
| uuid  | string | 充值唯一标示 |


* 出参

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |
| currency_code  | string | 币种类型 |
| uuid  | string | 订单唯一标示 |
| amount   | string | 金额 |
| address  | string | 地址 |



* 请求示例

```json
# Request
GET {{TEST_URL}}/vv1beta/withdraws/29cb14ee-1b35-4b8c-8825-e65044afdac5

# Response
{
    "data":{
        "uuid":"29cb14ee-1b35-4b8c-8825-e65044afdac5",
        "address":"rng nb",
        "amount":"0.002",
        "currency_code":"eth"
    }
}
```



#### `POST {{NOTIFY_URL}}`  充值回调
* NOTIFY_URL

 ` 合作方提供{{NOTIFY_URL}} `
 
* 入参

| 字段        | 类型    | 解释      |
| ----------- | ------- | -----   |
| type  | string | 类型。充值/提现=Deposit/Withdraw   |
| txid  | string | TxHash          |
| state  | string | 状态。depositing/done=充值中/充值完成  |
| currency_code | string | 币种code  |
| memo  | string | 备注              |
| address  | string | 地址           |
| amount  | string | 金额            |


* 出参 （由双方沟通协定，默认如下）

| 字段        | 类型    | 解释       |
| ----------- | ------- | -------- |
| message     | string | 请求结果   |



* 请求示例

```

# Request
{
    "type": "Deposit",
    "currency_code": "eth",
    "txid": "0xethereumtxid",
    "address": "0xfromaddress",
    "state": "depositing", 
    "amount": "1.0",
    "memo": "blockchain memo"
}

```



#### `POST {{NOTIFY_URL}}`  提现回调
* NOTIFY_URL

 ` 合作方提供{{NOTIFY_URL}} `
 
* 入参

| 字段        | 类型    | 解释        |
| ----------- | ------- | -----     |
| type  | string | 类型。充值/提现=Deposit/Withdraw   |
| txid  | string | TxHash           |
| state  | string | 状态。withdrawing/sent=提现中/提现完成   |
| currency_code | string | 币种code  |
| external_uuid  | string | 合作方交易唯一标示 |
| memo  | string | 备注 |




* 出参 （由双方沟通协定，默认如下）

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |
| message  | string | 请求结果 |



* 请求示例

```
# Request
{
    "type": "Withdraw",
    "currency_code": "eth",
    "txid": "0xethereumtxid",
    "state": "withdrawing", 
    "external_uuid": "merchant withdraw uuid"
}

```





## 错误码对照表

| 错误码       |       意义         | 中文解释                   |
| ----------- | ------------------ |  --------                  |
| 1001        | Internal error,SQS had broken down. | 内部错误，消息推送服务异常。   |
| 1002        | Internal error,Mysql had broken down. |  内部错误，数据库返回失败。    |
| 1003        | Missing params. |  必要参数不能为空。           |
| 1004        | Blockchain return error.  |  区块链返回错误。            |
| 1005        | Failed to json marshal. |  格式化数据失败。             |
| 1006        | The amount is too small. |  金额太小。             |
| 1500        | Internal error.          | 内部错误。 |
| 2003        | Auth signature incorrect. | 验签出错。|
| 3000        | Unknown currency.  |  未知币种。             |
| 3001        | Unknown account. |  未知账户。             |
| 3002        | Unknown merchant. |  未知商户。             |
| 3003        | Failed authentication. |  认证失败。             |
| 4001        | Exsited withdraw apply. | 提现请求已存在。        |
| 4002        | Failed to create deposition apply. | 创建提现请求失败。      |
| 4003        | Lack of money.|  金额不足。             |
| 4004        | Not found deposit's record. | 没有找到充值记录。      |
| 4005        | Not found withdraw's record. |  没有找到提现记录。      |
| 4006        | Failed to create a new address.  |  创建新地址失败。        |