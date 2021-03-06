# 后端CoreLib架构

[查看后端所有文档](./)   

此文档将详细规范OPENAPI4.1 (InterActivePDK 2020 Backend) 开发中的核心库架构设计, 数据库结构设计和代码交互逻辑.

---

文档对应CoreLib版本: <= 0.1.0


## 1.0 基本概念
OPENAPI4.1 底层架构致力于简化开发过程中繁琐的代码重合和交互逻辑, 我们希望将每个功能区块完整的区分出来并明确他们之间的界限. 同时底层的交互逻辑应减少与数据库连接的次数, 减少服务器总体压力, 提升性能表现.   
OPENAPI4.1 架构方面将分成两块, 核心功能层和API功能层.   

### 1.1 核心功能层模块介绍

|模块名|模块功能|
|-|-|
|DatabaseModule|基础模块之一, 提供全局MySQLi的连接|
|LogModule|基础模块之一, 提供其他模块记录系统日志|
|UserModule|用户创建/查找/删除, 用户名密码验证/修改, 邮箱/手机查看/验证/修改, 头像昵称查看/修改, 用户权限/用户组查看/修改, 用户设置查看/修改, 主站Token分配/删除, 查找用户|
|NotificationModule|提醒消息查找/删除, 给用户发送重置密码邮件/短信, 验证码, 第三方APP邮件, 主站提醒|
|ThirdAPPModule|第三方APP创建/删除/查找, 第三方APP信息(回调地址, 简介, APP Logo等)查看/修改, 第三方APP拥有者查看/修改, 第三方APP管理组名单查看/修改|
|ThirdAuthModule|第三方授权登录公开接口, 这个模块只是定义一个Autoload Class类, 并规定固定的类接口给第三方授权登录APP|
|UserAuthModule|此模块管理用户对特定APP的授权信息. 支持查找/创建/修改/删除授权|
|UserGroupModule|此模块管理用户组, 可以查找/创建/修改/删除用户组, 修改用户组权限|
|OAuthModule|此模块管理OAuth相关信息, 包括第三方APP临时访问Authorization-Grant, 支持创建/编辑/查找/删除Authorization-Grant|

## 2.0 加密方法定义

To be filled.   

## 3.0 在设置中可更改的部分变量注解

