# MediaPicker
Android多媒体图片音频视频可限制大小时间自定义选择器库

项目中许多时候需要选择图片、音视频，并有大小和时间限制，没有找到合适的库，所以自己提供一个满足需求的基础版本。
一.可以选择图片、音频、视频
二.可以限制选择数量、音视频大小、时长
三.可以进行图片、音视频的预览播放

## 一.实现效果
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019051319181356.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsMjAxOGdvZA==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190513191836136.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsMjAxOGdvZA==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190513192050374.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsMjAxOGdvZA==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190513192101200.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NsMjAxOGdvZA==,size_16,color_FFFFFF,t_70)
可以设置显示主题，默认为深色主题
## 二.引用
1.Add it in your root build.gradle at the end of repositories
```
allprojects {
		repositories {
			...
			maven { url 'https://jitpack.io' }
		}
	}
```
2.Add the dependency

```
dependencies {
	        implementation 'com.github.MingYueChunQiu:MediaPicker:0.1'
	}
```
## 三.使用
### 1.基础使用
最简单的使用，全部为默认配置选择图片
```
MediaPicker.init(MainActivity.this)
                        	.pick();
```
获取到的结果在

```
@Override
    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (data != null && requestCode == MP_REQUEST_START_MEDIA_PICKER && resultCode == RESULT_OK) {
            ArrayList<MediaInfo> list = data.getParcelableArrayListExtra(EXTRA_PICKED_MEDIA_LIST);
            for (MediaInfo info : list) {
                Log.d("份", info.getTitle() + "   fds    " + info.getName() + " " + info.getFilePath() + " " +
                        info.getSize() + " " + info.getDuration() + " " + info.getBucketId() + " "
                        + info.getBucketName());
            }
        }
    }
```
结果存储在MediaInfo中

```
public class MediaInfo：
    private String title;//标题
    private String name;//名称（带扩展名）
    private MediaPickerType type;//多媒体类型
    private String filePath;//视频路径
    private String thumbnail;//缩略图
    private long addDate;//添加到Media Provider的时间
    private long duration;//时长
    private long size;//大小
    private String bucketId;//多媒体所属文件夹ID
    private String bucketName;//多媒体所属文件夹名称
```

在MediaPicker主要是提供MediaPickerControlable接口实例，默认提供的是MediaPickerControl子类

```
public class MediaPicker {

    public static final MediaPicker INSTANCE;//单例
    private MediaPickerControlable mControl;

    private MediaPicker() {
    }

    static {
        INSTANCE = new MediaPicker();
    }

    public static MediaPickerControlable init(@NonNull Context context) {
        return init(context, new MediaPickerStore(context), null);
    }

    public static MediaPickerControlable init(@NonNull Context context, MediaPickerStoreable store, MediaPickerInterceptable intercept) {
        INSTANCE.mControl = new MediaPickerControl(context, store, intercept);
        return INSTANCE.mControl;
    }

    public MediaPickerControlable getMediaPickerControl() {
        return INSTANCE.mControl;
    }

    public static ImageEngine getImageEngine() {
        return INSTANCE.mControl.getImageEngine();
    }
}
```
在拿到MediaPickerControlable后，设置相关配置，MediaPickerControlable里持有MediaPickerStoreable接口，默认提供的子类实现是MediaPickerStore。

MediaPickerStore主要是用来持有MediaPickerConfig，进行配置设置，提供MediaPickerControlable和MediaPickerStoreable主要是既可以通过传入MediaPickerConfig配置，也可以直接调用MediaPickerControlable进行配置，最终都转换为MediaPickerConfig。

MediaPickerControlable包裹MediaPickerStoreable，使用与实现中间层拦截器，方便实现中间额外操作，所以提供MediaPickerInterceptable接口，默认提供了MediaPickerIntercept空实现子类，可以对所有方法进行拦截监听
### 2.MediaPickerControlable

```
public interface MediaPickerControlable {

    MediaPickerControlable setMediaPickerConfig(MediaPickerConfig config);

    MediaPickerControlable setMediaPickerIntercept(MediaPickerInterceptable intercept);

    MediaPickerControlable setMediaPickerType(MediaPickerType mediaPickerType);

    MediaPickerControlable setMaxSelectMediaCount(int maxSelectMediaCount);

    MediaPickerControlable setLimitSize(long limitSize);

    MediaPickerControlable setLimitDuration(long limitDuration);

    MediaPickerControlable setFilterLimitMedia(boolean filterLimitMedia);

    MediaPickerControlable setColumnCount(int columnCount);

    MediaPickerControlable setStartPreviewByThird(boolean startPreviewByThird);

    MediaPickerControlable setThemeConfig(MediaPickerThemeConfig config);

    MediaPickerControlable setImageEngine(ImageEngine engine);

    ImageEngine getImageEngine();

    MediaPickerStoreable getMediaPickerStore();

    void pick();

    void release();
}
```
### 3.MediaPickerStoreable

