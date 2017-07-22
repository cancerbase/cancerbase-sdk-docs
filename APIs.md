# User Accounts

To authenticate one of your users with CancerBase, we provide a react-native
`LoginButton` component.

```javascript
import CancerBaseSDK, { LoginButton } from 'cancerbase-sdk';

// define what cancerbase data you would like access to
const MY_APP_SCOPES = [
  'cb.appData.read',
  'cb.profile',
  'cb.timeline'
];

class MainScreen extends React.Component {
  // callback for login errors
  onError = error => {

  };

  // callback for login success
  onLogin = () => {
    CancerBaseSDK.user.getProfile()
      .then(profile => { ... });
  };

  render() {
    return (
      <LoginButton
        scope={MY_APP_SCOPES}
        onLogin={this.onLogin}
        onError={this.onError}
      />;
    );
  }
}
```

The CancerBase user object has the following structure:

```typescript
{
  firstName : String,
  lastName : String,
  userId : String<UUID>,
  email : String,
  accessToken : String,
  scopes : Array<String>
}
```

 - `firstName`, `lastName`: the user’s name
 - `userId`: a globally unique string that identifies the current user.  It is a UUID, for example `8ffcdce7-2aa4-4a42-b89b-8ee7c3862ca5`.
 - `email`: this value will be the user’s email address.  Note that it will be `undefined` unless you request access to it using the `email` scope.
 - `accessToken`: the OAuth access token
 - `scopes`: list of scopes the user has approved

## Scopes

An OAuth scope is a string like `email` that defines what data your app can
access from CancerBase.  Adding `email` to your list of requested scopes asks
the user to provide your app with his or her email address.  If you do not ask
for this scope, or the user does not grant you access to this scope, the
user's email will be `undefined`.

The user must approve each scope that you request, so in general, you should
only ask the user for scopes that your app absolutely needs.

The following scopes are supported:

 - `email`: read the user’s email address (if you do not ask for this scope,
   the user’s email will be `undefined`).  The user may reject the `email`
   scope if you ask for it, so your app must be able to operate even if the
   user does not provide his or her email.
 - `cb.profile`: read and write the basic CancerBase profile data of the user.
   This includes initial diagnosis location and date, metastasis, approximate
   age, and sex.
 - `cb.location`: read and write the approximate location for the user.
 - `cb.timeline`: read and write the user’s timeline events

If you only need read access (not write access) to a user’s timeline or
profile, you can request the more specific scopes:

- `cb.profile.read`: read-only access to the user’s profile
- `cb.timeline.read`: read-only access to the user’s timeline

Broader scopes overrule narrower scopes, so if you request both
`cb.profile.read` and `cb.profile`, you will have both read and write access
to the user’s profile (the more specific `read` request is unnecessary).

A reasonable set of default scopes might look something like:

```javascript
const DEFAULT_SCOPES = [
  'cb.profile.read',
  'cb.timeline.read'
];
```

## Login Status

An API call may fail with an error if the user is logged out.  You may want to
check the user's login status before executing a series of API calls to ensure
that they will likely not fail for this reason.

Internally, the grants a user has given your application take the form of an
access token that is automatically included in every API request.  Checking
the login status confirms that this access token is still valid and that the
user did not disconnect his or her CancerBase account from your app (which
revokes all access tokens from that user).

You can check the user’s login status two ways:

  1. Synchronous

  ```javascript
  CancerBaseSDK.isLoggedIn() → true/false
  ```

    The synchronous method is not guaranteed to be accurate, but it is very
    likely accurate unless the user signs out of your app from cancerbase.org
    (not currently possible).

  2. Asynchronous

  ```typescript
  CancerBaseSDK.getLoginStatus() → Promise<status : string>
  ```

    The asynchronous method verifies that the current OAuth access token is still valid.

Generally, you can use the synchronous version.

## Logout

Your app must respond to the logout event and return the user to the login screen.

```typescript
CancerBaseSDK.on(‘logout’, () => app.logoutAndReturnToSplashScreen());
```

When logging out, you should clear all user data from the device.  You should
only persist your app’s per-user data on your servers or using the UserData
API (described later).

# Data APIs

All APIs for data reading and writing return JavaScript
[Promise](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Promise)
objects.

Once the user is logged in, you can make the following requests:

### Profiles

Profiles in CancerBase look like:

