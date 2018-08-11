# hexo-to-your-server
带你跳过各种坑，一次性把 Hexo 博客部署到自己的服务器

## 一、完成效果

[点击查看个人网站](http://www.freeze61.me/)

## 二、准备

基本上每一步都会给出链接，可以多多参考

### 本地配置`node`环境

1. `node`下载 [官网下载](https://nodejs.org/zh-cn/download/)

参考链接：

- [史上最详细的Hexo博客搭建图文教程](https://xuanwo.org/2015/03/26/hexo-intor/)

- [通过Git将Hexo博客部署到服务器](https://www.jianshu.com/p/e03e363713f9)

可以使用官方的`node`安装包，也可以使用`nvm`管理 node 版本，但千万**不要混用**，不然会环境和管理上的麻烦

2. 安装：基本都是下一步，但记得把目录改到其他盘，这里我具体是在 `D:\programming\nodejs`

![node更换路径](hexo-server/node-path.jpg)

打开`cmd`查看`node`安装情况，分别执行如下命令

```bash
node -v
npm -v
```

3. 配置全局环境

参考链接：

- [windows npm -g 全局安装的命令找不到](https://blog.csdn.net/jizhuanfan0742/article/details/80910187)

进入安装目录，创建文件夹`node_global`和`node_cache`，分别执行如下命令

```bash
npm config set prefix "D:\programming\nodejs\node_global"
npm config set cache "D:\programming\nodejs\node_cache"
```

环境配置：新增环境变量`NODE_PATH`和添加`Path`，两个值都为 `D:\programming\nodejs\node_global`

![node更换路径](hexo-server/env-node.jpg)

4. 测试全局环境

打开`cmd`安装`hexo-cli`，分别执行如下命令

```bash
npm install hexo-cli -g
hexo
```

如果出现下面情况，恭喜你成功跳过`全局模块不能调用`的坑

![node更换路径](hexo-server/test-hexo.jpg)

5. 如果出现`命令未找到，或不是可执行程序`，别着急！**先仔细重复 3-4 步**

6. 如果第 4 步有错即不能调用到`hexo`全局命令，往后看，如果没错，**直接跳过**

随便找个地方初始化文件，分别执行如下命令

```bash
mkdir hexo-blog
cd hexo-blog && npm init
然后回车键一阵乱敲！
```

现在有 3 种解决方法，**任选其一**

- 法 1：检查`D:\programming\nodejs\node_global`目录是否有`hexo`模块，执行如下命令

```bash
D:\programming\nodejs\node_global\hexo
```

如果成功显示，步骤 4 的情况，则调用成功。如果觉得每次加一系列前缀麻烦，往后看

- 法 2：使用`link`命令，执行如下命令

```bash
npm link hexo
```

package.json 中新建脚本如下，并执行

```bash
npm run hexo
```

![新建npm脚本](hexo-server/npm-script.jpg)

- 法 3：你还可以直接在`hexo-blog`中下载，分别执行如下命令

```bash
npm install hexo-cli -S
npm run hexo （还是要在package.json中新建脚本）
```

### 初始化`hexo`项目

1. 如果是按照上一节步骤 6 过来的，则就在`hexo-blog`下初始化，分别执行如下命令

```bash
hexo init myblog && cd myblog
npm install
```

2. 下载主题，执行如下命令

参考链接：

- [Hexo设置主题以及Next主题个性设置](https://www.jianshu.com/p/b20fc983005f)

- [官网全部主题](https://hexo.io/themes/)

```bash
git clone https://github.com/iissnan/hexo-theme-next themes/next
```

在**本地配置文件**中设置`theme`

![添加next主题](hexo-server/hexo-theme.jpg)

3. 本地执行`hexo`项目

统一添加`start`脚本，并执行如下命令

```bash
npm start
```

![新建npm-start脚本](hexo-server/npm-script-start.jpg)

自己可以打开`http://localhost:4000/`验证效果吧

### `git`环境搭建

1. `git`安装

参考资料：

- [廖雪峰老师的 git 教程](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)

- [git 官网下载](https://gitforwindows.org/)

2. 生成`ssh`认证，执行如下命令

```bash
git config --global user.name "yourname"
git config --global user.email youremail@example.com
ssh-keygen -t rsa -C "youremail@example.com"
```

获取到的`ssh`认证在`C:\Users\yourname\.ssh`中

---

## 三、服务器配置

### 搭建远程`Git`私库

1. 登录到远程服务器，建议使用`Xshell 5`

2. 安装 git，分别执行如下命令

```bash
git --version // 如无，则安装
yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel perl-devel
yum install -y git
```

3. 创建用户并配置其仓库，分别执行如下命令

参考资料：

- [使用 Git Hook 自动部署 Hexo 到个人 VPS](http://www.swiftyper.com/2016/04/17/deploy-hexo-with-git-hook/)

```bash
useradd git
passwd git // 设置密码
su git // 这步很重要，不换用户后面会很麻烦
cd /home/git/
mkdir -p projects/blog // 项目存在的真实目录
mkdir repos && cd repos
git init --bare blog.git // 创建一个裸露的仓库
cd blog.git/hooks
vi post-receive // 创建hook钩子函数，输入了内容如下
```

```bash
#!/bin/sh
git --work-tree=/home/git/projects/blog --git-dir=/home/git/repos/blog.git checkout -f
```

```bash
// 添加完毕后修改权限
chmod +x post-receive
exit // 退出到 root 登录
chown -R git:git home/git/repos/blog.git // 添加权限
```

4. 测试`git仓库`是否可用，随便空白文件夹，执行如下命令

```bash
git clone git@server_ip:/home/git/repos/blog.git
```

如果能把空仓库拉下来，就说明 git 仓库搭建成功了

![git仓库测试](hexo-server/git-clone.jpg)

5. 建立`ssh`信任关系，在**本地电脑**分别执行如下命令

参考资料：

- [ssh-copy-id 帮你建立信任](http://roclinux.cn/?p=2551)

```bash
ssh-copy-id -i C:/Users/yourname/.ssh/id_rsa.pub root@server_ip
ssh root@server_ip // 测试能否登录
```

同理，添加 git 的登录方式（添加 root 是为了以后开发，而 git 用户在确认建立联系后会关闭 ssh 登录方式）

```bash
ssh-copy-id -i C:/Users/yourname/.ssh/id_rsa.pub git@server_ip
ssh git@server_ip
```

**注**：此时的 ssh 登录不需要密码！否则就有错，请仔细重复步骤 3-4

6. 如果第 5 步能成功，则禁用`git用户`的 shell 登录权限，分别执行如下命令

参考资料：

- [Git Server - 限制 Git 用户使用 SSH 登陆操作](https://blog.csdn.net/lgyaxx/article/details/72954121)

```bash
cat /etc/shells // 查看`git-shell`是否在登录方式里面，有则跳过
which git-shell // 查看是否安装
vi /etc/shells
添加上2步显示出来的路劲，通常在/usr/bin/git-shell
```

修改`/etc/passwd`中的权限，将原来的

```bash
`git:x:1000:1000::/home/git:/bin/bash`
```

修改为

```bash
git:x:1000:1000:,,,:/home/git:/usr/bin/git-shell
```

### 搭建`nginx`服务器

1. 下载并安装`nginx`，分别执行如下命令

参考资料：

- [Nginx 源码安装和简单的配置](https://www.jianshu.com/p/4134a44a09c2)

```bash
cd /usr/local/src
wget http://nginx.org/download/nginx-1.15.2.tar.gz
tar xzvf nginx-1.15.2.tar.gz
cd nginx-1.15.2
./configure // 如果后面还想要配置 SSL 协议，就执行后面一句！
./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module --with-file-aio --with-http_realip_module
make && make install
alias nginx='/usr/local/nginx/sbin/nginx' // 为nginx取别名，后面可直接用
```

2. 配置`nginx`文件

参考资料：

- [Linux 中 nginx 基本操作命令](https://blog.csdn.net/yang_xu_1987/article/details/77929658)

先启动是否安装成功，执行如下命令

```bash
nginx // 直接来，浏览器查看 server_ip，默认是80端口
```

配置文件，分别执行如下命令

```bash
nginx -s stop // 先停止nginx
cd /usr/local/nginx/conf
vi nginx.conf
修改 root 解析路径
```

![修改nginx配置](hexo-server/nginx-conf.jpg)

---

## 四、链接

至此我们就把本地和服务器的环境全部搭建完成，现在利用 hexo 配置文件进行链接

### 配置`_config.yml`文件

1. 编辑`_config.yml`的`deploy`属性

![编辑本地deploy](hexo-server/config-deploy.jpg)

2. 在`package.json`中添加 npm 脚本

```json
"scripts": {
  "deploy": "hexo clean && hexo g -d",
  "start": "hexo clean && hexo g && hexo s"
},
```

3. 链接！这下在本地调试就用`npm start`，调试好了就上传到服务器，美滋滋~快通过你的服务器ip访问吧

```bash
npm run deploy
```

---

## 五、总结

本次教程介绍`node`环境配置，主要强调了全局模块的调用，然后是初始化 hexo 项目，建议多参考官方的配置。然后搭建本地和服务器的`git`环境，通过`ssh通行证`交互。接下来是通过`nginx.conf`文件来配置`nginx`。最后`_config.yml`的`deploy`参数来连接本地和服务器

---

## 六、最后说句

本人前端新手一枚，有错误的话欢迎指正

贴上 [个人网站](https://www.freeze61.me/)，建站初期，欢迎您的光临~

喜欢的话麻烦 [github](https://github.com/jeffery5461) 给个 ★ 哦
