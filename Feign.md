- 如何使用Feign

1.增加pom依赖

2.FeignClient接口，添加注解@FeignClient 指定服务提供者名称，编写具体调用方法

3.在具体使用处依赖注入FeignClient，调用接口中方法

- 自定义Feign日志级别

1.创建日志级别配置 类，类种@Bean一个 level方法，返回Logger.Lever对象，注意Logger为Feign中类。

2.在所定义的 FeignClient类注解中加上参数 configuration，值为日志级别配置类class

3.在配置文件中定义logging.lever.（FeignClient全路径名）= debug

注意：不能给这个类打上@configuration的注解，会引发父子上下文重复扫描的问题，打注解会导致这个配置被所有的Feign共享，所以如果需要打注解，移到启动类非同级及非下级目录。

属性配置方式：

```properties
feign:
  client:
    config:
      # 需要调用的微服务的名称
      user-center:
        loggerLevel: Full
```

- 配置全局日志级别
  - 代码方式：给启动类@EnableFeignClients注解增加参数defaultConfiguration 指定配置类class
  - 属性配置：不指定微服务的名称，改成default



- 支持的配置项，需要时查询

- Feign最佳实践

  - 首先需要明确，属性配置的优先级更高

  全部代码 < 全局属性 < 细粒度代码 < 细粒度属性

  - 属性配置情况可以满足大多数场景需求，线上修改灵活更加直观
  - 结论：尽量使用属性配置，同一微服务尽量使用单一方式不要混用。

- FeignClient的接口方法如果有对象内部包含多参数，Get请求需要给参数增加注解@SpringQueryMap（将无法使用Feign的继承）或者将参数全部拿出，单独使用（可使用Feign继承及Spring常规注解）Post请求不受影响，按SpringMVC进行。

- Feign调用未注册服务

  - 在注解@FeignCLient 中增加url参数，即可定向到指定地址，需要注意一定要有name参数，可以任意赋值不影响。

- Feign 与 RestTemplate 的比较

  - Feign优势：可读性、可维护性、开发体验
  - RestTemplate：性能、灵活性
  - 尽量使用Feign，杜绝使用RestTemplate。实际场景Feign在性能与灵活性 能满足99%的需求。

- Feign的性能优化

  - 为Feign配置连接池

    - pom增加依赖：feign-httpclient  性能提升大概15%

      ```yml
      # 除httpclient之外，还可以使用okhttp连接池
        httpclient:
          # 使用apache httpclient 替换默认的urlconnection
          enabled: true
          # feign的最大连接数  进一步优化可结合压测修改值
          max-connections: 200
          # feign单个路径的最大连接数
          max-connections-per-route: 50
      ```

    - 日志级别 生产环境设置为basic  开发环境可设置full

- feign的常见问题

  - www.imooc.com/article/289005

