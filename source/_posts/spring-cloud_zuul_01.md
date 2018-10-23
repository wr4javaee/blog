---
title:  "Spring Cloud Finchley 在业务系统中的实践总结 - Zuul的实践与增强"
date: 2018-10-19 15:38:00
categories: Spring Cloud
tags: Zuul
toc:
  on: true
  max_depth: 3
  nowrap: false
  list_number: true
toc_list_number: true
---
## 前言
　　本文主要针对Zuul在实际业务场景中的一些实践心得进行总结，重点围绕Zuul本身关于动态配置缺陷问题、老项目接入及老环境上线问题等方面进行讨论。关于Zuul的一些基本用法，可直接参考[Spring Cloud官方文档
](http://cloud.spring.io/spring-cloud-static/Finchley.SR1/single/spring-cloud.html)与[Zuul Github](https://github.com/Netflix/zuul)。
　　我们使用的Zuul基于Spring官方目前最新版本Spring Cloud Finchley.SR1。

---

<!-- more -->
## Zuul是什么
　　Zuul可以用两个单词总结，即路由和过滤器（Router and Filter），它是Netflix开源，并被Spring所集成，成为当前Spring Cloud最核心的组件之一。它的基本功能可概括如下：

* 认证鉴权（Authentication）
* 审查（Insights）
* 压力测试（Stress Testing）
* 金丝雀测试（Canary Testing）
* 动态路由（Dynamic Routing）
* 服务迁移（Service Migration）
* 负载剪裁（Load Shedding）
* 安全（Security）
* 静态应答处理（Static Response handling）
* Active/Active traffic management

---

## 为什么要使用它
　　路由是微服务的架构体系当中必不可少的一部分，那么缺少它会带来什么问题呢？这里以公司某业务线实际场景为例：
![Spring Cloud Zuul 01](/images/15397653100461/Spring%20Cloud%20Zuul%2001.jpg)
　　上图是是一个简单的前后端交互架构，来自PC/微信/H5/IOS/Android等客户端将请求发送至Nginx，后由Nginx反向代理到指定后端服务集群中。我们能够很方便的为后端服务集群提供负载均衡，同时通过Nginx的反向代理，可以隐藏后端真实IP，即提供了一定程度的安全保障，也使得前后端服务调用隔离，降低开发成本。
　　随着业务规模的不断扩大，后端微服务已从数个服务集群扩展到了数十个，此时Nginx为每一个服务集群都配置了唯一的域名用于反向代理，每当这些服务集群需要扩容或机器迁移时，需要运维投入大量的精力去手动修改配置。此外，这些数十个域名也造成了资源的极度浪费。
　　
　　![](/images/15397653100461/15398454116261.jpg)
    


　　在近些年火热的微服务浪潮下，我们都热衷于将每一个原子业务单元都拆分成微服务，这对运维来说造成了很多压力，就好像上图中，当你面相这种规模的服务集群时，传统的维护手段已接近于灾难，必须要想办法借助更多的工具来帮助我们。
　　相比Nginx，我们可以将Zuul理解为微服务集群的一个可靠的大管家，它是上述问题的解决方案之一。使用它，可以方便的用一个域名来替换掉之前的数十个域名，此外，当服务集群扩容或迁移时，Zull可以做到自动识别，即动态路由，它还具有服务鉴权、服务降级、熔断、分流控制、接口控制等Nginx所不具备的重要功能。

---

## 问题与挑战
　　Spring Cloud Finchley版本基于Spring Boot2.0构建，按照约定大于配置的思想，我们能够很轻易的上手使用这些开源框架。但是任何开源框架放到实际应用场景中，都不可能百分百的切合我们的实际需求，例如

### 当前Spring Cloud Zuul最新版本不支持路由的动态配置
　　虽然Zuul能够做到动态路由，但是反向代理的配置信息是需要在application.yml配置文件中维护的，这意味着每一次有新的服务集群需要进行反向代理时，我们需要像重启Nginx一样来重启Zuul，重启意味着增加了上线的风险与成本。

### 非Spring Boot项目接入问题
　　对于任何一个Spring Boot项目来说，可以非常简单方便的与Spring Cloud Zuul整合，但是官方并未提供一些工具来支持一些老项目的接入，目前实际业务线大部分服务均是非Spring Boot的老服务，我们需要自己来实现这些通用的工具组件。

### 老环境上线迁移问题
　　长期以来前端业务线一直采用的是Request-Nginx-Server架构。基于前文介绍，我们的Request端分散在各大手机应用市场、微信等合作渠道，对于这些已经集成了数十个不同服务域名的前端来说，接入Zuul会有一系列的问题需要解决：

* App端每次更新都要重新发版，时效性差
* App端接入Zuul后若出现问题，无法及时回滚
* 旧版本App无法修改接入Zuul

### 上线验证问题
* 测试与线上环境不一致，上线存在未知的风险
* 我们的业务模式，无法在线上环境进行全流程测试

　　基于业务模式，系统中有大量节点需要对接第三方机构，无法进行全流程测试，同时，当前我们的测试环境无法做到与线上环境完全一致，对于Zuul的接入来说是一个非常大的挑战。

---

## Zuul的核心架构
　　Zuul的核心其实很简单，就是Filter，如下图所示：
![](/images/15397653100461/15397741541838.png)

可以看到，Zuul将过滤器大致分为4类，Pre、Routing、Post/Error

![](/images/15397653100461/15398306875833.jpg)


* pre： 这种过滤器在请求被路由之前调用，安全与鉴权功能可在此实现。
* routing：通过HttpClient/Ribbon将请求路由到后端服务。
* post：路由后执行，可以接收到后端服务的响应信息，并转发给请求端。
* error：在其他阶段发生错误时执行该过滤器。

#### Spring Zuul提供的默认过滤器

![](/images/15397653100461/15399408807599.jpg)

　　Zuul已经为我们封装好了基本的过滤器，表格中Order是这些过滤器的执行顺序，可见Zuul为我们充分预留了很多空间来对默认过滤器进行增强，例如

* 安全/鉴权
* 流量控制
* 接口控制
* 访问统计
* ...

---

## 动态路由增强，实现路由的动态配置
　　Zuul相比较于Nginx的一个很大优势在于其提供的动态路由功能，前文已经介绍过，当服务过多的时候，运维需要大量的时间去手动维护路由的映射关系，极易造成严重的线上问题。
　　好在Zuul与Eureka（注册中心）在Spring Cloud中实现了完美的整合，Eureka能够实时管理不同集群的每一个节点信息，Zuul便能够通过注册中心获取每一个服务的详细清单，并通过Ribbon实现负载均衡，从而达到请求到转发的自动路由机制。
　　前文提过Zuul并没有实现路由的动态配置，每次新增反向代理规则都需要重启Zuul未免有点尴尬，是否可以修改Zuul来避免重启呢？我们需要看一下Zuul的动态路由的实现原理。

### 源码分析
　　找到spring-cloud-netflix-zuul-2.0.1.RELEASE.jar，看一下路由定位器相关实现：

![zuul_locato](/images/15239442308370/zuul_locator.jpg)


* RouteLocator 路由定位基础接口
* RefreshableRouteLocator 提供刷新接口
* SimpleRouteLocator 基础实现定位器，主要实现了路由定位与路由加载逻辑
* CompositeRouteLocator 复合定位器，提供路由定位、路由刷新功能
* DiscoveryClientRouteLocator 组合静态以及配置好的路由

　　通过查看Zuul的源码发现，Zuul已经为我们定义好路由定位于刷新的接口标准，RouteLocator接口主要定义了路由定位器，如下

```
	/**
	 * Ignored route paths (or patterns), if any.
	 */
	Collection<String> getIgnoredPaths();

	/**
	 * A map of route path (pattern) to location (e.g. service id or URL).
	 */
	List<Route> getRoutes();

	/**
	 * Maps a path to an actual route with full metadata.
	 */
	Route getMatchingRoute(String path);
```

　　RefreshableRouteLocator接口，只定义了一个路由信息刷新接口

```
    void refresh();
```

　　SimpleRouteLocator，主要实现了RouteLocator接口，在内部引入了ZuulProperties。ZuulProperties的配置是我们在application.yml中配置好的Zuul相关信息，其中包含了反向代理的配置，SimpleRouteLocator是基于静态配置文件的路由定位器的重要实现。

```
private ZuulProperties properties;
...

    @Override
	public List<Route> getRoutes() {
		List<Route> values = new ArrayList<>();
		for (Entry<String, ZuulRoute> entry : getRoutesMap().entrySet()) {
			ZuulRoute route = entry.getValue();
			String path = route.getPath();
			values.add(getRoute(route, path));
		}
		return values;
	}

...

	@Override
	public Route getMatchingRoute(final String path) {
		return getSimpleMatchingRoute(path);

	}
	
```
　　接下来再来看看实现了刷新接口的CompositeRouteLocator与DiscoveryClientRouteLocator，DiscoveryClientRouteLocator继承了SimpleRouteLocator，同时实现了RefreshableRouteLocator。

```
public class DiscoveryClientRouteLocator extends SimpleRouteLocator
		implements RefreshableRouteLocator {
```

　　它主要增加了DiscoveryClientRouteLocator方法，用于通过DiscoveryClient（例如Eureka）发现路由信息，以及实现了动态的路由刷新接口，这里的doRefresh仅仅是调用了SimpleRouteLocator的方法。

```
	/**
	 * Calculate all the routes and set up a cache for the values. Subclasses can call
	 * this method if they need to implement {@link RefreshableRouteLocator}.
	 */
	protected void doRefresh() {
		this.routes.set(locateRoutes());
	}
	
```

　　CompositeRouteLocator，其内部维护了routeLocators的集合，并继承了RefreshableRouteLocator方法。

```
/**
 * RouteLocator that composes multiple RouteLocators.
 *
 * @author Johannes Edmeier
 *
 */
public class CompositeRouteLocator implements RefreshableRouteLocator {
	private final Collection<? extends RouteLocator> routeLocators;
	private ArrayList<RouteLocator> rl;

	public CompositeRouteLocator(Collection<? extends RouteLocator> routeLocators) {
		Assert.notNull(routeLocators, "'routeLocators' must not be null");
		rl = new ArrayList<>(routeLocators);
		AnnotationAwareOrderComparator.sort(rl);
		this.routeLocators = rl;
	}

	@Override
	public Collection<String> getIgnoredPaths() {
		List<String> ignoredPaths = new ArrayList<>();
		for (RouteLocator locator : routeLocators) {
			ignoredPaths.addAll(locator.getIgnoredPaths());
		}
		return ignoredPaths;
	}

	@Override
	public List<Route> getRoutes() {
		List<Route> route = new ArrayList<>();
		for (RouteLocator locator : routeLocators) {
			route.addAll(locator.getRoutes());
		}
		return route;
	}

	@Override
	public Route getMatchingRoute(String path) {
		for (RouteLocator locator : routeLocators) {
			Route route = locator.getMatchingRoute(path);
			if (route != null) {
				return route;
			}
		}
		return null;
	}

	@Override
	public void refresh() {
		for (RouteLocator locator : routeLocators) {
			if (locator instanceof RefreshableRouteLocator) {
				((RefreshableRouteLocator) locator).refresh();
			}
		}
	}
}
```
　　routeLocators就是我们需要的入口了，如果我们可以自定义一个通过数据库来加载的路由Locator，并自己实现refresh方法，那么就初步实现了路由动态配置最重要的一步。
　　如何将自定义Locator添加至routeLocators集合中呢？我们来梳理一下Zuul初始化的过程，在jar中我们能够找到有两个分账关键的配置类，分别是ZuulServerAutoConfiguration与ZuulProxyAutoConfiguration。
　　ZuulServerAutoConfiguration，里面初始化了CompositeRouteLocator、SimpleRouteLocator、ZuulController、各种关键默认的Filter、ZuulRefreshListener等，是我们要实现目的的关键入口。

```
　　 @Bean
	@Primary
	public CompositeRouteLocator primaryRouteLocator(
			Collection<RouteLocator> routeLocators) {
		return new CompositeRouteLocator(routeLocators);
	}

```
这里是CompositeRouteLocator初始化入口，由此可见我们完全可以将自定义的Locator加载进去。

### 自定义路由定位器与路由刷新策略

　　根据上面的方法，我们便可以将自定义的locator添加进CompositeRouteLocator中的locators集合。
　　定义一个MyDynamicRouteLocator，实现RefreshableRouteLocator，Ordered接口，其核心功能有两点，其一是将数据库中配置的路由信息加载，作为路由定位方法判定的基础，其二是实现刷新方法，使之能够识别数据库中变化的配置信息。
    
```
public class MyDynamicRouteLocator implements RefreshableRouteLocator, Ordered {
```


然后，在项目启动时初始化MyDynamicRouteLocator


```
@Configuration
public class DynamicRouteConfiguration {
    ...
    
    @Bean
    public MyDynamicRouteLocator dynamicRouteLocator() {
       ... 
    }

}

```

### 路由动态配置的优化
　　Zuul默认会维持心跳每隔30秒调用一次refresh方法来刷新路由信息，如果路由配置发生变更时，我们希望它可以实时生效，如何做呢？
　　前面提到ZuulServerAutoConfiguration内部初始化了一个ZuulRefreshListener，它实现了Spring的ApplicationListener接口，当它监听到一些指定的Event时，便最终会调用refresh方法来刷新路由配置信息。

```
	private static class ZuulRefreshListener
			implements ApplicationListener<ApplicationEvent> {

		@Autowired
		private ZuulHandlerMapping zuulHandlerMapping;

		private HeartbeatMonitor heartbeatMonitor = new HeartbeatMonitor();

		@Override
		public void onApplicationEvent(ApplicationEvent event) {
			if (event instanceof ContextRefreshedEvent
					|| event instanceof RefreshScopeRefreshedEvent
					|| event instanceof RoutesRefreshedEvent
					|| event instanceof InstanceRegisteredEvent) {
				reset();
			}
			else if (event instanceof ParentHeartbeatEvent) {
				ParentHeartbeatEvent e = (ParentHeartbeatEvent) event;
				resetIfNeeded(e.getValue());
			}
			else if (event instanceof HeartbeatEvent) {
				HeartbeatEvent e = (HeartbeatEvent) event;
				resetIfNeeded(e.getValue());
			}
		}

		private void resetIfNeeded(Object value) {
			if (this.heartbeatMonitor.update(value)) {
				reset();
			}
		}

		private void reset() {
			this.zuulHandlerMapping.setDirty(true);
		}
	}
```

　　因此，我们可以通过广播-订阅的方式，建立一个Zuul的管理后台，当我们认为将路由配置信息变更时，发送一个广播，在Zuul集群的服务中实时监听该消息，当监听到刷新请求后，通过ApplicationEventPublisher发布RoutesRefreshedEvent，Zuul的事件监听器会自动为我们处理后续流程，从而实现了配置修改的实时刷新。
　　
![](/images/15397653100461/15398449182455.jpg)

核心方法

```
    /**
     * 取得订阅的消息后的处理
     *
     * 说明：当接收到路由规则刷新通知时, 发布路由刷新事件
     * @param channel
     * @param message
     */
    @Override
    public void onMessage(String channel, String message) {
        // step. 发布路由刷新事件
        this.publisher.publishEvent(new RoutesRefreshedEvent(compositeRouteLocator));
    }
```

---

## Zuul与非Spring Boot项目集成

　　前面介绍过，Zuul的动态路由与负载均衡离不开Spring Cloud的Discovery Client（例如Eureka）与Ribbon，对于Spring Boot项目来说，其接入成本即引入一个jar包，添加几行配置与注解即可与Zuul集成。
　　对于非Spring Boot项目，我们需要自己封装一个类似的spring-cloud-starter-netflix-eureka-client组件。


### 自定义非Spring Boot项目spring-cloud-starter-netflix-eureka-client组件
　　参考官方starter，该组件内部的核心是eureka-client.jar，核心逻辑很简单，即项目启动时将项目的地址、Host等信息注册至Eureka中，并开启Eureka的心跳机制，以便监听服务状态实时更新注册中心。

#### 首先引入eureka-client
引入版本需与spring-cloud-starter-netflix-eureka-client所用一致

```
<!-- Eureka -->
<dependency>
    <groupId>com.netflix.eureka</groupId>
    <artifactId>eureka-client</artifactId>
    <version>${eureka.version}</version>
    <exclusions>
        <exclusion>
            <groupId>javax.servlet</groupId>
            <artifactId>servlet-api</artifactId>
        </exclusion>
    </exclusions>
</dependency>
            
```
　　这里需要注意一点，由于旧项目所用servlet-api版本不同，因此需要注意组件的依赖关系。

#### 定义Listener
Listener必须具有以下功能：

* 项目启动时服务信息注册到Eureka，并提供心跳监测　　
* 项目销毁时注销Eureka
* 维持心跳监测

```
/**
 * 用途：Eureka服务注册
 * 说明：
 *      1、项目启动时服务信息注册到Eureka, 并提供心跳监测
 *      2、项目销毁时注销Eureka
 *      3、Eureka客户端默认配置的覆盖选项从Classpath下寻找eureka-client.properties中加载
 *
 * @author Wang Ran <br/>
 */
public class EurekaDiscoveryListener implements ServletContextListener {

    private static ApplicationInfoManager applicationInfoManager;
    private static EurekaClient eurekaClient;
    private static EurekaInstanceConfig instanceConfig;
    private static EurekaClientConfig clientConfig = new DefaultEurekaClientConfig();

    /**
     * 服务启动时调用, 注册Eureka
     * @param sce
     */
    @Override
    public void contextInitialized(ServletContextEvent servletContextEvent) {
        // step. 初始化Eureka Client
        try {
            final String ipAddress = EurekaClientUtils.getLocalIpAddress();
            instanceConfig = new MyDataCenterInstanceConfig() {
                /**
                 * 强制将注册到Eureka的hostName从主机名换成IP地址加端口号的形式
                 * @param refresh
                 * @return
                 */
                @Override
                public String getHostName(boolean refresh) {
                    return ipAddress;
                }

                /**
                 * 强制将Eureka上显示的实例名称初始化为ip:appname:port的形式
                 * @return
                 */
                @Override
                public String getInstanceId() {
                    return ipAddress.concat(":").concat(this.getAppname())
                            .concat(":").concat(String.valueOf(this.getNonSecurePort()));
                }
            };
        } catch (SocketException e) {
            throw new RuntimeException("Eureka Register init process occurred SocketException", e);
        }
        // 初始化Eureka ApplicationInfoManager
        initializeApplicationInfoManager(instanceConfig);
        // 初始化Eureka客户端并向Eureka注册
        initializeEurekaClient(applicationInfoManager, clientConfig);
        // 注册成功后向Eureka通知注册状态
        applicationInfoManager.setInstanceStatus(InstanceInfo.InstanceStatus.UP);
    }

    /**
     * 服务销毁时调用, 注销Eureka
     * @param servletContextEvent
     */
    @Override
    public void contextDestroyed(ServletContextEvent servletContextEvent) {
        eurekaClient.shutdown();
    }

    /**
     * 初始化Eureka ApplicationInfoManager
     * @param instanceConfig
     * @return
     */
    private static synchronized ApplicationInfoManager initializeApplicationInfoManager(
            EurekaInstanceConfig instanceConfig) {
        if (applicationInfoManager == null) {
            InstanceInfo instanceInfo = new EurekaConfigBasedInstanceInfoProvider(instanceConfig).get();
            applicationInfoManager = new ApplicationInfoManager(instanceConfig, instanceInfo);
        }

        return applicationInfoManager;
    }

    /**
     * 初始化Eureka客户端
     * @param applicationInfoManager
     * @param clientConfig
     * @return
     */
    private static synchronized EurekaClient initializeEurekaClient(
            ApplicationInfoManager applicationInfoManager, EurekaClientConfig clientConfig) {
        if (eurekaClient == null) {
            eurekaClient = new DiscoveryClient(applicationInfoManager, clientConfig);
        }
        return eurekaClient;
    }
}
```

### 非Spring Boot项目客户端接入Eureka
#### 引入自定义Client

```
<dependency>
    <groupId>com.xxx</groupId>
    <artifactId>my-spring-cloud-starter-netflix-eureka-client</artifactId>
    <version>${xxx.version}</version>
</dependency>
```

#### 在Classpath添加文件：eureka-client.properties

```
### Eureka Client configuration

# configuration related to reaching the eureka servers
# 例如：http://eureka-node1:port/eureka/,http://eureka-node2:port/eureka/
eureka.serviceUrl.default=${EUREKA_URI}
# 输入客户端服务集群的id, 名称需为英文, 例如: xxx-server
eureka.name=${EUREKA-CLIENT-ID}
# 输入客户端服务端口, 例如8080
eureka.port=${EUREKA-CLIENT-PORT}
# 向服务发现注册真实ip
eureka.instance.prefer-ip-address=true
# 心跳时间，即服务续约间隔时间（缺省为30s）
eureka.instance.lease-renewal-interval-in-seconds=5
# 发呆时间，即服务续约到期时间（缺省为90s）
eureka.instance.lease-expiration-duration-in-seconds=20
# 客户识别此服务的虚拟主机名
eureka.vipAddress=${eureka.name}
eureka.vitualVipAddress=${eureka.name}
## configuration related to reaching the eureka servers
eureka.preferSameZone=true
# 是否要使用基于DNS的查找来确定其他eureka服务器
eureka.shouldUseDns=false
# 是否注册自身到eureka
eureka.registration.enabled=true
eureka.decoderName=JacksonJson
```

#### web.xml文件引入Eureka监听器

```
<listener>
    <description>Eureka Listener</description>
    <listener-class>com.xxx.listener.EurekaDiscoveryListener</listener-class>
</listener>
    
```

---

## 分流上线 & 验证
　　前文所述，我们现有集群架构接入Zuul面临了一些上线问题：

* App端每次更新都要重新发版，时效性差
* App端接入Zuul后若出现问题，无法及时回滚
* 旧版本App无法修改接入Zuul
* 测试与线上环境不一致，上线存在未知的风险
* 我们的业务模式，无法在线上环境进行全流程测试

　　为了解决上述问题，我们必须做到在前端不进行任何修改的情况下，以一种分流且可验证的形式接入Zuul中。

### 分流方案
　　　　![Spring Cloud Zuul 02 -1-](/images/15397653100461/Spring%20Cloud%20Zuul%2002%20-1-.jpg)
　　若想做到前端在前期不做任何修改，意味着Zuul当前必须与前端已有的Nginx域名对接。
　　我们假设Zuul的域名为：my-zuul.com，旧服务1的域名为old-service-01.com，映射的机器为10.10.10.1，10.10.10.2。
　　对于分流方案，我们可以直接以Nginx的轮询策略为基准，将my-zuul.com作为类似10.10.10.1的一个节点，通过权重的形式将一部分访问old-service-01.com的流量分发到10.10.10.1，一部分流量分发到my-zuul.com。
　　但是这里存在一个问题，前端需要将old-service-01.com域名替换为my-zuul.com/service-01，而Nginx的upstream在映射到zuul.com时无法为域名添加/service-01，因此需要引入old-service-01-neiwang.com域名来解决这个问题。

```
　upstream old-service-01.com {
　   server 10.10.10.1:8080 fail_timeout=10s max_fails=3 weight 1;
　   ...
　   server old-service-01-neiwang.com weight 1;
　}
```
　　然后，在Nginx中针对old-service-01-neiwang.com域名进行redirect，使之转到my-zuul.com/service-01，最终的架构详见下图。

　　![Spring Cloud Zuul 03](/images/15397653100461/Spring%20Cloud%20Zuul%2003.jpg)


　　
### 验证方案

　　通过Nginx我们实现了分流的上线策略，那么便可通过新增Post类型的自定义过滤器，拦截服务之间的请求与响应关键信息，进而通过streaming实时分析日志记录，监测Zuul的访问情况与业务数据的指标范围，来确定上线是否存在异常情形，进而可以逐步提升Zuul在Nginx的分流权重，直至最终将老架构过渡到以Zuul为核心的最终方案。

![](/images/15397653100461/15398532205321.jpg)
![](/images/15397653100461/15398533229844.jpg)

---

## 总结
　　本文主要与大家分享一下我们在使用Spring Cloud中关于Zuul的一些实践心得，鉴于作者水平有限，文章中不免会有理解不到位的地方。关于Zuul本身与其他Spring Cloud组件，以及其他微服务架构有非常多的地方值得深入探讨，若您对文章中某些问题存在疑问，或是发现某些内容存在错误、有更好的解决方案、意见以及建议等，欢迎留言，非常希望能与各路大神广泛交流。
 
---
## 参考资料

[Spring Cloud官方文档
](http://cloud.spring.io/spring-cloud-static/Finchley.SR1/single/spring-cloud.html)

[Zuul Github
](https://github.com/Netflix/zuul)

