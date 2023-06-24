# npm私服搭建和项目发布

## 项目打包
- `npm pack` 项目进行打包，打包出来的就是 .tgz 压缩包
    + 可以本地进行安装试一下效果 `npm i ./p-nrm-1.0.0.tgz`，或全局安装`npm i ./p-nrm-1.0.0.tgz -g`
## 版本迭代处理
- 可以直接修改 package.json 里面的 version 版本号
- 使用命令自动迭代版本号
    + `npm version major -m "主版本迭代的描述信息"`
        * 当功能模块有较大变动，比如增加模块或者整体架构发生变化
    + `npm version minor -m "次版本迭代的描述信息"`
        * 相对于主版本而言，次版本的升级对应的只是局部的变动，但该局部的变动造成和以前版本不能兼容
    + `npm version patch -m "小版本迭代的描述信息"`
        * 修正版本号，一般是BUG的修复或是一些小的变动或是一些功能的扩充

## 私服搭建
### 什么是npm私服
&emsp;&emsp;私服与私有的git仓库有着非常相似的作用，当公司中开发了很多 JavaScript 的插件后，若想要良好的在公司内部分享，必要需要将代码打包构建，分享给
其他部门开发者使用，但每次都构建成tgz文件在使用上会出现很多问题

&emsp;&emsp;这时候公司内部则需要一个私有化的npm服务器用来管理公司内部开发的js依赖包，这主要因为公司开发的插件若不想开源或不希望服务公司外的用户时，
若直接发布到 npmjs.com 网站上会造成很多隐私泄露问题，所以此时公司更希望存在 npm 的私人服务器，这就是npm的私服。

### verdaccio简介
&emsp;&emsp;Verdaccio 是一个简单的、零配置本地私有 npm 软件包代理注册表。Verdaccio 开箱即用，拥有自己的小型数据库，能够代理其他注册表（例如：npmjs.org），缓存下载的模块。

### verdaccio 使用
- `npm i verdaccio -g` 全局安装
- `verdaccio` 运行 verdaccio 服务器，运行在本地的4873端口：`http://localhost:4873/`
- 切换本地npm源
    + `nrm add local http://localhost:4873/` 添加 verdaccio 服务地址作为新npm源
    + `nrm use local` 切换本地npm源
    + 可以使用 `npm config get registry` 查看源地址是否切换成功
- 添加当前源下的用户
    + `npm adduser` 然后按提示输入 用户名、密码、邮箱，创建成功即登录
    + 可以使用 `npm logout` 进行登出操作，使用 `npm login` 然后输入刚刚注册的信息进行登录
- 在网页里进行登录
    + 前一步的登录只是在命令行中登录
- `npm publish` 发布到私服上，然后就可以在网页里搜索到你发布的包了
- 然后用户就可以使用 `npm i p-nrm` 将私服上的包下载下来了
    + 切记 npm 源一定要切换到私服
    + 不用当心安装非私服上的包，因为 verdaccio 在私服上找不到这个包，就会自动代理到外网下载