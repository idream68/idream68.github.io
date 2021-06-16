---
title: shiro 自定义加密方法
date: 2021-06-15 20:21:42
categories:
  - spring
  - springboot
  - shiro
tags:
  - springboot
  - shiro
  - encryption
---

1. 自定义 filter

   根据自定义条件筛选访问是否被允许，允许访问则进行后续的授权或者认证流程，不允许则直接返回相应。

   ```java
   @Slf4j
   public class CustomFilter extends BasicHttpAuthenticationFilter {
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
           return ((HttpServletRequest) request).getHeader("Authorization") == null;
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
           String authorization = httpServletRequest.getHeader("Authorization");
           CustomToken token = new CustomToken(authorization);
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
           if (token instanceof CustomToken) {
               newToken = ((CustomToken) token).getToken();
           }
           if (newToken != null)
               httpResponse.setHeader("Authorization", newToken);
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
   ```

   

2. 自定义 realm

   自定义认证和授权方法，认证方法处理用户是否存在和用户状态，判断是否可以登录；授权方法结构化用户所属的用户组和所拥有的权限。每次授权流程操作之前都会有认证流程， 认证不通过，则直接返回，不会进行授权流程。认证密码（token）状态需要自定义 credentialsMatcher 流程进行判断。

   ```java
   public class CustomRealm extends AuthorizingRealm {
   
       /**
        * 过滤自定义token类型
        * @param token
        * @return
        */
       @Override
       public boolean supports(AuthenticationToken token) {
           return token instanceof CustomToken;
       }
   
       /**
        * 授权，权限和用户组都为 {xx,yy}；根据具体情况修改，此处仅为示例
        * @param principals
        * @return
        */
       @Override
       protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
           SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
           Set<String> roles = new HashSet<>();
           roles.add("xx");
           roles.add("yy");
           info.setRoles(roles);
           info.setStringPermissions(roles);
           return info;
       }
   
       /**
        * 认证，只有token是ss时才能通过认证，进行下一步验证，根据具体情况修改，此处仅为示例
        * @param token
        * @return
        * @throws AuthenticationException
        */
       @Override
       protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
           String t = token.getPrincipal().toString();
           if (t.equals("ss")) {
               return new SimpleAuthenticationInfo(t, t, getName());
           } else {
               throw new AuthenticationException("xxx");
           }
       }
   ```

   

3. 自定义 credentialsMatcher

   判断用户密码、token等是否和所需的一致（经过自定以加密之后）。
   
   ```java
   public class CustomCredentialsMatcher implements CredentialsMatcher {
   
       /**
        * 自定义认证方法（密码，token等自定义验证方法），登录，和其他方法都需要验证；根据具体情况修改，此处仅为示例，全部通过验证
        * @param token
        * @param info
        * @return true 验证成功，false 验证失败
        */
       @Override
       public boolean doCredentialsMatch(AuthenticationToken token, AuthenticationInfo info) {
           return true;
       }
   }
   ```
   
   
   
4. 将自定义的filter,realm,creadentialsMatcher配置到shiro配置中

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
        * 配置访问资源需要的权限
        */
       @Bean
       ShiroFilterFactoryBean shiroFilterFactoryBean(SecurityManager securityManager) {
           ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
           shiroFilterFactoryBean.setSecurityManager(securityManager);
           shiroFilterFactoryBean.setLoginUrl("/login");
           shiroFilterFactoryBean.setSuccessUrl("/authorized");
           shiroFilterFactoryBean.setUnauthorizedUrl("/unauthorized");
   
           Map<String, Filter> filterMap = new LinkedHashMap<>();
           CustomFilter customFilter = new CustomFilter();
           filterMap.put("custom", customFilter);
           shiroFilterFactoryBean.setFilters(filterMap);
   
           LinkedHashMap<String, String> filterChainDefinitionMap = new LinkedHashMap<String, String>();
           filterChainDefinitionMap.put("/login", "custom"); // 自定义
   //        filterChainDefinitionMap.put("/logout", "logout"); // 退出登录
           filterChainDefinitionMap.put("/**", "custom"); // 自定义
           shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);
           return shiroFilterFactoryBean;
       }
   
   
       @Bean
       CustomRealm customRealm() {
           CustomRealm customRealm = new CustomRealm();
           CustomCredentialsMatcher customCredentialsMatcher = new CustomCredentialsMatcher();
           customRealm.setCredentialsMatcher(customCredentialsMatcher);
           return customRealm;
       }
   
   
       /**
        * 配置 SecurityManager
        */
       @Bean
       public SecurityManager securityManager() {
           DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
           List<Realm> realms = new ArrayList<>(4);
           realms.add(customRealm());
           securityManager.setRealms(realms);
           return securityManager;
       }
   ```

   

[完整示例代码](https://github.com/idream68/spring-demo/tree/master/shiro_custom_encryption)

