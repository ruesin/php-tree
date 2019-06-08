# Docker

容器其实是一种沙盒技术，沙盒能够像一个集装箱一样，把你的应用“装”起来的技术。应用与应用之间，就因为有了边界而不至于相互干扰；而被装进集装箱的应用，也可以被方便地搬来搬去。

容器本身没有价值，有价值的是“容器编排”。Docker只是实现容器的一种方式，还有其他很多容器实现方案。

容器技术的核心功能，就是通过约束和修改进程的动态表现，从而为其创造出一个“边界”。

对于 Docker 等大多数 Linux 容器来说，Cgroups 技术是用来制造约束的主要手段，而Namespace 技术则是用来修改进程视图的主要方法。

## Namespace（隔离）
Linux Namespace是Linux提供的一种内核级别环境隔离的方法。

很早以前的Unix有一个叫chroot的系统调用（通过修改根目录把用户jail到一个特定目录下），chroot提供了一种简单的隔离模式：chroot内部的文件系统无法访问外部的内容。Linux Namespace在此基础上，提供了对UTS、IPC、mount、PID、network、User等的隔离机制。

Linux下的超级父亲进程的PID是1，所以，同chroot一样，如果我们可以把用户的进程空间jail到某个进程分支下，并像chroot那样让其下面的进程看到的那个超级父进程的PID为1，于是就可以达到资源隔离的效果了。

利用对被隔离应用的进程空间做的手脚，使得原进程只能看到重新计算过的进程编号，比如 PID=1。实际上，他们在宿主机的操作系统里，还是原来的进程编号，不同的PID namespace中的进程无法看到彼此。

Namespace 的使用方式非常简单：其实只是 Linux 创建新进程的一个可选参数。我们知道，在 Linux 系统中创建线程的系统调用是 clone()，比如：
```
int pid = clone(main_function, stack_size, SIGCHLD, NULL);
```
这个系统调用就会为我们创建一个新的进程，并且返回它的进程号 pid。

而当我们用 clone() 系统调用创建一个新进程时，就可以在参数中指定 CLONE_NEWPID 参数，比如：
```
int pid = clone(main_function, stack_size, CLONE_NEWPID | SIGCHLD, NULL); 
```
这时，新创建的这个进程将会“看到”一个全新的进程空间，在这个进程空间里，它的 PID 是 1。之所以说“看到”，是因为这只是一个“障眼法”，在宿主机真实的进程空间里，这个进程的 PID 还是真实的数值，比如 100。

除了我们刚刚用到的 PID Namespace，Linux 操作系统还提供了 Mount、UTS、IPC、Network 和 User 这些 Namespace，用来对各种不同的进程上下文进行“障眼法”操作。

比如，Mount Namespace，用于让被隔离进程只看到当前 Namespace 里的挂载点信息；Network Namespace，用于让被隔离进程看到当前 Namespace 里的网络设备和配置。

这，就是 Linux 容器最基本的实现原理了。

所以，Docker 容器实际上是在创建容器进程时，指定了这个进程所需要启用的一组 Namespace 参数。这样，容器就只能“看”到当前 Namespace 所限定的资源、文件、设备、状态，或者配置。而对于宿主机以及其他不相关的程序，它就完全看不到了。

容器，其实是一种特殊的进程而已。

Namespace 的三个系统调用：
- clone()：实现线程的系统调用，用来创建一个新的进程，并可以通过设计上述参数达到隔离。
- unshare()： 使某进程脱离某个namespace
- setns()：把某进程加入到某个namespace

系统调用参数：
- Mount namespaces：CLONE_NEWNS
- UTS namespaces：CLONE_NEWUTS
- IPC namespaces：CLONE_NEWIPC
- PID namespaces：CLONE_NEWPID
- Network namespaces：CLONE_NEWNET
- User namespaces：CLONE_NEWUSER

