# Introduction to Steel SDK usage for iOS

Welcome to the Steel SDK for iOS documentation. This SDK is designed to provide developers with a robust and efficient set of tools for integrating and utilizing the capabilities of the Steel platform within their iOS applications.

The Steel SDK encapsulates a wide range of functionalities, including device management, network communication, and peripheral handling, making it easier for developers to focus on creating innovative and user-friendly applications.

Whether you are developing an application that requires advanced network communication or you are building a new IoT solution, the Steel SDK for iOS provides a streamlined, efficient, and easy-to-use API that can help you achieve your goals.

In the following sections, we will guide you through the process of installing, configuring, and using the Steel SDK in your iOS projects. We will also provide you with detailed API references, usage examples, and troubleshooting tips to help you get the most out of the Steel SDK.

Steel SDK for iOS provides a set of instruments to interact with Steel server, recover the list of locks and keys and interact with smartlocks, sincronize PINs, control battery state and so on.
This guide illustrates basic principles on how to use SDK by code examples.

## SDK Example iOS application

You can find a demo application of the basic functions of Steel SDK in the repository

```
git clone git@bitbucket.org:alia/steel-sdk-ios-example.git
```

and this documentation gives an introduction to the operations available based on the example provided.

## Install dependencies

CocoaPods is a dependency manager for Swift and Objective-C projects. To import libraries into your project via CocoaPods, follow these steps:

1. **Install CocoaPods**: If you haven't already installed CocoaPods, you can do so by running the following command in your terminal:

   ```bash
   sudo gem install cocoapods
   ```

2. **Create a Podfile**: Navigate to your project directory in your terminal and create a Podfile by running the command:

   ```bash
   pod init
   ```

   This will create a file called `Podfile` in your project directory.

3. **Add libraries to your Podfile**: Open your Podfile in a text editor and add the libraries you want to import. For example, if you want to import the `SteelSDK`, `SteelFoundation`, `SteelNetwork`, `SteelBle`, and `SteelCore` libraries, your Podfile might look like this:

   ```ruby
   platform :ios, '13.0'
   use_frameworks!

   def steel_pods
       pod 'SteelSDK', :git => 'https://bitbucket.com/alia/steel-sdk-ios.git'
       pod 'SteelFoundation', :git => 'https://bitbucket.com/alia/steel-foundation-ios.git'
       pod 'SteelNetwork', :git => 'https://bitbucket.com/alia/steel-network-ios.git'
       pod 'SteelBle', :git => 'https://bitbucket.com/alia/steel-ble-ios.git'
       pod 'SteelCore', :git => 'https://bitbucket.com/alia/steel-core-ios.git'
   end

   target 'Example' do
       steel_pods
       pod 'OtherLibrary', :git => 'https://github.com/other-library'
       ... and so on ...
   end
   ```

   Replace `'Example'` with the name of your target.

4. **Install the libraries**: Save your Podfile and then run the following command in your terminal to install the libraries:

   ```bash
   pod install
   ```

   This command will download and install the libraries and their dependencies, and create a `.xcworkspace` file for your project.

5. **Open your project**: From now on, you will need to open the `.xcworkspace` file instead of the `.xcodeproj` file when working on your project. You can do this by running the following command in your terminal:

   ```bash
   open Example.xcworkspace
   ```

   Replace `'Example'` with the name of your project.

Now you should be able to import and use the libraries in your project. For example, in a Swift file, you might do:

```swift
import SteelSDK
```

And in an Objective-C file, you might do:

```objective-c
#import <SteelSDK/SteelSDK.h>
```

Remember to replace `'SteelSDK'` with the name of the library you want to import.


## SDK Operations

### Instantiate the SDK

Start by instantiating its shared instance like this:
```objective-c
    // retrieve the Singleton
    SteelSDK *steel = [SteelSDK sharedInstance];
```

and setup the instance
```objective-c
    // activate the SDK with API key and secret
    [steel provideAPIKey:@"xxxxxxxxxxxxxxxxxxxxxxx" APISecret:@"xxxxxxxxxxxxxxxxxxxxxxx"];
```

### Login to Alia cloud service

calling `loginWithUsername` enable the SDK to retrieve locks and auth codes info for proper operation. The callback in the example populates the ui with list of peripherals registered on cloud:

```objective-c
    // login user with API
    [steel loginWithUsername:@"user@domain.xxx" password:@"xxxxxxxxx" callback:^(BOOL success, User *user, ResponseObject *failureObject) {

        if (success) {

            if (steel.getPeripherals.count > 0) {
                [self reloadData];
            }

            // retrieve peripherals
            [steel getPeripheralsCallback:^(BOOL success, NSArray<Peripheral> *peripherals, ResponseObject *failureObject) {

                if (success) {

                    // populate UI
                    [self reloadData];
                }

            }];
        }

    }];
```

