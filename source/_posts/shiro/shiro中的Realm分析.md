---
title: shiro中的Realm分析
date: 2018-06-05 22:26:30
tags: shiro
category: shiro
---

# shiro中的Realm分析撒
`Realm`是shiro中定义认证和权限的地方。

## `Realm`接口

    public interface Realm {
        String getName();
        boolean supports(AuthenticationToken var1);//判断是否支持一个AuthenticationToken类型
        AuthenticationInfo getAuthenticationInfo(AuthenticationToken var1) throws AuthenticationException;
    }

## `CachingRealm`抽象类
    
    public abstract class CachingRealm implements Realm, Nameable, CacheManagerAware, LogoutAware {
        private static final AtomicInteger INSTANCE_COUNT = new AtomicInteger();
        private String name;
        private boolean cachingEnabled = true;
        private CacheManager cacheManager;
    }
增加了`CacheManager`类型的域 `cacheManager`实现缓存功能。

## `AuthenticatingRealm`类
    
    public abstract class AuthenticatingRealm extends CachingRealm implements Initializable {
        private static final Logger log = LoggerFactory.getLogger(AuthenticatingRealm.class);
        private static final AtomicInteger INSTANCE_COUNT = new AtomicInteger();
        private static final String DEFAULT_AUTHORIZATION_CACHE_SUFFIX = ".authenticationCache";
        private CredentialsMatcher credentialsMatcher;
        private Cache<Object, AuthenticationInfo> authenticationCache;
        private boolean authenticationCachingEnabled;
        private String authenticationCacheName;
        private Class<? extends AuthenticationToken> authenticationTokenClass;
        ...
    }
看其重写的`getAuthenticationInfo`方法：

    public final AuthenticationInfo getAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        AuthenticationInfo info = this.getCachedAuthenticationInfo(token);
        if (info == null) {
            info = this.doGetAuthenticationInfo(token);
            if (token != null && info != null) {
                this.cacheAuthenticationInfoIfPossible(token, info);
            }
        } else {
        }
        if (info != null) {
            this.assertCredentialsMatch(token, info);
        } else {
        }
        return info;
    }
首先尝试从缓存中获取`AuthenticationInfo`，如果缓存中没有，调用`doGetAuthenticationInfo`方法获取，获取到将其缓存起来。如果成功获取到，则会调用`assertCredentialsMatch`方法判断token和返回的信息是否匹配：

    protected void assertCredentialsMatch(AuthenticationToken token, AuthenticationInfo info) throws AuthenticationException {
        CredentialsMatcher cm = this.getCredentialsMatcher();
        if (cm != null) {
            if (!cm.doCredentialsMatch(token, info)) {
                String msg = "Submitted credentials for token [" + token + "] did not match the expected credentials.";
                throw new IncorrectCredentialsException(msg);
            }
        } else {
            throw new AuthenticationException("A CredentialsMatcher must be configured in order to verify credentials during authentication.  If you do not wish for credentials to be examined, you can configure an " + AllowAllCredentialsMatcher.class.getName() + " instance.");
        }
    }
不匹配就报错。
