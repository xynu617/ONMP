
Entware 是一个适用于嵌入式系统的软件包库，使用 opkg 包管理系统进行管理，现在在官方的源上已经有超过 2000 个软件包了，可以说是非常的丰富

官方地址：[Entware](https://entware.net/)

## U盘格式化

我们的设备本身的储存较少，而且如果哪天崩了，数据还有找不回的风险，所以我们一般把软件包和程序安装到U盘之类的外置设备上，所以需要对它进行格式化，NTFS 格式我个人不推荐使用

[如何在路由器上格式化 U 盘为 ext4](https://github.com/xzhih/ONMP/wiki/如何在路由器上格式化-U-盘为-ext4)

## 一键安装

用以下命令可以实现一键安装，此脚本只支持 ext4 分区

运行命令，选择要安装到的分区，等待安装完成

```shell 
$ sh -c "$(curl -kfsSL https://raw.githubusercontent.com/xynu617/ONMP/master/lede-ent.sh)"
```

如果提示找不到 curl 命令，可以用下面这个方法

```shell
$ cd /tmp
$ wget -c -O https://raw.githubusercontent.com/xynu617/ONMP/master/lede-ent.sh
$ chmod +x ./lede-ent.sh
$ ./lede-ent.sh
```

一键安装如果出错，或者重启后 Entware 失效，可以使用下面的手动安装

## 手动安装

手动安装可一键安装不同，一键安装使用软连接 opt 的方式，而手动则是以挂载点的方式，可以说更稳定

### U盘挂载

分区、格式都没问题之后，开始挂载

```bash
$ mkdir /mnt/onmp
# 挂载方法1
$ mount -t ext4 /dev/sda1 /mnt/onmp/
# 这样就挂载上了
$ df -h
Filesystem                Size      Used Available Use% Mounted on
/dev/sda1               975.5M      2.5M    906.6M   0% /mnt/onmp
# 可以看到已经挂载

# 挂载方法2（推荐）
$ vi /etc/fstab # 按一下i编辑文件
# <file system> <mount point> <type> <options> <dump> <pass>
/dev/sda1 /mnt/onmp ext4 defaults 0 1 # 添加这一行
# 按一下Esc再输入冒号`:`，输入wq回车保存
$ mount -a # 以后每次要挂载就直接输入这个命令
```

开机自动挂载

```bash
$ vi /etc/rc.local # 编辑，vim基本用法和上面一样
mount -a # 在exit 0之前添加命令，开机后自动执行挂载
exit 0
```

### 安装和使用 Entware

**1. 挂载opt**

在 U 盘上创建一个空的 opt 文件夹

```bash
$ mkdir /mnt/onmp/opt
```

在系统根目录创建opt文件夹，并绑定U盘的opt文件夹

```bash
$ mkdir /opt
$ mount -o bind /mnt/onmp/opt /opt
# 可以用 mount 或 df -h 命令查看是否挂载成功
```

**2. 运行 Entware 安装命令**

不同的 CPU 平台有不同的命令，可以使用 `uname -m` 命令查看平台

- armv5

```shell
$ wget -O - http://bin.entware.net/armv5sf-k3.2/installer/generic.sh | /bin/sh
```

- armv7

```shell
$ wget -O - http://bin.entware.net/armv7sf-k3.2/installer/generic.sh | /bin/sh
```

- armv8 (aarch64)

```shell
$ wget -O - http://bin.entware.net/aarch64-k3.10/installer/generic.sh | /bin/sh
```

- x86-64

```shell
$ wget -O - http://bin.entware.net/x64-k3.2/installer/generic.sh | /bin/sh
```

- x86-32

```shell
$ wget -O - http://bin.entware.net/x86-k2.6/installer/generic.sh | /bin/sh
```

- MIPS

```shell
$ wget -O - http://bin.entware.net/mipselsf-k3.4/installer/generic.sh | /bin/sh
```

在输入命令之后之后会自己跑起来，出现以下结果就代表成功，没成功的记得把 U 盘上的 opt 文件夹清空再来

```bash
Info: Congratulations!
Info: If there are no errors above then Entware was successfully initialized.
```

**3. 开机启动**

编辑 `/etc/rc.local` 将以下代码加在 `exit 0` 之前，`mount -a` 之后

```bash
mkdir -p /opt
mount -o bind /mnt/onmp/opt /opt
/opt/etc/init.d/rc.unslung start
```

**4. 环境变量**

编辑 `/etc/profile` 在他的最后加入以下代码

```bash
. /opt/etc/profile
```

这样开机之后将会添加 `/opt/bin` 和 `/opt/sbin` 到环境变量 PATH 里

**5. 重启**

重启之后，可以使用一下命令检查是否成功

```bash
# 检查环境变量
$ echo $PATH
/opt/bin:/opt/sbin:/usr/sbin:/usr/bin:/sbin:/bin # 可以看到已经有 /opt 的路径了

# 检查 `/opt` 挂载情况
$ df -h
/dev/sda1               975.5M     13.9M    895.2M   2% /mnt/onmp # U 盘挂载成功
/dev/sda1               975.5M     13.9M    895.2M   2% /opt # opt 挂载成功

# opkg 更新数据
$ opkg update
Downloading http://pkg.entware.net/binaries/x86-64/Packages.gz # 默认从 entware 下载
Updated list of available packages in /opt/var/opkg-lists/packages # 成功
```

经过以上步骤，已经可以从 `Entware` 上进行下载安装包并安装到U盘上

这下可以享受丰富的软件包，还不占用内部储存空间，非常适合 LEDE 软路由
我的 onmp 一键包也可以在 LEDE 上使用了

## Tips

每次升级固件后如果失效了，重新设置开机启动和环境变量即可

**参考**

[Install on Synology NAS](https://github.com/Entware/Entware/wiki/Install-on-Synology-NAS)

[How To Configure Routers Asus RT-N56U/RT-N65U For Entware Usage](https://bitbucket.org/padavan/rt-n56u/wiki/EN/HowToConfigureEntware)