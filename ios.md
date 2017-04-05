# 安裝

- 將 `AhoySDK.framework` 和 `WebRTC.framework` 新增至 Project root 底下，新增時請勾選 **Copy items if need**
- 將 `AhoySDK.framework` 和 `WebRTC.framework` 放在 **General** 底下的 **Embedded Binaries** and **Linked Frameworks and Libraries**
- 將 `incallmanager_ringtone.mp3`, `incallmanager_busytone.mp3` 和 `incallmanager_ringback.mp3` 新增至 Project root 底下，新增時請勾選 **Copy items if need**

# 設定

## Build Settings

- Swift support
  - (Xcode < 8 ) **Embedded Content Contains Swift Code** = **Yes**
  - (Xcode >= 8) **ALWAYS_EMBED_SWIFT_STANDARD_LIBRARIES** = **Yes**

- **Enable Bitcode** = **No**

## Build Phases

- 確認 `AhoySDK.framework` 和 `WebRTC.framework` 在 **Link Binary With Libraries** 以及 **Embed Frameworks** 底下
- 確認 `incallmanager_ringtone.mp3`, `incallmanager_busytone.mp3` 和 `incallmanager_ringback.mp3` 在 **Copy Bundle Resources** 底下

## Push Notifications

- 點選 **Capabilities**
- 將 **Push Notifications** 開啟為 **On**
- 將 **Background Modes** 底下的 **Voice over IP** 打勾

## Info.plist

- 將以下資訊加入 `Info.plist` 裡面，必須在最外層的 `<dict></dict>` 之間

### Privacy Description (since iOS 10, see [this](https://developer.apple.com/library/content/qa/qa1937/_index.html))

**Note**: Description 內容可以自訂

```xml
<key>NSContactsUsageDescription</key>
<string>Used to find your friends</string>
<key>NSMicrophoneUsageDescription</key>
<string>Used to communicate with others</string>
<key>NSCameraUsageDescription</key>
<string>Used to make video chats with others</string>
```

### Code Push Deployment Key

```xml
<key>CodePushDeploymentKey</key>
<string>ENTER_THE_KEY_PROVIDED_BY_US</string>
```

# API

## initAhoySDK

```obj-c
- (void)initAhoySDK:(UIApplication *)application
      launchOptions:(NSDictionary *)launchOptions
        serviceType:(int)serviceType

- (void)initAhoySDK:(UIApplication *)application
      launchOptions:(NSDictionary *)launchOptions
        serviceType:(int)serviceType
           username:(NSString *)username
           password:(NSString *)password
```

- **application**: UIApplication
- **launchOptions**: NSDictionary
- **serviceType**: int
  - 0: 可接聽，可撥打
  - 1: 僅可撥打
  - 2: 僅可接聽
  - 3: 不可接聽，不可撥打
- **username**: NSString (Optional)
- **password**: NSString (Optional)

#### Example

```obj-c

// full-feature AHOY
[[AhoySDKAppDelegate sharedInstance] initAhoySDK:application
                                   launchOptions:launchOptions
                                     serviceType:0];

// outgoing call only
[[AhoySDKAppDelegate sharedInstance] initAhoySDK:application
                                   launchOptions:launchOptions
                                     serviceType:1];

// incoming call only
[[AhoySDKAppDelegate sharedInstance] initAhoySDK:application
                                   launchOptions:launchOptions
                                     serviceType:2];

// non-feature AHOY
[[AhoySDKAppDelegate sharedInstance] initAhoySDK:application
                                   launchOptions:launchOptions
                                     serviceType:3];

// full-feature AHOY with account info
[[AhoySDKAppDelegate sharedInstance] initAhoySDK:application
                                   launchOptions:launchOptions
                                     serviceType:0
                                        username:@"xxx"
                                        password:@"xxx"];
```

## handleUpdateAhoyAccountInfo

AHOY 會利用參數提供的 Username/Password 進行登入驗證，登入結果會用 NSNotificationCenter 通知

```obj-c
+ (void)handleUpdateAhoyAccountInfo:(NSString *)username password:(NSString *)password
```

