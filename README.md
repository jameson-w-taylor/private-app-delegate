# AppDelegate Test
There has been a known issue with `AppDelegate` being taken by Capacitor for Cordova compatibility. Currently there is a simple workaround, and that is to simply use a different name in your own project. Recently it was mentioned that there may be a potential solution for this by marking `AppDelegate` as private.

This project is a sample to test out that theory with.
It was created using the Ionic CLI and the blank starter template in order to replicate this issue using the development process that the large majority of the Capacitor community follows. It was then modified to define it's own obj-c `AppDelegate` in order to create the naming conflict issue.

A Cordova plugin that relies on `AppDelegate` ([cordova-plugin-app-event](https://github.com/katzer/cordova-plugin-app-event)) is included as a smoke test to see if these changes break the Cordova compatibility in Capacitor. This plugin did have some compilation issues in Xcode unrelated to this test, so there are directions to comment out the offending code.

The instructions below will guide you to make the necessary changes locally to modify Capacitor to mark the `AppDelegate` as private. 

## Setup
1. Run `npm install`
2. Modify the cordova-plugin-app-event `node_modules`
    - This plugin has some compilation issues related to local notifications, which are not needed to perform this test
    - Comment out these sections of code in `node_modules/cordova-plugin-app-event/src/ios/AppDelegate+APPAppEvent.m`:
    ```c
    //#if CORDOVA_VERSION_MIN_REQUIRED >= 40000
    //    [self exchange_methods:@selector(application:didReceiveLocalNotification:)
    //                  swizzled:@selector(swizzled_application:didReceiveLocalNotification:)];
    //#endif


    // #if CORDOVA_VERSION_MIN_REQUIRED >= 40000
    // /**
    //  * Repost all local notification using the default NSNotificationCenter so
    //  * multiple plugins may respond.
    //  */
    // - (void)   swizzled_application:(UIApplication*)application
    //     didReceiveLocalNotification:(UILocalNotification*)notification
    // {
    //     // re-post (broadcast)
    //     [self postNotificationName:CDVLocalNotification object:notification];
    //     // This actually calls the original method over in AppDelegate
    //     [self swizzled_application:application didReceiveLocalNotification:notification];
    // }
    // #endif
    ```
    - Comment out these sections of code in `node_modules/cordova-plugin-app-event/src/ios/CDVPlugin+APPAppEvent.m`:
    ```c
    // #ifdef __CORDOVA_5_0_0
    // NSString* const CDVLocalNotification = @"CDVLocalNotification";
    // #endif


    // [self addObserver:NSSelectorFromString(@"didReceiveLocalNotification:")
    //              name:CDVLocalNotification
    //            object:NULL];
    ```
    
3. Modify the Capacitor `node_modules`
    1. Create a new folder `node_modules/@capacitor/ios/CapacitorCordova/CapacitorCordova/Classes/Private`
    2. Move `node_modules/@capacitor/ios/CapacitorCordova/CapacitorCordova/Classes/Public/AppDelegate.h` into the new `Private` folder
    3. Move `node_modules/@capacitor/ios/CapacitorCordova/CapacitorCordova/Classes/Public/AppDelegate.m` into the new `Private` folder
    4. Comment out the `<Cordova/AppDelegate.h>` import in `node_modules/@capacitor/ios/CapacitorCordova/CapacitorCordova/CapacitorCordova.h`
    5. Add the following to `node_modules/@capacitor/ios/CapacitorCordova.podspec`:
    ```
    s.private_header_files = "#{prefix}CapacitorCordova/CapacitorCordova/Classes/Private/*.h"
    ```
4. Run `ionic cap sync ios`
5. Open Xcode and build/deploy the app to a simulator.
6. Once the app launches, you'll see in the Xcode output a message similar to the example below which indicates the `AppDelegate` naming issue is still present.
```
Class AppDelegate is implemented in both FrameworkA and FrameworkB. One of the two will be used. Which one is undefined.
```