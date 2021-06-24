---
title: shiro 基础
date: 2021-06-15 20:10:55
categories:
  - spring
  - springboot
  - shiro
tags:
  - springboot
  - shiro
---

三个核心组件：Subject, SecurityManager 和 Realms.

Subject：即“当前操作用户”。但是，在Shiro中，Subject这一概念并不仅仅指人，也可以是第三方进程、后台帐户（Daemon Account）或其他类似事物。它仅仅意味着“当前跟软件交互的东西”。但考虑到大多数目的和用途，你可以把它认为是Shiro的“用户”概念。 Subject代表了当前用户的安全操作，SecurityManager则管理所有用户的安全操作。

SecurityManager：它是Shiro框架的核心，典型的Facade模式，Shiro通过SecurityManager来管理内部组件实例，并通过它来提供安全管理的各种服务。

Realm： Realm充当了Shiro与应用安全数据间的“桥梁”或者“连接器”。也就是说，当对用户执行认证（登录）和授权（访问控制）验证时，Shiro会从应用配置的Realm中查找用户及其权限信息。 从这个意义上讲，Realm实质上是一个安全相关的DAO：它封装了数据源的连接细节，并在需要时将相关数据提供给Shiro。当配置Shiro时，你必须至少指定一个Realm，用于认证和（或）授权。配置多个Realm是可以的，但是至少需要一个。 Shiro内置了可以连接大量安全数据源（又名目录）的Realm，如LDAP、关系数据库（JDBC）、类似INI的文本配置资源以及属性文件等。如果缺省的Realm不能满足需求，你还可以插入代表自定义数据源的自己的Realm实现。

细分功能

1. Authentication：身份认证/登录(账号密码验证)。
2. Authorization：授权，即角色或者权限验证。
3. Session Manager：会话管理，用户登录后的session相关管理。
4. Cryptography：加密，密码加密等。
5. Web Support：Web支持，集成Web环境。
6. Caching：缓存，用户信息、角色、权限等缓存到如redis等缓存中。
7. Concurrency：多线程并发验证，在一个线程中开启另一个线程，可以把权限自动传播过去。
8. Testing：测试支持；
9. Run As：允许一个用户假装为另一个用户（如果他们允许）的身份进行访问。
10. Remember Me：记住我，登录后，下次再来的话不用登录了。

[以下引用官网介绍](https://shiro.apache.org/architecture.html)

> The following diagram shows Shiro’s core architectural concepts followed by short summaries of each:
>
> ![images](/images/shiro/ShiroArchitecture.png)
>
> - Subject (org.apache.shiro.subject.Subject)
>
> A security-specific ‘view’ of the entity (user, 3rd-party service, cron job, etc) currently interacting with the software.
>
> - SecurityManager (org.apache.shiro.mgt.SecurityManager)
>
> As mentioned above, the SecurityManager is the heart of Shiro’s architecture. It is mostly an ‘umbrella’ object that coordinates its managed components to ensure they work smoothly together. It also manages Shiro’s view of every application user, so it knows how to perform security operations per user.
>
> - Authenticator (org.apache.shiro.authc.Authenticator)
>
> The Authenticator is the component that is responsible for executing and reacting to authentication (log-in) attempts by users. When a user tries to log-in, that logic is executed by the Authenticator. The Authenticator knows how to coordinate with one or more Realms that store relevant user/account information. The data obtained from these Realms is used to verify the user’s identity to guarantee the user really is who they say they are.
>
> - Authentication Strategy (org.apache.shiro.authc.pam.AuthenticationStrategy)
>
> If more than one Realm is configured, the AuthenticationStrategy will coordinate the Realms to determine the conditions under which an authentication attempt succeeds or fails (for example, if one realm succeeds but others fail, is the attempt successful? Must all realms succeed? Only the first?).
>
> - Authorizer (org.apache.shiro.authz.Authorizer)
>
> The Authorizer is the component responsible determining users’ access control in the application. It is the mechanism that ultimately says if a user is allowed to do something or not. Like the Authenticator, the Authorizer also knows how to coordinate with multiple back-end data sources to access role and permission information. The Authorizer uses this information to determine exactly if a user is allowed to perform a given action.
>
> - SessionManager (org.apache.shiro.session.mgt.SessionManager)
>
> The SessionManager knows how to create and manage user Session lifecycles to provide a robust Session experience for users in all environments. This is a unique feature in the world of security frameworks - Shiro has the ability to natively manage user Sessions in any environment, even if there is no Web/Servlet or EJB container available. By default, Shiro will use an existing session mechanism if available, (e.g. Servlet Container), but if there isn’t one, such as in a standalone application or non-web environment, it will use its built-in enterprise session management to offer the same programming experience. The SessionDAO exists to allow any datasource to be used to persist sessions.
>
> - SessionDAO (org.apache.shiro.session.mgt.eis.SessionDAO)
>
> The SessionDAO performs Session persistence (CRUD) operations on behalf of the SessionManager. This allows any data store to be plugged in to the Session Management infrastructure.
>
> - CacheManager (org.apache.shiro.cache.CacheManager) The CacheManager creates and manages Cache instance lifecycles used by other Shiro components. Because Shiro can access many back-end data sources for authentication, authorization and session management, caching has always been a first-class architectural feature in the framework to improve performance while using these data sources. Any of the modern open-source and/or enterprise caching products can be plugged in to Shiro to provide a fast and efficient user-experience.
> - Cryptography (org.apache.shiro.crypto.*)
>
> Cryptography is a natural addition to an enterprise security framework. Shiro’s crypto package contains easy-to-use and understand representations of crytographic Ciphers, Hashes (aka digests) and different codec implementations. All of the classes in this package are carefully designed to be very easy to use and easy to understand. Anyone who has used Java’s native cryptography support knows it can be a challenging animal to tame. Shiro’s crypto APIs simplify the complicated Java mechanisms and make cryptography easy to use for normal mortal human beings.
>
> - Realms (org.apache.shiro.realm.Realm)
>
> As mentioned above, Realms act as the ‘bridge’ or ‘connector’ between Shiro and your application’s security data. When it comes time to actually interact with security-related data like user accounts to perform authentication (login) and authorization (access control), Shiro looks up many of these things from one or more Realms configured for an application. You can configure as many Realms as you need (usually one per data source) and Shiro will coordinate with them as necessary for both authentication and authorization.



过滤器说明

| 过滤器 | 描述 |
| :--: | :--: |
| anon | 表示可以匿名使用 |
| authc | 表示需要认证(登录)才能使用 |
| authcBasic | 表示httpBasic认证 |
| perms | 当有多个参数时必须每个参数都通过才通过 perms[“user:add:”] |
| port | port[8081] 跳转到schemal://serverName:8081?queryString |
| rest | 权限 |
| roles | 角色 |
| ssl | 表示安全的url请求 |
| user | 表示必须存在用户，当登入操作时不做检查 |
| ... | 自定义 |


[代码示例](https://github.com/idream68/spring-demo/tree/master/shiro_base)

