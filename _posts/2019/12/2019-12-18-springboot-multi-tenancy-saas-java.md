---
layout: post
title: Springboot项目使用动态切换数据源实现多租户SaaS方案 
category: arch
tags: [arch]
keywords: 架构
---

一、前言
====

工作中遇到了多组户的需求，因为以前并没有接触过，所以多番查找资料，最后总算做出来了，再此做个总结，记录一下以便日后复习也可以帮助用得着的朋友。

实现多租户大体可以分为三种方案：

**1、独立数据库，通过动态切换数据源来实现多租户，安全性最高，但成本也高。**

**2、共享数据库，隔离数据架构，比如使用oracle用多个schema。**

**3、共享数据库，共享数据库表，使用字段来区分不同租户，此方案成本最低，但同时安全性最低。**

详细介绍可以[点这里](https://www.cnblogs.com/niuben/p/11063731.html)参考这篇文章。

本项目因为对数据安全性要求较高，所以选择的第一种独立数据库切换动态数据源的方案。

二、实现方案
======

（一）AbstractRoutingDataSource
----------------------------

首先了解下 AbstractRoutingDataSource，看名字是一个数据源的路由，也就是由它来确定数据源，咱们先看一下源码

    public abstract class AbstractRoutingDataSource extends AbstractDataSource implements InitializingBean {
    
    	@Nullable
    	private Map<Object, Object> targetDataSources;
    	@Nullable
    	private Object defaultTargetDataSource;
    
    	private boolean lenientFallback = true;
    
    	private DataSourceLookup dataSourceLookup = new JndiDataSourceLookup();
    
    	@Nullable
    	private Map<Object, DataSource> resolvedDataSources;
    
    	@Nullable
    	private DataSource resolvedDefaultDataSource;
    
    	public void setTargetDataSources(Map<Object, Object> targetDataSources) {
    		this.targetDataSources = targetDataSources;
    	}
    	public void setDefaultTargetDataSource(Object defaultTargetDataSource) {
    		this.defaultTargetDataSource = defaultTargetDataSource;
    	}
    	public void setLenientFallback(boolean lenientFallback) {
    		this.lenientFallback = lenientFallback;
    	}
    	public void setDataSourceLookup(@Nullable DataSourceLookup dataSourceLookup) {
    		this.dataSourceLookup = (dataSourceLookup != null ? dataSourceLookup : new JndiDataSourceLookup());
    	}
    
    	@Override
    	public void afterPropertiesSet() {
    		if (this.targetDataSources == null) {
    			throw new IllegalArgumentException("Property 'targetDataSources' is required");
    		}
    		this.resolvedDataSources = new HashMap<>(this.targetDataSources.size());
    		this.targetDataSources.forEach((key, value) -> {
    			Object lookupKey = resolveSpecifiedLookupKey(key);
    			DataSource dataSource = resolveSpecifiedDataSource(value);
    			this.resolvedDataSources.put(lookupKey, dataSource);
    		});
    		if (this.defaultTargetDataSource != null) {
    			this.resolvedDefaultDataSource = resolveSpecifiedDataSource(this.defaultTargetDataSource);
    		}
    	}
    	
    	protected Object resolveSpecifiedLookupKey(Object lookupKey) {
    		return lookupKey;
    	}
    
    	protected DataSource resolveSpecifiedDataSource(Object dataSource) throws IllegalArgumentException {
    		if (dataSource instanceof DataSource) {
    			return (DataSource) dataSource;
    		}
    		else if (dataSource instanceof String) {
    			return this.dataSourceLookup.getDataSource((String) dataSource);
    		}
    		else {
    			throw new IllegalArgumentException(
    					"Illegal data source value - only [javax.sql.DataSource] and String supported: " + dataSource);
    		}
    	}
    
    	@Override
    	public Connection getConnection() throws SQLException {
    		return determineTargetDataSource().getConnection();
    	}
    
    	@Override
    	public Connection getConnection(String username, String password) throws SQLException {
    		return determineTargetDataSource().getConnection(username, password);
    	}
    
    	@Override
    	@SuppressWarnings("unchecked")
    	public <T> T unwrap(Class<T> iface) throws SQLException {
    		if (iface.isInstance(this)) {
    			return (T) this;
    		}
    		return determineTargetDataSource().unwrap(iface);
    	}
    
    	@Override
    	public boolean isWrapperFor(Class<?> iface) throws SQLException {
    		return (iface.isInstance(this) || determineTargetDataSource().isWrapperFor(iface));
    	}
    
    	protected DataSource determineTargetDataSource() {
    		Assert.notNull(this.resolvedDataSources, "DataSource router not initialized");
    		Object lookupKey = determineCurrentLookupKey();
    		DataSource dataSource = this.resolvedDataSources.get(lookupKey);
    		if (dataSource == null && (this.lenientFallback || lookupKey == null)) {
    			dataSource = this.resolvedDefaultDataSource;
    		}
    		if (dataSource == null) {
    			throw new IllegalStateException("Cannot determine target DataSource for lookup key [" + lookupKey + "]");
    		}
    		return dataSource;
    	}
    
    	@Nullable
    	protected abstract Object determineCurrentLookupKey();
    
    }
    

### 1\. afterPropertiesSet()

可以看到里面维护了一个 targetDataSources 和 defaultTargetDataSource，初始化时将数据源分别进行复制到resolvedDataSources和resolvedDefaultDataSource中，代码如下

    @Override
    public void afterPropertiesSet() {
    	if (this.targetDataSources == null) {
    		throw new IllegalArgumentException("Property 'targetDataSources' is required");
    	}
    	this.resolvedDataSources = new HashMap<>(this.targetDataSources.size());
    	this.targetDataSources.forEach((key, value) -> {
    		Object lookupKey = resolveSpecifiedLookupKey(key);
    		DataSource dataSource = resolveSpecifiedDataSource(value);
    		this.resolvedDataSources.put(lookupKey, dataSource);
    	});
    	if (this.defaultTargetDataSource != null) {
    		this.resolvedDefaultDataSource = resolveSpecifiedDataSource(this.defaultTargetDataSource);
    	}
    }

### 2\. getConnection()

调用此方法获取连接的时候，如下代码determineTargetDataSource().getConnection()，先调用determineTargetDataSource()方法返回当前的DataSource，然后再调用getConnection()。

### 3\. determineTargetDataSource

此方法的就是根据lookupkey获取map中的dataSource，而lookupkey是从determineCurrentLookupKey方法返回的，如下：

    protected DataSource determineTargetDataSource() {
    	Assert.notNull(this.resolvedDataSources, "DataSource router not initialized");
    	Object lookupKey = determineCurrentLookupKey();
    	DataSource dataSource = this.resolvedDataSources.get(lookupKey);
    	if (dataSource == null && (this.lenientFallback || lookupKey == null)) {
    		dataSource = this.resolvedDefaultDataSource;
    	}
        if (dataSource == null) {
    		throw new IllegalStateException("Cannot determine target DataSource for lookup key [" + lookupKey + "]");
    	}
    	return dataSource;
    }

### 4.determineCurrentLookupKey

此方法要我们自己实现，是切换数据源的方法，通过自己的实现返回lookupKey，根据lookupKey获取对应数据源达到切换动态切换的功能。

（二）自定义DynamicDataSource
-----------------------

自定义DynamicDataSource继承 AbstractRoutingDataSource，由上得知，我们先要有个方法能设置 targetDataSources，然后要重写determineCurrentLookupKey方法，来实现动态切换，代码如下：

    /**
     * （切换数据源必须在调用service之前进行，也就是开启事务之前）
     * 动态数据源实现类
     * @author Louis
     * @date Oct 31, 2018
     */
    public class DynamicDataSource extends AbstractRoutingDataSource {
        /**
         * 如果不希望数据源在启动配置时就加载好，可以定制这个方法，从任何你希望的地方读取并返回数据源
         * 比如从数据库、文件、外部接口等读取数据源信息，并最终返回一个DataSource实现类对象即可
         */
        @Override
        protected DataSource determineTargetDataSource() {
            return super.determineTargetDataSource();
        }
        /**
         * 如果希望所有数据源在启动配置时就加载好，这里通过设置数据源Key值来切换数据，定制这个方法
         */
        @Override
        protected Object determineCurrentLookupKey() {
            return DynamicDataSourceContextHolder.getDataSourceKey();
        }
        /**
         * 设置默认数据源
         * @param defaultDataSource
         */
        public void setDefaultDataSource(Object defaultDataSource) {
            super.setDefaultTargetDataSource(defaultDataSource);
        }
        /**
         * 设置数据源
         * @param dataSources
         */
        public void setDataSources(Map<Object, Object> dataSources) {
            super.setTargetDataSources(dataSources);
            // 将数据源的 key 放到数据源上下文的 key 集合中，用于切换时判断数据源是否有效
            DynamicDataSourceContextHolder.addDataSourceKeys(dataSources.keySet());
        }
    }

（三）DynamicDataSourceContextHolder
---------------------------------

为了线程安全，我们要把lookupKey放入ThreadLocal里面，因此我们写了一个DynamicDataSourceContextHolder来切换数据源，就是改变当前线程保存的lookupKey，上面DynamicDataSource.determineCurrentLookupKey从当前线程取出即可，代码如下：

    /**
     * （切换数据源必须在调用service之前进行，也就是开启事务之前）
     * 动态数据源上下文
     * @author guomh
     * @date 2019/11/06
     */
    public class DynamicDataSourceContextHolder {
        private static final ThreadLocal<String> contextHolder = new ThreadLocal<String>() {
            /**
             * 将 master 数据源的 key作为默认数据源的 key
             */
            @Override
            protected String initialValue() {
                return "master";
            }
        };
        /**
         * 数据源的 key集合，用于切换时判断数据源是否存在
         */
        public static List<Object> dataSourceKeys = new ArrayList<>();
        /**
         * 切换数据源
         * @param key
         */
        public static void setDataSourceKey(String key) {
            contextHolder.set(key);
        }
        /**
         * 获取数据源
         * @return
         */
        public static String getDataSourceKey() {
            return contextHolder.get();
        }
        /**
         * 重置数据源
         */
        public static void clearDataSourceKey() {
            contextHolder.remove();
        }
        /**
         * 判断是否包含数据源
         * @param key 数据源key
         * @return
         */
        public static boolean containDataSourceKey(String key) {
            return dataSourceKeys.contains(key);
        }
        /**
         * 添加数据源keys
         * @param keys
         * @return
         */
        public static boolean addDataSourceKeys(Collection<? extends Object> keys) {
            return dataSourceKeys.addAll(keys);
        }
    }

（四）初始化数据源
---------

### 1\. tenant\_info表

以上配置好了，就差配置数据源了，为了便于维护数据源，我们可以有一个主数据源，里面建一张表来维护租户的数据源，这表可以根据自己需求建立，粘一下我的表结构

    
    CREATE TABLE `tenant_info`  (
      `id` bigint(20) NOT NULL AUTO_INCREMENT,
      `TENANT_ID` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '租户id',
      `TENANT_NAME` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '租户名称',
      `DATASOURCE_URL` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '数据源url',
      `DATASOURCE_USERNAME` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '数据源用户名',
      `DATASOURCE_PASSWORD` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '数据源密码',
      `DATASOURCE_DRIVER` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '数据源驱动',
      `SYSTEM_ACCOUNT` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '系统账号',
      `SYSTEM_PASSWORD` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '账号密码',
      `SYSTEM_PROJECT` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT '系统PROJECT',
      `STATUS` tinyint(1) NULL DEFAULT NULL COMMENT '是否启用（1是0否）',
      `CREATE_TIME` datetime(0) NULL DEFAULT NULL COMMENT '创建时间',
      `UPDATE_TIME` datetime(0) NULL DEFAULT NULL COMMENT '更新时间',
      PRIMARY KEY (`id`) USING BTREE
    ) ENGINE = InnoDB AUTO_INCREMENT = 4 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
    
    SET FOREIGN_KEY_CHECKS = 1;

### 2\. 配置动态数据源生效、默认主数据源

看下mybatis的配置如下

    /**
     * @Author: guomh
     * @Date: 2019/11/06
     * @Description: mybatis配置*
     */
    @EnableTransactionManagement
    @Configuration
    @MapperScan({"com.sino.teamwork.base.dao","com.sino.teamwork.*.*.mapper"})
    public class MybatisPlusConfig {
        @Bean("master")
        @Primary
        @ConfigurationProperties(prefix = "spring.datasource")
        public DataSource master() {
            return DataSourceBuilder.create().build();
        }
        @Bean("dynamicDataSource")
        public DataSource dynamicDataSource() {
            DynamicDataSource dynamicDataSource = new DynamicDataSource();
            Map<Object, Object> dataSourceMap = new HashMap<>();
            dataSourceMap.put("master", master());
            // 将 master 数据源作为默认指定的数据源
            dynamicDataSource.setDefaultDataSource(master());
            // 将 master 和 slave 数据源作为指定的数据源
            dynamicDataSource.setDataSources(dataSourceMap);
            return dynamicDataSource;
        }
        @Bean
        public MybatisSqlSessionFactoryBean sqlSessionFactoryBean() throws Exception {
            MybatisSqlSessionFactoryBean sessionFactory = new MybatisSqlSessionFactoryBean();
            /**
             * 重点，使分页插件生效
             */
            Interceptor[] plugins = new Interceptor[1];
            plugins[0] = paginationInterceptor();
            sessionFactory.setPlugins(plugins);
            //配置数据源，此处配置为关键配置，如果没有将 dynamicDataSource作为数据源则不能实现切换
            sessionFactory.setDataSource(dynamicDataSource());
            sessionFactory.setTypeAliasesPackage("com.sino.teamwork.*.*.entity,com.sino.teamwork.base.model");    // 扫描Model
            PathMatchingResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();
            sessionFactory.setMapperLocations(resolver.getResources("classpath*:mapper/*/*Mapper.xml"));    // 扫描映射文件
            return sessionFactory;
        }
        @Bean
        public PlatformTransactionManager transactionManager() {
            // 配置事务管理, 使用事务时在方法头部添加@Transactional注解即可
            return new DataSourceTransactionManager(dynamicDataSource());
        }
        /**
         * 加载分页插件
         * @return
         */
        @Bean
        public PaginationInterceptor paginationInterceptor() {
            PaginationInterceptor paginationInterceptor = new PaginationInterceptor();
    
            List<ISqlParser> sqlParserList = new ArrayList<>();
            // 攻击 SQL 阻断解析器、加入解析链
            sqlParserList.add(new BlockAttackSqlParser());
            paginationInterceptor.setSqlParserList(sqlParserList);
            return paginationInterceptor;
        }
    }

可以看到有如下配置：

*   **配置了主数据源叫master，主数据源放在spring配置文件里**
*   **配置动态数据源，并将主数据源加入动态数据源中，设为默认数据源**
*   **配置sqlSessionfactoryBean，并将动态数据源注入，sessionFactory.setDataSource(dynamicDataSource());**
*   **配置事务管理器，并将动态数据源注入new DataSourceTransactionManager(dynamicDataSource())；**
*   注意事项：
*   此处还有一点容易出错，就是分页问题，因为之前按spring默认配置，是不用在此配置数据源跟sqlSessionFactoryBean，配置了分页插件后，spring默认给你注入到了sqlSessionFactoryBean，但是此处因我们自己配置了sqlSessionFactoryBean，所以要自己手动注入，不然分页会无效，如下

    /**
    * 重点，使分页插件生效
    */
    Interceptor[] plugins = new Interceptor[1];
    plugins[0] = paginationInterceptor();
    sessionFactory.setPlugins(plugins);

还有一点要配置的，就是去掉springboot默认自动配置数据源

@SpringBootApplication(exclude = DataSourceAutoConfiguration.class})

    @EnableGlobalMethodSecurity(prePostEnabled = true,securedEnabled = true)
    @EnableScheduling
    @SpringBootApplication(exclude = DataSourceAutoConfiguration.class})
    public class TeamworkApplication {
    
        public static void main(String[] args) {
            SpringApplication.run(TeamworkApplication.class, args);
        }
    
    }
    

