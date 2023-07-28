# Introduction to Steel SDK for Android

Welcome to the Steel SDK for Android documentation. This SDK is designed to provide developers with a robust and efficient set of tools for integrating and utilizing the capabilities of the Steel platform within their Android applications.

The Steel SDK encapsulates a wide range of functionalities, including device management, network communication, and peripheral handling, making it easier for developers to focus on creating innovative and user-friendly applications.

Whether you are developing an application that requires advanced network communication or you are building a new IoT solution, the Steel SDK for Android provides a streamlined, efficient, and easy-to-use API that can help you achieve your goals.

In the following sections, we will guide you through the process of installing, configuring, and using the Steel SDK in your Android projects. We will also provide you with detailed API references, usage examples, and troubleshooting tips to help you get the most out of the Steel SDK.

## SDK Example Android application

You can find a demo application of the basic functions of Steel SDK in the repository

```
git clone git@bitbucket.org:alia/steel-sdk-android-example.git
```

and this documentation gives an introduction to the operations available based on the example provided.

# System Requirements

Before you start integrating the Steel SDK into your Android project, it's important to ensure that your development environment meets the following system requirements:

1. **Operating System**: Windows, Mac OS, or Linux.

2. **Java Development Kit (JDK)**: JDK 8 or higher is required for Android development.

3. **Android Studio**: The latest stable version of Android Studio is recommended for the best development experience.

4. **Android SDK**: Android SDK with API level 21 (Android 5.0, Lollipop) or higher. The Steel SDK is designed to work with these versions of Android and may not be compatible with earlier versions.

5. **Gradle**: The latest version of Gradle is recommended to ensure the smooth building of your Android project.

6. **Device or Emulator**: An Android device running Android 5.0 (API level 21) or higher, or an emulator set up with equivalent specifications.

Please make sure that your system meets these requirements before proceeding with the installation and configuration of the Steel SDK.

# Installation and Configuration

Integrating the Steel SDK into your Android project involves a few steps. Here's a step-by-step guide to help you through the process:

## Add the SDK Dependency

First, you need to add the Steel SDK as a dependency in your project. This can be done by adding the following line to your `build.gradle` file under the `dependencies` section:

```gradle
dependencies {
    implementation 'steel-sdk-android:1.0.0' // replace 1.0.0 with the actual version
    implementation 'steel-base-android:1.0.0'
    implementation 'steel-network-android:1.0.0'
    implementation 'steel-ble-android:1.0.0'
    implementation 'steel-sdk-realm-android:1.0.0'
}
```

Make sure to replace `1.0.0` with the actual version of the Steel SDK that you want to use.

## Sync Your Project

After adding the dependency, you need to sync your project. You can do this by clicking on the `Sync Now` button that appears in the top right corner of the Android Studio interface.

## SDK Operations

### Initialize the SDK

Once the project sync is complete, you can initialize the Steel SDK in your application. This is typically done in the `onCreate` method of your `Application` class:

```java
import SteelSDK;

public class MyApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
        SteelSDK.init(this, STEEL_API_DOMAIN, ACCOUNT_CODE));
    }
}
```

Where
| STEEL_API_DOMAIN  | "https://api.mrn.ovh"   |
|-------------------|-------------------------|
| ACCOUNT_CODE      | "it.meroni.alia.account"|

Make sure to replace `MyApplication` with the actual name of your `Application` class.

### Instantiate the SDK

Depending on your application's needs, you need to get the singleton instance of the Steel SDK using the `SteelSDK.getInstance()` method.

```java
//Retrieve STEEL SDK instance (singleton)
SteelSDK steel = SteelSDK.getInstance();
```

and initialize it
```java
steel.initializeWithoutUser();
```

and terminate it at the end of application lifecycle
```java
steel.terminate();
```

That's it! You have now successfully integrated the Steel SDK into your Android project. You can now start using the various features and functionalities provided by the SDK.

### Locks

Here is a working code example to manage locks in SteelSDK.
- `steelPeripheral` is the instance of SDK class
- `peripheralsAdapter` is an array adapter for a ListView

