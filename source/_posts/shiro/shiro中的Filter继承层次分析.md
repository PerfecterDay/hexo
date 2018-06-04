---
title: shiro中的Filter继承层次分析
date: 2018-06-04 16:40:35
tags: shiro
category: shiro
---

# shiro中的Filter继承层次分析

## `AbstractFilter`直接实现`javax.servlet.Filter`接口
有一个`javax.servlet.FilterConfig`类域，可以通过这个对象来获取Filter的一些信息。该对象由Servlet规范在调用Filter时，传递给Filter对象。

该类重写了Filter的init()方法，

    public final void init(FilterConfig filterConfig) throws ServletException {
            this.setFilterConfig(filterConfig);  //获取传递过来的FilterConfig对象并保存
            try {
                //调用自定义的onFilterConfigSet方法，子类可重写此方法实现某些需求
                this.onFilterConfigSet(); 
            } catch (Exception var3) {
                if(var3 instanceof ServletException) {
                    throw (ServletException)var3;
                } else {
                    if(log.isErrorEnabled()) {
                        log.error("Unable to start Filter: [" + var3.getMessage() + "].", var3);
                    }

                    throw new ServletException(var3);
                }
            }
        }
## `NameableFilter`继承自`AbstractFilter`
该类添加了一个String类型的name域，用来为Filter命名，此name与FilterConfig中的FilterName不同，属于shiro为Filter自定义的名字。

## `OncePerRequestFilter`继承自`NameableFilter`
添加了如下域：

    public static final String ALREADY_FILTERED_SUFFIX = ".FILTERED";
    private boolean enabled = true;//代表该filter是否可用
重写了`doFilter`方法：
    
    public final void doFilter(ServletRequest request, ServletResponse response, FilterChain filterChain) throws ServletException, IOException {
            String alreadyFilteredAttributeName = this.getAlreadyFilteredAttributeName();
            if(request.getAttribute(alreadyFilteredAttributeName) != null) {
                log.trace("Filter \'{}\' already executed.  Proceeding without invoking this filter.", this.getName());
                filterChain.doFilter(request, response);
            } else if(this.isEnabled(request, response) && !this.shouldNotFilter(request)) {
                log.trace("Filter \'{}\' not yet executed.  Executing now.", this.getName());
                request.setAttribute(alreadyFilteredAttributeName, Boolean.TRUE);

                try {
                    this.doFilterInternal(request, response, filterChain);
                } finally {
                    request.removeAttribute(alreadyFilteredAttributeName);
                }
            } else {
                log.debug("Filter \'{}\' is not enabled for the current request.  Proceeding without invoking this filter.", this.getName());
                filterChain.doFilter(request, response);
            }

        }
可以看出该方法首先获取一个字符串：
    
    String alreadyFilteredAttributeName = this.getAlreadyFilteredAttributeName();
该字符串是name+".FILTERED"的一个字符串。
然后，用此String去调用requets.getAttribute(alreadyFilteredAttributeName)判断是否有值：
1. 如果有，代表该filter被执行过，直接调用`filterChain.doFilter(request, response);`执行filter链中的下一个。
2. 如果没有，判断`this.isEnabled(request, response) && !this.shouldNotFilter(request)`，如果该filter可用，则将`alreadyFilteredAttributeName`加入到request的attribute中，并调用`doFilterInternal`方法。并在执行完后调用`request.removeAttribute(alreadyFilteredAttributeName);`方法清除调用过的标记。
3. 其它情况直接执行filter链中的下一个。

## `AdviceFilter`继承自`OncePerRequestFilter`
Advice是增强的意思，类似Spring AOP中的Advice.该类并没有添加其它域，只是重写了`doFilterInternal`方法，为其子类提供多个扩展点。

    public void doFilterInternal(ServletRequest request, ServletResponse response, FilterChain chain) throws ServletException, IOException {
            Exception exception = null;
            try {
                boolean e = this.preHandle(request, response);
                if(log.isTraceEnabled()) {
                    log.trace("Invoked preHandle method.  Continuing chain?: [" + e + "]");
                }
                if(e) {
                    this.executeChain(request, response, chain);
                }
                this.postHandle(request, response);
                if(log.isTraceEnabled()) {
                    log.trace("Successfully invoked postHandle method");
                }
            } catch (Exception var9) {
                exception = var9;
            } finally {
                this.cleanup(request, response, exception);
            }
        }