- **username**: NSString
- **password**: NSString

#### Example

```obj-c
[AhoySDKAppDelegate handleUpdateAhoyAccountInfo:@"886xxxxxxxxx" password:@"PASSWORD"];
```

## handleUpdateAhoyServiceType

更新 service type

```obj-c
+ (void)handleUpdateAhoyServiceType:(int)servicetype
```

- **serviceType**: int

#### Example

```obj-c
// update service type to incoming call only
[AhoySDKAppDelegate handleUpdateAhoyServiceType:2];
```

## handlePresentAhoyViewController

開啟 AHOY view

```obj-c
+ (void)handlePresentAhoyViewController
```

#### Example

```obj-c
[AhoySDKAppDelegate handlePresentAhoyViewController];
```

# Constants

所有 constants 的 type 都是 string

- **AHOY_LOGIN_STATUS_NOTIFICATION**
- **AHOY_LOGIN_SUCCESS**
- **AHOY_LOGIN_NOT_RECEIVE_USERNAME_PASSWORD**
- **AHOY_LOGIN_FAILED**
- **AHOY_LOGIN_RESOURCE_LOCKED**
- **AHOY_LOGIN_SERVER_ERROR**
- **AHOY_LOGIN_NETWORK_UNAVAILABLE**

# LogIn Status Events

AHOY 執行登入程序之後，會利用 NSNotificationCenter 通知登入結果，所以在 App 裡面要註冊一個 observer 來接收

- **NSNotification Name**
  - AHOY_LOGIN_STATUS_NOTIFICATION
- **Event Result Code**
  - AHOY_LOGIN_SUCCESS
  - AHOY_LOGIN_NOT_RECEIVE_USERNAME_PASSWORD
  - AHOY_LOGIN_FAILED
  - AHOY_LOGIN_SERVER_ERROR
  - AHOY_LOGIN_NETWORK_UNAVAILABLE

使用範例請參考底下的整合

# 整合

## AppDelegate.m

### Import required libraries

```obj-c
#import <PushKit/PushKit.h>
#import <AhoySDK/AhoySDK.h>
```

### Initialise AHOY SDK

在 `didFinishLaunchingWithOptions` method 裡面加入

```obj-c
// full-feature AHOY
[[AhoySDKAppDelegate sharedInstance] initAhoySDK:application
                                   launchOptions:launchOptions
                                     serviceType:0];

// outgoing call only
[[AhoySDKAppDelegate sharedInstance] initAhoySDK:application
                                   launchOptions:launchOptions
                                     serviceType:1];

// incoming call only
[[AhoySDKAppDelegate sharedInstance] initAhoySDK:application
                                   launchOptions:launchOptions
                                     serviceType:2];

// non-feature AHOY
[[AhoySDKAppDelegate sharedInstance] initAhoySDK:application
                                   launchOptions:launchOptions
                                     serviceType:3];

// full-feature AHOY with account info
[[AhoySDKAppDelegate sharedInstance] initAhoySDK:application
                                   launchOptions:launchOptions
                                     serviceType:0
                                        username:@"xxx"
                                        password:@"xxx"];
```

### 註冊 Notification Center Observer 接收帳號登入驗證結果

- **Notification Name**: constants `AHOY_LOGIN_STATUS_NOTIFICATION`
- 在 `didFinishLaunchingWithOptions` method 裡面加入

```obj-c
[[NSNotificationCenter defaultCenter] addObserver:self
                                         selector:@selector(handleAhoyLoginStatus:)
                                             name:AHOY_LOGIN_STATUS_NOTIFICATION
                                           object:nil];
```

- 新增 **handleAhoyLogInStatus** method

