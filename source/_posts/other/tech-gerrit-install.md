---
title: Gerrit安装与配置
date: 2016-05-14 13:46:46
category: [其它]
tags: git
toc: true
---
## 简介
Gerrit是一套代码审核服务
本人尝试了两套linux系统，其中ubuntu系统为虚拟机环境，centos系统为正式环境。
两套linux环境均为64位操作系统。
1. ubuntu 14.10 amd-64 server版
2. CentOS Linux release 7.1.1503 (Core)

<!-- more -->

## 软件环境
安装gerrit所需的相关软件包

### jdk/jre
 - ubuntu 安装Java（jre） 
 
 {% asset_img image001.png %}
 - centos 
 `# yum -y list java*`
 列出可安装的软件包
 `# yum -y install java-1.7.0-openjdk.x86_64`
 安装jdk 7
### git
 - ubuntu
 ` $ sudo apt-get install git git-core git-doc`
 {% asset_img image002.png %}
 
 {% asset_img image011.png %}

### apache2
#### ubuntu
```bash
$ sudo apt-get install apache2 apache2.2-common apache2-utils
```
其中，使用htpasswd指令需要安装apache2-utils包

{% asset_img image003.png %}

{% asset_img image005.png %}

安装成功后，在/etc/apache2/sites-enable/目录下，新建<kbd>gerrit.conf</kbd>用于配置gerrit服务
ubuntu下，需要手动添加LoadModule proxy_http_module
添加后 <kbd>/etc/apache2/mods-enabled/proxy.load</kbd>内容如下：

{% asset_img image007.png %}

