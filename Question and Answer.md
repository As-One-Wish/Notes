# Q:该文件没有与之关联的应用...

Win11资源管理器，在其他地方可以打开，固定到任务栏之后，打开就显示该文件没有与之关联的...

**解决：**

1.**使用命令行工具修复**

- 以管理员身份运行cmd。

- 输入以下命令并按回车：

```shell
sfc /scannow # 扫描并修复系统文件
```

- 扫描完成后，重启电脑。

2.**注册表修复**

- `win+R`输入`regedit`打开

- 导航到以下路径

```shell
HKEY_CLASSES_ROOT\Folder\shell\opennewwindow\command
```

- 修改默认值为

```shell
explorer.exe /e, %1
```

