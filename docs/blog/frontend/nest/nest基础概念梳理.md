![image.png](https://codertzm.oss-cn-chengdu.aliyuncs.com/20240806223955.png)
**controller**：控制器，用于处理路由，解析请求参数

**handler**：控制器里处理路由的方法

**service**：实现业务逻辑的地方，比如操作数据库等

**dto**：data transfer object，数据传输对象，用于封装请求体里数据的对象

**module**：模块，包含 controller、service 等，比如用户模块、书籍模块

**entity**：对应数据库表的实体

**ioc**：Inverse of Controller，反转控制或者叫依赖注入，只要声明依赖，运行时 Nest 会自动注入依赖的实例

**aop**：Aspect Oriented Programming 面向切面编程，在多个请求响应流程中可以复用的逻辑，比如日志记录等，具体包含 middleware、interceotor、guard、exception filter、pipe

**nest cli**：创建项目、创建模块、创建 controller、创建 service 等都可以用这个 cli 工具来做