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

# Q:"错误CS0006:未能找到元数据文件"

**解决：**

右键解决方案->点击批生成->勾选缺少的元数据文件，重新生成

https://blog.csdn.net/weixin_44144122/article/details/103174114



# Q:使用Nuget报错："程序包还原失败，正在回滚..."

**解决：**

![image-20250120101051848](C:\Users\28968\AppData\Roaming\Typora\typora-user-images\image-20250120101051848.png)

在Nuget的程序包源中去掉不需要的(原错误是删除了本地的Devpress文件，但是Nuget中还有其程序包源)
