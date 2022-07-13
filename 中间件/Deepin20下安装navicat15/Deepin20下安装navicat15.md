## Deepin20下安装navicat15

> 适用于Linix/Unix内核的安装，不限于Deepin

deepin20的安装就不说了，建议在实体机上直接装，流畅度比vmware高太多了，可用作为第一生产力机器来用了，换个界面换种心情。

**deepin 15.6之前是可以使用deepin的官方安装器免U盘安装的，15.6之后不支持体验安装，只能用u盘了，喜欢的朋友快上**

好吧，开始安装navicat15.

链接: https://pan.baidu.com/s/1G7yoCW2Vl36-aXFISECUtQ  密码: anjs

直接下载整个文件夹。

> 只能激活当前文件夹下的navicat15，不要去官网新下载，无法激活！！！已经尝试过！

### 准备工作keygen和patcher

这里的kengen和patcher已经编译好了, 因此只需要确保你装了以下几个库, 就可以跳过此节了。

#### 前提条件

1. 但是请确保你安装了下面几个库：

- `capstone`
- `keystone`
- `rapidjson`
- `openssl`

你可以通过下面的命令来安装它们：

```shell
# install capstone
$ sudo apt-get install libcapstone-dev

# install keystone
$ sudo apt-get install cmake
$ git clone https://github.com/keystone-engine/keystone.git
$ cd keystone
$ mkdir build
$ cd build
$ ../make-share.sh
$ sudo make install
$ sudo ldconfig

# install rapidjson
$ sudo apt-get install rapidjson-dev

# install openssl
$ sudo apt-get install openssl
```

2. 你的gcc支持C++17特性。

#### 编译

```shell
$ git clone -b linux --single-branch https://github.com/DoubleLabyrinth/navicat-keygen.git
$ cd navicat-keygen
$ make all
```

生成完成后，你会在 `bin/` 文件夹下看到编译后的keygen/patcher。

### 挂载并提取文件

假设你的目录直接就是从网盘下载好的Navicat15，且和我一样路径~/Navicat15，以下操作全部在这个目录下

```shell
## 把navicat15-premium-cs.AppImage挂载到一个目录，以便于把内部文件提取出来用patcher破解
# 挂载
$ mkdir navicat15-premium-cs
$ sudo mount -o loop avicat15-premium-cs.AppImage navicat15-premium-cs

# 提取文件
$ cp -r /navicat15-premium-cs navicat15-premium-cs-patched
```

文件提取完之后，挂载点就可用删了，我会在安装完毕之后一起来清理这些东西。

### Patcher破解

```shell
# 破解
$ chmod +x navicat-patcher
./navicat-patcher navicat15-premium-cs-patched
```

以上没有报错的话会有如下类似输出。