```java
    Log.i(TAG, "1. get SteelSDK singleton instance");
    steelPeripheral = SteelSDK.getInstance();
```

<!-- still work in progress
provide application id/secret:
```java
    Log.i(TAG, "2. activate SDK with project id/secret");
    String projectKey = "1234567890";
    String projectSecret = "1234567890abcdef1234567890abcdef";
    steelPeripheral.provideAPIKey(this, projectKey, projectSecret);
```
 -->

manage existing peripherals in the SDK:
```java
    // get already stored keys from SDK
    List<Peripheral> peripherals = steelPeripheral.getPeripherals();
    if (peripherals.size() > 0) {
        peripheralsAdapter.addAll(peripherals);
        peripheralsAdapter.notifyDataSetChanged();
    }
```

check BLE permissions for the app using `STLAndroidPermissionsManager` class
```java
    Log.i(TAG, "3. request location GPS permission (ble needs it)");
    STLAndroidPermissionsManager.checkLocationPermissions(this, success -> {

    });
```

login in Alia cloud API and retrieve locks info on success using a callback:
```java
    Log.i(TAG, "4. login with username / password");

    STLFacade.getInstance().setAccountType(getString(R.string.STEEL_ACCOUNT_TYPE));

    String username = "someuser@steel.com";
    String password = "somepassword";
    steelPeripheral.loginWithUserNamePassword(username, password, (success, user, responseObject) -> {
        if (success) {

            Log.i(TAG, "5. get user peripherals");
            steelPeripheral.getPeripheralsCallback(this, (success1, peripherals1, responseObject1) -> {
                if (success1) {

                    Log.i(TAG, "6. populate list view");
                    peripheralsAdapter.clear();
                    peripheralsAdapter.addAll(peripherals1);
                    peripheralsAdapter.notifyDataSetChanged();

                } else {
                    Toast.makeText(LocksActivity.this, "get peripherals error", Toast.LENGTH_LONG).show();
                }
            });

        } else {
            Toast.makeText(LocksActivity.this, "login error", Toast.LENGTH_LONG).show();
        }
    });
```
example is taken from `LocksActivity.java` in `steel-sdk-example-android`

### BLE (Bluetooth)

Here is a working code example to manage BLE in SteelSDK.

#### Scanning / discovery

Setup
```java
        // enable the use of Proximity Scan in SDK
        STLAppSettings appSettings = STLSettingsUtilities.getInstance().getAppSettings();
        appSettings.setUseProximity(true);

        // ask the use to grant location permission if needed
        STLAndroidPermissionsManager.checkLocationPermissions(this);
```

Start scanning
```java
    private void startScan() {
        STLBluetoothScannerManager scannerManager = STLBluetoothScannerManager.getInstance();
        scannerManager.setBluetoothScannerListener(this);

        // here you start the scan
        // you can pass
        // 1) true: only peripherals found in SDK, setted with setPeripherals(), will be shown
        // 2) false: every STEEL peripheral will be shown
        scannerManager.startScan(false);
    }
```

Stop scanning
```java
    private void stopScan() {
        STLBluetoothScannerManager scannerManager = STLBluetoothScannerManager.getInstance();
        scannerManager.stopScan(false, () -> Log.i(TAG, "scanner stopped"));
    }
```

Check results
```java
    // if you have a Peripheral model:
    Peripheral somePeripheralToCheck = null;
    SteelPeripheral discovered = STLBluetoothScannerManager.getDiscoveredPeripheralWithPeripheral(somePeripheralToCheck);
    if (null != discovered && discovered.isDiscovered()) {
        // FOUND
    }

    // if you only have the BTCODE identifier of the peripheral...
    String btcode = null;
    STLDiscoveredPeripheral discovered2 = STLBluetoothScannerManager.getInstance().getDiscoveredPeripheral(btcode);
    if (null != discovered && discovered.isDiscovered()) {
        // FOUND
    }
```
example is taken from `BtScanActivity.java` in `steel-sdk-example-android`

#### Installation

The first installation of a new smart lock

