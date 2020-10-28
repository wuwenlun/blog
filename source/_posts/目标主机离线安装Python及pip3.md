---
title: 目标主机离线安装Python及pip3
date: 2020-09-24 17:09:27
categories:
- [Python]
tags:
- Python
---

将Python项目文件打包到无法联网的主机上部署执行

![](https://gitee.com/wuwenlun/img-bed/raw/master/img/20201028143859.png)

<!-- more -->

### 前言

最近遇到了一个场景：需要将Python项目文件打包到无法联网的主机上部署执行，本篇文章记录针对于该场景的处理方案。

说明：

源主机（可联网）：安装了Python3和pip3

目标主机（无法联网）：需安装和源主机相同的Python版本和pip3，部署执行项目文件

主机系统为centos，Python版本为3.5.2，通过虚拟环境+pip进行迁移

### 目标主机离线安装Python及pip3

### 源主机中下载所需包

**Python3**

首先，下载Python3，可以在[官网](https://link.zhihu.com/?target=https%3A//www.python.org/downloads/source/)或者通过源主机（可联网的其它主机）wget：

```text
wget --no-check-certificate https://www.python.org/ftp/python/3.5.2/Python-3.5.2.tgz
```

**Python3依赖包**

然后，需要下载Python3的依赖包，可以通过centos镜像中去copy，不过我更推荐用yum生成的方式：

```text
yum install zlib-devel bzip2-devel openssl-devel ncurses-devel  epel-release gcc gcc-c++ xz-devel readline-devel gdbm-devel sqlite-devel tk-devel db4-devel libpcap-devel libffi-devel --downloadonly --downloaddir=/packages
```

命令执行完毕，你就会在/packages目录下发现所需的所有.rpm文件。

如果，源主机中已经安装了这些依赖，那么你可以用：

```text
yum reinstall zlib-devel bzip2-devel openssl-devel ncurses-devel  epel-release gcc gcc-c++ xz-devel readline-devel gdbm-devel sqlite-devel tk-devel db4-devel libpcap-devel libffi-devel --downloadonly --downloaddir=/packages
```

打包：

```text
zip -r packages.zip packages/
```

### 目标主机中安装

将Python-3.5.2.tgz和packages.zip上传至目标主机。

首先，安装Python3依赖：

```text
unzip packages.zip
cd packages/
rpm -Uvh  *.rpm --nodeps --force
```

然后，安装Python3：

```text
tar -zxvf Python-3.5.2.tgz

mkdir /usr/local/python3
cd Python-3.5.2 
./configure --prefix=/usr/local/python3                    # 将Python3安装在/usr/local/python3
make && make install                                       # 编译安装

ln -s /usr/local/python3/bin/python3 /usr/bin/python3      # 创建python3软链接
ln -s /usr/local/python3/bin/pip3 /usr/bin/pip3            # 创建pip3软链接
```

安装完毕，可通过：

```text
Python3 -V
pip3 -V
```

查看并检查安装的版本

### 源主机中打包项目文件

若项目中创建虚拟环境，首先激活虚环境，然后进入项目文件，执行：

```text
pip3 freeze > requirements.txt
```

将当前项目中的库列表生成并保存在requirements.txt中。

然后，通过pip生成批量离线安装包（whl文件）：

```text
pip wheel --wheel-dir=./tmp/packages -r requirements.txt
```

执行完毕之后，你会发现/tmp/packages中包含了项目所需的所有.whl

打包项目文件：

```text
zip A.zip A/
```

### 目标主机中部署

上传A.zip至目标主机，创建虚环境，并激活（python3 自带了venv）：

```text
python3 -m venv test_venv
cd test_venv
source bin/activate
```

解压项目代码A.zip，并切换：

```text
unzip A.zip
cd A/
```

安装项目Python依赖模块：

```text
pip3 install --no-index --find-links=tmp/packages -r requirements
```

安装完毕，检查：

```text
pip3 freeze
```

当然你也可以通过python命令行import进行检验哈哈。

最后，执行项目启动脚本（startup.sh）部署：

```text
chmod +x  ./startup.sh
nohup ./startup.sh > a-log 2>&1 &
```

以上，就完成了整个项目的迁移部署。

### 参考资料

* [https://zhuanlan.zhihu.com/p/114290069](https://zhuanlan.zhihu.com/p/114290069)

