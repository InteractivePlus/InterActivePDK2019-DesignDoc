# 后端API标准

[查看后端所有文档](./)   

此文档将详细规范OPENAPI4.1 (InterActivePDK 2020 Backend) 开发中的API交互设计.   

适用Backend版本: <= 1.0.0

## 1.0 设计理念

形随意动OPENAPI 4.1在设计后端API时着重考虑到API的易读性和可理解性, 使用Restful API结构.   

API域名: pdkapi.interactiveplus.org/

### 1.1 API URL 细则

所有API的URL皆使用以下的命名规则:   

```
https://pdkapi.interactiveplus.org/版本号/API所属类型/操作对象/可拓展内容
``` 

让我们来看下下面的例子:   
```
https://pdkapi.interactiveplus.org/v1/interactiveLiveID/token
```

## 2.0 官网API

官网API定义了和形随意动用户系统前端(InterActiveLiveID)交互所需的所有API.

### 2.1 通用用户API

#### 2.1.1 登录API

请求方法:   
GET   

URL:   

```
/v1/interactiveLiveID/token   
```

Captcha ActionID: 1   

附带GET参数:   

|参数名|类型|解释|
|-|-|-|
|accountType|int|登录账户类型, 0 = 用户名, 1 = 邮箱, 2 = 手机|
|account|string|用户账户|
|password|string|密码原文|
|captchaPhrase|mixed|验证码|

返回HTTP Status Code:   

|Status Code|解释|
|-|-|
|200|请求完全正常|
|400|请求中部分参数格式不正确或者没有填写,或者验证码填写错误|
