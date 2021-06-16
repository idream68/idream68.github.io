---
title: shiro + jwt 基础
date: 2021-06-15 20:20:14
categories:
  - spring
  - springboot
  - shiro
tags:
  - springboot
  - shiro
  - jwt
---

### jwt 简介

> JSON Web Token (JWT)是一个开放标准(RFC 7519)，它定义了一种紧凑的、自包含的方式，用于作为JSON对象在各方之间安全地传输信息。该信息可以被验证和信任，因为它是数字签名的。

JSON Web Token由三部分组成，它们之间用圆点(.)连接。这三部分分别是：

- Header
- Payload
- Signature

因此，一个典型的JWT看起来是这个样子的：

> xxxxx.yyyyy.zzzzz

### shiro中集成JWT

1. 定义 JWT filter

   在shiro配置中定义哪些 url 使用 JWT 验证，jwt根据自定义条件判断是否可以进行访问

   ```java
   @Slf4j
   public class JwtFilter extends BasicHttpAuthenticationFilter {
       /**
        * 前置处理
        */
       @Override
       protected boolean preHandle(ServletRequest request, ServletResponse response) throws Exception {
           HttpServletRequest httpServletRequest = WebUtils.toHttp(request);
           HttpServletResponse httpServletResponse = WebUtils.toHttp(response);
           // 跨域时会首先发送一个option请求，这里我们给option请求直接返回正常状态
           if (httpServletRequest.getMethod().equals(RequestMethod.OPTIONS.name())) {
               httpServletResponse.setStatus(HttpStatus.OK.value());
               return false;
           }
           return super.preHandle(request, response);
       }
   
       /**
        * 后置处理
        */
       @Override
       protected void postHandle(ServletRequest request, ServletResponse response) {
           // 添加跨域支持
           this.fillCorsHeader(WebUtils.toHttp(request), WebUtils.toHttp(response));
       }
   
       /**
        * 过滤器拦截请求的入口方法
        * 返回 true 则允许访问
        * 返回false 则禁止访问，会进入 onAccessDenied()
        */
       @Override
       protected boolean isAccessAllowed(ServletRequest request, ServletResponse response, Object mappedValue) {
           // 原用来判断是否是登录请求，在本例中不会拦截登录请求，用来检测Header中是否包含 JWT token 字段
           if (this.isLoginRequest(request, response))
               return false;
           boolean allowed = false;
           try {
               // 检测Header里的 JWT token内容是否正确，尝试使用 token进行登录
               allowed = executeLogin(request, response);
           } catch (IllegalStateException e) { // not found any token
               log.error("Not found any token");
           } catch (Exception e) {
               log.error("Error occurs when login", e);
           }
           return allowed || super.isPermissive(mappedValue);
       }
   
       /**
        * 检测Header中是否包含 JWT token 字段
        */
       @Override
       protected boolean isLoginAttempt(ServletRequest request, ServletResponse response) {
           return ((HttpServletRequest) request).getHeader(JwtUtils.AUTH_HEADER) == null;
       }
   
       /**
        * 身份验证,检查 JWT token 是否合法
        */
       @Override
       protected boolean executeLogin(ServletRequest request, ServletResponse response) throws Exception {
           AuthenticationToken token = createToken(request, response);
           if (token == null) {
               String msg = "createToken method implementation returned null. A valid non-null AuthenticationToken "
                       + "must be created in order to execute a login attempt.";
               throw new IllegalStateException(msg);
           }
           try {
               Subject subject = getSubject(request, response);
               subject.login(token); // 交给 Shiro 去进行登录验证
               return onLoginSuccess(token, subject, request, response);
           } catch (AuthenticationException e) {
               return onLoginFailure(token, e, request, response);
           }
       }
   
       /**
        * 从 Header 里提取 JWT token
        */
       @Override
       protected AuthenticationToken createToken(ServletRequest servletRequest, ServletResponse servletResponse) {
           HttpServletRequest httpServletRequest = (HttpServletRequest) servletRequest;
           String authorization = httpServletRequest.getHeader(JwtUtils.AUTH_HEADER);
           JwtToken token = new JwtToken(authorization);
           return token;
       }
   
       /**
        * isAccessAllowed()方法返回false，会进入该方法，表示拒绝访问
        */
       @Override
       protected boolean onAccessDenied(ServletRequest servletRequest, ServletResponse servletResponse) throws Exception {
           HttpServletResponse httpResponse = WebUtils.toHttp(servletResponse);
           httpResponse.setCharacterEncoding("UTF-8");
           httpResponse.setContentType("application/json;charset=UTF-8");
           httpResponse.setStatus(HttpStatus.UNAUTHORIZED.value());
           PrintWriter writer = httpResponse.getWriter();
           writer.write("{\"errCode\": 401, \"msg\": \"UNAUTHORIZED\"}");
           fillCorsHeader(WebUtils.toHttp(servletRequest), httpResponse);
           return false;
       }
   
       /**
        * Shiro 利用 JWT token 登录成功，会进入该方法
        */
       @Override
       protected boolean onLoginSuccess(AuthenticationToken token, Subject subject, ServletRequest request,
                                        ServletResponse response) throws Exception {
           HttpServletResponse httpResponse = WebUtils.toHttp(response);
           String newToken = null;
           if (token instanceof JwtToken) {
               newToken = JwtUtils.refreshTokenExpired(token.getCredentials().toString(), JwtUtils.SECRET);
           }
           if (newToken != null)
               httpResponse.setHeader(JwtUtils.AUTH_HEADER, newToken);
           return true;
       }
   
       /**
        * Shiro 利用 JWT token 登录失败，会进入该方法
        */
       @Override
       protected boolean onLoginFailure(AuthenticationToken token, AuthenticationException e, ServletRequest request,
                                        ServletResponse response) {
           // 此处直接返回 false ，交给后面的  onAccessDenied()方法进行处理
           return false;
       }
   
       /**
        * 添加跨域支持
        */
       protected void fillCorsHeader(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse) {
           httpServletResponse.setHeader("Access-control-Allow-Origin", httpServletRequest.getHeader("Origin"));
           httpServletResponse.setHeader("Access-Control-Allow-Methods", "GET,POST,OPTIONS,HEAD");
           httpServletResponse.setHeader("Access-Control-Allow-Headers",
                   httpServletRequest.getHeader("Access-Control-Request-Headers"));
       }
   }
   ```

   