*TODO*
```typescript
type profileShape = {
  born : Date,
  initialDate : Date,
  metastasis : Boolean
  diseaseState : String,
  sex : String,
  nickname : String
};
```

Read a user's profile.

```typescript
CancerBaseSDK.user.getProfile()
  .then(profile => {

  }, error => {
  // something went wrong
  });
```

Write a user's profile.

```typescript
CancerBaseSDK.user.updateProfile(newProfileValues);
```

### UserData

CancerBase provides per-user data storage for apps that do not want a full
server to function.  The UserData API provides a per-user key-value store.
Values must be JSON-stringifiable.

```typescript
CancerBaseSDK.user.set(key, value)
  .then(() => console.log('saved!'));

CancerBaseSDK.user.get(key)
  .then(value => console.log(`the value is ${value}`));
```

You can optionally pass in a second argument to speed up the API:

```typescript
CancerBaseSDK.user.get(key, {cached: true})
```

If a cached value exists on the device, this will resolve the promise with the
cached value rather than fetching the current value from the server.  Note
that if another device has updated this value, it will be incorrect.

The UserData set/get APIs are not safe for concurrent use.  If a user is
running your app on multiple devices and reading/writing data at the same
time, you will have a race condition and may lose data.  This may be a rare
scenario for your app, but you should be aware of it and ensure that your app
will continue to function in case of errors.

In most cases, this is not an issue.  If a user writes the same key from two
devices at the same time, it is arbitrary which one will be stored.  For
advanced uses, like implementing a counter, you can use the testAndSet API:

```javascript
CancerBaseSDK.user.testAndSet(key, prevValue, newValue);
```

For example:
```javascript
async function onLogin() {
  const key = 'user-login-counter';
  const value = await CancerBaseSDK.user.get(key);
  try {
    // increment the counter by 1
    await CancerBaseSDK.user.testAndSet(key, value, value + 1);
    console.log('it worked!');
  } catch (error) {
    if (error.code === 'old') {
      console.log(`value out of date, current value is ${error.currentValue}`);
      setTimeout(onLogin, 100); // try again in 100ms
    }
  }
}
```

Please note that there are no guarantees about the reliability or performance
of the user data APIs.  It is intended for light use only.  If you plan on
making heavy use of server-side data, please use your own server.

## AppData

CancerBase provides a global key value store for app data.  This data is read-
only from the app and identical in all apps.  As a developer, you can use this
API to push dynamic content to your app.  It can only be written to using the
app owner's service account credentials.  This prevents malicious users from
modifying the global app data from within your app.

```typescript
CancerBaseSDK.appData.get(key)
  .then(value => console.log(`the value is ${value}`));
```

### Writing App Data

To update the global app data, you can use your service account id and secret
and the command line tool, `set-app-data` from the npm module `cancerbase-
tools`.  Your service account id and secret should never be included in your
app or shared with anyone outside of your organization.  If you are afraid
that your service account details have been compromised, or if for example you
accidentally commit them into your git repo, you can contact us to revoke and
replace them.  CancerBase may rotate your service account id and secret at any
time.

# Timeline APIs

Get a timeline event by ID.

```typescript
CancerBaseSDK.timeline.get(eventId)
  .then(event => console.log(event));
```

Query a user's timeline for events.

```typescript
CancerBaseSDK.timeline.get({
  from : Date,
  to : Date,
  category : String
}).then(res => {
  res.page == 1;
  res.lastPage == false;
  res.items.map(item => {
     const { createdAt, createdBy, date ... } = item;
     // createdBy is a unique identifier for an application
  });
});
```

To get the entire timeline, call with no arguments, e.g. `.get()`

```typescript
CancerBaseSDK.timeline.create({
  date : date,
  group : string,
  category : string,
  tags : string,
  title : string (256)
  description : string,
  data : object
}).then(({ eventId }) => console.log(`event ${eventID} created!`));
```

Update an existing timeline event.

```typescript
CancerBaseSDK.timeline.update({
  eventId : string,
  < same as createEvent >
});
```

Delete a timeline event.
```typescript
CancerBaseSDK.timeline.delete({
  eventId : string
});
```

Deleted timeline events can be restored (you can offer the user an undo
button) as long as you still remember the `eventId`.

```typescript
CancerBaseSDK.timeline.undelete({
  eventId : string
});
```
