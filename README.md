# 百度聚屏广告SDK接入文档V1.2.2

## 产品介绍
百度聚屏广告SDK，合入了百度安全监播端-云一体方案，端上通过自有安全播放器、广告安全校验等技术，保障端上播放行为安全可控；云上通过自研安全算法、区块链等技术，从播放可信、设备可信、数据可信3个维度，保障广告播放行为真实可信；同时，SDK内置广告播放器具备高兼容性，支持高清流畅播放mp4、jpg等主流图片/视频广告格式。通过以上核心能力，可从端到云，全面保障媒体广告播放的真实性、安全性和流畅性。

## SDK接入须知
### 接入设备性能要求
为保证SDK稳定运行，媒体接入设备，性能需达到以下要求： 

* CPU：性能不低于ARM Cortex-A7；
* RAM：预留200-300MB以上；
* ROM：
   * SDK 1.0版本接入需预留300MB内部存储空间；
   * SDK 1.1及以上版本接入，含SDK卡总共需预留800MB以上；
* 系统版本：Android 4.2.2以上；
* 网络下载速度：2MB/s以上；
### 接入前信息准备
SDK接入前，需提前准备以下参数：

* appSid：资源id，由聚屏平台分配；
* Slotid：广告位id，一般包含图片和视频至少各1个广告位id，由聚屏平台分配；
* mediaId：
   * 若接入设备的广告类型为户外，则需准备该字段，该字段需保证唯一性且已在聚屏平台注册；
   * 若接入设备的广告类型为OTT或开屏广告，则可以填写为null。
* channel：渠道名，一般在交付邮件中将会列出；

以上信息若有疑问，可联系我们获取。


## 集成
在根gradle文件下加入：
```javascript { .theme-peacock }

allprojects {
    repositories {
        google()
        jcenter()
        maven { url 'https://dl.bintray.com/bdaiotsecurity/maven' }
    }
}
```


在app的gradle下加入：
```c
repositories {
    flatDir {
        dirs 'libs'
    }
}
dependencies {
    // 广告sdk
    implementation (name: 'adsdk_release_vxxx', ext: 'aar')
    // 请求SDK
    implementation (name: 'libtianjiadsdk-release-vxxx', ext: 'aar')
    // 内部库
    implementation "com.baidu.adsdk:saferequest:1.2.1.3"
    // glide
    implementation("com.github.bumptech.glide:glide:4.8.0") {
        exclude group: 'com.android.support'
    }
    // kotlin基础库
    implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk7:1.3.31"
    implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-android:1.2.1'
    // 视频播放器
    implementation 'com.google.android.exoplayer:exoplayer-core:2.10.3'
    // protobuf
    implementation 'com.google.protobuf:protobuf-lite:3.0.0'
    // okhttp
    implementation 'com.squareup.okio:okio:1.13.0'
    implementation 'com.squareup.okhttp3:okhttp:3.9.0'
}
//加入java 8支持
compileOptions {
  targetCompatibility JavaVersion.VERSION_1_8
}
```


## 初始化(必须）
在Application的onCreate中初始化。
```javascript { .theme-peacock }
@Override
public void onCreate() {
    super.onCreate();
    /**
     * @param appSid : 填入聚屏分配的appSid
     * @param channel : 填入聚屏分配的channel
     * @param mediaId :
     * 1. 如果广告位类型是户外，必须填写mediaid（64位以内，仅英文或数字或下划线_或冒号:）, 需要自己保证设备唯一性。
     * 另外，这个mediaid需要在聚屏方注册。
     * 2. 如果你的广告位类型是ott或者开屏广告，则填写为null,sdk将会自动获取mac进行请求。
     */
    AdWrapper.INSTANCE.init(this, 聚屏分配的appSid, 渠道， mediaId);
    AdWrapper.INSTANCE.enableTest(); // 如果需要使用测试环境才需要enable，否则不填
}
```

