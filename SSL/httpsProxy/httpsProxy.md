> # HTTPS导致的HTTP协议升级

依稀还记得大约两年前搞过一手Nginx加壳打造https的操作，里面涉及ws协议自动升级到wss。

如今又遇到https下http的协议自动升级成https的情况，**如果页面是https的，那么页面内容中http地址协议会自动升级到https**，最容易遇到的，也是我遇到的问题是html页面上img标签的src，图片路径是http的，但是我们页面的协议是https，导致这个图片的协议直接重http升级到https，然后图片的就访问不到了。

解决这个问题比较简单，最容易的方式就是直接把图片服务器给搞成HTTPS，然后问题就解决了，好了，就写到这儿吧？

当然，既然来了， 就要扯会儿犊子。

项目中你会遇到对接外部系统吧？那如果这个对象存储服务是对方的，你没办法控制，别个就要用http，那这个时候就得我们自己出手了，其实也很简单。

我们首先要明白的就是如何把这个http给他换成https就好了，也就是说我们页面上要通过https去访问到别人的http的对象服务器拿资源，我想正常有开发经验的人很直接就能想到走代理，哈哈，如果你看过我那篇<通过Nginx加壳打造https>的文章，我想这个问题对你来讲很简单了。

解决这个问题有很多种方式，我列举一下我处理过的几种，都是没有问题的。

```
1. nginx，nginx套壳，前端https打到nginx, nginx通过http代理到target
2. 如果是k8s, k8s增加service和自定义的endPoints(这个endPoints指向target), 通过traefik或者其他负载均衡工具直接增加新的https的域名规则，通过service, endPoints打出去
3. 前两种在target为域名时不会存在问题，但是如果是多IP的情况，就稍微有点点麻烦，但还是能解决问题。
   3.1 通用型，直接部署代理服务，类似对象存储的回源操作，内部部署一个https的代理服务，接口入参为需要代理的图片地址http://xxx/xx.jpg, 代理服务拉取图片信息，然后直接返回
   3.2 demo, https://domain.com/proxy?url=http://xxx/xx.jpg,代理服务拿到参数url,访问并返回内容
```

第三种方式虽然方便，但是需要多部署一个服务，适当选择吧，各有优势，如果对方IP固定，那我认为第二种是最简单，最快速的。我们目前也是用的这种方式。如果有自己的域名，那我任务nginx代理最简单，如果最求通用，那就第三种吧，到哪里都能用。

第一种加壳代理的我就不解释了，和之前打造https类似。

## K8S Service

这个玩意儿比较简单，不好演示得，懂的都懂，我贴个配置吧。

比如图片地址是`http://123.123.123.123:8882/xxx/xxx.png`

> k8s yaml资源文件

```yaml
apiVersion: v1
kind: Service
metadata:
  name: obj-proxy
  namespace: xxx
  labels:
    k8s-app: obj-proxy
spec:
  type: ClusterIP
  # ClusterIP: None        ##ClusterIP设置为None将会直接暴露到宿主机端口上
  ports:
  - name: port
    port: 80   ## service的集群端口
    protocol: TCP
---
apiVersion: v1
kind: Endpoints
metadata:
  name: obj-proxy
  namespace: xxx
  labels:
    k8s-app: obj-proxy
subsets:
- addresses:
  - ip: 123.123.123.123            ## 目标服务地址
    nodeName: 123.123.123.123
  ports:
  - name: port
    port: 8882            ## 目标端口
    protocol: TCP
```

traefik的规则部分

```yaml
    - kind: Rule
      match: Host(`www.inner.com`) && PathPrefix(`/`)
      services:
        - name: obj-proxy     ## 指向上面的service
          namespace: xxx      ## service所在的命名空间
          port: 3102          ## service的端口
```

我们通过内部域名https://www.inner.com打到traefik，traefik根据这个路由规则，打到叫做`obj-proxy`的service，然后service从它对应的endPoints指定的目标地址(123.123.123.123)打出去，看吧，是不是跟nignx加差不多，只是方式变了而已。

## 代理镜像构建

这里简单来操作一下第三种方式，代理服务随便写，java，go等等都可以，主要逻辑就是拿到图片的url，然后获取到图片流，返回，是不是很简单？我属实不想写这玩意儿了，用个运维老大哥写的python脚本吧(感谢某某运维老大哥)，嘻嘻。

程序代理的python脚本

`pyproxy.py`

```python
from flask import Flask, abort, Response, request
import os
import requests

app = Flask(__name__)
app.config["DEBUG"]  = False

file_obj = None

def generate(file_obj):
    while True:
        content = file_obj.raw.read(102400)
        if content:
            yield content
        else:
            break

@app.route('/proxy', methods=['GET'])
def download_file():
    try:
        url = request.args.get('url').encode('utf-8')
        print(url)
        filename = os.path.basename(url)
        file_obj = requests.get(url, stream=True)
        headers = file_obj.headers

        s = Response(generate(file_obj), content_type=headers.get('Content-Type'))

        for k, v in headers.items():
            s.headers[k] = v
        s.headers['Content-Disposition'] = "attachment; filename={}".format(filename)
        return s

    except Exception as e:
        if file_obj is not None:
            file_obj.close()
        Response.close()
        abort(404)

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8000)
```

`Dockerfile`

```dockerfile
FROM  rackspacedot/python37:32
RUN pip3 install requests flask
COPY pyproxy.py /pyproxy.py
CMD ["python3","/pyproxy.py"]
```

### 构建运行

```shell
[root@master1 pyproxy]# docker build -t pyproxy:v1 .
[root@master1 pyproxy]# docker run -p 8000:8000 pyproxy:v1
 * Serving Flask app 'pyproxy' (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on all addresses.
   WARNING: This is a development server. Do not use it in a production deployment.
 * Running on http://172.17.0.2:8000/ (Press CTRL+C to quit)

```

我就不去搭建https了，验证一下这个服务正常吧。转发一张OSChina的图。

`test.html`

```
...
<img src="http://192.168.2.14:8000/proxy?url=https://oscimg.oschina.net/oscnet/up-3d152f8f91bd03bc979d17d6d398779c8b1.png"/>
...
```

图太占空间就不贴了，没有问题的，已验证。
