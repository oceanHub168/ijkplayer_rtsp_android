# ijkplayer_rtsp_android
优化后ijkplayer播放器支持rtsp拉流、低延迟


## 编译ijkplayer ,支持rtsp拉流、低延迟
https://www.jianshu.com/p/b48e58fa9694?tdsourcetag=s_pctim_aiomsg


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

## 项目中使用ijkplayer 


1.将AndroidIjkplayer-1.0.0.aar 放到app--->libs 目录下
2.在app的 build.gradle 中dependencies节点前单独添加
repositories {
    flatDir {
        dirs 'libs'
    }
}

依赖添加
implementation(name: 'AndroidIjkplayer-1.0.0.aar', ext: 'aar')

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