可以看出，`doFilterInternal`方法依次调用了`preHandle`、`executeChain`、`postHandle`方法。`executeChain`只是`chain.doFilter(request, response);`的封装，代表直接跳到下一个filter去执行。

`preHandle`,`postHandle`是扩展点，子类可以重写这两个方法。

## `PathMatchingFilter`继承自`AdviceFilter`
该类添加了两个域：

    protected PatternMatcher pathMatcher = new AntPathMatcher();
    protected Map<String, Object> appliedPaths = new LinkedHashMap();
`pathMatcher`是解析path路径的解析器，appliedPaths代表接收的路径。看下面代码分析。

该类重写了`preHandle`方法：

    protected boolean preHandle(ServletRequest request, ServletResponse response) throws Exception {
            if(this.appliedPaths != null && !this.appliedPaths.isEmpty()) {
                Iterator var3 = this.appliedPaths.keySet().iterator();

                String path;
                do {
                    if(!var3.hasNext()) {
                        return true;
                    }

                    path = (String)var3.next();
                } while(!this.pathsMatch(path, request));

                log.trace("Current requestURI matches pattern \'{}\'.  Determining filter chain execution...", path);
                Object config = this.appliedPaths.get(path);
                return this.isFilterChainContinued(request, response, path, config);
            } else {
                if(log.isTraceEnabled()) {
                    log.trace("appliedPaths property is null or empty.  This Filter will passthrough immediately.");
                }

                return true;
            }
        }
该方法首先用键遍历`appliedPaths` Map中的对象，对比request的路径和遍历对象的值，直到找到一个匹配的路径，这个过程中会用到`pathMatcher`路径解析器。如果有某个路径匹配request路径，调用

    Object config = this.appliedPaths.get(path);
    return this.isFilterChainContinued(request, response, path, config);
获取对应path键的值，并调用`isFilterChainContinued`方法。

    private boolean isFilterChainContinued(ServletRequest request, ServletResponse response, String path, Object pathConfig) throws Exception {
            if(this.isEnabled(request, response, path, pathConfig)) {
                ......
                return this.onPreHandle(request, response, pathConfig);
            } else {
                ......
                return true;
            }
        }
该方法中调用了`onPreHandle`方法，此方法也是一个扩展点。

## `AccessControlFilter`继承自`PathMatchingFilter`
该类增加了四个域：
    
    public static final String DEFAULT_LOGIN_URL = "/login.jsp";
    public static final String GET_METHOD = "GET";
    public static final String POST_METHOD = "POST";
    private String loginUrl = "/login.jsp";
最重要的是重写了`onPreHandle`方法：
    
    public boolean onPreHandle(ServletRequest request, ServletResponse response, Object mappedValue) throws Exception {
        return this.isAccessAllowed(request, response, mappedValue) || this.onAccessDenied(request, response, mappedValue);
    }
注意这里的用法：短路算法。如果`isAccessAllowed`为真则后边`onAccessDenied`的不会执行，否则会执行`onAccessDenied`方法。

`isAccessAllowed`是个扩展点，子类可以重写此方法根据自身需求定义请求是否允许的判断功能。`onAccessDenied`也是个扩展点，子类可以重写此方法定义拒绝访问后的操作。

此类中还定义`isLoginRequest`、`saveRequestAndRedirectToLogin`等方法，其中：
    
    protected void saveRequest(ServletRequest request) {
        WebUtils.saveRequest(request);
    }
WebUtils中的saveRequest方法定义：

    public static void saveRequest(ServletRequest request) {
            Subject subject = SecurityUtils.getSubject();
            Session session = subject.getSession();
            HttpServletRequest httpRequest = toHttp(request);
            SavedRequest savedRequest = new SavedRequest(httpRequest);
            session.setAttribute("shiroSavedRequest", savedRequest);
        }
通过session保存了此次请求的信息。

## `AuthenticationFilter`继承自`AccessControlFilter`
该类主要是重写了`isAccessAllowed`方法：

    protected boolean isAccessAllowed(ServletRequest request, ServletResponse response, Object mappedValue) {
        Subject subject = this.getSubject(request, response);
        return subject.isAuthenticated();
    }
调用Subject的isAuthenticated()方法，返回该用户是否被认证过。

## `AuthenticatingFilter`继承自`AuthenticationFilter`

    protected boolean isAccessAllowed(ServletRequest request, ServletResponse response, Object mappedValue) {
        return super.isAccessAllowed(request, response, mappedValue) || !this.isLoginRequest(request, response) && this.isPermissive(mappedValue);
    }