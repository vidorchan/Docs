JsonUtil.stringToObject(String, Class)//Json格式的字符串转换成指定类型
DateUtils.addDays
ApplicationListener、InitializingBean 、DisposableBean
LinkedHashMap 插入顺序，在邮件附件中使用

ArrayUtils.toMap() //将一个二维数组转Map
BeanUtils.getProperty(object, property)//得到对象的属性值
StringUtil.isNullOrEmpty()
JsonPath.read(jsonString, "$.path") //输出json中path的值
--淘汰：Jaxb处理Java对象和xml之间转换常用的annotation：@XmlType @XmlElement @XmlRootElement @XmlAttribute @XmlAccessorType @XmlAccessorOrder @XmlAccessorOrder @XmlTransient @XmlJavaTypeAdapter
<![CDATA[              ]]>  中间的内容会被解析器忽略

grant select(权限) on 资源 to 账户 with grant option; 
--with grant option 权限赋予/取消是级联的，但是管理员不能直接取消下级的权限
--with admin option 权限赋予/取消不是级联的，但是管理员可以直接取消下级的权限
tablespace ？？
CREATE TABLE 表名(字段名 类型...)
COMMENT ON COLUMN 列名 IS ''
ALTER TABLE 表名 ADD CONSTRAINT 约束名 PRIMARY KEY(列名)
CREATE INDEX 索引名 ON 表名(列名)
GRANT SELECT(权限) ON 资源 TO 新用户 WITH GRANT OPTION
decode(条件,值1,翻译值1,值2,翻译值2,...值n,翻译值n,缺省值)
sign()函数根据某个值是0、正数还是负数，分别返回0、1、-1
如何监控当前数据库谁在运行什么SQL语句？
SELECT osuser, username, sql_text from v$session a, v$sqltext b
where a.sql_address =b.address order by address, piece；
如何监控 SGA 的命中率？
select a.value + b.value "logical_reads", c.value "phys_reads",
round(100 * ((a.value+b.value)-c.value) / (a.value+b.value)) "BUFFER HIT RATIO"
from v$sysstat a, v$sysstat b, v$sysstat c
where a.statistic# = 38 and b.statistic# = 39
and c.statistic# = 40；
如何监控 SGA 中共享缓存区的命中率，应该小于1% ？
select sum(pins) "Total Pins", sum(reloads) "Total Reloads",
sum(reloads)/sum(pins) *100 libcache
from v$librarycache；
select sum(pinhits-reloads)/sum(pins) "hit radio",sum(reloads)/sum(pins) "reload percent"
from v$librarycache；
表明对语句块选择基于开销的优化方法，并获得最佳吞吐量，使资源消耗最小化。
例如：
SELECT /*+ALL+_ROWS*/ EMP_NO，EMP_NAM，DAT_IN FROM BSEMPMS WHERE EMP_NO='CCBZZP'；
表明对表选择索引的扫描方法。
例如：
SELECT /*+INDEX(BSEMPMS SEX_INDEX) USE SEX_INDEX BECAUSE THERE ARE FEWMALE BSEMPMS */ FROM BSEMPMS WHERE SEX='M'；

NVL
NVL2
INSTR
SUBSTR

JAVA	XML		Oracle			Mysql		Mybatis
Long			Number(10)
String	VARCHAR		varchar2
Date	TimeStamp	Date
Double	DECIMAL		NUMBER(8,2)		

(([1-9][0-9]{0,5})|0)([.][0-9]{1,2})?
？前面指定出现0次或者1次
[]指定字符集
{m,n}最少m次，最多n次

分页查询使用Mybatis插件扩展机制，拦截分页处理的sql，在里面调用*Count计算总行数 PageInterceptor implements Interceptor
// 从Executor中获取事务的连接
Connection connection = ConnectionLogger.newInstance(executor.getTransaction().getConnection(), statementLog);

spring.jackson.serialization.write-dates-as-timestamps=true

Mybatis 如何优雅的输出SQL： Mybatis Log Plugin

动态SQL拼接，使用trim就是为了删掉最后字段的“,”<trim prefix="set" suffixOverrides=","><if test="srcId!=null">SRC_ID=#{srcId},</if>

