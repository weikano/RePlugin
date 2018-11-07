### RePluginApplication流程详解

attachBaseContext

> 调用RePlugin.App.attachBaseContext 
>
> - 其中RePluginInternal.init(app)只是将application保存至RePluginInternal中的sAppContext静态变量。
> - RePluginConfig.initDefaults()用来初始化pnInstallDir和callback
> - IPC.init用来获取当前进程名称(通过/proc/self/cmdline)，进程id以及宿主包名，然后还要获取常驻进程名，并将是否在主进程(app包名)以及是否在常驻进程保存在sIsUIProcess和sIsPersistentProcess两个静态变量中
> - HostConfigHelper.init()主要是触发HostConfigHelper的静态代码块，用来初始化HostConfigHelper中的值（从com.qihoo360.replugin.ren.RePluginHostConfig中）
> - PluginStatusController.setAppContext()将context保存至sAppContext中
> - PMF.init：将application保存至sContext，一堆初始化AIDL类的方法。然后调用PatchClassLoaderUtils.patch(application)方法，hook掉PackageInfo中的mClassLoader，用RePluginClassLoader代替并且将当前线程的ContextClassLoader替换为RePluginClassLoader
> - PMF.callAttach()方法最终会调用PmBase.callAttach，中间会调用Plugin.load方法将插件加载进内存并调用插件Application的attachBaseContext和onCreate方法(callApp())
>

### RePluginClassLoader

> loadClass 首先通过PMF.loadClass方法查找className的class。PMF通过PmBase.loadClass方法查找对应的class：先从mContainerXXX(Androidmanifest.xml中的坑位)查找，然后从DynamicClass中查找动态注册的

### PluginDexClassLoader

> 插件的classloader
>
> loadClass中先调用super.loadClass，找到就返回对应的class。如果抛出ClassNotFoundException并且是okhttp或者apache httpclient相关的类，那么直接loadClassFromHost，从宿主的classloader(RePluginClassLoader)中获取对应的class；如果super.loadClass返回null，那么根据RePluginConfig中的useHostClassIfNotFound属性来判断是否需要从宿主的classloader查找类。
>
> 何时hook掉插件的classloader？
>
> Loader.loadDex()<-Plugin.doLoad()<-Plugin.loadLocked()<-Plugin.load
>
> Plugin.load有两个分支：
>
> - PmBase.callAttach，即在RePluginApplication.attachBaseContext中会触发
> - PmBase.loadPackageInfoPlugin/PmBase.loadResourcePlugin/PmBase.loadDexPlugin/PmBase.loadAppPlugin/PmBase.loadPlugin，一直回溯到PluginFastInstallProvider.install<-RePlugin.preload()

### PluginContext

> 插件的Context
>
> - 在Loader.loadDex时初始化，会传入插件对应的ClassLoader以及根据apk文件生成的Resource，默认的主题android.R.style.Theme。
> - Plugin.callAppLocked时会hook掉原有的Application中的mBase，通过attachBaseContext方法。
> - replugin-plugin-library中，PluginActivity(其他基类插件Activity也同样)，会通过RePluginInternal.createActivityContext方法，反射调用Factory2.createActivityContext生成PluginContext，然后调用super.attachBaseContext覆盖原有的mBase，达到替换资源等目的。
>
>