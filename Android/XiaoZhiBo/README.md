# 小直播支持自定义采集和处理

#### 本地视频采集及观众观看：

###### 通过 camera2 的 ImageReader 获取相机采集到的实时视频帧数据，格式为ImageFormat.YUV_420_888，个别手机为422采集率，只解决了三星S8的采集率转换问题：

视频数据采集及上传，本地预览，观众观看视频，封装为如下module

主要使用的是LivePlayView以及LivePushView

[liveKit](https://github.com/desfate/liveKit)



#### 腾讯小直播需要修改的部分

<u>代码里都用fixme注释了，可以直接搜索</u>

1）manifest.xml 的 TCCameraAnchorActivity（主播）， TCAudienceActivity（观众）

`android:configChanges="keyboardHidden|orientation"`

横屏切换时不重置生命周期

2）主播相关

```java
// ！！！ 保证推流前这两个属性必须设置
TXLivePushConfig config = mTXLivePusher.getConfig();
config.setCustomModeType(TXLiveConstants.CUSTOM_MODE_VIDEO_CAPTURE);    //使用自定义视频采集
 config.setVideoResolution(TXLiveConstants.VIDEO_RESOLUTION_TYPE_1920_1080);  //设置推流视频分辨率

// 原有的这个要去掉
mTXLivePusher.startCameraPreview(view)
```

3）观众相关

```java
 // 这里可以拿到视频宽高
 mTXLivePlayer.setPlayListener(new ITXLivePlayListener() {
            @Override
            public void onPlayEvent(final int event, final Bundle param) {
                if (event == TXLiveConstants.PLAY_EVT_CHANGE_RESOLUTION) {
                    int width = param.getInt(TXLiveConstants.EVT_PARAM1, 0);
                    int height = param.getInt(TXLiveConstants.EVT_PARAM2, 0);
                    // 这里可以获得 拿到的视频大小
                    // 视频推流过来时的 视频宽高  这里要根据视频宽高生成对应的显示页面
                    // fixme：这里记录拿到的视频宽高
                    playWidth = width;
                    playHeight = height;                
                }
            }
 		}
 }
                               
 //setPlayerView必须传null
 mTXLivePlayer.setPlayerView(null);
 mTXLivePlayer.setSurface(view.getmSurface());  // 绑定自己的surface
                          
```

#### 配置修改

```
minSdkVersion = 24
targetSdkVersion = 24
// 因为module使用的时camera2 所以最低要24
implementation 'com.github.desfate:liveKit:0.76'
// 导入module即可
```