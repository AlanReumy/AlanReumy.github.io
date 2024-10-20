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

此外，你还可以自定义日志打印的方式，定义一个实现 `LoggerService` 接口的类：

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

在创建应用时指定这个 `logger`：

![image.png](https://codertzm.oss-cn-chengdu.aliyuncs.com/20241020141006.png)

我们可以单独搞一个模块来放 `Logger`，添加 `@Injectable()` 装饰器，代表这是一个 `provider`，并且要在 `Module` 里引入：

```ts
import { Inject } from '@nestjs/common';
import { ConsoleLogger, Injectable } from '@nestjs/common';
import { AppService } from './app.service';

@Injectable()
export class MyLogger extends ConsoleLogger {
  @Inject(AppService)
  private appService: AppService;

  log(message, context) {
    console.log(this.appService.getHello());
    console.log(`[${context}]`, message);
    console.log('--------------')
  }
}
```

```ts
import { Global, Module } from '@nestjs/common';
import { MyLogger } from 'src/MyLogger';

@Global()
@Module({
  providers: [MyLogger],
  exports: [MyLogger]
})

export class LoggerModule { }
```

在 AppService 里注入下：

```ts
import { Inject, Injectable } from '@nestjs/common';
import { MyLogger } from './logger2/MyLogger';

@Injectable()
export class AppService {

  @Inject(MyLogger)
  private logger: MyLogger;

  getHello(): string {
    this.logger.log('yyy');
    
    return 'Hello World!';
  }
}
```

### 动态配置

也可以声明一个动态模块，每次 imports 的时候配置下：

```ts
import { DynamicModule, Global, Module } from '@nestjs/common';
import { MyLogger } from './MyLogger';

@Module({})
export class Logger2Module{
    static register(options): DynamicModule {
        return {
            module: Logger2Module,
            providers: [
                MyLogger, 
                {
                    provide: 'LOG_OPTIONS',
                    useValue: options
                }
            ],
            exports: [MyLogger, 'LOG_OPTIONS']
        }
    }
}
```

```ts
import { Inject } from '@nestjs/common';
import { ConsoleLogger, Injectable } from '@nestjs/common';
import { AppService } from './app.service';

@Injectable()
export class MyLogger extends ConsoleLogger {
  @Inject(AppService)
  private appService: AppService;

  @Inject('LOG_OPTIONS')
  private logOptions: Record<string, any>

  log(message, context) {
    console.log(this.logOptions);
    console.log(this.appService.getHello());
    console.log(`[${context}]`, message);
    console.log('--------------')
  }

}
```

每次 imports 的时候传入不同的配置：

![](https://codertzm.oss-cn-chengdu.aliyuncs.com/20241020141908.png)

## 使用 winston 打印日志

打印日志需要有的功能：

- 写入文件或数据库
- 分级别
- 带上时间戳、代码位置等信息

### winston 基本使用

```js
import winston from 'winston';

const logger = winston.createLogger({
    level: 'debug',
    format: winston.format.simple(),
    transports: [
        new winston.transports.Console(),
        new winston.transports.File({ 
            dirname: 'log', filename: 'test.log' 
        }),
    ]
});

logger.info('xxx');
logger.error('xxxx');
logger.debug(66666666);
```

### 文件大小分割

`winston` 支持按照大小自动分割文件，这样就不用担心所有日志都写在一个文件里，那这个文件最终会特别大：

![image.png](https://codertzm.oss-cn-chengdu.aliyuncs.com/20241020142951.png)

### 日期分割

一般日志都是按照日期自动分割的，比如 2024-10-20 的日志文件，2024-10-20 的日志文件，这样之后也好管理。

winston 也是支持的，不过要换别的 `Transport` 。

![image.png](https://codertzm.oss-cn-chengdu.aliyuncs.com/20241020143233.png)

`Console`、`File`、`Http`、`Stream` 这几个 `Transport` 是内置的。

我们可以使用这里的 `DailyRotateFile` 就是按照日期滚动存储到日志文件的 `Transport`。

```js
import winston from 'winston';
import 'winston-daily-rotate-file';

const logger = winston.createLogger({
    level: 'debug',
    format: winston.format.simple(),
    transports: [
        new winston.transports.Console(),
        new winston.transports.DailyRotateFile({
            level: 'info',
            dirname: 'log2',
            filename: 'test-%DATE%.log',
            datePattern: 'YYYY-MM-DD-HH-mm',
            maxSize: '1k'
        })
    ]
});

logger.info('xxx');
logger.error('xxxx');
logger.debug(66666666);
```

### 格式化

日志可以通过 format 指定格式，比如：simple、json、prettyPrint、label、colorize 等，也可以使用 combine 让他们相结合

```js
const logger = winston.createLogger({
  level: "debug",
  format: winston.format.combine(
    winston.format.simple(),
    winston.format.json()
  ),
  transports: [
    new winston.transports.Console(),
    new winston.transports.DailyRotateFile({
      level: "info",
      dirname: "log2",
      filename: "test-%DATE%.log",
      datePattern: "YYYY-MM-DD-HH-mm",
      maxSize: "1k",
    }),
  ],
});
```

如果不同的 transport 需要指定不同的格式，也可以给每个 transport 单独指定 format 

### 配置拆分

如果我有的日志只想 console，而有的日志希望写入文件，而且配置都不同，我们可以创建多个 logger 实例，每个 logger 实例有不同的 format、transport、level 等配置：

```js
import winston from 'winston';

winston.loggers.add('console', {
    format: winston.format.combine(
        winston.format.colorize(),
        winston.format.simple()
    ),
    transports: [
        new winston.transports.Console()
    ]
});

winston.loggers.add('file', {
    format:winston.format.combine(
        winston.format.timestamp(),
        winston.format.json()
    ),
    transports: [
        new winston.transports.File({
            dirname: 'log4',
            filename: 'test.log',
            format: winston.format.json()
        })
    ]
});


const logger1 = winston.loggers.get('console');

logger1.info('aaaaa');
logger1.error('bbbbb');

const logger2 = winston.loggers.get('file');

logger2.info('xxxx');
logger2.info('yyyy');
```

### 指定未捕获的错误日志

winston 还支持指定如何处理未捕获的错误的日志

```js
import winston from 'winston';

const logger = winston.createLogger({
    level: 'debug',
    format: winston.format.simple(),
    transports: [
        new winston.transports.Console()
    ],
    exceptionHandlers: [
        new winston.transports.File({
            filename: 'error.log'
        })
    ]
});

throw new Error('xxx');

logger.info('xxx');
logger.error('xxxx');
logger.debug(66666666);
```

除了 error 外，Promise 的未捕获异常也可以指定如何处理日志：

```js
import winston from 'winston';

const logger = winston.createLogger({
    level: 'debug',
    format: winston.format.simple(),
    transports: [
        new winston.transports.Console()
    ],
    rejectionHandlers: [
        new winston.transports.File({
            filename: 'rejection.log'
        })
    ]
});

(async function(){
    throw Error('yyy');
})();

logger.info('xxx');
logger.error('xxxx');
logger.debug(66666666);
```