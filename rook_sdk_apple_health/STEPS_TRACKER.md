# Rook Steps Tracker

This SDK enables apps to extract steps from IPhones.

## Features

* Get current day total steps.

## Installation

![Pub Version](https://img.shields.io/pub/v/rook_sdk_apple_health?style=for-the-badge&logo=flutter&label=pubdev&color=7200F7)

```text
flutter pub add rook_sdk_apple_health
```

### Development environment

This package was developed with the following sdk constraints:

* dart: ">=3.0.0 <4.0.0"
* flutter: ">=3.0.0"

### Required dependencies

To use this package you'll need to add the following dependencies to your project:

* [rook_sdk_core](https://pub.dev/packages/rook_sdk_core): "0.1.0"

## Getting started

### IOS configuration

On root folder run:

```text
flutter pub get
```

On ios/Podfile add the following

```text
platform :ios, '13.0'
```

On ios folder run:

```text
pod install
```

Open the ios folder with Xcode

Add HealthKit capability to your Xcode project:

1. Open your project in Xcode.
2. Click on your project file (most cases is Runner) in the Project Navigator.
3. Click on the "Signing and capabilities" tab.
4. Click on the "+ Capability" button and select "HealthKit" from the list.
5. Check the "Background delivery" option

Add HealthKit framework to your Xcode project:

1. Open your project in Xcode.
2. Click on your project file (most cases is Runner) in the Project Navigator.
3. Click on the "Build Phases" tab.
4. Click on the "+" button under the "Link Binary With Libraries" section and select "HealthKit.framework" from the
   list.

Declare the privacy permissions used by this SDK. You will need to include the NSHealthShareUsageDescription and
NSHealthUpdateUsageDescription keys in your app's Info.plist file.

These keys provide a description of why your app needs to access HealthKit data and will be displayed to the user in the
permission request dialog.

1. Open ios folder
2. Open Runner folder
3. Add the following to **Info.plist**

```plist
<key>NSHealthShareUsageDescription</key>
<string>This app requires access to your health and fitness data in order to track your workouts and activity levels.</string>
<key>NSHealthUpdateUsageDescription</key>
<string>This app requires permission to write workout data to HealthKit.</string>
```

## Usage

### Initialize

Go to the main [Initialize](README.md#initialize) and [Update userID](README.md#update-userid) sections to initialize.

### Steps Tracker Configuration

#### Permissions

To use the `IosStepsTracker` you will need PHYSICAL pillar permissions, go
to [Request permissions](README.md#request-permissions) to learn how to request permissions.

#### Tracker controls

To start tracking steps call `IosStepsTracker.start` :

```dart
void startStepsTracker() async {
  try {
    await IosStepsTracker.start();

    // IosStepsTracker start request send successfully. 
    // Use IosStepsTracker.isActive() to ensure proper activation.
  } catch (exception) {
    final error = switch (exception) {
      _ => exception.toString(),
    };

    // Error sending IosStepsTracker start request. 
  }
}
```

To stop tracking steps call `IosStepsTracker.stop`:

```dart
void stopStepsTracker() async {
  try {
    await IosStepsTracker.stop();

    // IosStepsTracker stop request send successfully. 
    // Use IosStepsTracker.isActive() to ensure proper deactivation.
  } catch (exception) {
    final error = switch (exception) {
      _ => exception.toString(),
    };

    // Error sending IosStepsTracker stop request. 
  }
}
```

#### Recommendations

Calling `IosStepsTracker.start`/ `IosStepsTracker.stop` when the service is active/inactive will do nothing,
however you can check if the service is active with `IosStepsTracker.isActive`:

```dart
void checkStatus() async {
  final isActive = await IosStepsTracker.isActive();
}
```

#### Get today step count

Call `IosStepsTracker.getTodaySteps` to obtain the last stored step count of the current day:

```dart
void getTodaySteps() {
  IosStepsTracker.getTodaySteps().then((todaySteps) {
    // Steps obtained successfully.
  }).catchError((exception) {
    final error = switch (exception) {
      _ => exception.toString(),
    };

    // Error obtaining steps.
  });
}
```

This value is updated every time the IPhone and Watch sync (To speed this up, open the Fitness app, so it will force a
sync), this value will be reset to zero every day. You can call this function as many times as you like, although we
recommend to put a delay of 1-3 seconds between each call.

```dart
void getTodaySteps() {
  // 1000 to 3000 milliseconds
  Timer.periodic(const Duration(seconds: 3), (timer) {
    IosStepsTracker.getTodaySteps().then((todaySteps) {
      // Steps obtained successfully.
    }).catchError((exception) {
      final error = switch (exception) {
        _ => exception.toString(),
      };

      // Error obtaining steps.
    });
  });
}
```
