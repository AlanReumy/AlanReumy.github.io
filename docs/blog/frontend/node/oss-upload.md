# OSS 上传方案

文件上传是常见需求，一般我们不会把文件直接上传到应用服务器，因为单台服务器存储空间是有限的，不好扩展。我们会用单独的 OSS （Object Storage Service）对象存储服务来上传下载文件，比如一般会买阿里云的 OSS 服务。

对象存储就是 key-value 存储，分布式的方式实现的，存储容量无限。

我们本地文件存储是目录-文件的组织方式：
![image.png](https://codertzm.oss-cn-chengdu.aliyuncs.com/20241019114812.png)

而 OSS 服务的存储结构是这样的（没有目录层级结构，阿里云控制台的目录是模拟实现的）：
![image.png](https://codertzm.oss-cn-chengdu.aliyuncs.com/20241019114830.png)
Object 会存储 id、文件内容、元数据三部分信息：
![image.png](https://codertzm.oss-cn-chengdu.aliyuncs.com/20241019114933.png)

## ali-oss实现上传

```JS
const OSS = require('ali-oss')

const client = new OSS({
    region: 'oss-cn-beijing',
    bucket: 'guang-333',
    accessKeyId: '',
    accessKeySecret: '',
});

async function put () {
  try {
    const result = await client.put('cat.png', './mao.png');
    console.log(result);
  } catch (e) {
    console.log(e);
  }
}

put();
```

这里的 accessKeyId 和 acessKeySecret 是什么呢？

本来我们身份认证都是通过用户名密码：

![image.png](https://codertzm.oss-cn-chengdu.aliyuncs.com/20241019115138.png)
但这样不够安全，所以我们创建了 accessKey 用来代表身份，用它来做身份认证，就算泄漏了，也不影响别的：

![image.png](https://codertzm.oss-cn-chengdu.aliyuncs.com/20241019115201.png)

我们不要直接用 accessKey，而是创建一个子用户再创建 accessKey。

![image.png](https://codertzm.oss-cn-chengdu.aliyuncs.com/20241019115323.png)

然后用这个 accessKey 的 id 和 secret，但你直接换上它还不行，会提示你 403，没有权限。此时需要在访问控制中配置一下权限：![](https://codertzm.oss-cn-chengdu.aliyuncs.com/20241019115555.png)

### 安全性

回过头来看下，不得不说阿里云在安全这一块设计的就很巧妙。

如果我们直接用用户名密码验证呢：

![image.png](https://codertzm.oss-cn-chengdu.aliyuncs.com/20241019115654.png)

那万一泄漏了不就完蛋了么？

但是如果创建个 accessKey 用它来做身份认证，就算泄漏了也可以禁用：

![image.png](https://codertzm.oss-cn-chengdu.aliyuncs.com/20241019115705.png)

![image.png](https://codertzm.oss-cn-chengdu.aliyuncs.com/20241019115713.png)

再进一步，直接用这个 accessKey 它是有所有权限的。

我们先创建个 RAM 子用户，再分配给他某些权限，这样就算泄漏了，是不是能做的事情就更少了？

当然就更安全。