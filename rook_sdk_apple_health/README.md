# Rook SDK Apple Health

This package enables apps to extract and upload data from Apple Health.

* This package was developed with our modular packages as a way to simplify their implementation:
    * rook-auth (native)
    * rook-users (native)
    * rook-apple-health (native)
    * rook-transmission (native)

## Features

* Get authorization
* Register users
* Extract health data (Apple Health)
* Upload health data (Apple Health)

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

## Usage

### Initialize

Create an instance of `AHRookConfigurationManager`:

```dart

final rookConfigurationManager = AHRookConfigurationManager();
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
  
  rookConfigurationManager.setConfiguration(rookConfiguration);

  rookConfigurationManager.initRook().then((_) {
    // SDK initialized successfully
  }).catchError((exception) {
    // Error initializing SDK

    final error = switch (exception) {
      (MissingConfigurationException it) => 'MissingConfigurationException: ${it.message}',
      _ => exception.toString(),
    };
  });
}
```

#### Recommendations

When you initialize with `initRook` the SDK an HTTP request is made, so you should only initialize the sdk once. We
recommend to code the initialization in the highest level of your widget tree or treat the `AHRookConfigurationManager` as
a singleton if you are using dependency injection.

### Update userID

Update the [userID](https://docs.tryrook.io/docs/Definitions#user_id):

```dart
void updateUserID() {
  rookConfigurationManager.updateUserID(userID).then((_) {
    // userID updated successfully
  }).catchError((exception) {
    // Error updating userID

    final error = switch (exception) {
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
APPLE_HEALTH data source once removed rook servers won't accept any health data from APPLE_HEALTH.

```dart
void deleteUser() {
  rookConfigurationManager.deleteUserFromRook().then((_) {
    // User deleted from rook
  }).catchError((exception) {
    final error = switch (exception) {
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
      _ => exception.toString(),
    };

    logger.info('Error updating user timezone:');
    logger.info(error);
  });
}
```

### Request permissions

Create a `AHRookHealthPermissionsManager` instance:

There are dedicated functions for each [Health Pillar](https://docs.tryrook.io/docs/Definitions#health-data-pillars)
(Sleep, Physical and Body) to request permissions. These functions follow the
convention: `request_data_type_Permissions`

You can also call `requestAllPermissions` to request all health pillar permissions.

```dart
void requestPermissions() {
  logger.info("Requesting all permissions...");

  rookHealthPermissionsManager.requestAllPermissions().then((_) {
    // All permissions request sent
  }).catchError((exception) {
    // Error requesting all permissions

    final error = switch (exception) {
      _ => exception.toString(),
    };
  });
}
```

### Sync health data

There are 2 types of health data **Summaries** and **Events**.

| Health Data | Timezone | Oldest date of retrieval | Soonest date of retrieval | Class                |
|-------------|----------|--------------------------|---------------------------|----------------------|
| Summary     | UTC      | N/A                      | Yesterday                 | AHRookSummaryManager |
| Event       | UTC      | N/A                      | Today                     | AHRookEventManager   |

Create the required instances:

```dart

final rookSummaryManager = AHRookSummaryManager();
final rookEventManager = AHRookEventManager();
```

#### Sync summaries

To sync any type of summary, you need to provide a date. This date cannot be the current
day. See the examples below:

| Current date | Provided date | Is valid?                          |
|--------------|---------------|------------------------------------|
| 2023-01-08   | 2023-01-08    | No, the date is from today         |
| 2023-01-08   | 2023-01-07    | Yes, the date is from yesterday    |
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
      _ => exception.toString(),
    };
  }
}
```

#### Sync pending summaries

When you call `sync_xxx_summary`, what happens it's that:

1. Health data is extracted from Apple Health
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
      _ => exception.toString(),
    };
  });
}
```

#### Recommendations

Summaries are collections of health data from a past day, so it's not necessary to sync them many times per day.

```dart
/// This function assumes that you have already permissions.
Future<void> syncSummaries() async {
  final today = DateTime.now();
  final yesterday = today.subtract(const Duration(days: 1));

  try {
    await rookSummaryManager.syncSleepSummary(yesterday);

    // Sleep summary synced successfully
  } catch (exception) {
    // Error syncing Sleep summary
  };

  // Sync for other types of summaries...
  
  // Finally you can call syncPendingSummaries to retry all failed uploads (optional)
  // try {
  //   await rookEventManager.syncPendingSummaries();
  //
  //   // Success
  // } catch (exception) {
  //   // Error
  // }
}
```

#### Sync events

To sync any type of event, you need to provide a date. See the examples below:

| Current date | Provided date | Is valid?                          |
|--------------|---------------|------------------------------------|
| 2023-01-08   | 2023-01-08    | Yes, the date is from today        |
| 2023-01-08   | 2023-01-07    | Yes, the date is from yesterday    |
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
      _ => exception.toString(),
    };
  }
}
```

#### Sync pending events

When you call `sync_xxx_events`, what happens it's that:

1. Health data is extracted from Apple Health
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
      _ => exception.toString(),
    };
  });
}
```

#### Recommendations

Events are collections of health data divided in intervals of 1 hour, so you can/should sync them frequently. Our
recommendation is that every time your app is opened you should sync events.

```dart
/// This function assumes that you have already checked permissions.
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

## Other resources

* See a complete list of rook-sdk-apple-health methods in the [API Reference](https://pub.dev/documentation/rook_sdk_apple_health/latest/rook_sdk_apple_health/rook_sdk_apple_health-library.html)
* Download and compile the demo application from our [Repository](https://github.com/RookeriesDevelopment/rook_demo_app_flutter_rook_sdk)
