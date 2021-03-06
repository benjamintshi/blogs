# 图解跨域请求、反向代理原理，对前端更友好的反向代理服务器 - Caddy

## 写在开头

本文采用图文解析、结合实战的方式进行网络原理解析，帮助大家去掌握一些网络知识，并了解 `Caddy` 的基本使用（见下图）。

![caddy](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/blogs/caddy/48.png)

本人计划在近几年将持续输出深度好文，如果对这类文章感兴趣的话，还请您点个 `关注` 和 `赞` 支持一下吧！

## 引言

大家好呀~

本篇文章主要是安利一个对前端更友好的 `web` 服务器 `Caddy`，我们会介绍 `Caddy` 的安装使用，并通过图文来解析其原理。

`Caddy` 是唯一一个在默认情况下自动使用 `HTTPS` 的 `Web` 服务器，可以用来完成跨域请求、反向代理、静态文件服务器、部署 `History SPA` 应用、负载均衡等等功能，在可读性、可维护性和易用性方面都做的很好，对前端更友好！

> 如果你还是不太理解 `Caddy` 到底是用来做什么的，那你可以把它简单理解为对前端更友好的 `nginx`。

## 反向代理

> 本文讨论的 `代理` 仅限于 `HTTP 代理`，不涉及其他协议。

`Caddy` 是一个简单好用的 `Web` 服务器，`反向代理` 是它的一个核心功能。所以，在介绍 `Caddy` 之前，我们先介绍一下 `反向代理` 是什么，`反向代理` 可以帮我们做什么事情。

我们先来了解一下正向代理，正向代理就是在客户端与服务器之间实现一个代理服务器，客户端的所有请求先经过代理服务器，由代理服务器再去请求真实服务器，请求成功后再由代理服务器将真实服务器的响应结果发回至客户端。

正向代理的经典案例就是公司内部的 `VPN` 代理，企业员工在 `远程开发` 时需要先连接 `VPN`，再由 `VPN` 连接至公司服务器。这样做可以防止一些陌生连接，拒绝除 `VPN` 外的所有外网连接，只有连接 `VPN` 才能正常访问公司服务器。

我们来画一张图帮助大家理解什么是 `正向代理`（见下图）

![caddy](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/blogs/caddy/1.png)

而反向代理正好相反，反向代理一般是在服务器端，客户端发起的网络请求首先被反向代理服务器收到，再由反向代理服务器决定转发到某个具体的服务。换而言之，反向代理服务器将决定客户端最终访问到的目标服务器，常见的反向代理案例有负载均衡、CDN 加速。

我们在实际开发中，可以使用反向代理来 `解决前端跨域问题`、`部署前端服务` 等等，我们本篇教程也是主要介绍这两个功能的使用。

我们来画一张图帮助大家理解什么是 `反向代理`（见下图）

![caddy](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/blogs/caddy/2.png)

最后使用一句话概括就是：正向代理隐藏真实客户端，反向代理隐藏真实服务端。

### Caddy 的优势

我们在实际开发中，可以使用 `Caddy` 来搭建反向代理服务器，从而完成跨域请求、静态文件服务器、部署 `History SPA` 应用、负载均衡等等功能，使用 `Caddy` 来做这些工作的好处是我们通过几行配置文件就可以完成这些工作，非常的简单易用。

在日常开发中我们通常使用 `webpack` 解决开发环境的跨域和请求转发问题，`webpack` 的 `proxy` 选项可以解决大部分跨域和请求转发问题，但是对 `history` 路由的支持性较差，并且组内开发的成员之间的配置可能会导致冲突，造成额外的维护成本。

使用 `nginx` 可以解决这些问题，但是 `nginx` 比较复杂，对前端人员并不是特别友好。在学习 `nginx` 的过程中我们可能会渐行渐远，忘记了我们的初衷只是为了解决跨域和请求转发问题。

`Caddy` 使用 `Go` 语言编写，跨平台性强，配置文件具有高可读性，对前端更友好。在可读性、可维护性和易用性方面的优势成为了选择 `Caddy` 的理由。

## 安装 `Caddy`

