# libvirt
## Setup
```/etc/nixos/configuration.nix
virtualisation.libvirtd = {
  enable = true;
  qemu = {
    package = pkgs.qemu_kvm;
    runAsRoot = true;
    swtpm.enable = true;
    ovmf = {
      enable = true;
      packages = [(pkgs.unstable.OVMF.override {
        secureBoot = true;
        tpmSupport = true;
      }).fd];
    };
  };
};
```

```/etc/nixos/configuration.nix
users.users.myuser = {
  extraGroups = [ "libvirtd" ];
};
```
## Configuration
### UEFI with OVMF
```~/.config/libvirt/qemu.conf
# Adapted from /var/lib/libvirt/qemu.conf
# Note that AAVMF and OVMF are for Aarch64 and x86 respectively
nvram = [ "/run/libvirt/nix-ovmf/AAVMF_CODE.fd:/run/libvirt/nix-ovmf/AAVMF_VARS.fd", "/run/libvirt/nix-ovmf/OVMF_CODE.fd:/run/libvirt/nix-ovmf/OVMF_VARS.fd" ]
```

# Client

## Virt-manager

显示协议spice
```xml
<graphics type='spice' port='5900' autoport='no' listen='0.0.0.0'/>
```

## remote-viewer

连接 
```url
spice://27.25.156.200:5900
```