setup the SDK
```java
// get the instance to SDK
steel = SteelSDK.getInstance();

// initialize SDK without a logged user
steel.initializeWithoutUser();

// set OAUTH2 Bearer token
steel.setOauth2BearerToken("<---- SOME TOKEN --->");
```

start scanning BLE devices and get notified for discovery
```java
// set this activity to be notified about BLE Scanner results
steel.setBluetoothScannerListener(this);
```

checking also permissions on the phone
```java
// ask the use to grant location permission if needed
if (!STLAndroidPermissionsManager.checkLocationPermissions(this)) {

    // location permission is granted, proceed with starting BLE scanner
    steel.startScan();
} else {
    // a request for granting the permission is automatically sent to the user
}
```

you need the Bluetooth Code of device from scanning on the next step to call `getPeripheralWithBtcode()`

```java
selectedBtcode = btcode;

// extract peripheral from SDK
Peripheral peripheral = steel.getPeripheralWithBtcode(btcode);
```

if peripheral is not already loaded in SDK you can request info to the cloud via `STLFacade` class:
```java
if (null == peripheral) {

    // request peripheral informations

    STLFacade.getInstance().getPeripheralCallback(selectedBtcode, (success, response, responseResult) -> {
        if (success) {
            // load the peripheral inside the SDK to initialize it
            steel.addPeripheral(response);
        } else {
            // show what went wrong, response object has "error_code" and "error_message" fields
            AlertUtils.sendServerResponseAlert(responseResult, "server error", InstallationBasicActivity.this);
        }
    });
}
```

to get peripheral infos, you can alternatively call Alia Cloud API like this:
```java
// you can also make a server to server API call:
// GET https://<api domain>/peripherals/{selectedBtcode}
// headers:
// Authorization: token {{token}}
// Content-Type: application/json

// then you use this code to add the peripheral to the SDK, converting from json to Peripheral

String json = "<the json response>";
Peripheral peripheral = STLFacade.gson.fromJson(json, Peripheral.class);
steel.addPeripheral(peripheral);
```

#### identification
you can ask to identify devices that were discovered:
```java
final SteelPeripheral steelPeripheral = steel.getSteelPeripheral(selectedBtcode);
steelPeripheral.sendIdentify();
```

#### connection and unlock
or test them by making a connection and operating lock:
```java
final SteelPeripheral steelPeripheral = steel.getSteelPeripheral(selectedBtcode);
        steelPeripheral.connectAuthCallback(new BluetoothResponseCallback() {
            @Override
            public void callback(boolean success, @Nullable BluetoothResponseException exception) {
                steelPeripheral.operateCommandCallback(SteelCommand.Impulse, new BluetoothResponseCallback() {
                    @Override
                    public void callback(boolean success, @Nullable BluetoothResponseException exception) {
                        steelPeripheral.disconnect();
                    }
                });
            }
        });
```

example is taken from `InstallationBasicActivity.java` in `steel-sdk-example-android`



## Classes

### SteelSDK (steel-sdk-android)

##### summary of methods

The `SteelSDK` class in the `com.steel.steelsdk` package of the Steel SDK for Android contains several methods. Here's the full list of the methods:

