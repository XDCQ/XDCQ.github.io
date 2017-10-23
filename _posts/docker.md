---
layout: post
title: Docker
description: ~~~
image: assets/images/02.jpg
---

# docker
## 1.什么是docker
&emsp;&emsp;Docker最初是dotCloud公司创始人Solomon Hykes在法国期间发起的一个公司内部项目,它是基于dotCloud	公司多年云服务技术的一次革新,并于2013年3月以Apache2.0授权协议开源,主要项目代码在GitHub上进行维护。
&emsp;&emsp;Docker使用Go开发,基于Linux内核的cgroup,namespace,以及AUFS类的	UnionFS等技术,对进程进行封装隔离,属于操作系统层面的虚拟化技术。由于隔离的进程独立于宿主和其它的隔离的进程,因此也称其为容器。最初实现是基于LXC,从0.7	以后开始去除LXC,转而使用自行开发的	libcontainer,从	1.11开始,则进一步演进为使用runC和containerd。
&emsp;&emsp;Docker在容器的基础上,进行了进一步的封装,从文件系统、网络互联到进程隔离等等,极大的简化了容器的创建和维护。使得	Docker技术比虚拟机技术更为轻便、快捷。
### 1.1传统的虚拟机结构（一类和二类）
拥有一个Hypervisor作为虚拟机管理程序
Hypervisor分为两类
<img src="https://upload.wikimedia.org/wikipedia/commons/5/53/VMM-Type1.JPG" width = "300" height = "200" alt="1" align=center />
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;一类Hypervisor
目前的VM使用这种结构，Hypervisor直接运行在主机的硬件来控制硬件和管理客体操作系统上。需要硬件支持，开启虚拟化。
<img src="https://upload.wikimedia.org/wikipedia/commons/1/1a/VMM-Type2.JPG" width = "300" height = "200" alt="2" align=center />
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;二类Hypervisor
早期VM使用这种结构，在本机操作系统上虚拟出一套硬件后,在其上运行一个完整操作系统,在该系统上再运行所需应用进程；

### 1.2docker的结构
<img src="http://dockerone.com/uploads/article/20160503/bbd6d8b6ca1556f737b452b72d105c78.png" width = "450" height = "300" alt="docker" align=center />
###1.3docker vs VMs
<img src="http://cdn3.infoqstatic.com/statics_s2_20171017-0336-1/resource/articles/docker-core-technology-preview/zh/resources/0731013.jpg" width = "450" height = "300" alt="3" align=center />
###1.4为什么要使用docker
作为一种新兴的虚拟化方式,Docker跟传统的虚拟化方式相比具有众多的优势。

1.更高效的利用系统资源
由于容器不需要进行硬件虚拟以及运行完整操作系统等额外开销,Docker对系统资源的利用率更高。
2.更快速的启动时间
传统的虚拟机技术启动应用服务往往需要数分钟,而Docker容器应用,由于直接运行于宿主内核,无需启动完整的操作系统,因此可以做到秒级、甚至毫秒级的启动时间。
3.一致的运行环境
开发过程中一个常见的问题是环境一致性问题。由于开发环境、测试环境、生产环境不一致,导致有些bug并未在开发过程中被发现。而Docker的镜像提供了除内核外完整的运行时环境,确保了应用运行环境一致性.
4.持续交付和部署
5.更轻松的迁移

对比传统虚拟机总结:
| 特性        | 容器   |  虚拟机  |
| --------   | -----:  | :----:  |
| 启动      | 秒级   |   分钟级     |
| 硬盘使用        |    一般为MB    |   一般为GB   |
| 性能        |    接近原生    |  弱于  |
| 系统支持量        |    单机支持上千个容器    |  一般几十个  |

