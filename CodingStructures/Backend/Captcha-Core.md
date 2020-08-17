# 验证码 Captcha-Core 标准

[查看后端所有文档](./)   

此文档将详细规范OPENAPI4.1 (InterActivePDK 2020 Backend) 开发中的验证码核心库架构设计, 数据库结构设计和代码交互逻辑. 此文档类似于CoreLib的设计库, 但外部API在使用时可以选择不使用此库.   

## 1.0 数据库结构

分表:

|表名|表用途|
|-|-|
|captchas|储存验证码|

### 1.1 captchas表

|字段名|数据类型|解释|算法|注释|
|-|-|-|-|-|
|phrase|VARCHAR(5)|验证码口令|-|不区分大小写, 小写储存|
|gen_time|INT|验证码生成时间|-|-|
|expire_time|INT|验证码过期时间|-|-|
|clientAddr|VARCHAR(40)|用户IP地址|-|-|
|actionID|INT|用途|-|见CoreLib中定义的Log ActionID|