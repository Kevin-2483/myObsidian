# Windows

```sh
curl -O https://www.sqlite.org/2024/sqlite-dll-win-x86-3460000.zip

unzip *.zip
```

使用Developer PowerShell for VS 2022

```sh
lib /def:sqlite3.def /out:sqlite3.lib /machine:x64
```

设置PATH

# Ubuntu

```sh
sudo apt install libsqlite3-dev
```
