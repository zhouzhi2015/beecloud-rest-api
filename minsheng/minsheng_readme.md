## BeeCloud REST API(Minsheng)

## 简介


BeeCloud REST API(Minsheng)提供网关快捷支付，退款和查询的API接口.

## 1. 支付（网关）

此接口为网关支付流程的第一步，主要功能在于生成订单，获取必要的参数信息，来进行下一步的支付流程.

#### URL:   */1/rest/minsheng/bill*
#### Method: *POST*
#### 请求参数格式: *JSON: Map*

#### 请求参数详情:
- 以下为公共参数：

参数名 | 类型 | 含义 | 描述 | 例子 | 是否必填
----  | ---- | ---- | ---- | ---- | ----
app_id | String | BeeCloud平台的AppID | App在BeeCloud平台的唯一标识 | 0950c062-5e41-44e3-8f52-f89d8cf2b6eb | 是
timestamp | Long | 签名生成时间 | 时间戳，毫秒数 | 1435890533866 | 是
app_sign | String | 加密签名 | 算法: md5(app\_id+timestamp+app\_secret)，32位16进制格式,不区分大小写 | b927899dda6f9a04afc57f21ddf69d69 | 是
subject| String | 商品种类| 该参数，是从民生电商处获得 | 1172001| 是
channel| String | 渠道类型 | 根据不同场景选择不同的支付方式 | MS_WEB| 是
total_fee | Integer | 订单总金额 | 必须是正整数，单位为分，最低100分 | 100 | 是
bill_no | String | 商户订单号 | 8到30位数字和/或字母组合，请**<mark>务必</mark>**确保在商户系统中唯一，同一订单号不可重复提交，否则会造成订单重复 | 201506101035040000001 | 是
title| String | 订单标题 | UTF8编码格式，32个字节内，最长支持16个汉字 | 白开水 | 是
optional | Map | 附加数据 | 用户自定义的参数，将会在webhook通知中原样返回，该字段主要用于商户携带订单的自定义数据 | {"key1":"value1","key2":"value2",...} | 否
return_url | String | 同步返回页面| 支付渠道处理完请求后,当前页面自动跳转到商户网站里指定页面的http路径,不要以包含localhost，否则渠道会认为非法 | beecloud.cn/returnUrl.jsp | 当为MS_WEB时，必填
> 注：channel的参数值含义：  
MS\_WEB: 民生网关  
 

#### 返回类型: *JSON: Map*
#### 返回参数:
- **公共返回参数**

参数名 | 类型 | 含义 
---- | ---- | ----
result_code | Integer | 返回码，0为正常
result_msg  | String | 返回信息， OK为正常
err_detail  | String | 具体错误信息
id  | String | 成功发起支付后返回支付表记录唯一标识
- **公共返回参数取值列表及其含义**

result_code | result_msg             | 含义
----        | ----      		       | ----
0           | OK                     | 调用成功
1           | APP\_INVALID           | 根据app\_id找不到对应的APP或者app\_sign不正确
2           | PAY\_FACTOR_NOT\_SET   | 支付要素在后台没有设置
3           | CHANNEL\_INVALID       | channel参数不合法
4           | MISS\_PARAM            | 缺少必填参数
5           | PARAM\_INVALID         | 参数不合法
6           | CERT\_FILE\_ERROR      | 证书错误
7           | CHANNEL\_ERROR         | 渠道内部错误
14          | RUN\_TIME_ERROR        | 实时未知错误，请与技术联系帮助查看

> **<mark>当result_code不为0时，如需详细信息，请查看err\_detail字段**

**<mark>以下字段在result_code为0时有返回</mark>**
- MS\_WEB

参数名 | 类型 | 含义 
---- | ---- | ----
html | String | 民生电商网关跳转form，是一段HTML代码，自动提交


## 2. 支付（快捷之鉴权）

此接口为快捷支付流程的第一步，主要功能在于获取令牌信息，来进行下一步的支付流程.

#### URL:   */1/rest/minsheng/bill*
#### Method: *POST*
#### 请求参数格式: *JSON: Map*

#### 请求参数详情:
- 以下为公共参数：

