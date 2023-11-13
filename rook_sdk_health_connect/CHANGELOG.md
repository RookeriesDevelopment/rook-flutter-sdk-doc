# Changelog

## 0.2.1

* Optimized the number of calls to Health Connect required to extract Temperature Events.

## 0.2.0

### Android 14

This version will only compile against Android 14 (SDK 34). Please follow the steps to migrate

1. Update your gradle version to 8.1.2
2. Update your `compileSdk` and `targetSdk` to 34
3. Update your `sourceCompatibility`, `targetCompatibility` and `jvmTarget` to Java 17

### Changes

* Permissions functions have changed, see [Permissions](README.md#permissions) for more information.
* Privacy policy configuration has changed, see [Privacy policy](README.md#privacy-policy) for more information.
* Updated data access links, see [Privacy policy](README.md#request-data-access) for more information.
* Added new obfuscation rules, see [Obfuscation](README.md#obfuscation) for more information.

### Known issues

If you are using the flutter plugin [receive_intent](https://pub.dev/packages/receive_intent) you may encounter some
compilation errors, this is because receive_intent is not ready for java 17 and gradle 8+, this can be solved by editing
the receive_intent build.gradle file, add the following inside the `android` block:

```groovy
android {
  namespace "com.bhikadia.receive_intent"
  
  compileOptions {
    sourceCompatibility JavaVersion.VERSION_17
    targetCompatibility JavaVersion.VERSION_17
  }

  kotlinOptions {
    jvmTarget = JavaVersion.VERSION_17.toString()
  }
}
```

## 0.1.0

* Changed `setLocalLoggingLevel` to `enableNativeLogs`, see [Logging](README.md#logging) for more information.
* Added `RookEnvironment` to configure internal behaviour, see [Environment](README.md#environment) for more information.
* Changed `RookConfiguration` constructor parameters, see [Initialize](README.md#initialize) for more information.

## 0.0.2

* Added Time Zone sync, see [Update userID](README.md#update-userid) for more information.

## 0.0.1

* Initial release.
