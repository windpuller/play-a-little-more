title: Shutdown spring boot embedded jetty servlet container gracefully
tags:
  - spring-boot
  - jetty
  - gracefully shutdown
date: 2018-08-10 11:24:15
---

#### 1. 不优雅的原因
一般情况下，spring-boot服务停机时先关闭application context然后关闭jetty，两者关闭的时间差期间jetty依然再接收新的请求，而此时能够处理请求的相关资源已经在逐步关闭，导致抛出大量异常，直至jetty完全关闭。
#### 2. 优雅的步骤
spring-boot收到服务停机请求时：
(1) jetty停止接收新的请求
(2) jetty处理收到停机请求之前的请求并返回
(3) 关闭application context
#### 3. 具体实现
在application context关机事件发生时关闭jetty，`server.stop()`会实现以上(1)(2)两步的功能:
```
@Slf4j
public class GracefulShutdownBean implements ApplicationListener<ContextClosedEvent> {
 
    private static Server server;
 
    @Override
    public void onApplicationEvent(ContextClosedEvent event) {
        if (server == null) {
            log.error("Jetty server variable is null, this should not happen!");
            return;
        }
        log.info("Entering shutdown for Jetty.");
        if (!(server.getHandler() instanceof StatisticsHandler)) {
            log.error("Root handler is not StatisticsHandler, graceful shutdown may not work at all!");
        } else {
            log.info("Active requests: " + ((StatisticsHandler) server.getHandler()).getRequestsActive());
        }
        try {
            long begin = System.currentTimeMillis();
            server.stop();
            log.info("Shutdown took " + (System.currentTimeMillis() - begin) + " ms.");
        } catch (Exception e) {
            log.error("Fail to shutdown gracefully.", e);
        }
    }
 
    public static void setServer(Server server) {
        GracefulShutdownBean.server = server;
    }
}
```

将`GracefulShutdownBean`声明为单例bean，交给spring管理:
```
@Bean
public GracefulShutdownBean initGracefulShutdownBean() {
    return new GracefulShutdownBean();
}
```

将jetty的server交给GracefulShutdownBean管理：
```
@Bean
JettyServletWebServerFactory embeddedServletContainerFactory(@Value("${server.port}") int port,
                                                             @Value("${server.jetty.threadPool.maxThreads}") int maxThreads,
                                                             @Value("${server.jetty.threadPool.minThreads}") int minThreads,
                                                             @Value("${server.jetty.threadPool.idleTimeout}") int idleTimeout,
                                                             @Value("${server.jetty.shutdown.waitTime}") int shutdownWaitTime) {

    JettyServletWebServerFactory factory = new JettyServletWebServerFactory(port);

    factory.addServerCustomizers(server -> {

        QueuedThreadPool threadPool = server.getBean(QueuedThreadPool.class);
        threadPool.setMaxThreads(maxThreads);
        threadPool.setMinThreads(minThreads);
        threadPool.setIdleTimeout(idleTimeout);

        GracefulShutdownBean.setServer(server);

        if (shutdownWaitTime > 0) {
            StatisticsHandler handler = new StatisticsHandler();
            handler.setHandler(server.getHandler());
            server.setHandler(handler);
            log.info("Shutdown wait time: " + shutdownWaitTime + "ms");
            server.setStopTimeout(shutdownWaitTime);
            // We will stop it through GracefulShutdownBean class.
            server.setStopAtShutdown(false);
        }
    });

    return factory;
}
```

#### 参考文献：
1. https://github.com/spring-projects/spring-boot/issues/4657