```
**********************************************************
*       Navicat Patcher (Linux) by @DoubleLabyrinth      *
*                  Version: 1.0                          *
**********************************************************

Press ENTER to continue or Ctrl + C to abort.

[+] Try to open libcc.so ... Ok!

[+] PatchSolution0 ...... Ready to apply
   RefSegment      =  1
   MachineCodeRva  =  0x0000000001413e10
   PatchMarkOffset = +0x00000000029ecf40

[*] Generating new RSA private key, it may take a long time...
[*] Your RSA private key:
   -----BEGIN RSA PRIVATE KEY-----
   MIIEowIBAAKCAQEArRsg1+6JZxZNMhGyuM8d+Ue/ky9LSv/XyKh+wppQMS5wx7QE
   XFcdDgaByNZeLMenh8sgungahWbPo/5jmkDuuHHrVMU748q2JLL1E3nFraPZqoRD
   ...
   ...
   B1Z5AoGBAK8cWMvNYf1pfQ9w6nD4gc3NgRVYLctxFLmkGylqrzs8faoLLBkFq3iI
   s2vdYwF//wuN2aq8JHldGriyb6xkDjdqiEk+0c98LmyKNmEVt8XghjrZuUrn8dA0
   0hfInLdRpaB7b+UeIQavw9yLH0ilijAcMkGzzom7vdqDPizoLpXQ
   -----END RSA PRIVATE KEY-----
[*] Your RSA public key:
   -----BEGIN PUBLIC KEY-----
   MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEArRsg1+6JZxZNMhGyuM8d
   +Ue/ky9LSv/XyKh+wppQMS5wx7QEXFcdDgaByNZeLMenh8sgungahWbPo/5jmkDu
   ...
   ...
   GrVJ3o8aDm35EzGymp4ON+A0fdAkweqKV6FqxEJqLWIDRYh+Z01JXUZIrKmnCkgf
   QQIDAQAB
   -----END PUBLIC KEY-----

*******************************************************
*                   PatchSolution0                    *
*******************************************************
[*] Previous:
+0x0000000000000070                          01 00 00 00 05 00 00 00          ........
+0x0000000000000080  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
+0x0000000000000090  00 00 00 00 00 00 00 00 40 cf 9e 02 00 00 00 00  ........@.......
+0x00000000000000a0  40 cf 9e 02 00 00 00 00 00 10 00 00 00 00 00 00  @...............
[*] After:
+0x0000000000000070                          01 00 00 00 05 00 00 00          ........
+0x0000000000000080  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
+0x0000000000000090  00 00 00 00 00 00 00 00 d0 d0 9e 02 00 00 00 00  ................
+0x00000000000000a0  d0 d0 9e 02 00 00 00 00 00 10 00 00 00 00 00 00  ................

[*] Previous:
+0x00000000029ecf40  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
+0x00000000029ecf50  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
+0x00000000029ecf60  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
...
...
+0x00000000029ed0c0  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
[*] After:
+0x00000000029ecf40  ef be ad de 4d 49 49 42 49 6a 41 4e 42 67 6b 71  ....MIIBIjANBgkq
+0x00000000029ecf50  68 6b 69 47 39 77 30 42 41 51 45 46 41 41 4f 43  hkiG9w0BAQEFAAOC
+0x00000000029ecf60  41 51 38 41 4d 49 49 42 43 67 4b 43 41 51 45 41  AQ8AMIIBCgKCAQEA
...
...
...
+0x00000000029ed0c0  43 6b 67 66 51 51 49 44 41 51 41 42 ad de ef be  CkgfQQIDAQAB....

[*] Previous:
+0x0000000001413e10  44 0f b6 24 18 48 8b 44 24 28 8b 50 f8 85 d2 79  D..$.H.D$(.P...y
+0x0000000001413e20  6f                                               o               
[*] After:
+0x0000000001413e10  45 31 e4 48 8d 05 2a 91 5d 01 90 90 90 90 90 90  E1.H..*.].......
+0x0000000001413e20  90                                               .               

[*] New RSA-2048 private key has been saved to
   /home/doublesine/github.com/navicat-keygen/RegPrivateKey.pem

*******************************************************
*           PATCH HAS BEEN DONE SUCCESSFULLY!         *
*                  HAVE FUN AND ENJOY~                *
*******************************************************
```

### 重新打包

将破解后的文件夹重新打包成AppImage.

```shell
$ chmod +x appimagetool-x86_64.AppImage
$ ./appimagetool-x86_64.AppImage navicat15-premium-cs-patched navicat15-premium-cs-patched.AppImage
```

### keygen激活

**后续所有操作在断网下进行！！！**

**后续所有操作在断网下进行！！！**

**后续所有操作在断网下进行！！！**

patcher破解完之后，当前文件夹下会有一个名为RegPrivateKey.pem的文件，如果当前文件没有，那么后续就无法在进行下去了。

#### 激活

```shell
$ ./navicat-keygen --text RegPrivateKey.pem
```

类似如下输出和操作:

第一步选product: 1  premium

第二步选语言: 1 simple chinese 

**我当前的这个是中文版本的，选0会导致序列号不对而导致后续无法点击激活按钮**

第三步选主版本: 15

然后你会得到一个序列号，且当前界面会停住，让你输入name等信息。

**先别动，不要关窗口！**

```
**********************************************************
*       Navicat Keygen (Linux) by @DoubleLabyrinth       *
*                   Version: 1.0                         *
**********************************************************

[*] Select Navicat product:
0. DataModeler
1. Premium
2. MySQL
3. PostgreSQL
4. Oracle
5. SQLServer
6. SQLite
7. MariaDB
8. MongoDB
9. ReportViewer

(Input index)> 1

[*] Select product language:
0. English
1. Simplified Chinese
2. Traditional Chinese
3. Japanese
4. Polish
5. Spanish
6. French
7. German
8. Korean
9. Russian
10. Portuguese

(Input index)> 0

[*] Input major version number:
(range: 0 ~ 15, default: 12)> 15

[*] Serial number:
NGHK-LKCJ-EO8J-KRGL

[*] Your name:
```

第四步:  运行navicat,也就是我们重新打包后的AppImage.

```shell
$ chmod +x navicat15-premium-cs-patched.AppImage
$ ./navicat15-premium-cs-patched.AppImage
```

> 网上很多教程都是先运行，然后执行keygen，我这里是先执行的keygen，没有任何问题，如果你遇到了问题，你也可以试试这种方式。

执行之后navicat打开了，输入刚刚的序列号，激活，因为没有联网，所以会激活失败，有个弹窗，点击手动激活，并复制手动激活窗口中的那串请求码(激活过windows版本的朋友应该很熟悉了)。

第五步： 回到keygen窗口，输入你的名字和组织，随便输，短一点。

