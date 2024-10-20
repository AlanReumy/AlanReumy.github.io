# node 如何发邮件

除了微信外，邮件也是我们常用的通讯方式，大多数人用邮箱客户端发邮件，比如qq邮箱，但是这样体验并不好。

比如：

1. 写邮件的时候：我有个漂亮的 `html` 页面，想直接把它作为邮件内容，或者我想用 `markdown` 来写邮件，但是它只支持富文本编辑器
2. 收邮件的时候：我想把一些重要邮件的内容保存下来，附件啥的都下载到本地。但是邮件多了的话，一个个手动下载太麻烦了。

我们可以使用 `node` 来用代码的方式实现收发邮件

## 邮件协议

邮件有专门的协议：

- 发邮件用 `SMTP` 协议。
- 收邮件用 `POP3` 协议、或者 `IMAP` 协议。

在 `node` 里也有对应的包，发邮件用 `nodemailer` 包，收邮件用 `imap` 包。

## 实践

首先，要开启 smtp、imap 等服务，这里以 qq 邮箱举例，在邮箱帮助中心 [service.mail.qq.com/](https://link.juejin.cn/?target=https%3A%2F%2Fservice.mail.qq.com%2F "https://service.mail.qq.com/") 可以搜到如何开启 smtp、imap 等服务

![image.png](https://codertzm.oss-cn-chengdu.aliyuncs.com/20241020132812.png)

开启后，然后在帮助中心页面搜索授权码

![image.png](https://codertzm.oss-cn-chengdu.aliyuncs.com/20241020133052.png)
发邮件代码：

```JS
const nodemailer = require("nodemailer");

const transporter = nodemailer.createTransport({
    host: "smtp.qq.com",
    port: 587,
    secure: false,
    auth: {
        user: 'xxxxx@qq.com',
        pass: '你的授权码'
    },
});

async function main() {
  const info = await transporter.sendMail({
    from: '"xxxxx@xx.com',
    to: "发到的邮箱",
    subject: "Hello", 
    text: "xxxxx"
  });

  console.log("邮件发送成功：", info.messageId);
}

main().catch(console.error);
```

收邮件代码：

```js
const Imap = require('imap');

const imap = new Imap({
    user: 'xxx@qq.com',
    password: '你的授权码',
    host: 'imap.qq.com',
    port: 993,
    tls: true
});

imap.once('ready', () => {
    imap.openBox('INBOX', true, (err) => {
        imap.search([['SEEN'], ['SINCE', new Date('2023-07-10 19:00:00').toLocaleString()]], (err, results) => {
            if (!err) {
                console.log(results);
            } else {
                throw err;
            }
        });
    });
});

imap.connect();
```

`nest` 中实现发邮件的 `service` 代码：

```ts
import { Injectable } from '@nestjs/common';
import { createTransport, Transporter} from 'nodemailer';

@Injectable()
export class EmailService {

    transporter: Transporter
    
    constructor() {
        this.transporter = createTransport({
            host: "smtp.qq.com",
            port: 587,
            secure: false,
            auth: {
                user: 'xx@xx.com',
                pass: '你的授权码'
            },
        });
    }

    async sendMail({ to, subject, html }) {
      await this.transporter.sendMail({
        from: {
          name: '系统邮件',
          address: 'xx@xx.com'
        },
        to,
        subject,
        html
      });
    }
}
```