Aimazing SDK Documentation (v1.1)
===

###### tags: `aimazing`, `sdkdoc`, `release`, `dash`

---
Table of Contents
===

[TOC]

---
ChangeLog
===

#### v1.1 (15th August 2017)
+ Aimazing SDK supports iOS7.
+ NETWORK_ERROR case added into UtilityError interface.
+ isCalibrated() method added into Utility class.

---
User Experience
===

![figure1](https://i.imgur.com/AwSxkSd.png)

1. User launches Dash application(Dash APP) and taps the payment button.
2. After tapping the payment button, user takes mobile device to approach Dash speaker terminal. `Recognizer` instance should be initialized and execute `start()` in order to receive sound wave data from Dash speaker terminal.
3. `Recognizer` instance received counter code from Dash speaker terminal.

---
Prerequisites
===

1. Aimazing SDK is using device’s microphone, it requires permission of microphone.
2. `Utility` instance should be initialized and executes `calibration()` when user launches Dash APP(Aimazing SDK installed version) at the **first time launching** and **after OS update**. It is to be sure that the device fully supports Aimazing SDK. The calibration process takes at most **8 seconds**.
3. Aimazing SDK has to be activated before receiving sound waves from Dash speaker terminal. SDK activation process will be initialized when `calibration()` in `Utility` class is called, it is automatically initialized by Aimazing SDK itself, no additional api call is needed for Dash App.

---
Sequence Diagram
===

## SDK Activation
```sequence
Dash App->Aimazing SDK: calibration()
Note over Aimazing SDK: Calibration\n& Activation
Aimazing SDK->Aimazing Cloud: Device Unique ID, App ID
Aimazing Cloud-->Aimazing SDK: 200 OK
Aimazing SDK-->Dash App: onSuccess()
```
The SDK activation process will be automatically initialized during calibration process. Aimazing SDK sends SDK activation HTTP request to complete the activation process.

## Transaction Tracking
```sequence
Dash App->Aimazing SDK: recognizer.start()
Aimazing SDK->Dash App: onSuccess(Counter Code)
Dash App->Dash Backend: Counter Code
Aimazing SDK->Aimazing Cloud: Counter Code
Note over Aimazing Cloud: Record\nTracking
Aimazing Cloud-->Aimazing SDK: 200 OK
```
Aimazing needs to track the usage of Aimazing SDK, thus a transaction tracking procedure is proposed. Aimazing SDK will send SDK usage tracking HTTP request after successfully receiving counter code from Dash speaker terminal. Current Dash App transaction flow will not be affected because transaction tracking is working in the background and running automatically during transaction.

---
Aimazing SDK
===

## Features
Recognizer
- Receive & processing sound waves data.
- Headset plugged detection.

Utility
- Calibration utility ensures the device’s mic/speaker is supported.
- Check required permissions are granted.
- Request required permissions.

## Requirements
Hardware
- Microphone can receives 16kHz ~ 20kHz.
- Speaker can produces 16kHz ~ 20kHz.

Software
- Android Jelly Bean (4.1.x) or higher.
- iOS 7.0 or higher.

---
Android SDK Usage
===

## Installation
**Put `aimazinglib-dash.aar` into `app/libs` directory**
![](https://i.imgur.com/s2vursV.png)

**Modify `build.gradle`**
Add the following lines into `build.gradle`.
```=
repositories {
  flatDir {
    dirs 'libs'
  }
}

dependencies {
  // ...
  compile(name: 'aimazinglib-dash', ext: 'aar')
  // ...
}
```

**Modify `AndroidManifest.xml`**
Add the following lines into `AndroidManifest.xml`.

```xml=
<uses-permission android:name="android.permission.MODIFY_AUDIO_SETTINGS"/>
<uses-permission android:name="android.permission.RECORD_AUDIO"/>
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
```

## Classes

### Recognizer

#### 1. Construct
Construct Recognizer instance to start using Aimazing SDK to receive data.
``` pseudo
new Recognizer(Activity activity);
```

##### Arguments:
| Type | Description |
| -------- | -------- |
| `activity` | Android activity instance |

##### Examples:
``` java=
Recognizer recognizer = new Recognizer(MainActivity.this);
```

#### 2. Start Receiving
Start receiving data. The result will be sent through callback functions. Please refer to the description of `RecognizerCallback` for further details.
``` pseudo
void start(recognizerCallback);
```

##### Arguments:
| Type | Description |
| -------- | -------- |
| `recognizerCallback` | RecognizerCallback interface |

##### Example:
``` java=
Recognizer recognizer = new Recognizer(MainActivity.this);

recognizer.start(new RecognizerCallback() { 
  @Override
  public void onSuccess(final String counterCode) {
    ReceiveActivity.this.runOnUiThread(new Runnable() {
      @Override
      public void run() {
        // Do something while getting counter code.
      }
    });
  }

  @Override
  public void onError(RecognizerError recognizerError) {
    switch (recognizerError) {
      case NOT_ACTIVATED:
        // SDK is not activated.
        break;
      case PERMISSION_DENIED:
        // Device's permission not granted.
        break;
      case HEADSET_PLUGGED:
        // Headset is plug into device.
        break;
    }
  }
});
```

#### 3. Stop Receiving
Stop the receiving process.
``` pseuso
void stop();
```

##### Example:
``` java=
recognizer.stop();
```

### Utility

#### 1. Construct
Construct Utility instance to use the utilities Aimazing SDK provide.
``` java
new Utility(Activity activity);
```

##### Arguments:
| Type | Description |
| -------- | -------- |
| `activity` | Android activity class |

##### Example:
``` java=
Utility utility = new Utility(MainActivity.this);
```

#### 2. Start Calibration Process
In order to start using Aimazing SDK, the microphone and speaker of the device need to pass this calibration process to ensure that the device is fully supports the functions Aimazing SDK provides. The result of calibration process will be sent through callback functions. Please refer to the description of `CalibrationCallback` for further details.
``` java
void calibration(calibrationCallback);
```

##### Arguments:
| Type | Description |
| -------- | -------- |
| `calibrationCallback` | CalibrationCallback interface |

##### Example:
```java=
Utility utility = new Utility(MainActivity.this);

utility.calibration(new CalibrationCallback() {
  @Override
  public void onSuccess() {
    // Do something while calibration success.
  }
  
  @Override
  public void onFailed() {
    // Do something while calibration failed.
  }
  
  @Override
  public void onError(UtilityError utilityError) {
    @Override
    public void onError(UtilityError utilityError) {
      switch (utilityError) {
        case PERMISSION_DENIED:
          // Permissions are not granted.
          break;
        case HEADSET_PLUGGED:
          // Headset is plugged into device.
          break;
        case NETWORK_ERROR:
          // Device is not connected to internet.
          break;
      }
    }
  }
});
```

#### 3. Check if Device Has Been Calibrated

```java
boolean isCalibrated();
```

#### 4. Check Required Permission Granted
Aimazing SDK requires the control of playback volume and microphone access. Please use this method to check whether the permissions mentioned before are granted.
``` java
boolean checkPermission();
```

##### Example:
``` java=
if (!utility.checkPermission())
  utility.requestPermission();
```

#### 5. Request Required Permissions
If the permissions are not granted, please use this method to request the permissions from users. Override `onRequestPermissionsResult` in Android Activity can catch the requesting result.
```java
void requestPermission();
```

##### Example:
```java=
if (!utility.checkPermission())
  utility.requestPermission();

@Override
public void onRequestPermissionsResult(int requestCode, String permissions[], int[] grantResults) {
  switch (requestCode) {
    case 1:
      if (grantResults[0] == PackageManager.PERMISSION_GRANTED) {
        // If permission granted
      } else {
        // If permission not granted
      }
  }
}
```

## Callback Interfaces

### CalibrationCallback
- `onSuccess()` - While the device pass the calibration test, this callback function will be called.
- `onFailed()` - While the device fail the calibration test, this callback function will be called.
- `onError(UtilityError utilityError)` - If headset is plugged or required permission not granted, this callback function will be called.

### RecognizerCallback
- `onSuccess(String counterCode)` - While recognizer got the counter code from Dash Speaker, this callback function will be called, the counter code will be passed as `counterCode`.
- `onError(RecognizerError recognizerError)` - If Aimazing SDK is not activated, headset plugged or required permission not granted, this callback function will be called.

## ENUM 

### UtilityError
- `PERMISSION_DENIED` - Required permissions are not granted.
- `HEADSET_PLUGGED` - Headset is plug into device.
- `NETWORK_ERROR` - Device doesn't have internet access.

### RecognizerError
- `NOT_ACTIVATED` - SDK is not activated.
- `PERMISSION_DENIED` - Required permissions are not granted.
- `HEADSET_PLUGGED` - Headset is plugged into device.

---
iOS SDK Usage
===

## Installation
**Add description into info.plist**
Microphone usage description and allowed server connection should be stated clearly in info.plist

```=
<key>NSMicrophoneUsageDescription</key>
<string>YOUR DESCRIPTION</string>
<key>NSAppTransportSecurity</key>
<dict>
  <key>NSExceptionDomains</key>
  <dict>
    <key>dash.aimazing.com.sg</key>
    <dict>
      <!--Include to specify minimum TLS version-->
      <key>NSExceptionMinimumTLSVersion</key>
      <string>TLSv1.2</string>
    </dict>
  </dict>
</dict>
```
:::danger
In Xcode Project, change "Always Embed Swift Standard Framework" to "YES" in Build Settings -> Build Options in project setting
:::

## Classes

### Recognizer

#### 1. Construct
Construct Recognizer instance to start using Aimazing SDK to receive data.
```Objective-C
[[Recognizer alloc] initWith_view:(UIView * _Nonnull)]
```

##### Arguments:
| Type | Description |
| -------- | -------- |
| `_view` | The view that the UIViewController manages |

##### Example:
```Objective-C=
@interface ViewController ()
@property Recognizer *recognizer;
@end
    
@implementation ViewController
- (void)viewDidLoad {
  [super viewDidLoad];
  _recognizer = [[Recognizer alloc] initWith_view: self.view];
}
@end
```

#### 2. Start Receiving
Start receiving data. The result will be given through callback functions.
```Objective-C
[_recognizer startOnSuccess:(NSString * _Nonnull)onSuccess onError:^(enum RecognizerError)onError]
```

##### Arguments:
| Type | Description |
| -------- | -------- |
| `onSuccess` | Callback function when successfully receiving counter code |
| `onError` | Callback function when failed to start receiving data, error value can refer to RecognizerError |

##### Example:
```Objective-C=
[_recognizer startOnSuccess:^(NSString* CounterCode) {
  // THINGS TO DO WHEN SUCCESSFULLY RECEIVING COUNTER CODE
} onError:^(RecognizerError err) {
  // THINGS TO DO WHEN FAILED TO START RECEIVING DATA
}];
```

#### 3. Stop Receiving
Stop the data receiving process.
```Objective-C
void stop()
```

##### Example:
```Objective-C=
[_recognizer stop];
```

### Utility

#### 1. Construct
Construct Utility instance.
```Objective-C
[[Utility alloc] initWith_view:(UIView * _Nonnull)]
```

##### Arguments:
| Type | Description |
| -------- | -------- |
| `_view` | The view that the UIViewController manages |

##### Example:
```Objective-C=
@interface ViewController ()
@property Utility *utility;
@end
    
@implementation ViewController
- (void)viewDidLoad {
    [super viewDidLoad];
    _utility = [[Utility alloc] initWith_view: self.view];
}
@end
```

#### 2. Start Calibration Process
In order to start using Aimazing SDK, the microphone has to pass calibration process to ensure that the device is fully supports functions that Aimazing SDK provided. The result of calibration process will be given through callback functions.
```Objective-C
[_utility calibrationOnSuccess:^(void)onSuccess onFailed:^(void)onFailed onError:^(enum UtilityError)onError]
```

##### Arguments:
| Type | Description |
| -------- | -------- |
| `onSuccess` | Callback function when calibration succeeded |
| `onFailed` | Callback function when device failed to use sound wave to receive data. It means that device can't fully support SDK functionalities |
| `onError` | Callback function when device failed to start calibration. Error value can refer to UtilityError |

##### Example:
```Objective-C=
[_utility calibrationOnSuccess:^ {
  // THINGS TO DO 
} onFailed:^ {
  // THINGS TO DO 
} onError:^(UtilityError err) {
  // THINGS TO DO 
}];
```

#### 3. Check if user's device have calibrated
```Objective-C
Bool isCalibrated()
```

##### Example:
```Objective-C=
if (_utility.isCalibrated) {
  // THINGS TO DO
}
```



## ENUM
### RecognizerError
- `NOT_ACTIVATED` - Error when SDK has not been activated.
- `HEADSET_PLUGGED` - Error when headset is detected.
- `PERMISSION_DENIED` - Error when user doesn't authorize microphone access.

### UtilityError
- `HEADSET_PLUGGED` - Error when headset is detected.
- `PERMISSION_DENIED` - Error when user doesn't authorize microphone access.
- `NETWORK_ERROR` - Error when user's device doesn't have internet access.
