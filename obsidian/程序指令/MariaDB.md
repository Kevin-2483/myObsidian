如果你没有为 root 用户设置密码，通常你可以使用系统管理员权限登录 MariaDB。

在大多数系统上，你可以使用以下命令以系统管理员权限登录 MariaDB：

```bash
sudo mariadb
```

这将使用 `sudo` 命令以管理员权限启动 MariaDB 客户端。在一些系统上，可能需要使用 `su` 命令来切换到 root 用户：

```bash
su -
mariadb
```

这会将你切换到 root 用户，然后启动 MariaDB 客户端。

在成功登录 MariaDB 后，你可以执行 `GRANT` 命令来为其他用户授予权限。例如，为 `kevin` 用户授予在 `cargo` 数据库上的所有权限：

```sql
GRANT ALL PRIVILEGES ON cargo.* TO 'kevin'@'localhost';
```

这样做后，`kevin` 用户就应该能够在 `cargo` 数据库上执行操作了。如果还有其他问题，请随时提问。

要显示用户的权限，你可以使用 `SHOW GRANTS` 命令。这个命令将显示给定用户（或当前用户）的授权信息。以下是基本的语法：

```sql
SHOW GRANTS FOR 'username'@'host';
```

- `username`：要查看权限的用户名。
- `host`：用户的主机名。

例如，如果你想查看用户 `kevin` 在 `localhost` 上的权限，可以执行以下命令：

```sql
SHOW GRANTS FOR 'kevin'@'localhost';
```

这将显示用户 `kevin` 在 `localhost` 上的授权信息，包括允许的数据库和对应的权限。

如果你想查看当前登录用户的权限，可以简单地执行 `SHOW GRANTS` 命令，而无需指定用户名和主机名：

```sql
SHOW GRANTS;
```

这将显示当前登录用户（通常是你登录的用户）的授权信息。
