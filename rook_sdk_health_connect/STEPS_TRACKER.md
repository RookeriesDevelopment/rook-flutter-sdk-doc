# Rook Steps Tracker

This SDK enables apps to extract steps from phones.

## Features

* Get current day total steps.

## Installation

![Pub Version](https://img.shields.io/pub/v/rook_sdk_health_connect?style=for-the-badge&logo=flutter&label=pubdev&color=7200F7)

```text
flutter pub add rook_sdk_health_connect
```

### Development environment

This package was developed with the following sdk constraints:

* dart: ">=3.0.0 <4.0.0"
* flutter: ">=3.0.0"

### Required dependencies

To use this package you'll need to add the following dependencies to your project:

* [rook_sdk_core](https://pub.dev/packages/rook_sdk_core): "0.1.0"

## Getting started

### Android configuration

In your build.gradle (app) set your min and target sdk version like below:

```groovy
minSdk 26
targetSdk 34
```

#### Obfuscation

If you are using obfuscation consider the following:

In the app folder create a **proguard-rules.pro** file and add the following:

```text
-keep class * extends com.google.protobuf.GeneratedMessageLite { *; }
```

Finally, in your build.gradle (app) add the `proguard-rules.pro` from previous step:

```groovy
buildTypes {
    release {
        proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
    }
}
```

In your gradle.properties (Project level) add the following to disable R8 full mode:

```properties
android.enableR8.fullMode=false
```

If you want to enable full mode add the following rules to proguard-rules.pro:

```text
# Keep generic signature of Call, Response (R8 full mode strips signatures from non-kept items).
-keep,allowobfuscation,allowshrinking interface retrofit2.Call
-keep,allowobfuscation,allowshrinking class retrofit2.Response

# With R8 full mode generic signatures are stripped for classes that are not
# kept. Suspend functions are wrapped in continuations where the type argument
# is used.
-keep,allowobfuscation,allowshrinking class kotlin.coroutines.Continuation
```

#### Included permissions

This SDK will add the following permissions to your manifest:

* RECEIVE_BOOT_COMPLETED
* ACTIVITY_RECOGNITION
* POST_NOTIFICATIONS
* FOREGROUND_SERVICE
* FOREGROUND_SERVICE_HEALTH

### Logging

Go to the main [Logging](README.md#logging) section to configure logs.

## Usage

### Initialize

Go to the main [Initialize](README.md#initialize) and [Update userID](README.md#update-userid) sections to initialize.

### Check availability

Before proceeding further, you need to ensure the user's device has the required sensors.
Call `AndroidStepsTracker.isAvailable`:

```dart
void isAvailable() async {
  final isAvailable = await AndroidStepsTracker.isAvailable();
}
```

### Check permissions

To check permissions call `AndroidStepsTracker.hasPermissions`:

```dart
void hasPermissions() async {
  final hasPermissions = await AndroidStepsTracker.hasPermissions();
}
```

### Request permissions

To request permissions call `AndroidStepsTracker.requestPermissions`:

```dart
void requestPermissions() async {
  await AndroidStepsTracker.requestPermissions();
}
```

### Steps Tracker Configuration

#### Notification

The steps tracker works using a Foreground service which requires a notification to be permanently displayed, although
from Android 13 (SDK 33) this notification can be dismissed without finishing the service associated with it, then the
service will be displayed in the active apps section (This may vary depending on device brand).

To use the steps tracker service you need to reference your own resources (icon, title and content) in the *
*AndroidManifest.xml** file:

```xml

<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <application>
        <meta-data
            android:name="io.tryrook.rook.realtime.STEPS_SERVICE_NOTIFICATION_ICON"
            android:resource="@drawable/my_custom_icon"/>

        <meta-data
            android:name="io.tryrook.rook.realtime.STEPS_SERVICE_NOTIFICATION_TITLE"
            android:resource="@string/my_custom_title"/>

        <meta-data
            android:name="io.tryrook.rook.realtime.STEPS_SERVICE_NOTIFICATION_CONTENT"
            android:resource="@string/my_custom_content"/>
    </application>
</manifest>
```

#### Tracker controls

To start tracking steps call `AndroidStepsTracker.start` :

```dart
void startStepsTracker() async {
  try {
    await AndroidStepsTracker.start();

    // AndroidStepsTracker start request send successfully. 
    // Use AndroidStepsTracker.isActive() to ensure proper activation.
  } catch (exception) {
    final error = switch (exception) {
      (SDKNotInitializedException it) =>
      'SDKNotInitializedException: ${it.message}',
      (MissingAndroidPermissionsException it) =>
      'MissingAndroidPermissionsException: ${it.message}',
      _ => exception.toString(),
    };

    // Error sending AndroidStepsTracker start request. 
  }
}
```

To stop tracking steps call `AndroidStepsTracker.stop`:

```dart
void stopStepsTracker() async {
  try {
    await AndroidStepsTracker.stop();

    // AndroidStepsTracker stop request send successfully. 
    // Use AndroidStepsTracker.isActive() to ensure proper deactivation.
  } catch (exception) {
    final error = switch (exception) {
      (SDKNotInitializedException it) =>
      'SDKNotInitializedException: ${it.message}',
      _ => exception.toString(),
    };

    // Error sending AndroidStepsTracker stop request. 
  }
}
```

#### Recommendations

Calling `AndroidStepsTracker.start`/ `AndroidStepsTracker.stop` when the service is active/inactive will do nothing,
however you can check if the service is
active with `AndroidStepsTracker.isActive`:

```dart
void requestPermissions() async {
  final isActive = await AndroidStepsTracker.isActive();
}
```

#### Get today step count

Call `AndroidStepsTracker.getTodaySteps` to obtain the last stored step count of the current day:

```dart
void getTodaySteps() {
  AndroidStepsTracker.getTodaySteps().then((todaySteps) {
    // Steps obtained successfully.
  }).catchError((exception) {
    final error = switch (exception) {
      (SDKNotInitializedException it) =>
      'SDKNotInitializedException: ${it.message}',
      _ => exception.toString(),
    };

    // Error obtaining steps.
  });
}
```

This value is updated every time the phone detects a step taken and will be reset to zero every day. You can call this
function as many times as you like, although we recommend to put a delay of 1-3 seconds between each call.

```dart
void getTodaySteps() {
  // 1000 to 3000 milliseconds
  Timer.periodic(const Duration(seconds: 3), (timer) {
    AndroidStepsTracker.getTodaySteps().then((todaySteps) {
      // Steps obtained successfully.
    }).catchError((exception) {
      final error = switch (exception) {
        (SDKNotInitializedException it) =>
        'SDKNotInitializedException: ${it.message}',
        _ => exception.toString(),
      };

      // Error obtaining steps.
    });
  });
}
```

#### Auto start

After a call to `AndroidStepsTracker.start()` if the device is restarted the Foreground service will start after the
user
unlocks their device for the first time (This may vary depending on device brand). This behaviour will be stopped when
calling `AndroidStepsTracker.stop()`
