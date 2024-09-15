`setfont ter-v32n` 增大字体大小
# 网络连接

## Wifi 连接

NixOS 默认使用 `wpa_supplicant` 作为无线守护程序。

```bash
sudo systemctl start wpa_supplicant  # 启动服务
```

```bash
sudo wpa_cli  # 进入 wpa 命令行交互模式
```

家庭网络

```wpa_cli
> add_network
0
> set_network 0 ssid "你家 WIFI 的 SSID"
OK
> set_network 0 psk "WIFI 密码"
OK
> set_network 0 key_mgmt WPA-PSK  
OK
> enable_network 0
OK
```

```bash
ping www.baidu.com -c 4  # 此项不通优先先检查域名解析服务器
ping 119.29.29.29 -c 4  # 腾讯 DNSPod，不通请检查网络连接
```

## 更换频道(换源)

```bash
sudo -i #自动以nixos身份登录。nixos用户帐户的密码为空，在没有密码的情况下使用 sudo
nix-channel --add https://mirrors.ustc.edu.cn/nix-channels/nixpkgs-unstable nixpkgs  # 订阅镜像仓库频道
nix-channel --add https://mirrors.ustc.edu.cn/nix-channels/nixos-23.11 nixos  # 请注意系统版本
nix-channel --list  # 列出频道
nix-channel --update  # 更新并解包频道
nixos-rebuild --option substituters "https://mirror.sjtu.edu.cn/nix-channels/store" switch --upgrade  # 临时切换二进制缓存源，并更新生成
```

## 分区与格式化

使用`lsblk`来查看分区情况

可使用 `parted` 、 `fdisk` 、 `gdisk` 、 `cfdisk` 和 `cgdisk` 。
使用选项 `-L label` 为文件系统分配唯一的符号标签，因为这使得文件系统配置独立于设备更改。例如：

```bash
mkfs.ext4 -L nixos /dev/sda1
```

使用`-n label`来为启动分区打上标签

```bash
mkfs.fat -F 32 -n boot /dev/sda3
```

使用`swapon`来建立swap分区
## 挂载

```bash
mount /dev/disk/by-label/nixos /mnt
```

```bash
mkdir -p /mnt/boot
mount /dev/disk/by-label/boot /mnt/boot
```

```bash
swapon /dev/sda2
```

## 生成初始配置

```bash
nixos-generate-config --root /mnt
```

此时可以编辑
```bash
nano /mnt/etc/nixos/configuration.nix
```

`uefi`系统建议使用`systemd-boot`来作为引导加载程序

```nix
boot.loader.systemd-boot.enable = true;
```

启用网络

```nix
networking.networkmanager.enable = true;
users.users.alice.extraGroups = [ "networkmanager" ];
```

启用ssh

```nix
services.openssh.enable = true;
services.openssh.settings.PermitRootLogin = "no" #完全禁用root ssh登录
```

启用 authorised RSA/DSA public keys 

```nix
users.users.alice.openssh.authorizedKeys.keys = [ "ssh-dss AAAAB3NzaC1kc3MAAACBAPIkGWVEt4..." ];
```

配置主机名

```nix
networking.hostName = "NixOS";
```

无线网络

```nix
networking.wireless.enable = true;
```

自定义内核

```
boot.kernelPackages = pkgs.linuxKernel.packages.linux_lts;
```

内核模块

```nix
boot.kernelModules = [ "fuse" "kvm-intel" "coretemp" ];
```

早启动模块

```nix
boot.initrd.kernelModules = [ "cifs" ];
```

VirtalBox

1. Add a New Machine in VirtualBox with OS Type “Linux / Other Linux”  
    在 VirtualBox 中添加操作系统类型为“Linux / Other Linux”的新计算机
    
2. Base Memory Size: 768 MB or higher.  
    基本内存大小：768 MB 或更高。
    
3. New Hard Disk of 8 GB or higher.  
    8 GB 或更大的新硬盘。
    
4. Mount the CD-ROM with the NixOS ISO (by clicking on CD/DVD-ROM)  
    使用 NixOS ISO 挂载 CD-ROM（通过单击 CD/DVD-ROM）
    
5. Click on Settings / System / Processor and enable PAE/NX  
    单击“设置”/“系统”/“处理器”并启用 PAE/NX
    
6. Click on Settings / System / Acceleration and enable “VT-x/AMD-V” acceleration  
    点击设置/系统/加速并启用“VT-x/AMD-V”加速
    
7. Click on Settings / Display / Screen and select VMSVGA as Graphics Controller  
    单击设置/显示/屏幕并选择 VMSVGA 作为图形控制器
    
8. Save the settings, start the virtual machine, and continue installation like normal  
    保存设置，启动虚拟机，然后像平常一样继续安装

```nix
boot.loader.grub.device = "/dev/sda";
```

```nix
boot.initrd.checkJournalingFS = false;
```

可以在 VirtualBox 设置中的主机系统中为共享文件夹指定名称和路径（计算机/设置/共享文件夹，然后单击“添加”图标）。将以下内容添加到 `/etc/nixos/configuration.nix` 以自动安装它们。如果不添加 `"nofail"` ，系统将无法正常启动。

```nix
{ config, pkgs, ...} :
{
  fileSystems."/virtualboxshare" = {
    fsType = "vboxsf";
    device = "nameofthesharedfolder";
    options = [ "rw" "nofail" ];
  };
}
```
该文件夹将直接位于根目录下。

## 安装

```bash
nixos-install
```

使用`--option substituters "https://mirror.sjtu.edu.cn/nix-channels/store"`来自定义缓存仓库

无人值守

```
nixos-install --no-root-passwd
```

重启之后

```
useradd -c 'Eelco Dolstra' -m **** $ passwd ****
```