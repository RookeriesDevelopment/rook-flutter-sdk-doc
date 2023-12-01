# Rook SDK Health Connect

This package enables apps to extract and upload data from Health Connect.

* This package was developed with our modular packages as a way to simplify their implementation:
    * rook-auth (native)
    * rook-users (native)
    * rook-health-connect (native)
    * rook-transmission (native)

## Features

* Get authorization
* Register users
* Extract health data (Health Connect)
* Upload health data (Health Connect)

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

* Health Connect will only work with devices of a minSdk 28 or later. The `minSdk 26` is to keep
  compatibility with other Rook SDKs that can be used with older SDKs.

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

#### Manifest

In your **AndroidManifest.xml** add a query for Health Connect

```xml

<manifest>
    <queries>
        <package android:name="com.google.android.apps.healthdata"/>
    </queries>
</manifest>
```

Then declare the health permissions used by this SDK:

```text
<uses-permission android:name="android.permission.health.READ_SLEEP" />
<uses-permission android:name="android.permission.health.READ_STEPS" />
<uses-permission android:name="android.permission.health.READ_DISTANCE" />
<uses-permission android:name="android.permission.health.READ_FLOORS_CLIMBED" />
<uses-permission android:name="android.permission.health.READ_ELEVATION_GAINED" />
<uses-permission android:name="android.permission.health.READ_OXYGEN_SATURATION" />
<uses-permission android:name="android.permission.health.READ_VO2_MAX" />
<uses-permission android:name="android.permission.health.READ_TOTAL_CALORIES_BURNED" />
<uses-permission android:name="android.permission.health.READ_ACTIVE_CALORIES_BURNED" />
<uses-permission android:name="android.permission.health.READ_HEART_RATE" />
<uses-permission android:name="android.permission.health.READ_RESTING_HEART_RATE" />
<uses-permission android:name="android.permission.health.READ_HEART_RATE_VARIABILITY" />
<uses-permission android:name="android.permission.health.READ_EXERCISE" />
<uses-permission android:name="android.permission.health.READ_SPEED" />
<uses-permission android:name="android.permission.health.READ_WEIGHT" />
<uses-permission android:name="android.permission.health.READ_HEIGHT" />
<uses-permission android:name="android.permission.health.READ_BLOOD_GLUCOSE" />
<uses-permission android:name="android.permission.health.READ_BLOOD_PRESSURE" />
<uses-permission android:name="android.permission.health.READ_HYDRATION" />
<uses-permission android:name="android.permission.health.READ_BODY_TEMPERATURE" />
<uses-permission android:name="android.permission.health.READ_RESPIRATORY_RATE" />
<uses-permission android:name="android.permission.health.READ_NUTRITION" />
<uses-permission android:name="android.permission.health.READ_MENSTRUATION" />
<uses-permission android:name="android.permission.health.READ_POWER" />
```

### Privacy policy

Health Connect requires a privacy policy where you inform your users how you will handle and use their data.

To comply with this you'll need an intent filter, inside your MainActivity declaration add an intent filter for the
Health Connect permissions action:

```xml

<application>
    <activity android:name=".MainActivity">
        <!-- For supported versions through Android 13, create an activity to show the rationale
             of Health Connect permissions once users click the privacy policy link. -->
        <intent-filter>
            <action android:name="androidx.health.ACTION_SHOW_PERMISSIONS_RATIONALE"/>
        </intent-filter>
    </activity>

    <!-- For versions starting Android 14, create an activity alias to show the rationale
         of Health Connect permissions once users click the privacy policy link. -->
    <activity-alias
        android:name="ViewPermissionUsageActivity"
        android:exported="true"
        android:permission="android.permission.START_VIEW_PERMISSION_USAGE"
        android:targetActivity=".MainActivity">

        <intent-filter>
            <action android:name="android.intent.action.VIEW_PERMISSION_USAGE"/>
            <category android:name="android.intent.category.HEALTH_PERMISSIONS"/>
        </intent-filter>
    </activity-alias>
</application>
```