参数名 | 类型 | 含义 | 描述 | 例子 | 是否必填
----  | ---- | ---- | ---- | ---- | ----
app_id | String | BeeCloud平台的AppID | App在BeeCloud平台的唯一标识 | 0950c062-5e41-44e3-8f52-f89d8cf2b6eb | 是
timestamp | Long | 签名生成时间 | 时间戳，毫秒数 | 1435890533866 | 是
app_sign | String | 加密签名 | 算法: md5(app\_id+timestamp+app\_secret)，32位16进制格式,不区分大小写 | b927899dda6f9a04afc57f21ddf69d69 | 是
subject| String | 商品种类| 该参数，是从民生电商处获得 | 1172001| 是
title| String | 订单标题 | UTF8编码格式，32个字节内，最长支持16个汉字 | 白开水 | 是
channel| String | 渠道类型 | 根据不同场景选择不同的支付方式 | MS_WAP| 是
total_fee | Integer | 订单总金额 | 必须是正整数，单位为分，最低100分 | 100 | 是
bill_no | String | 商户订单号 | 8到30位数字和/或字母组合，请自行确保在商户系统中唯一，同一订单号不可重复提交，否则会造成订单重复 | 201506101035040000001 | 是
cust_id | String | 客户号 | 28位以内的数字或字母的组合 | 1111111111111 | 是
cust_name | String | 客户姓名 | UTF8编码格式，30个字节内，最长支持16个汉字 | 黄晓明 | 是
cust\_id\_no | String | 证件号 | 身份证号，军官证等 | 88888888888888888， | 是
cust\_id\_type| String | 客户证件类型，详细请见**<mark>下方证件类型码</mark>** | 0-18 | 0代表身份证 | 是
card_no| String | 银行卡号 | 借记卡和信用卡的卡号 | 6666666666666666666 | 是
bank_no| String | 银行编号 | 需要快捷支付的银行号，详细请见**<mark>下方的银行列表</mark>** | 01040000，中国银行 | 是
expired_date| String | 信用卡有效日期| 信用卡的有效日期 | 0216 | **<mark>使用借记卡时可以不填，信用卡时必填</mark>**
cvv2| String | 信用卡验证码| 信用卡背后的三位验证码 | 110 | **<mark>使用借记卡时可以不填，信用卡时必填</mark>**
phone_no| String | 手机号| 手机11位号码 | 13666666666 | 是
flag| String |鉴权和支付的标志位| 用于区分快捷的鉴权操作和支付操作 | 鉴权时:sign | 是

> 注：channel的参数值含义：   
MS\_WAP: 民生快捷  

#### 返回类型: *JSON: Map*
#### 返回参数:
- **公共返回参数**

参数名 | 类型 | 含义 
---- | ---- | ----
result_code | Integer | 返回码，0为正常
result_msg  | String | 返回信息， OK为正常
err_detail  | String | 具体错误信息
> 公共返回参数取值及含义参见支付(网关)公共返回参数部分

> **<mark>当result_code不为0时，如需详细信息，请查看err\_detail字段**

**<mark>以下字段在result_code为0时有返回</mark>**

- MS\_WAP（鉴权）

参数名 | 类型 | 含义 
---- | ---- | ----
phoneToken | String | 令牌信息

- **<mark>银行列表</mark>**

银行号| 银行名称
---- | ----
03080000  | 招商银行 
01020000  | 中国工商银行
01030000  | 中国农业银行
01050000  | 中国建设银行 
01040000	| 中国银行 
03100000	| 浦发银行 

- **<mark>证件类型码</mark>**

证件类型| 描述
---- | ----
0	| 身份证类型

## 3. 支付（快捷之支付）

此接口为快捷支付流程的第二步，主要功能是在获取令牌信息之后，做支付操作.

#### URL:   */1/rest/minsheng/bill*
#### Method: *POST*
#### 请求参数格式: *JSON: Map*

#### 请求参数详情:
- 以下为公共参数：