Maven解决jar包冲突：exclusion标签  

IDEA插件：
free-idea-mybatis dao直接跳转到xml
Mybatis Log Plugin 优雅看sql，前提是已经在console输出sql语句Preparing、Parameters
Dependency Analyzer 分析jar依赖

安全：
1.
先：SSO 认证 单点登录，session，cookies  SsoFilter	  分布式的扩展性不行、session开销太大、不安全性-跨站请求伪造的攻击
JWT 认证	JwtRequestFiler JwtRequestFilterEnableAutoConfig	(请求头有JWT就走JWT，没有则走SSO)  头部Header.载荷PayLoad.签名Signature
开发环境推荐使用对称算法，性能较好
x-jwt-ms-token 微服务之间调用 HS512
x-jwt-gw-token 网关调用 RS512
window.atob()
最后：服务鉴权 AOP @JalorOperation SecurityInterceptor#invoke
	安全控制栈、入栈push、出栈pop、安全认证策略、资源类型、资源编码、操作编码
	权限传播问题--事务传播问题：将this调用改为自调用xxx.method()
2. 服务管控 拦截器
3.数据安全 安全性、可用性、可控性、完整性。  Mybatis拦截器实现数据范围(行权限)
4.安全加解密 密钥不落地、密钥自动轮换、根密钥HSM保护的加密、解密能力
防止CSRF漏洞,不在域白名单中URL请求直接返回

window.atob() 解码 window/btoa() 加码

JAX-RS CXF JAX-WS

spring提供一个工具类 StringUtils.hasText() isEmpty()
jdk Objects.nonNull(Object) 非空判断

oracle.jdbc.xa.client.OracleXADataSource 和oracle.jdbc.driver.OracleDriver 区别

性能指标：TPS、响应时间、并发、成功率
思考时间
长连接用户、短连接用户、无连接用户
资源状态达标：CPU<80% IO<60%

@JsonIgnoreProperties 序列化忽略对象属性

select * from v$version;   --查看Oracle DB版本

JsonPath.read(jsonStr, path) 从json字符串中取数据

MySQL：
校对规则 utf8_general_ci 大小写不敏感，性能更高	utf8_unicode_ci 德语、法语、俄语更精准的校对	并不影响数据的保存结果，影响数据的查询结果及排序。
<2000万 单表
2000万-6000万分区表
分布式中间件或历史数据迁移
表名字段名 小写字母，数字字母下划线 32个字符
主键名：pk_主键字段名构成
外键名：fk_外键表名_主键表名_外键字段名
普通索引：idx_索引字段
主键索引：idx_pk_索引字段
唯一索引：idx_uk_索引字段
外键索引：idx_fk_外键表名_主键表名_外键字段
同一个字段名在一个数据库只能代表一个意思。
临时表：tmp_t_表名_日期
备份表：bak_t_表名_日期
按日期分表：表名_YYYY[MM][DD]
主键：无符号BIGINT 0-18446744073709551615
不强制使用外键约束
单表字段数上线30，再多考虑垂直分表，建立主副表，主键一一关联
	冷热数据分离、大字段分离（特别对于有一个text/blob或很大长度的varchar字段时，更应考虑单独存储）、常在一起做条件的数据，返回列不分离
必须满足2NF，可以不满足3NF，以换取效率
updated_date TimeStamp 方便 占用4个字节 内部以UTC毫秒存储 timestamp可以在insert/update行时，自动更新时间字段(一个表只能定一个一列)
	set time timestamp not null default current_timestamp on update current_timestamp
