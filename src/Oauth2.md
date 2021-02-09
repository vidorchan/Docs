

sso-auth-code-server 认证服务器

1. 引入依赖：

    spring-cloud-starter-oauth2
    druid
    spring-boot-starter-data-redis
    spring-session-data-redis
    mysql-connector-java
    mybatis-spring-boot-starter
    spring-cloud-alibaba-nacos-discovery

1. 配置文件：

    server:
      port: 8888
    spring:
      application:
        name: auth-server
      cloud:
        nacos:
          discovery:
            server-addr: localhost:8848
      datasource:
        username: 
        password: 
        driver-class-name: com.mysql.jdbc.Driver
        url: jdbc:mysql://localhost:3306/sso?serverTimezone=Asia/Shanghai&useUnicode=true&characterEncoding=UTF-8&useSSL=false
        type: com.alibaba.druid.pool.DruidDataSource
    
      redis:
        host: 
        port: 6379
        password: 
      session:
        store-type: redis

1. 启动类配置：
   @EnableRedisHttpSession
2. 各种配置文件：

    @EnableAuthorizationServer
    AuthorizationServerConfigurerAdapter
    	configure(ClientDetailsServiceConfigurer clients) 加载已经在认证服务注册的客户端
        
        configure(AuthorizationServerEndpointsConfigurer endpoints) token存储，授权服务器的配置
        
        configure(AuthorizationServerSecurityConfigurer security) 授权服务器安全配置
        

    @EnableWebSecurity
    WebSecurityConfigurerAdapter
    	configure(AuthenticationManagerBuilder auth)
        
        configure(WebSecurity web)
        	web.ignoring().antMatchers("/assets/**", "/css/**", "/images/**"); 设置前台资源不拦截
        configure(HttpSecurity http)
        
        AuthenticationManager authenticationManagerBean()



    CorsFilter corsFilter() cors配置 cross-origin resouce sharing
    new CorsFilter(CorsConfigurationSource)
        new UrlBasedCorsConfigurationSource().registerCorsConfiguration(String path, CorsConfiguration config)
        	new CorsConfiguration()
        		setAllowCredentials(true); // 允许cookies跨域
    			config.setAllowCredentials(true); // 允许cookies跨域
    			config.addAllowedOrigin("*");// #允许向该服务器提交请求的URI，*表示全部允许，在SpringMVC中，如果设成*，会自动转成当前请求头中的Origin
            	config.addAllowedHeader("*");// #允许访问的头信息,*表示全部
            	config.setMaxAge(18000L);// 预检请求的缓存时间（秒），即在这个时间段里，对于相同的跨域请求不会再预检了
            	config.addAllowedMethod("*");// 允许提交请求的方法，*表示全部允许

    一般都会继承这个，赋予更多的属性
    org.springframework.security.core.userdetails.User

    定制logout页面：
    LogoutSuccessHandler
    	onLogoutSuccess
    		HttpServletResponse sendRedirect(String url)
        		HttpServletRequest getParameter("redirectUrl")

    

	 

sso-api-gateway 网关

    认证过滤器：
    	GlobalFilter,Ordered-0,InitializingBean-加载不需要验证的url
    		filter(ServerWebExchange exchange, GatewayFilterChain chain):
    			exchange.getRequest().getURI().getPath()
                 exchange.getRequest().getHeaders().getFirst("Authorization"); 获取请求头信息
                 请求oauth/check_token，拿到TokenInfo(自)信息
    
    鉴权过滤器：
    	GlobalFilter,Ordered-1,InitializingBean-加载不需要验证的url
    		filter(ServerWebExchange exchange, GatewayFilterChain chain)
                 exchange.getRequest().getURI().getPath();
    			exchange.getAttribute("tokenInfo");
    			匹配访问的资源是否在用户资源中--权限校验

    根据抛出的异常不同返回不同的报错信息
    DefaultErrorWebExceptionHandler
    	renderErrorResponse(ServerRequest request)
        	ServerResponse.status(HttpStatus.OK)
                    .contentType(MediaType.APPLICATION_JSON_UTF8)                .body(BodyInserters.fromObject(gateWayExceptionHandlerAdvice.handle(throwable)));
    	@ExceptionHandler
    
    自动加载该自定义的handler:
    @EnableConfigurationProperties...
    ExceptionAutoConfiguration
    	errorWebExceptionHandler(ErrorAttributes errorAttributes)
        	DefaultErrorWebExceptionHandler exceptionHandler = new CustomErrorWebExceptionHandler



sso-portal-code 授权码模式 前端

    拦截器配置：
    HandlerInterceptor
    case1-cookie:
    	HttpServletRequest getCookies(); 拿到所有的cookies,判断COOKIE-ACCESS-TOKEN-KEN是否存在
    	HttpServletRequest getCookies(); 拿到所有的cookies,判断COOKIE-REFRESH-TOKEN-KEN是否存在
    	access token存在，则返回token
    	access token不存在：
    		判断refresh token是否存在：
    			存在，重新请求token:oauth/token去刷新access token,如果报错的话，说明fresh token过期,则需要重新登录
    			不存在，重新登录 HttpServletResponse  sendRedirect()
    case2-session:
    	HttpServletRequest getSession() getAttribute("portal-token-info-key"); 取出session
    	判断是否有tokeninfo：
    		存在，判断是否过期，过期，则重新请求token:oauth/token去刷新access token，如果报错的话，说明fresh token过期,则需要重新登录
    		不存在，重新登录 HttpServletResponse  sendRedirect()

    Controller调用网关层
    



---



sso-api-gateway-jwt 网关

    GlobalFilter,Ordered,InitializingBean
    afterPropertiesSet中，去认证服务器拿秘钥：
    此时，需要自定义一个restTemplate，否则，此时的restTemplate不具备负载均衡的能力，&报错
    	restTemplate&ribbon加载过程
    解析token，判断权限

sso-auth-code-jwt-server 授权码模式-认证服务器

    @EnableAuthorizationServer
    AuthorizationServerConfigurerAdapter
    	new JwtTokenStore(jwtAccessTokenConverter());
    		new JwtAccessTokenConverter() setKeyPair(keyPair());
    			new KeyStoreKeyFactory(new ClassPathResource("keystore.jks"), "mypass".toCharArray());
    			keyStoreKeyFactory.getKeyPair(String alias, char[] password)

JDK工具生成秘钥：

keytool -genkeypair -alias mytest -keyalg RSA -keypass mypass -keystore keystore.jks -storepass mypass

    TokenEnhancer token增强器
    	OAuth2AccessToken enhance(OAuth2AccessToken accessToken, OAuth2Authentication authentication)
        	authentication.getPrincipal() 从这里取数数据，塞到additionalInformation中
        	((DefaultOAuth2AccessToken) accessToken).setAdditionalInformation(Map<String, Object> additionalInformation)

    configure(AuthorizationServerSecurityConfigurer security) 认证服务器安全控制
    	security.checkTokenAccess("isAuthenticated()")
         .tokenKeyAccess("isAuthenticated()");//来获取我们的tokenKey需要带入clientId,clientSecret
          security.allowFormAuthenticationForClients();
          
    configure(AuthorizationServerEndpointsConfigurer endpoints) 认证服务器配置
    	endpoints.tokenStore(tokenStore()) //授权服务器颁发的token 怎么存储的
          .tokenEnhancer(tokenEnhancerChain)...
       
    configure(ClientDetailsServiceConfigurer clients) 从数据库获取客户端注册信息
    	JdbcClientDetailsService(DataSource)