you can disable some checks
```objective-c
    // option: disable proximity check while opening the steel lock
    [STLPeripheralUsageManager sharedInstance].enableProximityCheck = NO;
```

and also alerts
```objective-c
    // option: disable Steel SDK alerts for access control check
    [STLPeripheralUsageManager sharedInstance].enableAlerts = NO;```
```

### BLE (Bluetooth)

#### Scanning / discovery

Start by instantiating its shared instance like this:
```objective-c
    STLCentralManager *centralManager = [STLCentralManager sharedInstance];
```

Bluetooth discovery uses a singleton instance of STLCentralManager from SteelBle library.

setup the central manager properties

```objective-c
    centralManager.monitorLostPeripherals = YES;
    centralManager.deviceIdentifiers = centralManager.allDeviceIdentifiers;
    centralManager.devicePreferences = @{
                                         kCentralModeBtcodeRootOption: @"4CE2F1",
                                         kCentralModeAutoScanOption: @NO,                    // scanner always on if possible (bluetooth turned on)
                                         kCentralModeScannerFrequencyOption: @0.1,           // scanner update frequency [0-10s] 0 = real-time
                                         kCentralModeSHA256ThresholdVersion: @3.0,
                                         kCentralModeAutoConnectOption: @NO,
                                         kCentralModeAutoAuthenticateOption: @YES,
                                         kCentralModeStopScanWhenConnectOption: @NO,         // should the scanner stop when connecting to a peripheral? then
                                         // when disconnected the scanner restarts if it was started before connection
                                         };
```

and initialize it
```objective-c
    [centralManager initialize];
```

you can customize the scanner frequency (s)
```objective-c
    _centralManager.scannerFrequency = @1;
```

and listen to state changes like BT activation and subsequent start scanning
```objective-c
    _centralManager.onDidUpdateState = ^(STLCentralManager *manager, CBManagerState state, CBManagerState previousState) {
        if (CBManagerStatePoweredOff == manager.state) {
            [myviewcontroller.view makeToast:@"Hey! turn on the bluetooth !!!"];
        } else if (![manager scanning]) {
            // finally, start the BLE scanner
            [manager startScan];
        }
    };
```

when a peripheral is discovered you can run your code to update a view.
The boolean **isNewPeripheral** when YES means that it's the first time the scanner finds it (since it was lost)
```objective-c
    _centralManager.onDidDiscoverPeripheral = ^(STLDiscoveredPeripheral *peripheral, BOOL isNewPeripheral) {
        if (myviewcontroller.isViewLoaded && myviewcontroller.view.window) {
            [myviewcontroller.tableView reloadData];
        }
    };
```

the same can be done when one or more periperals are lost
```objective-c
    _centralManager.onDidLostPeripherals = ^(NSArray *btcodes) {
        if (myviewcontroller.isViewLoaded && myviewcontroller.view.window) {
            [myviewcontroller.tableView reloadData];
        }
    };
```

you have also the option to register listeners to act on notifications like this
```objective-c
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(discoveryNotification:) name:kCentralManagerDidDiscoverPeripheralNotification object:nil];

    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(proximityNotification:) name:kCentralManagerDidChangeProximityNotification object:nil];

    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(lostNotification:) name:kCentralManagerDidLostPeripheralsNotification object:nil];
```

#### Actions

to operate on a smartlock device, you start from the address of device discovered and then get a Peripheral object to use on

**getPeripheralWithBtcode**
```objective-c
    NSString *btcode = centralManager.discoveredBtcodes[ indexOfDevice ];
    SteelSDK *steel = [SteelSDK sharedInstance];
    Peripheral *peripheral = [steel getPeripheralWithBtcode:btcode];
    if (peripheral) {
        ...
    }
```

**unlockPeripheral**
```objective-c
    [steel unlockPeripheral:peripheral evaluationCallback:^(BOOL success, Privilege *privilege, SteelActionError error, NSString *localizedErrorMessage) {
        if (!success) {
            [self.view makeToast:localizedErrorMessage];
        }
    } responseCallback:^(BOOL success, NSError *error) {

    }];
