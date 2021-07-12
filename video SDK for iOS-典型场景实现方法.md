# Video SDK for iOS典型场景实现方法

## 实现视频通话(内部采集、内部渲染) ##

### 相关接口
API的调用可使用“[YMVoiceService getInstance]”来直接操作，接口使用的基本流程为`初始化`->`收到初始化成功回调通知`->`加入房间`->`收到加入房间成功回调通知`->`使用其它接口`->`离开房间`->`反初始化`，要确保严格按照上述的顺序使用接口。

#### 初始化流程
`setLogLevel` 设置日志级别
`init` 引擎初始化

#### 加入房间流程
`setVideoPreviewFps` 设置本地预览帧率，建议设置为 30
`setVideoLocalResolution` 设置本地采集分辨率，建议设置为 480 * 640
`setVideoFps` 设置视频帧率（大流），建议设置为 10 - 15
`setVideoNetResolutionWidth` 设置编码分辨率（大流），建议设置，建议设置480 * 640
`setVBR` 设置视频编码是否采用VBR动态码率（大流），建议设置为 true
`setVideoCodeBitrate` 设置视频编码码率，若不调用则采用默认设置（大流）），建议设置为 600 - 160
`setVideoFpsForSecond` 设置视频帧率（小流），建议设置为 7
`setVideoNetResolutionWidthForSecond` 设置编码分辨率（小流），建议设置为 256 * 336 （16的整数被）
`setVBRForSecond` 设置视频编码是否采用VBR动态码率（小流），建议设置为true
`setVideoCodeBitrateForSecond` 设置视频编码码率，若不调用则采用默认设置（小流），200 - 100
`setVideoFpsForShare` 设置共享流帧率，一般共享建议设置为 10帧
`setVideoNetResolutionWidthForShare` 设置共享流编码分辨率，建议为 720p - 1080p
`setMicLevelCallback` 设置是否开启讲话音量回调
`setFarendVoiceLevelCallback` 设置是否开启远端说话人音量回调
`setAVStatisticInterval` 设置音视频统计上报间隔，建议为5秒
`setAutoSendStatus` 设置状态同步(mic,speaker)

`joinChannelSingleMode` 加入房间，建议 role建议设置为1，autoRecv设置为false


#### 本地设备控制接口
`startCapture` 打开摄像头采集
`createRender` 创建本端预览
`stopCapture` 关闭摄像头采集
`setMicrophoneMute`	设置麦克风状态
`setSpeakerMute` 设置扬声器状态
`switchCamera` 切换摄像头

#### 接收远端音视频
`createRender` 创建渲染
`deleteRender` 删除渲染
`setUsersVideoInfo` 设置观看用户的哪一路流(大小流)，autoRecv设置为false时，需要调用该接口后才会接收对方的视频流
`maskVideoByUserId` 设置是否屏蔽他人视频，屏蔽对方的所有视频流，包括共享流，音频流保留
`setListenOtherVoice` 设置是否听其他人语音
`onMemberChange` 房间内其它用户加入/离开回调通知
`YOUME_EVENT_OTHERS_VIDEO_INPUT_START` onEvent 事件，远端开启摄像头，可用于显示摄像头状态
`YOUME_EVENT_OTHERS_VIDEO_INPUT_STOP` onEvent 事件，远端关闭摄像头，可用于显示摄像头状态
`YOUME_EVENT_OTHERS_SHARE_INPUT_START` onEvent 事件，远端开启共享流，可用于显示共享画面
`YOUME_EVENT_OTHERS_SHARE_INPUT_STOP` onEvent 事件，远端关闭共享流，可用于移除共享画面

#### 离开房间
`leaveChannelAll` 退出所有房间

#### 反初始化
`unInit` 反初始化引擎

### 关键调用顺序
1. 初始化（init）, 等待初始化成功的通知事件`YOUME_EVENT_INIT_OK` 

2. 房间参数设置(视频参数、共享参数、QOS统计间隔等)

3. 加入房间（joinChannelSingleMode），等待加入房间成功的通知事件 `YOUME_EVENT_JOIN_OK`

4. 打开摄像头（startCapture），设置麦克风扬声器（setMicrophoneMute，setSpeakerMute）

5. 创建本端预览，同时接收到视频数据回调事件 `YOUME_EVENT_OTHERS_VIDEO_ON`，创建远端视频渲染（createRender）

6. 其它接口(开关摄像头、开关mic/speaker，屏蔽他人语音/视频)

7. 退出房间

8. 反初始化


### 实现回调

使用者要遵守协议 `VoiceEngineCallback` 并实现相关函数（回调函数）。回调都在子线程中执行，不能用于更新UI等耗时操作。

*  首先要遵守协议`VoiceEngineCallback`来注册回调事件：

