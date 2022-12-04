## 快速开始
推荐全局安装 docsify-cli 工具，可以方便地创建及在本地预览生成的文档。

```bash
npm i docsify-cli -g
```

![20211203004403](https://geekmice-blog.oss-cn-beijing.aliyuncs.com/blog/20211203004403.png)

### 初始化项目
```bash
mkdir temp
docsify init
```

![20211203004454](https://geekmice-blog.oss-cn-beijing.aliyuncs.com/blog/20211203004454.png)

### 开始写文档
初始化成功后,生成几个文件
- index.html 入口文件
- README.md 主页内容渲染
- .nojekyll 用于阻止 GitHub Pages 忽略掉下划线开头的文件
### 本地预览

![20211203004757](https://geekmice-blog.oss-cn-beijing.aliyuncs.com/blog/20211203004757.png)

## 多页文档

### 定制侧边栏
为了获得侧边栏，您需要创建自己的_sidebar.md，你也可以自定义加载的文件名。默认情况下侧边栏会通过 Markdown 文件自动生成，效果如当前的文档的侧边栏

![20211203005245](https://geekmice-blog.oss-cn-beijing.aliyuncs.com/blog/20211203005245.png)

配置loadSidebar项
```html
<!-- index.html -->

<script>
  window.$docsify = {
    loadSidebar: true
  }
</script>
<script src="//cdn.jsdelivr.net/npm/docsify/lib/docsify.min.js"></script>
```

创建 _sidebar.md 文件，内容如下
```md


```

### 嵌套的侧边栏
你可能想要浏览到一个目录时，只显示这个目录自己的侧边栏，这可以通过在每个文件夹中添加一个 _sidebar.md 文件来实现。

_sidebar.md 的加载逻辑是从每层目录下获取文件，如果当前目录不存在该文件则回退到上一级目录。例如当前路径为 /zh-cn/more-pages 则从 /zh-cn/_sidebar.md 获取文件，如果不存在则从 /_sidebar.md 获取。

当然你也可以配置 alias 避免不必要的回退过程。

![嵌套的侧边栏](https://geekmice-blog.oss-cn-beijing.aliyuncs.com/blog/嵌套的侧边栏.gif)

![20211203095910](https://geekmice-blog.oss-cn-beijing.aliyuncs.com/blog/20211203095910.png)

### 用侧边栏中选定的条目名称作为页面标题

### 显示目录

### 忽略副标题


## 定制导航栏

## 封面

### 基本用法

启渲染封面功能后在文档根目录创建 _coverpage.md 文件。渲染效果如本文档
```html
<!-- index.html -->

<script>
  window.$docsify = {
    coverpage: true
  }
</script>
<script src="//cdn.jsdelivr.net/npm/docsify/lib/docsify.min.js"></script>
```

```md
<!-- _coverpage.md -->

![logo](_media/icon.svg)

# docsify <small>3.5</small>

> 一个神奇的文档网站生成器。

- 简单、轻便 (压缩后 ~21kB)
- 无需生成 html 文件
- 众多主题

[GitHub](https://github.com/docsifyjs/docsify/)
[Get Started](#docsify)
```

### 自定义背景
```md
![](_media/bg.png)
```