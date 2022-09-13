* 目录
> [接口调用规则](#apiRule)
>
> [业务接口](#bussinessApi)
>
> * [支付下单接口](#payOrder)
> * [代付下单接口](#remitOrder)
> * [账户余额查询接口](#balanceQuery)
> * [支付（代付）回调通知](#payNotify)
> * [支付（代付）订单查询](#payOrderQuery)
> * [代付订单反查](#remitReverse)
> 
> [银行列表](#bankList)
> 
> [附录A](#appendixA)
>
> [附录B](#appendixB)

## <div id="apiRule">接口调用规则</div>
### 接口规范
1. 所有接口基于【HTTP】协议，采用【POST】方式请求；
2. 请求字符集统一用UTF-8；
3. 所有提交格式均使用JSON(Content-Type: application/json)；
4. 参数名称和参数说明中规定的固定值必须与列表中完全一致（大小写敏感）；

### 加密算法
1.  使用RSA非对称加密；
2. 待加密协议体指文档下文中的data包含的属性。

示例：

对以下系统参数值：
```
公钥(Key)：

MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCBtRZ9+GYet6bygAFZYdrjqYiIoDNxGPLNVObO4UoBHwuKdO8HCe/d2Xr2mJLNwgWu1MElEvZwtKMHTxtQ/xK+EX9waAawPEIrCm/bRe5hKS+Rk1TD2wzi6XAi8hFnzIu4FssMftNTWfnfMsYMxjeSKExg00VyR2zzBJoUGLz9vQIDAQAB
```
加密字符串为：
```
{"amount":10,"bankCode":"ICBC","expireTime":10,"merOrderNo":"123456789","notifyUrl":"http://127.0.0.1","orderIp":"127.0.0.1","payType":"ebank","submitTime":1555674515740,"returnViewUrl":"","sign":"75ED5B21E2718453AFAB5CBB83B8615E"}
```
加密结果为：
```
NIfHUX3zJ7LhS1jYjhIMHArBU0xKJxs2PUW9KE2S2ekhL3bePmuW6p+FT1P7lY5bzlkxFqYty7C/wBAVW7PLZ8LD13861KswyvJjhmQ/PYz13B0jZ/quZst/s707Al2kFsYDvPtqQ45aia6mq/9zvi8I20gZKA0SLkL7NqOJQUpdHZAIxyLjnh3ckOdyVBs4p69hIyV5S6cvmcPajLtn5so0zUvNV3OIrX9ff7VhvV8F8DxtRnfe+ml05+lUUJfNH1idQxwDk5VvfQVAdIe1H9Yo5AIVDCNqxs5XcdQ1OOZilRMSmNxZ10rraZtxkj0K3UGD6iOiAenveboRzEPcpw==
```
### <div id="signRule">签名规则</div>
对待签名字符串用 32 位 MD5 加密，得到的结果大写，得到最终的参数签名结果

示例：

参数列表(按照Key的自然排序（ascii）规则)
```
key1=val1&key2=val2&key3=val3
```

待签名内容： signKey=签名密钥，data=参数列表
```
data+"&key="+signKey
```
签名生成： signStr=待签名内容
```
MD5(signStr,"UTF-8")
```

## <div id="bussinessApi">业务接口</div>
### <div id="payOrder">支付下单接口</div>
#### 基本信息
请求路径：  /api/payOrder/prepay

#### 请求
1. 请求参数

参数代码 |参数名称 |类型 |必须 |备注
---|---|---|---|---
merId	|商户编号	|String	|是	|平台分配的商户ID
version	|版本号	|String	|是	|固定值：1.1
data	|业务加密数据	|String	|是	

data业务数据

参数代码 |参数名称	|类型 |必须 |参与签名 |备注
---|---|---|---|---|---
merOrderNo	|商户订单号	|String	|是	|是	
amount	|支付金额	|Decimal	|是	|是	|单位（元）
payType	|支付类型	|String	|是	|是	|网银转账：ebank 支付宝：alipay
businessType |订单类型 |Integer |是 |是 |订单类型：1（对私）,2（对公）<br>固定值：1
notifyUrl |回调地址	|String	|是	|是	
expireTime |过期时间	|Int	|是	|是	|订单有效时间（单位：分钟）
bankCode |银行编码	|String	|是	|是	|参考[附录B](#appendixB)章节
orderIp	|订单请求IP	|String	|是	|是	
submitTime |提交时间	|String	|是	|是	|时间戳格式(毫秒) <br>与服务器时间不能相差一天
returnViewUrl |回显地址 |String  |否 |否	
sign |签名信息	|String	|是	|否	|参照[签名规则](#signRule)
remarks	|备注	|String	|否	|否


2. 请求示例
```
{
	"merId":"200411200200001",
	"version":"1.1",
	"data":"PDaNB9p6IXL4QB1rSJwZBZld9JOWDtc3vg4HgEpYuIQgdnVZCilCbgZLp7+/A5YCB6Dz7BnxkEfH1D74u7Npxh/NdZYcQUZYrPPnEWTVXaxBOxmvEykezH9dBObRlPYZNItV+HPaZgAVfFHYIof/s4shiKw5VodObY0H5YNwwR1p+PCJ1/CUBCGoaQft4ByYUHCTYPd/X4XLQjC/nmDwSOpu1aLTUmsCf8vaHv62lTDiiimBZteH/9FqLVQ/Eoh6sX+eKsR2MU8ABUxvckag/TMNFNI/gK5YldYC0PsBNNPEQmnfwpQygt7z+BXVI/7CkvPlLX0bQc3akrIPrD4knA=="
}
```
#### 响应
1. 响应参数

接口响应为Json格式

参数代码	|参数名称	|类型 |必须 |备注
---|---|---|---|---
code	|响应编码	|Integer	|是	|参考[附录A](#appendixA)章节（返回正确code码）表示可用
message	|响应文本信息	|String	|是	
data	|响应业务消息体	|String	|是	

data业务数据

参数代码 |参数名称	|类型 |必须 |备注
---|---|---|---|---
payUrl |支付地址 |String |是 |支付地址

2. 响应示例

提交成功响应示例
```
{
    "code": 200,
    "message": "操作成功",
    "data": {
        "payUrl": "http://127.0.0.1:8888/api/payOrder/submit/2004112006370927003"
    }
}
```
提交失败响应示例
```
{
    "code": 998,
    "message": "版本号不能为空",
    "data": null
}
```

### <div id="remitOrder">代付下单接口</div>
#### 基本信息
请求路径： /api/remitOrder/submit
#### 请求
1. 请求参数

参数代码	|参数名称	|类型	|必须	|备注
---|---|---|---|---
merId	|商户编号	|String	|是	
version	|版本号	|String	|是	|固定值：1.1
data	|参数列表	|String	|是	|使用RSA加密

data业务数据

参数代码	|参数名称	|类型	|必须	|参与签名	|备注
---|---|----|---|---|---
merOrderNo	|商户订单号	|String	|是	|是	
amount	|金额	|Decimal	|是	|是	|单位（元）
submitTime	|订单提交时间	|String	|是	|是	|时间戳格式(毫秒)，与服务器时间不能相差24小时
notifyUrl	|回调地址	|String	|是	|是	
bankCode	|银行编码	|String	|是	|是	
bankAccountNo	|银行账号	|String	|是	|是	
bankAccountName	|持卡人名字	|String	|是	|是	
bankBranchName	|分行名称	|String	|否	|否	
sign	|签名信息	|String	|是	|否	|参照[签名规则](#signRule)
remarks	|订单备注	|String	|否	|否	

2. 请求示例
```
{
	"merId":"20000500",
	"version":"1.1",	"data":"YVXwPh4J97PGHnQnVcJt/rzQew4vvZ5Vk7eHxo/HFR+X3TAjv/e8F3SPJhIVBkUxa/uYkVHb+Nw1FnUUyQfmJ4xVCnQGvUk6QLxz59T/jfGKyHNP5UBanvzKFAoz/L7o5d4729yYGz8ojXrqLRizQChGJGWd7GX6/OEuSGOM4BQ="
}
```
#### 响应
1. 响应参数

接口响应为JSON格式

参数代码	|参数名称	|类型	|必须	|备注
---|---|---|---|---
code	|响应编码	|Integer	|是	|参考[附录A](#appendixA)章节
message	|响应文本信息	|String	|是	

2. 响应示例

查询成功响应示例
```
{
   "code":200,
   "message":"成功",
}
```

查询失败响应示例
```
{
	"code": 999,
	"message": "系统异常",
}
```

### <div id="balanceQuery">账户余额查询接口</div>
#### 基本信息
请求路径：/api/balance/query

> <span style="color:yellow">注：账户余额度查询与代付RSA秘钥和签名秘钥使用同一套。</span>

#### 请求

请求参数

参数代码	|参数名称	|类型	|必须	|备注
---|---|---|---|---
merId	|商户编号	|String	|是	
version	|版本号	|String	|是	|固定值：1.1
data	|参数列表	|String	|是	|使用RSA加密

data业务数据

参数代码	|参数名称	|类型	|必须	|参与签名	|备注
---|---|---|---|---|---
merId	|商户号	|String	|是	|是	
sign	|签名信息	|String	|是	|否	|参照[签名规则](#signRule)

#### 响应
响应参数

接口响应为Json格式

参数代码	|参数名称	|类型	|必须	|备注
---|---|---|---|---
code	|响应编码	|Integer |是	|参考[附录A](#appendixA)章节（返回正确code码）表示可用
message	|响应文本信息	|String	|是	
data	|响应业务消息体	|String	|是	

data业务数据

参数代码	|参数名称	|类型	|必须	|参与签名	|备注
---|---|---|---|---|---
usableAmount	|可用余额	|Decimal	|是	|是	|账户可用余额单位（元）
frezeeAmount	|冻结金额	|Decimal	|是	|是	|单位（元）
merId	|商户ID	|String	|是	|是	
sign	|签名	|String	|是	|否	

### <div id="payNotify">支付（代付）回调通知</div>
#### 基本信息

接口请求路径： ${notifyUrl}

> 注：此请求的服务地址由接入系统在提交订单请求的时候，通过参数【notifyUrl】指定，如果充值时没有指定此参数，则平台不会推送改订单的状态信息。

#### 回调规则
1.	回调请求收到正确的响应（success）之后才表示该订单状态推送完成，否则当成接入系统没有正确收到该状态推送；
2.	推送次数达到最大次数（18次）或者接入系统返回正确响应后，平台不再推送订单状态。

#### 请求
1. 请求参数

参数代码	|参数名称	|类型	|必须	|备注
---|---|---|---|---
merOrderNo	|商户订单号	|String	|是	|商户订单号
merId	|商户编号	|String	|是	
data	|参数列表	|String	|是	|使用RSA加密

data业务数据

参数代码	|参数名称	|类型	|必须	|签名	|备注
---|---|---|---|---|---
merOrderNo	|商户订单号	|String	|是	|是	|商户订单号
orderState	|订单状态	|Integer	|是	|是	|订单状态(0=处理中，1=成功，2=失败)
sign	|签名信息	|String	|是	|否	|参照[签名规则](#signRule)
orderNo	|平台订单号	|String	|是	|是	
amount	|订单金额	|Decimal	|是	|是	

2. 请求示例
```
{
	"merId":"20000500",
	"merOrderNo":"123456",
	"data":"YVXwPh4J97PGHnQnVcJt/rzQew4vvZ5Vk7eHxo/HFR+X3TAjv/e8F3SPJhIVBkUxa/uYkVHb+Nw1FnUUyQfmJ4xVCnQGvUk6QLxz59T/jfGKyHNP5UBanvzKFAoz/L7o5d4729yYGz8ojXrqLRizQChGJGWd7GX6/OEuSGOM4BQ="
}
```

#### 响应
1. 响应格式

   响应为【success】表示接入系统正确收到推送状态，该订单的推送完成。


2. 响应示例
```
success
```

### <div id="payOrderQuery">支付（代付）订单查询</div>
#### 基本信息

支付查询接口请求路径： /api/payOrder/query

代付查询接口请求路径： /api/remitOrder/query

#### 请求
1. 请求参数

参数代码	|参数名称	|类型	|必须	|备注
---|---|---|---|---
merId	|商户编号	|String	|是	
version	|版本号	|String	|是	|固定值：1.1
data	|参数列表	|String	|是	|使用RSA加密

data业务数据

参数代码	|参数名称 |类型	|必须	|备注
---|---|---|---|---
merOrderNo	|商户订单号	|String	|是	|商户订单号（参与签名）
submitTime	|订单提交时间	|String	|是	|时间戳格式(毫秒)（参与签名） 非当前时间戳
sign	|签名信息	|String	|是	|参照[签名规则](#signRule)

> <span style="color:yellow">注：“submitTime”指支付或者代付的订单提交时间（或者与订单提交时间相差不能超过24小时），使用任意时间可能会导致订单查询不存在。</span>

2. 请求示例
```
{
	"merId":"20000500",
	"version":"1.1",
	"data":"YVXwPh4J97PGHnQnVcJt/rzQew4vvZ5Vk7eHxo/HFR+X3TAjv/e8F3SPJhIVBkUxa/uYkVHb+Nw1FnUUyQfmJ4xVCnQGvUk6QLxz59T/jfGKyHNP5UBanvzKFAoz/L7o5d4729yYGz8ojXrqLRizQChGJGWd7GX6/OEuSGOM4BQ="
}
```
#### 响应
响应参数

接口响应为JSON格式

参数代码	|参数名称	|类型	|必须	|备注
---|---|---|---|---
code	|响应编码	|Integer |是	|参考[附录A](#appendixA)
message	|响应文本信息	|String	|是	
data	|参数列表	|String	|是	

data业务数据

参数代码	|参数名称	|类型	|必须	|签名	|备注
---|---|---|---|---|---
merOrderNo	|商户订单号	|String	|是	|是	|商户订单号
orderState	|订单状态	|Integer	|是	|是	|订单状态(0=处理中，1=成功，2=失败)
sign	|签名信息	|String	|是	|否	|参照[签名规则](#signRule)
orderNo	|平台订单号	|String	|是	|是	
amount	|订单金额	|Decimal	|是	|是	
merId	|商户ID	|String	|是	|是	

### <div id="remitReverse">代付订单反查</div>

> 注：反查地址需要商户向平台方运营人员申请，由运营人员绑定此地址，如果代付时没有绑定此地址或未开启反查功能，则平台不会调用反查。

#### 请求
请求参数

参数代码	|参数名称	|类型	|必须	|签名	|备注
---|---|---|---|---|---
merId	|商户编号	|String	|是	|是	
merOrderNo	|商户订单号	|String	|是	|是	
bankAccountNo	|银行卡号	|String	|是	|是	
bankAccountName	|持卡人姓名	|String	|是	|是	
amount	|提交金额	|Decimal	|是	|是	
submitTime	|订单提交时间	|String	|是	|是	|时间戳（毫秒）
sign	|签名信息	|String	|是	|否	|参照[签名规则](#signRule)

#### 响应
响应参数

接口响应为JSON格式

参数代码	|参数名称	|类型	|必须	|签名	|备注
---|---|---|---|---|---
merId	|商户ID	|String	|是	|是	
merOrderNo	|商户订单号	|String	|是	|是	
submitTime	|代付提交时间	|String	|是	|是	|时间戳（毫秒）
code	|响应码	|String	|是	|是	|参考[附录B](#appendixB)（返回正确code码）表示可用
message	|响应描述	|String	|是	|是	|响应描述,不能为空
sign	|签名	|String	|是	|否	|参照[签名规则](#signRule)

## <div id="bankList">银行列表</div>

银行名称	|编码
---|---
工商银行	|ICBC
建设银行	|CCB
农业银行	|ABC
邮政储蓄银行	|PSBS
中国银行	|BOC
交通银行	|BOCO
招商银行	|CMB
光大银行	|CEB
兴业银行	|CIB
民生银行	|CMBC
北京银行	|BCCB
中信银行	|CTTIC
广东发展银行	|GDB
深圳发展银行	|SDB
浦东发展银行	|SPDB
平安银行	|PINGANBANK
华夏银行	|HXB
上海银行	|SHB
渤海银行	|CBHB
东亚银行	|HKBEA
宁波银行	|NBCB
浙商银行	|CZB
南京银行	|NJCB
杭州银行	|HZCB
北京农村商业银行 |BJRCB
上海农商银行	|SRCB
齐鲁银行	|QLB
兰州银行	|LZCB
桂林银行	|GLB
青岛银行	|QDCB
广西北部湾银行	|BOBBG
其他银行（未在上方出现的所有银行，<br>主要用于支持地方小银行）	|OTHER


## <span id='appendixA'>附录A</span> - 接口状态码定义

状态码 |描述
---|---
200	|成功
550	|商户不存在
551	|商户不可用
552	|协议解析失败
553	|签名错误
554	|订单不存在
555	|商户组信息有误
556	|没有可用通道
557	|商户信息有误
558	|银行卡受限
559	|账户余额不足
560	|通道提交异常
561	|IP鉴权失败
562	|接口全选受限

## <span id='appendixB'>附录B</span> - 订单反查状态码定义

状态码 |描述
---|---
0   |成功
1001 |签名错误
1002 |订单不存在
1003 |银行卡号不匹配
1004 |金额不匹配
9999 |系统未知异常