跟真实存在的虚拟机不同，在使用 Docker 的时候，并没有一个真正的“Docker 容器”运行在宿主机里面。Docker 项目帮助用户启动的，还是原来的应用进程，只不过在创建这些进程时，Docker 为它们加上了各种各样的 Namespace 参数。

这时，这些进程就会觉得自己是各自 PID Namespace 里的第 1 号进程，只能看到各自 Mount Namespace 里挂载的目录和文件，只能访问到各自 Network Namespace 里的网络设备，就仿佛运行在一个个“容器”里面，与世隔绝。

用户运行在容器里的应用进程，跟宿主机上的其他进程一样，都由宿主机操作系统统一管理，只不过这些被隔离的进程拥有额外设置过的 Namespace 参数。而 Docker 项目在这里扮演的角色，更多的是旁路式的辅助和管理工作。

Namespace 技术实际上修改了应用进程看待整个计算机“视图”，即它的“视线”被操作系统做了限制，只能“看到”某些指定的内容。但对于宿主机来说，这些被“隔离”了的进程跟其他进程并没有太大区别。

根据实验，一个运行着 CentOS 的 KVM 虚拟机启动后，在不做优化的情况下，虚拟机自己就需要占用 100~200  MB 内存。此外，用户应用运行在虚拟机里面，它对宿主机操作系统的调用就不可避免地要经过虚拟化软件的拦截和处理，这本身又是一层性能损耗，尤其对计算资源、网络和磁盘 I/O 的损耗非常大。

而相比之下，容器化后的用户应用，却依然还是一个宿主机上的普通进程，这就意味着这些因为虚拟化而带来的性能损耗都是不存在的；而另一方面，使用 Namespace 作为隔离手段的容器并不需要单独的 Guest OS，这就使得容器额外的资源占用几乎可以忽略不计。

容器只是运行在宿主机上的一种特殊的进程，多个容器之间使用的是同一个宿主机的操作系统内核。

尽管你可以在容器里通过 Mount Namespace 单独挂载其他不同版本的操作系统文件，比如 CentOS 或者 Ubuntu，但这并不能改变共享宿主机内核的事实。这意味着，如果你要在 Windows 宿主机上运行 Linux 容器，或者在低版本的 Linux 宿主机上运行高版本的 Linux 容器，都是行不通的。

在 Linux 内核中，有很多资源和对象是不能被 Namespace 化的，最典型的例子就是：时间。

## CGROUP（限制）
Namespace解决的主要是环境隔离的问题，这只是虚拟化中最最基础的一步，我们还需要解决对计算机资源使用上的隔离。虽然通过Namespace把我Jail到一个特定的环境中去了，但是我在其中的进程使用用CPU、内存、磁盘等这些计算资源其实还是可以随心所欲的。所以，还需要对进程进行资源利用上的限制或控制。

Linux CGroup全称Linux Control Group，是Linux内核的一个功能，用来限制、控制与分离一个进程组群的资源（如CPU、内存、磁盘输入输出等）。

Cgroup 可以为系统中运行的任务（进程）的用户定义组群分配资源，比如CPU时间、系统内存、网络带宽等。可以监控配置的cgroup，拒绝cgroup访问某些资源。

Cgroups 还能够对进程进行优先级设置、审计，以及将进程挂起和恢复等操作。

Cgroup 的主要功能：
- Resource limitation: 限制资源使用，比如内存使用上限以及文件系统的缓存限制。
- Prioritization: 优先级控制，比如：CPU利用和磁盘IO吞吐。
- Accounting: 一些审计或一些统计，主要目的是为了计费。
- Control: 挂起进程，恢复执行进程

在 Linux 中，Cgroups 给用户暴露出来的操作接口是文件系统，即它以文件和目录的方式组织在操作系统的 /sys/fs/cgroup 路径下。可以用 mount 指令`$ mount -t cgroup`或者`$ lssubsys  -m`。

在 /sys/fs/cgroup 下面有很多诸如 cpuset、cpu、 memory 这样的子目录，也叫子系统。这些都是这台机器当前可以被 Cgroups 进行限制的资源种类。

