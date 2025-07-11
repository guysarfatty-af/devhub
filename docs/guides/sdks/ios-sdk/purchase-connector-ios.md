---
title: "Purchase connector"
slug: "purchase-connector-ios"
category: 5f9705393c689a065c409b23
parentDoc: 5fa043dd3b65b20045e35597
hidden: false
order: 10
---

## Overview

The AppsFlyer ROI360 purchase connector is used to validate and report in-app purchase and subscription revenue events. It’s part of the ROI360 in-app purchase and subscription revenue measurement solution.

- Using the purchase connector requires an ROI360 subscription.
- If you use this in-app purchase and subscription revenue measurement solution, you shouldn’t send [in-app purchase events](https://dev.appsflyer.com/hc/docs/in-app-events-ios) with revenue or execute [`validateAndLogInAppPurchase`](https://dev.appsflyer.com/hc/docs/validate-and-log-purchase-ios), as doing so results in duplicate revenue being reported.
- Before implementing the purchase connector, the ROI360 in-app purchase and subscription revenue measurement needs to be integrated with Google Play and the App Store. [See instructions (steps 1 and 2)](https://support.appsflyer.com/hc/en-us/articles/7459048170769)

## Prerequisites

- StoreKit SDK v1
- iOS version 9 and higher
- iOS AppsFlyer SDK **6.8.0** and higher

** ⚠️Important note: ** See the following table for purchase connector and AppsFlyer SDK version compatability and use the correct version to avoid unexpected behavior.

|  Purchase connector  | AppsFlyer SDK |
| :------: | :--------: |
| 6.8.0    | 6.8.0 - 6.9.2 |
| 6.8.1    | 6.8.0 - 6.9.2 |
| 6.10.0   |  6.10.0 |
| 6.10.1   |  6.10.1 |
| 6.12.2   |  6.12.2 |
| 6.12.3   |  6.12.2 |
| 6.17.0   |  6.17.0 |

## Install via Cocoapods

Add the following to your Podfile and run `pod install`:

```
pod 'PurchaseConnector'
```

## Install via Carthage 

1. Go to the `Carthage` folder in the root of the repository. 
2. Open `purchase-connector-dynamic.json` or `purchase-connector-static.json`.
3. Click **ra**.
4. Copy and paste one of the following file URLs to your `Cartfile`:

```
binary "https://raw.githubusercontent.com/AppsFlyerSDK/appsflyer-apple-purchase-connector/main/Carthage/purchase-connector-dynamic.json" == BIINARY_VERSION
binary "https://raw.githubusercontent.com/AppsFlyerSDK/AppsFlyerFramework/master/Carthage/appsflyer-ios.json" ~> 6.10.0
```

5. Open the project folder in the terminal and use the command `carthage update --use-xcframeworks`.
6. Drag and drop PurchaseConnector.xcframework binary and AppsFlyerLib.framework (from Carthage/Build/iOS folder).

[Learn more about Carthage binary artifacts integration](https://github.com/Carthage/Carthage/blob/master/Documentation/Artifacts.md).

## Install via SPM

Follow standard SPM dependency manager instructions.

> **Note**: 
> - This repository contains statically linked `PurchaseConnector.xcframework`. If you want to use dynamic .xcframework, integrate it for SPM from ththe following repository:
https://github.com/AppsFlyerSDK/PurchaseConnector-Dynamic*
> - PurchaseConnector has a dependency on [AppsFlyerLib framework](https://github.com/AppsFlyerSDK/AppsFlyerFramework), so make sure to integrate it as well for Carthage and SPM.

## Basic integration

> **Note**: Before implemening the purchase connector, make sure to set up AppsFlyer `appId` and `devKey`.

### Set up purchase connector

```swift
// Import the library
    import AppsFlyerLib
    import StoreKit
    import PurchaseConnector

// Default SDK Implementation
    AppsFlyerLib.shared().appsFlyerDevKey = "DEV_KEY"
    AppsFlyerLib.shared().appleAppID = "APPLE_APP_ID"
    //AppsFlyerLib.shared().isDebug = true

// Purchase connector implementation 
    PurchaseConnector.shared().purchaseRevenueDelegate = self
    PurchaseConnector.shared().purchaseRevenueDataSource = self
```
```objectivec
// Import the library
    #import "AppDelegate.h"
    #import <AppsFlyerLib/AppsFLyerLib.h>
    #import <PurchaseConnector/PurchaseConnector.h>

// Default SDK implementation
    [[AppsFlyerLib shared] setAppleAppID:@"APPLE_APP_ID"];
    [[AppsFlyerLib shared] setAppsFlyerDevKey:@"DEV_KEY"];
    //[[AppsFlyerLib shared] setIsDebug:YES];

// Purchase Connecor implementation
    [[PurchaseConnector shared] setPurchaseRevenueDelegate:self];
    [[PurchaseConnector shared] setPurchaseRevenueDataSource:self];
```

### Log auto-renewable subscriptions and in-app purchases

Enables automatic logging of auto-renewable subscriptions and in-app purchases.

```swift
PurchaseConnector.shared().autoLogPurchaseRevenue = [.autoRenewableSubscriptions, .inAppPurchases]
```
```objectivec
[[PurchaseConnector shared] setAutoLogPurchaseRevenue:AFSDKAutoLogPurchaseRevenueOptionsRenewable | AFSDKAutoLogPurchaseRevenueOptionsInAppPurchases];
```

> **Note**: If `autoLogPurchaseRevenue` hasn't been set, it is disabled by default. The value is an option set, so you can choose what kind of user purchases you want to observe.

### Conform to purchase connector data source and delegate protocols

- To receive purchase validation event callbacks, you should conform to and implement `PurchaseRevenueDelegate`(Swift) or `AppsFlyerPurchaseRevenueDelegate`(Objc-C) protocol.
- To be able to add your custom parameters to the purchase events that the purchase connector sends, conform to  and implement `PurchaseRevenueDataSource`(Swift) or `AppsFlyerPurchaseRevenueDataSource`(Obj-C) protocol.

```swift
extension AppDelegate: PurchaseRevenueDataSource, PurchaseRevenueDelegate {
    // PurchaseRevenueDelegate method implementation
    func didReceivePurchaseRevenueValidationInfo(_ validationInfo: [AnyHashable : Any]?, error: Error?) {
        print("PurchaseRevenueDelegate: \(validationInfo)")
        print("PurchaseRevenueDelegate: \(error)")
      // process validationInfo here 
}
    // PurchaseRevenueDataSource method implementation
    func purchaseRevenueAdditionalParameters(for products: Set<SKProduct>, transactions: Set<SKPaymentTransaction>?) -> [AnyHashable : Any]? {
        // Add additional parameters for SKTransactions here.
        return ["additionalParameters":["param1":"value1", "param2":"value2"]];
    }
}
```
```objectivec
@interface AppDelegate () <AppsFlyerPurchaseRevenueDelegate, AppsFlyerPurchaseRevenueDataSource>
@end

@implementation AppDelegate

- (void)applicationDidBecomeActive:(UIApplication *)application {
    [[AppsFlyerLib shared] start];
    [[PurchaseConnector shared] startObservingTransactions];
}

- (void)didReceivePurchaseRevenueValidationInfo:(NSDictionary *)validationInfo error:(NSError *)error {
    NSLog(@"Validation info: %@", validationInfo);
    NSLog(@"Error: %@", error);
    
    // Process validation info
}

- (NSDictionary *)purchaseRevenueAdditionalParametersForProducts:(NSSet<SKProduct *> *)products transactions:(NSSet<SKPaymentTransaction *> *)transactions {
    return @{@"key1" : @"param1"};
}

@end
```
The `didReceivePurchaseRevenueValidationInfo` function receives validation response for each transaction from AppsFlyer in real-time and processes the results.

#### Validation response parameters

The response contains two parameters:

- `validationInfo`:  Value indicating whether the purchase was successfully validated.
- `error`: Provides error information if the validation fails. The error object includes::
    - `status`: Specifies the error code from AppsFlyer or the App Store (e.g., `21003`).
    - `is_retryable`: A legacy key that always returns `false`.
    - `store_status`: (**New)** - Provides a specific status code from Apple's validation response (e.g., `4040010`).

> 📘 Note
> 
> The `store_status` parameter is available only if your app has been updated to use Apple’s new validation API.
> 
> - For **existing apps**, `store_status` becomes available after you provide your App Store Connect In-App Purchase key in the AppsFlyer platform.
> - For **new apps**, `store_status` is included by default.

The following table specifies the different status codes:

**Note**: The **Description** and **App Store Error code** values are not exposed in the validation error response

| **status** | **store_status** | **App Store Error Code** | **Description** |
| --- | --- | --- | --- |
| 21003 | 4000002 | [InvalidAppIdentifierError](https://developer.apple.com/documentation/appstoreserverapi/invalidappidentifiererror) | Invalid request app identifier. |
| 21003 | 4040003 | [AppNotFoundError](https://developer.apple.com/documentation/appstoreserverapi/appnotfounderror) | App not found. |
| 21003 | 4040004 | [AppNotFoundRetryableError](https://developer.apple.com/documentation/appstoreserverapi/appnotfoundretryableerror) | App not found. Please try again. |
| 21003 | 4040010 | [TransactionIdNotFoundError](https://developer.apple.com/documentation/appstoreserverapi/transactionidnotfounderror) | Transaction ID not found. |
| 21009 | 5000000 | [GeneralInternalError](https://developer.apple.com/documentation/appstoreserverapi/generalinternalerror) | An unknown error occurred. |
| 21009 | 5000001 | [GeneralInternalRetryableError](https://developer.apple.com/documentation/appstoreserverapi/generalinternalretryableerror) | An unknown error occurred. Please try again. |
| 21010 | 4040001 | [AccountNotFoundError](https://developer.apple.com/documentation/appstoreserverapi/accountnotfounderror) | Account not found. |
| 21010 | 4040002 | [AccountNotFoundRetryableError](https://developer.apple.com/documentation/appstoreserverapi/accountnotfoundretryableerror) | Account not found. Please try again. |
| 10001 | 4000006 | [InvalidTransactionIdError](https://developer.apple.com/documentation/appstoreserverapi/invalidtransactioniderror) | Invalid transaction ID. |
| 20003 | - | - | Authentication credentials are invalid. |

### Start observing transactions

To observe transactions, you need to call `startObservingTransactions`.
 > **Note**: Call this right after the AppsFlyer iOS SDK start method.

```swift
    PurchaseConnector.shared().startObservingTransactions()
```
```objectivec
    [[PurchaseConnector shared] startObservingTransactions];
```

### Stop observing transactions

To stop observing transactions, you need to call `stopObservingTransactions`.

```swift
    PurchaseConnector.shared().stopObservingTransactions()
```
```objectivec
    [[PurchaseConnector shared] stopObservingTransactions];
```

> **Note**: If you called `stopObservingTransactions` API, set the `autoLogPurchaseRevenue` value before you call `startObservingTransactions` next time.

## Test the implementation in Sandbox

To test purchases in an Xcode environment on a real device with a TestFlight sandbox account, set `isSandbox` to `true`.

```swift
    PurchaseConnector.shared().isSandbox = true
```
```objectivec
    [[PurchaseConnector shared] setIsSandbox:YES];
```

> **Important note**: Before releasing your app to production, be sure to remove `isSandbox` or set it to `false`. If the production purchase events are sent in sandbox mode, your events will not be validated properly.

## Full code examples

```swift
import AppsFlyerLib
import StoreKit
import PurchaseConnector

class AppDelegate: UIResponder, UIApplicationDelegate {
   func application(_ _: UIApplication, didFinishLaunchingWithOptions _: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
    // Default SDK Implementation
        AppsFlyerLib.shared().appsFlyerDevKey = "DEV_KEY"
        AppsFlyerLib.shared().appleAppID = "APLE_APP_ID"
        // AppsFlyerLib.shared().isDebug = true
      
   // Purchase Connector implementation
        PurchaseConnector.shared().purchaseRevenueDelegate = self
        PurchaseConnector.shared().purchaseRevenueDataSource = self
        PurchaseConnector.shared().autoLogPurchaseRevenue = .autoRenewableSubscriptions
   }

   func applicationDidBecomeActive(_ application: UIApplication) {
        AppsFlyerLib.shared().start()
        PurchaseConnector.shared().startObservingTransactions()
    }
}

extension AppDelegate: PurchaseRevenueDataSource, PurchaseRevenueDelegate {
    // PurchaseRevenueDelegate method implementation
    func didReceivePurchaseRevenueValidationInfo(_ validationInfo: [AnyHashable : Any]?, error: Error?) {
        print("PurchaseRevenueDelegate: \(validationInfo)")
        print("PurchaseRevenueDelegate: \(error)")
      // process validationInfo here 
}
    // PurchaseRevenueDataSource method implementation
    func purchaseRevenueAdditionalParameters(for products: Set<SKProduct>, transactions: Set<SKPaymentTransaction>?) -> [AnyHashable : Any]? {
        // Add additional parameters for SKTransactions here.
        return ["additionalParameters":["param1":"value1", "param2":"value2"]];
    }
}
```
```objectivec
#import "AppDelegate.h"
#import <PurchaseConnector/PurchaseConnector.h>
#import <AppsFlyerLib/AppsFLyerLib.h>

@interface AppDelegate () <AppsFlyerPurchaseRevenueDelegate, AppsFlyerPurchaseRevenueDataSource>

@end

@implementation AppDelegate


- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    
    // Set up AppsFlyerLib first
    [[AppsFlyerLib shared] setAppleAppID:@"APPLE_APP_ID"];
    [[AppsFlyerLib shared] setAppsFlyerDevKey:@"DEV_KEY"];
    // [[AppsFlyerLib shared] setIsDebug:YES];
    
    // Set up PurchaseConnector
    [[PurchaseConnector shared] setPurchaseRevenueDelegate:self];
    [[PurchaseConnector shared] setPurchaseRevenueDataSource:self];
    [[PurchaseConnector shared] setAutoLogPurchaseRevenue:AFSDKAutoLogPurchaseRevenueOptionsAutoRenewableSubscriptions];
    return YES;
}

- (void)applicationDidBecomeActive:(UIApplication *)application {
     [[AppsFlyerLib shared] start];
     [[PurchaseConnector shared] startObservingTransactions];
}

- (void)didReceivePurchaseRevenueValidationInfo:(NSDictionary *)validationInfo error:(NSError *)error {
    NSLog(@"Validation info: %@", validationInfo);
    NSLog(@"Error: %@", error);
    
    // Process validation info
}

- (NSDictionary *)purchaseRevenueAdditionalParametersForProducts:(NSSet<SKProduct *> *)products transactions:(NSSet<SKPaymentTransaction *> *)transactions {
    return @{@"key1" : @"param1"};
}

@end
```