2. 定义 JWT token

   ```java
   public class JwtToken implements AuthenticationToken {
       private static final long serialVersionUID = 1L;
   
       // 加密后的 JWT token串
       private String token;
   
       private String userName;
   
       public JwtToken(String token) {
           this.token = token;
           this.userName = JwtUtils.getClaimFiled(token, "username");
       }
   
       @Override
       public Object getPrincipal() {
           return this.userName;
       }
   
       @Override
       public Object getCredentials() {
           return token;
       }
   }
   ```

   

3. 定义 JWT Realm

   定义认证方法和授权方法

   ```java
   public class JwtRealm  extends AuthorizingRealm {
       /**
        * 限定这个 Realm 只处理我们自定义的 JwtToken
        */
       @Override
       public boolean supports(AuthenticationToken token) {
           return token instanceof JwtToken;
       }
   
       /**
        * 此处的 SimpleAuthenticationInfo 可返回任意值，密码校验时不会用到它
        */
       @Override
       protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authcToken)
               throws AuthenticationException {
           JwtToken jwtToken = (JwtToken) authcToken;
           if (jwtToken.getPrincipal() == null) {
               throw new AccountException("JWT token参数异常！");
           }
           // 从 JwtToken 中获取当前用户
           String username = jwtToken.getPrincipal().toString();
           // 查询数据库获取用户信息，此处使用 Map 来模拟数据库
           UserEntity user = ShiroRealm.userMap.get(username);
   
           // 用户不存在
           if (user == null) {
               throw new UnknownAccountException("用户不存在！");
           }
   
           // 用户被锁定
           if (user.getLocked()) {
               throw new LockedAccountException("该用户已被锁定,暂时无法登录！");
           }
   
           SimpleAuthenticationInfo info = new SimpleAuthenticationInfo(user, username, getName());
           return info;
       }
   
       @Override
       protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
           // 获取当前用户
           UserEntity currentUser = (UserEntity) SecurityUtils.getSubject().getPrincipal();
           // UserEntity currentUser = (UserEntity) principals.getPrimaryPrincipal();
           // 查询数据库，获取用户的角色信息
           Set<String> roles = ShiroRealm.roleMap.get(currentUser.getName());
           // 查询数据库，获取用户的权限信息
           Set<String> perms = ShiroRealm.permMap.get(currentUser.getName());
           SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
           info.setRoles(roles);
           info.setStringPermissions(perms);
           return info;
       }
   }
   
   ```

   

