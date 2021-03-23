分布式事务：BASE最终一致性

选型：

	XA:  准备+提交 2阶段；

		强一致性，读写隔离； 

		性能低，死锁，同步操作阻塞导致吞吐量低；提交阶段，某个微服务提交成功挂掉，协调器会认为挂掉，执行回滚，导致数据不一致！

	TCC：补偿事务-每一个操作需要写一个回滚的操作；try+confirm/cancel;

		自定义数据库操作粒度，降低锁冲突，提高吞吐量		

		业务入侵强，实现难度大，接口需要保证幂等性，不同的失败原因需要不同的回滚策略

	Saga：自动实现回滚

		Seata,Camel

		支持长事务，无锁，无阻塞；

		难debug, 没有读隔离; 需要监控，回滚失败可以人工介入

Producer

	向MQ发送预备消息half message【RMQ_SYS_TRANS_HALF_TOPIC,定时任务】

	提交事务execute

	向MQ发送 提交消息，二次确认	【从half中恢复真正的topic和queue，&写入】

		- 必须收到事务提交成功，才走这一步

		- 提交失败，则回滚预备消息，消息被删除【顺序写文件：OP消息-特定的topic中，Half消息的索引】

Rocketmq

	收到提交消息，变为可消费消息。

		- 出错没关系，会回查确认状态，15个等级【TRANS_CHECK_MAXTIME_TOPIC】

	失败自动重发

Consumer

	ack机制，先消费后提交，需要保证幂等性

	事务提交，返回结果

异步解耦高性能；事务消息；

无读写隔离，可能读到事务中间状态的数据

事务消息需要同步刷盘

    @MqTransactional
    @MqIdempotent
    事务日志表
    幂等日志表
    ---事务id,消息id,消息topic,消息tag,消息报文，来源微服务
    多方事务日志表--远端返回
    ---事务id,消息id,消息topic,消息tag,事务状态，事务发起方原始消息message,来源类名，来源方法名，来源参数，源微服务，目的地微服务，更新时间
    --日志表需要定期清理
    --推荐在xml中配置事务
    集群平摊，广播全量

---

性能：

	多快好省：吞吐量、服务延迟、拓展性、资源效率

	三部曲：抓重点（28定律），找方向，定量分析

	分析：性能瓶颈、内存溢出OOM、CPU占比过高的代码、FGC

	根据监控指标，日志分析，先分段定位，再定点分析，最后找到瓶颈点

	监控指标：CPU<80% 容器内存<75% Tomcat线程池<90% FGC 一小时一次，GC时长：毫秒级 数据库连接池 90%



	通过调用链查看到hmset时间很长，分析redis操作时长原因：

		并发不行，大对象，连接数不够

	查看到循环调用的操作，需要计算共消费时间，如果时间过大，需要优化

	慢sql 发现获取sequence花了3秒，改用雪花算法生成ID

		配置mysql启动慢sql：

			log_slow_queries开启，

			long_query_time配置时间，

			slow_query_log_file配置日志路径

			log_queries_not_using_indexes  //分析时打开

		mysqldumpslow -t 10  /data/mysql/mysql-slow.log  #显示出慢查询日志中最慢的10条sql 

		pt-query-digest 非自动，需安装

	OOM：

		top                    查看高进程

		top -H -p PID   查看高线程

		printf "%x\n" PID   转为16进制

		jstack -l -F 进程号 | grep 16进制号 > filepath

		追踪gc： jstat -gc PID 时间打印间隔 采样数

			YGCT FGUT GCT...

		jvm内存分配	jmap -heap pid

    NewRatio=2[N:O=1:2],SurvivorRatio=8[1:1:8],MaxMetaSpaceSize=512M
    Eden 390M
    From 100M
    TO 100M
    Old 485M
    -Xmx2475m -XX:MaxMetaspaceSoze=512m -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+UseGCLogFileRotation -XX:+GCLogFileSize=1G -XX:+NumberOfGCLogFiles=10 -XX:HeapDumpPath=/heapdump -XX:+HeapDumpOnOutMemoryError -XX:OnOutOfMemoryError=/script/jstack.sh -XX:MaxTenuringThreshold=9[经过多少次MinorGC进入到老年代|15]

	prometheus自动监控报警系统--监控容器

	性能优化策略：

		时空转换、并发/异步操作，预先/延后处理，缓存/批量合并，算法设计和数据结构

	sql优化：

		索引，批量查询，减少事务提交时间，加默认条件查询-缩小查询范围，

	高性能：

		负载均衡Nginx[ELB/F5], redis缓存，本地缓存，cache aside模式[先操作数据库再操作缓存]，读多写少-读写分离，读少写多-分库分表，MQ削峰异步解耦