参数名 | 类型 | 含义 | 描述 | 例子 | 是否必填
----  | ---- | ---- | ---- | ---- | ----
app_id | String | BeeCloud平台的AppID | App在BeeCloud平台的唯一标识 | 0950c062-5e41-44e3-8f52-f89d8cf2b6eb | 是
timestamp | Long | 签名生成时间 | 时间戳，毫秒数 | 1435890533866 | 是
app_sign | String | 加密签名 | 算法: md5(app\_id+timestamp+app\_secret)，32位16进制格式,不区分大小写 | b927899dda6f9a04afc57f21ddf69d69 | 是
subject| String | 商品种类| 该参数，是从民生电商处获得 | 1172001| 是
channel| String | 渠道类型 | 根据不同场景选择不同的支付方式 | MS_WAP| 是
total_fee | Integer | 订单总金额 | 必须是正整数，单位为分，最低100分 | 100 | 是
bill_no | String | 商户订单号 | 8到30位数字和/或字母组合，请**<mark>务必</mark>**确保在商户系统中唯一，同一订单号不可重复提交，否则会造成订单重复 | 201506101035040000001 | 是
card_no| String | 银行卡号 | 借记卡和信用卡的卡号 | 6666666666666666666 | 是
bank_no| String | 银行编号 | 需要快捷支付的银行号，详细请见**<mark>上方的银行列表</mark>** | 01040000，中国银行 | 是
expired_date | String | 信用卡有效日期| 信用卡的有效日期 | 0216 | **<mark>使用借记卡时可以不填，信用卡时必填</mark>**
cvv2| String | 信用卡验证码| 信用卡背后的三位验证码 | 110 |  **<mark>使用借记卡时可以不填，信用卡时必填</mark>**
cust_name | String | 客户姓名 |UTF8编码格式，30个字节内，最长支持16个汉字 | 黄晓明 | 是
cust\_id\_no | String | 证件号 | 身份证号，军官证等 | 88888888888888888 | 是
cust\_id\_type| String | 客户证件类型 | 0-18，详细请见**<mark>上方证件类型码</mark>** | 0代表身份证 | 是
cust_id | String | 客户号 | 28位以内的数字或字母的组合 | 1111111111111 | 是
phone_no| String | 手机号| 手机11位号码 | 13666666666 | 是
phone\_ver\_code| String | 手机验证码| 一般为6位，是民生电商发给用户的**<mark>快捷支付阶段，该值必填</mark>** | 888888 | 是
phone_token| String | 手机校验码令牌| **<mark>鉴权结果返回</mark>** | 666666 | 是
title| String | 订单标题 | UTF8编码格式，32个字节内，最长支持16个汉字 | 白开水 | 是
optional | Map | 附加数据 | 用户自定义的参数，将会在webhook通知中原样返回，该字段主要用于商户携带订单的自定义数据 | {"key1":"value1","key2":"value2",...} | 否
flag| String |鉴权和支付的标志位| 用于区分快捷的鉴权操作和支付操作 | 支付时:pay | 是

> 注：channel的参数值含义：  
MS\_WAP: 民生快捷  

#### 返回类型: *JSON: Map*
#### 返回参数:
- **公共返回参数**

参数名 | 类型 | 含义 
---- | ---- | ----
result_code | Integer | 返回码，0为正常
result_msg  | String | 返回信息， OK为正常
err_detail  | String | 具体错误信息
id  | String | 成功发起支付后返回支付表记录唯一标识
> 公共返回参数取值及含义参见支付(网关)公共返回参数部分

> **<mark>当result_code不为0时，如需详细信息，请查看err\_detail字段**

**<mark>以下字段在result_code为0时有返回</mark>**

- MS\_WAP（支付）

参数名 | 类型 | 含义 
---- | ---- | ----
refNo | String | 系统参考号,快捷后台唯一标示的记录,

## 4. 订单查询（快捷之单笔订单查询）

此接口为快捷的单笔查询.

#### URL:   */1/rest/minsheng/kjquerybyid*
#### Method: *GET*
#### 请求参数格式: *JSON: Map*

#### 请求参数详情:
- 以下为公共参数：

参数名 | 类型 | 含义 | 描述 | 例子 | 是否必填
----  | ---- | ---- | ---- | ---- | ----
app_id | String | BeeCloud平台的AppID | App在BeeCloud平台的唯一标识 | 0950c062-5e41-44e3-8f52-f89d8cf2b6eb | 是
timestamp | Long | 签名生成时间 | 时间戳，毫秒数 | 1435890533866 | 是
app_sign | String | 加密签名 | 算法: md5(app\_id+timestamp+app\_secret)，32位16进制格式,不区分大小写 | b927899dda6f9a04afc57f21ddf69d69 | 是
channel\_trade\_no | String | 渠道交易号，**<mark>就是快捷支付返回的refNo</mark>** | 由快捷支付时返回的| 201510101000001 | 是

