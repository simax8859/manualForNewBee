- ###### 常见的容错方案：
  
  - 超时
  - 限流
  - 仓壁模式：利用不同的线程池将业务请求进行划分，互不影响
- 断路器模式
  
- Sentinel：轻量级流量控制，熔断降级Java库

- 如何使用：添加依赖 `spring-cloud-starter-alibaba-sentinel`

- 如何验证：添加依赖 `spring-boot-starter-actuator`，启动项目并访问`/actuator/sentinel`接口



- Sentinel 控制台  github下载jar包，注意与依赖包core版本进行对应

  - 项目中增加配置参数

  ```yml
    cloud:
      nacos:
        discovery:
          server-addr: localhost:8848
          cluster-name: beijing
      # sentinel 配置
      sentinel:
        transport:
          # 指定sentinel控制台的访问路径
          dashboard: localhost:8080
  ```

  - 启动控制台，启动项目，任意访问一个接口，刷新控制台。观察实时监控发现请求信息。

  解释：sentinel与ribbon默认都为懒加载

  

- ###### 流控规则

  - 新增控制规则
    - 资源名只是一个名称，默认为请求路径
    - 针对来源，可指定微服务（名称），需要做扩展。默认全部default
    - 保存规则即生效
    - 关联：当关联资源达到阈值，限流自己。比如：需要优先修改，限制查询。可以在查询接口设置关联修改接口，当修改接口阈值到达一定数值则对查询进行限制
    - 链路：只记录指定链路上的流量
      - 注解`@SentinelResource("[name]")`添加在具体调用的方法上（`@Service`下），该方法可被多处入口方法（`@Controller`下）调用
      - 在蔟点链路处选择对指定入口资源下“[name]”设定控制规则选择链路，设置入口资源参数为指定请求地址，实现流控（ 细粒度针对来源的一种方式，API）
    - 流控效果
      - 直接失败：抛出异常
      - `Warm up`：可以让允许增加的流量缓慢的增加
      - 排队等待：匀速排队的方式，阈值设置必须为QPS的方式，设置超时时间，按QPS设置进行请求处理，其他请求进行等待，超过超时时间则丢弃



- ###### 降级规则

  `com.alibaba.csp.sentinel.slots.block.degrade.DegradeRule#passCheck 源码`

  - RT：平均响应时间（秒级）超过阈值RT并且在时间窗口（秒）内大于等于5次，则会打开断路器，直到时间窗口结束并且重新关闭断路器。

    注意：最大RT 设置为4900ms，可以通过

    `-Dcsp.sentinel.statistic.max.rt=XXX  修改`

  - 异常比例：QPS >=5 && 异常比例（秒级统计）超过阈值，触发降级。时间窗口结束，关闭降级。

  - 异常数（分钟统计）超过阈值，触发降级。时间窗口结束，关闭降级。

    注意：异常窗口小于60s可能出问题，因为异常数统计是按分钟级别。



- ###### 热点规则（对指定参数，指定参数值进行限流）

  `com.alibaba.csp.sentinel.slots.block.flow.param.ParamFlowChecker#passCheck`

  - 带有参数的方法，可以对某参数进行细粒度的阈值控制，带有该参数的请求会按照配置规则进行限流处理。
  - 高级选项：可以配置指定类型指定参数值独有的限流阈值，该值按新设阈值作为规则。
  - 注意：参数必须是基本类型或者String



- ###### 系统保护规则

  - load：对1分钟**load限制**，uptime命令可以查询机器load（分别输出1分钟，5分钟，15分钟平均load），同时**并发线程数**超过系统容量时触发，建议设置为CPU核心数2.5。*仅linux/unix有效。*

  `com.alibaba.csp.sentinel.slots.system.SystemRuleManager#checkBbr`

  - RT:所有入口流量的平均RT达到阈值触发
  - 线程数：所有入口流量的并发线程数达到阈值触发
  - 入口QPS：所有入口流量的QPS达到阈值触发

  `com.alibaba.csp.sentinel.slots.system.SystemRuleManager#checkSystem`



- ###### 代码配置规则

https://www.imooc.com/article/289345



- ###### 通信：

  - 实现一套服务发现机制，各个微服务中sentinel客户端注册到sentinel控制台
  - 控制台调用各微服务客户端的API 可以访问 url:8720/api获取信息



- ###### sentinel API

  ```yml
      # 测试时可关闭保护 避免干扰
      sentinel:
        filter:
          # 关闭掉对Spring MVC 端点的保护 需要手动添加资源
          enabled: false
  ```

  - `SphU.ebtry("[name]")` 该方法可实现添加资源

  - 需要注意，sentinel对于异常的判断并不是对全部异常，而是对`BlockException`及其子类型才做统计。其他异常即使抛出也不会被统计。

    处理方式：在捕获异常的同时，手动统计`Tracer.trace(e2);`

  - `ContextUtil.enter` 标记来源，实现流控。



- ###### @SentinelResource

  www.imooc.com/article/289384

  - 注解方式实现基本API功能，但不可设置来源