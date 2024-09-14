```bash
sudo efibootmgr -c -w -L “refind” -d /dev/nvme0n1 -p 7 -l \\EFI\\refind\\refind_x64 .efi
```

这个命令的作用是创建一个名为 `"BootOptionName"` 的引导选项，并将其设置为默认选项，该选项引导的是位于 `/dev/sda` 磁盘的第一个分区上的 EFI 可执行文件 `refind_x64.efi`。