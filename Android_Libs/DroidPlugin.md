##在Host中集成项目的方法：

1. 首先将DroidPlugin当作一个lib工程应用到主项目中，也可以将DroidPlugin打包成.aar，然后给相应的工程接入。

2. DroidPlugin需要做初始化，采取的接口是在Application的onCreate和attachBaseContext方法中添加代码：

```	@Override
	public void onCreate() {
		super.onCreate;
		PluginHelper.getInstance().applicationOnCreate(getBaseContext());
	}

	@Override
	protected void attachBaseContext(Context base) {
		PluginHelper.getInstance().applicationAttachBaseContext(base);
	}
```

3. 将插件中`AndroidManifest.xml`中所有的`provider`对应的`authorities`修改成自己的。目的是防止和其他使用该插件的app冲突。

##安装、卸载插件的方法：

1. 安装、更新插件使用如下的方法：

`	
int PluginManager.getInstance().installPackage(String filePath, int flags);
`	

flags设为0即可。

2. 卸载插件的方法：

`
int PluginManager.getInstance().deletePackage(String packageName, int flags);
`

packageName传插件包名即可， flags设为0。

3. 启动插件的方法：

跟通常启动插件的方法一样，只是由于不能直接引用用Activity，而是要使用对应的包名，传一个字符串给intent。

## 启动过程解析

初始化过程最终会调用到`HookFactory::installHook(Context context, ClassLoader classLoader)`,在这里hook掉的系统服务包括：

*Clipboard 剪贴板
*SearchManager 调用系统的搜索服务，这个服务的hook是我加上去的
*NotificationManager 发通知的一个service
*MountService 挂载管理，没弄清楚是干啥
*AudioService 音频服务
*ContentService 数据同步服务，似乎对也没有什么用
*WindowManager 这个大家就很熟悉了
*GrapphicsStats 不知道是干啥的服务
*MediaRouterService 这个服务可以远程操纵媒体，具体请参考[MediaRouterService](http://developer.android.com/reference/android/media/MediaRouter.html)
*SessionManager 看起来也跟媒体有关
*WifiManager 这个明显跟wifi有关, 具体参考 [WifiManager](http://developer.android.com/reference/android/net/wifi/WifiManager.html)
*InputMethodManager 输入法

以下这几个需要特别关注：

*PackageManager 包管理服务
*ActivityManager 活动管理，这个比较核心
*PluginCallback 这个跟组件生命周期的调度有关
*Instrumentation 这个也跟组件的生命周期有关
*LibCore 暂时不明觉厉

*SQLiteDatabaseHook 这个跟数据库有关
