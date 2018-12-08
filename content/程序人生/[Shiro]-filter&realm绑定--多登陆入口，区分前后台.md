---
title : "[Shiro]-filter&realm绑定--多登陆入口，区分前后台"
---
# Shiro框架 filter&realm绑定
## 这种实现方式有问题，会影响到后续的角色验证。不建议看了
## 新的实现方式 ： [自定义Token](http://www.jianshu.com/p/986a3c41dac1)
------------------------------
- **shiro filter与Realm绑定**
- **使用Spring整合Shiro**
- **多登陆入口，成功后跳转到不同地址**
- **重写Realm获取方式**
------------------------------
>  最近在项目中使用了apache的轻量级Shiro框架进行权限管理，使用了多个filter，Shiro会默认迭代Realm进行登陆，**即使第一个Realm通过校验，也会继续迭代，造成性能的浪费，和数据库查询**。又或者**前Realm里做了一些校验，抛出了异常，也会被后面的无用的Realm校验失败抛出的一场覆盖。**

------------

  不废话，上教程不废话，上教程
 注意：这里filter统一指shiro 定义的filter
  至于servlet的filter会写成 servlet filter

#### 这是先前的配置
已经实现方式赋予Realm对应的role角色，已经有多入口，前后台分开的功能，
```xml
<bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
        <!-- shiro 的核心安全接口 -->
        <property name="securityManager" ref="securityManager" />
        <!-- 未授权时要跳转的连接 -->
        <property name="unauthorizedUrl" value="/unauthorized.jsp" />
        <property name="filters">
            <map>
                <entry key="admin" value-ref="adminformAuthenticationFilter" />
                <entry key="authc" value-ref="formAuthenticationFilter" />
            </map>
        </property>
        <!-- shiro 连接约束配置 -->
        <property name="filterChainDefinitions">
            <value>
                /user.html = authc
                /admin.html =authc
                /login.html = authc
                /admin/login.html =admin
                /logout = logout
                /resource/** = anon
            </value>
        </property>
    </bean>

    <bean id="formAuthenticationFilter"
        class="org.apache.shiro.web.filter.authc.FormAuthenticationFilter">
        <property name="loginUrl" value="/login.html" />
        <property name="successUrl" value="/ok.html" />
    </bean>
    <bean id="adminformAuthenticationFilter"
        class="org.apache.shiro.web.filter.authc.FormAuthenticationFilter">
        <property name="loginUrl" value="/admin/login.html" />
        <property name="successUrl" value="/admin/ok.html" />
    </bean>
    <bean id="AdminRealm" class="cc.yihy.realm.AdminRealm">
        <property name="userService" ref="UserService" />
    </bean>
    <bean id="UserRealm" class="cc.yihy.realm.UserRealm">
        <property name="userService" ref="UserService" />
    </bean>

    <!-- 安全管理器 -->
    <bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
    <!-- 默认的Realm验证 -->
        <property name="realms">
            <list>
                <ref bean="UserRealm" />
                <ref bean="AdminRealm"></ref>
            </list>
        </property>
    </bean>
    <bean id="lifecycleBeanPostProcessor" class="org.apache.shiro.spring.LifecycleBeanPostProcessor" />
```
--------------------------
####在我们点击登陆后，Shiro会使用我们配置的FormAuthenticationFilter进行验证。
执行
```java
//父类里实现的方法
protected boolean executeLogin(ServletRequest request, ServletResponse response) throws Exception {
    //这里创建用户的令牌
        AuthenticationToken token = createToken(request, response);
        if (token == null) {
            String msg = "createToken method implementation returned null. A valid non-null AuthenticationToken " +
                    "must be created in order to execute a login attempt.";
            throw new IllegalStateException(msg);
        }
        try {
            Subject subject = getSubject(request, response);
            //在这里进行登陆验证
            subject.login(token);
            return onLoginSuccess(token, subject, request, response);
        } catch (AuthenticationException e) {
            return onLoginFailure(token, e, request, response);
        }
    }

      protected AuthenticationToken createToken(ServletRequest request, ServletResponse response) {
        String username = getUsername(request);
        String password = getPassword(request);
        return createToken(username, password, request, response);
    }
    //父类的创建令牌的实现
     protected AuthenticationToken createToken(String username, String password,
                                              ServletRequest request, ServletResponse response) {
        boolean rememberMe = isRememberMe(request);
        String host = getHost(request);
        return createToken(username, password, rememberMe, host);
    }
    //父类的创建令牌的实现
   protected AuthenticationToken createToken(String username, String password,
                                              boolean rememberMe, String host) {
        return new UsernamePasswordToken(username, password, rememberMe, host);
    }
```
subject.login(token);的实现类里，又调用了安全管理器的login。传的对象是一个Token令牌。
####再看看安全管理器里面是怎么进行登录的
 Subject subject = securityManager.login(this, token);
 
