---
layout:     post
title:      Ubuntu Desktop 安装教程
subtitle:   Ubuntu Desktop 初始化
date:       2020-08-22
author:     Eagle Cool
header-img: img/post-bg-ios9-web.jpg
catalog: 	true
tags:       Linux Ubuntu
---

# 常用环境配置

* 将镜像仓库地址修改为 阿里云镜像  [阿里云 Ubuntu 仓库镜像](https://developer.aliyun.com/mirror/ubuntu?spm=a2c6h.13651102.0.0.56401b11pSD6UH)
* 安装常用软件  
  ```shell
  sudo apt update && sudo apt install -y vim \
    tree \
    net-tools \
    curl \
    open-vm-tools # 虚拟机窗口适配
  ```
* 更新语言包
* 安装 Typora
  ```shell
  wget -qO - https://typora.io/linux/public-key.asc | sudo apt-key add -
  sudo add-apt-repository 'deb https://typora.io/linux ./'
  sudo apt-get update
  sudo apt-get install -y typora
  ```
* 安装 chrome 浏览器
  * wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
  * sudo dpkg -i 包名
* [使用 ibus 中文输入法](https://blog.csdn.net/ZhangRelay/article/details/105891175)
* 设置 root 密码
  ```shell
  sudo passwd root
  ```
* 添加环境变量 PATH:
  ```shell
  echo "export PATH=/home/eagle/application/bin:\$PATH" >> ~/.bashrc 
  source ~/.bashrc 
  ```
# 开发环境配置
* 在 home 目录下创建如下结构的文件目录
  * idea-workspace : idea 的工作目录
    * workspace-eagle: 存放个人项目
    * workspace-work: 存放工作项目
    * workspace-test: 存放测试项目 
    * workspace-view: 存放所有工作项目, 用于全局查看和搜索
* 安装 openjdk 8
  ```shell
  sudo apt install -y openjdk-8-jdk openjdk-8-source 
  ```
* 安装 maven
  ```shell
  sudo apt install -y maven
  ```
  * 新建 ~/.m2/settings.xml 文件, 并做如下配置: > ~/.m2/settings.xml
    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <settings xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.1.0 http://maven.apache.org/xsd/settings-1.1.0.xsd" xmlns="http://maven.apache.org/SETTINGS/1.1.0"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
    
      <profiles>
          <profile>
            <!-- id必须唯一 -->
            <id>aliyun</id>
            <repositories>
                <repository>
                    <!-- id必须唯一 -->
                    <id>aliyun-central</id>
                    <!-- 仓库的url地址 -->
                    <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
                    <releases>
                        <enabled>true</enabled>
                    </releases>
                    <snapshots>
                        <enabled>true</enabled>
                        <updatePolicy>always</updatePolicy>
                    </snapshots>
                </repository>
            </repositories>
        </profile>
      </profiles>
      <activeProfiles>
        <activeProfile>aliyun</activeProfile>
      </activeProfiles>
    </settings>
    ```
* 安装 git
  ```shell
  sudo apt install -y git
  git config --global user.name "覃国强" 
  git config --global user.email "qinguoqiang@xiaomi.com"  
  git config --global color.ui auto
  git config --global core.editor vim
  git config --global pull.rebase true
  ssh-keygen -t rsa -C "qinguoqiang@xiaomi.com"
  
  # gitee 还需执行
  ssh -T git@gitee.com
  ```
  * 将 id_rsa.pub 的内容添加到 github 和 gitlab 和 gitee
  * 配置git提交快捷方式
    ```
    cat <<EOF > ~/application/bin/git-push
    #!/bin/bash
    git add .
    git commit -m"Feature: git auto push"
    git push
    EOF
    chmod 777 git-push
    ```
* 安装 postman
  * wget https://dl.pstmn.io/download/latest/linux64  -->  https://www.postman.com/downloads/ 下载安装包, tar.gz
  * 解压到 ~/application/Postman
  * 下载 postman 图标到 ~/application/Postman/ :  ![postman](https://s1.ax1x.com/2020/08/23/d00ASg.png)
  * 在 ~/.local/share/applications/ 下创建 postman.desktop
    ```
    cat <<EOF >  ~/.local/share/applications/postman.desktop
    [Desktop Entry]
    Name=postman
    Exec=/home/eagle/application/Postman/Postman
    Icon=/home/eagle/application/Postman/postman.png
    Terminal=false
    Type=Application
    EOF
    ```
* 安装 idea
  * 下载安装包: https://www.jetbrains.com/idea/download/#section=linux
  * 解压到 ~/application/idea-${version}/idea-work; ~/application/idea-${version}/idea-test; ~/application/idea-${version}/idea-view
  * 修改 idea64.vmoptions
    ```
    -Xms2g
    -Xmx4g
    -XX:ReservedCodeCacheSize=512m
    -XX:+UseG1GC
    ```
  * 修改 idea.properties
    ```properties
    idea.config.share.path=${user.home}/.idea-2020.1.4
    idea.config.custom.path=${idea.config.share.path}/idea-work
    idea.config.path=${idea.config.custom.path}/config
    idea.system.path=${idea.config.custom.path}/system
    idea.plugins.path=${idea.config.share.path}/plugins
    idea.log.path=${idea.system.path}/log
     
    idea.max.intellisense.filesize=51200
    ```
  * 在 ~/.local/share/applications/ 下创建 2020.1.4-work.desktop
    ```
    # work.desktop
    cat <<EOF >  ~/.local/share/applications/2020.1.4-work.desktop
    [Desktop Entry]
    Name=2020.1.4-work
    Exec=/home/eagle/application/idea-2020.1.4/idea-work/bin/idea.sh
    Icon=/home/eagle/application/idea-2020.1.4/idea-work/bin/idea.svg
    Terminal=false
    Type=Application
    StartupWMClass=2020.1.4-work
    EOF
    
    # test.desktop
    cat <<EOF >  ~/.local/share/applications/2020.1.4-test.desktop
    [Desktop Entry]
    Name=2020.1.4-test
    Exec=/home/eagle/application/idea-2020.1.4/idea-test/bin/idea.sh
    Icon=/home/eagle/application/idea-2020.1.4/idea-test/bin/idea.svg
    Terminal=false
    Type=Application
    StartupWMClass=2020.1.4-test
    EOF
    ```
  * 安装插件
    * .ignore
    * Alibaba Java Coding Guidelines
    * codehelper.generator
    * json format
    * Lombok
    * Maven Helper
    * Rainbow Brackets
    * Native Terminal
    * Thrift Support
    * Translation
    * CheckStyle
    * MybatisPlus
  * setting 设置
    * appearance & Behavior
      - Appearance 
        - use custom font: courier 10 pitch ; size: 16
        - tool windows
          - side-by-size layout on the right
        - presentation mode
          - font size: 16
      - System Settings
        - confirm before exiting the IDE
        - reopen projects on startup
    * Editor
      - General
        - Auto import
          - add unambiguous
          - optimize imports
        - code folding
          - java → one-line methods
      - font
      - code style
        - line separator: unix and macos (\n)
        - java 导入 [intellij-java-google-style.xml](/download/intellij-java-google-style.xml)
      - file and code templates
        - includes file header
          ```
          /**
           * @author 覃国强
           * @date ${YEAR}-${MONTH}-${DAY} ${HOUR}:${MINUTE}
           */
          ```
        - file encodings : utf-8, transparent native-to-ascii
    * Build, Execution, Deployment
      * build tools : any changes
        - maven
          - maven home directory
          - user settings file
          - local repository
          - runner
            - environment variables: MAVEN_HOME=/home/eagle/application/maven/3.6.3
      * comliler: 1024
    * tools
      * terminal
        - run commands using IDE
    * other settings
      * native terminal plugin
        - deepin: /usr/bin/deepin-terminal
        - Ubuntu: gnome-terminal
      * translation
        - 使用 transla.google.com
        - 快速文档实时翻译
* 安装 Jekyll，本地 Blog 测试
  * [Jekyll on Ubuntu](https://jekyllrb.com/docs/installation/ubuntu/)
    ```shell script
    sudo apt install -y ruby-full build-essential zlib1g-dev
    mkdir -p ~/application/gems/
    echo 'export GEM_HOME="$HOME/application/gems"' >> ~/.bashrc
    echo 'export PATH="$HOME/application/gems/bin:$PATH"' >> ~/.bashrc
    source ~/.bashrc
    gem install jekyll bundler
    
    # 将 Ruby Gems 镜像 修改为 阿里云镜像
    gem sources --remove https://rubygems.org/
    gem sources -a https://mirrors.aliyun.com/rubygems/
    ```