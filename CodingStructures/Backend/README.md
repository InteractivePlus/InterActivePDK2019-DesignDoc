# 后端架构
此文档将详细规范OPENAPI4.1 (InterActivePDK 2019 Backend) 开发中的架构设计, 数据库结构设计和代码交互逻辑.

---

最新版本: 20200422-0.0.1 B

## 基本概念
OPENAPI4.1 底层架构致力于简化开发过程中繁琐的代码重合和交互逻辑, 我们希望将每个功能区块完整的区分出来并明确他们之间的界限. 同时底层的交互逻辑应减少与数据库连接的次数, 减少服务器总体压力, 提升性能表现.   
OPENAPI4.1 架构方面将分成两块, 核心功能层和API功能层.   

### 核心功能层模块介绍

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

## 加密方法定义

## 在设置中可更改的部分变量注解

|变量名|解释|
|-|-|
|USERNAME_MINLEN|用户名/用户组名最小长度|
|USERNAME_MAXLEN|用户名/用户组名最大长度|
|DISPLAYNAME_MINLEN|展示名最小长度|
|DISPLAYNAME_MAXLEN|展示名最大长度|
|SIGNATURE_MAXLEN|用户签名最大长度|
|PASSWORD_MINLEN|密码最小长度|
|PASSWORD_MAXLEN|密码最大长度|
|LOGIN_SINGLEIPMAXTRIAL_COUNT|单个IP在固定时间里最多尝试错误登录几次|
|LOGIN_SINGLEIPMAXTRIAL_DURATION|上面变量的固定时间长度(秒)|
|AVATOR_MAX_SIZE|用户头像最大大小(kB)|
|PASSWORD_SALT|密码加密Salt|
|TOKEN_SALT|TOKEN加密Salt|
|TOKEN_AVAILABLE_DURATION|TOKEN有效时间(秒)|


## 数据库结构定义

**数据表名称和数据字段名需要使用小写(参照MySQL手册)**
**数据库名称建议使用大驼峰命名法**


### 数据表结构定义

|表名|表用途|
|-|-|
|user_infos|用户信息储存表|
|usergroup_infos|用户组信息储存表|
|logged_infos|用户已登录信息储存表(主站Token储存表)|
|app_infos|第三方APP信息储存表|
|logs|日志信息储存表|
|avatars|用户头像储存表|

### 表内字段定义

#### user_infos表

|字段名|数据类型|解释|算法|注释|
|-|-|-|-|-|
|username|VARCHAR(`USERNAME_MAXLEN`)|用户名|-|-|
|display_name|VARCHAR(`DISPLAYNAME_MAXLEN`)|用户展示名|-|-|
|signature|VARCHAR(`SIGNATURE_MAXLEN`)|用户签名|-|-|
|password|CHAR(64)|密码哈希|sha256(original,`PASSWORD_SALT`)|-|
|email|VARCHAR(50)|邮箱|-|-|
|phone_number|VARCHAR(15)|用户绑定手机号|-|前三位放国家区号(不足3位前面放0), 后12位放手机号|
|settings|TEXT|用户设置|gzcompress(original json object)|-|
|thirdauth|TEXT|第三方登录绑定信息|gzcompress(original json object)|-|
|email_verified|TINYINT(1)|邮箱是否验证过|-|1(true) / 0(false)|
|phone_verified|TINYINT(1)|手机是否被验证过|-|1(true) / 0(false)|
|permission_override|TEXT|用户权限(覆盖组权限的内容)|gzcompress(original json object)|可留空|
|group|VARCHAR(`USERNAME_MAX`)|用户组|original|-|
|regtime|INT|用户注册时间|time()|-|
|relatedapps|TEXT|用户可以管理的APPID|gzcompress(original json array)|["appid1","appid2"], 可留空|
|is_admin|TINYINT(1)|是否是管理员用户|-|1(true) / 0(false)|
|avatar|CHAR(32)|用户头像md5|md5(头像数据)|如果是默认头像则留空|

#### usergroup_infos表

|字段名|数据类型|解释|算法|注释|
|-|-|-|-|-|
|groupid|VARCHAR(`USERNAME_MAXLEN`)|组id|-|-|
|parent-group-id|VARCHAR(`USERNAME_MAXLEN`)|父组id|-|-|
|display_name|VARCHAR(`DISPLAYNAME_MAXLEN`)|组展示名|-|-|
|description|VARCHAR(`SIGNATURE_MAXLEN`)|组详细信息|-|-|
|regtime|INT|组注册时间|time()|-|
|permissions|TEXT|组基础权限|original json object|-|
|avatar|CHAR(32)|组头像md5|md5(头像数据)|如果是默认头像则留空|

#### logged_infos表

|字段名|数据类型|解释|算法|注释|
|-|-|-|-|-|
|token|CHAR(32)|用户分配到的TOKEN|md5(username + rand(0, 10000), `TOKEN_SALT`)|-|
|issue_time|INT|token分配时间|time()|-|
|expire_time|INT|token过期时间|time() + `TOKEN_AVAILABLE_DURATION`|-|
|client_addr|VARCHAR(40)|用户客户端IP地址|original|ipv4/ipv6|

