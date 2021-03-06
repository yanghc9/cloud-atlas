.. _install-docker:

======================
安装Docker
======================


RHEL/CentOS平台安装Docker
===========================

- 通过发行版安装docker::

   sudo yum install docker

- 启动docker服务::

   sudo systemctl start docker

- 设置docker在操作系统启动时启动::

   sudo systemctl enable docker

Debian/Ubuntu平台安装Docker
===============================

使用Ubuntu发行版的软件仓库安装Docker
-----------------------------------------

Ubuntu默认发行版本 ``docker.io`` 是可以兼容在Ubuntu主推的LXD系统中，但是版本会较Docker官方低一些。安装非常简便::

   sudo apt install docker.io

安装完成后docker服务就已经启动，此时可以使用以下命令查看docker容器::

   docker ps

使用Docker官方提供的Docker CE安装Docker
-----------------------------------------

- Docker官方提供了 `Docker CE for Ubuntu <https://docs.docker.com/install/linux/docker-ce/ubuntu/>`_ ，需要删除掉Ubuntu系统自带 ``docker.io`` 软件之后才可以安装::

   sudo apt-get remove docker docker-engine docker.io

.. note::

   Docker CE on Ubuntu支持 ``overlay2`` 和 ``aufs`` 存储驱动

- 设置软件仓库::

   sudo apt-get update
   # 设置apt使用HTTPS访问软件仓库
   sudo apt-get install \
       apt-transport-https \
       ca-certificates \
       curl \
       software-properties-common

- 添加Docker官方GPG key::

   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

- 设置Docker官方的 ``stable`` 仓库::

   sudo add-apt-repository \
       "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
       $(lsb_release -cs) \
       stable"

- 安装Docker CE::

   sudo apt-get update
   sudo apt-get install docker-ce

- 验证Docker CE::

   sudo docker run hello-world

无需sudo运行docker
======================

使用 ``docker`` 指令连接docker服务默认是通过sock，所以用户需要有对 ``/var/run/docker.sock`` 读写的权限。

- 检查操作系统中 ``docker.sock`` 文件权限::

   $ ls -lh /var/run/docker.sock
   srw-rw---- 1 root docker 0 Feb 11 11:21 /var/run/docker.sock

可以看到 ``/var/run/docker.sock`` 属于 ``docker`` 用户组（ubuntu系统），如果你使用的操作系统不同，可能是其他用户组，如 ``root`` ，则对应加入到相应用户组::

   sudo usermod -aG docker $USER

.. note::

   实践发现上述将用户添加到 ``docker`` 用户组不能立即生效。我尝试直接重启 ``docker`` 服务未生效，实际是重启了操作系统之后才生效。

参考
======

- `Get Docker CE for Ubuntu <https://docs.docker.com/install/linux/docker-ce/ubuntu/>`_
- `How To Install Docker on Ubuntu 16.04 <https://medium.com/@Grigorkh/how-to-install-docker-on-ubuntu-16-04-3f509070d29c>`_
- `Docker Engine on Ubuntu <https://www.ubuntu.com/containers/docker-ubuntu>`_ - Ubuntu主推LXC容器(LXD)，不过也同时支持Docker Engine
