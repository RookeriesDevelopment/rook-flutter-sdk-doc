# Changelog

## 0.2.0

* Added `deleteUserFromRook`, this function will remove a user from both server and preferences,
  see [Removing registered users](README.md#removing-registered-users) for more information.
* Added StepsTracker to extract steps from phones, more information in the [Steps Tracker Documentation](STEPS_TRACKER.md)
* Added CaloriesTracker to extract steps from phones, more information in the [Calories Tracker Documentation](CALORIES_TRACKER.md)
* When registering a user the Apple Health data source status will be changed to active.
* When requesting permissions the Apple Health data source status will be changed to active.
* Changed all `clientPassword` instances to `secretKey`.

## 0.1.0

* Added `RookEnvironment` to configure internal behaviour, see [Environment](README.md#environment) for more information.
* Changed `RookConfiguration` constructor parameters, see [Initialize](README.md#initialize) for more information.

## 0.0.2

* Added Time Zone sync, see [Update userID](README.md#update-userid) for more information.

**Before running your app you MUST run `pod install` or `pod update` in your **/ios** directory**

## 0.0.1

* Initial release.
