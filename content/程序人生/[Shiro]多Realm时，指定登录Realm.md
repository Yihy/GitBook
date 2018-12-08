---
date : 2017-11-26T13:47:08+02:00
tags : []
title: 多Realm时，指定登录Realm
---


# [Shiro]多Realm时，指定登录Realm

> 之前写过多Realm登录前后台区分问题，实现了当时的需求，可是再下一步的时候，角色与权限校验会失效。为了满足当时的项目需求，又拓展了鉴权源，以及若干功能类。在本人看来，这种方式很Low，玷污了Shiro。
> **现在有一个较好的方式去实现前后台的认证与鉴权。通过自定义Token类来达到鉴别目的。**

-------------------

## Token的介绍

Shiro使用Token作为认证的对象。 在Filter中拦截登录信息创建Token调用登录，或者我们自己在控制器中手动登录。调用登录，会迭代配置的Realm进行验证。

### Shiro的FormAuthenticationFilter创建Token
FormAuthenticationFilter中创建Token的方法
![FormAuthenticationFilter](http://upload-images.jianshu.io/upload_images/2197548-d893653d809e521d?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

最终调用到它的父类AuthenticatingFilter
![AuthenticatingFilter](http://upload-images.jianshu.io/upload_images/2197548-47b5924f9ede154c?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 控制器里手动创建Token登录

![这里写图片描述](http://upload-images.jianshu.io/upload_images/2197548-26b38f9ebc14d420?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## Realm的定义

Shiro在进行登录验证时候，会检查Realm是否支持该Token，如果不支持跳过当前Realm，继续下一个Realm。

```java
	public Class getAuthenticationTokenClass() {
	    //支持的Token类的class
		return authenticationTokenClass;
	}
	//是否支持该token
	//Realm支持的类可以是token的子类或相同
    public boolean supports(AuthenticationToken token) {
        return token != null &&    getAuthenticationTokenClass().isAssignableFrom(token.getClass());
    }
```

自定义Realm一般会继承AuthorizingRealm来支持登录的验证。

我们可以重写public Class getAuthenticationTokenClass()来支持我们的Token。

## 自定义Token
Shiro的UsernamePasswordToken源码

```java
public class UsernamePasswordToken implements HostAuthenticationToken, RememberMeAuthenticationToken {

    private String username;

    private char[] password;

    private boolean rememberMe = false;

    private String host;

    public UsernamePasswordToken() {
    }

    public UsernamePasswordToken(final String username, final char[] password) {
        this(username, password, false, null);
    }
    public UsernamePasswordToken(final String username, final String password, final String host) {
        this(username, password != null ? password.toCharArray() : null, false, host);
    }
    public Object getPrincipal() {
        return getUsername();
    }

 
    public Object getCredentials() {
        return getPassword();
    }

  
    public String getHost() {
        return host;
    }

    public void setHost(String host) {
        this.host = host;
    }

    public boolean isRememberMe() {
        return rememberMe;
    }

    public void setRememberMe(boolean rememberMe) {
        this.rememberMe = rememberMe;
    }


    public void clear() {
        this.username = null;
        this.host = null;
        this.rememberMe = false;

        if (this.password != null) {
            for (int i = 0; i < password.length; i++) {
                this.password[i] = 0x00;
            }
            this.password = null;
        }

    }
}
```

UsernamePasswordToken实现了**AuthenticationToken**（认证）、**HostAuthenticationToken**（用户ip）、**RememberMeAuthenticationToken**（记住我）接口。

AuthenticationToken

```
public interface AuthenticationToken extends Serializable {

    /**
    *获取登陆用户名
    */
    Object getPrincipal();

    /**
     *获取密码
     */
    Object getCredentials();

}
```
由于Realm检查是否支持Token类的是允许支持验证子类的，我们不能去继承UsernamePasswordToken，需要自己实现一个Token。直接拷贝代码，修改类名就可以。或者根据你的业务需求，重写一部分功能。


----------


> **每一个自定义Realm对应支持一种Token。就可以使登录验证时不会重复调用Realm多次验证和异常消息的覆盖。**


[示例代码] [https://git.oschina.net/yihyforever/shiro-realms.git](https://git.oschina.net/yihyforever/shiro-realms.git)
----------
***下一篇，Shiro的缓存及坑点***
