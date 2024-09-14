# 生成密钥
```
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```
# 添加密钥
## 确保SSH 代理正在运行
启动ssh代理
```
eval `ssh-agent`
```
添加环境变量`SSH_AUTH_SOCK`
```
export SSH_AUTH_SOCK=/path/to/your/ssh-agent-socket
```
将私钥添加到 SSH 代理中。可以使用以下命令：
```
ssh-add /path/to/your/private/key
```
最后，使用 SSH 连接到目标主机。命令如下所示：
```
ssh username@hostname -i /path/to/your/private/key
```

确保私钥文件的权限设置为安全的权限。一般来说，私钥文件的权限应该为 600。你可以使用以下命令设置私钥文件的权限：
```
chmod 600 /path/to/your/private/key
```
