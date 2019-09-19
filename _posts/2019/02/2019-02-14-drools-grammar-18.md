---
layout: post
title: 基于Spring + Drools6.4规则引擎代码实例
category: drools
tags: [drools]
---

## 简介

官网地址：[https://drools.org/](https://drools.org/)

### 关于Drools（官网简介直接Copy过来）

Drools is a Business Rules Management System (BRMS) solution.

It provides a core Business Rules Engine (BRE), a web authoring and rules management application (Drools Workbench) and an Eclipse IDE plugin for core development.

最近有个消费返现和后付费保险类型项目，要求根据不同规则进行消费返现及保险静默投保etc. ，为避免过多硬编码ifelse逻辑判断，影响程序可读性及削弱程序可扩展性，因此引入了Drools规则引擎。

至于规则引擎到底是啥，在这里就不赘述了，google一下，你就知道。

## Code实现

下面基于一个简单的Mock User Register模拟流程，简单介绍一下关于Spring+Drools集成实现流程。

### 需求说明

<code class="language-java">新用户规则：

1.系统Mock生成用户注册数据，并进入Drools规则引擎处理;

2.对于新注册用户设定用户锁定状态，并初始用户等级为3级，覆盖新用户标识为非；

非新用户规则：

1.设定用户非锁定状态，每次Mock Code执行，用户等级+1（规则比较随意）；</code>
### 实现过程

添加Drools Pom依赖

     <code class="language-xml"><pro..>
           <drools-version>6.4.0.Final</drools-version>
        </pro..>

        <dependency>
            <groupId>org.drools</groupId>
            <artifactId>drools-core</artifactId>
            <version>${drools-version}</version>
        </dependency>

        <dependency>
            <groupId>org.drools</groupId>
            <artifactId>drools-compiler</artifactId>
            <version>${drools-version}</version>
        </dependency></code>

User实体Bean定义

    <code class="language-java">@Getter
    @Setter
    @Access(AccessType.FIELD)
    @Accessors(chain = true)
    @MetaData(value = "登陆/认证用户信息" , comments = "monster-web / monster-web-admin模块使用统一的auth信息")
    @Entity
    @Table(name = "auth_MsUser" , uniqueConstraints = @UniqueConstraint(columnNames ={"authUid","authType"}))
    @Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
    public class User extends BaseNativeEntity {

    ……
    ……

    @MetaData(value="用户全局唯一标识" , comments = "同时作为系统用户密码Salt值,Salt形式 [authUuid]")
    @Column(length = 64 , nullable = false)
    private String authUuid;

    @MetaData(value="authUid,登陆账号")
    @Column(length = 128 , nullable = false)
    private String authUid;

    @MetaData(value="密码",comments = "MD5加密串,格式:MD5({Salt}+sourcePassword)")
    @Column(length = 128 , nullable = false)
    private String password;

    @MetaData(value="用户类型")
    @Column(length = 32 , nullable = false)
    @Enumerated(EnumType.STRING)
    private AuthTypeEnum authType;

    @MetaData(value = "身份证号码")
    @Column(length = 32)
    private String idCardNo;

    @MetaData(value="性别")
    @Column(length = 32)
    @Enumerated(EnumType.STRING)
    private GenderEnum gender;

    @MetaData(value = "REST访问Token")
    @Column
    private String accessToken;

    @MetaData(value = "访问Token过期时间")
    @DateTimeFormat(pattern = DateUtils.DEFAULT_TIME_FORMAT)
    @JsonFormat(pattern = DateUtils.DEFAULT_TIME_FORMAT)
    private Date accessTokenExpireTime;

    @MetaData(value = "是否新用户",tooltips = "用于新用户相关业务规则")
    @Column
    private Boolean isNew = Boolean.TRUE;

    @MetaData(value = "账户未锁定标志", tooltips = "账号锁定后无法登录")
    @Column
    private Boolean accountNonLocked = Boolean.TRUE;

    ……
    ……     

}</code>

Drools相关Bean定义

提供Drools资源辅助类

    <code class="language-java">package org.monster.drools.technorage;
    
    import org.kie.api.io.ResourceType;
    
    @setter
    @getter
    public class DroolsResource {

	private String path;
	private ResourcePathType pathType;
	private ResourceType type;
	private String username = null;
	private String password = null;

	/**
	 * 
	 * @param path
	 *            The path to this resource.
	 * @param pathType
	 *            The type of path (FILE, URL, etc).
	 * @param type
	 *            The type of resource (DRL, Binary package, DSL, etc)
	 */
	public DroolsResource(String path, ResourcePathType pathType,
			ResourceType type) {
		this.path = path;
		this.pathType = pathType;
		this.type = type;
	}

	/**
	 * 
	 * @param path
	 *            The path to this resource.
	 * @param pathType
	 *            The type of path (FILE, URL, etc).
	 * @param type
	 *            The type of resource (DRL, Binary package, DSL, etc)
	 * @param username
	 *            The user name for connecting to the resource.
	 * @param password
	 *            The password for connecting to the resource.
	 */
	public DroolsResource(String path, ResourcePathType pathType,
			ResourceType type, String username, String password) {
		this.path = path;
		this.pathType = pathType;
		this.type = type;
		this.username = username;
		this.password = password;
	}

}</code> 

定义本地接口，并实现Drools提供的KieServices与KieContainer、KieSession接口

    <code class="language-java">import org.kie.api.KieServices;
    
    public interface KieServicesBean extends KieServices {
    
    }</code><code class="language-java">package org.monster.drools.technorage.spring;
    
    import org.kie.api.runtime.KieContainer;
    
    public interface KieContainerBean extends KieContainer {
    
    }</code><code class="language-java">package org.monster.drools.technorage.spring;
    
    import org.kie.api.runtime.KieSession;
    
    public interface KieSessionBean extends KieSession {
    
    }</code>

及其分别实现类

 <code class="language-java">public class DefaultKieServicesBean implements KieServicesBean {

    private static Logger log = LoggerFactory.getLogger(DefaultKieServicesBean.class);

    private DroolsResource[] resources;

    private KieServices kieServices;
    private KieFileSystem kfs; 

    public DefaultKieServicesBean(DroolsResource[] resources) throws KieBuildException {
        log.info("Initialising KnowledgeEnvironment with resources: " + this.resources);
        this.resources = resources;
        createAndBuildKieServices(resources);
    }

    /**
     * Initialises the {@link org.kie.api.KieServices} by downloading the package from the
     * Guvnor REST interface, at the location defined in the URL.
     * 
     * @param url The URL of the package via the Guvnor REST API.
     * @throws KieBuildException 
     */
    public DefaultKieServicesBean(String url) throws KieBuildException {
        log.info("Initialising KnowledgeEnvironment with resources: " + this.resources);
        this.resources = new DroolsResource[] { 
                new DroolsResource(url,
                        ResourcePathType.URL,
                        ResourceType.PKG
            )};
        createAndBuildKieServices(resources);
    }

    /**
     * 根据提供的URL生成{@link org.kie.api.KieServices}
     * 
     * @param url The URL of the package via the Guvnor REST API.
     * @param username The Guvnor user name.
     * @param password The Guvnor password.
     * @throws KieBuildException 
     */
    public DefaultKieServicesBean(String url, String username, String password) throws KieBuildException {
        this.resources = new DroolsResource[] { 
                new DroolsResource(url, 
                        ResourcePathType.URL, 
                        ResourceType.PKG, 
                        username, 
                        password
        )};
        createAndBuildKieServices(resources);
    }

    /**
     * 根据提供的DroolsResource创建 {@link org.kie.api.KieServices} 
     * 
     * @param resources
     *            An array of {@link DroolsResource} indicating where the
     *            various resources should be loaded from. These could be
     *            classpath, file or URL resources.
     * @return A new {@link org.kie.api.runtime.KieContainer}.
     * @throws KieBuildException 
     */
    private void createAndBuildKieServices(DroolsResource[] resources) throws KieBuildException {
        this.kieServices = KieServices.Factory.get();
        this.kfs = newKieFileSystem();

        for (DroolsResource resource : resources) {
            log.info("Resource: " + resource.getType() + ", path type="
                    + resource.getPathType() + ", path=" + resource.getPath());
            switch (resource.getPathType()) {
            case CLASSPATH:
                this.kfs.write(ResourceFactory.newClassPathResource(resource.getPath()));
                break;
            case FILE:
                this.kfs.write(ResourceFactory.newFileResource(resource.getPath()));
                break;
            case URL:
                UrlResource urlResource = (UrlResource) ResourceFactory
                        .newUrlResource(resource.getPath());

                if (resource.getUsername() != null) {
                    log.info("Setting authentication for: " + resource.getUsername());
                    urlResource.setBasicAuthentication("enabled");
                    urlResource.setUsername(resource.getUsername());
                    urlResource.setPassword(resource.getPassword());
                }

                this.kfs.write(urlResource);

                break;
            default:
                throw new IllegalArgumentException(
                        "Unable to build this resource path type.");
            }
        }

        KieBuilder kieBuilder = kieServices.newKieBuilder(kfs).buildAll();

        if (kieBuilder.getResults().hasMessages(Level.ERROR)) {
            List<Message> errors = kieBuilder.getResults().getMessages(Level.ERROR);
            StringBuilder sb = new StringBuilder("Errors:");
            for (Message msg : errors) {
                sb.append("\n  " + prettyBuildMessage(msg));
            }
            throw new KieBuildException(sb.toString());
        }

        log.info("KieServices built: " + toString());
    }

    private static String prettyBuildMessage(Message msg) {
        return "Message: {"
            + "id="+ msg.getId()
            + ", level=" + msg.getLevel()
            + ", path=" + msg.getPath()
            + ", line=" + msg.getLine()
            + ", column=" + msg.getColumn()
            + ", text=\"" + msg.getText() + "\""
            + "}";
    }

    //省略相关实现方法

}</code><code class="language-java">public class DefaultKieContainerBean implements KieContainerBean {

    private KieServicesBean kieServices;
    private KieContainer kieContainer;

    public DefaultKieContainerBean(KieServicesBean kieServicesBean) {
        this.kieServices = kieServicesBean;
        this.kieContainer = this.kieServices.newKieContainer(kieServices.getRepository().getDefaultReleaseId());
    }

        /**
            ……
            ……

          */

}</code><code class="language-java">public class DefaultKieSessionBean implements KieSessionBean {

    private static Logger log = LoggerFactory.getLogger(DefaultKieSessionBean.class);

    private KieSession kieSession;

    public DefaultKieSessionBean(KieServicesBean kieServices, KieContainerBean kieContainer) {
        this(kieServices, kieContainer, null);
    }

    public DefaultKieSessionBean(KieServicesBean kieServices, KieContainerBean kieContainer, Properties droolsProperties) {
        log.info("Initialising session...");

        KieSessionConfiguration conf;
        if (droolsProperties == null) {
            conf = SessionConfiguration.getDefaultInstance();
        } else {
            conf = SessionConfiguration.newInstance(droolsProperties);//new SessionConfiguration(droolsProperties);
        }
        this.kieSession = kieContainer.newKieSession(conf);
    }

    public void addEventListener(RuleRuntimeEventListener listener) {
        kieSession.addEventListener(listener);
    }

    public void addEventListener(ProcessEventListener listener) {
        kieSession.addEventListener(listener);
    }

    public ProcessInstance startProcess(String processId) {
        return kieSession.startProcess(processId);
    }

    public void removeEventListener(RuleRuntimeEventListener listener) {
        kieSession.removeEventListener(listener);
    }

    public int fireAllRules() {
        return kieSession.fireAllRules();
    }

    public void removeEventListener(ProcessEventListener listener) {
        kieSession.removeEventListener(listener);
    }

    public <T extends SessionClock> T getSessionClock() {
        return kieSession.getSessionClock();
    }

    public int fireAllRules(int max) {
        return kieSession.fireAllRules(max);
    }

    public Collection<RuleRuntimeEventListener> getWorkingMemoryEventListeners() {
        return kieSession.getRuleRuntimeEventListeners();
    }

    public Collection<ProcessEventListener> getProcessEventListeners() {
        return kieSession.getProcessEventListeners();
    }

    public void setGlobal(String identifier, Object value) {
        kieSession.setGlobal(identifier, value);
    }

    public void halt() {
        kieSession.halt();
    }

    public ProcessInstance startProcess(String processId,
            Map<String, Object> parameters) {
        return kieSession.startProcess(processId, parameters);
    }

    public void addEventListener(AgendaEventListener listener) {
        kieSession.addEventListener(listener);
    }

    public Object getGlobal(String identifier) {
        return kieSession.getGlobal(identifier);
    }

    public Globals getGlobals() {
        return kieSession.getGlobals();
    }

    public Calendars getCalendars() {
        return kieSession.getCalendars();
    }

    public void removeEventListener(AgendaEventListener listener) {
        kieSession.removeEventListener(listener);
    }

    public Environment getEnvironment() {
        return kieSession.getEnvironment();
    }

    public KieBase getKieBase() {
        return kieSession.getKieBase();
    }

    public int fireAllRules(AgendaFilter agendaFilter) {
        return kieSession.fireAllRules(agendaFilter);
    }

    public void registerChannel(String name, Channel channel) {
        kieSession.registerChannel(name, channel);
    }

    public Collection<AgendaEventListener> getAgendaEventListeners() {
        return kieSession.getAgendaEventListeners();
    }

    public String getEntryPointId() {
        return kieSession.getEntryPointId();
    }

    public void unregisterChannel(String name) {
        kieSession.unregisterChannel(name);
    }

    public Map<String, Channel> getChannels() {
        return kieSession.getChannels();
    }

    public Agenda getAgenda() {
        return kieSession.getAgenda();
    }

    public int fireAllRules(AgendaFilter agendaFilter, int max) {
        return kieSession.fireAllRules(agendaFilter, max);
    }

    public FactHandle insert(Object object) {
        return kieSession.insert(object);
    }

    public KieSessionConfiguration getSessionConfiguration() {
        return kieSession.getSessionConfiguration();
    }

	@Override
	public EntryPoint getEntryPoint(String name) {
		return kieSession.getEntryPoint(name);
	}
    public ProcessInstance createProcessInstance(String processId,
            Map<String, Object> parameters) {
        return kieSession.createProcessInstance(processId, parameters);
    }

    @Deprecated
    public void retract(FactHandle handle) {
        kieSession.retract(handle);
    }

    public Collection<? extends EntryPoint> getEntryPoints() {
        return kieSession.getEntryPoints();
    }

    public void fireUntilHalt() {
        kieSession.fireUntilHalt();
    }

    public <T> T execute(Command<T> command) {
        return kieSession.execute(command);
    }

    public void delete(FactHandle handle) {
        kieSession.delete(handle);
    }

    @Override
    public void delete(FactHandle handle, FactHandle.State fhState) {

    }

    public QueryResults getQueryResults(String query, Object... arguments) {
        return kieSession.getQueryResults(query, arguments);
    }

    public void update(FactHandle handle, Object object) {
        kieSession.update(handle, object);
    }

    public void fireUntilHalt(AgendaFilter agendaFilter) {
        kieSession.fireUntilHalt(agendaFilter);
    }

    public FactHandle getFactHandle(Object object) {
        return kieSession.getFactHandle(object);
    }

    public LiveQuery openLiveQuery(String query, Object[] arguments,
            ViewChangedEventListener listener) {
        return kieSession.openLiveQuery(query, arguments, listener);
    }

    public ProcessInstance startProcessInstance(long processInstanceId) {
        return kieSession.startProcessInstance(processInstanceId);
    }

    public Object getObject(FactHandle factHandle) {
        return kieSession.getObject(factHandle);
    }

    public int getId() {
        return kieSession.getId();
    }

    @Override
    public long getIdentifier() {
        return 0;
    }

    public void signalEvent(String type, Object event) {
        kieSession.signalEvent(type, event);
    }

    public void dispose() {
        kieSession.dispose();
    }

    public Collection<? extends Object> getObjects() {
        return kieSession.getObjects();
    }

    public void destroy() {
        kieSession.destroy();
    }

    public void signalEvent(String type, Object event, long processInstanceId) {
        kieSession.signalEvent(type, event, processInstanceId);
    }

    public Collection<? extends Object> getObjects(ObjectFilter filter) {
        return kieSession.getObjects(filter);
    }

    public <T extends FactHandle> Collection<T> getFactHandles() {
        return kieSession.getFactHandles();
    }

    public <T extends FactHandle> Collection<T> getFactHandles(
            ObjectFilter filter) {
        return kieSession.getFactHandles(filter);
    }

    public Collection<ProcessInstance> getProcessInstances() {
        return kieSession.getProcessInstances();
    }

    public long getFactCount() {
        return kieSession.getFactCount();
    }

    public ProcessInstance getProcessInstance(long processInstanceId) {
        return kieSession.getProcessInstance(processInstanceId);
    }

    public ProcessInstance getProcessInstance(long processInstanceId,
            boolean readonly) {
        return kieSession.getProcessInstance(processInstanceId, readonly);
    }

    public void abortProcessInstance(long processInstanceId) {
        kieSession.abortProcessInstance(processInstanceId);
    }

    public WorkItemManager getWorkItemManager() {
        return kieSession.getWorkItemManager();
    }

	@Override
	public KieRuntimeLogger getLogger() {
		return kieSession.getLogger();
	}

	@Override
	public Collection<RuleRuntimeEventListener> getRuleRuntimeEventListeners() {
		return kieSession.getRuleRuntimeEventListeners();
	}

}</code>

集成Spring,交由spring托管

<code class="language-java">@Configuration
//@Profile("drools")
public class BaseKieConfig {

    @Bean(name="kieServices")
    public KieServicesBean kieServices() throws KieBuildException {

        DroolsResource [] droolsResources = new DroolsResource[]{
                new DroolsResource("rules/user-level.drl", ResourcePathType.CLASSPATH , ResourceType.DRL)
        };

        KieServicesBean kieServicesBean = new DefaultKieServicesBean(droolsResources);

        return kieServicesBean;
    }

    @Bean(name="kieContainer")
    public KieContainerBean kieContainer(KieServicesBean kieServices){
        KieContainerBean kieContainer = new DefaultKieContainerBean(kieServices);
        return kieContainer;
    }

}</code>

定义相关业务接口

<code class="language-java">public interface UserRuleService<T> {

    List<T> userRegister( List<User> users );

    List<T> remUsers( List<User> users );

    List<T> checkUserStatus();

}</code><code class="language-java">@Service
@Scope(value = ConfigurableBeanFactory.SCOPE_SINGLETON , proxyMode = ScopedProxyMode.INTERFACES)
public class UserRuleServiceImpl implements UserRuleService<User> ,Serializable {

    private Logger logger = LoggerFactory.getLogger(UserRuleServiceImpl.class);

    public KieSessionBean kieSession;

    private TrackingAgendaEventListener agendaEventListener;

    private TrackingWorkingMemoryEventListener workingMemoryEventListener;

    private Map<String , FactHandle> fact2User = Maps.newHashMap();
    private FactFinder<User> userFactFinder = new FactFinder<User>(User.class);

    @Autowired
    public UserRuleServiceImpl(@Qualifier("kieServices") KieServicesBean kieServices ,
                               @Qualifier("kieContainer") KieContainerBean kieContainer){

        System.setProperty("drools.negatable", "on");

        kieSession                 = new DefaultKieSessionBean(kieServices , kieContainer);

        agendaEventListener        = new TrackingAgendaEventListener();
        workingMemoryEventListener = new TrackingWorkingMemoryEventListener();

        kieSession.addEventListener(agendaEventListener);
        kieSession.addEventListener(workingMemoryEventListener);

    }

    @Override
    public List<User> userRegister(List<User> users) {

        for ( User user : users )
        {
            if(logger.isDebugEnabled()){
                logger.debug("执行用户注册规则模板,对应用户:{}",user.toString());
            }
            if(!fact2User.containsKey(user.getId())){
                FactHandle userFireHandle = kieSession.insert(user);
                fact2User.put(String.valueOf(user.getId()) , userFireHandle);
            }
        }
        kieSession.fireAllRules();
        List<User> results = userFactFinder.findFacts(kieSession);
        pause();
        kieSession.dispose();
        return results;
    }

    @Override
    public List<User> remUsers(List<User> users) {

        for( User user : users )
        {
            if(fact2User.containsKey(String.valueOf(user.getId())))
            {
                kieSession.delete(fact2User.get(String.valueOf(user.getId())));
                fact2User.remove(user);
            }
        }

        kieSession.fireAllRules();
        List<User> result = userFactFinder.findFacts(kieSession);
        return result;
    }

    @Override
    public List<User> checkUserStatus() {
        return null;
    }

    public static void pause() {
        System.out.println( "Pressure enter to continue" );
        Scanner keyboard = new Scanner(System.in);
        keyboard.nextLine();
    }
}</code>

Mock代码

     <code class="language-java">……
     ……
     …… 
    
    int userCnt = countTable(User.class);
            for( int i = userCnt ; (i < INIT_USER_CNT && i<userCnt+5) ; i ++ )
            {
                String salt      =     UidUtils.UID();
                String format    =     String.format("%06d" , i);
                User user = MockUtils.buildMockEntity(User.class);
                user.setAuthUid("19999"+format);
                user.setRealName("用户" + format);
                user.setNickName("昵称" + format);
                user.setAuthUuid(salt);
                user.setAuthType(User.AuthTypeEnum.SYS);
                user.setPassword(passwordService.encryptPassword("123456" , salt));
                user.setIdCardNo("110010" + String.format("%012d" , i));
                user.setAccessToken(StringUtils.EMPTY);
    
                //user.setAccessToken(salt);
                users.add(user);
            }
    
            if(CollectionUtils.isEmpty(users))
            {
                users = userService.findAll();
            }
    
            //规则引擎服务处理
            users = (List<User>) userRuleService.userRegister(users);
            //存储用户信息
            userService.save(users);</code>
    
    定义规则文件（user-register.drl）
    
    <code class="language-xml">//User-Level规则
    package rules
    import org.monster.core.auth.entity.User;
    
    rule RegistNewUserRule
    //ruleflow-group "Basic-User-Group"
    when  $user:User(isNew == Boolean.TRUE)
    then
         //1.设置解除锁定状态
         $user.setAccountNonLocked(false);
         //2.用户注册送积分
         $user.setUserLevel(3);
         $user.setIsNew(Boolean.FALSE);
         //3.根据性别
         //System.out.println("++++++++++ Is New Flg True , [User:" + $user.toString() + "]");
    end
    
    rule RegistNewUserRuleNotNew
    //ruleflow-group "Basic-User-Group"
    when $user:User(isNew == false)
    then
         //1.设置锁定状态
             $user.setAccountNonLocked(true);
             //2.老用户统一设定每次启动增加一级
             $user.setUserLevel($user.getUserLevel()+1);
             //System.out.println("---------- Is New Flg False, [User:" + $user.toString() +"]");
    end</code>

————————————————————————————————————————————————————————————————————————————————

## Drools中，其三者关系

### 1.KieServices

1.1 容器通过ClassPathResource获取KieFileSystem

    <code class="language-java">//获取KieFileSystem
    this.kfs.write(ResourceFactory.newClassPathResource(resource.getPath()));</code>

1.2.并通过Factory.get()方法获取Service实例，进行相关装载校验

    <code class="language-java">//获取kieServices实例
    this.kieServices = KieServices.Factory.get();
    
    ……
    
    KieBuilder kieBuilder = kieServices.newKieBuilder(kfs).buildAll();
    
            if (kieBuilder.getResults().hasMessages(Level.ERROR)) {
                List<Message> errors = kieBuilder.getResults().getMessages(Level.ERROR);
                StringBuilder sb = new StringBuilder("Errors:");
                for (Message msg : errors) {
                    sb.append("\n  " + prettyBuildMessage(msg));
                }
                throw new KieBuildException(sb.toString());
            }</code> 
### 2.KieContainer

通过KieServices实例，构造KieContainer

<code class="language-java">this.kieContainer = this.kieServices.newKieContainer(kieServices.getRepository().getDefaultReleaseId());</code>
### 3.KieSession

通过KieContainer及相关KieSessionConfig获取current working Session对象

     <code class="language-java">KieSessionConfiguration conf;
            if (droolsProperties == null) {
                conf = SessionConfiguration.getDefaultInstance();
            } else {
                conf = SessionConfiguration.newInstance(droolsProperties);//new SessionConfiguration(droolsProperties);
            }
            this.kieSession = kieContainer.newKieSession(conf);</code>


**时间匆忙写的比较乱，基本贴的代码。有空整理**
