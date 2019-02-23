---
layout: post
title: ansible modules
description:
image: assets/images/ansible.jpeg
---

# 常用模块

- setup
用于统计系统的相关信息
```
# 例子
ansible test -m setup
# 参数
  - filter： 过滤要显示的内容
  - ansible test -m setup -a "filter=ansible_python_version" -k # 只获取主机ansible_python_version的值
  - 更多的帮助 ansible-doc -s setup
```

- file
设置文件属性
```
# 参数
  - force: 在两种情况下会强制创建软连接，默认值为 no
  - 源文件不存在但之后会建立的情况下
  - 目标软连接已存在，需要先取消之前的软连，然后创建新的软链
  - group：定义文件的属组
  - mode：定义文件的权限
  - owner：定义文件的属主
  - path：必选项，定义文件的路径
  - recurse：递归设置文件的属性，只对目录生效（-R）
  - src：要被链接的源文件的属性，只应用于state=link的情况
  - dest：被链接到的路径，只应用与state=link的情况
  - state：
  - directory：如果目录不存在创建目录
  - file：即使文件不存在，也不会被创建
  - link：创建软连接
  - hard：创建硬链接
  - touch：如果文件不存在，则会创建一个新的文件，如果文件存在，则会更新其最后修改时间
  - absent：删除目录，文件或者取消链接文件
 
# 例子
ansible test -m file -a 'src=/etc/fstab dest=/tmp/fstab state=link' -k                              # 等于 ln -s /etc/fstab /tmp/fstab
ansible test -m file -a 'path=/tmp/fstab state=absent' -k                                           # 删除文件
ansible test -m file -a 'path=/tmp/lixin.txt state=touch owner=nobody group=nobody mode=666' -k     # 创建文件指定其属主，属组，权限
```

 - copy
 复制文件到远程主机，每次备份会产生一个md5sum，如果两次赋值文件的md5sum相同，那么就不会再次执行复制动作。
 
```
 # 参数
  - backup：在覆盖之前将原文件备份，备份文件名包含时间信息。有两个选项：yes | no
  - src：源文件，如果是目录的话，会递归赋值，包括目录本身，如果目录为/，那么只会复制目录下的文件
  - dest：目标文件路径
  - others：所有file模块里的选项都可以在这里使用
  - content：用于替代'src'，可以直接设定文件的值
  - directory_mode：递归设定目录的权限，默认为系统默认权限
  - force：如果目标主机包含该文件，但内容不同，如果设置为yes，则强制覆盖，如果为no，则只有当前目标主机的目标位置不存在该文件时，才复制，默认为yes。
 
# 例子：
ansible test -m copy -a "content='hello world' dest=/tmp/tmp.txt" -k    # 在目录主机上创建tmp.txt文件，内容为hello world
```
 
- command
在远程主机上执行命令。（默认模块）
 
```
 # 参数
  - creates：文件名，当该文件存在时，则不执行后面的命令
  - chdir：执行命令前，先修改当前路径
  - removes：文件名，当文件不存在时，则不执行后面的命令
  - executable：切换shell来执行命令，该执行路径必须是一个绝对路径
```
 
- shell
和command模块相同，参数也相同。
 
```
 # command，shell，raw，script模块同属于commands类，都是用来执行命令，不同的是：
  - command模块执行的命令中不能包含：|，<,>,& 符号。
  - shell模块:可以执行远程服务器上的shell脚本文件，也支持 管道符等，脚本需要使用绝对路径。
  - raw模块和shell模块相同，可以执行任意命令就像在本机一样，相当于ssh之直接执行Linux命令，不会进入到ansible的模块子系统中。
  - script模块，用来执行一个shell脚本，其实将管理端的shell脚本copy到被管理端上然后执行，相当于 scp + shell 的组合。(需要在脚本所在目录下执行)
# 官方建议使用command模块，如果需求不满足，可以使用raw模块
```

 
 - service
用于管理服务

