========================
团队和仓库标签
========================

.. image:: https://governance.openstack.org/badges/kuryr.svg
    :target: https://governance.openstack.org/reference/tags/index.html

.. Change things from this point on

===============================
kuryr
===============================

.. image:: https://raw.githubusercontent.com/openstack/kuryr/master/doc/images/kuryr_logo.png
    :alt: Kuryr mascot
    :align: center


用于OpenStack Neutron的Docker

Kuryr是一个Docker网络插件，它使用Neutron来为Docker容器提供网络服务。它为常用的Neutron插件提供了容器化镜像。


* 免费软件：Apache协议（License）
* 文档：https://docs.openstack.org/kuryr/latest/
* 源码：https://git.openstack.org/cgit/openstack/kuryr
* Bug：https://bugs.launchpad.net/kuryr

特性
--------

* 待续


获取代码
------------

::

    $ git clone https://git.openstack.org/openstack/kuryr.git
    $ cd kuryr

前提条件
-------------

::

    $ sudo pip install -r requirements.txt


安装Kuryr的libnetwork驱动
------------------------------------

关于kuryr-libnetwork驱动的安装，请参考：

https://docs.openstack.org/kuryr-libnetwork/latest/readme.html


配置Kuryr
-----------------

执行以下命令生成示例配置 ``etc/kuryr.conf.sample`` ：

::

    $ tox -e genconfig


重命名配置文件，并把配置文件复制到指定路径：

::

    $ cp etc/kuryr.conf.sample /etc/kuryr/kuryr.conf


编辑 ``/etc/kuryr/kuryr.conf`` 文件中的keystone部分，替换ADMIN_PASSWORD：

::

    www_authenticate_uri = http://127.0.0.1:35357/v2.0
    admin_user = admin
    admin_tenant_name = service
    admin_password = ADMIN_PASSWORD


在 ``/etc/kuryr/kuryr.conf`` 文件中去掉 ``bindir`` 参数的注释，配置为Kuryr vif绑定的可执行文件的路径。

::

    bindir = /usr/local/libexec/kuryr

默认情况下，Kuryr会使用veth对来执行绑定。然而，Kuryr库（Library）附带了另外两个驱动，你可以在 **binding** 部分中配置这两个驱动：

    [binding]
    #driver = kuryr.lib.binding.drivers.ipvlan
    #driver = kuryr.lib.binding.drivers.macvlan

驱动可以使用其他 **binding** 选项。前面代码片段中Kuryr库的两个驱动都可以进行进一步配置，设置那些将会作为虚拟设备链路接口的接口。

    link_iface = enp4s0


运行Kuryr
-------------

现在，Kuryr使用一个bash脚本来启动服务。在执行以下命令之前，确定你已经安装了 ``tox`` 。

::

    $ sudo ./scripts/run_kuryr.sh

启动之后，请重启你的Docker服务，例如：

::

    $ sudo service docker restart

如果缺失以下文件，bash脚本会创建这个文件。

* ``/usr/lib/docker/plugins/kuryr/kuryr.json`` ：libnetwork的Json规范文件。

注意，使用 `pyroute2 <http://docs.pyroute2.org/>`_ 来执行veth对的创建和删除命令时，需要root权限。

测试Kuryr
-------------

创建一个网络来快速验证Kuryr是否正常工作：

::

    $ docker network create --driver kuryr test_net
    785f8c1b5ae480c4ebcb54c1c48ab875754e4680d915b270279e4f6a1aa52283
    $ docker network ls
    NETWORK ID          NAME                DRIVER
    785f8c1b5ae4        test_net            kuryr

用tox来测试：

::

    $ tox

你还可以使用 ``-e`` 标记来执行特定的测试用例，例如，只执行 *fullstack* 测试用例。

::

    $ tox -e fullstack

生成文档
------------------------


我们使用 `Sphinx <https://pypi.python.org/pypi/Sphinx>`_ 来维护文档。你可以使用pip来安装Sphinx。

::

    $ pip install -U Sphinx

除了Sphinx，你还需要以下东西（ ``requirements.txt`` 中没有涵盖的）

    $ pip install openstackdocstheme reno 'reno[sphinx]'

文档源码在 *doc* 目录中，你可以使用以下命令来生成html文件。如果文档生成成功， *doc* 目录下会创建一个 *build/html* 目录。

::

    $ cd doc
    $ make html

现在你就可以把http://localhost:8080作为一个简单的网站来提供文档了。

::

    $ cd build/html
    $ python -m SimpleHTTPServer 8080
