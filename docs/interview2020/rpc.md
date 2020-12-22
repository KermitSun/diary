# Dubbo
## 应用重启，dubbo调用超时，如何应对上线不平滑问题？ 
[https://cloud.tencent.com/developer/article/1518747](https://cloud.tencent.com/developer/article/1518747) 
dubbo支持优雅停机  
Java 应用有 JVM shutdown hook 这样的概念，通过Runtime注册ShutdownHook实现

    public abstract class AbstractConfig implements Serializable {
        static {
            Runtime.getRuntime().addShutdownHook(new Thread(new Runnable() {
                public void run() {
                    ProtocolConfig.destroyAll();
                }
            }, "DubboShutdownHook"));
        }
    }
   
逻辑说明：

    Registry 注销   
        等待 -Ddubbo.service.shutdown.wait 秒，等待消费方收到下线通知  
    Protocol 注销  
        DubboProtocol 注销  
            NettyServer 注销  
                等待处理中的请求完毕  
                停止发送心跳  
                关闭 Netty 相关资源  
            NettyClient 注销  
                停止发送心跳  
                等待处理中的请求完毕  
                关闭 Netty 相关资源  

Spring 如此受欢迎的原因之一便是它的扩展点非常丰富，例如它提供了 ApplicationListener 接口，开发者可以实现这个接口监听到 Spring 容器的关闭事件，为解决 shutdown hook 并发执行的问题，在 Dubbo 2.6.3 中新增了 ShutdownHookListener 类，用作 Spring 容器下的关闭 Dubbo 应用的钩子。
    
    private static class ShutdownHookListener implements ApplicationListener {
            @Override
            public void onApplicationEvent(ApplicationEvent event) {
                if (event instanceof ContextClosedEvent) {
                    // we call it anyway since dubbo shutdown hook make sure its destroyAll() is re-entrant.
                    // pls. note we should not remove dubbo shutdown hook when spring framework is present, this is because
                    // its shutdown hook may not be installed.
                    DubboShutdownHook shutdownHook = DubboShutdownHook.getDubboShutdownHook();
                    shutdownHook.destroyAll();
                }
            }
        }

当服务提供者 ServiceBean 和服务消费者 ReferenceBean 被初始化时，会触发该钩子被创建。
# Nacos