```
public interface MediaPickerStoreable {

    MediaPickerStoreable setMediaPickerConfig(MediaPickerConfig config);

    MediaPickerStoreable setMediaPickerType(MediaPickerType mediaPickerType);

    MediaPickerStoreable setMaxSelectMediaCount(int maxSelectMediaCount);

    MediaPickerStoreable setLimitSize(long limitSize);

    MediaPickerStoreable setLimitDuration(long limitDuration);

    MediaPickerStoreable setFilterLimitMedia(boolean filterLimitMedia);

    MediaPickerStoreable setColumnCount(int columnCount);

    MediaPickerStoreable setStartPreviewByThird(boolean startPreviewByThird);

    MediaPickerStoreable setThemeConfig(MediaPickerThemeConfig config);

    MediaPickerStoreable setImageEngine(ImageEngine engine);

    MediaPickerConfig getMediaPickerConfig();

    void pick();

    void release();
}
```
### 4.MediaPickerConfig
MediaPickerStore实现类里会持有MediaPickerConfig

```
private MediaPickerType mediaPickerType;//多媒体选择类型

    private int maxSelectMediaCount;//最多可选择多媒体个数

    private long limitSize;//限制大小（单位B）

    private long limitDuration;//限制时长（毫秒）

    private boolean filterLimitMedia;//是否过滤超出限制的多媒体信息

    private int columnCount;//一行列数

    private int loadAnimation;//Item加载动画

    private boolean startPreviewByThird;//以第三方应用方式打开预览多媒体

    private MediaPickerThemeConfig themeConfig;//主题配置

    private ImageEngine engine;//图片加载引擎
```
可以通过startPreviewByThird来设置是否通过调用第三方应用来预览，默认是库自带的预览效果，目前设置后只有视频可以打开调用其他第三方应用预览。

在MediaPickerConfig里可以配置界面主题深色和浅色，默认为深色，同时也可以自定义设置

```
public class MediaPickerThemeConfig：

	private int themeType;//主题类型（浅色或深色，请设置此类时一定要配置涉及到状态栏配置，默认为深色）

    private @ColorInt
    int topBackgroundColor;//顶部背景颜色

    private @ColorInt
    int bottomBackgroundColor;//底部背景颜色

    private @ColorInt
    int topTextColor;//顶部文字颜色

    private @ColorInt
    int bottomTextColor;//底部文字颜色

    private @DrawableRes
    int backIconResId;//返回Icon资源ID

    private @DrawableRes
    int upTriangleIconResId;//向上三角Icon资源ID

    private @DrawableRes
    int downTriangleIconResId;//向下三角Icon资源ID
```
MediaPickerThemeConfig默认提供了buildLightTheme和buildDarkTheme

```
/**
         * 创建默认浅色主题配置信息
         *
         * @return 返回浅色主题配置信息对象
         */
        public MediaPickerThemeConfig buildLightTheme() {
            mConfig.topBackgroundColor = Color.WHITE;
            mConfig.bottomBackgroundColor = Color.parseColor("#33000000");
            mConfig.topTextColor = Color.BLACK;
            mConfig.bottomTextColor = Color.BLACK;
            mConfig.backIconResId = R.drawable.mp_back_dark;
            mConfig.upTriangleIconResId = R.drawable.mp_up_triangle_dark;
            mConfig.downTriangleIconResId = R.drawable.mp_down_triangle_dark;
            mConfig.themeType = Constants.ThemeTypeConstants.TYPE_LIGHT;
            return mConfig;
        }

        /**
         * 创建默认深色主题配置信息
         *
         * @return 返回深色主题配置信息对象
         */
        public MediaPickerThemeConfig buildDarkTheme() {
            mConfig.topBackgroundColor = Color.parseColor("#2C2C34");
            mConfig.bottomBackgroundColor = Color.parseColor("#2C2C34");
            mConfig.topTextColor = Color.WHITE;
            mConfig.bottomTextColor = Color.WHITE;
            mConfig.backIconResId = R.drawable.mp_back_light;
            mConfig.upTriangleIconResId = R.drawable.mp_up_triangle_light;
            mConfig.downTriangleIconResId = R.drawable.mp_down_triangle_light;
            mConfig.themeType = Constants.ThemeTypeConstants.TYPE_DARK;
            return mConfig;
        }
```
库同时还提供了MediaUtils工具类，里面有许多工具方法

