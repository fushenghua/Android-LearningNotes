

# Android性能调优





本文主要分享自己在appstore项目中的性能调优点，包括**同步改异步、缓存、Layout优化、数据库优化、算法优化、延迟执行**等。

目前性能优化专题已完成以下部分：

[性能优化总纲——性能问题及性能调优方式](http://www.trinea.cn/android/performance/ "性能优化——性能问题及性能调优方式")

[性能优化第四篇——移动网络优化](http://www.trinea.cn/android/mobile-performance-optimization/ "移动端网络优化")

[性能优化第三篇——Java(Android)代码优化](http://www.trinea.cn/android/java-android-performance/ "性能优化——Java(Android)代码优化")
[性能优化第二篇——布局优化](http://www.trinea.cn/android/layout-performance/ "性能优化之布局优化")
[性能优化第一篇——数据库性能优化](http://www.trinea.cn/android/database-performance/ "性能优化之数据库优化")

[性能优化实例](http://www.trinea.cn/android/android-performance-demo/ "Android性能调优")

**一、性能瓶颈点**

整个页面主要由6个Page的ViewPager，每个Page为一个GridView，GridView一屏大概显示4*4的item信息(本文最后有附图)。由于网络数据获取较多且随时需要保持页面内app下载进度及状态，所以出现以下性能问题

a.  ViewPager左右滑动明显卡顿

b.  GridView上下滚动明显卡顿

c.  其他Activity返回ViewPager Activity较慢

d.  网络获取到展现速度较慢

**二、性能调试及定位**

主要使用[Traceview](http://www.trinea.cn/android/android%e6%80%a7%e8%83%bd%e8%b0%83%e4%bc%98%e5%b7%a5%e5%85%b7traceview%e4%bb%8b%e7%bb%8d/ "Android性能调优工具TraceView介绍")、monkey、monkey runner调试，traceview类似java web调优的visualvm，使用方法如下：

```
在需要调优的activity onCreate函数中添加

android.os.debug.startMethodTracing("Entertainment");
onDestrory函数中添加

android.os.debug.stopMethodTracing();
```

程序退出后会在sd卡根目录下生成Entertainment.trace这个文件，cmd到android sdk的tools目录下运行traceview.bat Entertainment.trace即可，截图如下

![android traceview](/img/performance1.jpg)从中可以看出各个函数的调用时间、调用次数、平均调用时间、时间占用百分比等从而定位到耗时的操作。monkey、monkey runner更详细的见后面博客介绍

**三、性能调优点**

主要包括**同步改异步、缓存、Layout优化、数据库优化、算法优化、延迟执行。**

**1\. 同步改异步**

这个就不用多讲了，耗时操作放在线程中执行防止占用主线程，一定程度上解决anr。

但需要注意线程和service结合（防止activity被回收后线程也被回收）以及线程的数量

线程池使用可见[java的线程池](http://www.trinea.cn/android/java-android%e7%ba%bf%e7%a8%8b%e6%b1%a0/ "Java(Android)线程池")

**2\. 缓存**

java的对象创建需要分配资源较耗费时间，加上创建的对象越多会造成越频繁的gc影响系统响应。主要使用**单例模式、缓存(图片缓存、线程池、View缓存、IO缓存、消息缓存、通知栏notification缓存)及其他方式**减少对象创建。

**(1). 单例模式**

对于创建开销较大的类可使用此方法，保证全局一个实例，在程序运行过程中该类不会因新建额外对象产生开销。示例代码如下：



单例模式

``` Java
public  class  Singleton  {

    private  static  Object    obj      =  new  Object();

    private  static  Singleton instance  =  null;

    private  Singleton(){

    }

    public  static  Singleton getInstance()  {

        // if already inited, no need to get lock everytime

        if  (instance  ==  null)  {

            synchronized  (obj)  {

                if  (instance  ==  null)  {

                    instance  =  new  Singleton();

                }

            }

        }

        return  instance;

    }

}
```


(2). 缓存

程序中用到了**图片缓存、线程池、View缓存、IO缓存、消息缓存、通知栏notification缓存**等。

**a. 图片缓存：**见[ImageCache](http://www.trinea.cn/android/android-imagecache%e5%9b%be%e7%89%87%e7%bc%93%e5%ad%98%ef%bc%8c%e4%bd%bf%e7%94%a8%e7%ae%80%e5%8d%95%ef%bc%8c%e6%94%af%e6%8c%81%e9%a2%84%e5%8f%96%ef%bc%8c%e6%94%af%e6%8c%81%e5%a4%9a%e7%a7%8d%e7%bc%93/ "Android ImageCache图片缓存，使用简单，支持预取，支持多种缓存算法，支持不同网络类型，扩展性强")和[ImageSdCache](http://www.trinea.cn/android/android-%e5%9b%be%e7%89%87sd%e5%8d%a1%e7%bc%93%e5%ad%98-%e4%bd%bf%e7%94%a8%e7%ae%80%e5%8d%95-%e6%94%af%e6%8c%81%e9%a2%84%e5%8f%96-%e6%94%af%e6%8c%81%e5%a4%9a%e7%a7%8d%e7%bc%93%e5%ad%98%e7%ae%97/ "Android 图片SD卡缓存 使用简单 支持预取 支持多种缓存算法 支持不同网络类型 支持序列化")

**b. 线程池：**使用[Java的Executors](http://www.trinea.cn/android/java-android%e7%ba%bf%e7%a8%8b%e6%b1%a0/ "Java(Android)线程池")类，通过newCachedThreadPool、newFixedThreadPool、newSingleThreadExecutor、newScheduledThreadPool提供四种不同类型的线程池

**c. View缓存：**

可见[ListView缓存机制](http://www.trinea.cn/android/android-listview%e6%bb%91%e5%8a%a8%e8%bf%87%e7%a8%8b%e4%b8%ad%e5%9b%be%e7%89%87%e6%98%be%e7%a4%ba%e9%87%8d%e5%a4%8d%e9%94%99%e4%b9%b1%e9%97%aa%e7%83%81%e9%97%ae%e9%a2%98%e8%a7%a3%e5%86%b3/ "Android ListView缓存")



listView的getView缓存



通过convertView是否为null减少layout inflate次数，通过静态的ViewHolder减少findViewById的次数，这两个函数尤其是inflate是相当费时间的

**d. IO缓存：**

使用具有缓存策略的输入流，BufferedInputStream替代InputStream，BufferedReader替代Reader，BufferedReader替代BufferedInputStream.对文件、网络IO皆适用。

**e. 消息缓存：**通过 Handler 的 obtainMessage 回收 Message 对象，减少 Message 对象的创建开销

handler.sendMessage(handler.obtainMessage(1));

**f. 通知栏notification缓存：**下载中需要不断改变通知栏进度条状态，如果不断新建Notification会导致通知栏很卡。这里我们可以使用最简单的缓存

Map notificationMap = new HashMap();如果notificationMap中不存在，则新建notification并且put into map.

**(3). 其他**

**能创建基类解决问题就不用具体子类：**除需要设置优先级的线程使用new Thread创建外，其余线程创建使用new Runnable。因为子类会有自己的属性创建需要更多开销。

**控制最大并发数量：**使用Java的Executors类，通过Executors.newFixedThreadPool(nThreads)控制线程池最大线程并发

**对于http请求增加timeout**

**3\. [Layout优化](http://www.trinea.cn/android/layout-performance/ "性能优化之布局优化")**

使用抽象布局标签(include, viewstub, merge)、去除不必要的嵌套和View节点、减少不必要的infalte及其他Layout方面可调优点，顺带提及布局调优相关工具(hierarchy viewer和lint)。具体可见[性能优化之布局优化](http://www.trinea.cn/android/layout-performance/ "性能优化之布局优化")

TextView属性优化：TextView的`android:ellipsize=”marquee”`跑马灯效果极耗性能，具体原因还在深入源码中

**4\. 数据库优化**

主要包括索引和事务及针对Sqlite的优化。具体可见[性能优化之数据库优化](http://www.trinea.cn/android/database-performance/ "性能优化之数据库优化")

**5\. 算法优化**

这个就是个博大精深的话题了，只介绍本应用中使用的。

使用hashMap代替arrayList，时间复杂度降低一个数量级

**6\. 延迟执行**

对于很多耗时逻辑没必要立即执行，这时候我们可以将其延迟执行。

**线程延迟执行** `ScheduledExecutorService scheduledThreadPool = Executors.newScheduledThreadPool(10);`

**消息延迟发送** `handler.sendMessageDelayed(handler.obtainMessage(0), 1000);`

**四、本程序性能调优结果**

**1\. ViewPager左右滑动明显卡顿**

**2\. GridView上下滚动明显卡顿**

(1). 去掉TextView的android:ellipsize=”marquee”

(2). 修改图片缓存的最大线程数，增加http timeout

(3). 修改设置app是否已安装的状态，具体代码修改如下：

``` java
List  installedPackageList  =  getPackageManager().getInstalledPackages(PackageManager.GET_UNINSTALLED_PACKAGES);

List  installedAppList  =  function(installedAppList)
for  (App app  :  appList)  {
	for  (App installedApp  :  installedAppList)  {
		
	}
}
```
修改为


``` java
for  (App app  :  appList)  {

PairInteger,  String>  versionInfo  =  INSTALLED_APP_MAP.get(app.getPackageName());
	
	if  (versionInfo  !=  null)  {
		}  else  {	
	}
}

```

从每次获取`List installedAppList = getPackageManager().getInstalledPackages(PackageManager.GET_UNINSTALLED_PACKAGES);`修改为只在有应用安装或卸载广播时获取应用列表，并且用hashMap代替installedAppList减少查询时间。

将平均执行时间从201ms降低到1ms。

**3\. 其他Activity返回ViewPager Activity较慢**

定位：在onStart函数

解决：使用延迟策略，具体代码修改如下：

```@Override

public  void  onStart()  {

super.onStart();

appUpdateListAdapter.notifyDataSetChanged();

}
```
改为

优化后代码

```
public void onStart() {
	super.onStart();
	// delay send message
	handler.sendMessageDelayed(handler.obtainMessage(MessageConstants.WHAT_NOTIFY_DATA_CHANGED), 100);
}
 
private class MyHandler extends Handler {
 
	@Override
	public void handleMessage(Message msg) {
		super.handleMessage(msg);
 
		switch (msg.what) {
			case MessageConstants.WHAT_NOTIFY_DATA_CHANGED:
				if (appUpdateListAdapter != null) {
					appUpdateListAdapter.notifyDataSetChanged();
				}
				break;
		}
	}
}
```

**4\. 网络获取到展现速度较慢**

定位：在HttpURLConnection.getInputStream()之后的处理

解决：使用BufferedReader替代BufferedInputStream获取时间从100ms降低到3ms，具体代码修改如下：

```
HttpURLConnection con  =  (HttpURLConnection)url.openConnection();

InputStream input  =  con.getInputStream();

while  (input.read(buffer,  0,  1024)  !=  -1)  {

}
```

改为

```
HttpURLConnection con  =  (HttpURLConnection)url.openConnection();

BufferedReader input  =  new  BufferedReader(new  InputStreamReader(con.getInputStream()));

String  s;

while  ((s  =  input.readLine())  !=  null)  {

}

```


