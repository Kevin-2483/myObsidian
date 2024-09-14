# 网络相关
你可以使用以下命令来列出网络接口（包括 Internet 接口和 Wi-Fi 接口）：
```bash
ip addr show
ip link show
```

`ip addr show` 和 `ip link show` 是 Linux 系统中用于管理网络接口的两个命令，它们的功能略有不同：

- `ip addr show`：这个命令显示每个网络接口的详细信息，包括接口的 IP 地址、子网掩码、MAC 地址等。它可以让你查看网络接口的配置信息以及当前状态。

- `ip link show`：这个命令显示系统中所有网络接口的基本信息，包括接口名称、MAC 地址、状态等。它通常用于列出网络接口的基本属性，但不包括 IP 地址等详细信息。

所以，如果你想要查看网络接口的 IP 地址等详细信息，可以使用 `ip addr show` 命令；如果你只是想要查看接口的基本属性，可以使用 `ip link show` 命令。喵~

# 文件操作

## 符号链接
要为文件夹创建符号链接，你可以使用类似于以下命令的语法：

```bash
ln -s 源文件夹路径 目标链接路径
```

例如，如果你想要为名为`/path/to/source_folder`的文件夹创建一个符号链接到`/path/to/target_link`，你可以运行以下命令：

```bash
ln -s /path/to/source_folder /path/to/target_link
```

记得替换实际的路径喵~

# 设备操作
您可以尝试使用以下命令来提取您的设备名称：

```bash
pactl list sources
```

这个命令会从输出中提取包含 "名稱" 和 "monitor" 的行，并使用 awk 命令将行拆分为两部分，然后打印第二部分，即设备名称。

# 服务相关
## systemd服务
要查看 Linux 上所有正在运行的服务，你可以使用以下命令：

```bash
service --status-all
```

或者，如果你使用的是 Systemd，你可以运行以下命令来列出所有正在运行的服务：

```bash
systemctl list-units --type=service --state=running
```

运行这些命令后，你会看到列出的所有正在运行的服务列表。喵~

## screen指令






# 用户和组

要查看Linux系统中的用户组，您可以使用以下命令：

```
cat /etc/group
```

这将显示系统上所有用户组的列表及其成员。喵~

要查看您自己所属的组，您可以使用以下命令：

```
groups
```

这将显示您当前登录用户所属的组列表。喵~