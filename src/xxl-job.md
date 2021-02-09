下载源码：

https://github.com/xuxueli/xxl-job/tree/v2.2.0



运行sql:

xxl-job-2.2.0\doc\db\tables_xxl_job.sql



修改xxl-job-admin的配置文件，然后运行项目：

	com.xxl.job.admin.XxlJobAdminApplication



然后，可运行samples下一个springboot项目

	com.xxl.job.executor.XxlJobExecutorApplication



学习简单，轻量级，易扩展

特性：

1. 页面动态修改，实时生效
2. 弹性扩容缩荣
3. 超时控制，失败重试、任务告警机制
4. 



---

BEAN模式：最后都要在页面进行配置



类形式：一个任务对应一个JAVA类

优点：使用范围广，Servletless都支持；

缺点：类太多，需要手动注入到执行器容器

使用：

    继承IJobHandler，实现execute()
    // 手动注册
    XxlJobExecutor.registJobHandler("demoJobHandler", new DemoJobHandler());



方法形式：一个任务一个方法

优点：快速，自动注册到执行器容器

缺点：要求spring容器

使用：

    @XxlJob("demoJobHandler")
    // 日志打印
    XxlJobLogger.log



    XxlJob：
    	value： name
    	init: init handler
    	destroy: 资源回收



DB设计说明：

    - xxl_job_lock：任务调度锁表；
    - xxl_job_group：执行器信息表，维护任务执行器信息；
    - xxl_job_info：调度扩展信息表： 用于保存XXL-JOB调度任务的扩展信息，如任务分组、任务名、机器地址、执行器、执行入参和报警邮件等等；
    - xxl_job_log：调度日志表： 用于保存XXL-JOB任务调度的历史信息，如调度结果、执行结果、调度入参、调度机器和执行器等等；
    - xxl_job_log_report：调度日志报表：用户存储XXL-JOB任务调度日志的报表，调度中心报表功能页面会用到；
    - xxl_job_logglue：任务GLUE日志：用于保存GLUE更新历史，用于支持GLUE的版本回溯功能；
    - xxl_job_registry：执行器注册表，维护在线的执行器和调度中心机器地址信息；
    - xxl_job_user：系统用户表；



 



调度模块（调度中心）： 负责管理调度信息，按照调度配置发出调度请求，自身不承担业务代码。 

执行模块（执行器）： 负责接收调度请求并执行任务逻辑。 



与Quartz 对比：

    相同点：都适用于分布式调度
    不同点：
    	Quartz:
    		1. 调用API的的方式操作任务，不人性化; -- 以此方式实现集群分布式调度
    		2. 调度逻辑和持久化业务QuartzJobBean耦合严重，性能降低
    		3. 以“抢占式”获取DB锁并由抢占成功节点负责运行任务
    	XXL-JOB：
    		执行器实现“协同分配式”运行任务，集群性能更高
    		调度+执行



调度模块：

    1. 线程池
    2. 并行调度
    	调度室并行的，但是执行器那边是串行执行
    3. 处理过期策略
    4. 日志回调服务
    5. 故障转移FailOver
    	路由策略选择故障转移，发起每次调度时，会按顺序发送心跳检测请求，第一个存活的会接收到请求
    6. 任务依赖
    	子任务存在，在当前任务执行成功后，执行
    7. 异步

执行器：

    内嵌的server，默认端口9999
    子线程记录日志



分片广播：

1. 支持动态扩容集群机器
2. 动态增加分片数量
   场景：10个执行器处理10w条数据、广播进行缓存更新、广播运行shell脚本



    xxl.job.accessToken=



执行器灰度上线：

	将执行器改为手动注册，先不注册需要重启的机器，待更新重启完后，再注册其他还没更新的机器。



任务超时中断和任务终止机制：

	通过interrupt中断程序，最外层抛出异常：InterruptedException



通讯要素：

1. 序列化
   1. 数据协议
   2. 时间戳校验System.currentMilliseconds-数据加密



告警邮箱发送：EmailJobAlarm



    ### xxl-job executor log-retention-days
    xxl.job.executor.logretentiondays=30
    
    ### xxl-job, log retention days
    xxl.job.logretentiondays=30



调度记录停留在 "运行中" 状态超过10min，且对应执行器心跳注册失败不在线，则将本地调度主动标记失败 



调度中心api: JobApiController 

执行器api: com.xxl.job.core.biz.ExecutorBiz 	ExecutorBizImpl

	com.xxl.job.core.server.EmbedServer.EmbedHttpServerHandler#channelRead0 执行器执行任务入口：

		基于Netty SimpleChannelInboundHandler<FullHttpRequest>

		拿到任务，异步执行：

    ThreadPoolExecutor bizThreadPool = new ThreadPoolExecutor(
                            0,
                            200,
                            60L,
                            TimeUnit.SECONDS,
                            new LinkedBlockingQueue<Runnable>(2000),
                            new ThreadFactory() {
                                @Override
                                public Thread newThread(Runnable r) {
                                    return new Thread(r, "xxl-rpc, EmbedServer bizThreadPool-" + r.hashCode());
                                }
                            },
                            new RejectedExecutionHandler() {
                                @Override
                                public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
                                    throw new RuntimeException("xxl-job, EmbedServer bizThreadPool is EXHAUSTED!");
                                }
                            });



    





代码段：

    // com.xxl.job.admin.core.util.I18nUtil  获取i18n中国际化数据
    String i18nFile = MessageFormat.format("i18n/message_{0}", language);
    Resource resource = new ClassPathResource(i18nFile);
    EncodedResource encoudedResource = new EncodedRecource(resource, "UTF-8");
    Properties prop = PropertiesLoaderUtils.loadProperties(encodedResource);
    String value = pro.getProperty(key);



    // 滚动日志查看效果
    -前端实现： ajax------------------------------------
    间隔调用：
    var logRun = setInterval(function () {
            pullLog()
        }, 3000);
    每次调用完，都检查是否已经读完，读完则，清除定时：
    	fromLineNum > toLineNum && isEnd = true
    	window.clearInterval(logRun)
    fromLineNum = data.content.toLineNum + 1; // 每次调用完设置下次加载的初始位置
    $('#logConsole').append(data.content.logContent); // 每次结果追加到div
    加载完全，scrollTo(0, document.body.scrollHeight); // 滚动到底
    
    每次进来triggerTime 不会变  
    日志：/data/applogs/xxl-job/jobhandler/2020-02-09/xxl_job_log的id.log
    
    -java后台实现----------------------------------------
    判断是否已经读到文件的尾部：
    logResult.getContent()!=null && 
    logResult.getContent().getFromLineNum() > logResult.getContent().getToLineNum()
        设置：isEnd = true
        
    
    // 一行一行读文件    
    StringBuffer logContentBuffer = new StringBuffer();
    int toLineNum = 0;
    LineNumberReader reader = null;
    reader = new LineNumberReader(new InputStreamReader(new FileInputStream(logFile), "utf-8"));
    String line = null;
    
    while ((line = reader.readLine())!=null) {
        toLineNum = reader.getLineNumber();		// [from, to], start as 1
        if (toLineNum >= fromLineNum) {
            logContentBuffer.append(line).append("\n");
        }
    }