```

## Classes

### SteelSDK (steel-sdk-ios)

##### summary of methods

The `SteelSDK` class in the `SteelSDK.h` library of the Steel SDK for iOS contains several methods. Here's the full list of the methods:

1. `+ (SteelSDK * _Nonnull) sharedInstance`
2. `- (void) provideAPIKey:(NSString * _Nonnull)key APISecret:(NSString * _Nonnull)secret`
3. `- (void) provideCustomEndpoint:(NSString * _Nonnull)endpoint`
4. `- (void) provideCustomGroupTags:(NSArray<NSString*> * _Nonnull)groupTags`
5. `- (void) loginWithUsername:(NSString * _Nonnull)username password:(NSString * _Nonnull)password callback:(LoginCallback _Nullable)callback`
6. `- (void) provideAuthToken:(NSString * _Nonnull)authToken callback:(LoginCallback _Nullable)callback`
7. `- (User * _Nullable) getUser`
8. `- (void) getPeripheralsCallback:(PeripheralsCallback _Nullable)callback`
9. `- (NSArray<STLDiscoveredPeripheral*> * _Nonnull) getDiscoveredPeripherals`
10. `- (void) accessControlCheckForPeripheral:(Peripheral * _Nonnull)peripheral customDate:(NSDate * _Nullable)date callback:(PrivilegeEvaluationCallback _Nullable)callback`
11. `- (void) unlockPeripheral:(Peripheral * _Nonnull)peripheral evaluationCallback:(PrivilegeEvaluationCallback _Nullable)evaluationCallback responseCallback:(BluetoothResponseErrorCallback _Nullable)responseCallback`
12. `- (void) lockPeripheral:(Peripheral * _Nonnull)peripheral evaluationCallback:(PrivilegeEvaluationCallback _Nullable)evaluationCallback responseCallback:(BluetoothResponseErrorCallback _Nullable)responseCallback`
13. `- (void) initializeWithoutUser`
14. `- (NSArray<Peripheral * > * _Nonnull) getPeripherals`
15. `- (Peripheral * _Nullable) getPeripheralWithBtcode:(NSString * _Nonnull)btcode`
16. `- (void) setPeripherals:(Peripherals * _Nonnull)peripherals`

Please note that this is a high-level overview based on the header file. For a detailed understanding of the methods and their functionalities, you would typically refer to the source code of the SDK.

##### basic reference

Here are some details about a selection of the methods in the `SteelSDK` class:

1. `+ (SteelSDK * _Nonnull) sharedInstance`: This class method returns the shared instance of the `SteelSDK` class.

2. `- (void) provideAPIKey:(NSString * _Nonnull)key APISecret:(NSString * _Nonnull)secret`: This method is used to provide the API key and secret to the SDK.

3. `- (void) provideCustomEndpoint:(NSString * _Nonnull)endpoint`: This method is used to provide a custom API endpoint.

4. `- (void) provideCustomGroupTags:(NSArray<NSString*> * _Nonnull)groupTags`: This method is used to provide custom group tags.

5. `- (void) loginWithUsername:(NSString * _Nonnull)username password:(NSString * _Nonnull)password callback:(LoginCallback _Nullable)callback`: This method is used to log in a user with a username and password.

6. `- (void) provideAuthToken:(NSString * _Nonnull)authToken callback:(LoginCallback _Nullable)callback`: This method is used to provide an authorization token.

7. `- (User * _Nullable) getUser`: This method returns the currently logged-in user.

8. `- (void) getPeripheralsCallback:(PeripheralsCallback _Nullable)callback`: This method retrieves the user's peripherals (keys).

9. `- (NSArray<STLDiscoveredPeripheral*> * _Nonnull) getDiscoveredPeripherals`: This method retrieves the discovered Bluetooth peripherals.

10. `- (void) accessControlCheckForPeripheral:(Peripheral * _Nonnull)peripheral customDate:(NSDate * _Nullable)date callback:(PrivilegeEvaluationCallback _Nullable)callback`: This method checks if access can be granted to a peripheral at a specific time.

11. `- (void) unlockPeripheral:(Peripheral * _Nonnull)peripheral evaluationCallback:(PrivilegeEvaluationCallback _Nullable)evaluationCallback responseCallback:(BluetoothResponseErrorCallback _Nullable)responseCallback`: This method requests an unlock command on a peripheral.

12. `- (void) lockPeripheral:(Peripheral * _Nonnull)peripheral evaluationCallback:(PrivilegeEvaluationCallback _Nullable)evaluationCallback responseCallback:(BluetoothResponseErrorCallback _Nullable)responseCallback`: This method requests a lock command on a peripheral.

13. `- (void) initializeWithoutUser`: This method initializes the SDK without a user.

14. `- (NSArray<Peripheral * > * _Nonnull) getPeripherals`: This method returns the list of locks currently managed by the SDK.

15. `- (Peripheral * _Nullable) getPeripheralWithBtcode:(NSString * _Nonnull)btcode`: This method returns the peripheral that has a specific Bluetooth code.

16. `- (void) setPeripherals:(Peripherals * _Nonnull)peripherals`: This method sets the list of peripherals to be loaded inside the SDK.

For a complete understanding of what each method does, you would need to look at the implementation of each method in the source code.
