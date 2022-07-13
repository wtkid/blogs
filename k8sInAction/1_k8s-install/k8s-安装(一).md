> # k8s安装(一)

## Docker单独安装步骤

**装k8s不走这步，这里只是记录一下，直接用sealos或者minikube，一步到位**

《kubernetes in action》章节2.1.1，[官方安装文档](http://docs.docker.corn/engine/installation/)

```shell
## 安装yum-util
yum install -y yum-utils

## 添加yum源
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

## 安装docker
yum install docker-ce docker-ce-cli containerd.io

##启动docker守护进程
systemctl start docker

## 测试
docker run hello-world
```

## K8S安装

> 基于虚拟机多节点安装，minikube好使的可以直接上minikube，方便快捷

### 一些URL

```
https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
https://developer.aliyun.com/article/763983
https://www.sealyun.com/instructions
http://mirrors.aliyun.com/centos/8.3.2011/isos/x86_64/
```


```shell
# 参考 https://www.sealyun.com/instructions

# 下载安装工具
$ wget -c https://sealyun.oss-cn-beijing.aliyuncs.com/latest/sealos && chmod +x sealos && mv sealos /usr/bin

# 安装
$ sealos init --passwd 'root' --master 192.168.2.14 --node 192.168.2.15 --pkg-url /root/kube1.19.0.tar.gz --version v1.19.0
```

正常情况下安装完之后所有节点都是Ready状态。

安装过程中建议给虚拟机创建一个还原点，当你安装失败时，直接退回还原点，无残留，进行第二次尝试。

本人有幸经历过好多次NotReady，好多+1次才好。

```shell
[root@master1 ~]# kubectl get node
NAME             STATUS   ROLES    AGE   VERSION
master1.wt.com   Ready    master   27d   v1.19.0
node1.wt.com     Ready    <none>   27d   v1.19.0
[root@master1 ~]# kubectl get po -A
NAMESPACE     NAME                                       READY   STATUS             RESTARTS   AGE
kube-system   calico-kube-controllers-69b47f4dfb-sdtgn   1/1     Running            2          34h
kube-system   calico-kube-controllers-69b47f4dfb-zqzlb   0/1     NodeAffinity       0          34h
kube-system   calico-node-hgk74                          1/1     Running            2          34h
kube-system   calico-node-ktd2x                          1/1     Running            2          34h
kube-system   coredns-f9fd979d6-77hfr                    1/1     Running            13         27d
kube-system   coredns-f9fd979d6-c5tgg                    1/1     Running            13         27d
kube-system   etcd-master1.wt.com                        0/1     Running            13         27d
kube-system   kube-apiserver-master1.wt.com              1/1     Running            14         27d
kube-system   kube-controller-manager-master1.wt.com     1/1     Running            13         27d
kube-system   kube-proxy-7757c                           1/1     Running            13         27d
kube-system   kube-proxy-m9gbb                           1/1     Running            13         27d
kube-system   kube-scheduler-master1.wt.com              0/1     Running            13         27d
kube-system   kube-sealyun-lvscare-node1.wt.com          1/1     Running            14         27d
[root@master1 ~]# 
```

## kuboard可视化插件安装

**不想安装的可以略过，用命令的时间多**

```shell
## 参考 https://www.sealyun.com/instructions
## 直接url安装(这个版本有点低，我不是装的这个)
$ sealos install --pkg-url https://github.com/sealstore/dashboard/releases/download/v1.0-1/kuboard.tar
## 去上面那个url的下载页，下载高版本的离线包，离线安装，我的版本是3.0了
$ sealos install --pkg-url ./kuboard.tar
```

耐心等待安装完就行，安装好之后查看pod会看到多出来集合kuboard的，且状态是running，那么你就装好了，访问宿主机30080端口就行。

```shell
[root@master1 ~]# kubectl get po -A
NAMESPACE     NAME                                       READY   STATUS             RESTARTS   AGE
...
kuboard       kuboard-agent-2-68b9cf9585-gr9zh           1/1     running              1          8d
kuboard       kuboard-agent-5bc7d9cfbf-f6nkw             1/1     running              1          8d
kuboard       kuboard-etcd-qtwzd                         1/1     running              1          8d
kuboard       kuboard-v3-75b4bcbb45-jh88x                1/1     running              1          8d
[root@master1 ~]#
```

## Ingress安装

这个组件安装之后可以支持Ingress资源，用于k8s的Service服务，可以到时候在安装，我只是把所有安装的东西都集中在这里。

```shell
## 参考 https://www.sealyun.com/instructions
$ sealos install --pkg-url https://github.com/sealstore/ingress/releases/download/v0.15.2/contour.tar

## 上面url已经是最新版了，我是直接下载下来之后安装的,我是宿主机直接下的，嘻嘻，你也可以wget
$ wget https://github.com/sealstore/ingress/releases/download/v0.15.2/contour.tar
## 去上面那个url的下载页，下载高版本的离线包，离线安装，我的版本是3.0了
$ sealos install --pkg-url ./contour.tar
```

> 安装完之后

```shell
[root@master1 ~]# kubectl get po -A
NAMESPACE        NAME                         READY   STATUS         RESTARTS   AGE
...
heptio-contour   contour-6db688f7d9-2qwrg     2/2     Running        0          6m17s
heptio-contour   contour-6db688f7d9-xvvrf     2/2     Running        0          6m18s
```