### 3\. 初始化加载租户的数据源

我们写一个类来初始化加载所有租户的数据源，代码也很简单，就是查询主数据源的数据库，查出所有租户的数据源信息，添加到动态数据源中（此处也可以加上把动态数据源交托spring管理）

    /**
     * 初始化动态数据源
     * @author guomh
     * @date 2019/11/06
     */
    @Slf4j
    @Configuration
    public class DynamicDataSourceInit {
    
        @Autowired
        private ITenantInfoService tenantInfoService;
        
        @PostConstruct
        public void InitDataSource()  {
            log.info("=====初始化动态数据源=====");
            DynamicDataSource dynamicDataSource = (DynamicDataSource)ApplicationContextProvider.getBean("dynamicDataSource");
            HikariDataSource master = (HikariDataSource)ApplicationContextProvider.getBean("master");
            Map<Object, Object> dataSourceMap = new HashMap<>();
            dataSourceMap.put("master", master);
            
            List<TenantInfo> tenantList = tenantInfoService.InitTenantInfo();
            for (TenantInfo tenantInfo : tenantList) {
                log.info(tenantInfo.toString());
                HikariDataSource dataSource = new HikariDataSource();
                dataSource.setDriverClassName(tenantInfo.getDatasourceDriver());
                dataSource.setJdbcUrl(tenantInfo.getDatasourceUrl());
                dataSource.setUsername(tenantInfo.getDatasourceUsername());
                dataSource.setPassword(tenantInfo.getDatasourcePassword());
                dataSource.setDataSourceProperties(master.getDataSourceProperties());
                dataSourceMap.put(tenantInfo.getTenantId(), dataSource);
            }
            //设置数据源
            dynamicDataSource.setDataSources(dataSourceMap);
            /**
             * 必须执行此操作，才会重新初始化AbstractRoutingDataSource 中的 resolvedDataSources，也只有这样，动态切换才会起效
             */
            dynamicDataSource.afterPropertiesSet();
        }
    
    }
    

