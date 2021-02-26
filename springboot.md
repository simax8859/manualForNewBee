**回顾springboot中一些注意的点**

1.公司内网中部署Spring Iniializr ，用idea --》Spring Initializr --》Custom 填入内网地址。

2.启动项目之前，确保构建项目成功再启动

```properties
mvn clean install
```

3.springboot actuator端点 /actuator

4.yml和properties的解析机制的不同

在需要设置 * 代表全部的时候 yml的机制使用需要加引号    ‘*’

如果需要保证参数顺序的话，推荐使用yml

5.使用连字符来区分不同的项目环境

spring.profiles.active 参数用以指定具体哪个环境



最佳实践：简单就是美





微服务拆分--个人心得

- 按照职责划分：明确业务边界。
- 按照通用性划分，通用性功能做成微服务。



数据建模软件：mysql workbench  支持导出成sql语句

