> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/1915712657853493765)

Joplin 是一款功能全面且高度可定制的开源笔记应用程序，它支持创建待办事项列表、制定计划以及保存信息等多种功能，非常适合用来管理个人笔记、项目规划和知识积累。本文将详细介绍如何在 Windows 系统本地部署 Joplin 并结合路由侠实现外网访问本地部署的 Joplin 。

### 第一步，本地部署安装 Joplin

1，本教程操作环境为 Windows11 系统，首先访问 Joplin 官网，下载适用于 Windows 的最新版本 Joplin 桌面应用程序，**[点此下载](https://joplinapp.org/)**。

![](https://pic1.zhimg.com/v2-da152504aeb4c30ef769b89167720c70_r.jpg)

2，双击下载的安装文件，根据提示完成安装过程。

![](https://pic1.zhimg.com/v2-f067f0b65f731254b027f3c3e1fd9464_r.jpg)

3，安装完成后，打开 Joplin 应用程序。

![](https://pic3.zhimg.com/v2-ed58399ea1a6b3fcfc6f3995a888e430_r.jpg)

4，首次运行会要求你进行初始化设置，设置完成，就可以开始创建笔记、待办事项列表等了，所有数据都会自动保存到本地的 SQLite 数据库中，无需额外配置。

![](https://pica.zhimg.com/v2-a5f88c86a06638ef3680183df4e1d242_r.jpg)

5，点击菜单栏【工具】→【选项】。

![](https://pic4.zhimg.com/v2-c9403b2227519f7db0e54385b84ae6cf_r.jpg)

6，点击左侧【同步】，选择自己的同步目标，本教程以 “ Nextcloud ” 为例。

![](https://pica.zhimg.com/v2-4566d7f88e4a15fa948e9fc7221897ee_r.jpg)

7，在浏览器中输入地址 http:// 你的服务器 IP ，登录到你的 Nextcloud 实例。

![](https://pic1.zhimg.com/v2-8a0f4abd8f087545cfe649c8a6843218_r.jpg)

8，回到 Joplin 应用程序，在 “URL” 一栏输入自建的 Nextcloud 服务器，一般格式为 http:// 你的 Nextcloud 实例服务器 IP /remote.php/dav/files / 你的用户名 / 。

然后填写你的 Nextcloud 用户名和密码以用于身份验证。

最后点击【检查同步配置】按钮，Joplin 会尝试连接到你的 Nextcloud 并通过 WebDAV 进行同步。如果一切正常，会显示一个成功的消息。

![](https://pic4.zhimg.com/v2-bb80138c68f31e14f820d8bcc8639ab3_r.jpg)

### 第二步，外网访问本地 Joplin

在内网的电脑上安装路由侠，**[点此下载](https://www.luyouxia.com/?from=help)**

1，下载安装完成后，打开路由侠界面，点击【内网映射】。

![](https://pic1.zhimg.com/v2-615650f1f0ca60f5830126d096aa479c_r.jpg)

2，点击【添加映射】。

![](https://pic3.zhimg.com/v2-9e00121c4a8ee5c1d58f78eab6ed36a6_r.jpg)

3，选择【原生端口】。

![](https://pic1.zhimg.com/v2-c5b0784f37fbc67f4d28802e943146b8_r.jpg)

4，在内网地址填写你的服务器 IP 和端口 80 后点击【创建】按钮，如下图。

![](https://pic4.zhimg.com/v2-fb5a445420cc885b417301dfefb3cf75_r.jpg)

5，创建好后，就可以看到一条映射的公网地址，鼠标右键点击【复制地址】。

![](https://pic2.zhimg.com/v2-8dc45071fdbcbc394cd544fbc1a4f9cf_r.jpg)

6，在外网电脑上，打开浏览器，在地址栏输入从路由侠生成的外网地址，就可以看到内网部署的 Nextcloud 登录界面了。

![](https://pic2.zhimg.com/v2-5080117f3f97b47d6180c7429630c30d_r.jpg)

7，打开 Joplin 应用程序，把 “URL” 一栏改成从路由侠复制的外网地址，格式为 [http://lyxbook.a1.luyouxia.net:22711/remote.php/dav/files/](http://lyxbook.a1.luyouxia.net:22711/remote.php/dav/files/) 你的用户名 / 。点击【检查同步配置】按钮，出现成功信息即可进行同步工作。

![](https://pic1.zhimg.com/v2-eb5a6bddb4d16efda178b7c5265848fa_r.jpg)