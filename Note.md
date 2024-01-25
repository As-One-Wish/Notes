# 一、Visual Studio 2022的工具箱控件灰色不可用

1.打开工具-命令行-开发者命令提示

2.输入`devenv /resetsettings`

3.输入`devenv /setup`

4.重启项目

> 会导致字体颜色的相关设置重置

# 二、DevExpress Tips

使用`DevExpress.XtraVerticalGrid.PropertyGridControl`的`SelectedObject`获取对象的值，如果有修改，需要光标移出才可以获取最新值

```c#
public T GetSelectedObject()
{
    editProGrid.CloseEditor(); // 或者可以关闭编辑
    return (T)editProGrid.SelectedObject;
}
```

# 三、Git上传代码失败

推荐VPN，然后查看自己打开VPN后的代理

<img src=".\Images\note_img_1.png">

然后设置Git Config

```shell
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890
```