介绍了那么多，我们差不多可以进入到实战部分了，先从 `安装` 开始吧！

`Caddy` 目前有 `1.0` 和 `2.0` 两个大版本，本文是针对 `2.0` 版本的教程，如果需要使用 `1.0` 版本的话建议查看 [Caddy 1.0 官方文档](https://caddyserver.com/v1/)。

> 如果想要先了解 `Caddy` 好不好用，可以先跳过 `安装` 这一节。

### Mac 平台

> Mac 非常适合开发者，欢迎广大开发者加入 Mac 大家庭。

首先我们需要[下载 Caddy](https://github.com/caddyserver/caddy/releases/download/v2.0.0/caddy_2.0.0_mac_amd64.tar.gz)，你也可以去 [官方地址](https://github.com/caddyserver/caddy/releases) 下载最新版本。

由于 `Caddy` 由 `go` 编写，`go` 编译后的文件可以直接执行，所以我们下载完成后我们直接解压到自己的目录，比如 `~/bin/` 目录。然后我们加上一个映射就可以使用啦，我们使用 `vi ~/.bash_profile` 命令编辑文件，添加下面这行代码：

```bash
export PATH=~/bin
```

添加了全局映射后，我们使用下面这行命令使我们的改动生效

```bash
source ~/.bash_profile
```

接下来我们输入 `caddy version` 来验证我们的安装是否生效，如果可以正确输出 `caddy` 的版本说明已经安装成功啦~（见下图）

![caddy](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/blogs/caddy/3.png)

### Windows 平台

首先我们需要[下载 Caddy](https://github.com/caddyserver/caddy/releases/download/v2.0.0/caddy_2.0.0_windows_amd64.zip)，你也可以去 [官方地址](https://github.com/caddyserver/caddy/releases) 下载最新版本。

下载完成后，解压到你的常用目录（路径最好别带中文），然后我们复制 `Caddy` 所在目录的路径（见下图）

![caddy](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/blogs/caddy/57.png)

然后，我们使用 `Win + E` 唤起文件管理器，然后右键点击我的电脑，点击 `属性`（见下图）

![caddy](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/blogs/caddy/54.png)

然后，我们选择 `高级`，点击 `环境变量`（见下图）

![caddy](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/blogs/caddy/55.png)

然后我们在弹出的窗口中，选中 `Path` 这一栏（见下图）

![caddy](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/blogs/caddy/56.png)

然后，我们在弹出的窗口中点击新建，将我们复制的 `Caddy` 目录路径粘贴进去（见下图）

![caddy](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/blogs/caddy/58.png)

最后，我们点击 `确定`，保存设置。我们在命令行中输入 `caddy`，安装成功啦！（见下图）

![caddy](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/blogs/caddy/59.png)

### Linux 平台

首先我们使用 `curl` 命令下载 `Caddy` 的安装包，如下

```bash
curl -OL https://github.com/caddyserver/caddy/releases/download/v2.0.0/caddy_2.0.0_linux_amd64.tar.gz
```

> 大家根据自己的需要下载对应版本的安装包。

我们使用 `tar zxvf caddy_2.0.0_linux_amd64.tar.gz` 解压文件，解压后的 `caddy` 文件是可执行文件，我们再配置相应的映射，将命令映射到全局即可（见下图）

![caddy](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/blogs/caddy/60.png)

## `Caddy` 使用教程

在 `Caddy` 安装完成后，我们来学习如何使用 `Caddy` 吧。

## 使用 `Caddy` 解决跨域问题

我们先使用 `Caddy` 来解决一个前端最常见的跨域问题，我们以一个[简单 Demo](https://github.com/a1029563229/Blogs/tree/master/BestPractices/caddy) 为例。在该案例中，我们使用 `fetch` 发起一个网络请求，请求一个网络资源（见下图）

![caddy](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/blogs/caddy/16.png)

从上图我们可以看出，我们在使用 `fetch` 发起了一个网络请求后，将请求的结果打印出来。现在，我们打开浏览器，查看请求结果（见下图）。

![caddy](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/blogs/caddy/17.png)

从上图可以看到，我们的请求失败了，请求失败的原因是因为浏览器的 `同源策略` 导致的跨域问题。

> 同源策略是一个重要的安全策略，它用于限制一个 `origin` 的文档如何能与另一个源的资源进行交互，在使用 `XMLHttpRequest` 或 `fetch` 时则会受到同源策略的约束。

我们想要解决这个问题的话，需要服务端返回指定的响应头（`Access-Control-Allow-*`），这些响应头可以通过浏览器的 `同源策略` 检测。

如果需要在服务端配置响应头的话，则需要后端人员配合，由前端推动后端的工作在效率上是不高的，还可能有些后端人员难以配合（可能是异地、第三方接口、不知道跨域是啥...）。

我们现在来使用 `Caddy` 解决这个问题，我们需要通过简单的两步来解决这个跨域问题：

- 配置 `Caddyfile` （`Caddy` 的配置文件），启动 `Caddy`；
- 配置 `hosts` 文件；

### 配置 `Caddyfile`

`Caddyfile` 是 `Caddy` 的配置文件，我们在 `Demo 的根目录` 下新建文件 `Caddyfile`，添加下面几行代码

```bash
http://proxy.dev-api-mall.jt-gmall.com {
  reverse_proxy http://dev-api-mall.jt-gmall.com {
    header_up Host dev-api-mall.jt-gmall.com
    header_down Access-Control-Allow-Origin *
    header_down Access-Control-Allow-Methods *
    header_down Access-Control-Allow-Headers *
  }
}
```

我们对这几行配置进行简单的解析（见下图）：

![caddy](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/blogs/caddy/18.png)

我们来分析一下上面几行核心配置代码的含义吧，解析如下：

- `第 1 行`：拦截对 `http://proxy.dev-api-mall.jt-gmall.com` 这条 `url` 的访问请求，进行内部逻辑处理；
- `第 2 行`：将 `拦截的请求` 转发（反向代理）到 `http://dev-api-mall.jt-gmall.com`（我们的目标地址）；
- `第 3 行`：在转发请求时，添加首部字段 `Host: dev-api-mall.jt-mall.com`，这一步的目的是为了让目标服务器能够识别请求源；
- `第 4~6 行`：在响应结果时，添加 `Access-Control-Allow-*: *` 等多个首部字段信息，这样可以通过浏览器的 `同源策略` 检测；

我们通过嵌套结构的几行代码就可以将 `Caddyfile` 配置完成啦！

### 配置 `hosts` 文件

在配置好 `Caddyfile` 后，我们将我们请求的地址修改为 `http://proxy.dev-api-mall.jt-gmall.com`，代码实现如下：

![caddy](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/blogs/caddy/19.png)

我们在命令行工具使用 `caddy run --watch` 命令运行 `caddy`（运行 `caddy` 时请保证 `80` 端口是空闲的），`caddy` 运行成功后将会输出下面的结果（见下图）

![caddy](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/blogs/caddy/22.png)

然后我们打开浏览器，打开 `http://localhost:3000`（`Demo` 的运行地址），查看控制台输出的请求结果（见下图）

![caddy](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/blogs/caddy/20.png)

从上图可以看出，我们的请求失败了，这是因为我们在访问代理地址（`http://proxy.dev-api-mall.jt-gmall.com`）时，由于这个域名没有注册，将会导致 `DNS` 解析失败，最终导致请求失败。

此时我们只需要配置 `hosts` 文件，将这条 `hostname` 的 `IP`  地址指向本机即可，在 `hosts` 文件中添加下面这条记录：

```bash
127.0.0.1 proxy.dev-api-mall.jt-gmall.com
```

> `hosts` 文件是一个操作系统文件，以表的形式存储了 主机名 和 `IP` 地址，用于查找主机名称。
>
> 这条记录代表的是在访问 `proxy.dev-api-mall.jt-gmall.com` 时，将 `IP` 地址解析为 `127.0.0.1`（本机）。
> 
> 不同系统的 `hosts` 文件配置方法在本文的 `最后一节`。


配置好了 `hosts` 文件后，我们刷新浏览器，可以看到我们的请求结果被打印在控制台了！（见下图）

![caddy](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/blogs/caddy/21.png)

![caddy](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/blogs/caddy/23.png)

我们从上图可以看出，我们通过 `Caddy` 的反向代理功能解决了跨域问题，并且更好的模拟了真实环境的网络请求。

### 原理解析

我们来简单梳理一遍流程，分析一下 `Caddy` 做了什么，帮助我们解决了跨域问题。

我们从客户-服务端的视角来进行解析，我们的浏览器就是客户端，`Caddy` 同时作为服务端与客户端，目标服务器属于服务端。

#### 浏览器 - 客户端

首先，我们在客户端（浏览器）发起了一个请求，请求的地址是 `http://proxy.dev-api-mall.jt-gmall.com/vegetable/list?page=1&pageSize=20`，浏览器首先解析出 `hostname` 的值为 `proxy.dev-api-mall.jt-gmall.com`。

在解析出了 `hostname` 后，浏览器读取主机的 `hosts` 文件配置，查询是否匹配，此时将命中我们在 `hosts` 文件中设置的 `127.0.0.1 proxy.dev-api-mall.jt-gmall.com` 规则，将域名解析为 `IP` 地址 - `127.0.0.1`，也就是本机地址。

将域名解析完成后，浏览器解析到请求的端口为空，请求协议为 `http`，然后使用 `http` 的默认端口 `80` 与 `IP` 地址创建了网络套接字 `127.0.0.1:80`（如下图）。

![caddy](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/blogs/caddy/49.png)

创建好了网络套接字后，浏览器将与目标地址 `127.0.0.1:80`（我们运行的 `Caddy` 服务） 创建 `TCP` 连接，然后按照 `http` 协议标准封装好请求信息，以数据分组（`segment`）的形式发送给服务端。

#### `Caddy` - 服务端 + 客户端

我们的 `Caddy` 服务（服务端）运行在本地端口 `80` 上，对应的地址就是 `127.0.0.1:80`。所以， `Caddy` 服务收到了这个 `TCP` 连接请求，`Caddy` 将 `TCP` 的数据分组（`segment`）解析后，解析到了 `http` 请求（见下图）。

![caddy](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/blogs/caddy/24.png)

从上面可以看出，我们的请求源是 `127.0.0.1:57721`（`IP` 地址为我们本机的 `IP`，端口为 `浏览器` 发起请求时使用的的随机端口 - `浏览器` 客户端），目的地址是 `127.0.0.1:80`（`IP` 地址为我们本机的 `IP`，端口为 `Caddy` 的运行端口 - `Caddy` 服务端）。我们的 `Host` 请求头为 `proxy.dev-api...`（代理地址），请求来源（发起方）是 `http://localhost:3000`（我们的本地服务）。

`Caddy` 收到了这个 `http` 请求后，解析到协议为 `HTTP`，`Host` 为 `proxy.dev-api-mall.jt-gmall.com`，组合起来后匹配到了下面这条配置规则。（见下图）

![caddy](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/blogs/caddy/25.png)

从上图可以看出，`Caddy` 在匹配到内部规则后，开始处理这条请求。根据配置规则，`Caddy` 将这条请求转发到 `http://dev-api-mall.jt-gmall.com`。此时，`Caddy` 先进行 `DNS` 查询和端口查询，组合了 `IP` 地址与端口后再与该地址建立 `TCP` 连接，将客户端的请求原封不动的转发到指定地址（见下图）。

![caddy](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/blogs/caddy/27.png)

从上图可以看出，这条请求由作为客户端的 `Caddy` 发出。我们的请求源是 `10.8.71.38:52170`（`IP` 地址为本机的 `IP`，端口是 `Caddy` 使用的随机端口 - `Caddy` 客户端），目的地址是 `39.98.164.255:80`（`IP` 地址为目标服务器 `IP`，端口为 `HTTP` 协议默认端口号 - 目标服务器）。我们的 `Host` 请求头为 `dev-api...`（我们在 `Caddyfile` 中指定的 `Host` 首部），其余的首部字段及请求信息都由 `Caddy` 直接转发到目标服务器。

远程服务器接收到请求后，处理请求后返回响应结果。（见下图）

![caddy](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/blogs/caddy/28.png)

我们从上图可以看出，这条响应结果的源地址是 `39.98.164.255:80`（`IP` 地址为请求的服务器 `IP`，端口为请求的服务器端口 `80` - 远程服务器），目的地址是 `10.8.71.38:52170`（`IP` 地址为我们本机的 `IP`，端口是 `Caddy` 使用的随机端口 - `Caddy` 客户端）。服务器将响应结果发送到 `Caddy` 客户端，我们的 `Caddy` 客户端接收到响应结果后，由 `Caddy` 服务器进行处理。

我们的 `Caddy` 服务器在处理响应结果时，根据 `Caddyfile` 配置在响应结果中添加 `Access-Control-Allow-...` 三条首部信息，最后将这条响应结果发送给浏览器客户端。（见下图）

![caddy](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/blogs/caddy/26.png)

我们从上图可以看出，这条响应结果的源地址是 `127.0.0.1:80`（`IP` 地址为我们本机的 `IP`，端口为 `Caddy` 的运行端口 - `Caddy` 服务端），目的地址是 `127.0.0.1:57721`（`IP` 地址为我们本机的 `IP`，端口为 `浏览器` 发起请求时使用的的随机端口 - `浏览器` 客户端）。

我们可以在响应结果中看到，我们在 `Caddyfile` 设置的首部信息 `Access-Control-Allow-...` 被添加在了响应结果中，响应结果中有这三个首部字段就可以通过浏览器的 `同源策略` 限制。我们在响应首部中可以看到两个 `Server` 首部，一个是我们本地的 `Caddy` 服务自动添加，另一个可能是远程服务器上的 `Caddy` 服务器所添加的。最后，数据被正常返回，我们在浏览器的控制台也可以看到请求成功啦！（见下图）

![caddy](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/blogs/caddy/29.png)

从上图看出，我们通过 `Caddy` 的反向代理功能，解决了跨域问题！

我们最后来通过一张图帮助大家理解上面的流程吧！（见下图）

![caddy](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/blogs/caddy/32.png)

> 图有点大，建议点击查看原图，这样可以看到更多细节。

## 使用 `Caddy` 搭建反向代理服务器

在这一节我们将使用 `Caddy` 搭建反向代理服务器，`Caddy` 可以轻松地完成这项工作。

使用 `Caddy` 搭建反向代理服务器的思路和解决跨域问题的思路是差不多的，都是使用 `reverse_proxy` 属性。

我们想要实现的效果是，在访问 `http://www.caddy-test.com` 域名时，将其反向代理到我们的本地服务 `http://localhost:3000` 上。

我们先在 `http://localhost:3000` 服务加上一些样式，修改后效果如下图

![caddy](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/blogs/caddy/33.png)

我们从上图可以看出，我们的服务允许在本地的 `3000` 端口上，我们使用 `/list` 路径访问了一个列表页。

此时我们打开 `http://www.caddy-test.com/list`（见下图）

![caddy](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/blogs/caddy/34.png)

从上图可以看出，由于这个域名尚未注册，所以导致我们的 `DNS` 查询失败啦！

### 配置 `hosts` 文件

我们在本地开发时，只需要配置 `hosts` 文件，将这条 `hostname` 的 `IP`  地址指向本机即可，我们在 `hosts` 文件中添加这条记录：

```bash
127.0.0.1 www.caddy-test.com
```

> 不同系统的 `hosts` 文件配置方法在本文的 `最后一节`。

这条记录表示，当匹配到 `www.caddy-test.com` 域名时，返回 `IP` 地址 `127.0.0.1`（本机 `IP`）。我们在配置好了 `hosts` 文件后，我们再次打开 `http://www.caddy-test.com/list`（如下图）

![caddy](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/blogs/caddy/61.png)

从上图可以看出，我们此时的页面是一片空白。这是因为在解析了域名和端口后，浏览器最终访问到了 `127.0.0.1:80` 上的 `Caddy` 服务（我们在第一节的时候运行了 `Caddy`），而 `Caddy` 服务对这条域名的访问并没有做配置，无法做出正确响应。接下来，我们将会进行 `Caddyfile` 的配置。

> 扩展阅读：
>
> 如果此时访问 `http://www.caddy-test.com:3000/list`（指定端口）会发现页面可访问，也可能返回了 `Invalid Host header` 字符串（这是因为被 `webpack` 自带的一些安全策略拦截了正确的响应结果，但是我们已经成功访问到了服务）。
>
> 这是因为在指定了端口后，我们访问的地址就被解析成了 `127.0.0.1:3000`，直接访问指定端口的服务。
>
> 这样的方式既不安全（需要暴露可访问端口），也不优雅（带个端口号太难记啦）。

### 配置 `Caddyfile`

我们现在需要配置我们的 `Caddyfile`，配置如下：

```bash
http://www.caddy-test.com {
  reverse_proxy localhost:3000 {
    header_up Host localhost
  }
}
```

> 由于我们启动 `Caddy` 的命令加上了 `--watch`，所以此时我们的 `Caddy` 将会检测到 `Caddyfile` 的变化后自动重启。

我们现在再打开 `http://www.caddy-test.com/list` 就可以看到，我们的页面可以正常访问啦（见下图）！

![caddy](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/blogs/caddy/36.png)

从上图可以看出，在我们访问我们配置的 `测试域名` 时，展示了我们在本地 `3000` 端口运行的服务所返回的页面，我们的反向代理配置成功啦！

> 扩展阅读：
>
> 如果我们的域名不是配置在 `hosts` 文件中，而是注册在真实的 `域名注册机构`，那我们的 `Caddy` 服务就是 `“真正的”` 反向代理服务器啦！

接下来我们对 `Caddyfile` 配置文件进行逐行解析（见下图）。

![caddy](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/blogs/caddy/37.png)

我们来逐行解析一下：
  - `第 10 行`：拦截对 `http://www.caddy-test.com` 这条 `url` 的访问请求，进行内部逻辑处理；
  - `第 11 行`：将 `拦截的请求` 转发（反向代理）到 `localhost:3000`（我们的本地服务）；
  - `第 12 行`：转发请求时，带上首部字段 `Host: localhost`，这一步的目的是为了通过 `webpack` 自带的 `Host` 首部安全检查；

### 原理解析

其实反向代理的原理和解决跨域问题的原理是一样的，只是把远程服务器地址换成了内网地址，所以我们直接用一张长图来进行解释吧（见下图）。

![caddy](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/blogs/caddy/50.png)

> 图有点大，建议点击查看原图，这样可以看到更多细节。

## 使用 `Caddy` 部署 `SPA - History` 路由模式项目

在介绍完了反向代理后，我们来介绍一下如何使用 `Caddy` 部署 `history` 路由模式的单页应用吧。

目前前端的两种路由模式主要分为 `hash` 和 `history` 模式两种。`hash` 模式是指通过地址栏 `URL` 中的 `#` 符号区分路由，而 `history` 模式就是通过路径 `/xxx` 来区分路由。

在单页（`SPA`）应用中使用 `history` 路由模式需要服务器配置支持，我们在开发过程中可以通过 `webpack` 来配置 `history` 路由模式。在我们将应用打包后，我们可以通过 `Caddy` 配置，使我们的 `Caddy` 服务器支持 `history` 路由模式的 `SPA` 应用。

首先，我们在 `SPA` 应用中配置 `history` 路由模式，然后使用打包命令 `npm run build` （不同技术栈的打包大同小异）将我们的应用打包，最后项目的目录层级看起来像是这样的（见下图）

![caddy](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/blogs/caddy/41.png)

我们构建好的代码在 `dist` 目录下，`Caddyfile` 与 `dist` 同级，接下来我们配置一下 `Caddyfile`，配置如下：

```bash
http://localhost:3000 {
  file_server
  root * ./dist
  try_files {path} /index.html
}
```

配置完成后，我们打开浏览器，输入 `http://localhost:3000/list`，会发现我们的页面成功渲染啦（见下图）！

![caddy](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/blogs/caddy/42.png)

我们简单剖析一下这几行配置（见下图）

![caddy](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/blogs/caddy/43.png)

我们来进行逐行解析一下：

- `第 16 行`：拦截对 `http://localhost:3000` 这条 `url` 的访问请求，进行内部逻辑处理（在测试或生产环境时，这里应该配置一个真实域名）；
- `第 17 行`：启用静态文件服务器；
- `第 18 行`：静态文件服务器访问的根目录在 `./dist` - 在 `dist` 文件夹外的内容无法访问；
- `第 19 行`：这行代码是处理 `history` 路由模式的关键 - 如果 `URL` 匹配不到任何静态资源，将会返回 `index.html`（解决 `404` 问题）；

从上面可以看出，使用 `Caddy` 部署 `history` 路由模式的单页应用还是比较简单的。这里还涉及了一些服务器运维的知识，先不作展开啦，有兴趣的童鞋可以自己去了解一下。

## 使用 `Caddy` 进行负载均衡

使用 `Caddy` 进行负载均衡也是建立在反向代理的基础之上，我们将 `Demo` 分别在三个端口运行（模拟多个服务器运行的多个实例），最后运行效果如下：

![caddy](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/blogs/caddy/40.png)

从上图可以看出，我们启动了三个同样的 `Demo` 服务，使用网站的 `title` 来进行区分。

使用 `Caddy` 做负载均衡，只需要将多个服务挂在同一个 `reverse_proxy` 属性下即可（见下图）

![caddy](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/blogs/caddy/44.png)

在配置完成后，我们打开浏览器，输入 `http://www.caddy-test.com`，然后多刷新几次，看看效果（见下图）：

![caddy](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/blogs/caddy/45.png)

![caddy](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/blogs/caddy/46.png)

![caddy](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/blogs/caddy/47.png)

从上面三张图可以看出，在不断刷新的过程中，`Caddy` 自动将我们的请求随机分流分配到某个服务上，从而达到负载均衡的效果。

注意，实际生产环境的负载均衡要比文中描述的复杂的多，有需要的童鞋最好自己去了解一下。负载均衡并不是本教程的重点，就不作展开讨论了。

## 不同平台的 `hosts` 文件配置

如果你知道 `hosts` 文件如何配置，那么你可以跳过本节内容~

### Mac

Mac 修改 `hosts` 文件很简单，使用 `vi` 命令即可，如下：

```bash
# 可能需要 root 权限
sudo vi /etc/hosts
```

在命令行输入命令行，键盘 `i` 可进入编辑模式，编辑完成后使用 `Esc` 键退出编辑模式。

最后，同时按下 `shift + :` 键，输入 `wq!` 即可保存更改。

### Linux

与 `Mac` 的方法类似，不做复述。

### Windows

首先使用 `Win + R` 键唤起 `运行` 输入框（如下图）

![caddy](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/blogs/caddy/51.png)

然后我们输入 `C:\Windows\System32\drivers\etc\hosts` 后按下 `确定` 按钮（见下图）

![caddy](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/blogs/caddy/52.png)

点击 `确定` 按钮后，选择使用 `记事本` 打开，然后进行修改、保存就可以啦（可能需要管理员权限）。

## 小结

最后，我们使用 `Caddy` 完成了跨域请求、反向代理、静态文件服务器、部署 `History SPA` 应用、负载均衡多种功能。

从上面的案例中我们可以看出，`Caddy` 在可读性、可维护性和易用性方面确实做的不错，通过简单的学习就可以上手使用。

如果只是用于本地开发、中小型应用，那么强烈推荐你使用 `Caddy`！

如果想要用于复杂的大型项目，那么建议你可以先参考下面这些资料，再决定是否使用：
  - [Caddy 官网](https://caddyserver.com)
  - [Caddy 社区](https://caddy.community)
  - [Caddy 源码](https://github.com/caddyserver/caddy)

## 最后一件事

如果您已经看到这里了，希望您还是点个赞再走吧~

您的点赞是对作者的最大鼓励，也可以让更多人看到本篇文章！

如果觉得本文对您有帮助，请帮忙在 [github](https://github.com/a1029563229/Blogs) 上点亮 `star` 鼓励一下吧！

![personal](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/blogs/other/6.png)