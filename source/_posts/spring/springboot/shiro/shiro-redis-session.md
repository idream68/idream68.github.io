---
title: shiro + redis 基础
date: 2021-06-15 20:20:28
categories:
  - spring
  - springboot
  - shiro
tags:
  - springboot
  - redis
  - shiro
---

将 session 保存到 redis 中

1. 导入依赖（gradle版）

```
implementation group: 'org.crazycake', name: 'shiro-redis', version: '3.3.1'
```

使用shiro-redis作为将session保存到redis的插件

2. 修改配置文件，将session管理设置成redis

   ```java
   @Configuration
   public class ShiroConfig {
       @Autowired(required = false)
       private ResourcesService resourcesService;
   
       @Value("${spring.redis.host}")
       private String host;
   
       @Value("${spring.redis.timeout}")
       private int timeout;
   
       @Value("${spring.redis.password}")
       private String password;
   
       @Bean
       public static LifecycleBeanPostProcessor getLifecycleBeanPostProcessor() {
           return new LifecycleBeanPostProcessor();
       }
   
       /**
        * ShiroFilterFactoryBean 处理拦截资源文件问题。
        * 注意：单独一个ShiroFilterFactoryBean配置是或报错的，因为在
        * 初始化ShiroFilterFactoryBean的时候需要注入：SecurityManager
        *
        Filter Chain定义说明
        1、一个URL可以配置多个Filter，使用逗号分隔
        2、当设置多个过滤器时，全部验证通过，才视为通过
        3、部分过滤器可指定参数，如perms，roles
        *
        */
       @Bean
       public ShiroFilterFactoryBean shirFilter(SecurityManager securityManager){
           System.out.println("ShiroConfiguration.shirFilter()");
           ShiroFilterFactoryBean shiroFilterFactoryBean  = new ShiroFilterFactoryBean();
   
           // 必须设置 SecurityManager
           shiroFilterFactoryBean.setSecurityManager(securityManager);
           // 如果不设置默认会自动寻找Web工程根目录下的"/login.jsp"页面
           shiroFilterFactoryBean.setLoginUrl("/login");
           // 登录成功后要跳转的链接
           shiroFilterFactoryBean.setSuccessUrl("/users");
           //未授权界面;
           shiroFilterFactoryBean.setUnauthorizedUrl("/403");
           //拦截器.
           Map<String,String> filterChainDefinitionMap = new LinkedHashMap<String,String>();
   
           //配置退出 过滤器,其中的具体的退出代码Shiro已经替我们实现了
           filterChainDefinitionMap.put("/logout", "logout");
           filterChainDefinitionMap.put("/css/**","anon");
           filterChainDefinitionMap.put("/js/**","anon");
           filterChainDefinitionMap.put("/img/**","anon");
           filterChainDefinitionMap.put("/font-awesome/**","anon");
           //<!-- 过滤链定义，从上向下顺序执行，一般将 /**放在最为下边 -->:这是一个坑呢，一不小心代码就不好使了;
           //<!-- authc:所有url都必须认证通过才可以访问; anon:所有url都都可以匿名访问-->
           //自定义加载权限资源关系
           List<Resources> resourcesList = resourcesService.list();
           for(Resources resources:resourcesList){
   
               if (StringUtils.hasText(resources.getResUrl())) {
                   String permission = "perms[" + resources.getResUrl()+ "]";
                   filterChainDefinitionMap.put(resources.getResUrl(),permission);
               }
           }
           filterChainDefinitionMap.put("/**", "authc");
   
   
           shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);
           return shiroFilterFactoryBean;
       }
   
   
       @Bean
       public SecurityManager securityManager(){
           DefaultWebSecurityManager securityManager =  new DefaultWebSecurityManager();
           //设置realm.
           securityManager.setRealm(myShiroRealm());
           // 自定义缓存实现 使用redis
           securityManager.setCacheManager(cacheManager());
           // 自定义session管理 使用redis
           securityManager.setSessionManager(sessionManager());
           return securityManager;
       }
   
       @Bean
       public MyShiroRealm myShiroRealm(){
           MyShiroRealm myShiroRealm = new MyShiroRealm();
           myShiroRealm.setCredentialsMatcher(hashedCredentialsMatcher());
           return myShiroRealm;
       }
   
       /**
        * 凭证匹配器
        * （由于我们的密码校验交给Shiro的SimpleAuthenticationInfo进行处理了
        *  所以我们需要修改下doGetAuthenticationInfo中的代码;
        * ）
        * @return
        */
       @Bean
       public HashedCredentialsMatcher hashedCredentialsMatcher(){
           HashedCredentialsMatcher hashedCredentialsMatcher = new HashedCredentialsMatcher();
   
           hashedCredentialsMatcher.setHashAlgorithmName("SHA-1");
           hashedCredentialsMatcher.setHashIterations(16);
   
           return hashedCredentialsMatcher;
       }
   
   
       /**
        *  开启shiro aop注解支持.
        *  使用代理方式;所以需要开启代码支持;
        * @param securityManager
        * @return
        */
       @Bean
       public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor(SecurityManager securityManager){
           AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor = new AuthorizationAttributeSourceAdvisor();
           authorizationAttributeSourceAdvisor.setSecurityManager(securityManager);
           return authorizationAttributeSourceAdvisor;
       }
   
       /**
        * 配置shiro redisManager
        * 使用的是shiro-redis开源插件
        * @return
        */
       public RedisManager redisManager() {
           RedisManager redisManager = new RedisManager();
           redisManager.setHost(host);
           redisManager.setTimeout(timeout);
           if (StringUtils.hasText(password)){
               redisManager.setPassword(password);
           }
           return redisManager;
       }
   
       /**
        * cacheManager 缓存 redis实现
        * 使用的是shiro-redis开源插件
        * @return
        */
       public RedisCacheManager cacheManager() {
           RedisCacheManager redisCacheManager = new RedisCacheManager();
           redisCacheManager.setRedisManager(redisManager());
           return redisCacheManager;
       }
   
   
       /**
        * RedisSessionDAO shiro sessionDao层的实现 通过redis
        * 使用的是shiro-redis开源插件
        */
       @Bean
       public RedisSessionDAO redisSessionDAO() {
           RedisSessionDAO redisSessionDAO = new RedisSessionDAO();
           redisSessionDAO.setRedisManager(redisManager());
           return redisSessionDAO;
       }
   
       /**
        * shiro session的管理
        */
       @Bean
       public DefaultWebSessionManager sessionManager() {
           DefaultWebSessionManager sessionManager = new DefaultWebSessionManager();
           sessionManager.setSessionDAO(redisSessionDAO());
           return sessionManager;
       }
   
   ```

   [完整代码示例](https://github.com/idream68/spring-demo/tree/master/shiro_redis)