#### 返回类型: *JSON: Map*
#### 返回参数:
- **公共返回参数**

参数名 | 类型 | 含义 
---- | ---- | ----
result_code | Integer | 返回码，0为正常
result_msg  | String | 返回信息， OK为正常
err_detail  | String | 具体错误信息
> 公共返回参数取值及含义参见支付(网关)公共返回参数部分

> **<mark>当result_code不为0时，如需详细信息，请查看err\_detail字段**

**<mark>以下字段在result_code为0时有返回</mark>**

参数名 | 类型 | 含义 
---- | ---- | ----
txnType | String | 交易类型，QP0001，指消费，
txnStat | String |交易状态，0-处理中；1-成功；2-失败
amount| String |金额
merTransTime | String | 商户交易时间，格式为yyyyMMddHHmmss，例如：20140825010101
merOrderId | String | 商户订单号

## 5. 订单查询（网关之单笔订单查询）

此接口为网关的单笔订单查询，主要功能在于获得单笔订单信息.

#### URL:   */1/rest/minsheng/webquerybyid*
#### Method: *GET*
#### 请求参数格式: *JSON: Map*

#### 请求参数详情:
- 以下为公共参数：

参数名 | 类型 | 含义 | 描述 | 例子 | 是否必填
----  | ---- | ---- | ---- | ---- | ----
app_id | String | BeeCloud平台的AppID | App在BeeCloud平台的唯一标识 | 0950c062-5e41-44e3-8f52-f89d8cf2b6eb | 是
timestamp | Long | 签名生成时间 | 时间戳，毫秒数 | 1435890533866 | 是
app_sign | String | 加密签名 | 算法: md5(app\_id+timestamp+app\_secret)，32位16进制格式,不区分大小写 | b927899dda6f9a04afc57f21ddf69d69 | 是
bill_no | String | 商户订单号 | 8到30位数字和/或字母组合，请自行确保在商户系统中唯一，同一订单号不可重复提交，否则会造成订单重复 | 201506101035040000001 | 是

#### 返回类型: *JSON: Map*
#### 返回参数:
- **公共返回参数**

参数名 | 类型 | 含义 
---- | ---- | ----
result_code | Integer | 返回码，0为正常
result_msg  | String | 返回信息， OK为正常
err_detail  | String | 具体错误信息，当为渠道错误时，除了验签出错，网络异常和MD5异常之外，错误信息会返回"checkFailure,error id: 1"这个的错误描述，其中error id: 1 可参考下面“查询出错信息对照表”中的描述获取相应的中文描述信息
> 公共返回参数取值及含义参见支付(网关)公共返回参数部分

> **<mark>当result_code不为0时，如需详细信息，请查看err\_detail字段**

**<mark>以下字段在result_code为0时有返回</mark>**

参数名 | 类型 | 含义 
---- | ---- | ----
payOrderId | String | 平台交易号
merOrderId | String | 商户订单号
merSendTime  | String | 平台接受订单时间
amountSum | String |金额
payBank  | String |  支付银行
state | String | 状态
type | String |交易类型

- **<mark>查询出错信息对照表</mark>**

Error id	|信息
---- | ----

## 6. 订单查询（网关之批量订单查询）

此接口为网关的单笔订单查询，主要功能在于获得单笔订单信息.

#### URL:   */1/rest/minsheng/querys*
#### Method: *GET*
#### 请求参数格式: *JSON: Map*

#### 请求参数详情:
- 以下为公共参数：

