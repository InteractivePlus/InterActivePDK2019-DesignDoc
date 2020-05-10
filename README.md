# InterActivePDK2019-DesignDoc

## 这个项目是什么? | What is this project?

[形随意动](http://www.interactiveplus.org/) InterActivePDK2020最新官方文档, 用于标准化以及提前声明开发过程中使用到的前后端架构和代码标准 / 通信协议   
This is an official standardization doc for InterActivePDK 2020 developed by [InterActivePlus](http://www.interactiveplus.org/). This doc is used to standardize the code structure, standard and communication protocols between frontends and backends.

## 目录与进度 | Directory and Progress

- [ ] [项目策划书 | Project Plan](ProjectPlan/)
- [ ] 编码规范 | Coding Standards
    - [ ] 前端编码规范 | Frontend Coding Standards
        - [ ] JS编码规范 | Javascript Coding Standards
        - [ ] HTML编码规范 | HTML Coding Standards
    - [ ] 后端编码规范 | Backend Coding Standards
        - [ ] PHP编码规范 | PHP Coding Standards
        - [ ] 后端代码提交审核规范 | Backend Code Submit & Review Standard
- [ ] [后端架构 | Backend Coding Structures](CodingStructures/Backend/)
    - [ ] 数据库结构和储存算法 | Database Structure And Information Storage Algorithms
    - [ ] 核心库架构 | Core Library Structure
    - [ ] 外部API包装架构 | API Wrapper Structure
    - [ ] (可选) 插件系统架构 | (Optional) Plugin System Structure
- [ ] 前端架构 | Frontend Coding Structures
    - [ ] 核心库架构 | Core Library Structure
    - [ ] 外部JS文件分布 | Outer Javascript File Distribution

## 系统综述 | System Overview

形随意动PDK全称形随意动公用开发套件, 专门用来处理对形随意动用户的登录请求和第三方应用授权请求.   
形随意动PDK前端登录命名为BlueAirLive(新开发版本为2.1), 后端命名为OPENAPI(新开发版本为4.1)   
InterActivePDK's full name is InterActive+ Public Development Kit, is designed specifically for dealing with the login of our user systems and auth request from 3rd party apps.   
InterActivePDK's frontend login page is called BlueAirLive(New version is 2.1), backend is called OPENAPI(New version is 4.1)   

---

形随意动PDK是由四方组成的用户授权系统:

1. 用户
2. 形随意动PDK
3. 第三方APP
4. 第三方登录方

其中用户可以用自己的`PDK`帐号(形随意动账户)登录`PDK`服务, 或用`第三方登录`的帐号登录到`PDK`. 在登录到`PDK`后用户可以享受到`第三方APP`和`形随意动官方`提供的服务. 同时用户可以调整自己在`PDK`中的个人喜好, 控制`第三方应用`和`形随意动官方`可以获得的权限(包括发送邮件, 获取个人信息等), 或直接取消授权某一个应用.   

---

InterActivePDK is consist of four groups:

1. User
2. InterActivePDK
3. Third-Party APP
4. Third-Party Authentication Authority

Here users can use their own`PDK` Account(InterActive+ Account) to login into `PDK` services, or use `Third Party Auth` Account to login into `PDK`. After logging into `PDK`, users can enjoy services provided by `Third Party Apps` and `InterActivePlus`. At the same time, users can change their preferences inside `PDK`, allowing or forbiddening `Third Party Apps` or `Interactive Services` to gain certain rights(including sending emails, getting user's personal information, etc.), or just cancel their certificate to a certain app.


## 项目目的 | Motivations

---

1. 由于形随意动提供的旗下服务包括了多个有用户登录功能的网站, 统一模块让所有旗下服务的用户系统达成一致对提升用户体验非常重要.
2. 为了避免重复代码以及减少工作量, 形随意动旗下新产品在开发时一旦有需要储存用户信息 / 云同步服务的, 使用此项目

---

1. Our services include a lot of websites that have user login systems. Making them unified under one module is extremely important for us to elevate the user experience.
2. In order to avoid repetitive code and reduce workload, whenever we make a new service that needs login / cloud synchronization service, we will use this project.

---
