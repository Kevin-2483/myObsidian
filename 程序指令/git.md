要连接到远程 Git 仓库，首先需要将本地仓库与远程仓库关联起来，然后推送（push）或拉取（pull）代码。下面是一些基本步骤：

1. 添加远程仓库 URL：
   ```
   git remote add origin 远程仓库的 URL 喵~
   ```

2. 拉取远程仓库的代码到本地：
   ```
   git pull origin 主分支名称 喵~
   ```

3. 推送本地代码到远程仓库：
   ```
   git push origin 主分支名称 喵~
   ```

在上述命令中：
- "origin" 是远程仓库的默认名称，你也可以使用其他名称。
- "主分支名称" 是远程仓库的主分支名称，通常是 "master" 或 "main"。

如果是首次推送，可能需要先执行初始化并添加文件：
```
git init
```

```
git add .
```

```
git commit -m "初始提交"
```

这样你就能够连接到远程库并进行代码的拉取和推送了喵~

要切换分支，可以使用以下命令：

```
git checkout 分支名称 喵~
```

例如，如果要切换到名为 "feature" 的分支，可以这样做：

```
git checkout feature 喵~
```

如果要创建一个新的分支并立即切换到该分支，可以使用以下命令：

```
git checkout -b 新分支名称 喵~
```

例如，要创建并切换到名为 "bugfix" 的新分支，可以这样做：

```
git checkout -b bugfix 喵~
```

这样你就可以在不同的分支上进行工作啦喵~

**配置本地 .ssh/config以使用指定的 SSH 密钥**：
```
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/your_specific_ssh_key
    ProxyCommand connect -S 127.0.0.1:7890 -a none %h %p #使用代理
    PreferredAuthentications publickey #使用公钥验证

```
将 `your_specific_ssh_key` 替换为你要使用的特定 SSH 密钥文件的路径。

**测试连接**： 尝试使用 SSH 连接到远程 Git 服务器，例如 GitHub。使用以下命令：

cssCopy code

`ssh -T git@github.com`

如果配置生效，你应该会看到一条消息，确认你连接到了 GitHub，类似于：
```
Hi username! You've successfully authenticated, but GitHub does not provide shell access.
```

要回滚到之前的提交，你可以使用git命令中的`git reset`。下面是一些可能有用的选项：

1. 如果你只想回滚到上一个提交，可以使用：
```
git reset --hard HEAD^
```

2. 如果你知道要回滚到哪个具体的提交，可以使用提交的哈希值：
```
git reset --hard <commit_hash>
```

3. 如果你只是想回滚到上一个提交，但是保留修改文件的更改，可以使用：
```
git reset --soft HEAD^
```

记得在执行这些命令之前，先做好备份以免丢失数据。喵~

要显示远程仓库的信息，你可以使用以下命令：

```
git remote -v
```

这将列出所有远程仓库的名称和它们的URL。例如：

```
origin  https://github.com/username/repository.git (fetch)
origin  https://github.com/username/repository.git (push)
```

这里的`origin`是远程仓库的名称，`https://github.com/username/repository.git`是它的URL。喵~

要将远程仓库的URL从HTTPS更改为SSH，你可以使用以下命令：

```
git remote set-url origin git@github.com:username/repository.git
```

请确保将`username/repository.git`替换为你的存储库的正确地址。这个命令会将远程仓库的URL修改为SSH格式。

完成后，你可以使用`git remote -v`命令验证更改是否生效。喵~

![[Pasted image 20240409014928.png]]

```
git log
```

获取版本哈希