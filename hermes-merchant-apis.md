# Hermes-Merchant 开发者接口 
```当前版本：1.0```

| 版本历史      | 备注               |时间                                          |
| ---------- | -------------------           |           ---|
| 1.0 | 增加基本接口                |  2018/11/01  |

测试地址TEST_URL:`https://www-hermes.ocx.im`

## 1.API

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
curl -i -H 'Content-Type: application/json' -XPOST  http://localhost:8080/login?username=test&password=123
```

成功返回：

~~~

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Set-Cookie: sess=MTU0MTA1NTM0M3xEdi1CQkFFQ180SUFBUkFCRUFBQVNQLUNBQUVHYzNSeWFXNW5EQXdBQ214dloybHVYM1YxYVdRR2MzUnlhVzVuRENZQUpHUXlOek0xWkRaa0xUWXpPR0l0TkRNNE9TMDVNVEE1TFRrNFkyWTJORGhsTjJRNFlnPT188LSlVwRbX5RPKRhVgYhYjEc899geElJC9-ItMkCUBu4=; Path=/; Expires=Thu, 01 Nov 2018 07:55:43 GMT; Max-Age=3600
Date: Thu, 01 Nov 2018 06:55:43 GMT
Content-Length: 35

{"data":"login success","error":{}}

~~~

然后在登录成功的基础上查看其他信息：

```

curl -b "sess=MTU0MTA1NTM0M3xEdi1CQkFFQ180SUFBUkFCRUFBQVNQLUNBQUVHYzNSeWFXNW5EQXdBQ214dloybHVYM1YxYVdRR2MzUnlhVzVuRENZQUpHUXlOek0xWkRaa0xUWXpPR0l0TkRNNE9TMDVNVEE1TFRrNFkyWTJORGhsTjJRNFlnPT188LSlVwRbX5RPKRhVgYhYjEc899geElJC9-ItMkCUBu4=; Path=/; Expires=Thu, 01 Nov 2018 07:55:43 GMT; Max-Age=3600" http://localhost:8080/withdrawlist


```

成功返回：

```
{
    "data":[
        {
            "ID":1,
            "UUID":"thr",
            "Txid":"fgbf",
            "Confirmations":5,
            "MerchantCode":"ocx",
            "CurrencyCode":"eth",
            "Currency":{
                "Code":"",
                "Name":"",
                "Coin":false,
                "Decimals":0,
                "TokenAddress":"",
                "CreatedAt":"0001-01-01T00:00:00Z",
                "UpdatedAt":"0001-01-01T00:00:00Z",
                "ChainCode":"",
                "MinConfirmations":0,
                "MaxConfirmations":0
            },
            "Address":"",
            "Amount":"1",
            "Memo":"",
            "TransactionID":0,
            "CreatedAt":"0001-01-01T00:00:00Z",
            "UpdatedAt":"0001-01-01T00:00:00Z",
            "State":"",
            "ExternalUUID":""
        }
    ],
    "error":{

    }
}

```

如果在没有登录的情况下请求接口会返回：

```
HTTP/1.1 401 Unauthorized
Content-Type: application/json; charset=utf-8
Date: Thu, 01 Nov 2018 07:04:34 GMT
Content-Length: 51

{"error":{"code":"1005","message":"Login failed."}}