```
 # 参数
  - arguments: 给命令行提供一些选项
  - enable：是否开机启动 yes | no
  - name：服务名称
  - pattern：定义一个模式，如果通过status命令来查看服务的状态时，没有响应，就会通过ps指令在进程中根据该模式进行查找，如果匹配到，则认为该服务已经在运行，否则会认为未启动( ps aux | grep `pattern` )
  - runlevel：运行级别
  - sleep：如果执行了，restarted，则在 stop 和 start 之间沉睡几秒钟
  - state：started、stoped、restarted和reloaded，其中started和stoped是幂等的，也就是说如果服务已经停止，那么运行stoped不会执行任何操作
 
# 例子
ansible test -m service -a 'name=nginx state=started enabled=yes' -k                   # 启动nginx进程，并设置为开机启动
ansible test -m service -a 'name=nginx state=stopped' -k                               # 关闭nginx进程
ansible test -m service -a 'name=network state=restarted arguments=eth0' -k            # 重启network进程，并传递eth0 作为参数，即：重启eth0网卡
ansible test -m service -a 'name=nginx pattern=/usr/local/nginx state=started' -k      # 如果无法使用 service nginx status查寻到nginx的状态，那么会使用 ps 来过滤 pattern指定的关键字，如果存在，则表示程序已经正常启动　

```

 - cron
 用于管理定时任务(crontab 来操作)
```
# 参数
  - backup: 对远程主机上的原计划内容修改之前做备份
  - cront_file: 如果指定该选项，则用该文件替换远程主机上的cron.d目录下的用户计划任务
  - day: 日(1-31,*,*/2,......)
  - hour: 小时(0-23,*,*/2,......)
  - minute: 分钟(0-59,*,*/2,......)
  - mouth: 月(1-12,*,*/2,......)
  - weekday: 周(0-7,*,......)
  - job: 要执行的任务，依赖于state=present
  - name: 该任务的描述，必选项。
  - special_time：指定什么时候执行（被触发），参数：reboot，yearly，annually，monthly，weekly，daily，hourly。
  - state：确认该任务计划是创建还是删除 Present(启用) | Absent(停用)
  - user：以哪个用户的身份执行
 
# 例子
ansible test -m cron -a "hour=3 month=2 day=1 job='echo hello' name='test' state=present " -k  # 添加一条定时任务，* 3 1 2 * echo hello
ansible test -m cron -a 'name=test state=absent' -k                                            # 删除name为test的计划任务

```

- filesystem
用于在块设备上创建文件系统。
```
# 参数
  - dev：目标块设备
  - force：在一个已有的文件系统的设备上强制创建
  - fstype：文件系统类型
  - opts：传递给mkfs命令的选项
```

- yum

```
# 参数
  - config_file：yum的配置文件
  - disable_gpg_check: 关闭gpg_check
  - disablerepo: 不启用某个源
  - enablerepo：启用某个源
  - list：列出repo源
  - name：软件包的名称，可以为一个url路径，或者本地一个rpm包的路径
  - state：状态
    # 安装
    - present
    - installed *
    - latest
    # 删除
    - absent
    - removed *
 
# 例子
ansible test -m yum -a 'name=httpd state=installed' -k     # yum 安装 httpd　　
```
- user
用于管理用户（useradd,userdel） 也可以用command模块来执行shell命令创建和管理用户
```
# 参数
  - home：指定用户的家目录
  - createhome: 是否创建用户家目录 yes | no
  - groups：用户的属组
  - uid：用户的uid
  - password：用户的密码
  - name：用户的名称
  - system：是否是系统用户
  - remove：是否删除用户家目录
  - state：具体的操作， 删除/添加， present | absent
  - shell： 指定用户的shell　
```

- synchronize
使用rsync同步文件
```
# 参数
  - archive：是否进行归档，默认为yes，相当于同时开启recursive,links,perms,times,owner,group -D等选项
  - checksum：是否校验和
  - copy_links：是否复制链接文件
  - delete：删除源中没有而目标存在的文件
  - dest_port: 对方用于接收的端口
  - dirs: 非递归传输目录
  - mode：模式，推和拉模式， push | pull ,默认为push(推)
  - src：同步的数据源的位置
  - rsync_opts: 指定rsync的选项，多个选项可以用逗号分隔
  - dest：目标文件
  - compress: 默认为yes，表示在文件同步过程中是否启用压缩<br>
# 例子
ansible test -m synchronize -a 'src=/etc/yum.repos.d/epel.repo dest=/tmp/epel.repo' -k                  # rsync 传输文件
ansible test -m synchronize -a 'src=/tmp/123/ dest=/tmp/456/ delete=yes' -k                             # 类似于 rsync --delete
ansible test -m synchronize -a 'src=/tmp/123/ dest=/tmp/test/ rsync_opts="-avz,--exclude=passwd"' -k    # 同步文件，添加rsync的参数-avz，并且排除passwd文件
ansible test -m synchronize -a 'src=/tmp/test/abc.txt dest=/tmp/123/ mode=pull' -k                      # 把远程的文件，拉到本地的/tmp/123/目录下　　
```