|变量名|解释|变量类型|
|-|-|-|
|USER_SYSTEM_NAME|用户系统名称|MultiLang[string]|
|THIRD_PARTY_SYSTEM_NAME|第三方APP系统名称|MultiLang[string]|
|USER_SYSTEM_LINKS|用户系统回调链接([见格式](#82-用户系统回调链接格式))|MultiLang[array[string=>string]]|
|THIRD_PARTY_SYSTEM_LINKS|第三方系统回调链接(以后完善格式)|MultiLang[array[string=>string]]|
|USERNAME_MINLEN|用户名/用户组名最小长度|int|
|USERNAME_MAXLEN|用户名/用户组名最大长度|int|
|USERNAME_REGEX|用户名合法性验证正则(可留空)|string|
|DISPLAYNAME_MINLEN|展示名最小长度|int|
|DISPLAYNAME_MAXLEN|展示名最大长度|int
|DISPLAYNAME_REGEX|展示名合法性验证正则(可留空)|string|
|SIGNATURE_MAXLEN|用户签名最大长度|int|
|SIGNATURE_REGEX|用户签名合法性验证正则(可留空)|string|
|EMAIL_MAXLEN|邮箱地址最大长度|int|
|PASSWORD_MINLEN|密码最小长度|int|
|PASSWORD_MAXLEN|密码最大长度|int|
|PASSWORD_REGEX|密码合法性验证正则(可留空)|string|
|AVATOR_MAX_SIZE|用户头像最大大小(kB)|int|
|PASSWORD_SALT|密码加密Salt|string|
|TOKEN_AVAILABLE_DURATION|TOKEN有效时间(秒)|int|
|REFRESH_TOKEN_AVAILABLE_DURATION|刷新TOKEN有效时间(秒)|int|
|VERIFICATION_CODE_AVAILABLE_DURATION|验证码有效时间(秒)|int|
|OAUTH_AUTH_CODE_AVAILABLE_DURATION|OAuth接口的授权码有效时间(秒)|int|
|OAUTH_ACCESS_TOKEN_AVAILABLE_DURATION|OAuth接口的Access Token有效时间(秒)|int|
|OAUTH_REFRESH_TOKEN_AVAILABLE_DURATION|OAuth接口的Refresh Token有效时间(秒)|int|
|DEFAULT_COUNTRY|默认国家(CN,GB,US,...)|string|
|DEFAULT_LOCALE|默认语言(zh_CN, en_US, ...)|string|
|DEFAULT_GROUP_PERMISSION|默认用户组权限数组(Key=>Value)|array[string=>unknown]|
|ALLOW_TOKEN_IP_CHANGE|允许用户登录状态在更改IP后继续使用|bool|
|ALLOW_VERICODE_IP_CHANGE|允许用户在申请发送验证码和使用验证码时更改IP|bool|
|AVATAR_SIZE_PIXELS|储存时最大的头像宽和高|int|
|DEBUG_MODE|是否在调试模式中|bool|

## 4.0 数据库结构定义

**数据表名称和数据字段名需要使用小写(参照MySQL手册)**
**数据库名称建议使用大驼峰命名法**


### 4.1 数据表结构定义

|表名|表用途|
|-|-|
|user_infos|用户信息储存表|
|usergroup_infos|用户组信息储存表|
|logged_infos|用户已登录信息储存表(主站Token储存表)|
|verification_codes|用户验证码信息储存表|
|app_infos|第三方APP信息储存表|
|app_manage_infos|第三方APP管理信息储存表|
|oauth_authorization_codes|OAuth Auth Code储存表|
|oauth_tokens|OAuth Access Token / Refresh Token储存表|
|thirdauth_infos|第三方登录信息储存表|
|logs|日志信息储存表|
|avatars|用户头像储存表|

### 4.2 表内字段定义

#### 4.2.1 user_infos表

|字段名|数据类型|解释|算法|注释|
|-|-|-|-|-|
|uid|BIGINT UNSIGNED NOT NULL AUTO_INCREMENT|用户UID|-|唯一索引|
|username|VARCHAR(`USERNAME_MAXLEN`) NOT NULL|用户名|-|唯一索引|
|display_name|VARCHAR(`DISPLAYNAME_MAXLEN`) NOT NULL|用户展示名|-|唯一索引|
|signature|VARCHAR(`SIGNATURE_MAXLEN`)|用户签名|-|-|
|password|CHAR(64)|密码哈希|sha256(original,`PASSWORD_SALT`)|大写|
|locale|CHAR(6)|用户的语言|zh_CN, en_US, en_GB, etc|-|
|area|CHAR(2)|用户的地区|CN, US, GB, etc.|-|
|email|VARCHAR(`EMAIL_MAXLEN`)|邮箱|-|索引|
|phone_number|CHAR(15)|用户绑定手机号|E.164|索引|
|settings|BLOB|用户设置|gzcompress(original json object)|-|
|email_verified|TINYINT(1)|邮箱是否验证过|-|1(true) / 0(false)|
|phone_verified|TINYINT(1)|手机是否被验证过|-|1(true) / 0(false)|
|permission_override|TEXT|用户权限(覆盖组权限的内容)|gzcompress(original json object)|可留空|
|group|VARCHAR(`USERNAME_MAX`)|用户组|original|-|
|regtime|INT|用户注册时间|time()|-|
|reg_client_addr|VARCHAR(40)|用户注册时的IP地址|-|ipv4/ipv6|
|is_admin|TINYINT(1)|是否是管理员用户|-|1(true) / 0(false)|
|avatar|CHAR(32)|用户头像md5|md5(头像数据)|如果是默认头像则留空|
|is_frozen|TINYINT(1)|是否被冻结|-|1(true) / 0(false)|

#### 4.2.2 usergroup_infos表

|字段名|数据类型|解释|算法|注释|
|-|-|-|-|-|
|groupid|VARCHAR(`USERNAME_MAXLEN`) NOT NULL|组id|-|唯一索引|
|parent_group_id|VARCHAR(`USERNAME_MAXLEN`)|父组id|-|-|
|display_name|VARCHAR(`DISPLAYNAME_MAXLEN`)|组展示名|-|-|
|description|VARCHAR(`SIGNATURE_MAXLEN`)|组详细信息|-|-|
|regtime|INT|组注册时间|time()|-|
|permissions|TEXT|组基础权限|original json object|-|
|avatar|CHAR(32)|组头像md5|md5(头像数据)|如果是默认头像则留空|

#### 4.2.3 logged_infos表

|字段名|数据类型|解释|算法|注释|
|-|-|-|-|-|
|token|CHAR(32) NOT NULL|用户分配到的TOKEN|bin2hex(random_bytes(16))|唯一索引, 大写|
|refresh_token|CHAR(32) NOT NULL|用户分配到的刷新TOKEN|bin2hex(random_bytes(16))|大写, 唯一索引|
|uid|BIGINT UNSIGNED NOT NULL|token关联的用户uid|-|索引|
|issue_time|INT|token分配时间|time()|-|
|expire_time|INT|token过期时间|time() + `TOKEN_AVAILABLE_DURATION`|-|
|refresh_expire_time|INT|refresh token过期时间|time() + `REFRESH_TOKEN_AVAILABLE_DURATION`|-|
|renew_time|INT|token更新时间|time()|-|
|client_addr|VARCHAR(40)|用户客户端IP地址|original|ipv4/ipv6|

#### 4.2.4 verification_codes表

|字段名|数据类型|解释|算法|注释|
|-|-|-|-|-|
|veri_code|CHAR(32) NOT NULL|验证码|bin2hex(random_bytes(16))|唯一索引,大写|
|uid|BIGINT UNSIGNED NOT NULL|验证码关联的用户uid|-|索引|
|action_id|INT|此验证码用来做什么|-|-|
|action_param|TEXT|验证码操作参数|gzcompress(original JSON object)|-|
|sent_method|TINYINT|发送方式|-|0 = 未发送, 1 = 邮箱, 2 = 手机短信, 3 = 电话|
|issue_time|INT|验证码分配时间|time()|-|
|expire_time|INT|验证码过期时间|time() + `VERIFICATION_CODE_AVAILABLE_DURATION`|-|
|used_stage|TINYINT|是否使用过了|-|1(used), 0(valid), -1(invalid)|
|trigger_client_ip|VARCHAR(40)|用户请求此验证码时的ip地址|-|ipv4/ipv6|

#### 4.2.5 app_infos表

|字段名|数据类型|解释|算法|注释|
|-|-|-|-|-|
|appuid|BIGINT UNSIGNED NOT NULL AUTO_INCREMENT|APP的uid|-|唯一索引|
|display_name|VARCHAR(`DISPLAYNAME_MAXLEN`)|APP展示名|-|-|
|client_id|CHAR(32)|APP在使用OAuth API时的client_id|bin2hex(random_bytes(16))|唯一索引,小写|
|client_secret|CHAR(64)|APP在使用OAuth API时的client_secret|bin2hex(random_bytes(32))|可留空,小写|
|client_type|UNSIGNED TINYINT|APP类型|-|0 = first party public, 1 = first party private, 2 = trusted third party public, 3 = trusted third party private, 4 = third party public, 5 = third party private|
|reg_area|CHAR(2)|注册时用户的国家/地区|-|-|
|reg_time|INT|APP注册时间|time()|-|
|redirect_uri|VARCHAR(500)|APP回调地址|-|-|
|avatar|CHAR(32)|APP头像MD5|md5(头像数据)|如果是默认头像请留空|

#### 4.2.6 app_manage_infos表

|字段名|数据类型|解释|算法|注释|
|-|-|-|-|-|
|appuid|BIGINT UNSIGNED NOT NULL|APP的uid|-|索引|
|uid|BIGINT UNSIGNED NOT NULL|用户的uid|-|索引|
|role|TINYINT|用户角色(对应权限)|-|-|

#### 4.2.7 oauth_authorization_codes表

|字段名|数据类型|解释|算法|注释|
|-|-|-|-|-|
|authorization_code|CHAR(40) NOT NULL|Authorization Code|bin2hex(random_bytes(20))|唯一索引, 小写|
|appuid|BIGINT UNSIGNED NOT NULL|申请授权的APPUID|-|索引|
|uid|BIGINT UNSIGNED NOT NULL|申请授权的用户UID|-|索引|
|redirect_uri|VARCHAR(500)|回调地址|-|报错时这个字段如果格式不对, 抛出错误代码为30002|
|issue_time|INT|发布时间|time()|-|
|expire_time|INT|过期时间|time() + `OAUTH_AUTH_CODE_AVAILABLE_DURATION`|-|
|scope|VARCHAR(200)|授权范围|-|每个scope用空格分隔|
|code_challenge|VARCHAR(128)|Code Challenge|-|原文为43-128字符长, S256则是64,S256保存大写|
|challenge_type|TINYINT|Code Challenge类型|-|2 = S256, 1 = plain|

#### 4.2.8 oauth_tokens表

|字段名|数据类型|解释|算法|注释|
|-|-|-|-|-|
|access_token|CHAR(40) NOT NULL|Access Token|bin2hex(random_bytes(20))|唯一索引, 小写|
|refresh_token|CHAR(40)|Refresh Token|bin2hex(random_bytes(20))|索引, 可以留空, 小写|
|appuid|BIGINT UNSIGNED NOT NULL|授权的APPUID|-|索引|
|uid|BIGINT UNSIGNED NOT NULL|授权的UID|-|索引|
|issue_time|INT|发布时间|time()|-|
|access_expire_time|INT|Access Token过期时间|time() + `OAUTH_ACCESS_TOKEN_AVAILABLE_DURATION`|-|
|refresh_expire_time|INT|Refresh Token过期时间|time() + `OAUTH_REFRESH_TOKEN_AVAILABLE_DURATION`|-|
|scope|VARCHAR(200)|授权范围|-|每个scope用空格分隔|

#### 4.2.9 logs表

|字段名|数据类型|解释|算法|注释|
|-|-|-|-|-|
|actionID|int|操作actionID|-|如果没有ActionID, 则填写0|
|appuid|BIGINT UNSIGNED NOT NULL|操作appuid|-|如果是主站,appuid填写0|
|time|int|操作时间|-|索引|
|logLevel|int|日志等级|-|EMERGENCY = 8, ALERT = 7, CRITICAL = 6, ERROR = 5, WARNING = 4, NOTICE = 3, INFO = 2, DEBUG = 1|
|message|TEXT|日志信息|-|-|
|success|TINYINT|成功?|-|0 = no, 1 = yes|
|PDKExceptionCode|INT|PDK错误代码|-|无错误为0|
|context|MEDIUMBLOB|日志详细信息|gzcompress(Original JSON String)|-|
|clientAddr|VARCHAR(40)|客户端IP|-|-|

#### 4.2.10 avatars表

|字段名|数据类型|解释|算法|注释|
|-|-|-|-|-|
|hash|CHAR(32)|头像MD5|md5(original data)|唯一索引, 全小写|
|data|MEDIUMBLOB|头像数据|gzcompress(original data)|-|

## 5.0 数据库内部表键数据结构定义

### 5.1 用户/用户组权限定义
用户权限是一个多键位`JSON Object`, 每一个键位代表一项权限.

|权限键位|数据类型|解释|算法|注释|
|-|-|-|-|-|
|createApp|boolean|是否可以创建第三方APP|-|-|
|numAppLimit|integer|APP创建数量限制|-|0 = 不受限制|

### 5.2 用户设置权限定义

用户权限是一个多键位`JSON Object`, 每一个键位代表一项设置.

|设置键位|数据类型|解释|算法|注释|
|-|-|-|-|-|
|email_options|JSON Object|邮件接收设置|-|见下面邮件偏好的详细定义|
|sms_options|JSON Object|手机短信接收设置|-|见下面手机SMS消息偏好的详细定义|
|call_options|JSON Object|手机电话接收设置|-|见下面手机电话偏好的详细定义|

#### 5.2.1 邮件接收偏好

|键位|数据类型|解释|算法|注释|
|-|-|-|-|-|
|official_push_notifications|boolean|是否接收第一方APP的消息通知|-|此为总开关, 可单独控制|
|official_updates|boolean|是否接收形随意动官方新闻更新|-|-|
|official_promotions|boolean|是否接收形随意动官方产品促销信息|-|-|
|third_party_push_notifications|boolean|是否接收第三方APP的消息通知|-|此为总开关, 可单独控制|
|third_party_updates|boolean|是否接收第三方软件新闻更新|-|此为总开关, 用户要控制单独开关可在授权页面更改|
|third_party_promotions|boolean|是否接收第三方软件促销信息|-|此为总开关, 用户要控制单独开关可在授权页面更改|

#### 5.2.2 SMS接收偏好

|键位|数据类型|解释|算法|注释|
|-|-|-|-|-|
|official_push_notifications|boolean|是否接收第一方APP的消息通知|-|此为总开关, 可单独控制|
|official_updates|boolean|是否接收形随意动官方新闻更新|-|-|
|official_promotions|boolean|是否接收形随意动官方产品促销信息|-|-|
|third_party_push_notifications|boolean|是否接收第三方APP的消息通知|-|此为总开关, 可单独控制|
|third_party_updates|boolean|是否接收第三方软件新闻更新|-|此为总开关, 用户要控制单独开关可在授权页面更改|
|third_party_promotions|boolean|是否接收第三方软件促销信息|-|此为总开关, 用户要控制单独开关可在授权页面更改|

#### 5.2.3 电话偏好

|键位|数据类型|解释|算法|注释|
|-|-|-|-|-|
|allow_calls|boolean|是否允许官方对您进行定期用户体验回访和用户安全预警|-|-|

### 5.3 验证码相关定义

#### 5.3.1 验证码类型定义

|验证码action_id|名称|方式|注释|验证码Action是自动触发还是手动触发|
|-|-|-|-|-|
|10001|邮箱验证|EMAIL|-|自动|
|10002|手机验证|SMS \| PHONE_CALL|-|自动|
|20001|密码修改申请验证|ANY_METHOD|-|手动<sup>1</sup>|
|20002|修改邮箱验证|ANY_METHOD|-|自动|
|20003|修改手机验证|ANY_METHOD|-|自动|
|30001|管理员操作验证|ANY_METHOD|-|手动|
|90001|第三方APP重要操作验证|ANY_METHOD|-|手动|
|90002|第三方APP删除内容验证|ANY_METHOD|-|手动|

<sup>1</sup> action_id 为 20001 的验证码需要在接口层设计两个接口, 一个接口验证验证码是否正确, 另一个接口进行修改密码操作.   

#### 5.3.2 验证码参数定义

##### 5.3.2.10001 邮箱验证

留空

##### 5.3.2.10002 手机验证

留空

##### 5.3.2.20001 密码修改申请验证

类型: `JSON Object`   
数据定义:   

|键位|数据类型|解释|
|-|-|-|
|client_ip_addr|String|尝试修改密码的用户IP|

##### 5.3.2.20002 修改邮箱验证

类型: `JSON Object`   
数据定义:   

|键位|数据类型|解释|
|-|-|-|
|new_email|String|新邮箱地址, 留空为删除|
|client_ip_addr|String|修改邮箱的用户IP|

自动触发的Action执行步骤:   

1. 更改原用户邮箱地址为新邮箱地址, 设置用户状态为邮箱未验证
2. 生成一个新的VeriCode用以验证新的邮箱地址

##### 5.3.2.20003 修改手机验证

类型: `JSON Object`   
数据定义:   

|键位|数据类型|解释|
|-|-|-|
|new_phone|String|新手机, 留空为删除|
|client_ip_addr|String|修改手机的用户IP|

自动触发的Action执行步骤:   

1. 更改原用户手机为新手机号, 设置用户状态为手机未验证

**注意**: 不会生成一个新的VeriCode用以验证新的手机号, 因为需要用户选择方式(SMS/电话)

##### 5.3.2.30001 管理员操作验证
类型: `JSON Object`   
数据定义:   

|键位|数据类型|解释|
|-|-|-|
|client_ip_addr|String|操作的管理员用户IP|

##### 5.3.2.90001 第三方APP重要操作验证
类型: `JSON Object`   
数据定义:   

|键位|数据类型|解释|
|-|-|-|
|client_ip_addr|String|修改APP的用户IP|

##### 5.3.2.90002 第三方APP删除内容验证
类型: `JSON Object`   
数据定义:   

|键位|数据类型|解释|
|-|-|-|
|client_ip_addr|String|删除内容的用户IP|

## 6.0 邮箱/短信信息发送

发送方式在核心支持库中只定义接口, 实现将在外部Wrapper中实现发送类.   
邮箱: 自建SMTP / 在线接口如[Mailgun](https://www.mailgun.com/)   
SMS: 在线接口如[短信通](http://www.dxton.com/jiekou.html)   

### 6.1 邮箱信息发送

每封邮件的正文都用会输出为标准HTML5语言, 即每个邮件正文都会以以下代码开始:   

```html
<!DOCTYPE html>
<html>
`CONTENT`
</html>
```

核心库会使用PHP字符串替换, 模版中使用变量需要用{{ variableName }}或{{variableName}}来表示.   
邮件的模版文件会放在核心库的`/templates/email/` + `LOCALE_NAME` 目录中, 比如`/templates/email/zh_CN/`.      
手机的模版文件会放在核心库的`/templates/SMS/` + `LOCALE_NAME` 目录中, 不包括标题文件   
电话的模板文件(TTS)会放在核心库的`/templates/PhoneCall_TTS/` + `LOCALE_NAME` 目录中, 不包括标题文件   

#### 6.1.1 邮件/手机模版

##### 6.1.1.10001 邮箱验证模版
发送方式: EMAIL   
验证码 action_id: 10001   
文件名: verification_10001.tpl   
标题文件名: verification_10001.title   
变量列表:   

|变量名|解释|注释|在哪些发送方式中可见|
|-|-|-|-|
|systemName|用户系统名称|-|ANY_METHOD|
|username|用户名|-|ANY_METHOD|
|userDisplayName|用户展示名|-|ANY_METHOD|
|userEmail|用户邮箱|-|ANY_METHOD|
|veriLink|验证地址|-|ANY_METHOD|

##### 6.1.1.10002 手机验证模板
发送方式: SMS \| PHONE_CALL   
验证码 action_id: 10002   
文件名: verification_10002.tpl   
标题文件名: verification_10002.title   

变量列表: 

|变量名|解释|注释|在哪些发送方式中可见|
|-|-|-|-|
|systemName|用户系统名称|-|ANY_METHOD|
|username|用户名|-|ANY_METHOD|
|userDisplayName|用户展示名|-|ANY_METHOD|
|userPhone|用户手机|-|ANY_METHOD|
|veriCode|验证码|-|ANY_METHOD|
|veriLink|验证地址|-|SMS|

##### 6.1.1.20001 密码修改申请验证模版
发送方式: EMAIL \| SMS \| PHONE_CALL   
验证码 action_id: 20001   
文件名: verification_20001.tpl   
标题文件名: verification_20001.title   
变量列表:   

|变量名|解释|注释|在哪种发送方式中可见|
|-|-|-|-|
|systemName|用户系统名称|-|ANY_METHOD|
|username|用户名|-|ANY_METHOD|
|userDisplayName|用户展示名|-|ANY_METHOD|
|veriCode|验证码|-|ANY_METHOD|
|veriLink|可以前往修改密码的URL|-|EMAIL \| SMS|

##### 6.1.1.20002 修改邮箱验证
发送方式: EMAIL \| SMS \| PHONE_CALL   
验证码 action_id: 20002   
文件名: verification_20002.tpl   
标题文件名: verification_20002.title   
变量列表:   

|变量名|解释|注释|在哪种发送方式中可见|
|-|-|-|-|
|systemName|用户系统名称|-|ANY_METHOD|
|username|用户名|-|ANY_METHOD|
|userDisplayName|用户展示名|-|ANY_METHOD|
|userEmail|用户邮箱|-|ANY_METHOD|
|veriCode|验证码|-|ANY_METHOD|
|veriLink|可以前往确认邮箱修改的URL|-|EMAIL \| SMS|
|newEmail|申请更改到的新邮箱|-|ANY_METHOD|

##### 6.1.1.20003 修改手机验证
发送方式: EMAIL \| SMS \| PHONE_CALL   
验证码 action_id: 20003   
文件名: verification_20003.tpl   
标题文件名: verification_20003.title   
变量列表:   

|变量名|解释|注释|在哪种发送方式中可见|
|-|-|-|-|
|systemName|用户系统名称|-|ANY_METHOD|
|username|用户名|-|ANY_METHOD|
|userDisplayName|用户展示名|-|ANY_METHOD|
|userPhone|用户手机号码|-|ANY_METHOD|
|veriCode|验证码|-|ANY_METHOD|
|veriLink|可以前往确认手机号码修改的URL|-|EMAIL \| SMS|
|newPhone|申请更改到的新手机号码|-|ANY_METHOD|

##### 6.1.1.30001 管理员操作验证
发送方式: EMAIL \| SMS \| PHONE_CALL   
验证码 action_id: 30001   
文件名: verification_30001.tpl   
标题文件名: verification_30001.title   
变量列表:   

|变量名|解释|注释|在哪种发送方式中可见|
|-|-|-|-|
|systemName|用户系统名称|-|ANY_METHOD|
|username|用户名|-|ANY_METHOD|
|userDisplayName|用户展示名|-|ANY_METHOD|
|veriCode|验证码|-|ANY_METHOD|

##### 6.1.1.90001 第三方APP重要操作验证
发送方式: EMAIL \| SMS \| PHONE_CALL   
验证码 action_id: 90001   
文件名: verification_90001.tpl   
标题文件名: verification_90001.title   
变量列表:   

|变量名|解释|注释|在哪种发送方式中可见|
|-|-|-|-|
|systemName|APP系统名称|-|ANY_METHOD|
|username|用户名|-|ANY_METHOD|
|userDisplayName|用户展示名|-|ANY_METHOD|
|veriCode|验证码|-|ANY_METHOD|

##### 6.1.1.90002 第三方APP删除内容验证
发送方式: EMAIL \| SMS \| PHONE_CALL   
验证码 action_id: 90002   
文件名: verification_90002.tpl   
标题文件名: verification_90002.title   
变量列表:   

|变量名|解释|注释|在哪种发送方式中可见|
|-|-|-|-|
|systemName|APP系统名称|-|ANY_METHOD|
|username|用户名|-|ANY_METHOD|
|userDisplayName|用户展示名|-|ANY_METHOD|
|veriCode|验证码|-|ANY_METHOD|

## 7.0 PDK Core 错误代码表

InteractivePDK后端实现中, 核心支持库扔出的异常都会是`PDKException`类的实例. PDK Exception支持声明 `Error_Code(错误代码)`, `Message(详细信息)`, `Params(错误参数)`, 和 `Previous(上一个堆栈中的错误)` 4个参数. 

|Error_Code|名称|解释|批注|参数(Params)|
|-|-|-|-|-|
|10001|User non-existant|用户不存在|-|-|
|10002|User frozen|用户被注销|-|-|
|10003|User not verified|用户未验证(邮箱或手机)|-|-|
|10004|User already exist|用户已存在|-|-|
|10005|User email already exist|用户邮箱已存在|-|-|
|10006|User phone already exist|用户手机已存在|-|-|
|10007|User display name already exist|用户昵称已存在|-|-|
|20001|APP non-existant|第三方APP不存在|-|-|
|20002|APP frozen|第三方APP被注销|-|-|
|20003|APP error|第三方APP内部错误|-|-|
|20004|APP client_id already exist|第三方APP client_id已被占用|-|-|
|20005|APP display name already exist|-|-|
|20006|APP Management Relations already exist|第三方APP的管理关系表已经建立|-|-|
|20007|APP Management Relations non-existant|第三方APP的管理关系表没有建立|-|-|
|30001|Credentials not correct|验证凭据不正确|-|-|
|30002|Credentials not formatted|验证凭据与规定格式不符|-|credential="`database_col_name`"|
|30003|Permission Denied|无权限|-|-|
|30004|IP Address not match|操作时IP不一致|-|-|
|40001|Spam Message|垃圾信息|-|-|
|40002|Operation Too Frequent|操作过于频繁|-|-|
|50001|System Busy|系统繁忙|-|-|
|50002|Email Service Unavailable|邮件系统繁忙|-|-|
|50003|Email Service Auth Failure|邮件系统登录失败|-|-|
|50004|SMS Service Unavailable|短信系统繁忙|-|-|
|50005|SMS Service Auth Failure|短信系统登录失败|-|-|
|50006|General Message Sending Error|发送失败|-|-|
|50006|DB Conn Error|数据库连接错误|-|-|
|50007|DB Query Error|数据库提交/获取数据错误|-|errNo,errMsg|
|50008|DB Timeout Error|数据库超时错误|-|-|
|51000|Inner Error|内部错误|-|-|
|60001|Usergroup non-existant|用户组不存在|-|-|
|60002|Usergroup already exist|用户组已存在|-|-|
|60003|Usergroup display name already exist|用户展示名已存在|-|-|
|60004|Parentgroup non-existant|父组不存在|-|-|
|70001|Token expired|登录凭据已超时|-|-|
|70002|Token not found|登录凭据不存在|-|-|
|70003|Token already exist|登录凭据ID已存在|-|-|
|70004|Refresh Token expired|刷新凭据已超时|-|-|
|70005|Refresh Token not found|刷新凭据不存在|-|-|
|70006|Refresh Token already exist|刷新凭据已存在|-|-|
|80001|Verification Code expired|验证码已超时|-|-|
|80002|Verification Code not found|验证码不存在|-|-|
|80003|Verification Code already exist|验证码已存在|-|-|
|80004|Cannot send in this method|当前验证码类型无法以此方式发送|-|sent_method, action_id|
|80005|Verification Code Action Execution Error|当前验证码的自动触发机制产生错误|-|errMsg|
|90001|OAuth Authorization Code already exist|Authorization Code已存在|-|-|
|90002|OAuth Authorization Code non-existant|Authorization Code不存在|-|-|
|90003|OAuth Access Token already exist|Access Token已存在|-|-|
|90004|OAuth Access Token non-existant|Access Token不存在|-|-|
|90005|OAuth Refresh Token already exist|Refresh Token已存在|-|-|
|90006|OAuth Refresh Token non-existant|Refresh Token不存在|-|-|
|90100|OAuth Credential Format Incorrect|验证OAuth凭据格式不符|-|credential="`database_col_name`"|
|100001|Avatar Hash non-existant|头像哈希不存在|-|-|

## 8.0 系统设置中的变量数据格式定义
### 8.1 多语言变量格式(MultiLang)
每个多语言变量都是一个多键位JSON Object(PHP中用array), 每一个键位代表一种语言的内容.    
键位用标准LOCALE_NAME来表示, 如`zh_CN`, `en_US`.   
e.g.  

```PHP
class Setting{
    const USER_SYSTEM_NAME = array(
        'zh_CN' => '幽径',
        'en_US' => 'Solitary Trail'
    );
}
```

### 8.2 用户系统回调链接格式
变量名: USER_SYSTEM_LINKS   
格式: 多语言变量, 每个LOCALE_NAME中都是一个JSON Object, 键位如下:   

|URL键位|数据类型|解释|例子|模板变量|
|-|-|-|-|-|
|confirm_email_url|string|确认邮箱的时候点击链接的URL模版|https://user.interactiveplus.org/zh_CN/confirmEmail/?veri_code={{veri_code}}|veri_code|
|confirm_phone_url|string|确认手机的时候点击链接的URL模板|https://user.interactiveplus.org/zh_CN/confirmPhone/?veri_code={{veri_code}}|veri_code|
|change_pwd_url|string|修改密码时点击链接可以来这里修改密码|https://user.interactiveplus.org/zh_CN/changePassword?veri_code={{veri_code}}|veri_code|
|confirm_email_change_url|string|确认更改邮箱的时候点击链接的URL模版|https://user.interactiveplus.org/zh_CN/ConfirmEmailChange/?veri_code={{veri_code}}|veri_code|
|confirm_phone_change_url|string|确认更改手机的时候点击链接的URL模版|https://user.interactiveplus.org/zh_CN/ConfirmPhoneChange/?veri_code={{veri_code}}|veri_code|

## 9.0 OAuth2.0和第三方APP相关定义

### 9.1 总览

OAuth2.0 在 [InterActivePDK2020-CoreLib](https://github.com/InteractivePlus/InteractivePDK2020-CoreLib) 中将使用 [league/oauth2-server](https://github.com/thephpleague/oauth2-server/), 因为关于OAuth2的Doc实在是太多了, 我们团队没有任何想法自己实现一个.   

参考:   
1. [OAuth2.0 API接口设计](https://developer.aliyun.com/article/44243)
2. [OAuth.com](https://www.oauth.com)
3. [OAuth.net](https://oauth.net/)

### 9.2 APP类型定义

APP根据来源分类: 

|类型|解释|
|-|-|
|1stParty|第一方APP, 既形随意动官方APP|
|trusted-3rdParty|被信任的第三方APP|
|3rdParty|普通第三方APP|

APP根据用途分类:

|类型|解释|
|-|-|
|public|公共APP, 既APP没有后端|
|private|私人APP, APP拥有后端服务器|

注: 不同用途会导致授权方式

### 9.2 授权方式(Grant Type)定义

|授权方式(response_type)|授权方式详细信息|可用来源|可用类型|注释|
|-|-|-|-|-|
|Authorization Code(code)|Client申请Authorization Code后申请Access Token|ALL|private|使用PKCE密钥交换, 需要提交client_id和client_secret|
|Explicit Authorization Code(code)|Client申请Authorization Code后申请Access Token|ALL|public|使用PKCE密钥交换, 只需要提交client_id|
|Refresh Token(grant_type=refresh_token)|Client申请通过之前获取的Refresh Token来获取新的Access Token和Refresh Token对|ALL|ALL|如果类型为private, 则需要提供client_secret|

### 9.3 授权范围定义

|scope identifier|范围详细信息|可用grant_type|注释|
|-|-|-|-|
|basic|用户基础信息, 包括用户名, 头像, 昵称, 用户签名, 语言和地区数据, 不包括联系方式|ALL|-|
|push_notif|APP可以给用户发送推送信息(APP, 微信公众号)|ALL|-|
|email_notif|APP可以给用户推送邮箱信息, 但不能获取用户具体联系方式|ALL|-|
|sms_notif|APP可以给用户发送手机短信, 但不能获取用户具体联系方式|ALL|-|
|email|用户邮箱地址|ALL|-|
|phone_num|用户手机号|ALL|-|

### 9.4 APP管理权限

在`app_manage_infos`表中, 每个APP管理员都有相对应的role(角色), 对应不同的管理权限.

|role|详细权限|int value|
|-|-|-|
|owner|删除APP及所有admin的权限|5|
|admin|设置管理员列表以及所有write权限|4|
|write|设置APP头像, 昵称, 回调地址以及maintain权限|3|
|maintain|管理工单, 以及read权限|2|
|read|查看APP头像, 查看日志, 查看工单, 查看APP信息|1|

注: 一个APP只能拥有一个owner, 其他角色人数不限.

## 10.0 Log中ActionID的定义

|ActionID|操作解释|context变量|
|-|-|-|
|10001|登录操作|method=0/1/2, uid, \[token\], BrowserUA|
|10002|注册操作|username, \[email\], \[phoneNum\], uid, displayName, BrowserUA|
|10003|验证Token操作|tokenID, uid|
|10004|刷新Token操作|usedRefreshToken, \[newTokenID\], \[newRefreshToken\]|
|10005|删除用户操作|uid, \[phoneNum\], \[email\], \[displayName\], BrowserUA|
|10006|更改用户信息操作|uid|
|10007|更改设置操作|uid|
|10008|验证手机操作|uid|
|10009|验证邮箱操作|uid|
|10010|更改手机操作|uid|
|10011|更改邮箱操作|uid|
|10012|申请验证码操作|uid, actionID, action_param, send_method, \[veri_code\]|
|10013|使用验证码操作|uid, veri_code|
|20001|创建APP操作|uid, \[appuid\], displayName, \[clientID\], \[clientSecret\], \[clientType\], regArea, BrowserUA|
|20002|修改APP信息操作|uid, appuid|
|20003|更改APP管理列表操作|uid, appuid|
|20004|转让APP操作|uid, appuid, newOwnerUID|
|20003|删除APP操作|uid, appuid|
|80001|申请Captcha验证码操作|\[uid\],\[appuid\], \[captchaPhrase\], BrowserUA|
|90001|PSR Logger Log|不限|