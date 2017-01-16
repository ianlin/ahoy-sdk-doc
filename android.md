# Terminology

- **Main App**: 任何用 AHOY SDK 的 App

# 下載

# 設定

## Eclipse

- 將 `AhoySDKWrapperEclipse` 目錄放在任一地方，如 **`~/Documents/AhoySDKWrapperEclipse`** 
- 打開 Eclipse，import `AhoySDKWrapperEclipse` 目錄中所有 project
- 建議先 import `AhoySDKWrapper` **以外** 的 project，再 import `AhoySDKWrapper`
- import 時不要勾選 `Copy projects into workspace`
- 將 sample project `SampleAppEclipse` 的 build target 設定為 6.0 (API 23)

## Gradle

- 將 `ahoy-sdk-wrapper` 目錄放在任一地方，如 **`~/Documents/ahoy-sdk-wrapper`**

### settings.gradle

```gradle
include ':AhoySDKWrapper'
project(':AhoySDKWrapper').projectDir = new File('~/Documents/ahoy-sdk-wrapper')
```

### app/build.gradle

```gradle
allprojects {
    repositories {
        jcenter()
        flatDir {
            dirs 'libs', '~/Documents/ahoy-sdk-wrapper/libs'
        }
    }
}
```

```diff
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
        exclude group: 'com.android.support', module: 'support-annotations'
    })
    compile 'com.android.support:multidex:1.0.0'
    compile 'com.android.support:appcompat-v7:23.0.1'
    testCompile 'junit:junit:4.12'

+   compile project(':AhoySDKWrapper')
}
```

# API

## initAhoySDK

- **application**: Application
- **activity**: Activity
- **serviceType**: Integer
  - 0: 可接聽，可撥打
  - 1: 僅可撥打
  - 2: 僅可接聽
  - 3: 不可接聽，不可撥打
- **username**: String (Optional)
- **password**: String (Optional)

這個 API 需要在 `MainActivity` 的 `onCreate()` 去呼叫，理論上在此階段應該還沒抓到 DB 裡儲存的 Username/Password

所以這個 API 有兩個呼叫方式如下：

```java
public static void initAhoySDK(Application application, Activity activity, Integer serviceType)

public static void initAhoySDK(Application application, Activity activity, Integer serviceType, String username, String password)
```

## handleUpdateAhoyAccountInfo

- **username**: String
- **password**: String

AHOY 會利用參數提供的 Username/Password 進行登入驗證，登入結果會用 local broadcast 通知

```java
public static void handleUpdateAhoyAccountInfo(String username, String password)
```

## handleUpdateAhoyServiceType

- **serviceType**: int

更新 service type

```java
public static void handleUpdateAhoyServiceType(int serviceType)
```

## handlePresentAhoyView

開啟 AHOY Activity

```java
public static void handlePresentAhoyView()
```

## handleMessageReceived

- **application**: Application
- **bundle**: Bundle

處理 push notification，只會處理屬於 AHOY 的 push notification

```java
public static void handleMessageReceived(Application application, Bundle bundle)
```

## isAhoyPush

**Input**

- **bundle**: Bundle

**Output**: boolean

判斷是否為 AHOY 的 push notification

```java
public static boolean isAhoyPush(Bundle bundle)
```

## onDestroy

