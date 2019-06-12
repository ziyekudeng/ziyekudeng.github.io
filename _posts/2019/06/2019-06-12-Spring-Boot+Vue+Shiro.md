---
layout: post
title: Spring Boot + Vue + Shiro 实现前后端分离、权限控制
category: arch
tags: [arch]
keywords: 架构
---

来源：http://sina.lt/gauR

*   一、前后端分离思想

*   二、后端 Springboot

*   三、前端 Vue + ElementUI + Vue router + Vuex + axios + webpack

*   四、前后端分离项目中集成shiro

*   五、部署项目

* * *

本文总结自实习中对项目的重构。原先项目采用Springboot+freemarker模版，开发过程中觉得前端逻辑写的实在恶心，后端Controller层还必须返回Freemarker模版的ModelAndView，逐渐有了前后端分离的想法，由于之前，没有接触过，主要参考的还是网上的一些博客教程等，初步完成了前后端分离，在此记录以备查阅。

# 一、前后端分离思想

前端从后端剥离，形成一个前端工程，前端只利用Json来和后端进行交互，后端不返回页面，只返回Json数据。前后端之间完全通过public API约定。

# 二、后端 Springboot

Springboot就不再赘述了，Controller层返回Json数据。

     @RequestMapping(value = "/add", method = RequestMethod.POST)
        @ResponseBody
        public JSONResult addClient(@RequestBody String param) {
            JSONObject jsonObject = JSON.parseObject(param);
            String task = jsonObject.getString("task");
            List<Object> list = jsonObject.getJSONArray("attributes");
            List<String> attrList = new LinkedList(list);
            Client client = JSON.parseObject(jsonObject.getJSONObject("client").toJSONString(),new TypeReference<Client>(){});
            clientService.addClient(client, task, attrList);
            return JSONResult.ok();
        }

Post请求使用@RequestBody参数接收。

# 三、前端 Vue + ElementUI + Vue router + Vuex + axios + webpack

主要参考：

> https://cn.vuejs.org/v2/guide/
> 
> https://github.com/PanJiaChen/vue-admin-template/blob/master/README-zh.md
> 
> https://github.com/PanJiaChen/vue-element-admin

这里主要说一下开发工程中遇到的问题：

**1.跨域**

由于开发中前端工程使用webpack启了一个服务，所以前后端并不在一个端口下，必然涉及到跨域：

> XMLHttpRequest会遵守同源策略(same-origin policy). 也即脚本只能访问相同协议/相同主机名/相同端口的资源, 如果要突破这个限制, 那就是所谓的跨域, 此时需要遵守CORS(Cross-Origin Resource Sharing)机制。

解决跨域分两种：

**1、server端是自己开发的，这样可以在在后端增加一个拦截器**

    @Component
    public class CommonIntercepter implements HandlerInterceptor {
    
        private final Logger logger = LoggerFactory.getLogger(this.getClass());
    
        @Override
        public boolean preHandle(HttpServletRequest request,
                                 HttpServletResponse response, Object handler) throws Exception {
            //允许跨域,不能放在postHandle内
            response.setHeader("Access-Control-Allow-Origin", "*");
            if (request.getMethod().equals("OPTIONS")) {
                response.addHeader("Access-Control-Allow-Methods", "GET,HEAD,POST,PUT,DELETE,TRACE,OPTIONS,PATCH");
                response.addHeader("Access-Control-Allow-Headers", "Content-Type, Accept, Authorization");
            }
            return true;
        }
    }

> response.setHeader("Access-Control-Allow-Origin", "*");

主要就是在Response Header中增加 "Access-Control-Allow-Origin: *"

     if (request.getMethod().equals("OPTIONS")) {
                response.addHeader("Access-Control-Allow-Methods", "GET,HEAD,POST,PUT,DELETE,TRACE,OPTIONS,PATCH");
                response.addHeader("Access-Control-Allow-Headers", "Content-Type, Accept, Authorization");
            }