## 2.docker的使用
### 2.1docker的基本概念
Docker	包括三个基本概念
.镜像(Image)
.容器(Container)
.仓库(Repository）

#### 2.1.1镜像(Image)
&emsp;&emsp;而Docker镜像(Image),就相当于是一个root文件系统。比如官方镜像ubuntu就包含了完整的一套Ubuntu最小系统的root文件系统。
&emsp;&emsp;Docker镜像是一个特殊的文件系统,除了提供容器运行时所需的程序、库、资源、配置等文件外,还包含了一些为运行时准备的一些配置参数(如匿名卷、环境变量、用户等。

----------
分层存储
因为镜像包含操作系统完整的root文件系统,其体积往往是庞大的,因此在Docker设计时,就充分利用UnionFS的技术,将其设计为分层存储的架构。所以严格来说,镜像并非是像一个ISO那样的打包文件,镜像只是一个虚拟的概念,其实际体现并非由一个文件组成,而是由一组文件系统组成,或者说,由多层文件系统联合组成。镜像构建时,会一层层构建,前一层是后一层的基础。每一层构建完就不会再发生改变,后一层上的任何改变只发生在自己这一层。比如,删除前一层文件的操作,实际不是真的删除前一层的文件,而是仅在当前层标记为该文件已删除。在最终容器运行的时候,虽然不会看到这个文件,但是实际上该文件会一直跟随镜像。因此,在构建镜像的时候,需要额外小心,每一层尽量只包含该层需要添加的东西,任何额外的东西应该在该层构建结束前清理掉。
----------

#### 2.1.2容器(Container)
&emsp;&emsp;镜像和容器的关系，就是类的实例的关系，镜像是静态的定义，容器是运行时的实体，容器可以被创建、启动、停止、删除、暂停等。
容器的实质是进程,但与直接在宿主执行的进程不同,容器进程运行于属于自己的独立的命名空间。因此容器可以拥有自己的root文件系统、自己的网络配置、自己的进程空间,甚至自己的用户ID空间。

每运行一个容器会在镜像为基础层上创建一个容器存储层，容器存储层的生存周期和容器一样,容器消亡时,容器存储层也随之消亡。

#### 2.1.3仓库(Repository）
一个Docker Registry中可以包含多个仓库(Repository);每个仓库可以包含多个标签(Tag);每个标签对应一个镜像。
镜像名字包含三段：
    username/registry_name:tag
    
### 2.2使用镜像
```
docker search   #查找镜像
docker pull     #拉取镜像
docker images   #列出镜像
docker run      #运行，--link使用1.环境变量2./etc/hosts
docker commit   #提交更改，在原有镜像的基础上,再叠加上容器的存储层,并构成新的镜像
docker diff     #容器的修改，ADC
```
docker images列表中的镜像体积总和并非是所有镜像实际硬盘消耗。由于Docker镜像是多层存储结构,并且可以继承、复用,因
此不同镜像可能会因为使用相同的基础镜像,从而拥有共同的层。由于	Docker使用Union FS,相同的层只需要保存一份即可,因此实际镜像硬盘占用空间很可能
要比这个列表镜像大小的总和要小的多。
虚悬镜像 dangling image 和中间层镜像
```
docker rmi	$(docker images -q -f dangling=true)#-f为filter，过滤
docker images -a
docker images -f since=mongo:3.2#before，label
--format
```

```
docker commit [选项] <容器ID或容器名> [<仓库名>[:<标签>]]
```
不要使用docker commit定制镜像,定制行为应该使用Dockerfiler来完成。commit每次增加一层使得镜像更加臃肿。


### 2.3网络
![](leanote://file/getImage?fileId=59ed9f7dab644170d2001a04)
```
docker daemon -H 0.0.0.0:2375
```
*映射容器端口到宿主主机的实现
容器所有到外部网络的连接,源地址都会被NAT成本地系统的IP地址。这是使用iptables的源地址伪装操作实现的。
```
$sudo iptables -t nat -nL
Chain	POSTROUTING	(policy	ACCEPT)
target					prot	opt	source															destination
MASQUERADE		all		--		172.17.0.0/16							!172.17.0.0/16
```
其中,上述规则将所有源地址在172.17.0.0/16网段,目标地址为其他网段(外部网络)的流量动态伪装为从系统网卡发出。MASQUERADE跟传统SNAT
的好处是它能动态从网卡获取地址。
*外部访问容器实现
容器允许外部访问,可以在docker run 时候通过-p或-P参数来启用。不管用那种办法,其实也是在本地的iptable的nat表中添加相应的规则。使用-P时:
```
$iptables -t nat -nL
Chain	DOCKER	(2	references)
target					prot	opt	source															destination
DNAT							tcp		--		0.0.0.0/0												0.0.0.0/0												tcp	dpt:49153	to:172.17.0.2:80
```
### 2.4备份
commit 容器->镜像
save 镜像->持久化
export 容器->持久化
###2.5回滚
docker tag <LAYER ID> <IMAGE NAME>
```
docker logs
docker ps
docker images
docker rm 
docker rmi
docker inspect
docker stats
docker run 
docker start/stop
docker export/import
docker save/load
docker attach
docker exec
docker cp
```
