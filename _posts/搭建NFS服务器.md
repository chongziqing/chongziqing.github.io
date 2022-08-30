### sh1.安装NFS （网络文件系统）

```shell
sudo apt-get install nfs-kernel-server
sudo apt-get install nfs-common
```

### 2. 配置/etc/exports

第一步：sudo vi /etc/exports，文本末添加：

```shell
#NFS服务器的工作目录
#开发板通过NFS挂载文件系统时，根文件系统就要在这个路径
/home/casey/mine/nfs/X210_rootfs *(rw,sync,no_root_squash,no_subtree_check)
```

第二步：终端执行以下命令

```shell
chmod 777 -R /home/casey/mine/nfs/X210_rootfs
sudo showmount -e    //执行后显示 clnt-create : RPC : Program not registered，继续下一步
sudo exportfs -r     //更新
sudo showmount localhost -e //如果依旧显示Program not registered，其实也可以正常工作
                            //正常显示为：Export list for 192.168.1.116
                                         /home/sijifan/mine/nfs/X210_rootfs *
```

### 3. 启用

```shell
sudo /etc/init.d/nfs-kernel-server restart         //重启 NFS 服务
 
//显示如下即表示完成
* Stopping NFS kernel daemon [ OK ]
* Unexporting directories for NFS kernel daemon... [ OK ]
* Exporting directories for NFS kernel daemon... [ OK ]
* Starting NFS kernel daemon [ OK ]
```

### 4. 挂载测试

```
mount -t nfs -o nolock localhost:/home/casey/mine/nfs/X210_rootfs /mnt
执行后，进入/mnt 目录中，如果可以看到/home/sijifan/mine/nfs/X210_rootfs 路径中的内容，则说明 nfs 搭建成功！
umount /mnt即可取消挂载
```

### 5. 补充

```shell
mount -t nfs 192.168.4.211:/data/nfs_share /root/remote_dir        //挂载远程共享目录，其中，-t选项用于指定文件系统的类型为nfs。
umount /root/remote_dir                                            //共享目录使用结束之后，卸载共享目录
```

### 6. Ubuntu静态IP设置

在设备端(或者开发板)通过NFS挂载开发机上的文件系统时，需要在设备端的bootcmd中设置NFS服务器的IP地址，所以将开发机的IP设置为静态IP比较方便。

6.1 打开/etc/network/interfaces文件，在文件中输入以下内容：

```shell
auto lo
iface lo inet loopback
 
auto eth0
iface eth0 inet static
address 192.168.1.100
netmask 255.255.255.0
gateway 192.168.1.1
```

6.2 这里我将静态地址设置为 192.168.1.100。要注意的是在桥接方式下，虚拟机相当于是局域网中一台独立的主机，因此这个静态地址不能和局域网中别的主机冲突。

6.3 如果对网络设置做了更改后无法连接网络，可以尝试使用重启网络。

```shell
sudo ifdown eth0
sudo ifup eth0
```