- `getInstance()`
- `init(@NonNull Context context, @NonNull String apiDomain, @Nullable String authenticatorAccountName)`
- `init(@NonNull Context context, @NonNull String apiDomain, @Nullable String authenticatorAccountName, String[] acceptableGroupTags)`
- `terminate()`
- `getUser()`
- `getGson()`
- `getPeripherals()`
- `getDiscoveredPeripherals()`
- `setPeripherals(@NonNull Context context, @NonNull Peripherals peripherals)`
- `addPeripheral(Peripheral peripheral)`
- `getPeripheralWithBtcode(@NonNull String btcode)`
- `provideAPIKey(@NonNull Context context, @NonNull String key, @NonNull String secret)`
- `provideAuthToken(@NonNull String authToken, final ResponseCallback<AuthToken> loginCallback)`
- `provideCustomEndpoint(@NonNull String endpoint)`
- `provideCustomGroupTags(@NonNull GroupTags[] groupTags)`
- `setAppInBackground(final boolean inBackground)`
- `initializeWithoutUser()`
- `setOauth2BearerToken(@NonNull String token)`
- `loginWithUserNamePassword(final String username, final String password, final LoginCallback loginCallback)`
- `logout()`
- `clear()`
- `getPeripheralsCallback(@NonNull Context context, final @Nullable PeripheralsCallback peripheralsCallback)`
- `getPeripheralsCallback(@NonNull Context context, boolean differentialUpdate, final @Nullable PeripheralsCallback peripheralsCallback)`
- `accessControlCheckWithKey(@NonNull Context context, @NonNull Peripheral peripheral, @NonNull DateTime time, @Nullable final AccessControlCallback callback)`
- `openPeripheral(@NonNull Context context, @NonNull Peripheral peripheral, @Nullable final AccessControlCallback callback, @Nullable final BluetoothResponseCallback bleCallback)`
- `commandPeripheral(@NonNull STLPeripheralUsage usage, @Nullable final AccessControlCallback callback, @Nullable final BluetoothResponseCallback bleCallback)`
- `startScan()`
- `stopScan()`
- `addBleScanListener(@NonNull DiscoveredPeripheralsObserver listener)`
- `removeBleScanListener(@NonNull BluetoothScannerResults listener)`
- `setBluetoothScannerListener(@NonNull STLBluetoothScannerManager.BluetoothScannerListener listener)`
- `unsetBluetoothScannerListener()`
- `getSteelPeripheral(@NonNull String btcode)`

For a complete and accurate documentation, please refer to the official Steel SDK documentation or the source code of the SDK.

##### basic reference

Here are some details about a selection of the methods in the `SteelSDK` class:

- `getInstance()`: This is a standard singleton method that returns the instance of the `SteelSDK` class. If the instance doesn't exist, it creates one.

- `init(@NonNull Context context, @NonNull String apiDomain, @Nullable String authenticatorAccountName)`: This method initializes the SteelSDK with a context, API domain, and an optional account name. It sets up default acceptable group tags for all projects.

- `getUser()`: This method retrieves the user model that was populated by the `loginWithUsername()` method.

- `getPeripherals()`: This method returns a list of Peripherals (Locks) currently loaded in the SDK.

- `provideAPIKey(@NonNull Context context, @NonNull String key, @NonNull String secret)`: This method initializes the SDK using a project id, API token, and secret. It checks if BLE is supported and sets up the Bluetooth steel framework.

- `loginWithUserNamePassword(final String username, final String password, final LoginCallback loginCallback)`: This method logs a user in with their username (email or phone number) and password. If the user is logged in, the SDK statefully remembers the user and keeps using their data.

- `logout()`: This method logs out the user and then clears the SDK of user token and data.

- `getPeripheralsCallback(@NonNull Context context, final @Nullable PeripheralsCallback peripheralsCallback)`: This method retrieves the logged user peripherals (keys).

- `openPeripheral(@NonNull Context context, @NonNull Peripheral peripheral, @Nullable final AccessControlCallback callback, @Nullable final BluetoothResponseCallback bleCallback)`: This method requests an open command on a peripheral. It performs an access control check with the current timestamp before the request is sent. If the access control check fails, the open command is not issued.

- `startScan()`: This method starts the Bluetooth scan.

- `stopScan()`: This method stops the Bluetooth scan.

- `getSteelPeripheral(@NonNull String btcode)`: This method returns a `SteelPeripheral` object for a given Bluetooth code.

For a complete understanding of what each method does, you would need to look at the implementation of each method in the source code.

### SteelBle (steel-ble-android)

This library contains Bluetooth protocol code used to manage devices

### SteelNetwork (steel-network-android)

This library contains Network protocol code used to access REST APIs on Alia Cloud servers and retrieve informations on managed devices

### SteelBase (steel-base-android)

This library contains common classes and utilities used in other libraries

## Notes

On Android it is possible to connect to a device without scanning first with the library, while iOS does not allow it.

The application, using Apple CoreBluetooth must discovery the device first and build a reference to it. For this reason on Steel for iOS the call to getOrRestorePeripheral will always return "null" without first scanning.