在子系统对应的资源种类下，可以看到该类资源具体可以被限制的方法。比如，对 CPU 子系统来说，可以看到如下几个配置文件：
```
ls /sys/fs/cgroup/cpu
cgroup.clone_children cpu.cfs_period_us cpu.rt_period_us  cpu.shares notify_on_release
cgroup.procs      cpu.cfs_quota_us  cpu.rt_runtime_us cpu.stat  tasks

```

其中 cfs_period 和 cfs_quota 这两个参数需要组合使用，可以用来限制进程在长度为 cfs_period 的一段时间内，只能被分配到总量为 cfs_quota 的 CPU 时间。

### CPU

在CPU子系统下面创建一个目录：
```
root@ubuntu:/sys/fs/cgroup/cpu$ mkdir container
root@ubuntu:/sys/fs/cgroup/cpu$ ls container/
cgroup.clone_children cpu.cfs_period_us cpu.rt_period_us  cpu.shares notify_on_release
cgroup.procs      cpu.cfs_quota_us  cpu.rt_runtime_us cpu.stat  tasks
```

这个目录就称为一个“控制组”。操作系统会在新创建的 container 目录下，自动生成该子系统对应的资源限制文件。

启动一个非常吃CPU的进程：
```
$ while : ; do : ; done &
[1] 226
```
查看 container 目录下的文件，看到 container 控制组里的 CPU quota 还没有任何限制（即：-1），CPU period 则是默认的 100  ms（100000  us）：
```
$ cat /sys/fs/cgroup/cpu/container/cpu.cfs_quota_us 
-1
$ cat /sys/fs/cgroup/cpu/container/cpu.cfs_period_us 
100000
```
修改 cfs_quota 文件为 20  ms（20000  us）：
```
echo 20000 > /sys/fs/cgroup/cpu/container/cpu.cfs_quota_us
```
表示在每 100  ms 的时间里，被该控制组限制的进程只能使用 20  ms 的 CPU 时间，也就是说这个进程只能使用到 20% 的 CPU 带宽。

然后把被限制的进程的 PID 写入 container 组里的 tasks 文件，上面的设置就会对该进程生效了：
```
$ echo 226 > /sys/fs/cgroup/cpu/container/tasks 
```

CGroup的子系统：
- blkio：为​​​块​​​设​​​备​​​设​​​定​​​I/O 限​制，一般用于磁盘等设备；
- CPU：使用调度程序提供对CPU的限制
- cpuacct：自动生成cgroup中任务所使用的CPU报告
- cpuset：为进程分配单独的 CPU 核和对应的内存节点；
- devices：允许或者拒绝 cgroup 中的任务访问设备
- freezer：挂起或者恢复 cgroup 中的任务
- memory：为进程设定内存使用的限制。
- net_cls：使用等级识别符（classid）标记网络数据包，可允许 Linux 流量控制程序（tc）识别从具体 cgroup 中生成的数据包
- 名称空间子系统

Linux Cgroups 可以理解为一个子系统目录加上一组资源限制文件的组合。而对于 Docker 等 Linux 容器项目来说，它们只需要在每个子系统下面，为每个容器创建一个控制组（即创建一个新目录），然后在启动容器进程之后，把这个进程的 PID 填写到对应控制组的 tasks 文件中就可以了。

而至于在这些控制组下面的资源文件里填上什么值，就靠用户执行 docker run 时的参数指定了，比如这样一条命令：
```
$ docker run -it --cpu-period=100000 --cpu-quota=20000 ubuntu /bin/bash
```
启动容器后，查看 Cgroups 文件系统下，CPU 子系统中，“docker”这个控制组里的资源限制文件的内容：
```
$ cat /sys/fs/cgroup/cpu/docker/5d5c9f67d/cpu.cfs_period_us 
100000
$ cat /sys/fs/cgroup/cpu/docker/5d5c9f67d/cpu.cfs_quota_us 
20000
```

