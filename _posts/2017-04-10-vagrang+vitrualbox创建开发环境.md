---
layout: post
book: true
background-image: http://ot1cc1u9t.bkt.clouddn.com/17-7-16/91630214.jpg
category: 工具
title: vagrant+vitrualbox打造开发环境
tags:
- vagrant
- vitrualbox
---

安装 vagrant 和 vbox
---
```
https://www.virtualbox.org/wiki/Downloads
https://www.vagrantup.com/downloads.html
```

下载虚拟机镜像
---
镜像是后缀名为 .box 文件，如下载一个 centos7.0比较纯净的版本,总大小只有 400多M:

```
https://github.com/tommy-muehle/puppet-vagrant-boxes/releases/download/1.1.0/centos-7.0-x86_64.box

```

添加镜像
---
 
```
vagrant box add centos7 /data/soft/centos-7.0-x86_64.box
```

查看当前镜像列表
---

```
vagrant box list
```

关于 vagrant 的命令,可以使用 ```vagrant list-commonds``` 查看它所有的命令
  
box 相关的操作，可查看: ```vagrant box list-commonds```,<br />
   ***add: 添加 box 文件到镜像列表<br />***
   ***list: 列出所有的镜像<br />***
   ***outdated: 检查当前项目使用的box是否有更新<br />***
   ***prune:<br />***
   ***remove: 从镜像列表移除某个<br />***
   ***repackage: 重打包box到当前目录,其中3个参数由vagrant box list获取<br />***
	***update: 从远程更新<br />***

自定义配置
---
创建虚拟机目录 /Develop/centos7 及虚拟机描述文件 Vagrantfile 内容如下:

```
  # -*- mode: ruby -*-
  # vi: set ft=ruby :

  # Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
  VAGRANTFILE_API_VERSION = "2"

  Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
      config.vm.box = "centos7"
      config.vm.network "private_network", ip: "192.168.123.123"
      config.vm.provider "virtualbox" do |vb|
        vb.customize ["modifyvm", :id, "--memory", "2048"]
      end
  end
```

设定好IP和该虚拟机要使用的 box 名称,还可以把本地电脑上的目录绑定到虚拟机上。这样可以实现在本地开发,虚拟机里文件同步的功能。方便调试。

还可以让虚拟机启动后执行一些脚本:

```
# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
    config.vm.box = "edengdev-CentOS6.4x86_64"
    config.vm.network "private_network", ip: "192.168.192.192"
    config.vm.synced_folder "../../sites", "/home/httpd/sites", create:true, owner:"www", group:"www"
    config.vm.provision "shell", inline: "/etc/init.d/nginx.sh start", run: "always"
    config.vm.provision "shell", inline: "/etc/init.d/php-fpm start", run: "always"
    config.vm.provider "virtualbox" do |vb|
      vb.customize ["modifyvm", :id, "--memory", "2048"]
    end
end
```

>这里在绑定目录时添加了 owner, group 参数,表明了它所属的用户和组。这样就不会受本机目录权限的影响导致程序运行时出错。

启动虚拟机
---

```
vagrant up	// 启动
vagrant ssh	// 登录
```


导出虚拟机
---
我们可以通过 ```vagrant package``` 命令把当前运行的镜像导出到一个box,发给别人使用(当然是指虚拟机里面环境安装好了之后)。用法如下：

```
vagrant package -h
	Usage: vagrant package [options] [name|id]
	Options:
		--base NAME                  虚拟机在 VirtualBox 中的名字,如: Html_default_1522149094762_41259
		--output NAME                导出后,box的名称
		--include FILE,FILE..        和镜像一起导出的文件，比如项目配置文件，代码文件等
		--vagrantfile FILE           和镜像一起导出的 Vagrantfile 文件，方便别人使用。因为里面可能已经设置了许多启动脚本，目录关联。
          
```

示例:

```
vagrant package --base Html_default_1522149094762_41259 --output lnmp.box --vagrantfile Vagrantfile --include btjdxx_xinqigu_com.conf,demo_xinqigu_com.conf
```
导出的时候它会先关闭虚拟机.如果不指定 --base ，它会找到当前目录所处的虚拟机。

导出完毕后，当前目录下就会多一个 box 了,可以把它添加到镜像列表，然后再创建一个虚拟机试试。

```
vagrant box add centos7_lnmp lnmp.box
```

在另外一个目录:

```
vagrant init centos7_lnmp
vagrant up
```

启动后 ```vagrant shh``` 进入虚拟机查看，发现 ip, 其它配置都和之前的一样，而且之前配置的vagrantfile里的设置也在该虚拟机上实现，添加的文件也在。导出成功!

删除虚拟机
---

```
vagrant halt	// 停止虚拟机
vagrant destroy	// 删除虚拟机
vagrant box remove centos7_lnmp	// 从镜像列表移除
```

关联已经存在的虚拟机
---
>本地使用 virtualbox 创建的虚拟机，然后使用 vagrant 来作为简易的管理。

同样的场景可能是：但有一天没关虚拟机就关电脑了，导致一些文件丢失，其中就有 vagrant 用来和虚拟机关联的，.vagrant/ 目录。

使用 vagrant init 命令可以创建一些配置。它会在当前目录下创建一个隐藏目录，叫 .vagrant，结构如：```.vagrant/machines/default/virtualbox/```

最终文件夹下的文件分别是：

```
action_provision
action_set_name
id
synced_folders
```

其中 id  文件中的内容就是和它关联的虚拟机的唯一ID，该文件夹丢失，导致使用 vagrant status 来查看虚拟机时，总是提示未创建。但是该虚拟机在 virtualbox 中确实是存在的，于是，用如下方法来手动关联：

先查看当前所有的 virtualbox 虚拟机：

```
VBoxManage list vms
"nginx_web_conf_default_1417763838159_49997" {b969dad8-37d8-4237-8d3c-a01243bb91b3}
"postdev-servers_default_1416493203261_51912" {00f2a72e-3431-430a-a1c6-25132ecdba63}
```

每一行的前面双引号中是虚拟机的名称，后面花括号中的虚拟机的ID。

然后再将要关联的虚拟机的ID，写入上面说的对应 vagrant 目录下对应的 id 文件中，如：

```
echo -n "b969dad8-37d8-4237-8d3c-a01243bb91b3" > ~/Develop/edeng/.vagrant/machines/default/virtualbox/id
```

然后再查看虚拟机状态：

```
vagrant status

Current machine states:

default                   poweroff (virtualbox)
```

发现现在是关机状态，而不再是未创建，这时候就可以成功启动虚拟机了：

```
vagrant status
```