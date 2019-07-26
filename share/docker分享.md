#### 1.docker？

> 美 [ˈdɑːkər]  搬运工，图像小海豚

Docker 使用 Google 公司推出的` Go 语言 `进行开发实现，基于 Linux 内核的 `cgroup`，`namespace`，以及 AUFS 类的` Union FS `等技术，`对进程进行封装隔离`，属于 `操作系统层面的虚拟化技术`。由于隔离的进程独立于宿主和其它的隔离的进程，因此也称其为容器。

总结：docker是一个属于操作系统曾米娜的虚拟化技术，由于隔离的进程独立于宿主和其他的隔离的进程，所以也被称作容器(隔离)，`之后提起docker都是docker是一个容器`

#### 2.相比于虚拟机(Virtual Machines)有什么不同？有什么优势？

结构图比较：

![virtualization](https://ws3.sinaimg.cn/large/005uxB9Yly1g3zh53ckxoj30j8079q33.jpg)

![image](https://wx1.sinaimg.cn/large/005uxB9Yly1g3zh6dda56j30j505ft91.jpg)

上面图片比较了 Docker 和传统虚拟化方式的不同之处。传统虚拟机技术是虚拟出一套硬件后，在其上运行一个完整操作系统，在该系统上再运行所需应用进程；而容器内的应用进程直接运行于宿主的内核，容器内没有自己的内核，而且也没有进行硬件虚拟。因此容器要比传统虚拟机更为轻便。

####3.下载安装(略)

