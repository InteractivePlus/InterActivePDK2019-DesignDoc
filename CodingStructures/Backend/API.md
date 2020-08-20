# 后端API标准

[查看后端所有文档](./)   

此文档将详细规范OPENAPI4.1 (InterActivePDK 2020 Backend) 开发中的API交互设计.   

适用Backend版本: <= 1.0.0

## 1.0 设计理念

形随意动OPENAPI 4.1在设计后端API时着重考虑到API的易读性和可理解性, 使用Restful API结构.   

API域名: pdkapi.interactiveplus.org/   

注意: 全局POST的Request必须设置Content-Type为application/json   

### 1.1 API URL 细则

所有API的URL皆使用以下的命名规则:   

```
https://pdkapi.interactiveplus.org/版本号/API所属类型/操作对象/可拓展内容
``` 

让我们来看下下面的例子:   
```
https://pdkapi.interactiveplus.org/v1/interactiveLiveID/token
```

### 1.2 全局Get参数

任何 MultiLang API:

|参数名|类型|解释|
|-|-|-|
|area|string|CN\|US|
|locale|string|zh_CN\|en_US|

## 2.0 官网API

官网API定义了和形随意动用户系统前端(InterActiveLiveID)交互所需的所有API.

### 2.1 基础API

#### 2.1.1 获取新验证码API

请求方法: `GET`      

URL:   

```
/v1/common/captcha   
```

ActionID: `80001`   

附带GET参数:   

|参数名|类型|解释|
|-|-|-|
|actionID|int|验证码用途|
|clientID|string|第三方APP ClientID(如果用于官网,则留空)|
|token|string|用户token(可选),是官网APP且需要token时填写|
|access_token|string|OAuth Access Token(可选), 为OAuth APP时填写|
|width|int|宽度(px), 不填默认150|
|height|int|高度(px), 不填默认40|

正常返回HTTP Status Code: `201 CREATED`   

返回 HTTP Body JSON数据:   

|数据键名|类型|解释|算法|注释|
|-|-|-|-|-|
|errCode|int|错误代码|-|0 = 无错误|
|errMessage|string|错误详情|-|-|
|errContext|array|错误环境|-|-|
|captchaInfo|array|验证码详细信息|-|仅在无错误时提供|

captchaInfo键位数据:   

|数据键名|类型|解释|算法|注释|
|-|-|-|-|-|
|captchaWidth|int|验证码宽|-|单位为像素(pixels)|
|captchaHeight|int|验证码高|-|px|
|captchaData|string|验证码数据|base64_encode(JpegData)|-|
|expires|int|过期UNIX时间戳|-|UTC Unix Timestamp|

### 2.2 通用用户API

#### 2.2.1 登录API

此API是**MultiLang API**   

请求方法: `GET`      

URL:   

```
/v1/interactiveLiveID/token
```

ActionID: `10001`   

附带GET参数:   

|参数名|类型|解释|
|-|-|-|
|accountType|int|登录账户类型, 0 = 用户名, 1 = 邮箱, 2 = 手机|
|account|string|用户账户|
|password|string|密码原文|
|captchaPhrase|mixed|验证码|

正常返回HTTP Status Code: `201 Created`   

返回 HTTP Body JSON数据:   

|数据键名|类型|解释|算法|注释|
|-|-|-|-|-|
|errCode|int|错误代码|-|0 = 无错误|
|errMessage|string|错误详情|-|-|
|errContext|array|错误环境|-|-|
|tokenInfo|array|登录凭据详细信息|-|仅在无错误时提供|

tokenInfo键位数据:   

|数据键名|类型|解释|算法|注释|
|-|-|-|-|-|
|uid|int|用户UID|-|-|
|token|string|登录凭据|-|-|
|refresh_token|string|刷新凭据|-|-|
|expires|int|过期时间(UTC)|-|-|
|refresh_expires|int|刷新令牌过期时间(UTC)|-|-|

#### 2.2.2 注册API

此API是**MultiLang API**   

请求方法: `POST`      

URL:   

```
/v1/interactiveLiveID/user
```

ActionID: `10002`   

附带POST参数:   

|参数名|类型|解释|
|-|-|-|
|accountType|int|登录账户类型, 1 = 邮箱, 2 = 手机|
|account|string|用户账户|
|username|string|用户名|
|displayName|string|展示名|
|password|string|密码原文|
|captchaPhrase|mixed|验证码|

正常返回HTTP Status Code: `201 Created`   

返回 HTTP Body JSON数据:   

|数据键名|类型|解释|算法|注释|
|-|-|-|-|-|
|errCode|int|错误代码|-|0 = 无错误|
|errMessage|string|错误详情|-|-|
|errContext|array|错误环境|-|-|
|uid|int|用户UID|-|-|

#### 2.2.3 验证Token API

请求方法: `GET`      

URL:   

```
/v1/interactiveLiveID/validationResult/token
```

ActionID: `10003`   

附带GET参数:   

|参数名|类型|解释|算法|注释|
|-|-|-|-|-|
|token|string|登录凭据|-|-|

正常返回HTTP Status Code: `200 Ok`   

返回 HTTP Body JSON数据:   

|数据键名|类型|解释|算法|注释|
|-|-|-|-|-|
|errCode|int|错误代码|-|0 = 无错误|
|errMessage|string|错误详情|-|-|
|errContext|array|错误环境|-|-|
|expires|int|过期时间(UTC UNIX TIMESTAMP)|-|-|
|uid|int|用户uid|-|-|

#### 2.2.4 刷新Token API

请求方法: `POST`      

URL:   

```
/v1/interactiveLiveID/token
```

ActionID: `10004`   

附带POST参数:   

|参数名|类型|解释|算法|注释|
|-|-|-|-|-|
|refresh_token|string|刷新凭据|-|-|

正常返回HTTP Status Code: `201 Created`   

返回 HTTP Body JSON数据:   

|数据键名|类型|解释|算法|注释|
|-|-|-|-|-|
|errCode|int|错误代码|-|0 = 无错误|
|errMessage|string|错误详情|-|-|
|errContext|array|错误环境|-|-|
|newTokenInfo|int|新登录凭据详细信息|-|-|

newTokenInfo键位数据:   

|数据键名|类型|解释|算法|注释|
|-|-|-|-|-|
|uid|int|用户UID|-|-|
|token|string|登录凭据|-|-|
|refresh_token|string|刷新凭据|-|-|
|expires|int|过期时间(UTC)|-|-|
|refresh_expires|int|刷新令牌过期时间(UTC)|-|-|

## 3.0 API错误代码一览

|错误代码|解释|批注|环境变量(Context)|HTTP Status Code|
|-|-|-|-|-|
|10001|Credential Incorrect|-|credential = `request_param_name`|401|
|10002|Credential Format Incorrect|-|credential = `request_param_name`|400|
|10003|Credential expired|-|credential = `request_param_name`|401|
|20001|Item not found|-|item=`request_param_name`|404|
|20002|Item already exist|-|item=`request_param_name`|409|
|30001|Permission Denied|-|-|403|
|30002|Account Frozen|-|-|403|
|30003|Account not verified|-|contact="phone"/contact="email"|403|
|90001|Operation Too Frequent|-|-|403|
|90002|System Busy|-|-|503|
|90003|Internal Error|-|pdkErrCode, pdkErrDescription, pdkErrParam|500|