>  Spring Boot 简化了spring大量的xml配置，使用了Java Config的配置，同时又提供了自动化配置工具。在引入新的组件时，只需要在配置文件上添加少量配置即可。如果需要更高自定义度配置，再添加极少的Java Config文件就好了。

## 简单介绍

### 废话

最近项目上要改造为Spring Boot，权限是本人使用的Shiro管理的。最快的办法是把Xml换成Java Config，但是配置也是不少。在网上查帖子也全是这样做的。本人想，Shiro有Spring Boot的自动化组件吗？打开Shiro的官网并没有找到，在Shiro的github的上，发现了它的自动化组件，有非Web环境的和Web环境的。在百度上没找到关于shiro-starter的使用，就想着写写博客讲一下。

### Shiro的Spring Boot 组件
![屏幕快照 2017-08-28 21.49.32.png](http://upload-images.jianshu.io/upload_images/2197548-ff5a4199b0ce06db.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

看了源码，shiro-spring-boot-web-starter依赖了shiro-spring-boot-starter，本人就说说shiro-spring-boot-web-starter的使用。

在resources/META-INF/**additional-spring-configuration-metadata.json**文件中，描述了Shiro提供的配置项。
``` json
{
  "groups": [
    {
      "name": "shiro"
    },
  ],
  "properties": [

    {
      "name": "shiro.web.enabled",
      "type": "java.lang.Boolean",
      "description": "shiro 自动化组件开关，默认值是打开",
      "defaultValue": true
    },
    {
      "name": "shiro.loginUrl",
      "type": "java.lang.String",
      "description": "配置shiro的认证(登录)Url",
      "defaultValue": "/login.jsp"
    },
    {
      "name": "shiro.successUrl",
      "type": "java.lang.String",
      "description": "配置shiro的认证(登录)成功后跳转的Url",
      "defaultValue": "/"
    },
    {
      "name": "shiro.unauthorizedUrl",
      "type": "java.lang.String",
      "description": "配置shiro的未授权跳转的Url，http状态码403",
      "defaultValue": null
    },
    {
      "name": "shiro.sessionManager.sessionIdCookieEnabled",
      "type": "java.lang.String",
      "description": "是否使用cookie存储session的id",
      "defaultValue": true
    },
    {
      "name": "shiro.sessionManager.sessionIdUrlRewritingEnabled",
      "type": "java.lang.String",
      "description": "通过URL参数启用或禁用会话跟踪。 如果您的网站需要Cookie，建议您禁用此功能。",
      "defaultValue": true
    }
  ]
}

```

配置项描述文件，在

![屏幕快照 2017-08-28 22.39.57.png](http://upload-images.jianshu.io/upload_images/2197548-80027e6233271def.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这个包里也有一部分配置。

### 全部配置

本人将所有的配置整理列出来
``` yml
shiro: 
  web: 
    ## 开启shiro web自动化配置，默认开启
    enabled: true
  loginUrl: /login.jsp
  successUrl: /
  ## 必须要配置为授权的url,否则在无权限的情况下，会找不到未授权url,导致找不到安全管理器（SecurityManager）
  unauthorizedUrl: null
  ## session管理方式，true使用shiro提供的session管理，false则使用servlet提供的session管理
  userNativeSessionManager: false
  ## 会话管理
  sessionManager: 
    sessionIdCookieEnabled: true
    sessionIdUrlRewritingEnabled: true
    deleteInvalidSessions: true
    cookie: 
      name: JSESSIONID
      maxAge: -1
      domain: null
      path: null
      secure: false
  ## 记住我管理
  rememberMeManager: 
    cookie: 
      name: rememberMe
      ## 默认一年
      maxAge: 60 * 60 * 24 * 365
      domain: null
      path: null
      secure: false
    
```


## 使用
下面就讲讲怎么使用

### 导入pom

```xml
		<dependency>
			<groupId>org.apache.shiro</groupId>
			<artifactId>shiro-spring-boot-web-starter</artifactId>
			<version>1.4.0</version>
		</dependency>
```

###  添加配置
```yml
shiro:
  loginUrl: /login.html
  unauthorizedUrl: /unauthorized
  ## 使用shiro管理会话
  userNativeSessionManager: true
  sessionManager:
    sessionIdUrlRewritingEnabled: false
    cookie:
      name: sid
```
### 增加配置类
```java
package cc.yihy.shiro.demo.config;

import org.apache.shiro.realm.Realm;
import org.apache.shiro.realm.text.TextConfigurationRealm;
import org.apache.shiro.spring.web.config.DefaultShiroFilterChainDefinition;
import org.apache.shiro.spring.web.config.ShiroFilterChainDefinition;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @Package: cc.yihy.shiro.demo.config
 * @date: 2017/8/28 23:21
 * @author: Yihy
 * @version: V1.0
 * @Description:
 */
@Configuration
public class ShiroConfig {

    /**
     * 验证用户
     * 可以声明多个Realm Bean，Shiro都会把它注入的
     * @return
     */
    @Bean
    public Realm realm() {
        TextConfigurationRealm realm = new TextConfigurationRealm();
        //添加两个用户
        //joe.coder=password 角色 user
        //jill.coder=password 角色 admin
        realm.setUserDefinitions("joe.coder=password,user\n" +
                "jill.coder=password,admin");

        //设置角色admin的权限是read,write
        //设置角色user的权限是read
        realm.setRoleDefinitions("admin=read,write\n" +
                "user=read");
        realm.setCachingEnabled(true);

        return realm;
    }

    /**
     * 配置shiro的url权限
     *
     * @return
     */
    @Bean
    public ShiroFilterChainDefinition shiroFilterChainDefinition() {
        DefaultShiroFilterChainDefinition chainDefinition = new DefaultShiroFilterChainDefinition();
        chainDefinition.addPathDefinition("/login.html", "authc"); // need to accept POSTs from the login form
        chainDefinition.addPathDefinition("/logout", "logout");
        chainDefinition.addPathDefinition("/account-info", "perms[write]");
        chainDefinition.addPathDefinition("/account-info1", "roles[admin]");
        return chainDefinition;
    }
}

```

这样就完成了Shiro的自动化配置。

### 补充 需要自定义shiro的Filter

让ShiroConfig继承ShiroWebFilterConfiguration（`org.apache.shiro.spring.config.web.autoconfigure.ShiroWebFilterConfiguration`）类，重写ShiroFilterFactoryBean方法,就可以达到目的。

```java
    @Override
    protected ShiroFilterFactoryBean shiroFilterFactoryBean() {
        ShiroFilterFactoryBean factoryBean = super.shiroFilterFactoryBean();
        Map<String,Filter> filterMap = new LinkedHashMap<>();
        //添加自定义的Filter,这里我随便new了一个filter
        filterMap.put("anyrole",new UserFilter());
        factoryBean.setFilters(filterMap);
        return factoryBean;
    }
```
## 小结

今天看的shiro的自动化组件，晚上写博文，本人shiro用的还不错，spring boot的东西也是用了很长时间。写博文的时候，同时也是头一次写了shiro自动化组件的demo。在测试授权的时候，当无权限的时候，一直给我报安全管理器找不到的问题。检查测试了半天，才发现是未授权url未配置的原因。

使用shiro的自动化配置，可以轻易的完成了shiro的配置。虽然很好，但也隐藏了很多的细节，以至于知道怎么用，出了问题就不知道怎么解决。建议有空还是看看底层如何实现的。

- 最后奉上demo : [点击](http://git.oschina.net/yihyforever/spring-boot-shiro-demo)