```obj-c
- (void)handleAhoyLoginStatus:(NSNotification *)notification {
    NSDictionary *userInfo = notification.userInfo;

    // successfully login
    if ([userInfo[@"resultCode"] isEqualToString:AHOY_LOGIN_SUCCESS]) {
        NSLog(@"TestApp.handleAhoyLoginStatus: AHOY_LOGIN_SUCCESS");

    // AHOY did not receive account info
    // this is probably when the app being launched for the first time
    // try to get the account info and call handleUpdateAhoyAccountInfo
    } else if ([userInfo[@"resultCode"] isEqualToString:AHOY_LOGIN_NOT_RECEIVE_USERNAME_PASSWORD]) {
        NSLog(@"TestApp.handleAhoyLoginStatus: AHOY_LOGIN_NOT_RECEIVE_USERNAME_PASSWORD");

    // Log in fail to AHOY server
    // try to get the account info and call handleUpdateAhoyAccountInfo
    } else if ([userInfo[@"resultCode"] isEqualToString:AHOY_LOGIN_FAILED]) {
        NSLog(@"TestApp.handleAhoyLoginStatus: AHOY_LOGIN_FAILED");
        //[AhoySDKAppDelegate handleUpdateAhoyAccountInfo:@"886xxxxxxxxx" password:@"PASSWORD"];

    // AHOY had received wrong username/password more than once
    } else if ([userInfo[@"resultCode"] isEqualToString:AHOY_LOGIN_RESOURCE_LOCKED]) {
        NSLog(@"TestApp.handleAhoyLoginStatus: AHOY_LOGIN_RESOURCE_LOCKED");

    // AHOY server error
    } else if ([userInfo[@"resultCode"] isEqualToString:AHOY_LOGIN_SERVER_ERROR]) {
        NSLog(@"TestApp.handleAhoyLoginStatus: AHOY_LOGIN_SERVER_ERROR");

    // network unavailable
    } else if ([userInfo[@"resultCode"] isEqualToString:AHOY_LOGIN_NETWORK_UNAVAILABLE]) {
        NSLog(@"TestApp.handleAhoyLoginStatus: AHOY_LOGIN_NETWORK_UNAVAILABLE");
    }
}
```


### UIApplication Delegate Methods

```obj-c

// for CallKit
- (BOOL)application:(UIApplication *)application
continueUserActivity:(NSUserActivity *)userActivity
 restorationHandler:(void(^)(NSArray * __nullable restorableObjects))restorationHandler
{
    return [[AhoySDKAppDelegate sharedInstance] application:application
                                       continueUserActivity:userActivity
                                         restorationHandler:restorationHandler];
}

```

### Push Notification

```obj-c

// Required to register for notifications
- (void)application:(UIApplication *)application didRegisterUserNotificationSettings:(UIUserNotificationSettings *)notificationSettings
{
    [[AhoySDKAppDelegate sharedInstance] application:application
                 didRegisterUserNotificationSettings:notificationSettings];
}

// Required for the register event.
- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken
{
    [[AhoySDKAppDelegate sharedInstance] application:application
    didRegisterForRemoteNotificationsWithDeviceToken:deviceToken];
}

// Required for the notification event.
- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)notification
{
    [[AhoySDKAppDelegate sharedInstance] application:application
                        didReceiveRemoteNotification:notification];
}

// Required for the localNotification event.
- (void)application:(UIApplication *)application didReceiveLocalNotification:(UILocalNotification *)notification
{
    [[AhoySDKAppDelegate sharedInstance] application:application
                         didReceiveLocalNotification:notification];
}
```

### VoIP Push Notification (PushKit)

```obj-c

//--- Add PushKit delegate method

// Handle updated push credentials
- (void)pushRegistry:(PKPushRegistry *)registry didUpdatePushCredentials:(PKPushCredentials *)credentials forType:(NSString *)type
{
    // Register VoIP push token (a property of PKPushCredentials) with server
    [[AhoySDKAppDelegate sharedInstance] pushRegistry:registry
                             didUpdatePushCredentials:credentials
                                              forType:(NSString *)type];
}

// Handle incoming pushes
- (void)pushRegistry:(PKPushRegistry *)registry didReceiveIncomingPushWithPayload:(PKPushPayload *)payload forType:(NSString *)type
{
    // Process the received push
    [[AhoySDKAppDelegate sharedInstance] pushRegistry:registry
                    didReceiveIncomingPushWithPayload:payload
                                              forType:(NSString *)type];
}

```