## 参数设置（可选，推荐填写）
下面在AdWrapper中，可通过`AdWrapper.Instance.XXX`访问，在init之后调用
```ruby
/**
 * @param screenSize : 填入设备的尺寸，不是像素，是宽高，单位用英寸
 */
fun setScreenSize(screenSize: PointF)

/**
 * @param isAIO : 是否是一体机。比如手机就是个一体机，智能电视也是一体机，电视盒子就不是一体机。
 */
fun setIsAIO(isAIO: Boolean)

/**
 * 传递请求时刻5s内探测到的强度TOP5的mac
 * 仅传递安卓设备的mac，过滤掉iOS及其他设备
 * 如选择填写, 如果填写就会利用此类信息触发广告,能获取到尽量传，可以传了可以触发更多的广告
 */
fun setProbInfo(infos: Map<String, Int>)
```

## 身份校验（必须）
以下信息需要给到百度，百度进行入库，否则将请求不到广告。出于安全性的考虑，后台需要对客户端鉴权，防止第三方拿到SDK之后作弊，目的是为了保障客户权益。
### 需要提供应用程序包名
例如：`com.baidu.xxxx`
也就是app得gradle文件中的`applicationId`
### APP签名的md5

```
keytool -printcert -file CERT.RSA
```

签名后从APK解压出来则可以看到此文件，文件放于在META-INF文件夹。
PS：keytool工具为JDK里自带。可在jdk\bin或者jdk\jre\bin目录下找到。
如果有debug运行需求，可以把debug版的签名一并给到我们（一般一台机器一个debug签名，可以按需求加）

## 权限（必须）
如果APP的apilevel>=23 也就是 Android版本>=6.0。
需要在第一个activity里申请权限。

```javascript { .theme-peacock }
if (Build.VERSION.SDK_INT > 23) {
    requestPermissions(new String[]{"android.permission.READ_PHONE_STATE",
                "android.permission.ACCESS_FINE_LOCATION"}, 101);
}
```

如果你的应用程序满足以下两个任意条件，可以忽略此步骤。

* targetSdkVersion < 23 选择将target 23设置小于23来规避此问题。
* 设备系统小于Android 6.0。

## 使用广告组件
目前提供以下两种接入方式。（注意，对同一个广告组建，要么使用自动方式，要么使用手动方式，在自动模式下如果标记为auto=true，内部将会自动根据设置来进行轮播，此时不需要外部进行控制。)
另外，必须在使用到组件的activity或fragment里的类静态初始化块中调用onActivityInit接口。
```
{
    // 必须在使用到组件的activity或fragment里的类静态初始化块中调用
    AdWrapper.INSTANCE.onActivityInit(this);
}
```

### xml 布局接入，简单，**推荐**
在Activity/Fragment的layout中像普通view一样布局。
其中公共属性如下：
`slotid`此参数需要向聚屏申请（重要，必填）
`auto` 是否是自动播放广告（选填）
#### 视频组件
```javascript { .theme-peacock }
// 外部这个laoyou自己决定
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".activitys.VideoViewAutoActivity">

    // 如果之前用的是BDVideoView，可以直接换成BDGLVideoView
    <com.baidu.adsdk.view.BDGLVideoView
        android:id="@+id/bd_video_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
	// 设置是否自动模式，如果这里填false，则必须使用手动代码方式来控制广告请求与播放。
        app:auto="true"
	// 设置视频广告位id
        app:slotid="Jy0FoE5qy" />
</android.support.constraint.ConstraintLayout>
```
接入后，视频组件将会一直持续播放，并自动请求广告。

#### 预先请求多个广告
* 通过Controller调用preLoadMoreAds(maxReqNums: Int, needAds: Int, callback: IPreLoadAdProgress)方法。
* 参数1 maxReqNums: 这次任务的最大请求数；
* 参数2 needAds: 这次任务需要下载的广告数，为避免计费串失效，最大为4；
* 参数3 callback: 这次任务的请求过程的回调,如下,参数意义为第几个请求，是否成功，错误码，请求信息