```


如果API调用失败，返回的请求会使用对应的HTTP status code, 同时返回包含了详细错误信息的JSON数据, 比如:

~~~json
{ "error": { "code": "1001", "message": "Internal error,bad database."}}
~~~

所有错误都遵循上面例子的格式，只是`code`和`message`不同。


### API列表

#### 公有接口

登录
`POST /login`

#### 私有接口

登出
`POST /logout`

获取商户账户余额信息
`GET /balance`

获取商户充值记录
`GET /depositlist`

获取商户提现记录
`GET /withdrawlist`

获取商户消息重发记录
`GET /notificationlist`

获取所有充值地址记录
`GET /addresslist`


#### `POST /login `  登录

* URL 

`{{TEST_URL}}/login`

* 入参
 
| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |
| username  | string | 姓名|
| password | string | 密码|

* 出参

| 字段        | 类型    | 解释     |
| ----------- | ------- | ------ |
| data        | string | 结果     |


#### `POST /logout`   登出

* URL 

`{{TEST_URL}}/logout`

* 入参

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |


* 出参

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |
| data  | string | 结果 |


####  `GET /balance ` 获取商户账户余额信息

 注意：需先登录

* URL 

`{{TEST_URL}}/balance`

* 入参

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |


* 出参

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |
| ID         | int     | id      |
| MerchantCode  | string | 商户code |
| CurrencyCode  | string | 币种code |
| Balance  | string | 可用金额 |
| Locked  | string | 锁定金额 |
| CreatedAt   | string | 创建时间 |
| UpdatedAt     | string | 更新时间 |

```
{
    "data":[
        {
            "ID":1,
            "MerchantCode":"ocx",
            "CurrencyCode":"eth",
            "Balance":"1",
            "Locked":"1",
            "CreatedAt":"0001-01-01T00:00:00Z",
            "UpdatedAt":"0001-01-01T00:00:00Z"
        }
    ],
    "error":{

    }
}

