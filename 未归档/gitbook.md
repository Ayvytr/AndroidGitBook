# [GitBook 使用](https://blog.csdn.net/lu_embedded/article/details/81100704)



1. 安装Node.js
2. 



换一台电脑打开GitBook项目居然运行不了，gitbook serve和gitbook build都报错。

Error: ENOENT: no such file or directory, stat ‘C:***demo_book\_book\gitbook\gitbook-plugin-fontsettings\fontsettings.js’

原来是一个Bug（Vesion：3.2.3）。

https://github.com/GitbookIO/gitbook/issues/1309

解决办法如下。

用户目录下找到以下文件。
<user>\.gitbook\versions\3.2.3\lib\output\website\copyPluginAssets.js

Replace all
confirm: true
with
confirm: false
--------------------- 