4. 定义 JWT creadentialsMatcher

   验证 token 是否合法

   ```java
   @Slf4j
   public class JwtCredentialsMatcher  implements CredentialsMatcher {
       /**
        * JwtCredentialsMatcher只需验证JwtToken内容是否合法
        */
       @Override
       public boolean doCredentialsMatch(AuthenticationToken authenticationToken, AuthenticationInfo authenticationInfo) {
   
           String token = authenticationToken.getCredentials().toString();
           String username = authenticationToken.getPrincipal().toString();
           try {
               Algorithm algorithm = Algorithm.HMAC256(JwtUtils.SECRET);
               JWTVerifier verifier = JWT.require(algorithm).withClaim("username", username).build();
               verifier.verify(token);
               return true;
           } catch (JWTVerificationException e) {
               log.error(e.getMessage());
           }
           return false;
       }
   
   }
   ```

   

5. 将 JWT 配置到 shiro 中

   ```java
   @Configuration
   public class ShiroConfig {
   
   
       /**
        * 交由 Spring 来自动地管理 Shiro-Bean 的生命周期
        */
       @Bean
       public static LifecycleBeanPostProcessor getLifecycleBeanPostProcessor() {
           return new LifecycleBeanPostProcessor();
       }
   
       /**
        * 为 Spring-Bean 开启对 Shiro 注解的支持
        */
       @Bean
       public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor(SecurityManager securityManager) {
           AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor = new AuthorizationAttributeSourceAdvisor();
           authorizationAttributeSourceAdvisor.setSecurityManager(securityManager);
           return authorizationAttributeSourceAdvisor;
       }
   
       @Bean
       public DefaultAdvisorAutoProxyCreator defaultAdvisorAutoProxyCreator() {
           DefaultAdvisorAutoProxyCreator app = new DefaultAdvisorAutoProxyCreator();
           app.setProxyTargetClass(true);
           return app;
   
       }
   
       /**
        * 不向 Spring容器中注册 JwtFilter Bean，防止 Spring 将 JwtFilter 注册为全局过滤器
        * 全局过滤器会对所有请求进行拦截，而本例中只需要拦截除 /login 和 /logout 外的请求
        * 另一种简单做法是：直接去掉 jwtFilter()上的 @Bean 注解
        */
       @Bean
       public FilterRegistrationBean<Filter> registration(JwtFilter filter) {
           FilterRegistrationBean<Filter> registration = new FilterRegistrationBean<Filter>(filter);
           registration.setEnabled(false);
           return registration;
       }
   
       @Bean
       public JwtFilter jwtFilter() {
           return new JwtFilter();
       }
   
       /**
        * 配置访问资源需要的权限
        */
       @Bean
       ShiroFilterFactoryBean shiroFilterFactoryBean(SecurityManager securityManager) {
           ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
           shiroFilterFactoryBean.setSecurityManager(securityManager);
           shiroFilterFactoryBean.setLoginUrl("/login");
           shiroFilterFactoryBean.setSuccessUrl("/authorized");
           shiroFilterFactoryBean.setUnauthorizedUrl("/unauthorized");
   
           // 添加 jwt 专用过滤器，拦截除 /login 和 /logout 外的请求
           Map<String, Filter> filterMap = new LinkedHashMap<>();
           filterMap.put("jwtFilter", jwtFilter());
           shiroFilterFactoryBean.setFilters(filterMap);
   
           LinkedHashMap<String, String> filterChainDefinitionMap = new LinkedHashMap<String, String>();
           filterChainDefinitionMap.put("/login", "anon"); // 可匿名访问
   //        filterChainDefinitionMap.put("/logout", "logout"); // 退出登录
           filterChainDefinitionMap.put("/**", "jwtFilter,authc"); // 需登录才能访问
           shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);
           return shiroFilterFactoryBean;
       }
   
       /**
        * 配置 ModularRealmAuthenticator
        */
       @Bean
       public ModularRealmAuthenticator authenticator() {
           ModularRealmAuthenticator authenticator = new MultiRealmAuthenticator();
           // 设置多 Realm的认证策略，默认 AtLeastOneSuccessfulStrategy
           AuthenticationStrategy strategy = new FirstSuccessfulStrategy();
           authenticator.setAuthenticationStrategy(strategy);
           return authenticator;
       }
   
       /**
        * 禁用session, 不保存用户登录状态。保证每次请求都重新认证
        */
       @Bean
       protected SessionStorageEvaluator sessionStorageEvaluator() {
           DefaultSessionStorageEvaluator sessionStorageEvaluator = new DefaultSessionStorageEvaluator();
           sessionStorageEvaluator.setSessionStorageEnabled(false);
           return sessionStorageEvaluator;
       }
   
       /**
        * JwtRealm 配置，需实现 Realm 接口
        */
       @Bean
       JwtRealm jwtRealm() {
           JwtRealm jwtRealm = new JwtRealm();
           // 设置加密算法
           CredentialsMatcher credentialsMatcher = new JwtCredentialsMatcher();
           // 设置加密次数
           jwtRealm.setCredentialsMatcher(credentialsMatcher);
           return jwtRealm;
       }
   
       /**
        * ShiroRealm 配置，需实现 Realm 接口
        */
       @Bean
       ShiroRealm shiroRealm() {
           ShiroRealm shiroRealm = new ShiroRealm();
           // 设置加密算法
           HashedCredentialsMatcher credentialsMatcher = new HashedCredentialsMatcher("SHA-1");
           // 设置加密次数
           credentialsMatcher.setHashIterations(16);
           shiroRealm.setCredentialsMatcher(credentialsMatcher);
           return shiroRealm;
       }
   
       /**
        * 配置 SecurityManager
        */
       @Bean
       public SecurityManager securityManager() {
           DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
   
           // 1.Authenticator
           securityManager.setAuthenticator(authenticator());
   
           // 2.Realm
           List<Realm> realms = new ArrayList<Realm>(16);
           realms.add(jwtRealm());
           realms.add(shiroRealm());
           securityManager.setRealms(realms);
   
           // 3.关闭shiro自带的session
           DefaultSubjectDAO subjectDAO = new DefaultSubjectDAO();
           subjectDAO.setSessionStorageEvaluator(sessionStorageEvaluator());
           securityManager.setSubjectDAO(subjectDAO);
   
           return securityManager;
       }
   }
   
   ```

### JWT 注意事项

1. JWT token 一次签发，永久有效，退出token依然有效，失效只能等到超过有效期，使用subject.logout 只会执行删除session之类的操作，并没有进行token的销毁，使用token依然可以正常访问

   解决思路

   1. 退出时将退出的token保存到redis中，作为黑名单
   2. 在token中保存生成时间，并将生成时间保存到redis中，两个生成时间相同则表示登录，否则表示退出

   都破坏了无状态，相当于在redis中保存了sessionId

[完整代码示例](https://github.com/idream68/spring-demo/tree/master/shiro_jwt)