由于我们在前后端分离中集成了shiro，因此需要在headers中自定义一个'Authorization'字段，此时普通的GET、POST等请求会变成preflighted request，即在GET、POST请求之前会预先发一个OPTIONS请求，这个后面再说。推荐一篇博客介绍 preflighted request。

> https://blog.csdn.net/cc1314_/article/details/78272329

**2、server端不是自己开发的，可以在前端加proxyTable。**

不过这个只能在开发的时候用，后续部署，可以把前端项目作为静态资源放到后端，这样就不存在跨域（由于项目需要，我现在是这么做的，根据网上博客介绍，可以使用nginx，具体怎么做可以在网上搜一下）。

遇到了网上很多人说的，proxyTable无论如何修改，都没效果的现象。

1、（非常重要）确保proxyTable配置的地址能访问，因为如果不能访问，在浏览器F12调试的时候看到的依然会是提示404。

> 并且注意，在F12看到的js提示错误的域名，是js写的那个域名，并不是代理后的域名。（l楼主就遇到这个问题，后端地址缺少了查询参数，代理设置为后端地址，然而F12看到的错误依然还是本地的域名，并不是代理后的域名）

2、就是要手动再执行一次npm run dev

# 四、前后端分离项目中集成shiro

可以参考：

> blog.csdn.net/u013615903/article/details/78781166

这里说一下实际开发集成过程中遇到的问题：

**1、OPTIONS请求不带'Authorization'请求头字段：**

前后端分离项目中，由于跨域，会导致复杂请求，即会发送preflighted request，这样会导致在GET／POST等请求之前会先发一个OPTIONS请求，但OPTIONS请求并不带shiro的'Authorization'字段（shiro的Session），即OPTIONS请求不能通过shiro验证，会返回未认证的信息。

解决方法：给shiro增加一个过滤器，过滤OPTIONS请求
    
    public class CORSAuthenticationFilter extends FormAuthenticationFilter {
    
        private static final Logger logger = LoggerFactory.getLogger(CORSAuthenticationFilter.class);
    
        public CORSAuthenticationFilter() {
            super();
        }
    
        @Override
        public boolean isAccessAllowed(ServletRequest request, ServletResponse response, Object mappedValue) {
            //Always return true if the request's method is OPTIONSif (request instanceof HttpServletRequest) {
                if (((HttpServletRequest) request).getMethod().toUpperCase().equals("OPTIONS")) {
                    return true;
                }
            }
    return super.isAccessAllowed(request, response, mappedValue);
        }
    
        @Override
        protected boolean onAccessDenied(ServletRequest request, ServletResponse response) throws Exception {
            HttpServletResponse res = (HttpServletResponse)response;
            res.setHeader("Access-Control-Allow-Origin", "*");
            res.setStatus(HttpServletResponse.SC_OK);
            res.setCharacterEncoding("UTF-8");
            PrintWriter writer = res.getWriter();
            Map<String, Object> map= new HashMap<>();
            map.put("code", 702);
            map.put("msg", "未登录");
            writer.write(JSON.toJSONString(map));
            writer.close();
            return false;
        }
    }

