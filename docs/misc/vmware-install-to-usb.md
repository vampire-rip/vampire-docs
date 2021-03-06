### 在 VMWare 上将系统安装到 U 盘

1. 创建虚拟机的时候 一定要选 64 位系统，不是 64 位的系统无法将引导模式设置成 UEFI，即使改配置文件成 UEFI 后也无法引导 UEFI

2. 创建完要将 USB 控制器改为 USB3.0，引导模式改成 UEFI，VMware Workstation Player 也是可以改的不过要修改 .vmx 配置文件，在里面加入 `
firmware = "efi"`

3. 安装之前把分区表换成 GPT

4. 分区的时候分一个约 500M 的 EFI 分区 挂载 /boot/efi，如果没有提示挂载 /boot/efi 的话大概率是 BIOS 引导的

#### 踩了的坑

+ 因为没有第二个 U 盘也不想用硬盘怕把 EFI 改爆就想了这种方法

+ 开始怎么都改不了 UEFI 就尝试用 BIOS 安装 + 手动还原 EFI... 想法很美好但是还原 EFI 也太麻烦了。。所以还是尝试用 UEFI 直接装吧。

+ 研究了很久，换成 VMWare Workstation 才发现创建完虚拟机，再改成 64 位是没用的，还是不能用 UEFI 引导（VMP 没有这个选单，直接改配置文件没有错误提示但是就是不能引导= =，看来对于技术不过关的家伙氪金还是必要的）。

+ UEFI 会查找 GPT 分区表中 GUID 是 C12A7328-F81F-11D2-BA4B-00A0C93EC93B 的 ESP 分区，并引导里面的 /BOOT/EFI/boot{arch}.efi，并不是随便整一个 FAT 分区里有这个 efi 都会引导的。MBR 分区表对应的分区标记应该是 EF 00，也和 FAT32 的不一样。

+ 那些看起来随便整的出现在引导选单上的 EFI 其实是写在 NVROM 里面的，并不存在于磁盘上。（所以如果设置了启动顺序移除 USB 什么的也不会丢掉）。

#### 重新启动 VMWare Tools

```bash
sudo /etc/init.d/vmware-tools restart
```

#### 自动挂载 U 盘

打开 `vmware.log` 查找 `USB: Found device`，找到需要自动挂载的设备，VMWare 可以通过 name、vid、pid 和 path 来定位设备

修改 .vmx 文件，在其中添加对应的 `usb.autoConnect.device{seq} = "{key}:{value} {key2}:{value2}"` 即可，如：

```
usb.autoConnect.device0 = "name:SanDisk\ USB\ Extreme\ Pro vid:0781 pid:8888"
```
