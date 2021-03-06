.. _install-run-minikube:

======================
安装和运行minikube
======================

minikube
=================

最简单体验完整的Kubernetes集群功能是使用 `minikube <https://github.com/kubernetes/minikube>`_ ，这是一个在单个节点上运行Kubernetes的方式，可以用于测试Kubernetes以及本地开发应用。

minikube是通常通过虚拟机来运行的，也就是说必须在主机上运行Hypervisor，可以是Virtualbox (跨平台) 也可以是 KVM (Linux) 或者 Hyper-V (Windows)，甚至可以使用非常小众的 HyperKit (macOS)。

.. note::

   之所以minikube通常需要在虚拟机中运行，是因为minikub是基于Docker容器运行的，Docker容器需要在Linux上运行，所以会先要求运行一个Linux虚拟机，然后才能在虚拟机中运行Docker以满足运行minikube的条件。

如果你是在Linux主机上，由于Linux可以直接运行Docker，所以甚至可以不需要Hypervisor的虚拟机，直接运行在Linux物理主机的Docker中。此时使用参数 ``--vm-driver=none`` 来运行。

安装Minikube
===================

Ubuntu平台安装MiniKube
----------------------------

.. note::

   这里实践采用Linux环境，已经安装部署了KVM。具体方法请参考 :ref:`studio`

安装命令::

   curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && sudo install minikube-linux-amd64 /usr/local/bin/minikube

macOS平台安装MiniKube
--------------------------

macOS安装hyperkit
~~~~~~~~~~~~~~~~~~~~

.. note::

   macOS的minikube可以选择virtualbox作为后端，也可以选择 `xhyve <https://github.com/moby/hyperkit>`_ 作为后端。目前我倾向于使用xhyve，以便使用macOS内置的kvm虚拟化。

- 为了在块设备后端支持qcow，需要安装OCaml `OPAM <https://opam.ocaml.org/>`_ 开发环境带有qcow模块::

   brew install opam libev
   opam init
   eval `opam config env`
   opam install uri qcow.0.10.4 conduit.1.0.0 lwt.3.1.0 qcow-tool mirage-block-unix.2.9.0 conf-libev logs fmt mirage-unix prometheus-app

.. note::

   为了能够在编译hyperkit之前找到ocaml环境，必须在编译前执行一次 ``opam config env``

   可以通过以下命令移除之前旧版本的 ``mirage-block-unix`` 的 ``pin`` 或者 ``qcow`` ::

      opam update
      opam pin remove mirage-block-unix
      opam pin remove qcow

- 安装HyperKit::

   git clone https://github.com/moby/hyperkit
   cd hyperkit
   make

.. note::

   二进制执行程序位于 ``build/hyperkit`` 。为了能够让 ``docker-machine-driver-hyperkit`` 找到hyperkit可执行程序，请将这个目录加入到环境变量，例如 ``~/.bash_profile`` 。

macOS安装minikube
~~~~~~~~~~~~~~~~~~~

- 通过brew安装minikube(我没有使用这个方法)::

   brew cask install minikube

- 直接安装罪行版本minikube（我使用这个方法）::

   curl -Lo minikube https://storage.googleapis.com/minikube/releases/v0.35.0/minikube-darwin-amd64 && chmod +x minikube && sudo cp minikube /usr/local/bin/ && rm minikube

.. note::

   每次启动minikube默认会检查是否有可用最新版本

启动minikube
==================

- （不推荐直接）启动minikube集群::

   minikube start

Linux平台使用kvm后端
-------------------------

.. note::

   minikube默认使用Virtualbox作为驱动，所以如果简单使用上述命令，会首先下载virtulbox镜像来运行。这可能和你的安装环境不同。所以需要参考 `Driver plugin installation <https://github.com/kubernetes/minikube/blob/master/docs/drivers.md#kvm2-driver>`_ 来安装驱动产检，并指定驱动来启动minikube。

   ``以下案例将采用KVM作为驱动来运行minikube``

   注意：我的实验室环境已经按照 :ref:`kvm_docker_in_studio` 安装了KVM驱动所需的 ``libvirt-clients libvirt-daemon-system qemu-kvm`` ，所以只需要安装 ``docker-machine-driver-kvm2`` 就可以。

- 安装 ``docker-machine-driver-kvm2`` 驱动::

   curl -LO https://storage.googleapis.com/minikube/releases/latest/docker-machine-driver-kvm2 \
     && sudo install docker-machine-driver-kvm2 /usr/local/bin/

- 直接使用kvm2驱动启动的命令如下::

   minikube start --vm-driver kvm2

- 不过，更好的方法是先指定默认驱动kvm2，然后再启动就不需要传递参数了::

   minikube config set vm-driver kvm2
   minikube start

此时会下载minikube的KVM镜像，然后运行这个虚拟机，通过 ``virsh list`` 可以看到系统新启动了一个KVM虚拟机::

   Id    Name                           State
   ----------------------------------------------------
   5     minikube                       running

.. note::

   创建的minikube配置: ``CPUs=2, Memory=2048MB, Disk=20000MB``

   ``minikube start`` 运行指令显示输出::

      kubectl is now configured to use "minikube"

   这表明当前Linux主机的kubectl已经被配置直接使用刚才所安装运行的minikube

macOS平台使用hyperkit后端
-----------------------------

- 安装Hyperkit驱动::

   brew install docker-machine-driver-hyperkit

   # docker-machine-driver-hyperkit need root owner and uid 
   sudo chown root:wheel /usr/local/opt/docker-machine-driver-hyperkit/bin/docker-machine-driver-hyperkit
   sudo chmod u+s /usr/local/opt/docker-machine-driver-hyperkit/bin/docker-machine-driver-hyperkit

- (建议跳过这步，用下一步采用先配置再启动)使用Hyperkit后端启动::

   minikube start --vm-driver hyperkit

- 使用hyperkit作为默认后端::

   minikube config set vm-driver hyperkit

- 启动minikube::

   minikube start

使用minikube
===============

- ssh登陆minikub方法::

   minikube ssh

停止和再次启动minikube
==========================

安装了minikube之后，通过 ``minikube stop`` 可以停止，然后通过 ``minikube start`` 可以再次启动。

.. note::

   每次启动minikube，系统都会尝试重新连接Google仓库更新镜像，所以需要先搭好梯子
