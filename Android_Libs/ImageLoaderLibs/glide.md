mptech/Glide
https://github.com/bumptech/glide.git

根据glide的自述文档，作为一款图片加载Library,功能主要包括媒体解码，内存和磁盘缓存，以及维护一个资源池。
glide支持获取，解码以及展示video stills(开始并不知道指的是什么,后来根据文档，发现应该指的是一段视频，可以用视频中截取相应的图片，然后展现成gif的效果)，图片，和gif。
同时支持不同的网络模块，默认使用HttpUrlConnection,接入OkHttp和Volley也很容易。
glide自述想达到的目的则是使得任何图片的滑动都尽量地顺畅和快速。当然做其他的工作也很合适。

接入的话（我实测过），按照他们家文档说的就行，当然还是用android studio比较容易。

另外也需要编译sdk的版本达到19以上，原因是需要用到Bitmap.getAllocationByteCount()方法。

使用的接口非常简单:
	 Glide.with(myFragment)
		.load(url)
		.centerCrop()
		.placeholder(R.drawable.default_pic)
		.crossFade()
		.into(ImageView)

可以使用 @implementation GlideModule 来简单设置@library Glide.

使用方法：
	$1 @class MyGlideModule 实现 @implementation GlideModule
	$2 设置混淆：-keepnames class com.mypackage.MyGlideModule
	$3 把设置加入AndroidManifest.xml
		<meta-data
			android:name="com.mypackage.MyGlideModule"
			android:value="GlideModule" />


修改磁盘缓存的位置和大小：
	$1大小：
		builder.setDiskCache(new InternalCacheDiskCacheFactory(context, yourSizeInBytes));
	$2位置：	
		builder.setDiskCache(new InternalCacheDiskCacheFactory(context, cacheDirectoryName, yourSizeInBytes));
	$3sdcard位置：
		builder.setDiskCache(new ExternalCacheDiskCacheFactory(context, cacheDirectoryName, yourSizeInBytes));
	$4更加复杂的决定缓存路径过程
		builder.setDiskCache(new DiskLruCacheFactory(new CacheDirectoryGetter() {
			@Override
			public File getCacheDirectory() {
				return getMyCacheLocationBlockingIO();
			}
		}), yourSizeInBytes);

内存缓存的配置：
	$1 可以配置MemorySizeCalculator,然后使用builder.setMemorySizeCalculator的方法来配置默认的Bitmap池和默认的内存缓存的大小。
	$2 配置内存缓存的大小，使用：
	builder.setMemoryCache(new LruResourceCache(yourSizeInBytes));
	$3 配置bitmap缓存池的大小：
	builder.setBitmapPool(new LruBitmapPool(sizeInBytes));
	
图片格式的配置：
	默认是RGB_565,Android系统默认的是ARGB_8888,如果想设置成Android默认的ARGB_8888可以配置：
	builder.setDecodeFormat(DecodeFormat.ALWAYS_ARGB_8888);


如果不是想加载图片后立刻展现可以使用SimpleTarget,具体使用方法如下：
	Glide.with(yourApplicationContext)
		.load(yourUrl)
		.asBitmap()
		.into(new SimpleTarget<Bitmap>(myWidth, myHeight) {
			@Override
			public void onResourceReady(Bitmap bitmap, GlideAnimation anim) {
				//doSomeThing
			}
		});

	针对这种情形，建议传applicationContext,因为并不想在activity或者fragment暂停的时候暂停对图片的下载。另外如果这是一个很长的过程的话，建议使用静态内部类，不要用匿名内部类，可能会导致内存泄露。


接入volley:
跟上面接入GlideModule的方法类似。
	$1 下载volley的jar包放入library中
	$2 然后在GlideModule的实现中的registerComponents方法中加上：
		registry.replace(GlideUrl.class, InputStream.class, new VolleyUrlLoader.Factory(context));

这里的VolleyUrlLoader的代码可以从glide的release界面下载到，下载后直接copy进入工程。但是在我们自己的工程里面如果已经有过RequestQueue了，那么就可以直接改代码拿到RequestQueue的单例，这样可以防止多个RequestQueue实例造成崩溃。

volley适用于小数据量且频繁的网络请求，支持多线程，也支持优先级排序。但是对于大数据量的请求

特殊的Api:
	$1 downloadOnly(int, int),会下载图片并且存储在本地。使用：
	Glide.with(yourFragment)
		.load(yourUrl)
		.diskCacheStrategy(DiskCacheStrategy.ALL/DiskCacheStrategy.SOURCE)
		.into(yourView)
	能够保证使用已经在downloadOnly()中下载的图片。

	$2 into(int, int) 这里是会去加载图片，然后调用get()方法得到相应的Bitmap
	Bitmap myBitmap = Glide.with(applicationContext)
		.load(yourUrl)
		.asBitmap()
		.centerCrop()
		.into(xpixellength, ypixellength)
		.get()


GC_CONCURRENT 和 GC_ALLOC:
	GC是garbage collection，垃圾回收，的简写。
	GC_CONCURRENT每次回收只会造成大概5ms的卡顿。
	然而GC_ALLOC每次回收会造成125ms的卡顿，所以会造成掉帧，出现明显的卡顿。


Glide会在listview复用view时自动地调用clear(),然后释放相应的资源调用。
Glide同样也采用了引用计数来判断bitmap能不能被回收。


fitCenter 和 centerCrop
fitCenter指一边和布局长度一样，另一边比布局长度小
centerCrop指一边和布局长度一样，另一边比布局长度大

自定义transformation的方法：
https://github.com/bumptech/glide/wiki/Transformations