参数名 | 类型 | 含义 | 描述 | 例子 | 是否必填
----  | ---- | ---- | ---- | ---- | ----
app_id | String | BeeCloud平台的AppID | App在BeeCloud平台的唯一标识 | 0950c062-5e41-44e3-8f52-f89d8cf2b6eb | 是
timestamp | Long | 签名生成时间 | 时间戳，毫秒数 | 1435890533866 | 是
app_sign | String | 加密签名 | 算法: md5(app\_id+timestamp+app\_secret)，32位16进制格式,不区分大小写 | b927899dda6f9a04afc57f21ddf69d69 | 是
pay_bank| String | 支付银行 | 要查询的银行支付渠道；无值则查所有银行 | 具体支持银行请参考“宝易互通所支持银行缩写表”| 否
start_time | String | 开始时间 | 格式：yyyy-MM-dd，共10位 |2015-10-10|是
end_time | String | 结束时间 | 格式：yyyy-MM-dd，共10位 |2015-10-10|是

#### 返回类型: *JSON: Map*
#### 返回参数:
- **公共返回参数**

参数名 | 类型 | 含义 
---- | ---- | ----
result_code | Integer | 返回码，0为正常
result_msg  | String | 返回信息， OK为正常
err_detail  | String | 具体错误信息，当为渠道错误时，除了验签出错，网络异常和MD5异常之外，错误信息会返回"checkFailure,error id: 1"这个的错误描述，其中error id: 1 可参考上面“查询出错信息对照表”中的描述获取相应的中文描述信息
count | Integer | 查询订单结果数量
bills | List<Map> | 订单列表
> 公共返回参数取值及含义参见支付公共返回参数部分  

> **<mark>当result_code不为0时，如需详细信息，请查看err\_detail字段**

**<mark>以下字段在result_code为0时有返回</mark>**

- bills说明，每个Map的key\-value

参数名         | 类型          | 含义 
----          | ----         | ----
payOrderId | String | 平台交易号
merOrderId | String | 商户订单号
merSendTime  | String | 平台接受订单时间
amountSum | String |金额
payBank  | String |  支付银行，具体支持银行请参考“宝易互通所支持银行缩写表”
state | String | 状态，0:未付款；1:成功相符；2:成功不符；3:失败；
type | String |交易类型，0:非会员交易；5:会员即时到账交易；6:会员担保交易支付

- 宝易互通所支持银行缩写表
 
银行缩写         | 银行名称          
----          | ----  
CCB	| 中国建设银行B2C支付渠道
GDB	|广东发展银行B2C支付渠道

## 7. 退款（网关）

此接口为网关的退款.

#### URL:   */1/rest/minsheng/refund*
#### Method: *POST*
#### 请求参数格式: *JSON: Map*

#### 请求参数详情:
- 以下为公共参数：

参数名 | 类型 | 含义 | 描述 | 例子 | 是否必填
----  | ---- | ---- | ---- | ---- | ----
app_id | String | BeeCloud平台的AppID | App在BeeCloud平台的唯一标识 | 0950c062-5e41-44e3-8f52-f89d8cf2b6eb | 是
timestamp | Long | 签名生成时间 | 时间戳，毫秒数 | 1435890533866 | 是
app_sign | String | 加密签名 | 算法: md5(app\_id+timestamp+app\_secret)，32位16进制格式,不区分大小写 | b927899dda6f9a04afc57f21ddf69d69 | 是
bill_no | String | 商户订单号 | 8到30位数字和/或字母组合，请自行确保在商户系统中唯一，同一订单号不可重复提交，否则会造成订单重复 | 201506101035040000001 | 是
refund_no | String | 商户退款单号 | 格式为:退款日期(8位) + 流水号(3~24 位)。请**<mark>务必</mark>**确保在商户系统中唯一，且退款日期必须是发起退款的当天日期,同一退款单号不可重复提交，否则会造成退款单重复。流水号可以接受数字或英文字符，建议使用数字，但不可接受“000” | 
refund_fee |Integer | 退款金额 | 必须是正整数，单位为分，最低100分 | 100 | 是

#### 返回类型: *JSON: Map*
#### 返回参数:
- **公共返回参数**

参数名 | 类型 | 含义 
---- | ---- | ----
result_code | Integer | 返回码，0为正常
result_msg  | String | 返回信息， OK为正常
err_detail  | String | 具体错误信息，
id  | String | 成功发起支付后返回支付表记录唯一标识，当为渠道错误时，除了验签出错，网络异常和MD5异常之外，错误信息会返回码，具体请参数下面“退款返回信息对照表”
 **<mark>当result_code不为0时，如需详细信息，请查看err\_detail字段**