```java
public static void onDestroy()
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

AHOY 執行登入程序之後，會發送 broadcast 通知登入結果，所以在 App 裡面要註冊一個 broadcast receiver 來接收

**Note**: 使用 android support library 裡面提供的 [LocalBroadcastManager][3]，同一個 app 才能接收此 broadcast

- **Broadcast Intent Filter**
  - AhoySDK.AHOY_LOGIN_STATUS_NOTIFICATION
- **Event Result Code**
  - AhoySDK.AHOY_LOGIN_SUCCESS
  - AhoySDK.AHOY_LOGIN_NOT_RECEIVE_USERNAME_PASSWORD
  - AhoySDK.AHOY_LOGIN_FAILED
  - AhoySDK.AHOY_LOGIN_SERVER_ERROR
  - AhoySDK.AHOY_LOGIN_NETWORK_UNAVAILABLE

使用範例請參考底下的整合

# 整合

## Import AHOY SDK

- In `MainActivity.java`

```java
import com.yoha.ahoy.sdk.AhoySDK;
```

## 註冊 Local Broadcast Receiver 接收帳號登入驗證結果

- In `MainActivity.java`

### Import LocalBroadcastManager

```java
import android.support.v4.content.LocalBroadcastManager;
```

### 定義 `BroadcastReceiver`

```java
private BroadcastReceiver mAhoySDKLoginStatusReceiver = new BroadcastReceiver() {
    @Override
    public void onReceive(Context context, Intent intent) {

        String resultCode = intent.getStringExtra("resultCode");
        if (resultCode == null) {
            return;
        }

        // successfully login
        if (resultCode.equals(AhoySDK.AHOY_LOGIN_SUCCESS)) {
            Log.d(TAG, "MainActivity.AHOY_LOGIN_STATUS_NOTIFICATION AHOY_LOGIN_SUCCESS");

        // AHOY did not receive account info
        // this is probably when the app being launched for the first time
        // try to get the account info and call handleUpdateAhoyAccountInfo
        } else if (resultCode.equals(AhoySDK.AHOY_LOGIN_NOT_RECEIVE_USERNAME_PASSWORD)) {
            Log.d(TAG, "MainActivity.AHOY_LOGIN_STATUS_NOTIFICATION AHOY_LOGIN_NOT_RECEIVE_USERNAME_PASSWORD");

        // Log in fail to AHOY server
        // try to get the account info and call handleUpdateAhoyAccountInfo
        } else if (resultCode.equals(AhoySDK.AHOY_LOGIN_FAILED)) {
            Log.d(TAG, "MainActivity.AHOY_LOGIN_STATUS_NOTIFICATION AHOY_LOGIN_FAILED");
            //AhoySDK.handleUpdateAhoyAccountInfo(AHOY_USERNAME, AHOY_PASSWORD);

        // AHOY had received wrong username/password more than once
        } else if (resultCode.equals(AhoySDK.AHOY_LOGIN_RESOURCE_LOCKED)) {
            Log.d(TAG, "MainActivity.AHOY_LOGIN_STATUS_NOTIFICATION AHOY_LOGIN_RESOURCE_LOCKED");

        // AHOY server error
        } else if (resultCode.equals(AhoySDK.AHOY_LOGIN_SERVER_ERROR)) {
            Log.d(TAG, "MainActivity.AHOY_LOGIN_STATUS_NOTIFICATION AHOY_LOGIN_SERVER_ERROR");

        // network unavailable
        } else if (resultCode.equals(AhoySDK.AHOY_LOGIN_NETWORK_UNAVAILABLE)) {
        Log.d(TAG, "MainActivity.AHOY_LOGIN_STATUS_NOTIFICATION AHOY_LOGIN_NETWORK_UNAVAILABLE");
        }
    }
};
```

### 在 `onCreate()` 裡面加入

```java
IntentFilter intentFilter = new IntentFilter(AhoySDK.AHOY_LOGIN_STATUS_NOTIFICATION);
LocalBroadcastManager.getInstance(this).registerReceiver(mAhoySDKLoginStatusReceiver, intentFilter);
```

### 在 `onDestroy()` 裡面加入

```java
unregisterReceiver(mAhoySDKLoginStatusReceiver);
```

## Initialise AHOY SDK

- In `MainActivity.java`
- 在 `onCreate()` 裡面加入：

```java

// full-feature AHOY
AhoySDK.initAhoySDK(getApplication(), this, 0);

// outgoing call only
//AhoySDK.initAhoySDK(getApplication(), this, 1);

// incoming call only
//AhoySDK.initAhoySDK(getApplication(), this, 2);

// none-feature AHOY
//AhoySDK.initAhoySDK(getApplication(), this, 3);

// full-feature AHOY with account information
//AhoySDK.initAhoySDK(getApplication(), this, AHOY_USERNAME, AHOY_PASSWORD);

```

## Lifecycle

- 在 `onDestroy()` 裡面加入

```java
AhoySDK.onDestroy();
```

## 處理 Push Notification

- 在收到 push 的 receiver 裡 (e.g. `onMessageReceived()`) 裡面加入

```java
@Override
public void onMessageReceived(String from, Bundle data) {
    AhoySDK.handleMessageReceived(getApplication(), data);
}
```

[1]: https://drive.google.com/file/d/0BwrJzC4h0_-5eUxJa3l5MWpQd1E/view?usp=sharing
[2]: https://drive.google.com/file/d/0BwrJzC4h0_-5S0JzaC0yWGRUSEU/view?usp=sharing
[3]: https://developer.android.com/reference/android/support/v4/content/LocalBroadcastManager.html