一个正在运行的 Docker 容器，其实就是一个启用了多个 Linux Namespace 的应用进程，而这个进程能够使用的资源量，则受 Cgroups 配置的限制。

这是容器技术中一个非常重要的概念：容器是一个“单进程”模型。

由于一个容器的本质就是一个进程，用户的应用进程实际上就是容器里 PID=1 的进程，也是其他后续创建的所有进程的父进程。这就意味着，在一个容器中，你没办法同时运行两个不同的应用，除非你能事先找到一个公共的 PID=1 的程序来充当两个不同应用的父进程，这也是为什么很多人都会用 systemd 或者 supervisord 这样的软件来代替应用本身作为容器的启动进程。

## images（文件系统）
Mount Namespace 修改的，是容器进程对文件系统“挂载点”的认知。但是，这也就意味着，只有在“挂载”这个操作发生之后，进程的视图才会被改变。而在此之前，新创建的容器会直接继承宿主机的各个挂载点。

一个解决办法：创建新进程时，除了声明要启用 Mount Namespace 之外，还可以告诉容器进程，有哪些目录需要重新挂载，就比如 /tmp 目录。在容器进程执行前可以添加一步重新挂载 /tmp 目录的操作，在容器进程启动之前，加上`mount(“none”, “/tmp”, “tmpfs”, 0, “”)`。告诉容器以 tmpfs（内存盘）格式，重新挂载 /tmp 目录，/tmp目录的挂载就生效了。

Mount Namespace 对容器进程视图的改变，一定是伴随着挂载操作（mount）才能生效。

在 Linux 操作系统里，有一个名为 chroot 的命令，作用是“change root file system”，即改变进程的根目录到你指定的位置。用法也非常简单。

### 示例1
假设有一个 $HOME/test 目录，想要把它作为一个 /bin/bash 进程的根目录。

```
## 创建一个`test`目录和几个`lib`文件夹
$ mkdir -p $HOME/test
$ mkdir -p $HOME/test/{bin,lib64,lib}
$ cd $T

## 把 bash 命令拷贝到 test 目录对应的 bin 路径下
$ cp -v /bin/{bash,ls} $HOME/test/bin

## 把 bash 命令需要的所有 so 文件拷贝到 test 目录对应的 lib 路径下
$ T=$HOME/test
$ list="$(ldd /bin/ls | egrep -o '/lib.*\.[0-9]')"
$ for i in $list; do cp -v "$i" "${T}${i}"; done
## 执行 chroot 命令，告诉操作系统，我们将使用 $HOME/test 目录作为 /bin/bash 进程的根目录：
$ chroot $HOME/test /bin/bash
```
此时执行 "ls /"，就会看到返回的是 $HOME/test 目录下面的内容，而不是宿主机的内容。

对于被 chroot 的进程来说，并不会感受到自己的根目录已经被“修改”成 $HOME/test 了。

实际上，Mount Namespace 正是基于对 chroot 的不断改良才被发明出来的，它也是 Linux 操作系统里的第一个 Namespace。

而这个挂载在容器根目录上、用来为容器进程提供隔离后执行环境的文件系统，就是所谓的“容器镜像”。它还有一个更为专业的名字，叫作：rootfs（根文件系统）。

所以，一个最常见的 rootfs，或者说容器镜像，会包括如下所示的一些目录和文件，比如 /bin，/etc，/proc 等等：
```
$ ls /
bin dev etc home lib lib64 mnt opt proc root run sbin sys tmp usr var
```
而进入容器之后执行的 /bin/bash，就是 /bin 目录下的可执行文件，与宿主机的 /bin/bash 完全不同。

对 Docker 项目来说，最核心的原理实就是为待创建的用户进程：
- 启用 Linux Namespace 配置；
- 设置指定的 Cgroups 参数；
- 切换进程的根目录（Change Root）。

需要明确的是，rootfs 只是一个操作系统所包含的文件、配置和目录，并不包括操作系统内核。在 Linux 操作系统中，这两部分是分开存放的，操作系统只有在开机启动时才会加载指定版本的内核镜像。