```

#### `GET /depositlist `  获取商户充值记录

 注意：需先登录

* URL 

`{{TEST_URL}}/depositlist `

* 入参

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |


* 出参

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |
| ID         | int     | id      |
| UUID  | string | UUID |
| Txid  | string | TxHash |
| MerchantCode  | string | 商户code |
| CurrencyCode  | string | 币种code |
| Address  | string |  交易地址|
| Amount  | string | 交易金额 |
| TransactionID  | int | 对应交易记录id |
| Memo  | string | 备注 |
| Amount  | string | 交易金额 |
| CreatedAt   | string | 创建时间 |
| UpdatedAt     | string | 更新时间 |
| State  | string | done/depositing=充值完成/充值中 |


```json
{
    "data":[
        {
            "ID":1,
            "UUID":"uuid",
            "Txid":"txid",
            "MerchantCode":"ocx",
            "CurrencyCode":"eth",
            "Currency":{
                "Code":"",
                "Name":"",
                "Coin":false,
                "Decimals":0,
                "TokenAddress":"",
                "CreatedAt":"0001-01-01T00:00:00Z",
                "UpdatedAt":"0001-01-01T00:00:00Z",
                "ChainCode":"",
                "MinConfirmations":0,
                "MaxConfirmations":0
            },
            "Address":"adddd",
            "Amount":"0.1",
            "TransactionID":1,
            "Transaction":{
                "ID":0,
                "Txid":"",
                "MerchantCode":"",
                "CurrencyCode":"",
                "From":"",
                "To":"",
                "Value":"0",
                "Memo":"",
                "CreatedAt":"0001-01-01T00:00:00Z",
                "UpdatedAt":"0001-01-01T00:00:00Z",
                "Confirmations":0,
                "Type":""
            },
            "Memo":"test",
            "CreatedAt":"0001-01-01T00:00:00Z",
            "UpdatedAt":"0001-01-01T00:00:00Z",
            "State":"done"
        }
    ],
    "error":{

    }
}
```

#### `GET /withdrawlist `  获取商户提现记录

 注意：需先登录

* URL 

`{{TEST_URL}}/withdrawlist `

* 入参

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |


* 出参

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |
| ID         | int     | id      |
| UUID  | string | UUID |
| Txid  | string | TxHash |
| MerchantCode  | string | 商户code |
| CurrencyCode  | string | 币种code |
| Address  | string |  交易地址|
| Amount  | string | 交易金额 |
| TransactionID  | int | 对应交易记录id |
| Amount  | string | 交易金额 |
| CreatedAt   | string | 创建时间 |
| UpdatedAt     | string | 更新时间 |
| State  | string | done/withdrawing=提现完成/提现中 |
| ExternalUUID     | string | 合作方提现订单唯一标示 |

```json
{
    "data":[
        {
            "ID":1,
            "UUID":"thr",
            "Txid":"fgbf",
            "Confirmations":5,
            "MerchantCode":"ocx",
            "CurrencyCode":"eth",
            "Currency":{
                "Code":"",
                "Name":"",
                "Coin":false,
                "Decimals":0,
                "TokenAddress":"",
                "CreatedAt":"0001-01-01T00:00:00Z",
                "UpdatedAt":"0001-01-01T00:00:00Z",
                "ChainCode":"",
                "MinConfirmations":0,
                "MaxConfirmations":0
            },
            "Address":"",
            "Amount":"1",
            "Memo":"",
            "TransactionID":0,
            "CreatedAt":"0001-01-01T00:00:00Z",
            "UpdatedAt":"0001-01-01T00:00:00Z",
            "State":"",
            "ExternalUUID":""
        }
    ],
    "error":{

    }
}
```

#### `GET /notificationlist `  获取商户消息重发记录

 注意：需先登录

* URL 

`{{TEST_URL}}/notificationlist `

* 入参

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |


* 出参

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |
| ID         | int     | id      |
| MerchantCode  | string | 商户code |
| CurrencyCode  | string | 币种code |
| SourceType  | string | Withdraw/Deposit=提现/充值 |
| SourceId  | int | 对应交易id |
| State  | string | done/withdrawing/depositing=交易完成/提现中/充值中 |
| WebhookType  | string |  回调通知方式 http/grpc|
| Payload  | string | 回调通知内容 |
| Sent  | bool | 是否已通知 true是false否 |
| Retries  | int | 已经重试的次数 |
| RetryPeriod   | int | 重试间隔（单位秒） |
| RetryLimit     | int | 重试总次数 |
| LastRetriedAt  | string | 上一次重试时间 |
| CreatedAt   | string | 创建时间 |
| UpdatedAt     | string | 更新时间 |

```json
{
    "data":[
        {
            "ID":1,
            "MerchantCode":"ocx",
            "CurrencyCode":"eth",
            "SourceType":"Withdraw",
            "SourceId":1,
            "State":"done",
            "WebhookType":"http",
            "Payload":"{}",
            "Sent":true,
            "Retries":0,
            "RetryPeriod":3600,
            "RetryLimit":5,
            "LastRetriedAt":"0001-01-01T00:00:00Z",
            "CreatedAt":"0001-01-01T00:00:00Z",
            "UpdatedAt":"0001-01-01T00:00:00Z"
        }
    ],
    "error":{

    }
}
```


#### `GET /notificationlist `  获取所有提现记录

 注意：需先登录

* URL 

`{{TEST_URL}}/notificationlist `

* 入参

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |


* 出参

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |
| ID         | int     | id      |
| MerchantCode  | string | 商户code |
| CurrencyCode  | string | 币种code |
| Address  | string |   充值地址|
| ChainCode  | string | 区块链产品标示 |
| SN  | string |  用户标示|
| CreatedAt   | string | 创建时间 |
| UpdatedAt     | string | 更新时间 |

```json
{
    "data":[
        {
            "ID":1,
            "MerchantCode":"ocx",
            "CurrencyCode":"eth",
            "Address":"0xtrbsrbtsrtbrstbdtneetyete",
            "ChainCode":"ethereum",
            "SN":"dgbdfs",
            "CreatedAt":"0001-01-01T00:00:00Z",
            "UpdatedAt":"0001-01-01T00:00:00Z"
        }
    ],
    "error":{

    }
}
```


## 错误码对照表

| 错误码       |       意义         | 中文解释                   |
| ----------- | ------------------ |  --------                  |
| 1001        | Internal error,bad database. | 内部错误，数据库异常。   |
| 1002        | Invalid param. |  内部错误，参数异常。    |
| 1003        | None param. |  没有数据。           |
| 1004        | Invalid password.  |  密码不可用。            |
| 1005        | Login failed. |  登录失败。             |
| 1006        | Unknown user.|  未知用户。             |
| 1007        | Logout failed.|  登出失败。             |
| 1008        | Bad session.|  会话异常。             |
| 1009        | Permission denied.|  禁止访问。             |
| 1010        | Exsited user.|  已存在用户。             |
| 1011        | Unknown merchant,please add a merchant first.|  未知商户，请先添加。   |
| 1012        | Register failed.|  注册失败。             |
