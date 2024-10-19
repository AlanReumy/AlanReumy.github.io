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

![image.png](https://codertzm.oss-cn-chengdu.aliyuncs.com/20241019115747.png)

## 上传方案

有了 OSS 服务之后，我们上传文件还需要经过应用服务器么？

可以经过也可以不经过。

如果经过应用服务器，那就要客户端上传文件之后，我们在服务里接受文件，上传 OSS：

![image.png](https://codertzm.oss-cn-chengdu.aliyuncs.com/20241019120056.png)

这样当然是可以的，还能保护 accessKey 不被人窃取。

只是会浪费应用服务器的流量。

那如果不经过呢？

![image.png](https://codertzm.oss-cn-chengdu.aliyuncs.com/20241019120109.png)
在客户端用 accessKey 把文件传到 OSS，之后把 URL 传给应用服务器就好了。

这样减少了应用服务器的流量消耗，但是增加了 accessKey 暴露的风险。

各有各的坏处。

那有没有啥两全其美的办法呢？

有。

![image.png](https://codertzm.oss-cn-chengdu.aliyuncs.com/20241019120136.png)


它给出的解决方案就是生成一个临时的签名来用，代码是这样的：

```js
const OSS = require('ali-oss')

async function main() {

    const config = {
        region: 'oss-cn-beijing',
        bucket: 'guang-333',
        accessKeyId: '',
        accessKeySecret: '',
    }

    const client = new OSS(config);
    
    const date = new Date();
    
    date.setDate(date.getDate() + 1);
    
    const res = client.calculatePostSignature({
        expiration: date.toISOString(),
        conditions: [
            ["content-length-range", 0, 1048576000], //设置上传文件的大小限制。      
        ]
    });
    
    console.log(res);
    
    const location = await client.getBucketLocation();
    
    const host = `http://${config.bucket}.${location.location}.aliyuncs.com`;

    console.log(host);
}

main();
```

前端：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <script src="https://unpkg.com/axios@1.6.5/dist/axios.min.js"></script>
</head>
<body>
    <input id="fileInput" type="file"/>
    
    <script>
        const fileInput = document.getElementById('fileInput');

        async function getOSSInfo() {
            await '请求应用服务器拿到临时凭证';
            return {
                OSSAccessKeyId: '',
                Signature: '',
                policy: 'eyJleHBpcmF0aW9uIjoiMjAyNC0wMS0yMFQwMzoyNjowOC4xMDZaIiwiY29uZGl0aW9ucyI6W1siY29udGVudC1sZW5ndGgtcmFuZ2UiLDAsMTA0ODU3NjAwMF1dfQ==',
                host: 'http://guang-333.oss-cn-beijing.aliyuncs.com'
            }
        }

        fileInput.onchange = async () => {
            const file = fileInput.files[0];

            const ossInfo = await getOSSInfo();


            const formdata = new FormData()
 
            formdata.append('key', file.name);
            formdata.append('OSSAccessKeyId', ossInfo.OSSAccessKeyId)
            formdata.append('policy', ossInfo.policy)
            formdata.append('signature', ossInfo.Signature)
            formdata.append('success_action_status', '200')
            formdata.append('file', file)

            const res = await axios.post(ossInfo.host, formdata);
            if(res.status === 200) {
                
                const img = document.createElement('img');
                img.src = ossInfo.host + '/' + file.name
                document.body.append(img);

                alert('上传成功');
            }
        }
    </script>
</body>
</html>

```

这里 getOSSInfo 应该是请求服务端的接口，拿到刚才我们控制台输出的那些东西。

这里就简化下，直接写死在代码里了。

这就是完美的 OSS 上传方案。

服务端用 RAM 子用户的 accessKey 来生成临时签名，然后返回给客户端，客户端用这个来直传文件到 OSS。

因为临时的签名过期时间很短，我们设置的是一天，所以暴露的风险也不大。

这样服务端就根本没有接受文件的压力，只要等客户端上传完之后，带上 URL 就好了。