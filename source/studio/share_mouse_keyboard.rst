.. _share_mouse_keyboard:

=======================
主机间共享键盘鼠标
=======================

在 :ref:`ubuntu_on_mbp` 之后，我的日常桌面工作在一台MacBook Pro上完成，桌面操作系统是macOS。使用macOS工作可以节约大量的折腾桌面的时间，可以集中精力做开发和运维工作。不过，既然我 :ref:`ubuntu_on_mbp` 用来模拟大规模的集群部署，Linux的桌面也运行起来，相当于在办公桌上又多了一块屏幕，不充分利用实在太可惜了。

.. note::

   我尝试过很多种共享屏幕的方法，希望能够提高工作效率，至少多一块屏幕可以同时查看文档、浏览网页，同时开发编程，不需要不断切换窗口，可以节约大量时间。

   我试过使用iPad作为第二块屏幕，甚至购买过能耦将macOS的屏幕扩展到iPad上的商用软件，然而，这种通过Lighten线输出屏幕绘制图形窗口非常消耗系统资源，实用性很差。

`Synergy <https://symless.com/synergy>`_ 是一款通过网络共享键盘和鼠标的软件，跨操作系统平台，这样只需要一套键盘鼠标，可以非常容易在不同的图形界面上操作。这样，我就可以把运行Ubuntu桌面的MacBook Pro架设起来，运行浏览器和阅读PDF，而运行macOS的MacBook Pro作为日常开发和运维工作，不仅方便输入，而且还支持剪贴板操作。

.. note::

   `Synergy <https://symless.com/synergy>`_ 最初是开源软件，并且现在也在GitHub上提供源代码。不过，作为商用软件销售的版本非常容易使用，作为支持开发者，非常建议购买。

.. warning::

   使用Synergy需要注意安全性，请在安全环境使用：你的笔记本是通过网络共享由远程键盘鼠标控制的，所以安全的方法是采用OpenSSH端口转发方式来实现加密通讯。在 `Synergy官方 <https://symless.com/synergy>`_ 提供的最新版本增加了加密通讯功能，所以更为安全可靠。

安装Synergy
=================

- 在macOS上安装好Synergy，启动后作为Server端（共享出键盘鼠标）

- Ubuntu的系统作为Client（提供屏幕）
  
先安装依赖库::

   sudo apt install libavahi-compat-libdnssd1 \
   qt5-style-plugins

.. note::

   安装 ``qt5-style-plugins`` 是为了在Xfce4桌面集成支持Qt5程序运行，将会安装相应的Qt5核心库程序包。

.. note::

   在Ubuntu官方仓库中提供的Synergy基于Qt4运行 （参考 `SynergyHowto <https://help.ubuntu.com/community/SynergyHowto>`_ ）；而Synergy官方提供的deb安装包是基于Qt5环境运行。所以上述安装依赖库首先安装Qt5运行库文件。

   比较简单的安装Ubuntu环境Qt5软件库是使用::

      sudo apt install qt5-default

   不过，实际上Xfce4提供了 ``qt5-style-plugins`` 来集成Qt5的程序显示，所以单纯要在Xfce4环境运行Qt5程序，例如 Synergy ,只需要安装 ``qt5-style-plugins`` 就足够::

      sudo apt install qt5-style-plugins

安装下载的deb包::

   sudo dpkg -i synergy_1.10.1.stable_b81+8941241e_ubuntu_amd64.deb

使用Synergy
===============

- 在macOS上启动Synergy，此时会提示需要访问 ``Accessibility`` 设置，即打开 ``System Preferences => Security & Privacy`` 选择 ``Accessibility`` ，通过设置允许 Synergy 控制你的电脑。然后启动 Synergy 就可以配置其为 Server 角色，启动程序后，会监听在网卡接口IP上。

- 在Ubuntu上启动Synergy，选择作为Client，填写 macOS 主机的IP，此时连接上Server会不断被Server拒绝。这是因为在Server上没有配置client的主机名。

- 回到macOS上，点击状态栏上的Synergy图标的 ``show`` 菜单，在管理界面上点击 ``Configure Server...`` 按钮，然后点击拖放右上角的电脑图标（代表Client），拖放到部署界面的位置，然后将Client主机的名字设置成和实际相同（例如，我的Ubuntu主机的名字是 ``xcloud`` ），完成后点击 ``Ok`` 。再次重启 Server端，就可以看到两台服务器建立了连接。

现在可以顺畅使用两台主机，Ubuntu的图形界面就是一块扩展屏幕。
