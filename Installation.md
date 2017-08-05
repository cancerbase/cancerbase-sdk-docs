# Installation of the CancerBase SDK

We recommend app developers use the CancerBase SDK to communicate with our
servers, which wraps up all of internal details in a react- native JavaScript
API.  We also provide REST API access to data, secured with OAuth2.

In your react-native project's main folder, run:

    yarn add cancerbase-sdk

When a new SDK version is released, you can upgrade `cancerbase-sdk` by
running:

    yarn upgrade --latest cancerbase-sdk

If there is a major version release that is not backwards compatible, you may
want to postpone the upgrade and instead upgrade to the latest patch version
by omitting the `--latest` flag.  In general, however, we recommend always
running the latest version.
