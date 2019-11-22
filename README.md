# cordova-plugin-umeng-analytics
集成了官方最新版本的SDK v8.0.0,github上很多旨在封装此功能的plugin都不能正常工作,
请注意看此插件PGCommonSDK中对官方init方法的修改，加上了
```$xslt
MobclickAgent.setPageCollectionMode(MobclickAgent.PageMode.LEGACY_MANUAL);
```

##Requirements

 - iOS 7 or higher

##Installation

    cordova plugin add cordova-plugin-umeng-analytics  --variable UMENGKEYIOS=ios_app_key --variable UMENGKEYANDROID=android_app_key

##Attention
iOS:
按照下面这样修改AppDelegate.m 文件:
```

 #import "UMCommonModule.h" 
 #import <UMCommon/UMCommon.h> 
 #import <UMAnalytics/MobClick.h>

@implementation AppDelegate

- (BOOL)application:(UIApplication*)application didFinishLaunchingWithOptions:(NSDictionary*)launchOptions
{
    self.viewController = [[MainViewController alloc] init]; 
 [UMConfigure setLogEnabled:YES];
 [UMCommonModule initWithAppkey:@"具体key"channel:@"具体渠道"]; 
 [MobClick setScenarioType:E_UM_NORMAL];
    return [super application:application didFinishLaunchingWithOptions:launchOptions];
}  
@end
```

Android:
按照下面这样修改Cordova自动生成的MainActivity.java文件
```
 import com.umeng.analytics.MobclickAgent; 
 import com.umeng.commonsdk.UMConfigure; 
 import com.umeng.plugin.PGCommonSDK;

    @Override
        public void onResume() {
            super.onResume();
    
            MobclickAgent.onPageStart("Home");//Umeng建议写上，此字符串随便定义，与桥接js调用传过来的pageName无关
            MobclickAgent.onResume(this);
        }
    
        @Override
        protected void onPause() {
            super.onPause();
            MobclickAgent.onPageEnd("Home");//Umeng建议写上，此字符串随便定义，与桥接js调用传过来的pageName无关
            MobclickAgent.onPause(this);
        }

    @Override
    public void onCreate(Bundle savedInstanceState)
    {
        super.onCreate(savedInstanceState);

        // enable Cordova apps to be started in the background
        Bundle extras = getIntent().getExtras();
        if (extras != null && extras.getBoolean("cdvStartInBackground", false)) {
            moveTaskToBack(true);
        }

        // Set by <content src="index.html" /> in config.xml
        loadUrl(launchUrl);
        UMConfigure.setLogEnabled(true); 
        PGCommonSDK.init(this,"your key","your channel",UMConfigure.DEVICE_TYPE_PHONE,"");
    }

 ``` 
 ```
持续集成建议：
在Cordova-Android （Cordova-iOS）自动生成MainActivity (AppDelegate.m)文件后，用事先准备好的文件覆盖掉自动生成的即可
```
使用
```
页面访问路径：注意一定要成对调用，不能交叉，必须要onPageBegin A -> onPageEnd A 后才能开始下一个页面onPageBegin B -> onPageEnd B
if(AnalyticsAgent){
        AnalyticsAgent.onPageBegin(pageName,successCallbak,errorCallback);
}
if(AnalyticsAgent){
        AnalyticsAgent.onPageEnd(pageName,successCallbak,errorCallback);
    
}

自定义事件埋点：(注意要事先在Umeng官网上注册好事件，然后才能看到数据)
 if (AnalyticsAgent){
    AnalyticsAgent.onEventWithParameters(
        umEvent,
        {key1:value1, key2:value2}, 
    function () {
        console.log("custom event success"）;
    }, function (reason) {
        console.error("custom event failed: " + reason);
    });
}
```