![注册](https://youme.im/doc/images/talk_ios_register.jpg)


* 然后具体实现回调方法：

``` objectivec
   -(void) onYouMeEvent:(YouMeEvent_t)eventType errcode:(YouMeErrorCode_t)iErrorCode roomid:(NSString *)roomid param:(NSString *)param {
    switch (eventType)
    {
            //case案例只覆盖了部分，仅供参考，详情请查询枚举类型YouMeEvent
        case YOUME_EVENT_INIT_OK:
            //"初始化成功";
            break;
        case YOUME_EVENT_INIT_FAILED:
            // "初始化失败，错误码：" + errorCode;   
            break;
        case YOUME_EVENT_JOIN_OK:
            //"加入频道成功";
            break;
        case YOUME_EVENT_LEAVED_ALL:
            // "离开频道成功"
            break;
        case YOUME_EVENT_JOIN_FAILED:
            // 进入频道失败
       		 break;
        case YOUME_EVENT_REC_PERMISSION_STATUS:
            // "通知录音权限状态，成功获取权限时错误码为YOUME_SUCCESS，获取失败为YOUME_ERROR_REC_NO_PERMISSION（此时不管麦克风mute状态如何，都没有声音输出）";
            break;
        case YOUME_EVENT_RECONNECTING:
            //"断网了，正在重连";
            break;
        case YOUME_EVENT_RECONNECTED:
            // "断网重连成功";
            break;
        case  YOUME_EVENT_OTHERS_MIC_OFF:
            //其他用户的麦克风关闭：
            break;
        case YOUME_EVENT_OTHERS_MIC_ON:
            //其他用户的麦克风打开：
            break;
        case YOUME_EVENT_OTHERS_SPEAKER_ON:
            //其他用户的扬声器打开：
            break;
        case YOUME_EVENT_OTHERS_SPEAKER_OFF:
            //其他用户的扬声器关闭
            break;
        case YOUME_EVENT_OTHERS_VOICE_ON:
            //其他用户开始讲话
            break;
        case YOUME_EVENT_OTHERS_VOICE_OFF:
            //其他用户停止讲话
            break;
        case YOUME_EVENT_MY_MIC_LEVEL:
            //麦克风的语音级别，值把iErrorCode转为整形即是音量值
            break;
        case YOUME_EVENT_MIC_CTR_ON:
            //麦克风被其他用户打开
            break;
        case YOUME_EVENT_MIC_CTR_OFF:
            //麦克风被其他用户关闭
            break;
        case YOUME_EVENT_SPEAKER_CTR_ON:
            //扬声器被其他用户打开
            break;
        case YOUME_EVENT_SPEAKER_CTR_OFF:
            //扬声器被其他用户关闭
            break;
        case YOUME_EVENT_LISTEN_OTHER_ON:
            //取消屏蔽某人语音
            break;
        case YOUME_EVENT_LISTEN_OTHER_OFF:
            //屏蔽某人语音
            break;
        default:
            //"事件类型" + eventType + ",错误码" +errcode
            break;
    }
}

// pcm数据回调
- (void)onPcmDataRemote: (int)channelNum samplingRateHz:(int)samplingRateHz bytesPerSample:(int)bytesPerSample data:(void*) data dataSizeInByte:(int)dataSizeInByte {
}

- (void)onPcmDataRecord: (int)channelNum samplingRateHz:(int)samplingRateHz bytesPerSample:(int)bytesPerSample data:(void*) data dataSizeInByte:(int)dataSizeInByte {
}

- (void)onPcmDataMix: (int)channelNum samplingRateHz:(int)samplingRateHz bytesPerSample:(int)bytesPerSample data:(void*) data dataSizeInByte:(int)dataSizeInByte {
}

//RestAPI回调
-(void) onRequestRestAPI: (int)requestID iErrorCode:(YouMeErrorCode_t) iErrorCode  query:(NSString*) strQuery  result:(NSString*) strResult {
}

//获取频道用户列表回调
-(void) onMemberChange:(NSString*) channelID changeList:(NSArray*) changeList { 
}

//SDK内置连麦抢麦接口对应的回调
- (void) onBroadcast:(YouMeBroadcast_t)bc strChannelID:(NSString*)channelID strParam1:(NSString*)param1 strParam2:(NSString*)param2 strContent:(NSString*)content;

//音视频通话码率、丢包率回调
- (void) onAVStatistic:(YouMeAVStatisticType_t)type  userID:(NSString*)userID  value:(int) value ;

//视频h264裸数据回调
- (void)onVideoPreDecodeDataForUser:(const char *)userId data:(const void*)data len:(int)dataSizeInByte 
{
}
```

## 实现视频直播 ##

### 相关接口

`init` 引擎初始化
`setVideoLocalResolution` 设置本地采集分辨率
`setVideoNetResolution` 设置网络传输分辨率
`joinChannelSingleMode` 加入房间
`createRender` 创建渲染
`startCapture` 开始摄像头采集
`setMicrophoneMute` 设置麦克风状态
`setSpeakerMute`    设置扬声器状态
`setHeadsetMonitorOn` 设置监听
`setBackgroundMusicVolume`  设置背景音乐播放音量
`playBackgroundMusic`   播放背景音乐

### 关键调用顺序
1. 以主播身份进入频道 初始化（init）->joinChannelSingleMode(参数三传主播身份YOUME_USER_HOST), 观众传的身份可以是听众 也可以是自由人（住进自由人进入频道 需要关闭麦克风）

2. 主播打开摄像头，麦克风等设备

3. 设置监听 setHeadsetMonitorOn（true,true）参数1表示是否监听麦克风 true表示监听,false表示不监听 ,参数2表示是否监听背景音乐,true表示监听,false表示不监听

4. setBackgroundMusicVolume（70）调节背景音量大小

5. playBackgroundMusic(string pFilePath, bool bRepeat) 播放本地的mp3音乐。 参数一 本地音乐路径， 参数二 是否重复播放 true重复，false不重复

6. 远端有视频流过来，会通知 `YOUME_EVENT_OTHERS_VIDEO_ON` 事件,此时调用createRender创建相关渲染。