贴一下我的config文件：
    
    @Configuration
    public class ShiroConfig {
    
        @Bean
        public Realm realm() {
            return new DDRealm();
        }
    
        @Bean
        public CacheManager cacheManager() {
            return new MemoryConstrainedCacheManager();
        }
    
        /**
         * cookie对象;
         * rememberMeCookie()方法是设置Cookie的生成模版，比如cookie的name，cookie的有效时间等等。
         * @return
         */
        @Bean
        public SimpleCookie rememberMeCookie(){
            //System.out.println("ShiroConfiguration.rememberMeCookie()");
            //这个参数是cookie的名称，对应前端的checkbox的name = rememberMe
            SimpleCookie simpleCookie = new SimpleCookie("rememberMe");
            //<!-- 记住我cookie生效时间30天 ,单位秒;-->
            simpleCookie.setMaxAge(259200);
            return simpleCookie;
        }
    
        /**
         * cookie管理对象;
         * rememberMeManager()方法是生成rememberMe管理器，而且要将这个rememberMe管理器设置到securityManager中
         * @return
         */
        @Bean
        public CookieRememberMeManager rememberMeManager(){
            //System.out.println("ShiroConfiguration.rememberMeManager()");
            CookieRememberMeManager cookieRememberMeManager = new CookieRememberMeManager();
            cookieRememberMeManager.setCookie(rememberMeCookie());
            //rememberMe cookie加密的密钥 建议每个项目都不一样 默认AES算法 密钥长度(128 256 512 位)
            cookieRememberMeManager.setCipherKey(Base64.decode("2AvVhdsgUs0FSA3SDFAdag=="));
            return cookieRememberMeManager;
        }
    
        @Bean
        public SecurityManager securityManager() {
            DefaultWebSecurityManager sm = new DefaultWebSecurityManager();
            sm.setRealm(realm());
            sm.setCacheManager(cacheManager());
            //注入记住我管理器
            sm.setRememberMeManager(rememberMeManager());
            //注入自定义sessionManager
            sm.setSessionManager(sessionManager());
            return sm;
        }
    
        //自定义sessionManager
        @Bean
        public SessionManager sessionManager() {
            return new CustomSessionManager();
        }
    
        public CORSAuthenticationFilter corsAuthenticationFilter(){
            return new CORSAuthenticationFilter();
        }
    
        @Bean(name = "shiroFilter")
        public ShiroFilterFactoryBean getShiroFilterFactoryBean(SecurityManager securityManager) {
            ShiroFilterFactoryBean shiroFilter = new ShiroFilterFactoryBean();
            shiroFilter.setSecurityManager(securityManager);
            //SecurityUtils.setSecurityManager(securityManager);
            Map<String, String> filterChainDefinitionMap = new LinkedHashMap<>();
            //配置不会被拦截的链接，顺序判断
            filterChainDefinitionMap.put("/", "anon");
            filterChainDefinitionMap.put("/static/js/**", "anon");
            filterChainDefinitionMap.put("/static/css/**", "anon");
            filterChainDefinitionMap.put("/static/fonts/**", "anon");
            filterChainDefinitionMap.put("/login/**", "anon");
            filterChainDefinitionMap.put("/corp/call_back/receive", "anon");
            //authc:所有url必须通过认证才能访问，anon:所有url都可以匿名访问
            filterChainDefinitionMap.put("/**", "corsAuthenticationFilter");
            shiroFilter.setFilterChainDefinitionMap(filterChainDefinitionMap);
            //自定义过滤器
            Map<String, Filter> filterMap = new LinkedHashMap<>();
            filterMap.put("corsAuthenticationFilter", corsAuthenticationFilter());
            shiroFilter.setFilters(filterMap);
    
            return shiroFilter;
        }
    
        /**
         * Shiro生命周期处理器 * @return
         */
        @Bean
        public LifecycleBeanPostProcessor lifecycleBeanPostProcessor() {
            return new LifecycleBeanPostProcessor();
        }
    
        /**
         * 开启Shiro的注解(如@RequiresRoles,@RequiresPermissions),需借助SpringAOP扫描使用Shiro注解的类,并在必要时进行安全逻辑验证 * 配置以下两个bean(DefaultAdvisorAutoProxyCreator(可选)和AuthorizationAttributeSourceAdvisor)即可实现此功能 * @return
         */
        @Bean
        @DependsOn({"lifecycleBeanPostProcessor"})
        public DefaultAdvisorAutoProxyCreator advisorAutoProxyCreator() {
            DefaultAdvisorAutoProxyCreator advisorAutoProxyCreator = new DefaultAdvisorAutoProxyCreator();
            advisorAutoProxyCreator.setProxyTargetClass(true);
            return advisorAutoProxyCreator;
        }
    
        @Bean
        public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor(SecurityManager securityManager) {
            AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor = new AuthorizationAttributeSourceAdvisor();
            authorizationAttributeSourceAdvisor.setSecurityManager(securityManager);
            return authorizationAttributeSourceAdvisor;
        }
    }