所以说，rootfs 只包括了操作系统的“躯壳”，并没有包括操作系统的“灵魂”，同一台机器上的所有容器，都共享宿主机操作系统的内核。

Docker 在镜像的设计中，使用联合文件系统（Union File System），引入了层（layer）的概念。也就是说，用户制作镜像的每一步操作，都会生成一个层，也就是一个增量 rootfs。

Union File System 也叫 UnionFS，最主要的功能是将多个不同位置的目录联合挂载（union mount）到同一个目录下。

比如，我现在有两个目录 A 和 B，它们分别有两个文件：
```
$ tree
.
├── A
│  ├── a
│  └── x
└── B
  ├── b
  └── x
```
使用联合挂载的方式，将这两个目录挂载到一个公共的目录 C 上：
```
$ mkdir C
$ mount -t aufs -o dirs=./A:./B none ./C
```

查看目录 C 的内容， A 和 B 下的文件被合并到了一起：
```
$ tree ./C
./C
├── a
├── b
└── x
```

可以看到，在这个合并后的目录 C 里，有 a、b、x 三个文件，并且 x 文件只有一份。这，就是“合并”的含义。此外，如果你在目录 C 里对 a、b、x 文件做修改，这些修改也会在对应的目录 A、B 中生效。



### 示例2
对于 AuFS 来说，它最关键的目录结构在 /var/lib/docker 路径下的 diff 目录：
```
/var/lib/docker/aufs/diff/<layer_id>
```

```
## 启动一个容器
$ docker run -d ubuntu:latest sleep 3600
```
Docker 会从 Docker Hub 上拉取一个 Ubuntu 镜像到本地。这个“镜像”，实际上就是一个 Ubuntu 操作系统的 rootfs，它的内容是 Ubuntu 操作系统的所有文件和目录。不过，与之前的 rootfs 稍微不同的是，Docker 镜像使用的 rootfs，往往由多个“层”组成：

```
$ docker image inspect ubuntu:latest
...
     "RootFS": {
      "Type": "layers",
      "Layers": [
        "sha256:f49017d4d5ce9c0f544c...",
        "sha256:8f2b771487e9d6354080...",
        "sha256:ccd4d61916aaa2159429...",
        "sha256:c01d74f99de40e097c73...",
        "sha256:268a067217b5fe78e000..."
      ]
    }
```

可以看到，这个 Ubuntu 镜像，实际上由五个层组成。这五个层就是五个增量 rootfs，每一层都是 Ubuntu 操作系统文件与目录的一部分；而在使用镜像时，Docker 会把这些增量联合挂载在一个统一的挂载点上（等价于前面例子里的“/C”目录）。

这个挂载点就是`/var/lib/docker/aufs/mnt/`
```
/var/lib/docker/aufs/mnt/6e3be5d2ecccae7cc0fcfa2a2f5c89dc21ee30e166be823ceaeba15dce645b3e
```
这个目录里面是一个完整的 Ubuntu 操作系统：
```
$ ls /var/lib/docker/aufs/mnt/6e3be5d2ecccae7cc0fcfa2a2f5c89dc21ee30e166be823ceaeba15dce645b3e
bin boot dev etc home lib lib64 media mnt opt proc root run sbin srv sys tmp usr var
```

前面提到的五个镜像层，又是如何被联合挂载成这样一个完整的 Ubuntu 文件系统的呢？

这个信息记录在 AuFS 的系统目录 /sys/fs/aufs 下面。

首先，通过查看 AuFS 的挂载信息，我们可以找到这个目录对应的 AuFS 的内部 ID（也叫：si）：
```
$ cat /proc/mounts| grep aufs
none /var/lib/docker/aufs/mnt/6e3be5d2ecccae7cc0fc... aufs rw,relatime,si=972c6d361e6b32ba,dio,dirperm1 0 0
```
即，si=972c6d361e6b32ba。