```java
 //安全管理器实现类DefaultSecurityManager里面的login
public Subject login(Subject subject, AuthenticationToken token) throws AuthenticationException {
      //
        AuthenticationInfo info;
        try {
            info = authenticate(token);
        } catch (AuthenticationException ae) {
            try {
                onFailedLogin(token, ae, subject);
            } catch (Exception e) {
                if (log.isInfoEnabled()) {
                    log.info("onFailedLogin method threw an " +
                            "exception.  Logging and propagating original AuthenticationException.", e);
                }
            }
            throw ae; //propagate
        }

        Subject loggedIn = createSubject(token, info, subject);

        onSuccessfulLogin(token, info, loggedIn);

        return loggedIn;
    }
  //这是它的父类里面 
   private Authenticator authenticator;

   public AuthenticationInfo authenticate(AuthenticationToken token) throws AuthenticationException {
       //真正进行登陆Realm的在这里
        return this.authenticator.authenticate(token);
    }
```
#### 还是看下Authenticator接口实现类（ModularRealmAuthenticator）的源码

```java
 这是配置的Realm集合
private Collection<Realm> realms;
//迭代Realm进行验证
   protected AuthenticationInfo doAuthenticate(AuthenticationToken authenticationToken) throws AuthenticationException {
        assertRealmsConfigured();
        Collection<Realm> realms = getRealms();
        if (realms.size() == 1) {
            return doSingleRealmAuthentication(realms.iterator().next(), authenticationToken);
        } else {
            return doMultiRealmAuthentication(realms, authenticationToken);
        }
    }
```
*到这里，filter调用realm进行验证的流程基本已经清晰了。*

filter创建一个Token对象，传到安全管理器（securityManager），让安全管理器调用Authenticator属性进行迭代Realm。
怎么让Filter找到对应权限的realm呢？他们中间传递一一个对象Token令牌。
Token里放了这些基本信息
**new UsernamePasswordToken(username, password, rememberMe, host)；**

从filter到realm并没有角色名的传递。
filter的name属性里放的是角色名。这点在Spring创建ShiroFilter 可以看到。


```java
//getFilters()拿到的是xml配置上的filters
        Map<String, Filter> filters = getFilters();
        if (!CollectionUtils.isEmpty(filters)) {
            for (Map.Entry<String, Filter> entry : filters.entrySet()) {
                String name = entry.getKey();
                Filter filter = entry.getValue();
                applyGlobalPropertiesIfNecessary(filter);
                if (filter instanceof Nameable) {
                    ((Nameable) filter).setName(name);
                }
                //'init' argument is false, since Spring-configured filters should be initialized
                //in Spring (i.e. 'init-method=blah') or implement InitializingBean:
                manager.addFilter(name, filter, false);
            }
        }
```
现在想要把filter的信息传给realm就需要在他们之间传递的对象动手脚了。让Token把Filter对象带过去就行了。
那那边怎么处理呢。

 #### 实例安全管理器的时候，会实例一个ModularRealmAuthorizer对象，并可以设置为我们自定义的。

```java
 private Authorizer authorizer;
//安全管理器实现类的父类，在实例安全管理器的时候，会创建一个ModularRealmAuthorizer对象，迭代Realm验证，就是ModularRealmAuthorizer做的工作
    /**
     * Default no-arg constructor that initializes an internal default
     * {@link org.apache.shiro.authz.ModularRealmAuthorizer ModularRealmAuthorizer}.
     */
    public AuthorizingSecurityManager() {
        super();
        this.authorizer = new ModularRealmAuthorizer();
    }

   public void setAuthorizer(Authorizer authorizer) {
        if (authorizer == null) {
            String msg = "Authorizer argument cannot be null.";
            throw new IllegalArgumentException(msg);
        }
        this.authorizer = authorizer;
    }
```
到这里，Filter&Realm绑定需要修改的地方基本都知道了。
**重写FormAuthenticationFilter的createToken(String username, String password,boolean rememberMe, String host)方法**，**自定义一个Token，可以存放Filter**。**重写ModularRealmAuthorizer对象中获取Realm的方法。**

