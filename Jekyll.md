# Jekyll使用方法

&nbsp;&nbsp;一种简单静态博客生成器，用于将纯文本转换为静态博客网站。

<!-- TOC -->

  - [1. Jekyll的安装](#1-jekyll的安装)
    - [1.1 Windows 10环境下安装Jekyll](#11-windows-10环境下安装jekyll)
    - [1.2 Ubuntu环境下安装Jekyll](#12-ubuntu环境下安装jekyll)
  - [2. 快速开始](#2-快速开始)
  - [3. 配置修改](#3-配置修改)
    - [3.1 预览地址](#31-预览地址)
  - [参考文档](#参考文档)

<!-- /TOC -->

---
## 1. Jekyll的安装

### 1.1 Windows 10环境下安装Jekyll

- 下载Ruby
  - 版本至少为2.4
  - 官方网址 (https://rubyinstaller.org)
  - 笔者使用的是rubyinstaller-devkit-2.6.6-2-x64 ([点此下载](https://github.com/oneclick/rubyinstaller2/releases/download/RubyInstaller-2.6.6-2/rubyinstaller-devkit-2.6.6-2-x64.exe))
- 下载Python
  - 官方网址 (https://www.python.org)
  - 笔者使用的是python-3.9.0-amd64 ([点此下载](https://www.python.org/ftp/python/3.9.0/python-3.9.0-amd64.exe))
  - 需要在安装过程中将Python加入路径
- 下载Jekyll和bundler
  - 启动命令行窗口 (Win+R，输入cmd后确定)
  - gem install jekyll bundler

### 1.2 Ubuntu环境下安装Jekyll

- 下载Ruby 
  - 版本至少为2.4，apt下载的版本为2.0
  - 笔者使用的是Ruby2.5.0，安装流程：
    - wget https://cache.ruby-lang.org/pub/ruby/2.5/ruby-2.5.0.tar.bz2
    - ./configure --enable-shared --disable-install-doc
    - sudo make
    - sudo make install
- 下载Python
  - sudo apt install python
- 下载Jekyll和bundler
  - sudo gem install jekyll bundler

## 2. 快速开始

- jekyll new <PATH\>
  - 在指定路径安装一个新的Jekyll站点，路径不为空可以使用--force强制执行。
- cd <PATH\>
- bundle install
- bundle exec jekyll serve
  - 在本地预览服务中编译站点，可以通过默认网址 [localhost:4000](http://localhost:4000) 访问。若想要后台运行，可以添加--detach。
- 示例博客首页：<br> <div style="align: center"><img alt="Jekyll Example" src="../images/Jekyll/2020-12-08-Jekyll_example.png" width="600x"></div>

## 3. 配置修改

### 3.1 预览地址

- 开发环境的默认预览地址是 (http://localhost:4000)，如果想要构建别的地址，请在 _config.yml 中


## 参考文档

[Jekyll使用教程笔记](https://juejin.cn/post/6844903623567081486)  
[如何使用Jekyll+GitHub Pages搭建个人博客站点](https://blog.csdn.net/u010454030/article/details/79908682)  
[用 jekyll + Github Pages搭建个人博客](https://blog.csdn.net/u013553529/article/details/54588010) 


[返回主页](./index.md)
