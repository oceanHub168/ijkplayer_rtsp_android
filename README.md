# ijkplayer_rtsp_android
优化后ijkplayer播放器支持rtsp拉流、低延迟


## 编译ijkplayer ,支持rtsp拉流、低延迟

git clone https://github.com/Bilibili/ijkplayer.git ijkplayer-android

cd ijkplayer-android

git checkout -B latest k0.8.4

./init-android.sh  
(修改文件支持低延时： 
1.参照ubutun18.04上 /opt/ijkplayer-android/ijkmedia/ijkplayer/ff_ffplay.c
2.参照ubutun18.04上 /opt/ijkplayer-android/ijkmedia/ijkplayer/ijkavformat/ijklivehook.c, 0.8.4没用到，在0.8.8)

rm -r module.sh
修改module-lite.sh 中配置项 vim module-lite.sh中添加如下配置
(参照D:\CLib\ijkplayer模板\module-lite.sh)


ln -s module-lite.sh module.sh


cd android/contrib
./compile-ffmpeg.sh clean
./compile-ffmpeg.sh all

cd ..
./compile-ijk.sh all

至此，编译出libijkffmpeg.so 、 libijkplayer.so 、 libijksdk.so

## 参数介绍
>  mMediaPlayer.setOption(IjkMediaPlayer.OPT_CATEGORY_PLAYER, "fast", 1);

类型|参数|参数类型|说明|备注|无
:-:|:-:|:-:|:-:|:-:|:-:
IjkMediaPlayer.OPT_CATEGORY_PLAYER |"fast"|int|丢帧阈值|1|
IjkMediaPlayer.OPT_CATEGORY_PLAYER |"probesize"|int|丢帧阈值|200|
IjkMediaPlayer.OPT_CATEGORY_PLAYER |"flush_packets"|int|丢帧阈值|1|
IjkMediaPlayer.OPT_CATEGORY_PLAYER |"framedrop"|int|丢帧阈值|1|
IjkMediaPlayer.OPT_CATEGORY_CODEC |"skip_loop_filter"|int|视频帧率|48|
IjkMediaPlayer.OPT_CATEGORY_PLAYER |"packet-buffering"|int|packet缓存|0|
IjkMediaPlayer.OPT_CATEGORY_PLAYER |"infbuf"|int|不限制拉流缓存大小|1|
IjkMediaPlayer.OPT_CATEGORY_FORMAT |"max-buffer-size"|int|最大缓存数量|0|
IjkMediaPlayer.OPT_CATEGORY_PLAYER |"min-frames"|int|最小解码帧数|2|
IjkMediaPlayer.OPT_CATEGORY_PLAYER |"start-on-prepared"|int|启动预加载|1|
IjkMediaPlayer.OPT_CATEGORY_PLAYER |"mediacodec"|int|是否开启硬加载|0：开启|
IjkMediaPlayer.OPT_CATEGORY_PLAYER |"mediacodec-auto-rotate"|int|自动旋屏|0|
IjkMediaPlayer.OPT_CATEGORY_PLAYER |"mediacodec-handle-resolution-change"|int|处理分辨率变化|0|
IjkMediaPlayer.OPT_CATEGORY_FORMAT |"analyzeduration"|int|设置分析流时长|2000000|
IjkMediaPlayer.OPT_CATEGORY_FORMAT |"rtsp_transport"|string|可以改为tcp协议：|tcp|
IjkMediaPlayer.OPT_CATEGORY_FORMAT |"rtsp_transport"|string|可以改为tcp协议：|tcp|
IjkMediaPlayer.OPT_CATEGORY_FORMAT |"rtsp_flags"|string|tcp|prefer_tcp|

## 项目中使用ijkplayer 

方法一：
1.将AndroidIjkplayer-1.0.0.aar 放到app--->libs 目录下
2.在app的 build.gradle 中dependencies节点前单独添加

repositories {

    flatDir {
    
        dirs 'libs'
        
    }
}

方法二：
- 在项目(最外层)的build.gradle 下面添加仓库地址 

```
allprojects {
    repositories {
        google()
        jcenter()
        maven { url "https://github.com/oceanhub168pub/maven_repo/raw/master" }  // 这一句
    }
}
```

依赖添加
implementation(name: 'AndroidIjkplayer-1.0.0', ext: 'aar')

代码中使用
1.xml布局中使用
    <FrameLayout
        android:id="@+id/video_layout"
        android:layout_width="fill_parent"
        android:layout_height="200dp"
        android:background="@android:color/black">

        <com.hub168.videoplayer.fullvideo.IjkYhMediaController
            android:id="@+id/media_controller"
            android:layout_width="fill_parent"
            android:layout_height="fill_parent"
            app:yh_scalable="true" />

        <com.hub168.videoplayer.fullvideo.IjkYhVideoView
            android:id="@+id/videoView"
            android:layout_width="fill_parent"
            android:layout_height="fill_parent"
            android:layout_gravity="center"
            app:yh_autoRotation="true"
            app:yh_fitXY="false"
            app:yh_playType="2" />

    </FrameLayout>
    
    
2. 实现 IjkYhVideoView.VideoViewCallback 接口

        mVideoView = (IjkYhVideoView) findViewById(R.id.videoView);
        mMediaController = (IjkYhMediaController) findViewById(R.id.media_controller);
        mVideoView.setMediaController(mMediaController);
        mVideoView.setVideoPath(VIDEO_URL);
        mVideoView.start();
        
        
    @Override
    public void onBackPressed() {
    
        if (this.isFullscreen) {
            mVideoView.setFullscreen(false);
        } else {
            super.onBackPressed();
        }
    }
    
    
    @Override
    public void onDestory() {
    
         mVideoView.closePlayer();
    }


