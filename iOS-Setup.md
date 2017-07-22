# iOS Setup

After installing the iOS SDK, you will need to perform these steps one time only.  These steps configure Xcode to compile the CancerBase SDK and configure your app to identify itself with the CancerBase server.

1. Link the library.  This can usually be accomplished by running:

    ```bash
    react-native link
    ```

    If this doesn't work, you may want to try manually linking the library as described in the [react-native ios linking docs](https://facebook.github.io/react-native/docs/linking-libraries-ios.html).

    This configures Xcode to compile the CancerBase SDK along with your app.

1. Setup a URL handler

    OAuth uses URL redirects to complete authentication.  Your app needs a unique URL protocol so that once the user logs in to CancerBase, they can be redirected back into your app.

    The first step is to configure your application so that when it is opened from a URL, the CancerBaseSDK is notified.

    Open your application delegate (often `AppDelegate.m`)  and add the following lines.

    At the top of the file, add the following import:

    ```objective-c
    #import "CancerBaseSDK.h"
    ```

    Inside the app delegate (e.g. between `@implementation AppDelegate` and `@end`), add the following method:

    ```objective-c
    - (BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(NSString *)sourceApplication annotation:(id)annotation {
      return [CancerBaseSDK application:application
                                openURL:url
                      sourceApplication:sourceApplication
                             annotation:annotation];
    }
    ```

    If your app delegate already defines this method, you can modify it to call the `CancerBaseSDK` first and then conditionally call the other code:

    ```objective-c
    - (BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(NSString *)sourceApplication annotation:(id)annotation {
      if ([CancerBaseSDK application:application
                             openURL:url
                   sourceApplication:sourceApplication
                          annotation:annotation]) {
        return YES;
      }

      // existing code here
    }
    ```

2. Update `Info.plist`:

    Add your OAuth2 `cbClientId` and `cbClientSecret` to `Info.plist` and register your URL scheme, which is your `cbClientId` prefixed with the letters `cb`.  You may use Xcode's plist editor or right click `Info.plist` and choose `Open As --> Source Code`.

    Given a `cbClientId` of `123456` and a secret of `abcdef`, the plist editor should look something like this after modifications:

    ![Xcode view of updated Info.plist](https://user-images.githubusercontent.com/143985/28490013-649fdfc6-6e85-11e7-8254-e4cb34456763.png)

    and the source code view should look something like this:

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/  DTDs/PropertyList-1.0.dtd">
    <plist version="1.0">
    <dict>
      ...
      <key>cbClientSecret</key>
      <string>abcdef</string>
      <key>cbClientId</key>
      <string>123456</string>
      <key>CFBundleURLTypes</key>
      <array>
        <dict>
          <key>CFBundleURLSchemes</key>
          <array>
            <string>cb123456</string>
          </array>
        </dict>
      </array>
    ```

    This step identifies your app to the CancerBaseSDK and tells iOS which URL scheme your app should respond to.