```
interface IPreLoadAdProgress {
	
    fun progress(index: Int, suc: Boolean, errCode: Int, info: RequestInfo)

}
```
##### 注意点：
* 因为每次请求不一定有广告，所以会有前面两个参数，避免频繁，无意义，不合理的请求；
* 参数1和参数2任一条件达到，此次任务结束；

##### 举例：
* 如果调用了 preLoadMoreAds(10, 3, callback), 那么最多请求10次广告，最多能提前准备好三个要播放的广告。如果请求了10次，还是没有能准备到3个广告，此次任务就会结束。

##### 和loadAdAsync方法的区别：
* preLoadMoreAds是提前准备，媒体可以根据回调判断提前请求的任务是否结束，结束后，调用loadAdAsync，会从已经准备好的广告列表里拿数据，有的话直接返回，没有的话再向聚屏服务器请求广告，其余流程不变。

#### 图片轮播组件
额外有一个独立的属性：
`intervalTime`设置x秒换一张图片，设置的时间必须在[5,30]秒中间，如果不在这个范围，默认15秒。
```javascript { .theme-peacock }
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".activitys.GalleryViewActivity">

    <com.baidu.adsdk.view.BDGalleryView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
	// 设置是否自动模式，如果这里填false，则必须使用手动代码方式来控制广告请求与播放。
        app:auto="true"
	// 图片播放时间，单位是s，设置的时间必须在[5,30]秒中间，如果不在这个范围，默认15秒。
        app:intervalTime="15"
	// 设置图片广告位id
        app:slotid="6660001" />
</LinearLayout>
```
接入后，图片轮播组件将会持续以固定的intervalTime间隔切换广告。

