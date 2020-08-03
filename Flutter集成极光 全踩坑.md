[toc]

# Flutter集成极光 全踩坑

## 普通集成（非付费版本，走极光通道）

普通集成没有什么难度，直接走极光的plugin就可以了：

### 导入对应的包

[插件包地址](https://pub.flutter-io.cn/packages/jpush_flutter)

直接yaml依赖，get一波

### 上报RegisterID

```dart
  _initRegisterID(phoneModel) {
    jPush.getRegistrationID().then((rid) {
      if (rid != null && rid.isNotEmpty == true) {
        print('---->rid:${rid}');
        var params = {};
        params["registrationId"] = rid;
        params["phoneModel"] = Platform.isAndroid ? 'Android' : 'IOS';
        LoginRepository.reportRegisterId(params);
      }
    });
  }
```



也可以使用其他的方式，更常见的是设置别名之类的方式来推送。

### 处理对应的回调

```dart
  jPush.applyPushAuthority(
        NotificationSettingsIOS(sound: true, alert: true, badge: true));
```

```dart
    try {
      jPush.addEventHandler(
          onReceiveNotification: (Map<String, dynamic> message) async {
        print('---->onReceiveNotification:$message');
      }, onOpenNotification: (Map<String, dynamic> message) async {
        print('1111---->接收到推送:$message');
       /// do some things
      });
    } on Exception {
      print("---->获取平台版本失败");
    }
```



### 遇到的一些问题

这样基本就是已经集成了，但是也会有一些问题：

1. [ios点击不触发事件](https://github.com/jpush/jpush-flutter-plugin/issues/172) 
2. [ios杀死进程后获取不到extras](https://github.com/jpush/jpush-flutter-plugin/issues/163)
3. [更多问题欢迎来到issues围观](https://github.com/jpush/jpush-flutter-plugin/issues)

而且pub上的极光好像不是最新的，依赖了

```dar
  jpush_flutter:
    git:
      url: git://github.com/jpush/jpush-flutter-plugin.git
      ref: master
```

问题1得到了解决。

但是其他问题依然存在，且！！

<img src="/Users/ly/Library/Application Support/typora-user-images/image-20200723145056878.png" alt="image-20200723145056878" style="zoom:50%;" />



## 付费厂商通道集成



重要的事情说三次：

**plugin是基础免费版本，没有付费版本！！！！**

**plugin是基础免费版本，没有付费版本！！！！**

**plugin是基础免费版本，没有付费版本！！！！**

付费版本需要自己实现plugin，



### 基础集成

基础的厂商集成需要coder 在其中的Android项目中手动集成各个厂商的sdk：

```dart
//    小米极光推送sdk
    implementation 'cn.jiguang.sdk.plugin:xiaomi:3.6.8'
//    华为推送
    implementation 'com.huawei.hms:push:4.0.2.300'
    implementation 'cn.jiguang.sdk.plugin:huawei:3.6.8'
//    魅族推送
    implementation 'cn.jiguang.sdk.plugin:meizu:3.6.8'
//    vivo
    implementation 'cn.jiguang.sdk.plugin:vivo:3.6.8'
//    oppo
    implementation 'cn.jiguang.sdk.plugin:oppo:3.6.8'
    implementation fileTree(include: ['*.jar'], dir: 'libs')
```

基础集成的文档可以参考这部分：

[各个厂商集成文档](https://www.yuque.com/docs/share/307d6d68-0cc2-41b6-935b-4ca8c77c63d5)

### 插件集成

因为是走厂商渠道，所以集成了之后，大部分的推送消息都不会走上面的callback，除了像魅族这样没排面的，都集成了sdk还走普通通道（狗头），

极光是希望开发者实现自己的插件，在原生端接收到推送消息，桥接到futter端进行消费：



1. 极光发送推送消息
2. 中转ac被拉起，拿到对应的推送字段
3. 跳转到mainActivity，并把数据传递过去，MainActivity拿到数据通过MethodChannel传递到flutter端
4. flutter端进行消费处理。

#### OpenClickActivity

```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
//        mTextView = new TextView(this);
//        setContentView(mTextView);
//        handleOpenClick();
        getWindow().addFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS);
        Intent intent = new Intent(this, MainActivity.class);
        intent.setData(getIntent().getData());
        if (getIntent().getExtras() != null) {
            intent.putExtras(getIntent().getExtras());
        }
        startActivity(intent);
        finish();
    }
```

#### MainActivity

```DART
private void handleOpenClick(MethodChannel channel, Intent intent) {

        String data = null;
        //获取华为平台附带的jpush信息
        if (intent.getData() != null) {
                data = intent.getData().toString();
        }

        //获取fcm、oppo、vivo、华硕、小米平台附带的jpush信息
        if(TextUtils.isEmpty(data) && intent.getExtras() != null){
            data = intent.getExtras().getString("JMessageExtra");
        }

        if (TextUtils.isEmpty(data)) return;
        try {
            JSONObject jsonObject = new JSONObject(data);
            String msgId = jsonObject.optString(KEY_MSGID);
            byte whichPushSDK = (byte) jsonObject.optInt(KEY_WHICH_PUSH_SDK);
            String title = jsonObject.optString(KEY_TITLE);
            String content = jsonObject.optString(KEY_CONTENT);
            String extras = jsonObject.optString(KEY_EXTRAS);

            final Handler mHandler = new Handler();
            Runnable r = () -> {
                //do something
                Map<String, String> map = new HashMap<>();
                map.put("msgId", msgId);
                map.put("whichPushSDK", whichPushSDK + "");
                map.put("title", title);
                map.put("content", content);
                map.put("extras", extras);
//                Toast.makeText(MainActivity.this, map.toString(), Toast.LENGTH_SHORT).show();
                channel.invokeMethod("notifyclick", map);
            };
            //主线程中调用：
            mHandler.postDelayed(r, 100);//延时100毫秒
        } catch (JSONException e) {
            Log.w(TAG, "parse notification error");
        }
    }
```

#### Flutter

```dart
 Future<Null> _handleMethod(MethodCall call) async {
    print("_handleMethod: ${call.method}");

    switch (call.method) {
      case "notifyclick":
        return notifyclick(call.arguments.cast<String, dynamic>());

      default:
        throw new UnsupportedError("Unrecognized Event");
    }
  }

  Future<dynamic> notifyclick(Map<String, dynamic> message) async {
    print('1111---->接收到推送:$message');
    String pushType;
    String afterSalesId;
    String appMessageId;
    String extra=message["extras"];
    print(extra);
    JpushModel jpushModel=JpushModel.fromJson(jsonDecode(extra));

    pushType=jpushModel?.pushType;
    afterSalesId=jpushModel?.afterSalesId;
    appMessageId=jpushModel?.appMessageId;
    if (pushType == "stationMessage") {
      print('----------appMessageId= $appMessageId');
      Navigator.push(
          context,
          MaterialPageRoute(
              builder: (context) => MessageDetailPage(
                    messageCellList:
                        MessageCellList(appMessageId: int.parse(appMessageId)),
                  )));
    } else {
//      if (Platform.isAndroid) {
        String _jumpUrl =
            JPushConf.getJumpUrlByJpush(pushType, afterSalesId: afterSalesId);
        WechatUtils().launchMiniPro(_jumpUrl);
//      } else {
//        WechatUtils().launchMiniPro('${WechatMiniUrl.choose_url}');
//      }
    }

```

#### AndroidManifest

```xml
   <activity android:name=".OpenClickActivity"
            android:theme="@android:style/Theme.Translucent.NoTitleBar"
            android:launchMode="singleTask"
            android:screenOrientation="portrait"
            android:exported="true">
            <intent-filter>
                <data android:path="/ypath" android:host="yhost" android:scheme="yscheme"></data>
                <action android:name="android.intent.action.VIEW"/>
                <category android:name="android.intent.category.DEFAULT" />
            </intent-filter>

        </activity>

        <!-- Don't delete the meta-data below.
             This is used by the Flutter tool to generate GeneratedPluginRegistrant.java -->
        <meta-data
            android:name="flutterEmbedding"
            android:value="2" />
```



### 需要注意的点

1. 中转ac的问题
   如果中转ac配置出错的话，点击通知可以拉起app，但是不会获取到extras，这个很恶心，对于中转ac在AndroidManifest中的配置，千万不要配错，不然很容易拉起了拿不到数据。

2. 需要后台配合

   之前版本的服务器sdk并没有提供*uri_activity*这个配置，如果发现没有这个字段，更新下版本：<img src="/Users/ly/Library/Application Support/typora-user-images/image-20200723153625161.png" alt="image-20200723153625161" style="zoom:50%;" />



到这里基本都完成了，华为小米即便被杀进程了，推送消息也会跟着厂商通道直达，点击也可以响应到各个页面。