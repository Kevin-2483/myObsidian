### docker相关指令

#### 关于容器和镜像
docker 使用镜像构建容器,构建后的容器与源镜像分离,对容器的任何更改不会保存

要提交 Docker 镜像，你需要使用 `docker commit` 命令。這個命令允許你將修改過的容器保存為一個新的鏡像。以下是一個基本的用法：

```
docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
```

其中：

- `CONTAINER` 是你要提交的容器的名稱或 ID。
- `REPOSITORY` 是你想要給這個新鏡像的名稱。
- `TAG` 是這個鏡像的標籤。如果不指定，則默認為 `latest`。

例如，如果要提交容器名為 `mycontainer` 到一個名為 `myimage` 的鏡像，可以執行：

```
docker commit mycontainer myimage
```

如果要為這個鏡像指定標籤，可以這樣做：

```
docker commit mycontainer myimage:tag1
```

請注意，`docker commit` 命令將會保存容器的當前狀態為一個新的鏡像，但這種方式並不是最佳的鏡像管理方法。建議使用 Dockerfile 來定義你的鏡像，然後使用 `docker build` 命令來創建鏡像。

要重新启动 Docker 容器，你可以使用 `docker restart` 命令，例如：

```
docker restart <container_name_or_id>
```

你需要将 `<container_name_or_id>` 替换为你要重新启动的容器的名称或ID。

#### 列表
- 列出网络设备
```
docker network ls
```
- 列出镜像
```
docker images    
```
- 列出所有容器,不带-a仅列出运行
```
docker ps -a
```
#### 删除

要删除已经拉取的 Docker 镜像，你可以使用 `docker rmi` 命令。

```
docker rmi my_image
```

如果你想要删除多个镜像，你可以在命令中列出它们的名称，用空格隔开：

```
docker rmi image1 image2 image3
```

另外，如果你想要删除所有未被使用的镜像，你可以运行以下命令：

```
docker image prune
```

这会删除你系统中未被任何容器使用的所有镜像。

關閉 Docker 容器可以使用 `docker stop` 命令，例如：

```
docker stop [容器名稱或ID]
```

這將會停止指定名稱或 ID 的容器。若要完全移除容器，可以使用 `docker rm` 命令，例如：

```
docker rm [容器名稱或ID]
```

這將會刪除指定名稱或 ID 的容器。若要停止並刪除所有的容器，可以執行：

```
docker stop $(docker ps -a -q)
docker rm $(docker ps -a -q)
```

請注意，這將會停止並刪除所有容器。

记得在执行这些命令之前，确认你真的想要删除这些镜像，因为删除后将无法恢复喵~

#### 关于网络

Docker 中有几种不同类型的网络，每种类型都适用于不同的使用场景和需求。以下是一些常见的网络类型：

1. **bridge（桥接网络）**：桥接网络是 Docker 默认创建的网络类型。当你启动一个容器时，它会连接到这个网络，容器之间可以相互通信。桥接网络通常用于在单个 Docker 主机上运行多个容器，并且这些容器需要相互通信。

2. **host（主机网络）**：主机网络模式将容器直接连接到主机网络，容器将共享主机的网络命名空间。在主机网络中，容器可以访问主机上的所有网络服务，而不需要进行端口映射。这种模式通常用于需要最大网络性能和最小网络延迟的场景。

3. **overlay（覆盖网络）**：覆盖网络允许将多个 Docker 守护程序连接起来，创建一个跨多个主机的虚拟网络。这种网络类型通常用于容器在集群环境中进行通信。

4. **macvlan（MACVLAN 网络）**：MACVLAN 网络允许容器拥有自己的 MAC 地址，使得它们可以直接连接到物理网络，就像物理设备一样。这种网络类型通常用于需要容器与外部网络直接通信的场景。

5. **none（无网络）**：在这种模式下，容器不会连接到任何网络。这意味着容器内部无法访问外部网络，也无法被外部网络访问。通常情况下，这种网络类型用于特定的安全需求，或者在容器内部手动配置网络。

每种网络类型都有自己的优势和适用场景，你可以根据具体的需求选择合适的网络类型。

创建网络时，你可以通过指定 `-d` 参数来选择网络的类型。常见的网络类型包括：

1. **bridge（桥接网络）**：用于在单个 Docker 主机上运行多个容器，并且这些容器需要相互通信。这是默认的网络类型。

2. **host（主机网络）**：将容器直接连接到主机网络，容器将共享主机的网络命名空间。

3. **overlay（覆盖网络）**：允许将多个 Docker 守护程序连接起来，创建一个跨多个主机的虚拟网络。

4. **macvlan（MACVLAN 网络）**：允许容器拥有自己的 MAC 地址，使得它们可以直接连接到物理网络，就像物理设备一样。

5. **none（无网络）**：容器不会连接到任何网络。

你可以使用 `-d` 参数并指定对应的网络类型来创建网络。例如，要创建一个桥接网络，可以执行以下命令：

```
docker network create -d bridge mynetwork
```

要创建其他类型的网络，只需将 `-d` 参数后面的值替换为对应的网络类型即可。

要将容器连接到网络，你可以使用 `docker network connect` 命令。以下是基本的用法：

```
docker network connect [OPTIONS] NETWORK CONTAINER
```

其中：

- `NETWORK` 是要连接的网络的名称或ID。
- `CONTAINER` 是要连接到网络的容器的名称或ID。

例如，要将名为 `mynetwork` 的网络连接到名为 `mycontainer` 的容器，可以执行以下命令：

```
docker network connect mynetwork mycontainer
```

这将会将 `mycontainer` 容器连接到 `mynetwork` 网络，使得它可以与网络中的其他容器进行通信。

需要注意的是，一个容器可以连接到多个网络。如果你想要断开容器与某个网络的连接，可以使用 `docker network disconnect` 命令。

```
docker network disconnect <network_name> <container_name_or_id>
```

你需要将 `<network_name>` 替换为你要断开的网络的名称，`<container_name_or_id>` 替换为你要从该网络中断开连接的容器的名称或ID。