- mount
用于磁盘挂载相关操
```
# 参数
  - fstype：挂载文件的类型
  - name：挂载点
  - opts：传递给mount命令的参数
  - src：要挂载的文件
  - state：
    - present：只会在 fstab 中添加，不会操作磁盘
    - mounted：自动创建挂载点并挂载
    - unmounted：卸载
    - absent：只会在 fstab 中删除，不会操作磁盘
 
# 例子
ansible test -m mount -a 'fstype=ext4 src=/dev/loop0 name=/aaa state=mounted' -k   # 挂在/dev/loop0 设备
```

- template
模版，用于将模版文件渲染后，输出到远程主机上，模版文件一般以.j2为结尾，标识其是一个jinja2模版文件
```
# 参数
  - src：模版文件的路径
  - dest：拷贝到远程主机上的位置
  - mode：权限
  - attributes: 特殊权限 类似于 chattr
  - force：存在覆盖
  - group：属组
  - owner：属主
```

- get_url
从互联网下载数据到本地，作用类似于Linux下的curl命令。
```
# 参数
  - url：必传参数，文件的下载地址
  - dest：必传参数，文件保存的绝对路径
  - mode：文件的权限mode
  - othes：所有file模块里的选项都可以在这里使用
  - checksum：文件的校验码
  - headers：传递给下载服务器的 HTTP Headers
  - backup：如果本地已经存在同名的配置文件，先行备份
  - timeout：下载的超时时间
```

- unarchive
用于解压文件，其作用类似于Linux下的tar命令，默认情况下unarchive的作用是将控制节点的压缩包拷贝到远程服务器上，然后进行解压
```
# 参数
  - remote_src: 用来表示需要解压的文件存在远程服务器中还是本地服务器中，默认为no，表示解压前先将控制节点上的文件复制到远程主机上中，然后再进行解压。
  - src：指定压缩文件的路径，该选项取决于remote_src的取值，如果remote_src取值为yes,则src指定的是远程服务器中压缩包的地址，如果remote_src取值为no，则src指向的是控制节点中的路径
  - dest：该选项指定的是远程服务器上的绝对路径，表示压缩文件解压的路径
  - list_files: 默认情况下该选项的取值为no，如果该选项取值为yes，也会解压缩文件，并且在ansible的返回值中列出压缩包里的文件；
  - exclude：解压文件时排出exclude选项指定的文件或目录列表
  - keep_newer: 默认值为False，如果该选项为True，那么当目标地址中存在同名文件，并且文件比压缩包中的文件更新时，不进行覆盖。
  - owner：文件或目录解压以后的所有者
  - group: 文件或目录解压以后所属的群组
  - mode: 文件或目录解压以后的权限
```

- git
在远程服务器上执行git相关操作。依赖于git命令，需要在远程主机上先进性安装。
```
# 参数
  - repo：远程git库的地址，可以是一个git协议，ssh协议或者http协议的git库地址
  - dest：必选参数，git库clone到本地服务器以后保存的绝对路径
  - version：克隆远程git库的版本，取值可以为HEAD，分支名称，tag的名称，也可以是一个commit的hash值
  - foces：默认为no，当该选项为yes时，如果本地git库有修改，将会抛弃本地的修改(强制覆盖)
  - accept_hostkey: 当该选项取值为yes时，如果git库服务器不再know_hosts中，则添加到know_hosts中，key_file指定克隆远程git库地址时使用的私钥。　
```


- stat
用于获取远程服务器上的文件信息，类似于Linux下的stat命令
```
# 参数
  - path：用于指定文件或目录的路径　
```

- sysctl
与Linux下的sysctl命令相似，用来控制Linux内核参数
```
# 参数：
  - name：需要设置的参数
  - value：需要设置的值
  - sysctl_file: sysctl.conf文件的绝对路径，默认路径为/etc/sysctl.conf
  - reload：该选项可以取值为 yes 或 no，默认为yes，用于表示设置完以后是否需要实行sysctl -p的操作
 
# 例子
ansible test -m sysctl -a 'name=vm.overcommit_memory value=1'  # 修改vm.overcommit_memory的值为1
```