下面是实现代码  

####  NewFormAuthenticationFilter 实现

```java
public class NewFormAuthenticationFilter extends FormAuthenticationFilter {
    /**
     * 创建自定义的令牌，加入当前filter
     */
    @Override
    protected AuthenticationToken createToken(String username, String password,
            boolean rememberMe, String host) {
        return new UsernamePasswordAndFilterToken(username, password,
                rememberMe, host, this);
    }

    /**
     * 获取当前Filter的名字（角色名）扩大访问范围
     */
    @Override
    public String getName() {
        return super.getName();
    }

}
```

#### UsernamePasswordAndFilterToken 实现

```java
public class UsernamePasswordAndFilterToken extends UsernamePasswordToken {

    /**
     * 
     */
    private static final long serialVersionUID = 1L;
    /**
     * 存放当前Filter
     */
    private NameableFilter loginFilter;

    public UsernamePasswordAndFilterToken() {
        super();
    }

    public UsernamePasswordAndFilterToken(final String username,
            final char[] password, final boolean rememberMe, final String host,
            final AdviceFilter loginFilter) {
        super(username, password, rememberMe, host);
        this.loginFilter = loginFilter;
    }

    public UsernamePasswordAndFilterToken(final String username,
            final String password, final boolean rememberMe, final String host,
            final AdviceFilter loginFilter) {
        super(username, password, rememberMe, host);
        this.loginFilter = loginFilter;
    }

    public NameableFilter getLoginFilter() {
        return loginFilter;
    }

    public void setLoginFilter(NameableFilter loginFilter) {
        this.loginFilter = loginFilter;
    }
}
```

####DefineModularRealmAuthenticator 实现

```java
public class DefineModularRealmAuthenticator extends ModularRealmAuthenticator {

    private static final Logger log = LoggerFactory
            .getLogger(DefineModularRealmAuthenticator.class);
    
    private Map<String, Realm> defineRealms;

    /**
     * 判断Realm是不是null
     */
    @Override
    protected void assertRealmsConfigured() throws IllegalStateException {
        defineRealms = getDefineRealms();
        if (CollectionUtils.isEmpty(defineRealms)) {
            String msg = "Configuration error:  No realms have been configured!  One or more realms must be "
                    + "present to execute an authentication attempt.";
            throw new IllegalStateException(msg);
        }
    }
    /**
     * 根据filter的name 取对应realm进行登录
     */
    @Override
    protected AuthenticationInfo doAuthenticate(
            AuthenticationToken authenticationToken)
            throws AuthenticationException {
        assertRealmsConfigured();
    
        /**
         * authenticationToken 如果是自定义的UsernamePasswordAndFilterToken，调用单个realm
         * 否则 使用默认的迭代realm方式
         */
        if(authenticationToken instanceof UsernamePasswordAndFilterToken){
                NewFormAuthenticationFilter loginFilter = (NewFormAuthenticationFilter)((UsernamePasswordAndFilterToken) authenticationToken).getLoginFilter();

                Realm realm = defineRealms.get(loginFilter.getName());
                if(realm==null){
                    log.error("没有配置NewFormAuthenticationFilter对应的Realm");
                    throw new RuntimeException("没有配置NewFormAuthenticationFilter对应的Realm");
                };
                return doSingleRealmAuthentication(realm,authenticationToken);
        }else {
            return oldDoAuthenticate(authenticationToken);
        }
            

    }

    private  AuthenticationInfo oldDoAuthenticate(AuthenticationToken authenticationToken)throws AuthenticationException{
        
        Collection<Realm> realms = defineRealms.values();
        if (realms.size() == 1) {
            return doSingleRealmAuthentication(realms.iterator().next(), authenticationToken);
        } else {
            return doMultiRealmAuthentication(realms, authenticationToken);
        }
    }
    
    public void setDefineRealms(Map<String, Realm> defineRealms) {
        this.defineRealms = defineRealms;
    }

    public Map<String, Realm> getDefineRealms() {
        return defineRealms;
    }
}
```

####在xml中的配置修改为