### 代码布局，媒体方自己通过代码控制广告请求与播放
下面这里例子使用了一个视频组件与一个图片轮播组件，模拟了一种场景：
播一个视频，播放成功后再播一个图片，再播放一个视频，由此往复循环。
当然你也可以使用我们提供的mix(二合一组件），里面已经把这个逻辑做好了。
这个例子主要是演示了几个接口如何使用。
```javascript { .theme-peacock }
/**
 * 自己控制广告流
 */
public class ManualActivity extends AppCompatActivity {

    Handler handler = new Handler();

    {
        AdWrapper.INSTANCE.onActivityInit(this);
    }

    /**
     * 视频广告组件
     */
    BDVideoView videoView;
    /**
     * 视频广告控制器
     */
    IController videoController;

    /**
     * 图片广告组件
     */
    BDGalleryView galleryView;
    /**
     * 图片广告控制器
     */
    IController galController;

    /**
     * 图片广告组件监听器
     */
    private GalListener galListener = new GalListener();

    /**
     * 视频广告组件监听器
     */
    private VideoListener videoListener = new VideoListener();

    class VideoListener extends DefaultAdListener {
        @Override
        public void onAdPrepared(@NotNull RequestInfo info) {
            // 收到视频广告准备OK的时候，可以自己做些业务逻辑，
            // 这个例子是将图片组件隐藏，显示视频组件，然后调用showAd接口
            videoView.setVisibility(View.VISIBLE);
            galleryView.setVisibility(View.INVISIBLE);
            videoController.showAd();
        }

        @Override
        public void onAdFailed(@NotNull RequestInfo info, int ec, @NotNull String msg) {
            // 当广告播放失败回调，这个例子在播放视频广告失败的时候继续请求视频广告
	    // 延迟1s，避免快速失败又快速请求刷屏
            handler.postDelayed(new Runnable() {
                @Override
                public void run() {
                    videoController.loadAdAsync();
                }
            }, 1000);
        }

        @Override
        public void onAdFinish(@NotNull RequestInfo requestInfo) {
            // 当广告完成时回调，这个例子在播放视频广告播放完成的时候请求图片广告
            galController.loadAdAsync();
        }
    }

    class GalListener extends DefaultAdListener {
        @Override
        public void onAdPrepared(@NotNull RequestInfo info) {
            // 收到图片广告准备OK的时候，可以自己做些业务逻辑，
            // 这个例子是将视频组件隐藏，显示图片组件，然后调用showAd接口
            videoView.setVisibility(View.INVISIBLE);
            galleryView.setVisibility(View.VISIBLE);
            galController.showAd();
        }

        @Override
        public void onAdFailed(@NotNull RequestInfo info, int ec, @NotNull String msg) {
            // 当图片广告播放失败回调，这个例子在图片广告失败再请求图片广告
	    // 延迟1秒，避免快速失败又快速请求刷屏
	    handler.postDelayed(new Runnable() {
                @Override
                public void run() {
                    galController.loadAdAsync();
                }
            }, 1000);
        }

        @Override
        public void onAdFinish(@NotNull RequestInfo requestInfo) {
            // 当图片广告播放完成回调，这个例子在图片广告播放完成请求视频广告
            videoController.loadAdAsync();
        }
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_video_manual);

        // 我们这个例子是再xml中布局视频与图片组件，
        // 你同样也可以在代码中使用LayoutParams布局
        videoView = findViewById(R.id.bdvv_test);
        galleryView = findViewById(R.id.bdvv_gal_test);

        // 初始化图片组件，设置监听器
        galController = galleryView.getController();
        galController.addAdListener(galListener);

        // 初始化视频组件，设置监听器
        videoController = videoView.getController();
        // 你可以通过setSlotid设置广告位id，也可以通过xml中设置
        videoController.setSlotid("Jy0FoE5qy");
        videoController.addAdListener(videoListener);

        // 发起视频组件的广告请求
        videoController.loadAdAsync();
    }
    
    @Override
    protected void onDestroy() {
        super.onDestroy();
        videoController.releaseAd();
        galController.releaseAd();
    }

    /**
     * 提供一个骨架方法，业务方不需要重写每个回调时使用。仅为降低代码冗余度
     */
    public static class DefaultAdListener implements IAdListener {
        @Override
        public void onAdPrepared(@NotNull RequestInfo info) {

        }

        @Override
        public void onAdStart() {

        }

        @Override
        public void onAdFailed(@NotNull RequestInfo info, int ec, @NotNull String msg) {

        }

        @Override
        public void onAdClick() {

        }

        @Override
        public void onAdFinish(@NotNull RequestInfo info) {

        }

        @Override
        public void onAdDismissed() {

        }
    }
}

```
## 截图接口
* 通过IController对象，调用captureAsync接口，返回正在播放的Bitmap，返回后需要自行回收！

## 播放本地素材接口
* 通过IController对象，调用playLocalMedia接口，AdData里面的url赋值给本地素材的路径。

## 监播回调接口
* 通过IController对象，调用setMonitorCallback接口，在广告播放过程中回回调监播情况。

## 音量控制

* 通过View先拿到IControl对象，接着调用setVolume(float volume)来设置音量的大小，volume取值范围在0~1，0代表无声，1代表最大音量。需要在loadAd之前调用。

## 广告播放回调接口(IAdListener, package com.baidu.adsdk.interfaces)

回调接口的具体含义如下：

* onAdPrepared  已经下载好了，但是还没有开始播放。preparedinfo里面有广告请求的唯一标识符与广告位id，此时可以通过controller调用showAd
* onAdStart  已经开始播放了。（是一个时间点）
* onAdFailed和onAdFinish平行，要么成功完成，要么失败
* onAdClick点击广告时回调
* onAdDismissed比如按了home键或者其他activtiy在这个View上面，这个方法就会被回调

## 额外特性

如果你还有第三方广告参与，需要做如下工作。
可以自定义的流程包含如下几个阶段：请求广告、监播结果、计费、自定义播放器
这部分整体流程如下图：
![image](https://github.com/biduaiotsecurity/adsdk_dev_help/blob/master/%E9%A2%9D%E5%A4%96%E7%89%B9%E6%80%A7.jpg)
### 请求广告
这部分关心如下几个接口：
```javascript { .theme-peacock }
package com.baidu.adsdk.model.interfaces

interface IADModel {
    // 将在一开始被回调一次
    fun init(context: Context, sdkContext: SdkContext)

    // 请求广告
    fun requestAd(requestId: String, slotId: String, request: IRequestResult)

    // 计费接口，没有就返回null
    fun createBill(adData: AdData): IBill?
}

interface IRequestResult {
    // 使用者内部请求到广告后，将数据回传给SDK，这个接口在上面requestAd时已传入内部
    fun onRequestSuccess(adData: AdData)
    // 使用者内部请求到广告后，将数据回传给SDK，这个接口在上面requestAd时已传入内部
    fun onRequestFailed(code: Int, requestInfo: RequestInfo)
}


interface IModelManager {
    // 初始化，SDK将在初始化时调用此接口一次。
    fun init(sdkContext: SdkContext)
    // 返回一个图片类别的model给SDK进行播控，当有多个model时候，可以自己定制策略返回适当的model。
    fun getImageModel(): IADModel?
    // 返回一个视频类别的model给SDK进行播控，当有多个model时候，可以自己定制策略返回适当的model。
    fun getVideoModel(): IADModel?
}
```

* 实例化一个或多个IADModel，可以负责请求图片类广告、请求视频类广告。
* 实例化一个IModelManager。
* AdWrapper.INSTANCE.addModelManager(传入自己的实例)

### 计费
这部分关心如下两个接口：

```javascript { .theme-peacock }
package com.baidu.adsdk.model.interfaces

interface IBill {
    /**
     * 给机会计费，只在这里传递通用数据，具体其它需要的数据可以在createBill里做自己设置进去。
     */
    fun calculateCost(adData: AdData, billResult: IBillResult)
}

interface IBillResult {
    fun onBillSuccess(adData: AdData)
    fun onBillFailed(adData: AdData, ec: Int)
}
```

* 创建一个实例实现IBill接口
* 在model的createBill接口实现部分返回你的IBill实例
* 在calculateCost实现计费逻辑，并将计费结果通过IBillResult接口返回给SDK

### 自定义播放器
SDK支持自定义播放器。
关心如下几个接口

```javascript { .theme-peacock }
com.baidu.adsdk.interfaces

interface IPlayer {

    /**
     * @param path while the file is downloaded.
     * */
    fun preparing(path: String)

    /**
     * start to play prepare video or image.
     * Before invoke start(), you must invokes {@link #prepare()} first.
     * */
    fun start()

    /**
     * stop to play current video or image.
     * */
    fun stop()

    /**
     * When view is being destroy, this function must be invoked
     * */
    fun release()

    /**
     * set Display surfaceholder
     * */
    fun setDisplay(sh: SurfaceHolder)

    /**
     * set surface
     * */
    fun setSurface(surface: Surface)

    /**
     * get duration
     * @return
     * */
    fun getDuration(): Long

    /**
     * set volume
     * */
    fun setVolume(leftVolume: Float, rightVolume: Float)

    fun setIPlayerCallback(callback: IPlayerCallback)

}

interface IPlayerCallback {

    fun notifyPlayerPrepared()

    fun notifyPlayerFail(errorCode: Int)

    fun notifyPlayerCompetition()

}
```
* 使用步骤：
* ①自己的播放器实现IPlayer接口。
* ②SDK内部自己会调用setIPlayerCallback传给播放器,播放器需要保存这个接口在适当时候回调给SDK内部。
* ③通过View拿到Controller,调用setIPlayer即可

## 限制

* 不允许在播放过程中遮盖、隐藏、调整大小广告组件，否则将会影响收益。
* 广告组件大小不能小于300*300px， 否则将会影响收益。

## 附录

### 时序图
![image](https://github.com/biduaiotsecurity/adsdk_dev_help/blob/master/%E8%81%9A%E5%B1%8FSDK%E6%97%B6%E5%BA%8F%E5%9B%BE.jpg)


### 错误列表（持续更新）
```javascript { .theme-peacock }
-1 unknow 大多是网络问题，网络不好的情况
-2 illegal argument
-3 time_out 连接超时
-4 permission_deny
-5 url is empty
-6 md5 doesn't match 下载后的文件与云端md5匹配与否
-7 not_support_breakpoint
-8 unknown_host_exception
-9 socket_exception
-10 runtimeexception
-11 identity_verification_failed sdk身份校验错误，这个一般是因为app包名与app签名的md5没有入库导致的。
-15 sslhandshake_exception
-16 ssl_exception
-17 timeout_cancellation_exception 这个就是下载物料超时，这一块可以检查下网络
-101 get_media_connection_error
-102 media_play_unknown_error
-103 media_decoder_init_error
-104 media_play_pause
-200 surface_no_create

-201 prepare_fail_file_not_exist
-202 prepare_fail_inputstream_valid
-203 prepare_fail_unknown
-204 start_fail
-205 stop_fail

-300 monitor_no_adta
-302 monitor_capture_null

SDK身份校验错误码：
retCode 为 0x01 包名或者md5不匹配
	   0x02 请求格式不正确或者参数错误
	   0x03 解密失败
	   0x04 鉴权失败
```
### 聚屏返回错误对照表  
百度聚屏 API 错误码对照表 （适用于API v6.0及以上版本）  
   
1 前言  
错误码对照表文件作为对API 接口的补充说明，在API 接口升级后，会不定期更新维护。每个错误码对应的 处理方法均为引导性建议，对于具体的错误需要相关技术人员根据具体情况来处理。 对文档中未说明的内容，请咨询百度聚屏。  
注：联调前，请媒体技术人员先本地验证proto序列化、反序列化功能正确。proto版本为2.4  
2 无错误响应 

|错误码| 定义| 解释|
|----|----|----|
|0| NO_ERROR| 请求处理正确|  
   
3 请求信息错误  
  
3.1 请求基础信息错误  

|错误码| 定义| 解释|  
|----|----|----|  
|100001| MISSING_REQUEST| 请求参数缺失 |  

注：请求数据无法解析，或解析后请求ID缺失/格式异常时。返回http状态码400，无错误码  
3.2 API 版本信息错误  

|错误码| 定义| 解释 |  
|----|----|----|  
|102000|  MISSING_API_VERSION |使用API 版本信息缺失 |  
|102010| MISSING_API_VERSION_MAJOR| API API主版本信息缺失 |  
|102011 |ERROR_API_VERSION |API API版本信息错误 |  

3.3 APP_ID信息错误  

|错误码| 定义| 解释|  
|----|----|----|  
|103010  |MISSING_APP_ID| APP_ID信息缺失 |  
|103011| ERROR_APP_ID| APP_ID信息错误，MSSP未收录 |  
   
3.4 设备信息错误   

|错误码| 定义| 解释|  
|----|----|----|  
|104000| MISSING_DEVICE_INFO| 设备信息缺失|  
|104010| MISSING_DEVICE_TYPE| 设备类型信息缺失|  
|104011| ERROR_DEVICE_TYPE| 设备类型信息错误|  
|104020| MISSING_OS_TYPE| 操作系统信息缺失|  
|104021| ERROR_OS_TYPE| 操作系统信息错误|  
|104030| MISSING_OS_VERSION| 操作系统版本信息缺失|  
|104040| MISSING_OS_VERSION|_MAJOR 操作系统主版本信息缺失|  
|104050| MISSING_VENDOR| 厂商信息缺失|  
|104060| MISSING_MODEL| 设备型号信息缺失|  
|104070| MISSING_UDID| 设备唯一标识符缺失|  
|104071| ERROR_UDID| 设备唯一标识符未备案|  
|104090| MISSING_SCREEN_SIZE| 设备屏幕尺寸信息缺失|  
|104100| MISSING_SCREEN_SIZE_WIDTH| 设备屏幕尺寸宽度缺失|  
|104110| MISSING_SCREEN_SIZE_HEIGHT| 设备屏幕尺寸高度缺失|  
|104120| MISSING_UDID_ID_TYPE| 设备唯一标识符类型缺失|  
|104121| ERROR_UDID_ID_TYPE| 设备唯一标识符类型错误|  
|104130| MISSING_UDID_ID| 设备唯一标识符ID值缺失|  
|104140| ERROR_FORMAT_MAC| 设备mac不符合约定格式|   

3.5 网络环境信息错误  

|错误码| 定义| 解释|  
|----|----|----|  
|105000| MISSING_NETWORK_INFO| 网络环境信息缺失|  
|105010| MISSING_IPV4| 网络地址信息缺失|  
|105011| ERROR_FORMAT_IPV4| 网络地址信息格式错误|  
|105020| MISSING_CONNECTION_TYPE| 网络连接类型缺失|  
|105021| ERROR_CONNECTION_TYPE| 网络连接类型错误|  
|105030| MISSING_OPERATOR_TYPE| 网络运营商类型缺失|  
|105031| ERROR_OPERATOR_TYPE| 网络运营商类型错误|  
|105040| MISSING_AP_MAC| Wi-Fi 热点地址信息缺失|  
|105041| ERROR_FORMAT_AP_MAC| Wi-Fi 热点地址信息格式错误|  
|105050| MISSING_RSSI| Wi-Fi 热点信号强度信息缺失|  
|105060| MISSING_AP_NAME| Wi-Fi 热点名称缺失|  
|105070| MISSING_AP_CONNECTION| Wi-Fi 连接状态信息缺失|  

3.6 GPS 坐标信息错误  

|错误码| 定义| 解释|  
|----|----|----|  
|106000| MISSING_COORDINATE_TYPE| 坐标类型信息缺失|  
|106001| ERROR_COORDINATE_TYPE| 坐标类型信息错误|  
|106010| MISSING_LONGITUDE| 经度信息缺失|  
|106020| MISSING_LATITUDE| 纬度信息缺失|  
|106030|  MISSING_GPS_TIMESTAMP| 定位时间戳信息缺失|   

3.7 广告位信息错误    

|错误码| 定义| 解释|  
|----|----|----|  
|107000| MISSING_ADSLOT_ID| 广告位ID缺失|  
|107001| ERROR_ADSLOT_ID| 广告位ID未收录|  
|107003| NOT_MATCH_ADSLOT_ID| 广告位ID与APP_ID不匹配|  
|107010| MISSING_ADSLOT_SIZE| 广告位尺寸信息缺失|  
|107040| MISSING_ADSLOT| 广告位信息缺失|  

3.8 其他错误  

|错误码| 定义| 解释|  
|----|----|----|  
|400000| FLOW_DROP_BY_TIME_CONTORL| 时段流量丢弃(仅户外、出行)|  

4 响应信息错误  
4.1 广告请求  

|错误码| 定义| 解释|  
|----|----|----|  
|200000| NO_AD| 请求处理正确，无广告返回|  
|200003| AD_ASP_RETURN_ERR| 请求处理正确，无广告返回(无预算导致)|  
|201010| AD_NO_SIGN |广告无签名|  
|201020| MISSING_CRETIVE_TYPE| 广告创意类型信息丢失|  
|201021| ERROR_CRETIVE_TYPE| 广告创意类型信息无法识别|    
|201030| MISSING_INTERATION_TYPE| 广告动作类型信息丢失|   
|201031| ERROR_INTERATION_TYPE| 广告动作类型信息无法识别|  
|201040| MISSING_WIN_NOTICE_URL| 曝光汇报地址丢失|  
|201041| ERROR_WIN_NOTICE_URL_SIZE| 曝光汇报地址异常|  
|201050| MISSING_CLICK_URL| 点击响应地址丢失|   
|201060| MISSING_TITLE| 推广标题丢失|  
|201070| MISSING_DESCRIPTION| 推广描述丢失|  
|201080| MISSING_APP_PACKAGE| 推广应用包名丢失|  
|201090| MISSING_APP_SIZE| 推广应用包大小丢失|  
|201100| MISSING_ICON_SRC| 推广图标丢失|  
|201110| MISSING_IMAGE_SRC| 推广图片丢失|  
|201111| AD_BAD_JSON| 广告json串错误|  
  
5 典型case  
1) 接口返回http状态码400 请求数据无法解析，或解析后请求ID 缺失/格式异常时。返回http状态码400，无错误码。请本地验证请求数据 可解析，并检查请求ID格式满足要求：仅英文字母和数字，32 位，大小写不敏感  
2) 设备ID未备案（104071 ERROR_UDID） 请求参数中udid未在媒体配置表中提供，认为是非法流量，返回 104071错误。  
3) API版本信息错误 API主版本号必填，初始版本为6.0.0  
4) APP_ID未收录（103011 ERROR_APP_ID） app_id填写错误，请确认与mssp申请的app_id一致  
5) 广告位ID 未收录（107001 ERROR_ADSLOT_ID） 广告位id填写错误，请确认与mssp申请的广告位id一致  
6) 广告位ID 与APP_ID不匹配（107003 NOT_MATCH_ADSLOT_ID） 请确认请求参数中，广告位ID、APP_ID在媒体配置表是对应关系  
7) 无广告返回处理 200000 NO_AD：指百度聚屏内部错误，导致无广告返回。 201000 AD_NO_DATA：指请求处理正常，但无广告主投放，因此无广告返回；返回201000 可以认为接口测 试通过。  
8) proto版本错误 请按proto2版本开发。使用proto3可能出现解析异常等问题。  
9) 必填参数缺失 proto注释中标明为“必填”参数，不可缺失，否则接口返回错误。特别注意以下两点 - 参数是否“必填”，以proto注释为准。请勿参照字段限定修饰符中的optional/required - wifi ap参数为选填，如选择填写，则ap_mac/rssi/ap_name不可为空。类似的还有探针参数、GPS参数  
10) 参数格式错误 部分参数有格式校验，如request_id、ipv4、udid、ap_mac、client_mac等。校验不通过，接口返回错误  
11) 枚举值参数超出全集 os_type、connection_type、operator_type、coordinate_type取值不可超出proto 给出的枚举值，否则接 口返回错误  
12) 返回信息http头部字段 返回信息中，http头部字段Content-Length与返回内容实际大小不一致。解析时请勿使用。  

