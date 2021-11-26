# nvm

- windows上安装nvm先删除之前安装的node
- nvm install 8   安装node8最高版本
- 针对http-server需要全局安装才能正常使用，默认安装的是14版本的http-server，在低版本的node上会报错，可以下载低的版本
- 包的全局安装是在C:\Users\clear\AppData\Roaming\nvm\v8.15\node_modules目录下;本地安装在自己的项目目录下安装包。本地安装为了使用不同版本的node和不同版本的包文件。


####刚刚使用本地的http-server，一样可以用呀哈哈，搞不懂了。另外发现了一点小坑，当前用node14安装的包，用node8去卸载会报错（npm uninstall --），所以下载卸载都应该使用同一个版本的node.
