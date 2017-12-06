# Local Setup

When you want to develop directly on the SDK source, you will want to create a link between your app and the SDK source code rather than installing the SDK with yarn.

## React-native Configuration

Create a file in your project root directory named `rn-cli.config.js`:

```javascript
const path = require('path');
const metroBundler = require('metro-bundler');

const config = {
  // where to find react and react-native
  extraNodeModules: {
    'react-native': path.resolve(__dirname, 'node_modules/react-native'),
    react: path.resolve(__dirname, 'node_modules/react')
  },
  // where to find the cancerbase sdk
  getProjectRoots() {
    return [
      path.resolve(__dirname), // your project, the default value
      CANCERBASE_SDK_PATH // <-- edit this to point to the sdk
    ];
  },
  getBlacklistRE: function() {
    // ignore the react and react-native modules inside the cancerbase-sdk
    return metroBundler.createBlacklist([
      /cancerbase-sdk\/node_modules\/react\/.*/,
      /cancerbase-sdk\/node_modules\/react-native\/.*/
    ]);
  }
};
module.exports = config;
```

The CancerBase SDK has a separate copy of react-native, which will cause duplicate modules in the react-native packager. This configuration tells the packager where to find the cancerbase sdk, react, and react-native.

Make sure to change the line above, replacing `CANCERBASE_SDK_PATH` with the path to the _folder containing the `cancerbase-sdk`_ folder. If you have a local copy of the `platform` repository, for example, this might look like `path.resolve(__dirname, '../platform/sdk/react-native')`.

## Android Configuration

You need to update the project directory in `settings.gradle` for the cancerbase-sdk.

Example project `settings.gradle`:
```gradle
rootProject.name = 'myapp'

include ':app'
include ':cancerbase-sdk'
project(':cancerbase-sdk').projectDir = new File(rootProject.projectDir, '../platform/sdk/react-native/cancerbase-sdk/android/')
```

The project directory is the android folder inside the cancerbase-sdk.

## iOS Configuration

To build iOS from source, if you've already linked your project with `libCancerBaseSDK.a`, remove this file from your project. Then you can add the CancerBaseSDK.xcodeproj to your project (or to your workspace if you have one) and link to the `libCancerBaseSDK.a` that this project generates. To link, go to your project settings, select `Build Phases`, find the section `Link Binary With Libraries`, click the `+` button, select `libCancerBaseSDK.a`, and click `Add`.