springmvc:

	@SessionAttributes 是将model设置到session中去。

	@SessionAttribute 是从session获取之前设置到session中的数据。

	默认单例，线程不安全，所以不要在控制器中写字段

	DispatchServlet接收到请求，并根据url从Handlermapping中得到handler,并构造出HandlerExecutionChain执行链，通过HandlerAdapter调用handler,先进入链，最后进入到handler. 先进后出，得到ModeAndView对象，通过ViewResolver得到View,并且调用render渲染，将结果返回。

	手写DispatchServlet：

		1.加载配置文件2.扫描配置文件&实例化对象3.属性填充4.初始化handlerMapping

	@RquestMapping

		value,method,consume,producer,params,headers

	@RestController(@Controller+@RequestBody)@ResponseBody@PathVariable@RequestParam

	注解本质是一个继承了Annotation的特殊接口，具体实现类是Java运行时生成的动态代理类

	&Struts2:

		前端控制器：DispatcherServlet&StrutsPreparedAndExecutorFilter

		单例&多例 都线程不安全

		返回不一样：ModeAndView/String ONGL存取数据 采用值栈存储请求和响应

	解决中文乱码：

		POST：CharacterEncodingFilter

		GET: Tomcat配置文件ConnectorURIEncoding="utf8" 参数指定编码new String(request.getParamter("").getBytes("ISO8859-1"),"utf8")