**<mark>以下字段在result_code为0时有返回</mark>**

参数名 | 类型 | 含义 
---- | ---- | ----
refundId | String | 平台退款流水号,在民生电商系统中唯一，用于单笔退款查询

- 退款返回信息对照表
返回码|	含义
---- | ---- 
0007	| 未找到要退款的记录，可能已结算.

## 8. 退款查询（网关之批量退款查询）

此接口为网关的退款查询，主要功能在于获得所有退款信息.

#### URL:   */1/rest/minsheng/refunds*
#### Method: *GET*
#### 请求参数格式: *JSON: Map*

#### 请求参数详情:
- 以下为公共参数：

参数名 | 类型 | 含义 | 描述 | 例子 | 是否必填
----  | ---- | ---- | ---- | ---- | ----
app_id | String | BeeCloud平台的AppID | App在BeeCloud平台的唯一标识 | 0950c062-5e41-44e3-8f52-f89d8cf2b6eb | 是
timestamp | Long | 签名生成时间 | 时间戳，毫秒数 | 1435890533866 | 是
app_sign | String | 加密签名 | 算法: md5(app\_id+timestamp+app\_secret)，32位16进制格式,不区分大小写 | b927899dda6f9a04afc57f21ddf69d69 | 是
begin_date | String | 开始时间 | 格式：yyyy-MM-dd，共10位 |2015-10-10|是
end_date | String | 结束时间 | 格式：yyyy-MM-dd，共10位 |2015-10-10|是

#### 返回类型: *JSON: Map*
#### 返回参数:
- **公共返回参数**

参数名 | 类型 | 含义 
---- | ---- | ----
result_code | Integer | 返回码，0为正常
result_msg  | String | 返回信息， OK为正常
err_detail  | String | 具体错误信息,当为渠道错误时，除了验签出错，网络异常和MD5异常之外，错误信息会返回码，具体请参数下面“退款信息返回信息对照表”
count | Integer | 查询订单结果数量
refunds | List<Map> | 订单列表
> 公共返回参数取值及含义参见支付公共返回参数部分  

**当<mark>result_code不为0时，如需详细信息，请查看err\_detail字段**

**<mark>以下字段在result_code为0时有返回</mark>**

- refunds说明，每个Map的key\-value

参数名         | 类型          | 含义 
----          | ----         | ----
refundId | String | 退款流水号
merchantId | String | 商户号
payorderId    | String | 原平台交易号
merOrderId   | String |商户订单号
amount  | String |  原订单金额
refundAmount | String | 退款申请金额
state | String |状态 0:新申请; 1:成功; 2:拒绝; 4:平台拒绝; 5:撤销申请
applyDate | String | 申请时间
startflag | String | 发起标记 0:商户; 1:平台


- 退款信息返回信息对照表

返回码|	含义
----  | ---- 
0000	| 查询成功

## 9. 退款更新（网关之单笔退款更新）

此接口为网关的单笔退款更新，主要功能在于更新单笔退款状态.

#### URL:   */1/rest/minsheng/refund/status*
#### Method: *GET*
#### 请求参数格式: *JSON: Map*

#### 请求参数详情:
- 以下为公共参数：

参数名 | 类型 | 含义 | 描述 | 例子 | 是否必填
----  | ---- | ---- | ---- | ---- | ----
app_id | String | BeeCloud平台的AppID | App在BeeCloud平台的唯一标识 | 0950c062-5e41-44e3-8f52-f89d8cf2b6eb | 是
timestamp | Long | 签名生成时间 | 时间戳，毫秒数 | 1435890533866 | 是
app_sign | String | 加密签名 | 算法: md5(app\_id+timestamp+app\_secret)，32位16进制格式,不区分大小写 | b927899dda6f9a04afc57f21ddf69d69 | 是
channel\_refund\_no| String | 平台退款流水号 | 平台退款流水号,在民生电商系统中唯一，用于单笔退款查询,从退款处获得 | 188888| 是

#### 返回类型: *JSON: Map*
#### 返回参数:
- **公共返回参数**

参数名 | 类型 | 含义 
---- | ---- | ----
result_code | Integer | 返回码，0为正常
result_msg  | String | 返回信息， OK为正常
err_detail  | String | 具体错误信息,当为渠道错误时，除了验签出错，网络异常和MD5异常之外，错误信息会返回码，具体请参数上面“退款信息返回信息对照表”