updated_by decimal
updated_version decimal
id bigint 20
varchar utf8 21844个汉字，或65532个英文
sphinx 第三方全文索引引擎 text 2^16
必须使用varchar存储电话号码 +-（）
mysql版本5.7.23
datetime 5个字节（5.5 8个字节）
索引字段必须定义not null约束，否则影响性能
存储IPV4 int unsigned inet_aton() inet_ntoa()互相转换
mysql分区：hash,key,range,list
单表索引最多5个，或不能超过字段个数的20%
where条件里面字段的顺序与索引顺序无关
最左前缀原则
Innodb中，没有直接存储地址，而是存储主键值，这样会牵涉到一个回表的操作
能确定只返回一条数据，使用limit 1
OR < IN < UNION 推荐使用
跳跃式分页
尽量避免extra列出现：Using File Sort，Using Temporary，rows超过1000的要谨慎上线。
Using filesort --文件排序 无法利用索引完成排序
禁止有super权限的应用程序账号存在 -- 会导致read only失效，导致较多诡异问题而且很难追踪。
一个事务中，把并发要求高的语句放到最后执行，这样可以缩小持有锁的时间，提升效率：执行顺序：1.写日志，2.减库存，3.增库存
并发线程数=CPU核数*2+【磁盘数】（SSD不加）
mysql架构+mvcc
分库：
 1000万数据+ 单库500G
 1. 增删改查需要带分库字段2.更新，需要改为删除和插入3.扩容不能引发数据迁移
 取模：会超，不适应。
schema号= int(物权/每个实例容纳物权个数)+1  如果从0开始，则不需要加1
交易日志表分库： 物权加日期  缺点：对当月的库存写的压力很大
行锁是在需要时才加上，提交才释放!!!
数据同步工具：
fastjson @JSONFiled(name="ZhiDingName")
@JsonNaming(PropertyNamingStrategy.UpperCamelCaseStrategy.class)
JSONObject.toJSONString
@JsonFormat(pattern="yyyy-MM-dd", timezone="GMT+8")
@JsonDeserialize(using = YearMonthDateJsonDateDeserializer.class)
?useAffectedRows=true  --mysql批量

server.tomcat.max-http-header-size=32768  -- 400的错误

线程泄漏：这是线程池应用中一个严重的问题，当任务执行完毕而线程没能返回池中就会发生线程泄漏现象

PowerMockRunner MockitoJUnitRunner

1. IDEA datasource为啥连不上localhost
2. properties文件怎么指定mybatis插件列表
3. findbugs配置
4.sqlsessionfactory下配置mybatis插件

安全：
认证：SSO，JWT
鉴权：AOP
黑白名单

SSOFilter: 会将所有的可疑字符进行替换，过高的力度限制了用户的功能
》》引起工具类，用户按需自行转码

XSS攻击 -> XssFilter 拦截请求，对可疑字符进行替换

CsrfFilter 防护 -> x-csrf-token

脱密日志打印
入参校验
SQL注入
请求头校验
跨目录访问防护 -- 需要主动调用工具类方法进行校验
DoS 攻击防护  - 配置访问频率 过滤器实现
重定向漏洞防护 - 对URL进行校验（https/http、正则、白名单校验）

服务治理：
负载均衡 轮询i=(i+1) mod n，随机 UP的server
  服务端-Nginx
  客户端-从注册中心获取到服务列表，然后，选择一个进行请求
容错： 采取重试、更换目标主机地址 最大调用次数=(同一地址最大重试次数+1)*(下一地址最大重试次数+1)
  目标服务不可达-网络异常、链接拒绝
  对指定异常代码进行容错重试
客户端熔断 hystrix USF CircuitBreakerHandler
  长时间没有响应-防止因服务故障或网络延迟导致请求堆积，占用服务器资源
  大量并发/达到熔断阈值

网关：
 认证、鉴权、会话管理、路由、Token颁发、服务限流

REST服务的发布 cxf暴露服务
  <jaxrs:server id="" address=""
访问地址：应用地址/内置CXFServlet响应地址services/address的值/REST接口的Path
网关访问地址规则：本机域名 + 网关端口 + 网关上下文根 + 微服务appId : 微服务subAppId + 微服务上下文根 + 服务地址；

@Path,@Consumes,@Procedures

nextval

JAR包用于docker部署，war包用于tomcat部署，tar.gz也用于docker

全链路调用的跟踪

隔离
  限流  触发限流的请求会被降级处理
  超时 请求超时做降级处理


log4j2 日志高亮输出：
-Dlog4j.skipJansi=false
logging.config=classpath:log4j2.xml
pattern="%d{yyyy-MM-dd HH:mm:ss.SSS} %highlight{%-5level} [%t] %highlight{%c{1.}.%M(%L)}: %msg%n"