```
/**
     * 启动选择本地图片界面
     *
     * @param activity    启动界面
     * @param requestCode 启动请求码
     */
    public static void startPickImage(@NonNull Activity activity, int requestCode) {
    }

    /**
     * 启动选择本地图片界面
     *
     * @param fragment    启动界面
     * @param requestCode 启动请求码
     */
    public static void startPickImage(@NonNull Fragment fragment, int requestCode) {
    }

    /**
     * 启动选择本地视频界面
     *
     * @param activity    启动界面
     * @param requestCode 启动请求码
     */
    public static void startPickVideo(@NonNull Activity activity, int requestCode) {
    }

    /**
     * 启动选择本地视频界面
     *
     * @param fragment    启动界面
     * @param requestCode 启动请求码
     */
    public static void startPickVideo(@NonNull Fragment fragment, int requestCode) {
    }

/**
     * 查询系统数据库地址中视频信息
     *
     * @param resolver Android组件
     * @param uri      视频本地地址
     * @return 如果成功获取数据，则返回MediaInfo，否则返回null
     */
    @Nullable
    public static MediaInfo queryVideoInfo(@NonNull ContentResolver resolver, @NonNull Uri uri) {
    }

    /**
     * 根据缩略图路径获取缩略图
     *
     * @param path 缩略图路径
     * @return 返回生成的缩略图
     */
    public static Bitmap getThumbnail(String path) {
    }

    /**
     * 设置视频播放声音
     *
     * @param volume 声音音量（0--1）
     * @param o      播放的对象
     */
    public static void setVolume(float volume, Object o) {
    }

    /**
     * 获取图片资源
     *
     * @param context  上下文
     * @param callback 浏览资源回调
     */
    public static void getImages(final Context context, BrowseMediaInfoCallback callback) {
    }

/**
     * 获取音频资源
     *
     * @param context  上下文
     * @param callback 浏览资源回调
     */
    public static void getAudios(final Context context, BrowseMediaInfoCallback callback) {
    }

/**
     * 获取视频资源
     *
     * @param context  上下文
     * @param callback 浏览资源回调
     */
    public static void getVideos(final Context context, BrowseMediaInfoCallback callback) {
    }

@NonNull
    private static Intent getPickImageIntent() {
    }

    /**
     * 获取选择视频的启动意图
     *
     * @return 返回启动意图
     */
    @NonNull
    private static Intent getPickVideoIntent() {
    }

 /**
     * 浏览多媒体信息回调
     */
    public interface BrowseMediaInfoCallback {

        /**
         * 当准备浏览多媒体信息时调用
         */
        void onPrepareBrowseMediaInfo();

        /**
         * 当开始浏览多媒体信息时调用
         *
         * @param count 多媒体总数
         */
        void onStartBrowseMediaInfo(int count);

        /**
         * 浏览多媒体资源信息时调用
         *
         * @param index 浏览的索引位置
         * @param info  多媒体信息数据
         */
        void onBrowseMediaInfo(int index, @NonNull MediaInfo info);

        /**
         * 当结束浏览多媒体信息时回调
         */
        void onEndBrowseMediaInfo();
    }
```

### 5.图片引擎
用户可以设置ImageEngine

```
 MediaPicker.init(MainActivity.this)
                        .setImageEngine()
                        .pick();
```
ImageEngine 接口如下，默认提供了GlideEngine，如果和项目有冲突，可以用自己的图片引擎替换
```
public interface ImageEngine {

    /**
     * 显示图片
     *
     * @param context       上下文
     * @param file          显示图片
     * @param placeholderId 占位图资源
     * @param errorId       错误图资源
     * @param imageView     显示控件
     */
    void showImage(Context context, File file,
                   @DrawableRes int placeholderId, @DrawableRes int errorId, ImageView imageView);

    /**
     * 显示图片
     *
     * @param fragment      界面
     * @param file          显示图片
     * @param placeholderId 占位图资源
     * @param errorId       错误图资源
     * @param imageView     显示控件
     */
    void showImage(Fragment fragment, File file,
                   @DrawableRes int placeholderId, @DrawableRes int errorId, ImageView imageView);
}
```
## 四.总结
目前用户所需要使用的源码都在上面，其他预览相关代码就暂不介绍，库刚出来，如果有什么疑问或建议欢迎提出，Github地址：https://github.com/MingYueChunQiu/MediaPicker.git ， 码云地址：https://gitee.com/MingYueChunQiu/MediaPicker.git ，感谢你的支持，如果不介意请Github点个star，谢谢。