4\. DynamicDataSourceAspect
---------------------------

我们可以使用面向切面编程，自动切换数据源，我是在用户登录时，将用户的租户信息放入session，租户的ID就对应数据源的lookupKey

    @Slf4j
    @Aspect
    @Component
    @Order(1) // 请注意：这里order一定要小于tx:annotation-driven的order，即先执行DynamicDataSourceAspectAdvice切面，再执行事务切面，才能获取到最终的数据源
    @EnableAspectJAutoProxy(proxyTargetClass = true)
    public class DynamicDataSourceAspect {
     
        @Around("execution(* com.sino.teamwork.core.*.controller.*.*(..)) "
            + "or execution(* com.sino.teamwork.base.action.*.*(..))")
        public Object doAround(ProceedingJoinPoint jp) throws Throwable {
            ServletRequestAttributes sra = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
            HttpSession session= sra.getRequest().getSession(true);
            String tenantId = (String)session.getAttribute("tenantId");
            
            log.info("当前租户Id:{}", tenantId);
            DynamicDataSourceContextHolder.setDataSourceKey(tenantId);
            Object result = jp.proceed();
            DynamicDataSourceContextHolder.clearDataSourceKey();
            return result;
        }
    }

5\. 对上述方案的修改
---------------------------

Application启动类

    
    import com.ebiz.access.admin.config.DataSourceStartup;
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration;
    import org.springframework.context.annotation.ComponentScan;
    import org.springframework.context.annotation.ComponentScans;
    
    @SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
    @ComponentScans(value = {@ComponentScan("com.ebiz.cache"),@ComponentScan("com.ebiz.common.tracer")})
    public class Application {
    
        public static void main(String[] args) {
            SpringApplication springApplication = new SpringApplication(Application.class);
            //项目启动后执行这个监听
            springApplication.addListeners(new DataSourceStartup());
            springApplication.run(args);
    
        }
    }

