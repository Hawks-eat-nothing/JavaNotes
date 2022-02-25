# Shiro
[TOC]

## 第一章 权限概述
### 1.权限

- 访问权限
**表示用户的角色能做哪些操作或访问哪些资源。**

- 数据权限
**表示某些数据是否属于用户或用户可以对数据的操作范围。**

### 2. 认证

身份认证，就是判断一个用户是否为合法用户的处理过程。最常用的简单身份认证方式是系统通过核对用户输入的用户名和密码，看其是否与系统中存储的该用户的用户名和密码一致，来判断用户身份是否正确。

**认证流程**

<img src="E:\Desktop\Coding\Notes\img\1580044671870.png" alt="1580044671870" style="zoom: 80%;" />

**关键对象**

**Subject（主体）:** 访问系统的用户，主体可以是用户、程序等，进行认证的都称为主体

**Principal（身份）:**主体进行身份认证的标识，标识必须具有唯一性。一个主体可以有多个身份，但是必须有一个主身份（Primary Principal）。

**credential（凭证）:**只有主体自己知道的安全信息，如密码、证书等。

### 3. 授权

**授权**

即访问控制，控制谁能访问哪些资源。主体进行身份认证后，系统会为其分配对应的权限，当访问资源时，会校验其是否有访问此资源的权限。

**四个对象：**
1. 用户User:当前操作的用户、程序
2. 资源Resource:当前被访问的对象
3. 角色Role:一组“权限操作许可权”的集合
4. 权限Permission:权限操作许可权

**授权流程**

<img src="E:\Desktop\Coding\Notes\img\1582789303655.png" alt="1582789303655" style="zoom:80%;" />

**关键对象**

**授权可简单理解为who对what进行How操作**

**Who:**主体，可以是一个用户、也可以是一个程序

**What:**资源

**How:**权限/许可

## 第二章 Shiro 概述

### 1. Shiro简介

它将软件系统的安全认证相关的功能抽取出来，==实现用户身份认证，权限授权、加密、会话管理等功能，==组成了一个通用的安全认证框架。

**特点**

- 易于理解的 Java Security API；
- 简单的身份认证（登录），支持多种数据源（LDAP，JDBC 等）；
- 对角色的简单的签权（访问控制），也支持细粒度的鉴权；
- 支持一级缓存，以提升应用程序的性能；
- 内置的基于 POJO 企业会话管理，适用于 Web 以及非 Web 的环境；
- 异构客户端会话访问；
- 非常简单的加密 API；
- 不跟任何的框架或者容器捆绑，可以独立运行。

### 2. 核心组件

**Shiro框架图**

![](E:\Desktop\Coding\Notes\img\9b959a65-799d-396e-b5f5-b4fcfe88f53c.png)

- Subject
Subject将用户作为当前操作的主体，这个主体可以是一个通过浏览器请求的用户，也可能是一个运行的程序。Subject在shiro中是一个接口，接口中定义了很多认证授相关的方法，**外部程序通过Subject进行认证授权，而Subject是通过SecurityManager安全管理器进行认证授权。**

- SecurityManager
Shiro的核心，**负责对所有的subject进行安全管理。通过SecurityManager可以完成subject的认证、授权等。SecurityManager是通过Authenticator进行认证，通过Authorizer进行授权，通过SessionManager进行会话管理等。**SecurityManager是一个接口，继承了Authenticator, Authorizer, SessionManager这三个接口

- Authenticator
Authenticator即认证器，**对用户登录时进行身份认证**

- Authorizer
Authorizer授权器，用户通过认证器认证通过，**在访问功能时需要通过授权器判断用户是否有此功能的操作权限。**

- Realm（数据库读取+认证功能+授权功能实现）
**SecurityManager进行安全认证需要通过Realm获取用户权限数据。**

- SessionManager
Shiro框架定义了一套会话管理，它不依赖web容器的session，所以Shiro可以使用在非web应用上，也可以将分布式应用的会话集中在一点管理，此特性可使它实现单点登录。

- SessionDAO
对session会话操作的一套接口

- CacheManager
CacheManager缓存管理，**将用户权限数据存储在缓存，这样可以提高性能.**

- Cryptography
Cryptography密码管理，Shiro提供了一套加密/解密的组件，方便开发。比如提供常用的散列、加/解密等功能

## 第三章 Shiro入门
### 1. 身份认证

**基本流程**

![1580047247677](.\img\1580047247677.png)

1. Shiro把用户的数据封装成标识token，token一般封装着用户名，密码等信息
2. 使用Subject门面获取到封装着用户的数据的标识token
3. Subject把标识token交给SecurityManager，在SecurityManager安全中心中，SecurityManager把标识token委托给认证器Authenticator进行身份验证。==认证器的作用一般是用来指定如何验证，它规定本次认证用到哪些Realm。==
4. 认证器Authenticator将传入的标识token，与数据源Realm对比，验证token是否合法

**实现步骤**

1. 权限定义：ini文件
2. 加载过程：
	1. 导入权限ini文件构建权限工厂
	2. 使用工厂构建安全管理器SecurityManager
	3. 使用SecurityUtils工具生效安全管理器 
	4. 使用SecurityUtils工具获得subject主体
	5. 构建账户密码，打包token
	6. 使用subject主体登录。

### 2. Realm

<img src="E:\Desktop\Coding\Notes\img\1580050317405.png" alt="1580050317405" style="zoom:80%;" />

一般在真实的项目中，我们不会直接实现Realm接口，我们一般的情况就是直接继承AuthorizingRealm，能够继承到认证与授权功能。它需要强制重写两个方法：

`doGetAuthenticationInfo()`

`doGetAuthorizationInfo()`