Spring：

	使用的设计模式：

		1.工厂模式-BeanFactory 2.代理模式-JDK代理&CGLIB代理 3.模板方法-RestTemplate 4.观察者模式-ApplicationListener 5.单例模式

	IOC-控制反转，bean的创建，管理交由Spring容器；

		作用：管理对象的生成和对象依赖关系的维护；解耦；托管类的产生过程；

		功能：1.依赖注入；2.依赖检查；3.自动装配；4.支持集合；5.初始化方法和销毁方法；6.回调

		优点：解耦，容易测试，支持饿汉式初始化和懒加载

		原理：工厂+反射

	BeanFactory&ApplicationContext:

		创建方式：前者编码，后者声明式创建

		注册方式：前者需要手动注册bean，后者自动注册bean

		加载方式：前者是延迟加载/在使用的时候，调用getObject()才生成bean，后者是在容器启动的时候加载bean

		功能：前者包含-bean的创建，管理bean的加载&实例化，控制bean的生命周期，依赖关系；后者包含所有前者的功能，额外还能实现，国际化，文件读取，上下文，注册事件等。		

	依赖注入：DI

		DI 是IOC实现的方式，通过构造函数+setter函数进行依赖注入，构造函数适用于多数据，setter函数使用少量数据注入。

	Spring配置元数据方式

		xml配置，注解配置，javaconfig配置

	Spring bean的作用域

		singleton	prototype	session	request	application

	单例bean是否是线程安全，如何处理并发问题

		不是

		状态bean 不是线程安全的

		有状态bean是线程安全的

		》ThreadLocal-空间换时间；synchronized锁-时间换空间（不支持）；prototype（性能不行）

	bean的生命周期

		实例化	属性注入		BeanNameAware BeanFactoryAware	ApplicationContextAware		

		BeanPostProcessor的before初始化方法		InitializationBean-afterPropertiesSet()   自定义初始化方法@PostConstruct/init-method		BeanPostProcessor的after初始化方法	

		DestoryBean-destory() 自定义销毁方法@PreDestory/destory-method

	spring中注入一个java集合：

    <bean>
        <property name="">
            <list></list>
            <set></set>
            <map></map>
            <props></props>
            <value></value> -- 空字符串        
        </property>
    </bean>

	自动装配Bean的方式

		no 需要自己制定ref

		byName

		byType

		byConstruct  -- 使用byType

		autodect 自动检测

	@Autowired原理

		容器启动，注册了AutowiredAnnotationBeanPostProcessor后置处理器，遇到@Autowired,@Resource,@Inject时去容器中自动查找bean  

		类型，名字，报错（required=false避免报错）

		@Qualifier(“指定名字”) --可以搭配使用

		@Resource 	--byName

		// 开启自动装配：context:annotation-config / 配置文件AutowiredAnnotationBeanPostProcessor

	Spring注入方式

		构造器	setter方法

	@Autowired@Qualifier@Resource(按名字，没有按类型)@Required

	JDBCTemplate

	事务管理类型

		编程式 编码 更灵活 更复杂

		声明式 注解/配置 更方便

		》原理：数据库支持事务，spring本身是无法提供事务功能

	7大事务传播行为：

		required，requires_new，supports, not_supported, never, mandatory, 

		nested(内嵌事务：savepoint 外部事务回滚会导致内嵌事务回滚，但是内嵌事务回滚-外部事务已经执行的代码不会回滚-即部分回滚,回滚到子事务创建之前的savepoint 外部事务提交，内嵌事务才能提交)

	4大隔离级别：

		RU	RC	RR	RS

		脏读	不可重复读-MVCC		幻读（插入修改删除导致） --范围锁解决

	AOP 横向抽取封装 降低重复代码，模块间的耦合 提升系统可维护性

		权限认证，日志，事务处理，多数据源，读写分离，Controller层的参数校验，信息过滤，页面转发，检测执行性能

		https://blog.csdn.net/xlgen157387/article/details/53930382

	AOP实现：Spring AOP & AspectJ AOP

		动态代理 不修改字节码，生成一个新的AOP对象，在特定的切点做增加处理

		静态代理 编译期间将Aspect织入到Java字节码

	JDK&CGLIB

		JDK代理：接口代理，InvocationHandler.invoke()

		CGLIB：类代理，通过继承的方式生成很多类重写方法，所以不能是final类

			proxy-target-class=true & 非接口

	AOP概念

		Aspect切面（切点和通知的结合）PointCut切点（一系列的接入点）PointJoint接入点Advice通知（切面的工作）weaving织入（切面应用到容器外部代理对象：编译时间AspectJ，类加载期AspectJ5，运行期SpringAOP）引介Introduction（给@DeclareParents对应的类增强方法-接口即实现类 类级别的增加，增加接口）

			织入：将切面与外部应用程序或类连接起来以创建通知对象的过程。--增加目标类的字节码，需要使用特殊的类加载器加载  通知/增加应到到目标对象的过程

	Advice执行顺序：

		around before - before - method - around after - after - after returing

		around before - before - method - around after - after - after throwing

	切点表达式：标识符（execution.....）、操作参数

		execution-方法; annotation-注解

	AOP应用：读写分离

	AOP原理：2层拦截器，外层是Spring核心控制流程，内层是AOP里的东西。

		代理的创建：创建代理工厂，并且在拦截器链的尾部增加一个拦截器用于目标方法的调用

		代理的调用：调用触发外层拦截器，执行拦截器链，最终调用目标方法。

			DynamicAdvisedInterceptor //为创建代理时，会指定callback

			ExposeInvocationInterceptor 4个增强Advice。先进后执行AspectJAfterThrowingAdvice...MethodBeforeAdviceInterceptor

		registerBeanPostProcessors注册AnnotationAwareAspectJAutoProxyCreator（InstantiationAwareBeanPostProcessor）到单例池

		finishBeanFactoryInitialization 生成代理对象

	@Target @Retention

	@Import

	多数据源：

	方式一：sqlSessionFactory, MapperScannerConfigurer, 配2套-datasource

			@ConfigurationProperties(Prefix="")@Primary 

			@EnableTransactionManagement  

	方式二：AOP 配2套datasource......

		重写AbstractRoutingDataSource  determineCurrentLookupKey 中取datasourceType

		DataSource setTargetDataSources(Map) Map中放2个数据源对象

		@before

		配置多个数据源，加载到spring工厂，创建多线程，每个数据源对应一个线程，实际调用通过aop匹配某一个具体数据源

	minIdle=10 maxIdel=100 maxTotal=10 查询超时20s maxWaitMillis=600s

		

	事务：

		TransactionManager

			TransactionStatus getTransaction(TransactionDefinition)

		 TransactionStatus TransactionInfo

		方法必须是public

		Aop.currentProxy()

		ThreadLocal 

		TransactionTemplate @Transactional(rollBackFor=)

		AbstractPlatformTransactionManager getTransaction

			savepoint

		事务性资源是存储在ThreadLocal<Map<Object, Object>>里，key就是DataSource对象，value就是ConnectionHolder对象

		SuspendedResourcesHolder  挂起事务

		事务对象要关联到一个物理事务ConnectionHolder存在，同时物理事务必须是活动的。

		从上下文获取事务，判断当前事务是否存在，根据当前事务的传播行为处理事务，

		配置事务：

			xml: 事务管理器transactionManager--datasource, 事务传播行为

			注解

	QPS： PV * 80% / (hour * 60 * 60) ----> 1000W * 0.8 / 4 * 60 * 60 = 555.556

		4---取网站高峰时间段：晚上8点-晚上12点

		555/80=7 向上取整

	Mybatis dao bean是如何注册成为bean的？

		invokeBeanFactoryPostProcessors 注册一个MapperFactoryBean

		实例化 ：

		重写改变扫描方式：ClassPathMapperScanner

			接口+独立类（默认Spring是：非抽象，单例，非懒加载）

		MapperFactoryBean的getObject()创建动态代理MapperProxy

	日志的发展历史：

		log4j(1)白色   jul

		jcl红色       slf4j(1)

		log4j2  logback(1)

		logging.config=classpath:log4j2.xml  放在resources下

		log4j的性能问题：多线程

			Category.callAppenders

				for向上循环竞争锁--synchronized parent

		log4j2 采用异步的方式 

		美团--采用高性能队列Disruptor

		三级缓存：

			

	rocketmq 消息消费失败了

		自动进入重试队列%RETRY%+consumerGroup，重试16次，延迟时间时间18个等级  (设置选择一个)

		自动进入私信队列 %DLQ%+consumerGroup

	rocketmq四大组成部分：producer consumer broker(存储消息，转发消息) name serving(对topic和路由信息的管理)

		consumer 推拉模式、支持集群消费（1-1 默认）、广播消费



	OOM场景：

		深度递归（栈帧溢出）、-Xmx值太小（heap）、gc时对象过多--调整GC策略、ThreadLocal 引发（最好手动remove()但需要结合业务考量，最好是保持数据太大/ThreadLocalMap）、native method的调用

	ThreadLocal：每个Thread 维护一个 ThreadLocalMap 映射表，这个映射表的 key 是 ThreadLocal实例本身，value 是真正需要存储的 Object  WeakReference

	jstat -gc 间隔 计算

	[Times: user=11.53 sys=1.38, real=1.03 secs] 



	IO模型：阻塞IO、非阻塞IO、多路复用IO、信号驱动IO以及异步IO 

	IO线程模型

		传统模型：BIO、BIO、AIO

		reactor模型：

			单Reactor单线程：一个接待员select,dispatch，一个服务员handler

			单Reactor多线程：一个接待员，多个服务员->多个厨师worker线程池

			主从Reactor多线程

		Proactor模型



	new ServerSocket(port, [请求的队列长度大小限制])

	accept() 等待客户端的连接

	new Socket(host, port);  // 后者 new Socket() socket.connect(port);



	tomcat优化：启动NIO-修改connector protocol maxThreads= minSpareThreads 初始化线程数 开启压缩



	装饰器是增加本身，需要将本身传递进来，代理是 借助。。适配器是改变对象的接口



	限流算法：

		计数器：redis的自增incr,时间窗口-失效时间ttl  临界问题：因为是固定时间内的计数-临时的话，临界左右加起来，就立马爆掉了

		令牌桶 guava的RateLimiter.create(10)实现 应对突然

		漏桶：固定的速率 超过丢弃（限流策略）无法应对短时间的突发流量 

		滑动窗口

	

	事务不生效：

		私有方法、

		同类调用AopContext.currentProxy()、

		没指定RollbackFor（未作任何配置的情况下，spring只对运行期异常，即RuntimeException进行回滚 ）、不要把注解放到接口上

			AbstractFallbackTransactionAttributeSource#computeTransactionAttribute

			什么都不加 是0 ， public  是1 ，private 是 2 ，protected 是 4，static 是 8 ，final 是 16 

			!Modifier.isPublic return null;        (mod & 0x00000001) != 0  所以=0代表false不支持，只有public满足条件

	

	伪共享：多线程操作各自的变量，但是这2个变量却共享一个缓存行cache line，导致竞争冲突。

		缓存行填充

		使用编译指示，强制使每个变量对齐。



	冒泡排序时间复杂度：O(n²) 

	怎么解决hash冲突？

		开放定址法

		链地址法

		再哈希法，直到不冲突

		建立一个公共溢出区

		

	https://blog.csdn.net/DPnice/article/details/92799375 架构设计好图



	线程池大小

		CPU密集型：CPU核数+1个线程的线程池

		IO密集型：尽可能更多-CPU*2   CPU/(1-阻塞系数)