**<mark>当result_code不为0时，如需详细信息，请查看err\_detail字段**

**<mark>以下字段在result_code为0时有返回</mark>**

参数名 | 类型 | 含义 
---- | ---- | ----
refund_status | String | **<mark>退款状态</mark>**   PROCESSING:退款中；SUCCESS:退款成功；FAIL：退款失败


## 9. WebHooK（快捷支付）
{"retryCounter":0,
"transaction_id":"be495cb4e4cedc9790c974",
"retry_counter":0,
"channelType":"MS",
"sub_channel_type":"MS_WAP",
"optional":{},
"transaction_type":"PAY",
"notify_url":"https://apihz.beecloud.cn/1/pay/webhook/receiver/c37d661d-7e61-49ea-96a5-68c34e83db3b",
"toSign":"c37d661d-7e61-49ea-96a5-68c34e83db3bc37d661d-7e61-49ea-96a5-68c34e83db3b","transactionId":"be495cb4e4cedc9790c974","transactionType":"PAY",
"transaction_fee":100,
"trade_success":true,
"notifyUrl":"https://apihz.beecloud.cn/1/pay/webhook/receiver/c37d661d-7e61-49ea-96a5-68c34e83db3b",
"channel_type":"MS",
"messageDetail":
{"refNo":"QP201510252058178751",
"amount":"1.00",
"storableCardNo":"6214836212",
"tradeSuccess":true,
"tranRespCode":"00",
"custId":"72Q/w78hc+GbrmEhBMy8KKdSH1TTt5tZ",
"respMsg":"交易成功",
"tranCode":"QP0001",
"tranFlow":"117220151025210446cedc9790c97",
"respCode":"C000000000",
"merchantNo":"1172",
"merOrderId":"be495cb4e4cedc9790c974"},
"message_detail":{
"refNo":"QP201510252058178751",
"amount":"1.00",
"storableCardNo":"6214836212",
"tradeSuccess":true,
"tranRespCode":"00",
"custId":"72Q/w78hc+GbrmEhBMy8KKdSH1TTt5tZ",
"respMsg":"交易成功",
"tranCode":"QP0001",
"tranFlow":"117220151025210446cedc9790c97",
"respCode":"C000000000",
"merchantNo":"1172",
"merOrderId":"be495cb4e4cedc9790c974"},
"trade_success":true}

*参数含义*

key  | value
---- | -----
transaction_type | PAY/REFUND  分别代表支付和退款的结果确认
transaction_fee | 交易金额，是以分为单位的整数，对应支付请求的total\_fee或者退款请求的refund\_fee
trade_success | true  交易成功
channel_type | MS 民生
message_detail | {orderId:xxx…..} 用一个map代表处理结果的详细信息，例如支付的订单号，金额， 商品信息
optional| 附加参数，为一个JSON格式的Map，客户在发起购买或者退款操作时添加的附加信息

*部分字段含义*

  Key             | 类型           | Example               | 含义
-------------     | ------------- | -------------         | -------
refNo | String | QP201510231008298527 | 系统参考号，即渠道交易号
amount| String| 1.00| 金额
storableCardNo| String| 6217907388| 短卡号
custId| String | iGfK3cetpDVsdSV18bmfcwUAMLiw1c8wN0kTUjDnGdE= | 通过数据密钥加密，custId经过md5加密之后的值
tranCode | String | QP0001 | 交易响应码
merOrderId | String | 20151023101407 | 订单号

