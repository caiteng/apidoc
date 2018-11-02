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

登录:

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

{"data":"Login success"}

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
            "txid":"fgbf",
            "confirmations":5,
            "currency_name":"eth",
            "address":"",
            "amount":"1",
            "memo":"",
            "created_at":"0001-01-01T00:00:00Z",
            "state":"",
            "external_uuid":""
        }
    ]
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
| data        | string | 结果    |


```
{"data":"Login success"}

```


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
| token  | string | token |


```
{"data":"Login success","token":"d2735d6d-638b-4389-9109-98cf648e7d8b"}

```

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
| currency_name  | string | 币种名称 |
| balance  | string | 可用金额 |
| locked  | string | 锁定金额 |
| created_at   | string | 创建时间 |

```
{
    "data":[
        {
            "currency_name":"以太币",
            "balance":"1",
            "locked":"1",
            "created_at":"0001-01-01T00:00:00Z"
        }
    ]
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
| txid  | string | TxHash |
| currency_name  | string | 币种名称 |
| address  | string |  交易地址|
| amount  | string | 交易金额 |
| memo  | string | 备注 |
| created_at   | string | 创建时间 |
| state  | string | done/depositing=充值完成/充值中 |


```json
{
    "data":[
        {
            "txid":"txid",
            "currency_name":"eth",
            "address":"adddd",
            "amount":"0.1",
            "memo":"test",
            "created_at":"0001-01-01T00:00:00Z",
            "state":"done"
        }
    ]
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
| txid  | string | TxHash |
| currency_name  | string | 币种名称 |
| address  | string |  交易地址|
| amount  | string | 交易金额 |
| created_at   | string | 创建时间 |
| state  | string | done/withdrawing=提现完成/提现中 |
| external_uuid     | string | 合作方提现订单唯一标示 |

```json
{
    "data":[
        {
            "txid":"fgbf",
            "confirmations":5,
            "currency_name":"eth",
            "address":"",
            "amount":"1",
            "memo":"",
            "created_at":"0001-01-01T00:00:00Z",
            "state":"",
            "external_uuid":""
        }
    ]
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
| currency_name  | string | 币种名称 |
| source_type  | string | Withdraw/Deposit=提现/充值 |
| state  | string | done/withdrawing/depositing=交易完成/提现中/充值中 |
| webhook_type  | string |  回调通知方式 http/grpc|
| payload  | string | 回调通知内容 |
| sent  | bool | 是否已通知 true是false否 |
| retries  | int | 已经重试的次数 |
| retry_period   | int | 重试间隔（单位秒） |
| retry_limit     | int | 重试总次数 |
| last_retried_at  | string | 上一次重试时间 |
| created_at   | string | 创建时间 |
| updated_at     | string | 更新时间 |

```json
{
    "data":[
        {
            "currency_name":"eth",
            "source_type":"Withdraw",
            "state":"done",
            "webhook_type":"http",
            "payload":"{}",
            "sent":true,
            "retries":0,
            "retry_period":3600,
            "retry_limit":5,
            "last_retried_at":"0001-01-01T00:00:00Z",
            "created_at":"0001-01-01T00:00:00Z",
            "updated_at":"0001-01-01T00:00:00Z"
        }
    ]
}
```


#### `GET /addresslist `  获取所有充值地址记录

 注意：需先登录

* URL 

`{{TEST_URL}}/addresslist `

* 入参

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |


* 出参

| 字段        | 类型    | 解释     |
| ----------- | ------- | -------- |
| currency_name  | string | 币种名称 |
| address  | string |   充值地址|
| chain_name  | string | 区块链产品名称 |
| sn  | string |  用户标示|
| created_at   | string | 创建时间 |

```json
{
    "data":[
        {
            "currency_name":"eth",
            "address":"0xtrbsrbtsrtbrstbdtneetyete",
            "chain_name":"ethereum",
            "sn":"dgbdfs",
            "created_at":"0001-01-01T00:00:00Z"
        }
    ]
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
