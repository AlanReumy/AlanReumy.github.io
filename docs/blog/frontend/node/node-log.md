# node 日志

## nest 中是如何打印日志的

```ts
import { ConsoleLogger, Controller, Get, Logger } from '@nestjs/common';
import { AppService } from './app.service';

@Controller()
export class AppController {
  private logger = new Logger();

  constructor(private readonly appService: AppService) {}

  @Get()
  getHello(): string {
    this.logger.debug('aaa', AppController.name);
    this.logger.error('bbb', AppController.name);
    this.logger.log('ccc', AppController.name);
    this.logger.verbose('ddd', AppController.name);
    this.logger.warn('eee', AppController.name);
    
    return this.appService.getHello();
  }
}
```

![image.png](https://codertzm.oss-cn-chengdu.aliyuncs.com/20241020140600.png)

这里的 `verbose`、`debug`、`log`、`warn`、`error` 就是日志级别，而 `[]` 中的是 `context`，也就是当前所在的上下文，最后是日志的内容。

这个日志是受 `nest` 控制的，可以在创建应用的时候指定是否开启，设置为 `false` 之后就没有日志了：

![image.png](https://codertzm.oss-cn-chengdu.aliyuncs.com/20241020140648.png)

![image.png](https://codertzm.oss-cn-chengdu.aliyuncs.com/20241020140713.png)

也可以自己决定输出什么级别的日志：

![image.png](https://codertzm.oss-cn-chengdu.aliyuncs.com/20241020140822.png)

此外，你还可以自定义日志打印的方式，定义一个实现 LoggerService 接口的类：

```ts
import { LoggerService, LogLevel } from '@nestjs/common';

export class MyLogger implements LoggerService {
    log(message: string, context: string) {
        console.log(`---log---[${context}]---`, message)
    }

    error(message: string, context: string) {
        console.log(`---error---[${context}]---`, message)
    }

    warn(message: string, context: string) {
        console.log(`---warn---[${context}]---`, message)
    }
}
```

在创建应用时指定这个 logger：

![image.png](https://codertzm.oss-cn-chengdu.aliyuncs.com/20241020141006.png)

也可以不自己实现 LoggerService 的全部方法，而是继承 ConsoleLogger，重写一些方法：

```ts
import { ConsoleLogger } from '@nestjs/common';

export class MyLogger2 extends ConsoleLogger{
    log(message: string, context: string) {
        console.log(`[${context}]`,message)
    }
}
```