## 10. WebHooK（网关支付）
{"retryCounter":0,
"transaction_id":"a030d58be4d333ec46118f7d266",
"retry_counter":0,
"channelType":"MS",
"sub_channel_type":"MS_WEB",
"optional":{},
"transaction_type":"PAY",
"notify_url":"https://apihz.beecloud.cn/1/pay/webhook/receiver/c37d661d-7e61-49ea-96a5-68c34e83db3b",
"toSign":"c37d661d-7e61-49ea-96a5-68c34e83db3bc37d661d-7e61-49ea-96a5-68c34e83db3b",
"transactionId":"a030d58be4d333ec46118f7d266",
"transactionType":"PAY",
"transactionFee":1,
"tradeSuccess":true,
"notifyUrl":"https://apihz.beecloud.cn/1/pay/webhook/receiver/c37d661d-7e61-49ea-96a5-68c34e83db3b",
"channel_type":"MS",
"messageDetail":
{"subject":"1172001",
"remark":"demo测试",
"banksendtime":"2015-10-25 18:34:09.0",
"payorderid":"201510251834090006981",
"interface":"5.00",
"mac":"5e9d2136ca9216aec6b25919722ac992",
"currencytype":"01",
"paybank":"BOS",
"merchantid":"1172",
"tradeSuccess":true,
"merorderid":"a030d58be4d333ec46118f7d266",
"state":"1",
"amountsum":"0.01",
"merrecvtime":"2015-10-25 18:34:11"},
"message_detail":{"subject":"1172001",
"remark":"demo测试",
"banksendtime":"2015-10-25 18:34:09.0",
"payorderid":"201510251834090006981",
"interface":"5.00",
"mac":"5e9d2136ca9216aec6b25919722ac992",
"currencytype":"01",
"paybank":"BOS",
"merchantid":"1172",
"tradeSuccess":true,
"merorderid":"a030d58be4d333ec46118f7d266",
"state":"1",
"amountsum":"0.01",
"merrecvtime":"2015-10-25 18:34:11"
},"trade_success":true}
*参数含义*

key  | value
---- | -----
transaction_type | PAY/REFUND  分别代表支付和退款的结果确认
transaction_fee | 交易金额，是以分为单位的整数，对应支付请求的total\_fee或者退款请求的refund\_fee
trade_success | true  交易成功
channel_type | MS 民生
message_detail | {orderId:xxx…..} 用一个map代表处理结果的详细信息，例如支付的订单号，金额， 商品信息
optional| 附加参数，为一个JSON格式的Map，客户在发起购买或者退款操作时添加的附加信息

*部分字段含义*

  Key             | 类型           | Example               | 含义
-------------     | ------------- | -------------         | -------
subject | String | 1172001 | 商品种类，由渠道分配
remark | String| 111| 跟传入值title相同
banksendtime | String| 2015-10-23 10:26:03.0| 发送到银行时间
payorderid | String | 20151023102603091623 | 渠道内部交易号
paybank | String | BOS | 银行缩写
merorderid | String | 20151023101407 | 订单号
state | String | 1 | 支付状态
merrecvtime| String | 2015-10-23 10:26:05 | 返回到商户时间

## 11. 测试环境和生产环境配置参数（网关支付）

在测试环境下，以上1-9涉及的接口，都需要向民生那边发起请求。而后端**<mark>务必<mark>**配置这些请求的地址。具体地址如下：

  Key            | 含义         | 类型           
------------- | -------------    | -------------
msWebSignUrl | 网关支付请求地址 |http://netpay.umbpay.com.cn:8086/pay2\_1\_/paymentImplAction.do  
msKjSignPayUrl | 快捷支付请求地址 |http://qpay.umbpay.com.cn:8086/QuickPay/msgProcess/acceptReq.do
msWebRefundUrl | 网关退款请求地址 |http://netpay.umbpay.com.cn:10086/mer/merAutoRefund.do
msKjQueryByOneUrl | 快捷单笔查询请求地址 |http://qpay.umbpay.com.cn:8086/QuickPay/msgProcess/acceptReq.do
msWebQueryByOneUrl | 网关单笔查询请求地址 |http://netpay.umbpay.com.cn:8086/pay2\_1\_/merCheckOut.do
msWebRefundQueryUrl | 网关退款查询地址 |http://netpay.umbpay.com.cn:10086/mer/merRefundQuery.do
msWebQueryUrl | 网关订单批量查询地址 |http://netpay.umbpay.com.cn:8086/pay2\_1\_/merAccountCheck.do

在生产环境下，也需要后端配置这些请求的地址。但这些地址需要民生那边给出。

## 12. 注意点

- 退款时，民生没有给异步通知，因此需要通过退款查询来更改退款状态；
- 批量订单，当交易量非常大的时候，民生提示这样操作会相当耗资源，因此建议使用民生商户平台导出的账单；
- 