Finally, you to listen when your app is launched from said intent you can code
your [own](https://docs.flutter.dev/get-started/flutter-for/android-devs#how-do-i-handle-incoming-intents-from-external-applications-in-flutter)
implementation from scratch or use a package like [receive_intent](https://pub.dev/packages/receive_intent). You can
find an example of the second approach in our demo app.

#### Request data access

When you are developing with the Health Connect SDK data access is unrestricted. In order to have data access when your
app is launched to the PlayStore you MUST complete the Developer Declaration Form, more
information [Here](https://developer.android.com/health-and-fitness/guides/health-connect/publish/request-access).

### Environment

The `RookEnvironment` enum allows to quickly configure the behaviour of **rook-sdk**, e.g. the
api used to communicate with ROOK servers.

Available environments:

* sandbox ➞ Use this during your app development process.
* production ➞ Use this ONLY when your app is published to the PlayStore.

You can use the `kDebugMode` property of you app to configure the environment:

```dart

const environment = kDebugMode ? RookEnvironment.sandbox : RookEnvironment.production;
```

### Logging

If you want to see the logs generated by the **rook-sdk** native SDK, before `setConfiguration` call `enableLocalLogs`.
You can use
the `kDebugMode` property of you app to configure logging:

```dart
void enableLogs() {
  if (kDebugMode) {
    rookConfigurationManager.enableNativeLogs();
  }
}
```

## Usage

### Initialize

Create an instance of `HCRookConfigurationManager`:

```dart

final rookConfigurationManager = HCRookConfigurationManager();
```

Set a configuration and initialize. The `RookConfiguration` requires the following parameters:

* [clientUUID](https://docs.tryrook.io/docs/Definitions/#client_uuid)
* [secretKey](https://docs.tryrook.io/docs/Definitions/#client_secret)
* [Environment](#environment)

```dart
void initialize() {
  const environment =
  kDebugMode ? RookEnvironment.sandbox : RookEnvironment.production;

  final rookConfiguration = RookConfiguration(
    clientUUID,
    secretKey,
    environment,
  );

  // MUST be called first if you want to enable native logs
  if (kDebugMode) {
    rookConfigurationManager.enableNativeLogs();
  }
  rookConfigurationManager.setConfiguration(rookConfiguration);

  rookConfigurationManager.initRook().then((_) {
    // SDK initialized successfully
  }).catchError((exception) {
    // Error initializing SDK

    final error = switch (exception) {
      (MissingConfigurationException it) => 'MissingConfigurationException: ${it.message}',
      (ConnectTimeoutException it) => 'TimeoutException: ${it.message}',
      _ => exception.toString(),
    };
  });
}
```

#### Recommendations

When you initialize with `initRook` the SDK an HTTP request is made, so you should only initialize the sdk once. We
recommend to code the initialization in the highest level of your widget tree or treat the `HCRookConfigurationManager`
as a singleton if you are using dependency injection.

### Update userID

Update the [userID](https://docs.tryrook.io/docs/Definitions#user_id):

```dart
void updateUserID() {
  rookConfigurationManager.updateUserID(userID).then((_) {
    // userID updated successfully
  }).catchError((exception) {
    // Error updating userID

    final error = switch (exception) {
      (SDKNotInitializedException it) => 'SDKNotInitializedException: ${it.message}',
      (ConnectTimeoutException it) => 'TimeoutException: ${it.message}',
      _ => exception.toString(),
    };
  });
}
```

#### Recommendations

The `updateUserID` should be called as a part of your "Log in" and/or "Is Logged in" flow, when your users log out from
your app call `clearUserID` (this is optional as any call to `updateUserID` will override the previous userID).

```dart
void clearUserID() {
  rookConfigurationManager.clearUserID();
}
```

### Removing registered users

If you want to remove a userID from a data source call `deleteUserFromRook` it will remove the current user from
HEALTH_CONNECT data source once removed rook servers won't accept any health data from HEALTH_CONNECT.

```dart
void deleteUser() {
  rookConfigurationManager.deleteUserFromRook().then((_) {
    // User deleted from rook
  }).catchError((exception) {
    final error = switch (exception) {
      (SDKNotInitializedException it) =>
      'SDKNotInitializedException: ${it.message}',
      (UserNotInitializedException it) =>
      'UserNotInitializedException: ${it.message}',
      _ => exception.toString(),
    };

    // Error deleting user from rook
  });
}
```

#### User timezone

Every time `updateUserID` completes successfully the timezone information will be updated, you can see the result of the
operation in the logs, when success you will see `Timezone information updated`
otherwise `Failed to update timezone information with error:`

In most cases the above behavior is more than enough. If you need to update the timezone information manually
call `syncUserTimeZone`:

```dart
void updateTimeZoneInformation() {
  logger.info('Updating user timezone...');

  rookConfigurationManager.syncUserTimeZone().then((_) {
    logger.info('User timezone updated successfully');
  }).catchError((exception) {
    final error = switch (exception) {
      (SDKNotInitializedException it) => 'SDKNotInitializedException: ${it.message}',
      (UserNotInitializedException it) => 'UserNotInitializedException: ${it.message}',
      (ConnectTimeoutException it) => 'ConnectTimeoutException: ${it.message}',
      (HttpRequestException it) => 'HttpRequestException: ${it.message}',
      _ => exception.toString(),
    };

    logger.info('Error updating user timezone:');
    logger.info(error);
  });
}
```

### Check availability

Before proceeding further, you need to ensure the user's device is compatible with
Health Connect and check if the [APK](https://play.google.com/store/apps/details?id=com.google.android.apps.healthdata)
is installed.

Create a `HCRookHealthPermissionsManager` instance:

```dart

final rookHealthPermissionsManager = HCRookHealthPermissionsManager();
```

Call `checkAvailability` and take the corresponding actions:

| Status       | Description                                 | What to do                                      |
|--------------|---------------------------------------------|-------------------------------------------------|
| installed    | APK is installed                            | Proceed to check permissions                    |
| notInstalled | APK is not installed                        | Prompt the user to install Health Connect.      |
| notSupported | This device does not support Health Connect | Take the user out of the Health Connect section |

```dart
void checkAvailability() {
  rookHealthPermissionsManager.checkAvailability().then((availability) {
    final string = switch (availability) {
      HealthConnectAvailabilityStatus.installed => 'Health Connect is installed! You can skip the next step',
      HealthConnectAvailabilityStatus.notInstalled =>
      'Health Connect is not installed. Please download from the Play Store',
      _ => 'This device is not compatible with health connect. Please close the app',
    };
  }).catchError((exception) {
    // Error checking availability

    final error = exception.toString();
  });
}
```

### Check permissions

To check permissions call `checkPermissions` and provide a `HealthPermission`, available permissions:

* SLEEP - [Sleep Health](https://docs.tryrook.io/docs/Definitions/#sleep-health-data-pillar) Data Pillar permissions.
* PHYSICAL - [Physical Health](https://docs.tryrook.io/docs/Definitions/#body-health-data-pillar) Data Pillar
  permissions.
* BODY - [Body Health](https://docs.tryrook.io/docs/Definitions/#body-health-data-pillar) Data Pillar permissions.
* ALL - All Health Data Pillar permissions.

```dart
void checkPermissions() {
  rookHealthPermissionsManager.checkPermissions(HealthPermission.all).then((hasPermissions) {
    final string = hasPermissions
        ? 'All permissions are granted! You can skip the next 2 steps'
        : 'There are missing permissions. Please grant them';
  }).catchError((exception) {
    // Error checking all permissions

    final error = switch (exception) {
      (SDKNotInitializedException it) => 'SDKNotInitializedException: ${it.message}',
      (UserNotInitializedException it) => 'UserNotInitializedException: ${it.message}',
      (HealthConnectNotInstalledException it) =>
      'HealthConnectNotInstalledException: ${it.message}',
      (DeviceNotSupportedException it) => 'DeviceNotSupportedException: ${it.message}',
      _ => exception.toString(),
    };
  });
}
```

### Request permissions

To request permissions call `launchPermissionsRequest` providing a `HealthPermission`.

```dart
void requestPermissions() {
  logger.info("Requesting all permissions...");

  rookHealthPermissionsManager.requestPermissions(HealthPermission.all).then((_) {
    // All permissions request sent
  }).catchError((exception) {
    // Error requesting all permissions

    final error = switch (exception) {
      _ => exception.toString(),
    };
  });
}
```

**Permissions denied**

If the user clicks cancel or navigates away from the permissions screen, Health Connect will take it as if the user
denied the permissions.

If the user 'denies' the permissions 2 times, your app will be blocked by Health Connect. This is
permanent and cannot be undone even if the user uninstalls your app.

When your app is blocked, any permissions request will be ignored.

To solve this problem, we recommend you also include an `Open Health Connect` button in your
permissions UI. This button will call `openHealthConnectSettings`, there your users can manually
grant permissions to your app.

```dart
  void openHealthConnect() {
  logger.info("Opening Health Connect...");

  rookHealthPermissionsManager.openHealthConnectSettings().then((_) {
    // Health Connect was opened
  }).catchError((exception) {
    // Error opening Health Connect

    final error = switch (exception) {
      (SDKNotInitializedException it) => 'SDKNotInitializedException: ${it.message}',
      (UserNotInitializedException it) => 'UserNotInitializedException: ${it.message}',
      (HealthConnectNotInstalledException it) =>
      'HealthConnectNotInstalledException: ${it.message}',
      (DeviceNotSupportedException it) => 'DeviceNotSupportedException: ${it.message}',
      _ => exception.toString(),
    };
  });
}
```

### Sync health data

There are 2 types of health data **Summaries** and **Events**.

| Health Data | Timezone | Oldest date of retrieval | Soonest date of retrieval | Class                |
|-------------|----------|--------------------------|---------------------------|----------------------|
| Summary     | UTC      | 29 days                  | Yesterday                 | HCRookSummaryManager |
| Event       | UTC      | 29 days                  | Today                     | HCRookEventManager   |

Create the required instances:

```dart

final rookSummaryManager = HCRookSummaryManager();
final rookEventManager = HCRookEventManager();
```

#### Sync summaries

To sync any type of summary, you need to provide a date. This date cannot be the current
day and cannot be older than 29 days. See the examples below:

| Current date | Provided date | Is valid?                          |
|--------------|---------------|------------------------------------|
| 2023-01-08   | 2023-01-08    | No, the date is from today         |
| 2023-01-08   | 2023-01-07    | Yes, the date is from yesterday    |
| 2023-01-08   | 2022-11-01    | No, the date is older than 29 days |
| 2023-01-08   | 2023-01-01    | Yes, the date is 7 days old        |

To get health data, call `sync_xxx_summary` and provide a LocalDate instance of the day you want to retrieve the data
from.

For example, if you want to sync yesterday's sleep summary, call `syncSleepSummary`.

```dart
Future<void> syncYesterdaySleepSummary() async {
  final today = DateTime.now();
  final yesterday = today.subtract(const Duration(days: 1));

  try {
    await rookSummaryManager.syncSleepSummary(yesterday);

    // Sleep summary synced successfully
  } catch (exception) {
    // Error syncing Sleep summary

    final error = switch (exception) {
      (SDKNotInitializedException it) => 'SDKNotInitializedException: ${it.message}',
      (UserNotInitializedException it) => 'UserNotInitializedException: ${it.message}',
      (HealthConnectNotInstalledException it) =>
      'HealthConnectNotInstalledException: ${it.message}',
      (DeviceNotSupportedException it) => 'DeviceNotSupportedException: ${it.message}',
      (MissingPermissionsException it) => 'MissingPermissionsException: ${it.message}',
      (ConnectTimeoutException it) => 'ConnectTimeoutException: ${it.message}',
      (HttpRequestException it) =>
      'HttpRequestException: code: ${it.code} message: ${it.message}',
      _ => exception.toString(),
    };
  }
}
```

#### Sync pending summaries

When you call `sync_xxx_summary`, what happens it's that:

1. Health data is extracted from Health Connect
2. The extracted data is enqueued
3. The enqueued data is uploaded to ROOK servers.
4. If success the queued is cleared. Otherwise, the summary is stored.

To retry sending those stored summaries call `syncPendingSummaries`

```dart
void syncPendingSummaries() {
  rookSummaryManager.syncPendingSummaries().then((_) {
    // Pending summaries synced successfully
  }).catchError((exception) {
    // Error syncing pending summaries

    final error = switch (exception) {
      (SDKNotInitializedException it) => 'SDKNotInitializedException: ${it.message}',
      (UserNotInitializedException it) => 'UserNotInitializedException: ${it.message}',
      (ConnectTimeoutException it) => 'ConnectTimeoutException: ${it.message}',
      (HttpRequestException it) =>
      'HttpRequestException: code: ${it.code} message: ${it.message}',
      _ => exception.toString(),
    };
  });
}
```

#### Recommendations

Summaries are collections of health data from a past day, so you should not sync them more than once. Our recommendation
is that every time your app is opened you should call `should_sync_xxx_summaries_for` to check if you have synced
yesterday's summaries.

If the returned value is true sync summaries, otherwise call `syncPendingSummaries` to retry any failed summary upload.

```dart
/// This function assumes that you have already checked availability and permissions.
Future<void> syncSummaries() async {
  final today = DateTime.now();
  final yesterday = today.subtract(const Duration(days: 1));

  try {
    final shouldSyncSummariesForYesterday = await rookSummaryManager.shouldSyncSleepSummariesFor(
      yesterday,
    );

    if (shouldSyncSummariesForYesterday) {
      try {
        await rookSummaryManager.syncSleepSummary(yesterday);

        // Sleep summary synced successfully
      } catch (exception) {
        // Error syncing Sleep summary
      };
    } else {
      try {
        await rookSummaryManager.syncPendingSummaries();

        // Pending summaries synced successfully
      } catch (exception) {
        // Error syncing pending summaries
      };
    }
  } catch (exception) {
    // Error

    final error = switch (exception) {
      (SDKNotInitializedException it) => 'SDKNotInitializedException: ${it.message}',
      (UserNotInitializedException it) => 'UserNotInitializedException: ${it.message}',
      (HealthConnectNotInstalledException it) =>
      'HealthConnectNotInstalledException: ${it.message}',
      (DeviceNotSupportedException it) => 'DeviceNotSupportedException: ${it.message}',
      _ => exception.toString(),
    };
  }

  // Check for other types of summaries...
}
```

#### Sync events

To sync any type of event, you need to provide a date. This date cannot be older than 29 days. See the examples
below:

| Current date | Provided date | Is valid?                          |
|--------------|---------------|------------------------------------|
| 2023-01-08   | 2023-01-08    | Yes, the date is from today        |
| 2023-01-08   | 2023-01-07    | Yes, the date is from yesterday    |
| 2023-01-08   | 2022-11-01    | No, the date is older than 29 days |
| 2023-01-08   | 2023-01-01    | Yes, the date is 7 days old        |

To get health data, call `sync_xxx_events` and provide a LocalDate instance of the day you want to retrieve the data
from.

For example, if you want to sync today's physical events, call `syncPhysicalEvents`.

```dart
Future<void> syncTodayPhysicalEvents() async {
  final today = DateTime.now();

  try {
    await rookEventManager.syncPhysicalEvents(today);

    // Physical events synced successfully
  } catch (exception) {
    // Error syncing Physical events

    final error = switch (exception) {
      (SDKNotInitializedException it) => 'SDKNotInitializedException: ${it.message}',
      (UserNotInitializedException it) => 'UserNotInitializedException: ${it.message}',
      (HealthConnectNotInstalledException it) =>
      'HealthConnectNotInstalledException: ${it.message}',
      (DeviceNotSupportedException it) => 'DeviceNotSupportedException: ${it.message}',
      (MissingPermissionsException it) => 'MissingPermissionsException: ${it.message}',
      (ConnectTimeoutException it) => 'ConnectTimeoutException: ${it.message}',
      (HttpRequestException it) =>
      'HttpRequestException: code: ${it.code} message: ${it.message}',
      _ => exception.toString(),
    };
  }
}
```

#### Sync pending events

When you call `sync_xxx_events`, what happens it's that:

1. Health data is extracted from Health Connect
2. The extracted data is enqueued
3. The enqueued data is uploaded to ROOK servers.
4. If success the queued is cleared. Otherwise, the events is stored.

To retry sending those stored events call `syncPendingEvents`

```dart
void syncPendingEvents() {
  rookEventManager.syncPendingEvents().then((_) {
    // Pending events synced successfully
  }).catchError((exception) {
    // Error syncing pending events

    final error = switch (exception) {
      (SDKNotInitializedException it) => 'SDKNotInitializedException: ${it.message}',
      (UserNotInitializedException it) => 'UserNotInitializedException: ${it.message}',
      (ConnectTimeoutException it) => 'ConnectTimeoutException: ${it.message}',
      (HttpRequestException it) =>
      'HttpRequestException: code: ${it.code} message: ${it.message}',
      _ => exception.toString(),
    };
  });
}
```

#### Recommendations

Events are collections of health data divided in intervals of 1 hour, so you can/should sync them frequently. Our
recommendation is that every time your app is opened you should sync events.

```dart
/// This function assumes that you have already checked availability and permissions.
Future<void> syncTodayPhysicalEvents() async {
  final today = DateTime.now();

  try {
    await rookEventManager.syncPhysicalEvents(today);

    // Physical events synced successfully
  } catch (exception) {
    // Error syncing Physical events
  }

  // Sync other types of events...

  // Finally you can call syncPendingEvents to retry all failed uploads (optional)
  // try {
  //   await rookEventManager.syncPendingEvents();
  //
  //   // Success
  // } catch (exception) {
  //   // Error
  // }
}
```

* Be cautions of not syncing events an excessive amount of times, Health Connect has a daily usage limit and your app
  could be blocked for some hours or a full day.

## Other resources

* See a complete list of rook-sdk-health-connect methods in the [API Reference](https://pub.dev/documentation/rook_sdk_health_connect/latest/rook_sdk_health_connect/rook_sdk_health_connect-library.html)
* Download and compile the demo application from our [Repository](https://github.com/RookeriesDevelopment/rook_demo_app_flutter_rook_sdk)
