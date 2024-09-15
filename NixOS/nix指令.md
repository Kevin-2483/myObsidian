# 系统和代
要清理NixOS系统上的构建，你可以运行以下命令：

```bash
sudo nix-collect-garbage -d
```

这会删除系统中不再需要的构建和旧版本的软件包。执行此命令后，系统将会清理不必要的构建文件。喵~

# 更新系统
在使用了 Nix Flakes 后，要更新系统也很简单，先更新 flake.lock 文件，然后部署即可。在配置文件夹中执行如下命令：

```bash
# 更新 flake.lock（更新所有依赖项）
nix flake update
```

```bash
# 或者也可以只更新特定的依赖项，比如只更新 home-manager:
nix flake lock --update-input home-manager
```

```bash
# 部署系统
sudo nixos-rebuild switch --flake .
```

```bash
# 部署系统
sudo nixos-rebuild switch
```

另外有时候安装新的包，跑 `sudo nixos-rebuild switch` 时可能会遇到 sha256 不匹配的报错，也可以尝试通过 `nix flake update` 更新 flake.lock 来解决。

# 包管理
手动添加包
```bash
nix-store add <>
```
临时使用包
```bash
nix-shell -p <>
```
使用非声明方法安装(慎用)
```bash
nix-env -iA <>
```

# 一些例子
### List available kernels

You can list available kernels using `nix repl` (previously `nix-repl`) by typing the package name and using the tab completion:
```bash
nix repl
```
添加变量
```bash
:l <nixpkgs>
```
输入`pkgs.linuxPackages`然后按下`Tab`来补全

# Custom kernel modules 自定义内核模块

请注意，如果您偏离默认内核版本，您还应该特别注意额外的内核模块必须与相同版本匹配。最安全的方法是使用 `config.boot.kernelPackages` 选择正确的模块集：
```nix
{ config, ... }:
{
  boot.extraModulePackages = with config.boot.kernelPackages; [ wireguard ];
}
```
