## RePlugin插件安装说明

> 基于RePlugin 2.1.5
>

RePlugin插件分为内置和外置两种。这两种插件都有自己的安装方式。

### 安装内置插件

像[wiki](https://github.com/Qihoo360/RePlugin/wiki/%E6%8F%92%E4%BB%B6%E7%9A%84%E7%AE%A1%E7%90%86)上描述的，将 [插件别名(包名)].apk 的后缀为jar，然后放到宿主的assets/plugins目下即可。

**不是随便的apk可以作为内置插件，必须由外置插件的方式开发生成的apk才可以，否则在运行内置插件时会出现异常。**

接着replugin-host-gradle插件会自动生成插件的主要信息，保存的plugins-builtin.json文件在宿主module的build\intermediates\assets\debug(release)\目录下。

> gradle-host-gradle生成内置插件信息的相关代码可以参考PluginBuiltinJsonCreator.groovy。

一切完成之后，在宿主application的attachBaseContext方法中调用RePlugin.attachBaseContext()会扫描本地插件，并根据插件的状态来更新宿主的插件信息（相关代码在Finder.search()，其中FinderBuiltin.loadPlugins()负责扫描内置插件）。至此，内置插件安装完成。

### 安装外置插件

外置插件的安装很简单。使用以下代码

```java
//apkPath为外置插件apk的完整路径，可以是在外置存储空间，也可以是内置的。
RePlugin.install(apkPath);
```

然而安装外置插件需要经过下列步骤，包括：

1. 确保你的apk文件存在并有读写权限（写权限非必须，RePluginConfig.setMoveFileWhenInstalling() ）。
2. 签名校验（非必须，RePluginConfig.setVerifySign()，开启后需要将apk的md5签名（去掉:），通过RePlugin.addCertSignature()方法添加进去）。
3. 读取apk的PackageInfo，并根据PackageInfo解析出PluginInfo。
4. 插件版本校验，现支持相同版本插件覆盖安装。
5. 根据RePluginConfig.isMoveFileWhenInstalling决定是否要将安装的apk移动至内置存储空间（PluginInfo.getApkFile()）。
6. 返回插件的Plugininfo信息。

上述步骤相关代码在RePluginManagerServer.installLocked()方法。



### 备注

**外置插件安装调试需要选择宿主的GuardService进行**