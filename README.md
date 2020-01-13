本仓库对应博客地址：http://nelsonblog.me

## 仓库说明：

1. 博客仓库使用 hexo 分支来保存基于 hexo 框架的源文件；hexo 生成的静态博客 html 文件默认放在 master 分支上；
2. 如果要更新博客主题，需要把主题拷贝至 theme 文件夹下；
3. hexo 分支源文件没有保存 node_modules 文件夹，使用时需要执行 `npm install` 命令来拉取 hexo 框架依赖；

具体可参考[这篇博客](https://www.jianshu.com/p/0b1fccce74e0)设置！

## 目录结构

* node_modules: hexo 需要的模块，需要 git 忽略，可以本地执行`npm install`获取
* themes 主题文件夹，如果需要更新主题，可以拷贝主题到这里即可）
* sources 博文 md 文件
* public 框架生成的静态网页（git 忽略）
* package.json 记录 hexo 需要的包信息
* _config.yml 全局配置文件
* .gitignore git 忽略配置

## hexo 命令

* hexo s --debug 可以在本地部署blog，localhost:4000即可，--debug是输出调试日志
* hexo g 生成html文件等
* hexo d 部署文章到 github.io，因为本地已经关联了github的账号，具体设置在 hexo 分支的 _config.yml 全局配置文件里
* hexo new "xx" xx是文章title，具体的tag和category可以在MD文件中设置

## 博客发布

流程：

1. 在仓库 hexo 分支上使用命令创建文章 `hexo new "xx"`
2. hexo s --debug 本地预览调试
3. hexo g 生成 html 静态网页
4. hexo d 快捷部署到 github 

## 安装 Hexo 

* 安装 Node 

hexo 使用 Node.js 编写，需要先安装 node，具体可以参考 [使用 nvm 管理不同版本的 node 与 npm](http://bubkoo.com/2017/01/08/quick-tip-multiple-versions-node-nvm/)。

详细步骤：

1. 终端执行 `curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.1/install.sh | bash` 命令安装来 nvm，其中 v0.35.1 是版本号，最新版本命令去这里获取 [Github/nvm](https://github.com/nvm-sh/nvm#install-script)；
2. 添加如下配置到 `~/.bash_profile`文件里，再执行 `source ~/.bash_profile` 命令 重新刷新下配置；
    ```
    export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"
    [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm
    ```
3. 安装 Node

    执行`nvm install 13.3.0`安装 Node，13.3.0 为版本号，最新版本号查看[这里](https://nodejs.org/en/)；

* 安装 Hexo 

使用 node nmp 包管理器来安装依赖模块

```
// 安装 hexo 
npm install hexo-cli -g

// 在自己博客文件夹下执行
hexo init 

// 安装依赖
npm install 

// 写博客/发布等等...
hexo new "xx" 创建文章
hexo generate 本地生成静态网页
hexo server 本地发布
```

## 错误

* npm install 报各种 permissions denied

hexo 框架使用 Node.js 编写，本地环境需要安装 Node.js，具体安装请参考[官网](https://docs.npmjs.com/resolving-eacces-permissions-errors-when-installing-packages-globally)，官方推荐用 nvm 管理器来安装，不推荐使用 `brew install node`，因为 node 的包管理器-npm 后面安装其他依赖时，会报各种 permissions 错误！ nvm 安装 node 可以参考 [使用 nvm 管理不同版本的 node 与 npm](http://bubkoo.com/2017/01/08/quick-tip-multiple-versions-node-nvm/)。

* npm install 报错

使用 nvm 安装 node 之后，npm install 其他依赖时还会报权限错误，可以忽略，不影响使用！


## 参考

hexo 

* [Github-hexojs/hexo](https://github.com/hexojs/hexo)
* [利用Hexo在多台电脑上提交和更新github pages博客](https://www.jianshu.com/p/0b1fccce74e0)
* [Hexo+Github+Next+GoDaddy搭建博客](http://lzhblog.site/2019/07/13/Hexo-Github-Next-GoDaddy%E6%90%AD%E5%BB%BA%E5%8D%9A%E5%AE%A2/)


Node 安装

* [使用 nvm 管理不同版本的 node 与 npm](http://bubkoo.com/2017/01/08/quick-tip-multiple-versions-node-nvm/)
* [mac上安装npm](https://www.jianshu.com/p/39b4339a9b60)

错误

* [npm 在安装的时候提示 没有权限操作的解决办法 Error: EACCES: permission denied](https://segmentfault.com/a/1190000018660227)
* [hexo s 404 页面不存在](https://github.com/hexojs/hexo/issues/1384)