**2、设置session失效时间**

shiro session默认失效时间是30min，我们在自定义的sessionManager的构造函数中设置失效时间为其他值
    
    public class CustomSessionManager extends DefaultWebSessionManager {
    
        private static final Logger logger = LoggerFactory.getLogger(CustomSessionManager.class);
    
        private static final String AUTHORIZATION = "Authorization";
    
        private static final String REFERENCED_SESSION_ID_SOURCE = "Stateless request";
    
        public CustomSessionManager() {
            super();
            setGlobalSessionTimeout(DEFAULT_GLOBAL_SESSION_TIMEOUT * 48);
        }
    
        @Override
        protected Serializable getSessionId(ServletRequest request, ServletResponse response) {
            String sessionId = WebUtils.toHttp(request).getHeader(AUTHORIZATION);//如果请求头中有 Authorization 则其值为sessionId
            if (!StringUtils.isEmpty(sessionId)) {
                request.setAttribute(ShiroHttpServletRequest.REFERENCED_SESSION_ID_SOURCE, REFERENCED_SESSION_ID_SOURCE);
                request.setAttribute(ShiroHttpServletRequest.REFERENCED_SESSION_ID, sessionId);
                request.setAttribute(ShiroHttpServletRequest.REFERENCED_SESSION_ID_IS_VALID, Boolean.TRUE);
                return sessionId;
            } else {
                //否则按默认规则从cookie取sessionId
                return super.getSessionId(request, response);
            }
        }
    }
# 五、部署项目

前端项目部署主要分两种方法：

**1.将前端项目打包（npm run build）成静态资源文件，放入后端，一起打包。**后端写一个Controller返回前端界面（我使用Vue开发的是单页面应用），但是这样其实又将前后端耦合在一起了，不过起码做到前后端分离开发，方便开发的目的已经达成，也初步达成了要求，由于项目的需要，我是这样做的，并且免去了跨域问题。

    @RequestMapping(value = {"/", "/index"}, method = RequestMethod.GET)
        public String index() {
            return "/index";
        }

**2.将前端工程另启一个服务（tomcat，nginx，nodejs），这样有跨域的问题。**

**说一下我遇到的问题：**

1、nginx反向代理，导致当访问无权限的页面时，shiro 302到unauth的controller，访问的地址是https，重定向地址是http，导致了无法访问。

不使用shiro的 shiroFilter.setLoginUrl("/unauth");

当页面无权限访问时，我们在过滤器里直接返回错误信息，不利用shiro自带的跳转。看过滤器中的onAccessDenied函数
    
    public class CORSAuthenticationFilter extends FormAuthenticationFilter {
    
        private static final Logger logger = LoggerFactory.getLogger(CORSAuthenticationFilter.class);
    
        public CORSAuthenticationFilter() {
            super();
        }
    
        @Override
        public boolean isAccessAllowed(ServletRequest request, ServletResponse response, Object mappedValue) {
            //Always return true if the request's method is OPTIONS
            if (request instanceof HttpServletRequest) {
                if (((HttpServletRequest) request).getMethod().toUpperCase().equals("OPTIONS")) {
                    return true;
                }
            }
            return super.isAccessAllowed(request, response, mappedValue);
        }
    
        @Override
        protected boolean onAccessDenied(ServletRequest request, ServletResponse response) throws Exception {
            HttpServletResponse res = (HttpServletResponse)response;
            res.setHeader("Access-Control-Allow-Origin", "*");
            res.setStatus(HttpServletResponse.SC_OK);
            res.setCharacterEncoding("UTF-8");
            PrintWriter writer = res.getWriter();
            Map<String, Object> map= new HashMap<>();
            map.put("code", 702);
            map.put("msg", "未登录");
            writer.write(JSON.toJSONString(map));
            writer.close();
            return false;
        }
    }

先记录这么多，有不对的地方，欢迎指出！