```
[*] Your name: kid
[*] Your organization: qita

[*] Input request code in Base64: (Double press ENTER to end)
```

然后输入刚刚从手动激活窗口中复制的请求码，两次回车结束。

```
[*] Input request code in Base64: (Double press ENTER to end)
OaGPC3MNjJ/pINbajFzLRkrV2OaSXYLr2tNLDW0fIthPOJQFXr84OOroCY1XN8R2xl2j7epZ182PL6q+BRaSC6hnHev/cZwhq/4LFNcLu0T0D/QUhEEBJl4QzFr8TlFSYI1qhWGLIxkGZggA8vMLMb/sLHYn9QebBigvleP9dNCS4sO82bilFrKFUtq3ch8r7V3mbcbXJCfLhXgrHRvT2FV/s1BFuZzuWZUujxlp37U6Y2PFD8fQgsgBUwrxYbF0XxnXKbCmvtgh2yaB3w9YnQLoDiipKp7io1IxEFMYHCpjmfTGk4WU01mSbdi2OS/wm9pq2Y62xvwawsq1WQJoMg==

[*] Request Info:
{"K":"NAVMRTVJEO42IODD", "DI":"4A12F84C6A088104D23E", "P":"linux"}

[*] Response Info:
{"K":"NAVMRTVJEO42IODD","DI":"4A12F84C6A088104D23E","N":"DoubleLabyrinth","O":"DoubleLabyrinth","T":1575543648}

[*] Activation Code:
i45HIr7T1g69Cm9g3bN1DBpM/Zio8idBw3LOFGXFQjXj0nPfy9yRGuxaUBQkWXSOWa5EAv7S9Z1sljlkZP6cKdfDGYsBb/4N1W5Oj1qogzNtRo5LGwKe9Re3zPY3SO8RXACfpNaKjdjpoOQa9GjQ/igDVH8r1k+Oc7nEnRPZBm0w9aJIM9kS42lbjynVuOJMZIotZbk1NloCodNyRQw3vEEP7kq6bRZsQFp2qF/mr+hIPH8lo/WF3hh+2NivdrzmrKKhPnoqSgSsEttL9a6ueGOP7Io3j2lAFqb9hEj1uC3tPRpYcBpTZX7GAloAENSasFwMdBIdszifDrRW42wzXw==
```

最后一步: 复制Activation Code到手动激活窗口，点击激活。

#### 可能遇到的错误

序列号输入后显示x，激活按钮无法点击.

```
检查是否是product或者语言或者版本号选择，填写错误。
我遇到的就是语言选成了0 English，然后激活按钮无法点击。
```

keygen执行时会报错: 

```
[-] ./navicat-keygen/main.cpp:174 ->
    BIO_new_file failed.
```

如果这个文件有问题，会报错：

```
[-] ./navicat-keygen/main.cpp:63 ->
    PEM_read_bio_RSAPrivateKey failed.
    Hints: Are you sure that you DO provide a valid RSA private key file?
```

我目前只遇到过上面这些错误，但都是操作不对引起的，正常流程不会有错。

### 最后工作

现在Navicat15文件夹下的`navicat15-premium-cs-patched.AppImage`就是最终破解了的应用，复制这个应用到你想要的地方就行。

闲麻烦的直接运行就用命令，会启动一个终端。

```shell
$ ./navicat15-premium-cs-patched.AppImage
```

#### 优化，将其添加到启动器当中

来自一个优秀(菜逼)程序员的强迫症，制止了我直接使用命令行启动的方式。

`我当前是deepin系统`。打开/usr/share/applicaions文件夹，这里面能看到所有的应用。

我们为navicat15创建一个叫navicat15.desktop的家伙。

```
[Desktop Entry]
Name=navicat15
Comment=navicat15
Exec=/home/wangtao/app/navicat15/navicat15-premium-cs-patched.AppImage
Icon=/home/wangtao/app/navicat15/navicat15.png
Terminal=false
Categories=Development
Type=Application
```

说明

```
第一行是必须的，就像shell脚本要加入#!/bin/bash一样，用于系统识别
第二行Name自己随意填，用于显示和搜索
第三行Comment备注，和Name填一样就行
第四行Exec是指应用可执行文件路径
第五行Icon是指应用图标的路径
第六行Terminal表示启动时是否需要显示终端，建议设置为false
第七行是指这个应用的分类
第八行Type就填Application
```

创建好之后就打开启动器就会看到，且能正常启动了。

### 安装后清理

```shell
# 取消挂载,挂载点用全路径，不确定全路径的用df -h直接看或者pwd
$ sudo umount /home/wangtao/Navicat15/navicat15-premium-cs-patched
# 清理多余文件
$ sudo rm -rf Navicat15/
```