### Q&A（持续更新）
* 1）Q：Unable to resolve dependency for XXX Could not download safehttp.aar (com.baidu.safehttp:safehttp:1.0.29)
* A：到https://bintray.com/bdaiotsecurity 或者咨询相应的开发人员获取aidl-1.0.0、safehttp-1.0.13以及saferequest-1.2.1.3等aar包。
```c
并且替换
    implementation "com.baidu.adsdk:saferequest:1.2.1.3"
为：
    implementation(name: 'aidl-1.0.0', ext: 'aar')
    implementation(name: 'safehttp-1.0.29', ext: 'aar')
    implementation(name: 'saferequest-1.2.1.3', ext: 'aar')
```

* 2)Q: protubuf和宿主版本不一致导致编译失败。
* A: SDK版本用的是protubuf3.0，会用到一些新特性，但是也向下兼容，宿主版本如果之前用的是3.0以下的，先clean一下工程，把之前protubuf自动生成的代码先清掉，再用3.0重新编译即可。

* 3)Q: 使用BDGalleryView崩溃，抛出NoXXXFoundException。
* A: 这个是因为宿主用的Glide版本和SDK不一致导致的，Glide版本间差异会比较大，不同版本引入的新特性不一定能兼容。

* 4) Q: NullPointerException异常，并且堆栈在：EncryptUtilWrapper的。
* A： 看下app的Manifest文件中Application节点中是否加入了:process属性。
      一定要在application onCreate中调用我们sdk。

