### http-server：命令行http服务器

   http-server是一个简单的零配置命令行http服务器。它足够强大，足以用于生产用途，但它既简单又易于破解，可用于测试，本地开发和学习。

### 使用前提

* http-server 基于node运行，需要安装nodeJs环境
* node自带的npm包管理器
* 通过npm安装http-server

```
npm i http-server -g
```

```这将http-server在全局安装，以便可以从命令行运行。```

### 使用

<mark>cd进入download目录</mark>

可执行以下指令， 80 是http的默认端口，后面是登录的用户和密码

[官方指令详细使用](https://www.npmjs.com/package/http-server)

配置好直接输入网址即可下载

``` 
http-server.cmd ./
或
http-server.cmd .\Downloads\ -p 80 --username ysx --password ysx
```

 ### 解决方案

> 以管理员的身份 运行powershell (直接在开始栏中搜)
>
> 执行以下命令
> `set-ExecutionPolicy RemoteSigned`
>
> `y`进行确认
>
> 搞定 不放心可以查看 通过命令
> `get-ExecutionPolicy`

