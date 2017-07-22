1. After installing the SDK, you will need to link the library.  This can usually be accomplished by running:

    ```bash
    react-native link
    ```

    If you already ran this for iOS setup, you do not need to run it again.

1. Configuring the application protocol

    Open your application's `android/app/build.gradle` file.  You should see something like:

    ```gradle
    android {
        compileSdkVersion 25
        buildToolsVersion '25.0.2'

        defaultConfig {
            applicationId "com.example"
            minSdkVersion 16
            targetSdkVersion 25
            versionCode 1
            versionName "1.0"
            ndk {
                abiFilters "armeabi-v7a", "x86"
            }
        }
        ...
    }
    ```
    Inside the `defaultConfig` block, add your CancerBase application id prefixed with `cb` as shown below:

            manifestPlaceholders = [
                'appAuthRedirectScheme': 'cbAPPLICATION_ID_HERE'
            ]

     For example, if your application id was `123456`, your `app/build.gradle` would now look like:

    ```gradle
    android {
        compileSdkVersion 25
        buildToolsVersion '25.0.2'

        defaultConfig {
            applicationId "com.example"
            minSdkVersion 16
            targetSdkVersion 25
            versionCode 1
            versionName "1.0"
            ndk {
                abiFilters "armeabi-v7a", "x86"
            }
            manifestPlaceholders = [
                'appAuthRedirectScheme': 'cb123456'
            ]
        }
        ...
    }
    ```