然后使用这个 ID，就可以在 /sys/fs/aufs 下查看被联合挂载在一起的各个层的信息：
```
$ cat /sys/fs/aufs/si_972c6d361e6b32ba/br[0-9]*
/var/lib/docker/aufs/diff/6e3be5d2ecccae7cc...=rw
/var/lib/docker/aufs/diff/6e3be5d2ecccae7cc...-init=ro+wh
/var/lib/docker/aufs/diff/32e8e20064858c0f2...=ro+wh
/var/lib/docker/aufs/diff/2b8858809bce62e62...=ro+wh
/var/lib/docker/aufs/diff/20707dce8efc0d267...=ro+wh
/var/lib/docker/aufs/diff/72b0744e06247c7d0...=ro+wh
/var/lib/docker/aufs/diff/a524a729adadedb90...=ro+wh
```

从这些信息里，我们可以看到，镜像的层都放置在 /var/lib/docker/aufs/diff 目录下，然后被联合挂载在 /var/lib/docker/aufs/mnt 里面。

而且，从这个结构可以看出来，这个容器的 rootfs 由如下图所示的三部分组成：

![images](../images/docker/images.png)

#### 只读层
它是这个容器的 rootfs 最下面的五层，对应的正是 ubuntu:latest 镜像的五层。可以看到，它们的挂载方式都是只读的（ro+wh，即 readonly+whiteout）。

查看分层的内容：
```
$ ls /var/lib/docker/aufs/diff/72b0744e06247c7d0...
etc sbin usr var
$ ls /var/lib/docker/aufs/diff/32e8e20064858c0f2...
run
$ ls /var/lib/docker/aufs/diff/a524a729adadedb900...
bin boot dev etc home lib lib64 media mnt opt proc root run sbin srv sys tmp usr var
```
可以看到，这些层，都以增量的方式分别包含了 Ubuntu 操作系统的一部分。

#### 可读写层

它是这个容器的 rootfs 最上面的一层（6e3be5d2ecccae7cc），它的挂载方式为：rw，即 read write。在没有写入文件之前，这个目录是空的。而一旦在容器里做了写操作，你修改产生的内容就会以增量的方式出现在这个层中。

如果要删除一个文件，为了实现这样的删除操作，AuFS 会在可读写层创建一个 whiteout 文件，把只读层里的文件“遮挡”起来。

比如，你要删除只读层里一个名叫 foo 的文件，那么这个删除操作实际上是在可读写层创建了一个名叫.wh.foo 的文件。这样，当这两个层被联合挂载之后，foo 文件就会被.wh.foo 文件“遮挡”起来，“消失”了。这个功能，就是“ro+wh”的挂载方式，即只读 +whiteout 的含义。

所以，最上面这个可读写层的作用，就是专门用来存放你修改 rootfs 后产生的增量，无论是增、删、改，都发生在这里。而当我们使用完了这个被修改过的容器之后，还可以使用 docker commit 和 push 指令，保存这个被修改过的可读写层，并上传到 Docker Hub 上，供其他人使用；而与此同时，原先的只读层里的内容则不会有任何变化。这，就是增量 rootfs 的好处。

#### Init 层

它是一个以“-init”结尾的层，夹在只读层和读写层之间。Init 层是 Docker 项目单独生成的一个内部层，专门用来存放 /etc/hosts、/etc/resolv.conf 等信息。

需要这样一层的原因是，这些文件本来属于只读的 Ubuntu 镜像的一部分，但是用户往往需要在启动容器时写入一些指定的值比如 hostname，所以就需要在可读写层对它们进行修改。

可是，这些修改往往只对当前的容器有效，我们并不希望执行 docker commit 时，把这些信息连同可读写层一起提交掉。

所以，Docker 做法是，在修改了这些文件之后，以一个单独的层挂载了出来。而用户执行 docker commit 只会提交可读写层，所以是不包含这些内容的。</p>

最终，这 7 个层都被联合挂载到 /var/lib/docker/aufs/mnt 目录下，表现为一个完整的 Ubuntu 操作系统供容器使用。

