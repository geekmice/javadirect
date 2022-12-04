# nodejs,docsify,gitee部署个人技术文档博客
## nodejs安装和配置
[node.js 安装详细步骤教程](https://blog.csdn.net/antma/article/details/86104068)
>测试是否安装成功

![20211130225329](https://geekmice-blog.oss-cn-beijing.aliyuncs.com/blog/20211130225329.png)
node和npm版本都可以看出来，说明安装成功
## gitee操作
### 创建gitee账户
[创建github或者gitee账户](https://blog.csdn.net/Ky_11111/article/details/120870524)
### 创建gitee仓库
![20211130225720](https://geekmice-blog.oss-cn-beijing.aliyuncs.com/blog/20211130225720.png)
1:新建仓库
2:仓库名称
3:仓库开源使用,其他人可以访问
4:配置readme.md文件
### 克隆并完成首次提交
#### git全局配置
```sh
git config --global user.name "geekmice"
git config --global user.email "katew4714@gmail.com"
```

![20211130230123](https://geekmice-blog.oss-cn-beijing.aliyuncs.com/blog/20211130230123.png)
#### 创建git仓库
```sh
mkdir javadirect
cd javadirect
git init
touch README.md
git add README.md
git commit -m "first commit"
git remote add origin https://gitee.com/geekmice/javadirect.git
git push -u origin master
```

![20211130230935](https://geekmice-blog.oss-cn-beijing.aliyuncs.com/blog/20211130230935.png)
## docsify
### 安装docsify
#### 配置淘宝镜像
```sh
# 使用淘宝镜像
npm config set registry https://registry.npm.taobao.org
# 验证是否ok
npm config get registry
```

> 验证是否成功

![20211130231120](https://geekmice-blog.oss-cn-beijing.aliyuncs.com/blog/20211130231120.png)

#### 全局安装docsify
执行指令`npm i docsify-cli -g`

![20211130231255](https://geekmice-blog.oss-cn-beijing.aliyuncs.com/blog/20211130231255.png)
### 测试安装情况

![20211130231324](https://geekmice-blog.oss-cn-beijing.aliyuncs.com/blog/20211130231324.png)
### 初始化博客项目
#### 在博客目录下执行初始化操作
```sh
  docsify init .
```
![20211130231616](https://geekmice-blog.oss-cn-beijing.aliyuncs.com/blog/20211130231616.png)

我们初始化后的目录内会存在一个.nojekyll的空文件，这个文件的作用是，在你提交代码到git仓库后，可以避免仓库忽略开头的文件，因为Docsify的很多文件都是以开头的。

另外一个index.html文件，就是Docsify唯一的配置文件。我们只需要维护这个入口html文件，即可完成所有的博客搭建操作。

#### 下来我们启动本地服务，查看下效果
```sh
# 输入本地启动命令
docsify serve .

Serving D:\BlogTest now.
Listening at http://localhost:3000
```

## Gitee版本控制
### 部署到Gitee(通过Gitee Pages)
Gitee Pages 是一个免费的静态网页托管服务，您可以使用 Gitee Pages 托管博客、项目官网等静态网页。如果您使用过 Github Pages 那么您会很快上手使用 Gitee 的 Pages服务。目前 Gitee Pages 支持 Jekyll、Hugo、Hexo编译静态资源。
>具体步骤如下

注意：首次使用需要验证个人信息（身份证信息，正面，背面，手持），一般需要一天时间左右就可以了

![20211203001236](https://geekmice-blog.oss-cn-beijing.aliyuncs.com/blog/20211203001236.png)

选中master分支，申请即可

![20211203001301](https://geekmice-blog.oss-cn-beijing.aliyuncs.com/blog/20211203001301.png)

之后本地修改之后代码提交给码云
```sh
git add .
git commit -m '提交时备注'
git pull origin master
git push origin master
```

访问地址如下
[我的个人笔记](https://geekmice.gitee.io/javadirect/)