```xml
<bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
        <!-- shiro 的核心安全接口 -->
        <property name="securityManager" ref="securityManager" />
        <!-- 未授权时要跳转的连接 -->
        <property name="unauthorizedUrl" value="/unauthorized.jsp" />
        <property name="filters">
            <map>
                <entry key="admin" value-ref="adminformAuthenticationFilter" />
                <entry key="authc" value-ref="formAuthenticationFilter" />
            </map>
        </property>
        <!-- shiro 连接约束配置 -->
        <property name="filterChainDefinitions">
            <value>
                /user.html = authc
                /admin.html =authc
                /login.html = authc
                /admin/login.html =admin
                /logout = logout
                /resource/** = anon
            </value>
        </property>
    </bean>
    <!-- filter&realm绑定 -->
    <bean id="DefineModularRealmAuthenticator"
        class="cc.yihy.shiro.authc.pam.DefineModularRealmAuthenticator">
        <property name="defineRealms">
            <map>
                <entry key="authc" value-ref="UserRealm" />
                <entry key="admin" value-ref="AdminRealm" />
            </map>
        </property>
    </bean>
    <bean id="formAuthenticationFilter"
        class="cc.yihy.shiro.filter.NewFormAuthenticationFilter">
        <property name="loginUrl" value="/login.html" />
        <property name="successUrl" value="/ok.html" />
    </bean>
    <bean id="adminformAuthenticationFilter"
        class="cc.yihy.shiro.filter.NewFormAuthenticationFilter">
        <property name="loginUrl" value="/admin/login.html" />
        <property name="successUrl" value="/admin/ok.html" />
    </bean>
    <bean id="AdminRealm" class="cc.yihy.realm.AdminRealm">
        <property name="userService" ref="UserService" />
    </bean>
    <bean id="UserRealm" class="cc.yihy.realm.UserRealm">
        <property name="userService" ref="UserService" />
    </bean>
    <!-- 安全管理器 -->
    <bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
    <!-- filter&realm绑定 -->
        <property name="authenticator" ref="DefineModularRealmAuthenticator" />
    </bean>
    <bean id="lifecycleBeanPostProcessor" class="org.apache.shiro.spring.LifecycleBeanPostProcessor" />

```
到此，绑定工作完成。。。

欢迎拍砖提问。
用MarkDown写博客感觉还不错。
表示差点把Shiro开始创建执行的过程写上去。

## 在后面进行页面上权限校验会报错，原因出在改变了realm的注入方式。解决办法，近期我会整理出来。

## 权限校验问题修复
权限校验会报错是因为修改了注入realm的位置，导致授权解释器拿不到realm集合，现在把授权解释器重写了，并注入realms，使其恢复正常

``` xml


<!-- realms Map -->

    <bean id="defineRealms" class="org.springframework.beans.factory.config.MapFactoryBean">

        <property name="sourceMap">

            <map>

                <entry key="authc" value-ref="SSOUserRealm"/>

                <!--<entry key="NormalUser" value-ref="UserRealm"/>-->

                <!--<entry key="authUser" value-ref="UserRealm"/>-->

                <entry key="SSOUser" value-ref="SSOUserRealm"/>


                <entry key="mxUser" value-ref="AUserRealm"/>

                <entry key="normalMx" value-ref="AUserRealm"/>

                <entry key="mxAdmin" value-ref="AUserRealm"/>

                <entry key="subMxAdmin" value-ref="AUserRealm"/>


                <entry key="xhAdmin" value-ref="XhUserRealm"/>

                <entry key="xhUser" value-ref="XhUserRealm"/>

                <entry key="subXhUser" value-ref="XhUserRealm"/>

            </map>

        </property>

    </bean>

 <!-- filter&realm绑定 -->
    <!--认证解释器-->
    <bean id="DefineModularRealmAuthenticator"

          class="cc.yihy.shiro.authc.pam.DefineModularRealmAuthenticator">

        <property name="defineRealms" ref="defineRealms"/>

    </bean>


    <!--授权解释器-->
    <bean id="DefineModularRealmAuthorizer"

          class="cc.yihy.shiro.authz.DefineModularRealmAuthorizer">

        <property name="defineRealms" ref="defineRealms"/>

    </bean>

 <!-- securityManager安全管理器 -->

    <bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">


        <!-- filter&realm绑定 登录 -->

        <property name="authenticator" ref="DefineModularRealmAuthenticator"/>

        <!-- filter&realm绑定 角色权限验证 -->

        <property name="authorizer" ref="DefineModularRealmAuthorizer"/>

        <!-- 注入缓存管理器 -->

        <property name="cacheManager" ref="ShiroCacheManager"/>

        <!-- 注入session管理器 -->

        <property name="sessionManager" ref="sessionManager"/>

        <!-- 记住我 -->

        <property name="rememberMeManager" ref="rememberMeManager"/>

    </bean>
```