#### centos
`# yum install httpd`
安装成功后，在**/etc/httpd/conf.d/**目录下，新建gerrit.conf用于配置gerrit服务

#### apache配置
<kbd>gerrit.conf</kbd>内容如下：

```apache
# NameVirtualHost 已不推荐使用，在ubuntu下，必须存在，centos可以注释掉
NameVirtualHost *:80
<VirtualHost *:80>
		# 服务器的ip地址
        ServerName 192.168.133.102
        ProxyRequests Off
        ProxyVia Off
        ProxyPreserveHost On
        <Proxy *>
                Order deny,allow
                Allow from all
        </Proxy>
        <Location "/gerrit/login/">
                AuthType Basic
                AuthName "Gerrit Code Review"
                AuthBasicProvider file
                AuthUserFile /home/gerrit/htpasswd.conf
        </Location>
        AllowEncodedSlashes On
        RedirectMatch ^gerrit$ /gerrit/
        ProxyPass /gerrit/ http://127.0.0.1:8081/gerrit/
        ProxyPassReverse /gerrit/ http://127.0.0.1:8081/gerrit/
</VirtualHost>
```

注意
 - AllowEncodedSlashes On设置将无法在gerrit上创建parent/child.git结构的project。
 - 在centos上出现过AuthUserFile无法读取的问题。新建gerrit专有账户后，无此问题。
 - gerrit.conf的内容与gerrit/etc/gerrit.config有关联。gerrit的配置请见后文

## Gerrit安装
### 下载gerrit
因为国内code.google比较难访问，建议先在windows上先下载gerrit安装包然后上传到服务器

gerrit release包列表：http://gerrit-releases.storage.googleapis.com/index.html

### 安装

1. 创建专有账号（可选）
 在centos上，创建了专有账号gerrit及gerrit组，并且禁止gerrit账户登录系统。
 ```shell
[root@Centos ~]# adduser gerrit
[root@Centos ~]# passwd --delete gerrit
[root@Centos ~]# sudo su - gerrit
[gerrit@Centos ~]$ ls
 ```
 第2行清除用户的密码

2. 安装 

 ```bash
[root@Centos ~]# java -jar gerrit-2.8.6.1.war init -d gerrit
```
 
 进入gerrit安装交互过程
 ```apache
    *** Gerrit Code Review 2.8.6.1
    ***
    Create '/root/gerrit'          [Y/n]? y
    *** Git Repositories
    ***
    Location of Git repositories   [git]: ~/git/
    *** SQL Database
    ***
    Database server type           [h2]:
    *** User Authentication
    ***
    Authentication method          [OPENID/?]: HTTP
    Get username from custom HTTP header [y/N]? n
    SSO logout URL                 :
    *** Email Delivery
    ***
    // Email 
    *** Container Process
    ***
    Run as                         [root]:
    Java runtime                   [/usr/lib/jvm/java-1.7.0-openjdk-1.7.0.85-2.6.1.2.el7_1.x86_64/jre]:
    Copy gerrit-2.8.6.1.war to /root/gerrit/bin/gerrit.war [Y/n]? y
    Copying gerrit-2.8.6.1.war to /root/gerrit/bin/gerrit.war
    *** SSH Daemon
    ***
    Listen on address              [*]:
    Listen on port                 [29418]:
    Gerrit Code Review is not shipped with Bouncy Castle Crypto SSL v149
      If available, Gerrit can take advantage of features
      in the library, but will also function without it.
    Download and install it now [Y/n]? y
    Downloading http://www.bouncycastle.org/download/bcpkix-jdk15on-149.jar ... OK
    Checksum bcpkix-jdk15on-149.jar OK
    Gerrit Code Review is not shipped with Bouncy Castle Crypto Provider v149
    ** This library is required by Bouncy Castle Crypto SSL v149. **
    Download and install it now [Y/n]? y
    Downloading http://www.bouncycastle.org/download/bcprov-jdk15on-149.jar ... OK
    Checksum bcprov-jdk15on-149.jar OK
    Generating SSH host key ... rsa... dsa... done
    *** HTTP Daemon
    ***
    Behind reverse proxy           [y/N]? y
    Proxy uses SSL (https://)      [y/N]? n
    Subdirectory on proxy server   [/]: gerrit
    Listen on address              [*]:
    Listen on port                 [8081]:
    Canonical URL                  [http://null]: http://localhost:8081/gerrit/
    *** Plugins
    ***
    Install plugin download-commands version v2.8.6.1 [y/N]? y
    Install plugin reviewnotes version v2.8.6.1 [y/N]? y
    Install plugin replication version v2.8.6.1 [y/N]? y
    Install plugin commit-message-length-validator version v2.8.6.1 [y/N]? y
 ```
 
 建议在第二步 Git Repositories选择独立于gerrit安装目录的一个空目录，不然默认使用gerrit/git/作为git仓库目录。
 然后其它的步骤基本只需默认即可

 不同版本的gerrit，使用的ssh key加密库是不一样的，以下是2.8使用的加密库
 http://www.bouncycastle.org/download/bcprov-jdk16-144.jar
 {% asset_img image013.png %}
 安装完成：
 {% asset_img image015.png %}
 
3. 配置
 <kbd>gerrit/etc/gerrit.config</kbd> 配置如下：
 {% asset_img image017.jpg %}

### 添加gitweb支持
```apache
[gitweb]
        cgi = /var/www/git/gitweb.cgi
		url = http://192.168.133.102/git/
```	
然后修改<kbd>/etc/gitweb.conf</kbd>

设定`$projectroot = /home/gerrit/gitprojects` 指向git仓库路径

重启服务后生效。

 - 注1：个人觉得gitweb.url可以设定路径/var/www/git/也是可行的，但未作尝试。
 - 注2：有人说gitweb cgi的owner必须和 $projectroot的ownwer一致。好像不一致也是可以的，因为我没有去更改gitprojects的所有者信息

## FAQ

 - centos 上访问gerrit出现http 503
 输入以下指令，第一个不行，试第二个
 ```bash
 # /usr/sbin/setsebool httpd_can_network_connect 1
 # /usr/sbin/setsebool -P httpd_can_network_connect 1
 ```
 参考：http://sysadminsjourney.com/content/2010/02/01/apache-modproxy-error-13permission-denied-error-rhel/

 - centos 上访问gerrit出现http 500
 关闭防火墙即可
 ```bash
 # systemctl stop firewalld.service
 ```
 因apache和gerrit的配置问题导致gerrit不能正常访问，建议多多查看apache和gerrit的log。


## 参考
- Gerrit2安装配置
http://openwares.net/linux/gerrit2_setup.html
- Gerrit Code Review for Git
https://gerrit-documentation.storage.googleapis.com/Documentation/2.8.1/index.html

 