DataSourceStartup类修改

        log.info("=====初始化动态数据源=====");
        DynamicDataSource dynamicDataSource = (DynamicDataSource) SpringUtils.getBean("dynamicDataSource");
        HikariDataSource master = (HikariDataSource) SpringUtils.getBean("master");
        Map<Object, Object> dataSourceMap = new HashMap<>();
        dataSourceMap.put("master", master);
        
SpringUtils工具类

    
    import lombok.Getter;
    import org.springframework.beans.BeansException;
    import org.springframework.context.ApplicationContext;
    import org.springframework.context.ApplicationContextAware;
    import org.springframework.stereotype.Component;
    
    @Component
    public final class SpringUtils implements ApplicationContextAware {
    
        @Getter
        private static ApplicationContext applicationContext;
    
        @Override
        public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
            if (SpringUtils.applicationContext == null) {
                SpringUtils.applicationContext = applicationContext;
            }
        }
    
        public static <T> T getBean(Class<T> clazz) {
            return SpringUtils.applicationContext.getBean(clazz);
        }
    
        public static Object getBean(String name) {
            return SpringUtils.applicationContext.getBean(name);
        }
    
        public static String getProperty(String key) {
            return SpringUtils.applicationContext.getEnvironment().getProperty(key);
    
        }
    }

=====
