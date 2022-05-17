---
title: "hugo在github配置静态网页"
date: 2021-12-18
description: 这是一个测试文档
menu:
   sidebar:
    name: hugo在github配置静态网页
    identifier: hugo在github配置网页
    parent: 安装与配置文件
    weight: 10
author:
  name: Gao
  image: /images/author/Gao.png
math: true
categories: ["Basic"]
---



hugo在github上搭建静态网页

1. 安装hugo

   在`https://github.com/gohugoio/hugo/releases`下载相应的安装包；

2. Ubuntu可以deb包直接进行安装

3. 验证安装是否成功 

   ```shell
   $ hugo version
   Hugo Static Site Generator v0.76.5 darwin/amd64 BuildDate: unknown
   ```

 4. 使用hugo

    在某个目录下执行如下命令创建一个网站，例如使用myblog这个名字；

    ```shell
    $ cd Documents/myblog/
    $hugo new site polarisxu
    Congratulations! Your new Hugo site is created in ./Documents/myblog/.
     
    Just a few more steps and you're ready to go:
    1. Download a theme into the same-named folder.
       Choose a theme from https://themes.gohugo.io/ or
       create your own with the "hugo new theme <THEMENAME>" command.
    2. Perhaps you want to add some content. You can add single files
       with "hugo new <SECTIONNAME>/<FILENAME>.<FORMAT>".
    3. Start the built-in live server via "hugo server".
    Visit https://gohugo.io/ for quickstart guide and full documentation.
    
    ```

 5. 下载使用主题

    ```shell
    $cd Documents/myblog/themes/
    $git clone https://github.com/MarcusVirg/forty
    ```

    将下载的themes文件中的exampleSite文件夹下的目录复制到/myblogs文件夹下。

    打开config.toml文件，将`baseURL = "http://example.org/"`修改为`baseURL = "https://xxxxx.github.io/"`

    其中xxxxx.github.io为在github上创建的空库。

    **注意将http改为https，否则网页可能不会完整显示**

 6. 预览

    ```shell
    $cd  Documents/myblog/
    $hugo server
    ```

    打开`http://localhost:1313/`即可看到。

 7. 部署github

    ```shell
    $cd Documents/myblog/public/
    $git clone https://github.com/xxxxx/xxxxx.github.io
    $cd xxxxx.github.io
    $git init
    $cd  Documents/myblog
    $hugo
    $cd Documents/myblog/public/
    $git add .
    $git commit -m '更改描述内容'
    $git remote set-url origin   https://token@github.com/xxxxxx/xxxxx.github.io.git
    $git push -u origin master
    
    ```






参考：https://blog.csdn.net/weixin_44471490/article/details/107544139

