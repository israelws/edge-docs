---
title: Edge API/SDK Reference

language_tabs:
  - javascript: Javascript

toc_footers:
  - <a href='https://developer.airbitz.co'>Sign Up for a Developer Key</a>

includes:
  - plugins
  - errors

search: true
---
# Introduction

AirbitzCore (ABC) is a Javascript client-side blockchain and Edge Security SDK providing auto-encrypted and auto-backed up accounts and wallets with zero-knowledge security and privacy. All blockchain/bitcoin private and public keys are fully encrypted by the users' credentials before being backed up on to peer to peer servers.

ABC allows developers to apply client-side data security, encrypted such that only the end-user can access the data. The Edge AbcAccount object allows developers to store arbitrary Edge-Secured data on the user’s account which is automatically encrypted, automatically backed up, and automatically synchronized between the user’s authenticated devices.

ABC is the foundation of the Edge Security SDK and Edge Wallet mobile app

To get started, you’ll first need an API key. Get one at our [developer portal.](https://developer.airbitz.co)

## Documentation/API conventions

```javascript
// Example callback style
abcContext.accountHasPassword("JoeHomey", function (error, hasPassword) {
    if (error) {
      // Error
    } else {
      if (hasPassword) {
        // This account has a password
      }
    }
})

// Example promise style
abcContext.accountHasPassword("JoeHomey").then(hasPassword => {
  if (hasPassword) {
    // This account has a password
  }
}).catch(error => {
  // Error
})

// Example using async/await syntax
try {
  const hasPassword = await abcContext.accountHasPassword("JoeHomey")
  if (hasPassword) {
    // This account has a password
  }
} catch(error) {
  // Error  
}
```

Documention and examples include parameter types in Flow style syntax. Due to Flow types not being standard Javascript, copy pasting of example code will fail under most environments except for React Native.

All asynchronous function methods are capable of either a callback or Promise style interface. If a callback is passed as the final parameter to an asynchronous routine, the callback parameters will use a NodeJS callback style with the first parameter being the `error` object followed by the return parameter or object. If the `callback` parameter is omitted, the routine will return a Promise that resolves to the return value.

Further documentation will use the async/await syntax for brevity

# Edge Security SDK

The Edge login system provides a way to backup and retrieve encrypted private keys. Users can login in using various methods:

* Password
* PIN
* Recovery questions
* Fingerprint
* Barcode scan

A single account (username and password) can log into multiple applications, each with its own keys. The application id (or `appId`) determines which keys are visible through this API. The keys provide access to various resources, including:

* Edge datastore repos for account settings
* Edge datastore repos for wallet metadata
* Public blockchains, like Bitcoin and Ethereum
* BitId identities

Edge provides a separate Wallet API for working with these keys once they have been retrieved by the login system.

## Install the SDK

The Edge SDK system runs in many different environments, including the web, Node.js and React Native. To handle the differences between these platforms, the login system talks to the outside world through an `io` object. Before intializing the login API, you need an `io` object for your particular platform.

### makeFakeIos

```javascript
const abc = require('airbitz-core-js')

const [io1, io2] = abc.makeFakeIos(2)
const context1 = abc.makeContext({ io: io1 })
const context2 = abc.makeContext({ io: io2 })
```

Creates an array of virtual IO objects with a simulated server and storage. This is useful for unit testing. All objects share the same virtual server, which makes it possible to simulate multi-phone login scenarios.

| Param | Type | Description |
| --- | --- | --- |
| count | `number` | Number of `io` objects to create

### makeBrowserIo

```javascript
const abc = require('airbitz-core-js')

const io = abc.makeBrowserIo()
const context = abc.makeContext({ io })
```

Gathers the various browser API's into an IO object. This allows Edge to work on the web.

### airbitz-io-node-js

```javascript
const abc = require('airbitz-core-js')
const abcNode = require('airbitz-io-node-js')

const io = abcNode.makeNodeIo('/home/username/.config/appname')
const context = abc.makeContext({ io })
```

Gathers various Node.js API's into an IO object.

| Param | Type | Description |
| --- | --- | --- |
| path | `string` | The filesystem path to save files to

### airbitz-core-react-native

```javascript
const abc = require('airbitz-core-js')
const abcReact = require('airbitz-core-react-native')

abcReact.makeReactNativeIo().then(io => {
  const context = abc.makeContext({ io })
})
```

Gathers various React Native API's into an IO object. This is an asynchronous function, and returns a `Promise`.

## AbcContext

Starting point of Airbitz Core SDK. Used for operations that do not require a logged in AbcAccount

### makeContext

```javascript
const abc = require('airbitz-core-js')

const abcContext = abc.makeContext({
  apiKey: '178a732cacebed37...',
  appId: 'com.yourname.yourapp',
  io: myPlatformSpecificIo
})
```

Initialize and create an AbcContext object. Required for functionality of ABC SDK.

| Option | Type | Description |
| --- | --- | --- |
| apiKey | `string` | Get an API Key from <https://developer.airbitz.co> |
| appId | `string` | Type of account that this application will be accessing. This should be in reverse-domain format, like `"com.domain.app"`. The `appId` determines which keys are available to your application, so a login performed by `"com.domain.app1"` will receive different keys from `"com.domain.app2"`. |
| io | `object` | (Optional) Platform-specifc io resources. Defaults to browser IO if not provided. |

| Return Param | Type | Description |
| --- | --- | --- |
| context | [`AbcContext`](#abccontext) | Initialized context |

### createAccount

```javascript
abcContext.createAccount(username,
                         password,
                         pin,
                         AbcAccountOptions,
                         callback)

// Example
const callbacks = {
  onDataChanged () {},
  onKeyListChanged () {},
  onLoggedOut () {},
  onOTPRequired () {},
  onOTPSkew () {},
  onRemotePasswordChange () {}
}

abcContext.createAccount(
  "myUsername",
  "myNot5oGoodPassw0rd",
  "2946",
  { callbacks },
  function (error, account) {
    if (error) {
      // Error creating account
    } else {
      abcAccount = account;
    }
  }
})
```

Create and log into a new AbcAccount

| Param | Type | Description |
| --- | --- | --- |
| username | `string` | Account username |
| password | `string` | Account password |
| pin | `string` | Account PIN for fast re-login |
| options | [`AbcAccountOptions`](#abcaccountoptions) | (Javascript) Callback event routines |
| delegate | [`AbcAccountDelegate`](#abcaccountdelegate) | (ObjC) Callback event delegates |
| callback | `Callback` | Callback function when routine completes|

| Return Param | Type | Description |
| --- | --- | --- |
| error | [`AbcError`](#abcerror) | Error object. `null` if no error |
| account | [`AbcAccount`](#abcaccount) | Initialized account |

### getLocalAccount

```javascript
abcContext.getLocalAccount(username, callbacks, options)

// Example
abcContext.getLocalAccount("JoeHomey", null,
                           function (error, account) {
    if (error) {
      // Failed to get account.
    } else {
      // Yay. Got local account info
      console.log("Account name = " + account.username)
    }
})
```

(Proposal)

Get local account details for a previously logged in account. This returns an AbcAccount object with a `null` `dataStore` object but with a functioning `localDataStore` object. This is effectively getting the non-encrypted account data which can be accessed without the user logging into the device with a password, PIN, or fingerprint.

Any [ABCWallet](#abcwallet) objects in the account will also have `null` `dataStore` objects but with functioning `localDataStore` objects. This is commonly used for background processing the accounts/wallets on a device to do querying of cryptocurrency transactions while the user is not logged in.

| Param | Type | Description |
| --- | --- | --- |
| username | `string` | Account username |
| options | [`AbcAccountOptions`](#abcaccountoptions) | (Javascript) Callback event routines |
| delegate | [`AbcAccountDelegate`](#abcaccountdelegate) | (ObjC) Callback event delegates |
| callback | `Callback` | Callback function when routine completes |

| Return Param | Type | Description |
| --- | --- | --- |
| error | [`AbcError`](#abcerror) | Error object. `null` if no error |
| account | [`AbcAccount`](#abcaccount) | Initialized account |

### loginWithPassword

```javascript
abcContext.loginWithPassword(username, password, options, callback)

// Example
abcContext.loginWithPassword(
  "JoeHomey",
  "My0KPa55WoRd@Airb1t5",
  { callbacks },
  function (error, account) {
    if (error) {
      if (error.name === 'OtpError') {
        console.log("otpResetToken: " + error.otpResetToken)
      } else {
        // login failed.
      }
    } else {
      // Yay. logged in
      console.log("Account name = " + account.username)
    }
})
```

```objc
-(void) loginWithPassword:(NSString *) username
                 password:(NSString *) password
                 delegate:(AbcAccountDelegate) delegate
                 callback:^(AbcError *error, AbcAccount *account);

// Example
AbcAccount *abcAccount;

[abcContext loginWithPassword:@"myUsername"
                     password:@"myNot5oGoodPassw0rd"
                     delegate:self
                     callback:^(AbcError *error, AbcAccount *account)
{
    if (error)
    {
      // Yikes
      if (error.code == ABCConditionCodeInvalidOTP) {
        NSLog(@"otpResetToken: %@", error.otpResetToken)
      }
    }
    else
    {
        NSLog(@"Account name = %@", account.username);
    }
}];
```

Login to an Edge account with a full password. May optionally send 'otp' key which is required for any accounts that have OTP enabled using [AbcAccount.enableOtp](#enableotp). OTP key can be retrieved from a device that has account logged in and OTP enabled using getOtpLocalKey.

If routine returns with `error.name` == 'OtpError', then the account has OTP enabled and needs the OTP key specified in parameter 'otp'. AbcError object may have properties otpResetToken and otpResetDate set which allow the user to call requestOtpReset to disable OTP.

This routine allows caller to receive back an error.otpResetToken which is used with requestOtpReset to remove OTP from the specified account.

The otpResetToken is only returned if the caller has provided the correct username and password but the account had OTP enabled. error.otpResetDate is the date when the account OTP will be disabled if a prior OTP reset was successfully requested. The reset date is set by default to 7 days from when a reset was initially requested.

| Param | Type | Description |
| --- | --- | --- |
| username | `string` | Account username |
| password | `string` | Account password |
| options | [`AbcAccountOptions`](#abcaccountoptions) | (Javascript) Callback event routines & OTP key |
| delegate | [`AbcAccountDelegate`](#abcaccountdelegate) | (ObjC) Callback event delegates |
| callback | `Callback` | Callback function when routine completes |

| Return Param | Type | Description |
| --- | --- | --- |
| error | [`AbcError`](#abcerror) | Error object. `null` if no error |
| account | [`AbcAccount`](#abcaccount) | Initialized account |

### loginWithPIN

```javascript
abcContext.loginWithPIN(username, pin, callbacks, opts)

// Example
abcContext.loginWithPIN(
  "JoeHomey",
  "2847",
  { callbacks },
  function (error, account) {
    if (error) {
      // Error
    } else {
      // Yay. logged in
    }
})
```

```objc
-(void) loginWithPIN:(NSString *) username
            password:(NSString *) password
            delegate:(AbcAccountDelegate) delegate
            callback:^(AbcError *error, AbcAccount *account);

// Example
AbcAccount *abcAccount;

[abcContext loginWithPIN:@"myUsername"
                     pin:@"4728"
                delegate:self
                callback:^(AbcError *error, AbcAccount *account)
{
    if (!error)
    {
        NSLog(@"Account name = %@", account.username);
    }
    else
    {
        // Yikes
    }
}];
```

Login to an Edge account with PIN. Used to sign into devices that have previously been logged into using a full username & password

| Param | Type | Description |
| --- | --- | --- |
| username | `string` | Account username |
| pin | `string` | Account PIN for fast re-login |
| options | [`AbcAccountOptions`](#abcaccountoptions) | (Javascript) Callback event routines |
| delegate | [`AbcAccountDelegate`](#abcaccountdelegate) | (ObjC) Callback event delegates |
| callback | `Callback` | Callback function when routine completes|

| Return Param | Type | Description |
| --- | --- | --- |
| error | [`AbcError`](#abcerror) | Error object. `null` if no error |
| account | [`AbcAccount`](#abcaccount) | Initialized account |


### accountHasPassword

```javascript
abcContext.accountHasPassword(username, callback)

// Example
abcContext.accountHasPassword("JoeHomey", function (error, hasPassword) {
    if (error) {
      // Error
    } else {
      if (hasPassword) {
        // This account has a password
      }
    }
})
```

```objc
-(BOOL) accountHasPassword:(NSString *) username
                     error:(AbcError *) error;

// Example
AbcError *error;

BOOL hasPassword = [abcContext accountHasPassword:@"myUsername"
                                            error:&error
```

Check if specified username has a password on the account or if it is a PIN-only account.

| Param | Type | Description |
| --- | --- | --- |
| username | `string` | Account username |
| callback | `Callback` | (Javascript) Callback function when routine completes |

| Return Param | Type | Description |
| --- | --- | --- |
| error | [`AbcError`](#abcerror) | Error object. `null` if no error |
| hasPassword | `Boolean` | True if account has a password |


### deleteLocalAccount

```javascript
abcContext.deleteLocalAccount(username, callback)

// Example
abcContext.deleteLocalAccount("JoeHomey", function (error) {
    if (error) {
      // Error
    } else {
      // Success deleting account from local device
    }
})
```

```objc
- (AbcError *) deleteLocalAccount:(NSString *) username;

// Example
AbcError *error;

AbcError *error = [abcContext deleteLocalAccount:@"myUsername"];
```

Deletes named account from local device. Account is recoverable if it contains a password. Use accountHasPassword to determine if account has a password. Recommend warning user before executing deleteLocalAccount if accountHasPassword returns FALSE.

| Param | Type | Description |
| --- | --- | --- |
| username | `string` | Account username |
| callback | `Callback` | (Javascript) Callback function when routine completes |

| Return Param | Type | Description |
| --- | --- | --- |
| error | [`AbcError`](#abcerror) | Error object. `null` if no error |
| hasPassword | `Boolean` | True if account has a password |

### pinLoginEnabled

```javascript
abcContext.pinLoginEnabled(username, callback)

// Example
abcContext.pinLoginEnabled(username,function (error, enabled) {
    if (!error) {
      console.log("PIN Login enabled state: " + enabled)
    }
})
```

```objc
- (BOOL)pinLoginEnabled:(NSString *)username error:(NSError **)error;

// Example
AbcError *error;
BOOL enabled = [abcContext pinLoginEnabled:username error:&error];
```

Checks if PIN login is possible for the given username. This checks if there is a local PIN package on the device from a prior login

| Param | Type | Description |
| --- | --- | --- |
| username | `String` | Account username |
| callback | `Callback` | (Javascript) Callback function when routine completes|

| Return Param | Type | Description |
| --- | --- | --- |
| error | [`AbcError`](#abcerror) | Error object. `null` if no error |
| enabled | `Boolean` | True if PIN login is enabled |

### getRecovery2Key

```javascript
recovery2Key = getRecovery2Key(username)
```

Retrieves the recovery key for a particular user. The user should email this key to themselves.

If this returns an error, the user does not have recovery set up on this device. The user needs to click the link in their email if they want to do a recovery login.

### fetchRecovery2Questions

```javascript
const questions = await fetchRecovery2Questions(recovery2Key, username)

// Result:
[
  'What is your name?',
  'What is your quest?',
  'What is your favorite color?'
]
```

Retrieves the recovery questions a user has configured for their account.

### loginWithRecovery2

```javascript
const answers = [
  'Sir Lancelot of Camelot',
  'To seek the Holy Grail',
  'Blue'
]

const account = await loginWithRecovery2(
  recovery2Key,
  username,
  answers,
  { callbacks }
)
```

Performs a recovery login.

### listRecoveryQuestionChoices

```javascript
const questions = await listRecoveryQuestionChoices()
```

Fetches a list of possible recovery questions from the server.

TODO: How does this work with localization? Why not build the question list into the app itself?

### listUsernames

```javascript
abcContext.listUsernames(callback)

// Example
abcContext.listUsernames(function (error, usernames) {
    if (!error) {
      console.log("username 0: " + usernames[0])
    }
})
```

```objc
- (NSArray *) listUsernames:(AbcError **) abcerror;

// Example
NSError *error;
NSArray *usernames = [abc listUsernames:&error];
if (!error) {
  NSLog(@"username 0: %@", usernames[0]);
}
```

Get a list of previously logged in usernames on this device

| Param | Type | Description |
| --- | --- | --- |
| callback | `Callback` | (Javascript) Callback function when routine completes|

| Return Param | Type | Description |
| --- | --- | --- |
| error | [`AbcError`](#abcerror) | Error object. `null` if no error |
| usernames | `Array` | Array |

### usernameAvailable

```javascript
abcContext.usernameAvailable(username, callback)

// Example
abcContext.usernameAvailable(username, function (error, available) {
    if (!error && available) {
      console.log("username available = " + available)
    }
})
```

```objc
- (BOOL)pinLoginEnabled:(NSString *)username error:(NSError **)error;

// Example
AbcError *error;
BOOL enabled = [abcContext pinLoginEnabled:username error:&error];
```

Checks if PIN login is possible for the given username. This checks if there is a local PIN package on the device from a prior login

| Param | Type | Description |
| --- | --- | --- |
| username | `String` | Account username |
| callback | `Callback` | (Javascript) Callback function when routine completes|

| Return Param | Type | Description |
| --- | --- | --- |
| error | [`AbcError`](#abcerror) | Error object. `null` if no error |
| enabled | `Boolean` | True if PIN login is enabled |

### requestOtpReset

```javascript
abcContext.requestOtpReset(username: string, otpResetToken: string): Promise<void>

// Example
try {
  await abcContext.requestOtpReset("JoeHomey", "ZR2G9d8TYW8DH6")
} catch (error) {
  console.log(error)
}
```
Launches an OTP reset timer on the server, which will disable the OTP authentication requirement on the account `username` when timeout expires. The expiration timeout is set when OTP is first enabled using [AbcAccount.enableOtp](#enableotp).

To obtain an otpResetToken, attempt a login into the OTP protected account using loginWithPassword with the correct username/password. The login should fail with error.name == 'OtpError'. The OTP token will be in the error.otpResetToken property of the AbcError object.

| Param | Type | Description |
| --- | --- | --- |
| username | `String` | Account username |
| otpResetToken | `String` | Reset token from loginWithPassword |

### getCurrencyPlugins

```javascript
const plugins = await abcContext.getCurrencyPlugins()

// Example value:
[
  { currencyInfo: { ... }, ... }, // AbcCurrencyPlugin
  { currencyInfo: { ... }, ... } // AbcCurrencyPlugin
]
```

Retrieves an array of [`AbcCurrencyPlugin`](#abccurrencyplugin) objects extracted from the plugin list passed in at context creation time. These can be used for parsing URI's, retrieving currency information, and other currency-related tasks.

### fetchMessages

```javascript
const messages = await context.fetchLoginMessages();

for (const username of Object.keys(messages)) {
  if (messages[username].otpResetPending) {
    console.log(`User ${username} has an OTP reset pending`)
  }
}
```

Fetches account status alerts for all users that have ever logged into this device.

| Return Type | Description |
| --- | --- |
| `Promise<AbcLoginMessages>` | A map from usernames to status flags. |

| Status Flag | Type | Description |
| --- | --- | --- |
| otpResetPending | boolean | True if someone is trying to turn off OTP (2-factor) protection on this account. |
| recovery2Corrupt | boolean | There was an error setting up recovery questions, so the user must re-configure these before recovery logins will work again. |

## AbcAccount

### Basic Login Properties

| Property | Type | Description |
| --- | --- | --- |
| loggedIn | `string` | True if [`AbcAccount.logout`](#logout) has not been called. |
| loginKey | `string` | The master decryption key for the account. This key gives full access to all account assets, including wallet private keys. It can be stored in the device secure element to enable fingerprint login via [`AbcContext.loginWithKey`](#loginwithkey). |
| recoveryKey | `string` | The email backup key for recovery login. |
| username | `string` | Account username |

### logout

```javascript
abcAccount.logout(function() {
  // Hooray. I'm out
})
```

```objc
[abcAccount logout];
```

Logout the currently logged in AbcAccount

| Param | Type | Description |
| --- | --- | --- |
| account | [`AbcAccount`](#abcaccount) | Account object |
| callback | `Callback` | (Javascript) Callback function |

### currencyWallets

```javascript
abcAccount.currencyWallets

// Example value:
{
  'BReGzRCkPABS3x3CFJv9Kv38Jeyt6smoOyWU4WNaPq4=': { ... }, // AbcCurrencyWallet
  'eT5WkFLlp6Mt8Ljyu2Mv3LzCTw2Zc0QebbhPljRyTYY=': { ... } // AbcCurrencyWallet
}
```

A key-value map. The keys are wallet id's, and the values are instances of [`AbcCurrencyWallet`](#abccurrencywallet). The account will automatically load and manage any wallet types that have corresponding plugins.

### activeWalletIds

```javascript
abcAccount.activeWalletIds

// Example value:
[
  'BReGzRCkPABS3x3CFJv9Kv38Jeyt6smoOyWU4WNaPq4=',
  'eT5WkFLlp6Mt8Ljyu2Mv3LzCTw2Zc0QebbhPljRyTYY='
]
```

An array of active wallet ID's, in their proper sort order.  This list only includes wallets are not deleted or archived and have a corresponding currency plugin.

### archivedWalletIds

```javascript
abcAccount.archivedWalletIds

// Example value:
[
  'BReGzRCkPABS3x3CFJv9Kv38Jeyt6smoOyWU4WNaPq4=',
  'eT5WkFLlp6Mt8Ljyu2Mv3LzCTw2Zc0QebbhPljRyTYY='
]
```

An array of archived wallet ID's, in their proper sort order. This list only includes wallets are not deleted and have a corresponding currency plugin.

### allKeys

```javascript
keys = abcAccount.allKeys
```

This read-only property contains a `Array` of all keys visible to the account, including keys that may belong to other applications (i.e. child logins).

### changePassword

```javascript
await abcAccount.changePassword(password)
```

```objc
- (void)changePassword:(NSString *)password
              callback:(void (^)(AbcError *error)) callback;

// Example
[abcAccount changePassword:password callback:^(AbcError *error) {
    if (error) {
        // Oh no
    } else {
        // Yay, new password set
    }
}];
```

Change the password of the currently logged in AbcAccount

| Param | Type | Description |
| --- | --- | --- |
| password | `string` | Password string |

| Return Type | Description |
| --- | --- |
| `Promise<void>` | Resolves or rejects once the function finishes. |

### changePin

```javascript
// Set up a new PIN, and use it for login:
await abcAccount.changePin({ pin: '1234', enableLogin: true })

// Keep the exising PIN, but enable / disable using it for login:
await abcAccount.changePin({ enableLogin: true })
await abcAccount.changePin({ enableLogin: false })
```

```objc
- (void)changePIN:(NSString *)pin
         callback:(void (^)(AbcError *error)) callback;

// Example
[abcAccount changePIN:pin callback:^(AbcError *error) {
    if (error) {
        // Oh no
    } else {
        // Yay, new PIN set
    }
}];
```

Change the PIN of the currently logged in AbcAccount, and configure whether the PIN may be used for logging in.

| Param | Type | Description |
| --- | --- | --- |
| pin | `string | void` | 4 digit PIN string, or `undefined` to leave the PIN unchanged and simply toggle its login capability. |
| enableLogin | `boolean` | True to enable logging in via PIN, or false to disable this feature. The PIN may still be used for in-app spending limits and other features via [AbcAccount.checkPin](#checkpin), even if it's disabled for login. |

| Return Type | Description |
| --- | --- |
| `Promise<void>` | Resolves or rejects once the function finishes. |

### changeRecovery

```javascript
const recoveryKey = await abcAccount.changeRecovery(
  [
    'What is your name?',
    'What is your quest?',
    'What is your favorite color?'
  ],
  [
    'Sir Lancelot of Camelot',
    'To seek the Holy Grail',
    'Blue'
  ]
)

// Now the user should email the recoveryKey to themselves...
```

Change the recovery questions and anwers for the the logged in AbcAccount.

| Param | Type | Description |
| --- | --- | --- |
| questions | `Array<string>` | An array of questions to show the user. |
| answers | `Array<string>` | An array of answers to the provided questions. The user must answer the questions exactly (character-for-character) to log in. |

| Return Type | Description |
| --- | --- |
| `Promise<string>` | Resolves or rejects once the function finishes. The resolved value is a recoveryKey, which the user should email to themselves. Logging in via recovery requires having access to this key. |

### checkPassword

```javascript
const isOk = await abcAccount.checkPassword('MyNotSoGoodPassword')
```

```objc
- (BOOL)checkPassword:(NSString *)password;

// Example
BOOL passwordCorrect = [abcAccount checkPassword:password];
```

Checks if password is the correct password for this account

| Param | Type | Description |
| --- | --- | --- |
| password | `string` | Account password |

| Return Type | Description |
| --- | --- | --- |
| `Promise<boolean>` | Resolves to `true` if the password is correct, or `false` if the password is wrong. |

### checkPin

```javascript
const isOk = await abcAccount.checkPin('1234')
```

Checks if the provided PIN is correct for this account.

| Param | Type | Description |
| --- | --- | --- |
| pin | `string` | Account PIN |

| Return Type | Description |
| --- | --- | --- |
| `Promise<boolean>` | Resolves to `true` if the PIN is correct, or `false` if the PIN is wrong. |

### deletePin

```javascript
await abcAccount.deletePin()
```

Removes the PIN from this account. The [`AbcAccount.checkPin`](#checkpin) function will no longer work, so this may be too extreme. Consider using [`AbcAccount.changePin`](#changepin) if the goal is to simply disable PIN login.

| Return Type | Description |
| --- | --- |
| `Promise<void>` | Resolves or rejects once the function finishes. |

### deleteRecovery

```javascript
await abcAccount.deleteRecovery()
```

Removes the recovery questions from this account.

| Return Type | Description |
| --- | --- |
| `Promise<void>` | Resolves or rejects once the function finishes. |

### OTP Properties

| Property | Type | Description |
| --- | --- | --- |
| otpKey | `string | void` | The OTP secret. This can be shown to the user for backup purposes. This will be `undefined` if OTP is not configured. |
| otpResetDate | `Date` | Set to a `Date` object if someone has requested an OTP reset for this account. The OTP will be turned off after the provided data. |

### cancelOtpReset

```javascript
await abcAccount.cancelOtpResetRequest()
```

Removes the OTP reset request from the server for the currently logged in user. When a user logs in on a new device for an account with OTP enabled, the login will fail with 'OtpError'. A reset request can then be made using [AbcContext.requestOtpReset](#requestOtpReset). This function allows a logged in user to cancel that request and prevent the remote device from logging in.

| Return Type | Description |
| --- | --- |
| `Promise<void>` | Resolves or rejects once the function finishes. |

### disableOtp

```javascript
await abcAccount.disableOtp()
```

Removes the OTP authentication requirement from the server for the currently logged in user.

| Return Type | Description |
| --- | --- |
| `Promise<void>` | Resolves or rejects once the function finishes. |

### enableOtp

```javascript
// Enable OTP protection with a 1-week reset requirement:
await abcAccount.enableOtp(7 * 24 * 60 * 60)
```

Enables 2-factor protection for this account.

| Param | Type | Description |
| --- | --- | --- |
| timeout | number | The number of seconds the user must wait for an OTP reset request to complete. Defaults to 1 week. |

| Return Type | Description |
| --- | --- |
| `Promise<void>` | Resolves or rejects once the function finishes. |

### fetchLobby

```javascript
const abcLobby = await abcAccount.fetchLobby(lobbyId)

if (abcLobby.loginRequest) {
  // Show the request to the user...

  await abcLobby.loginRequest.approve()
}
```

Retrieves an edge-login or wallet-sharing request from the Airbitz lobby server. The `lobbyId` can be found in the edge-login QR code.

| Param | Type | Description |
| --- | --- | --- |
| lobbyId | `string` | The lobby id, extracted from an edge-login QR code. |

| Return Param | Type | Description |
| --- | --- | --- |
| abcLobby | [`Promise<AbcLobby>`](#abclobby) | The lobby object. |

### signBitIDRequest (proposal)

```javascript
abcAccount.signBitIDRequest(uri, message, callback)

// Example
abcAccount.signBitIDRequest("bitid://airbitz.co/developer?nonce=12345",
                            "Hello World", function (error, abcSignature) {
  if (!error) {
    console.log("Public address of signature = " + abcSignature.address)
    console.log("Signature = " + abcSignature.signature)
  }
})
```

Sign an arbitrary message with a BitID URI. The URI determines the key derivation used to sign the message.

| Param | Type | Description |
| --- | --- | --- |
| url | `String` | URI used to derive the private key to do the signature |
| message | `String` | Message to sign |
| callback | `Callback` | (Javascript) Callback function |

| Return Param | Type | Description |
| --- | --- | --- |
| error | [`AbcError`](#abcerror) | (Javascript) Error object. `null` if no error |
| abcSignature | [`ABCBitIDSignature`](#ABCBitIDSignature) | BitID Signature Object |

### createWallet

```javascript
abcAccount.createWallet(walletType, abcWalletKeys)

// Example
const abcWalletKeys = {
  ethereumKey: new Buffer(secureRandom(32)).toString('hex')
}


abcAccount.createWallet("wallet:repo:ethereum",
                        abcWalletKeys,
                        function (error, id) { /* your callback */ })
```

Create a new [AbcWalletInfo](#abcwalletinfo) object and add it to the current account. Each wallet object represents key storage for a specific cryptocurrency type or other misc functionality such as BitID or general data storage. Once a wallet is created, the wallet keys cannot be modified. AbcWalletInfo objects may be shared between AbcAccount objects of the same or different users given permission by the user.

| Param | Type | Description |
| --- | --- | --- |
| walletType | `String` | Wallet type corresponding to the one of the following below |
| callback | `Callback` | (Javascript) Callback function |

| Callback Params | Type | Description |
| --- | --- | --- |
| error | [`AbcError`](#abcerror) | (Javascript) Error object. `null` if no error |
| id | `String` | Strings of wallet ID |

Please see the [AbcWalletInfo](#abcwalletinfo) documentation for the different wallet types Edge understands.

### listWalletIds

```javascript
const walletIds = abcAccount.listWalletIds()
```

Get an array list of wallet IDs in the current account

| Return Param | Type | Description |
| --- | --- | --- |
| walletIds | `Array` | Array of strings of wallet IDs |

### getWallet

```javascript
abcAccount.getWallet(walletId)

// Example
const abcWallet = abcAccount.getWallet(walletId)
```

Get an [AbcWalletInfo](#abcwalletinfo) object given a `walletId`

| Param | Type | Description |
| --- | --- | --- |
| walletId | `String` | Wallet ID from listWallets. `null` if no matching wallet |

| Return Param | Type | Description |
| --- | --- | --- |
| abcWallet | [`AbcWalletInfo`](#abcwalletinfo) | [AbcWalletInfo](#abcwalletinfo) object |

### getFirstWallet

```javascript
abcAccount.getFirstWallet(walletType)

// Example
const abcWallet = abcAccount.getFirstWallet('wallet:repo:ethereum')
```

Get the first [AbcWalletInfo](#abcwalletinfo) object of type `walletType`

| Param | Type | Description |
| --- | --- | --- |
| error | [`AbcError`](#abcerror) | (Javascript) Error object. `null` if no error |
| walletType | `String` | Wallet type |

| Return Param | Type | Description |
| --- | --- | --- |
| abcWallet | [`AbcWalletInfo`](#abcwalletinfo) | (Javascript) Error object. `null` if no error |

### changeWalletStates

```javascript
const walletStates = {
  "wallet-id-1": {
    archived: false,
    deleted: false,
    sortIndex: 0
  },
  "wallet-id-2": {
    archived: true,
    sortIndex: 2
  }
}

await account.changeWalletStates(keyStates)
```

Keys (which represent wallets) can have associated metadata. This method makes it possible to change the metadata for one or more wallets.

| Param | Type | Description |
| --- | --- | --- |
| walletStates | `Object` | An object mapping from wallet ID's to the new metadata. All properties are optional, and unspecified properties will remain unchanged. |
| callback | `Callback` | (Javascript) Callback function |

The metadata structure looks like this:

| Property | Type | Description |
| --- | --- | --- |
| archived | `boolean` | Marks the wallet as "archived". Archived wallets are still visible, but do not regularly poll the network. |
| deleted | `boolean` | Marks the wallet as "deleted". Deleted wallets are completely invisible to the user. |
| sortIndex | `number` | Used to place the wallets in a particular order in the user interface. Lower indices should sort first. |

### requestWallet (proposal)

```javascript
// Make the request:
const opts = {
  type: 'wallet:bitcoin',
  onWalletShared (walletId) {
    // We just received a wallet!
  }
}
const pendingRequest = await abcAccount.requestWallet(opts)

// Put this in a QR code and show it to the user:
const qrCodeUri = `edge://lobby/${pendingRequest.lobbyId}`

// If the user loses interest, cancel the request:
pendingRequest.cancel()
```

Creates a QR code that can be used to request a wallet from another user. If the remote user scans the barcode and approves the request, the wallet will be added to this account.

| Param | Type | Description |
| --- | --- | --- |
| type | `string` | A wallet type. See [`AbcWalletInfo`](#abcwalletinfo) for more information. |
| onWalletShared | `Function` | Called when a remote user fulfills the request. |

## AbcAccountOptions

```javascript
const options = {
  otp: 'PMLCAKJ2IUDQQCIB',

  callbacks: {
    onDataChanged () {
      // Reload any files we care about...
    },
    onLoggedOut () {
      // Return to the login screen...
    }
    // etc...
  }
}

const account = await context.loginWithPassword('user', 'pass', options)
```

Callback routines that notify application when various changes have occurred in the account. This is only utilized for Javascript. For ObjC, see [AbcAccountDelegate](#abcaccountdelegate).

| Property | Type | Description |
| --- | --- | --- |
| otp | `string` | The OTP generation seed. This is only needed when logging into a 2fa-protected account for the first time on a new device. |
| callbacks | `object` | A collection of callback functions for listening to changes in the account state. |

### Callbacks

| Property | Type | Description |
| --- | --- | --- |
| onDataChanged() | `Function` | The account's synced data has changed in any way. If the GUI is storing settings or other information in the account repo, it should refresh those. |
| onKeyListChanged() | `Function` | The user has changed the key list in some way, including adding, deleting, archiving, or sorting. The GUI should refresh its wallets list. |
| onLoggedOut() | `Function` | Account has been logged out. Not used. |
| onOTPRequired() | `Function` | Another device has enabled OTP, and this device does not have the correct OTP token. The GUI should notify the user and give them an opportunity to scan the OTP barcode if they desire. |
| onOTPSkew() | `Function` | This device's clock is out-of-sync with the Edge servers. The user should be notified to fix their clock. Otherwise, if the skew gets worse, they may not be able to log in again. |
| onRemotePasswordChange() | `Function` | The account password has been changed by a remote device. The GUI should notify the user and give them an opportunity to log in again, which will activate the change on this device as well. Otherwise, this device will continue to have the old password. |

## AbcLobby

```javascript
interface AbcLobby {
  loginRequest?: AbcLoginRequest,
  walletRequest?: AbcWalletRequest, // Not supported at this time
}
```

To perform an edge login, an app creates a "lobby" on the Edge server. This lobby contains the public keys for the reply encryption, as well as information about what the request is for. A logged-in wallet can download this lobby, validate the request, and post a reply back to the server.

| Property | Type | Description |
| --- | --- | --- |
| loginRequest | [`AbcLoginRequest`](#abcloginrequest) | A login request, if the user is trying to log in. |
| walletRequest | `AbcWalletRequest` | A wallet request, if the user wants a wallet. |

## AbcLoginRequest

```javascript
interface AbcLoginRequest {
  appId: string,
  approve(): Promise<void>,

  displayName: string,
  displayImageUrl: string
}
```

If a user is requesting an edge login, their lobby will contain this object. Calling the `approve` method will send the required keys to the requesting application.

In the future, `AbcLoginRequest` may contain information about the wallet types the requesting application wants. When this happens, the `approve` method will begin accepting an `opts` parameter to specify which wallets the user wishes to share.

## ABCBitIDSignature

### Class Properties

| Property | Type | Description |
| --- | --- | --- |
| address | `String` | Public address of private key used to sign |
| signature | `String` | Public address of private key used to sign |

## AbcWalletInfo

```javascript
{
  "id": "1EeW6nzRFeSrtjgiGWlQLZUb/HDHOZspfLgALd9j6UE=",
  "type": "wallet:bitcoin",
  "keys": {
    "bitcoinKey": "K83+b3/fuR4wzIymH3SltCAafJwgW54E/gQh91nmRSo=",
    "dataKey": "QIjkKYVg1tTT8eYgfyGWJp4T7R1yHcpHDUqHtSuPmt0=",
    "syncKey": "1JG0vR1L8tFbsWiS/kuZGgY6/hY="
  }
}
```

An `AbcWalletInfo` contains the keys needed to access a wallet or other resource. The Edge login system exists to store these wallet keys in an encrypted and backed-up manner.

The Edge SDK includes full send & receive capability for a variety of blockchains. If you use these features, you won't need to deal with these wallet keys directly.

Otherwise, if your application does its own blockchain access, keeping your wallet keys in this format will ensure that the Edge Wallet application can seamlessly interoperate with the keys your app creates.

| Property | Type | Description |
| --- | --- | --- |
| id | `String` | The globally-unique wallet ID. A 256-bit base64-encoded integer. |
| type | `String` | The type of wallet these keys unlock. |
| keys | `Object` | The contents of this object depend on the wallet type. See the documentation for the wallet types below. |

Every wallet has a globally-unique `id`. The simplest approach is to pick a random number for this `id`, using the same entropy source that would be used for private keys.

Some supported wallet types are documented in the sections below:

* [Storage Wallets](#storage-wallets)
* [Account Repos](#account-repos)
* [wallet:bitcoin](#wallet-bitcoin)
* [wallet:bitcoin-bip44](#wallet-bitcoin-bip44)
* [wallet:ethereum](#wallet-ethereum)

### Storage Wallets

Many different wallets include access to a Git repo for storage. In these cases, the `keys` property will have the following members:

| Property | Type | Description |
| --- | --- | --- |
| dataKey | `String` | The data encryption key. A 256-bit base64-encoded integer. |
| syncKey | `String` | The git repo identity. A 160-bit base64-encoded integer. |

For pure storage wallets, the wallet type should be a reverse domain-name starting with `storage:`, like `storage:com.yourdomain.yourtype`.

In the future, Edge may introduce a `readKey`, which provides only read-only access to the sync server. This would be something like `sha256(syncKey)`. In that case, the `keys` object would contain one, the other, or both of `syncKey` and `readKey`.

### Account Repos

```javascript
{
  "type": "account-repo:co.airbitz.wallet",
  "id": "91MD0VDrSH6i+Sq6oM+uzIT6CEJlmz3cG3VYDRu+s1Y=",
  "keys": {
    "dataKey": "u0dnSMP+/Qz/CfDpeqloT1HOSlrqM59lIOeMG2Yz44g=",
    "syncKey": "otzd6UewIXAQv+FHOPYVPWzJjZ8="
  }
}
```

| Property | Type | Description |
| --- | --- | --- |
| dataKey | `String` | See [Storage Wallets](#storage-wallets). |
| syncKey | `String` | See [Storage Wallets](#storage-wallets). |

Account repos have `type` set to `'account-repo:' + appId`. They have storage keys and nothing else; everything interesting is stored in Git. Each app has the freedom to decide what goes in their own account repo.

### wallet:bitcoin

```javascript
{
  "id": "1EeW6nzRFeSrtjgiGWlQLZUb/HDHOZspfLgALd9j6UE=",
  "type": "wallet:bitcoin",
  "keys": {
    "bitcoinKey": "K83+b3/fuR4wzIymH3SltCAafJwgW54E/gQh91nmRSo=",
    "bitcoinXpub": "xpub6Bk1pSnncGei3JScQNzoZT4Ry6z4Zv4FJPKMNhGiHNmuw6o77oN4sPbmMYwPe2ED7KXxcFx8ZBVnEiDEnBoUHor3jzpYuhJUhLh9wR6Shsx",
    "dataKey": "QIjkKYVg1tTT8eYgfyGWJp4T7R1yHcpHDUqHtSuPmt0=",
    "syncKey": "1JG0vR1L8tFbsWiS/kuZGgY6/hY="
  }
}
```

A BIP-32 Bitcoin wallet. The key derivation follows the same scheme specified in the original BIP 32 spec.

| Property | Type | Description |
| --- | --- | --- |
| bitcoinKey | `String` | The master entropy according to the BIP 32 spec. A 256-bit base64-encoded integer. |
| bitcoinXpub | `String` | Key `m/0` in `xpub` format, as defined by the BIP 32 spec |
| dataKey | `String` | See [Storage Wallets](#storage-wallets). |
| syncKey | `String` | See [Storage Wallets](#storage-wallets). |

A spending-capable bitcoin wallet will have a `bitcoinKey`, while a read-only bitcoin wallet will just have `bitcoinXpub`. Individual private keys are derived using the BIP32 path m/0/0/n where `n` is the index of the private key to derive.

The `airbitz-core-js` library is responsible for managing `dataKey` and `syncKey`; the currency plugins should ignore them.

### wallet:bitcoin-bip44

```javascript
{
  "id": "1EeW6nzRFeSrtjgiGWlQLZUb/HDHOZspfLgALd9j6UE=",
  "type": "wallet:bitcoin-bip44",
  "keys": {
    "bitcoinKey": "dawn subway charge into unhappy limb kind palace dwarf pear shoulder battery",
    "bitcoinXpub": "xpub6Bk1pSnncGei3JScQNzoZT4Ry6z4Zv4FJPKMNhGiHNmuw6o77oN4sPbmMYwPe2ED7KXxcFx8ZBVnEiDEnBoUHor3jzpYuhJUhLh9wR6Shsx",
    "dataKey": "QIjkKYVg1tTT8eYgfyGWJp4T7R1yHcpHDUqHtSuPmt0=",
    "syncKey": "1JG0vR1L8tFbsWiS/kuZGgY6/hY="
  }
}
```

A BIP-44 Bitcoin wallet. The key derivation follows the same scheme specified in the original BIP 44 spec.

| Property | Type | Description |
| --- | --- | --- |
| bitcoinKey | `String` | 12-24 word phrase master private seed as specified in BIP 44 spec |
| bitcoinXpub | `String` | Key `m/0` in `xpub` format, as defined by the BIP 44 spec. |
| dataKey | `String` | See [Storage Wallets](#storage-wallets). |
| syncKey | `String` | See [Storage Wallets](#storage-wallets). |

A spending-capable bitcoin wallet will have a `bitcoinKey`, while a read-only bitcoin wallet will just have `bitcoinXpub`.

The `airbitz-core-js` library is responsible for managing `dataKey` and `syncKey`; the currency plugins should ignore them.

### wallet:ethereum

```javascript
{
  "type": "wallet:ethereum",
  "id": "lZvB9W1waiDHn52JRzUjfqAbyp1wyN5jreKbdyto4pI=",
  "keys": {
    "ethereumKey": "65256374d98202d11b22d74a5d89960cf50d71f45a3d4f7641e1c2ce3b2bdc89",
    "dataKey": "X/ovC9WM7EClpjMc/U+0HHQv/cmUlgHFM+drtuiF73Q=",
    "syncKey": "VvijidoeI07d3sYvgyGghwGJBpU="
  }
}
```

An Ethereum wallet.

| Property | Type | Description |
| --- | --- | --- |
| ethereumKey | `String` | A hex-encoded 256-bit Ethereum private key without leading `0x` |
| ethereumAddress | `String` | A hex-encoded Ethereum payment address WITH a leading `0x` |
| dataKey | `String` | See [Storage Wallets](#storage-wallets). |
| syncKey | `String` | See [Storage Wallets](#storage-wallets). |

A spending-capable ethereum wallet will have an `ethereumKey`, while a read-only ethereum wallet will just have `ethereumAddress`.

The `airbitz-core-js` library is responsible for managing `dataKey` and `syncKey`; the currency plugins should ignore them.

## ABCDataStore

This API is no longer used. Please see [AbcStorageWallet](#abcstoragewallet).

### writeData

```javascript
abcWallet.dataStore.writeData(folder, key, value, callback)

// Example
abcWallet.dataStore.writeData("userAddress",
                              "state",
                              "California",
                              function(error) {
  if (error === null) {
    // Success
  }
})
```

| Param | Type | Description |
| --- | --- | --- |
| folder | `String` | Name of folder for data store |
| key | `String` | Name of data key |
| value | `String` | data value |
| callback | `Function` | (Javascript) Callback function |

| Callback Param | Type | Description |
| --- | --- | --- |
| error | [`AbcError`](#abcerror) | (Javascript) Error object. `null` if no error |

Writes a string of data to the specified dataStore folder using the given key. `folder` must not include the characters "/". The dataStore object at this point only provides a single depth folder for storing key/value pairs. Keys can be enumerated using the [ABCDataStore.listKeys](#listkeys) method. Data can read back using the [ABCDataStore.readData](#readdata) method.

### readData

```javascript
abcWallet.dataStore.readData(folder, key, callback)

// Example
abcWallet.dataStore.readData("userAddress",
                             "state",
                             function(error, value) {
  if (error === null) {
    console.log(value)
  }
})
```

| Param | Type | Description |
| --- | --- | --- |
| folder | `String` | Name of folder for data store |
| key | `String` | Name of data key |
| callback | `Callback` | (Javascript) Callback function |

| Callback Param | Type | Description |
| --- | --- | --- |
| error | [`AbcError`](#abcerror) | (Javascript) Error object. `null` if no error |
| value | `String` | data value |

Reads back a string of data from the specified dataStore folder using the given key. `folder` must not include the characters "/". The dataStore object at this point only provides a single depth folder for storing key/value pairs. Keys can be enumerated using the [ABCDataStore.listKeys](#listkeys) method.

### removeKey

```javascript
abcWallet.dataStore.removeKey(folder, key, callback)

// Example
abcWallet.dataStore.removeKey("userAddress",
                              "state",
                              function(error) {
  if (error === null) {
    // Success. Key/value pair removed
  }
})
```

| Param | Type | Description |
| --- | --- | --- |
| folder | `String` | Name of folder for data store |
| key | `String` | Name of data key |
| callback | `Callback` | (Javascript) Callback function |

| Callback Param | Type | Description |
| --- | --- | --- |
| error | [`AbcError`](#abcerror) | (Javascript) Error object. `null` if no error |

Removes the specified key/value pair from the dataStore.

### removeFolder

```javascript
abcWallet.dataStore.removeFolder(folder, callback)

// Example
abcWallet.dataStore.removeFolder("userAddress",
                                 function(error) {
  if (error === null) {
    // Success. Key/value pair removed
  }
})
```

| Param | Type | Description |
| --- | --- | --- |
| folder | `String` | Name of folder for data store |
| callback | `Callback` | (Javascript) Callback function |

| Callback Param | Type | Description |
| --- | --- | --- |
| error | [`AbcError`](#abcerror) | (Javascript) Error object. `null` if no error |

Removes the specified folder from the dataStore. All key/value pairs in the folder will also be removed.

### listKeys

```javascript
abcWallet.dataStore.listKeys(folder, callback)

// Example
abcWallet.dataStore.listKeys("userAddress",
                             function(error, keys) {
  if (error === null) {
  }
})
```

| Param | Type | Description |
| --- | --- | --- |
| folder | `String` | Name of folder for data store |
| callback | `Callback` | (Javascript) Callback function |

| Callback Param | Type | Description |
| --- | --- | --- |
| error | [`AbcError`](#abcerror) | (Javascript) Error object. `null` if no error |
| keys | `Array` | Array of strings of keys in folder |

List the keys in the specified folder

## Errors

```javascript
import { error } from 'airbitz-core-js'

try {
  // Something went wrong!
  throw new error.NetworkError('Cannot reach the Ethereum network')

  // Later...
} catch (e) {
  if (e.name === error.NetworkError.name) {
    console.log('Check your network cable!')
  }
}
```

All Edge API's use one of the standard Javascript error-reporting mechanisms:

* Throwing an exception
* Rejecting a promise
* Passing an error to a callback (for Node.js style API's)

Regardless of the reporting mechanism, the returned object will always be an instance of the built-in `Error` object. To determine the exact type of error, use the error object's `name` property.

The `airbitz-core-js` defines a number of error names with special meaning, documented below. The library also exports constructor functions in the `error` namespace, which can be used to create instances of all these error types and to access their name constants in a consistent way.

### DustSpendError

Trying to send an uneconomically small amount of money.

| Property | Type | Description |
| --- | --- | --- |
| name | `String` | "DustSpendError" |

### InsufficientFundsError

Trying to send more money than the wallet has available.

| Property | Type | Description |
| --- | --- | --- |
| name | `String` | "InsufficientFundsError" |

### NetworkError

A server was unreachable.

| Property | Type | Description |
| --- | --- | --- |
| name | `String` | "NetworkError" |

### ObsoleteApiError

The app is attempting to reach a server endpoint that no longer exists, which implies that the app needs to be upgraded.

| Property | Type | Description |
| --- | --- | --- |
| name | `String` | "ObsoleteApiError" |

### OtpError

The user provided the correct login credentials, but did not provide a 2-factor token.

| Property | Type | Description |
| --- | --- | --- |
| name | `String` | "OtpError" |
| resetToken | `String` | Use this token to request a 2-factor reset, if the user requests one. |
| resetDate | `Date` | If a 2-factor reset is pending, gives the reset time. |

### PasswordError

The user has entered an invalid password, PIN, or recovery answers.

| Property | Type | Description |
| --- | --- | --- |
| name | `String` | "PasswordError" |
| wait | `Number` | Number of seconds the user must wait before trying again |

### UsernameError

The user has entered an invalid username, or the username already exists for account creation.

| Property | Type | Description |
| --- | --- | --- |
| name | `String` | "UsernameError" |

# Wallet API

Once the user has logged in and retrieved their keys, there are serveral things that may take place with those keys, including:

* Accessing encrypted storage
* Sending and receiving digital currencies
* Signing BitId requests

The Edge SDK includes several helper classes which perform these capabilities on your behalf, using a set of provided keys.

## AbcStorageWallet

```javascript
// Save a JSON file:
const text = JSON.stringify(settingsJson)
abcStorageWallet.folder.file('settings.json').setText(text)

// Load a JSON file:
const text = await abcStorageWallet.folder.file('settings.json').getText()
const settingsJson = JSON.parse(text)

// Save a binary file:
const image = new Uint8Array(bytes)
abcStorageWallet.folder.file('avatar.png').setData(image)

// Save a text file in a folder:
abcStorageWallet.folder.folder('articles').file('blog.md').setText(article)

// Iterate over the files in a folder:
import { mapFiles } from 'disklet'

mapFiles(abcStorageWallet.folder.folder('articles'), async file => {
  console.log(await file.getText())
})
```

This wallet type handles encrypted and synced data. This is the base class for several other wallet types, including `AbcCurrencyWallet`. You can also use it directly if your application requires multiple encrypted and backed-up data stores. Bear in mind that each account also has its own data store, so you may not need this class unless you are doing something involving key sharing.

See the [Disklet](https://www.npmjs.com/package/disklet) documentation for information on the filesystem API.

### makeStorageWallet

```javascript
const abc = require('airbitz-core-js')

// Select some keys, using any approach you find useful:
const storageWalletKeys = abcAccount.allKeys.find(keyInfo => {
  return keyInfo.type === 'account:myApp'
})

// Access synced data:
abc
  .makeStorageWallet(storageWalletKeys, { account: abcAccount })
  .then(function (abcStorageWallet) {})
```

This function creates an instance of the storage wallet class. 

| Param | Type | Description |
| --- | --- | --- |
| keys | `String` | An [`AbcWalletInfo`](#abcwalletinfo) structure from the account. |
| opts.account | `Object` | The [`AbcAccount`](#abcaccount) object that these keys belong to. |
| opts.onDataChange | `Function` | Called when the data changes as part of a sync operation. |

### Class Properties

| Property | Type | Description |
| --- | --- | --- |
| id | `String` | The `id` property of the account-level [AbcWalletInfo](#abcwalletinfo) |
| type | `String` | The `type` property of the account-level [AbcWalletInfo](#abcwalletinfo) |
| keys | `Object` | The `keys` property of the account-level [AbcWalletInfo](#abcwalletinfo) |
| name | `String` | Human readable name of wallet. May be `null` if the wallet has no name. |
| folder | `Folder` | A [`Disklet`](https://www.npmjs.com/package/disklet) folder. This datastore object is encrypted by default, backed up to the cloud, and synchronized with any device the user logs into. Data modifications are versioned and can be rolled back but this functionality is not yet exposed via API. dataStore object may `null` if parent [AbcAccount](#abcaccount) has not been logged into yet |
| localFolder | `Folder` | A [`Disklet`](https://www.npmjs.com/package/disklet) folder that only exists on the current device. This data is not encrypted nor backed up. Not to be used for sensitive data but rather as a local cache of network data. Data is not version controlled and has no rollback capability. Common use case will be for local device specific wallet settings, blockchain cache information, and public address cache for use when account/wallet has not yet been decrypted (background processing) |

### renameWallet

```javascript
abcStorageWallet.renameWallet(name, callback)

// Example
abcStorageWallet.renameWallet("My Wallet", function (error) {
  if (!error) {
    // Success
  }
})
```

| Param | Type | Description |
| --- | --- | --- |
| name | `String` | New wallet name |
| callback | `Callback` | (Javascript) Callback function |

| Callback Param | Type | Description |
| --- | --- | --- |
| error | [`AbcError`](#abcerror) | (Javascript) Error object. `null` if no error |

Returns a list of transactions in the current wallet. Transactions are returned ordered from newest to oldest.

## AbcCurrencyWallet

Each AbcCurrencyWallet represents a single or HD cryptocurrency wallet tied to one specific blockchain such as bitcoin or ethereum. Various methods and fields in AbcCurrencyWallet are arrays or accept an index which selects which token system in the wallet is being referenced.

This class also includes all the methods and callbacks of [`AbcStorageWallet`](#abcstoragewallet).

### makeCurrencyWallet

```javascript
const abc = require('airbitz-core-js')
const abcBitcoin = require('airbitz-currency-bitcoin')

// This should probably happen once when the app first boots:
const bitcoinPlugin = abcBitcoin.makeBitcoinPlugin({
  io: yourPlatformSpecificIo
})

function onAddressesChecked (progressRatio) {}
function onBalanceChanged (balance) {}
function onTransactionsChanged (transactionArray) {}
function onNewTransactions (transactionArray) {}
function onBlockHeightChanged (height) {}

// Select some keys, using any approach you find useful:
const bitcoinWalletKeys = abcAccount.allKeys.find(
  keyInfo => keyInfo.type === 'wallet:bitcoin'
)

// Access synced data and transactions:
abc
  .makeCurrencyWallet(bitcoinWalletKeys, {
    account: abcAccount,
    plugin: bitcoinPlugin,
    callbacks: {
      onAddressesChecked,
      onBalanceChanged,
      onTransactionsChanged,
      onNewTransactions,
      onBlockHeightChanged
    }
  })
  .then(abcBitcoinWallet => {})
```

Creates a wallet capable of send and receive functionality. An [AbcCurrencyPlugin](#abccurrencyplugin) object must be passed in that exposes the entire [AbcCurrencyPlugin](#abccurrencyplugin) interface.

| Param | Type | Description |
| --- | --- | --- |
| keys | [`AbcWalletInfo`](#abcwalletinfo) | The wallet info obtained from the account |
| opts.account | `AbcAccount` | The account object this wallet belongs to. |
| opts.plugin | `AbcCurrencyPlugin` |  Object that exposes the [AbcCurrencyPlugin](#abccurrencyplugin) functions |
| opts.callbacks | `AbcCurrencyPlugin` | `Object` | Object with callback functions |

(Javascript) Returns a `Promise` for an AbcCurrencyWallet type.

| Callback name | Type | Description |
| --- | --- | --- |
| onAddressesChecked(progressRatio) | `Function` | Called as the engine checks addresses. If money has arrived while the app was shut down, this process will eventually detect it. The balance and history may be incomplete until `progressRatio` reaches 100%. |
| onBalanceChanged(balance) | `Function` | The spendable balance has changed, either from newly-detected transactions or from dropped transactions. |
| onBlockHeightChanged(height) | `Function` | Blockchain height changed. This is unused for sub wallets. The confirmation status of each transaction is `chainHeight - transactionHeight`, so this affects all confirmed transactions. |
| onNewTransactions(transactionArray) | `Function` | Array of AbcTransaction objects. These transactions are either previously-recognized funds that are being synced to this device for the first time, or updates to previously-seen transactions such as new metadata or confirmation status. |
| onTransactionsChanged(transactionArray) | `Function` | Array of never-before-seen AbcTransaction objects. These are new funds that the GUI should notify the user about. |
| onWalletNameChanged(name) | `Function` | The user has changed the wallet name, either on this device or on another device. |

This class also includes all callbacks of [`AbcStorageWallet`](#abcstoragewallet).

### Class Properties

| Property | Type | Description |
| --- | --- | --- |
| fiatCurrencyCode | `String` | 3 character fiat currency code |
| cryptoCurrencyCodes | `Array` | Array of Strings of crypto currency code. ie "BTC" |
| primaryCryptoCurrencyCode | `String` | String of primary crypto currency code. ie "BTC". Other codes in `cryptoCurrencyCodes` will reference meta-tokens of this currency's blockchain. |

### setupContract

(under construction)

```javascript
// Example
var abcSetupContractOptions = [
  { addMasterPublicKey: "xpub12fe094ab5d830ee9ac0102fbd223e" },
  { addMasterPublicKey: "xpubef92309571209798ab098ce0912fae" },
  { addMasterPublicKey: "xpuba39f948e0d0139bc99848cb10a0935" },
  { numRequiredKeys: 2 }
]

abcCurrencyWallet.setupContract(abcSetupContractOptions)
```

Setup the script/contract for this wallet. This is used to setup basic multisig wallets under currencies like bitcoin and ethereum.

### getDisplayPrivateSeed

```javascript
const privateSeed = abcCurrencyWallet.getDisplayPrivateSeed()
if (privateSeed !== null) {
  console.log(privateSeed)
}
```

Gets the master private seed (or private key if wallet is initialized from key and not from seed) from the engine as a string

| Return | Type | Description |
| --- | --- | --- |
| privateSeed | `String | null` | The private seed if exists |

### getDisplayPublicSeed

```javascript
const publicKey = abcCurrencyWallet.getDisplayPublicSeed()
if (publicKey !== null) {
  console.log(publicKey)
}
```

Gets the master public key (or address in case of the ethereum plugin) from the engine as a string

| Return | Type | Description |
| --- | --- | --- |
| publicKey | `String | null` | The public key if exists |

### resyncBlockchain

```javascript
const publicKey = await abcCurrencyWallet.resyncBlockchain()
```

Wipes out the local disk and memory cache of all transactions and forces a refetching of all data from the network.

| Return | Type | Description |
| --- | --- | --- |
| promise | `Promise<void>` | A promise that resolves when the cache has been cleared. |

### dumpData

```javascript
abcCurrencyWallet.dumpData()

// Example
const dataDump = abcCurrencyWallet.dumpData()
console.log(dataDump.walletId)
console.log(dataDump.walletType)
console.log(dataDump.pluginType)
console.log(dataDump.data)
```

Returns the entire memory and disk states of the wallet for debugging purposes

| Return | Type | Description |
| --- | --- | --- |
| dataDump | [`AbcDataDump`](#abcdatadump) | An object containing the dump of the entire wallet cache and memory |

### enableTokens

```javascript
// Example
const tokens = {
  tokens: [ "XCP", "TATIANACOIN" ]
}

abcCurrencyWallet.enableTokens(tokens).catch(handleError)
```

| Param | Type | Description |
| --- | --- | --- |
| tokens | `Array` | Array of strings specifying the currency codes of tokens to enable in this wallet |

| Return | Type | Description |
| --- | --- | --- |
| promise | `Promise<void>` | A promise that resolves when the tokens are ready to use. |

Enable support for meta tokens (ie. counterparty, colored coin, ethereum ERC20). Library should begin checking the blockchain for the specified tokens and triggering the callbacks specified in [`currencyPlugin.makeEngine`](#makeengine).

### disableTokens

```javascript
// Example
const tokens = {
  tokens: [ "XCP", "TATIANACOIN" ]
}

abcCurrencyWallet.disableTokens(tokens).catch(handleError)
```

| Param | Type | Description |
| --- | --- | --- |
| tokens | `Array` | Array of strings specifying the currency codes of tokens to disable in this wallet |

| Return | Type | Description |
| --- | --- | --- |
| promise | `Promise<void>` | A promise that resolves when the tokens are disabled. |

Disable support for meta tokens (ie. counterparty, colored coin, ethereum ERC20). Library will stop checking the blockchain for the specified tokens.

### getEnabledTokens

```javascript
getEnabledTokens():Promise<Array<string>>

// Example
try {
  const tokens = await abcCurrencyWallet.getEnabledTokens()
  console.log(tokens) // => ['WINGS', 'REP']
} catch (error) {
  console.log(error)
}
```

| Promise Return Param | Type | Description |
| --- | --- | --- |
| tokens | `Array<string>` | Array of strings of token currency codes. |

Query the library for the currently enabled tokens.

### getBalance

```javascript
abcCurrencyWallet.getBalance(opts)

// Example
const balance = abcCurrencyWallet.getBalance({ curencyCode: "BTC" })
```
| Param | Type | Description |
| --- | --- | --- |
| options | `Object` | May be `null` which will return the balance of the wallet's primary currency. |

| Option | Type | Description |
| --- | --- | --- |
| currencyCode | `String` | Selects which currency to return a balance for. |

| Return Param | Type | Description |
| --- | --- | --- |
| balance | `Int` | Wallet balance in smallest unit of currency. ie Satoshis |

Gets the current balance of the wallet denominated in the smallest unit of the currency. ie. Unit of satoshis for the currency bitcoin.

### getTransactions

```javascript
abcCurrencyWallet.getTransactions(options, callback)

// Example
// Create query that looks in the first 100 transactions filtered to the past 2 days that have meta data matching the string "Mom". Then returns the first 10 of those transactions
var end = new Date();
var start = new Date();
start.setDate(end.getDate() - 1);

const options = {
  currencyCode: "BTC",
  startIndex: 0,
  numEntries: 100,
  startDate: start,
  endDate: end,
  searchString: 'Mom',
  returnIndex: 0,
  returnEntries: 10
}

const abcTransactions = abcCurrencyWallet.getTransactions(options, function(error, abcTransactions) {
  const abcTransaction = abcTransactions[0]
})
```

| Param | Type | Description |
| --- | --- | --- |
| options | `Object` | May be `null` which will return all transactions |

| Option Params | Type | Description |
| --- | --- | --- |
| startIndex | `Int` | Index into the full list of transactions. If unspecified, no transactions are filtered and search passes on to `startDate`. Index 0 refers to the most recent transaction. |
| startEntries | `Int` | Number of entries to return from start of index. `startIndex` must be specified |
| startDate | `Date` | Date object, in local time, when to start returning transactions. If unspecified, transactions are not filtered by date and search passes on to `searchString` |
| endDate | `Date` | Date object, in local time, when to stop returning transactions. Must be later than `startDate` and `startDate` must be specified |
| searchString | `String` | Include only transactions that have metadata that matches `searchString`. (Optional) |
| returnIndex | `Int` | Index into the filtered list of transactions. If unspecified, no transactions are filtered and current results are returned.  Index 0 refers to the most recent transaction. |
| returnEntries | `Int` | Number of entries to return from index of filtered transactions. `returnEntries` must be specified |

| Return Param | Type | Description |
| --- | --- | --- |
| error | [`AbcError`](#abcerror) | (Javascript) Error object. `null` if no error |
| transactions | `Array` | Array of [AbcTransaction](#abctransaction) objects |

Returns a list of transactions in the current wallet. Options allow pruning of the search to a subset of the transactions in addition to string filtering. Options are applied in the following order:

Note that all indices start with the most recent transaction and work downwards to oldest.

1. `startIndex` and `startEntries` specify the starting index and number of entries from the full, un-filtered list of transactions.
2. `startDate` and `endDate` are applied to filter all transactions to only within specified dates.
3. `searchString` is applied to only include string matching transactions
4. `startIndex` and `numEntries` specify the starting index and number of entries from the Array created from steps 1 through 3. They do NOT refer to the Array of all transactions.

Transactions are returned ordered from newest to oldest.

### saveTxMetadata

```javascript
abcCurrencyWallet.saveTxMetadata(txid, currencyCode, metadata)

// Example
const txid = someTransaction.txid
const metadata = {
  name: "City Tacos",
  category: "Expense:Food & Dining"
}

abcCurrencyWallet.saveTxMetadata(txid, "BTC", metadata).then(() => {
  console.log("Transaction successfully updated")
})
```

| Param | Type | Description |
| --- | --- | --- |
| txid | `String` | The txid of an [`AbcTransaction`](#abctransaction) object |
| currencyCode | `String` | Which crypto-currency this metadata applies to, for wallets with multiple token support. |
| metadata | [`AbcMetadata`](#abcmetadata) | The metadata to save with the transaction. |
| callback | `Callback` | Called when the save is completed. |

| Callback Param | Type | Description |
| --- | --- | --- |
| error | [`AbcError`](#abcerror) | (Javascript) Error object. `null` if no error |

Updates the metadata saved with a transaction.

### getBlockHeight

```javascript
abcCurrencyWallet.getBlockHeight()

// Example
const height = abcCurrencyWallet.getBlockHeight()
```

| Param | Type | Description |
| --- | --- | --- |
| void | `Void` | none |

| Return Param | Type | Description |
| --- | --- | --- |
| height | `Int` | Block height of current wallet |

Gets the current blockchain height of the wallet's cryptocurrency. This is expected to be the same value for all meta-tokens in this wallet and hence does not require a currencyCode index.

### getReceiveAddress

```javascript
abcCurrencyWallet.getReceiveAddress(options, callback)

// Example to simply return an unused address
const abcReceiveAddress = abcCurrencyWallet.getReceiveAddress(null, function (error, abcReceiveAddress) {
  if (!error) {
    // Success
  }
})

// Example to return an unused address with the tag "QRCODE"
const abcReceiveAddress = abcCurrencyWallet.getReceiveAddress({'addressTag': 'QRCODE'}, function (error, abcReceiveAddress) {
  if (!error) {
    // Success
  }
})

// Example to return an unused address with the tag "QRCODE" and for the currency "REP"
const options = {
  'addressTag': 'QRCODE',
  'currencyCode': 'REP'
}
const abcReceiveAddress = abcCurrencyWallet.getReceiveAddress(options, function (error, abcReceiveAddress) {
  if (!error) {
    // Success
  }
})

// Example to get an ABCRequestAddress object for BTC currency and previous publicAddress '1FVBrmeuEeAxbNcj2EL4v2XsfBDbv7A9aE'
const options = {
  currencyCode: "BTC",
  publicAddress: '1FVBrmeuEeAxbNcj2EL4v2XsfBDbv7A9aE'
}

const abcReceiveAddress = abcCurrencyWallet.getReceiveAddress(options, , function (error, abcReceiveAddress) {
  if (!error) {
    // Success
  }
})
```

| Param | Type | Description |
| --- | --- | --- |
| options |  `Object` | Options object. See below
| callback | `Callback` | (Javascript) Callback function |

| Options Param | Type | Description |
| --- | --- | --- |
| currencyCode | `String` | (Optional) Chooses the currency or meta-token to get an address for. If not specified, uses the primary currency of this wallet |
| publicAddress | `String` | (Optional) Get an AbcReceiveAddress object using a previously returned public address. If `publicAddress` is set, `addressTag` must be null or unspecified |
| addressTag | `String` | (Optional) Arbitrary tag for this specific address request type. Future calls to getReceiveAddress with the same tag will return the same address as the previous call with the same tag unless that address has received funds. Useful for specifying a "display" address which is shown on screen but never used for email or SMS requests. Calling [lockReceiveAddress](#lockreceiveaddress) will cause this address to no longer be returned regardless of whether it has received funds. If `addressTag` is set, `publicAddress` must be null or unspecified |

| Callback Param | Type | Description |
| --- | --- | --- |
| error | [`AbcError`](#abcerror) | (Javascript) Error object. `null` if no error |
| abcReceiveAddress | `AbcReceiveAddress` | [AbcReceiveAddress](#abcreceiveaddress) |

Returns an [AbcReceiveAddress](#abcreceiveaddress) object. This routine is used to generate a new public address or return a previously request address object.  The `metadata` object

### saveReceiveAddress

```javascript
abcCurrencyWallet.saveReceiveAddress(abcReceiveAddress, callback)

// Example
abcCurrencyWallet.getReceiveAddress(null, function (error) {
  if (!error) {
    // Success
    console.log("My bitcoin address: " + abcReceiveAddress.publicAddress)
    abcReceiveAddress.nativeAmount = '150000000' // 1.5 BTC
    abcReceiveAddress.metadata.name = "Johnny Be Good"
    abcReceiveAddress.metadata.category = "Income:Rent"
    abcReceiveAddress.saveReceiveAddress(function(error) {
      // Meta data for this address has been saved
    })
  }
})
```

Updates the internal database of `metadata` corresponding to this `receiveAddress`. Any transactions that are received in this address are automatically tagged with the `metadata` from this `receiveAddress`.

### lockReceiveAddress

```javascript
abcCurrencyWallet.lockReceiveAddress(callback)

// Example
abcCurrencyWallet.lockReceiveAddress(function (error) {
  if (!error) {
    // Success
  }
})
```

Locks this `receiveAddress` so that any future calls to [AbcCurrencyWallet.getReceiveAddress](#getreceiveaddress), without the `publicAddress` specified, will no longer return this address. Funds can still be received on this address. Use [AbcCurrencyWallet.getReceiveAddress](#getreceiveaddress) with `publicAddress` specified to get back this object in the future.

### parseUri

```javascript
const abcParsedUri = wallet.parseUri("bitcoin:1CsaBND4GNA5eeGGvU5PhKUZWxyKYxrFqs?amount=1.2345&r=https%3A%2F%2Fbitpay.com%2Fi%2F7TEzdBg6rvsDVtWjNQ3C3X&label=Mom")

console.log(abcParsedUri.publicAddress) // -> 1CsaBND4GNA5eeGGvU5PhKUZWxyKYxrFqs
console.log(abcParsedUri.nativeAmount) // -> '123456700'
console.log(abcParsedUri.metadata.name) // -> 'Mom'
console.log(abcParsedUri.paymentProtocolURL) // -> https://bitpay.com/i/7TEzdBg6rvsDVtWjNQ3C3X
```

| Param | Type | Description |
| --- | --- | --- |
| uri | `String` | URI to parse |

| Return Param | Type | Description |
| --- | --- | --- |
| abcParsedUri | [`AbcParsedUri`](#abcparseduri) | Object with parsed parameters |

Parses a URI extracting various elements into an [AbcParsedUri](#abcparseduri) object

### encodeUri

```javascript
// Example
const abcParsedUri = {
  publicAddress: '1CsaBND4GNA5eeGGvU5PhKUZWxyKYxrFqs',
  nativeAmount: '123456700',
  metadata: {
    name: 'Mom'
  },
  paymentProtocolURL: 'https://bitpay.com/i/7TEzdBg6rvsDVtWjNQ3C3X'
}

const uri = wallet.encodeUri(abcParsedUri)

console.log(uri)
// ->  "bitcoin:1CsaBND4GNA5eeGGvU5PhKUZWxyKYxrFqs?amount=1.2345&label=Mom&r=https%3A%2F%2Fbitpay.com%2Fi%2F7TEzdBg6rvsDVtWjNQ3C3X")

```

| Param | Type | Description |
| --- | --- | --- |
| abcParsedUri | [`AbcParsedUri`](#abcparseduri) | Object with parsed parameters |

| Return Param | Type | Description |
| --- | --- | --- |
| uri | `String` | URI output |

Encodes a URI given a [AbcParsedUri](#abcparseduri) object


### makeAddressQrCode

```javascript
const qrCode = makeAddressQrCode(abcReceiveAddress)
```

| Return | Type | Description |
| --- | --- | --- |
| qrCode | `String` | Base64 encoded image of addressUri |

### makeAddressUri

```javascript
const uri = makeAddressUri(abcReceiveAddress)
```

| Return | Type | Description |
| --- | --- | --- |
| addressUri | `String` | BIP21 or equivalent URI that encodes public address and optionally requested amount, name of requestor, and category of requested funds |

### makeSpend

```javascript
// Example to spend to two bitcoin addresses
const abcSpendInfo = {
  networkFeeOption: 'high',
  currencyCode: 'BTC',
  metadata: {
    name: 'Tyra CPA',
    category: 'Expense:Professional Services'
  },
  spendTargets: [
    {
      publicAddress: '12xZEQL72YnGEbtW7bA4FPA1BUEHkxQoWN',
      nativeAmount: '210000000' // 2.1 BTC
    },
    {
      publicAddress: '1FSxyn9AbBMwGusKAFqvyS63763tM8KiA2',
      nativeAmount: '120000000' // 1.2 BTC 
    }
  ]
}

let abcTransaction: AbcTransaction = await abcCurrencyWallet.makeSpend(abcSpendInfo)
await abcCurrencyWallet.signBroadcastAndSave(abcTransaction)
console.log("Sent transaction with ID = " + abcTransaction.txid)

// Example to spend to a BIP70 payment request
const abcParsedUri = abcAccount.parseUri("bitcoin:1CsaBND4GNA5eeGGvU5PhKUZWxyKYxrFqs?amount=1.000000&r=https%3A%2F%2Fbitpay.com%2Fi%2F7TEzdBg6rvsDVtWjNQ3C3X")

let absSpendTarget: AbcSpendTarget = await abcCurrencyWallet.getPaymentProtocolInfo(abcParsedUri.paymentProtocolURL)
const abcSpendInfo: AbcSpendInfo = {
  networkFeeOption: 'high',
  metadata: {
    name: 'Tyra CPA',
    category: 'Expense:Professional Services'
  },
  spendTargets: [ spendTarget ]
}

const abcTransaction = abcCurrencyWallet.makeSpend(abcSpendInfo)
await abcCurrencyWallet.signBroadcastAndSave(abcTransaction)

console.log("Sent transaction with ID = " + abcTransaction.txid)

// Example wallet to wallet transfer
// (assuming these wallets are not `undefined`)
const walletIds = abcAccount.activeWalletIds
const srcWallet = abcAccount.currencyWallets[walletIds[0]]
const destWallet = abcAccount.currencyWallets[walletIds[1]]

const abcSpendInfo = {
  networkFeeOption: 'high',
  currencyCode: 'BTC',
  metadata: {
    name: 'Transfer to College Fund',
    category: 'Transfer:Wallet:College Fund'
  },
  spendTargets: [
    {
      destWallet,
      nativeAmount: '210000000' // 2.1 BTC
    }
  ]
}

const abcTransaction = await srcWallet.makeSpend(abcSpendInfo)
await srcWallet.signBroadcastAndSave(abcTransaction)
console.log("Sent transaction with ID = " + abcTransaction.txid)

// Example BTC -> ETH currency exchange between wallets specifying the destination amount
const walletIds = abcAccount.activeWalletIds
const srcWallet = abcAccount.currencyWallets[walletIds[0]] // ETH/REP wallet
const destWallet = abcAccount.currencyWallets[walletIds[1]] // BCH wallet

const abcSpendInfo = {
  networkFeeOption: 'high',
  currencyCode: 'BTC',
  spendTargets: [
    {
      destWallet,
      currencyCode: 'ETH',
      nativeAmount: '1200000000000000000' // 1.2 ETH
    }
  ]
}

try {
  let abcTransaction = await srcWallet.makeSpend(abcSpendInfo)
  abcTransaction = await srcWallet.signBroadcastAndSave(abcTransaction)
  console.log("Sent transaction with ID = " + abcTransaction.txid)
} catch (error) {
  console.log(error)
}

// Example REP -> BCH currency exchange between wallets specifying the source amount
const walletIds = abcAccount.listWalletIds()
const srcWallet = abcAccount.getWallet(walletId[0]) // ETH/REP wallet
const destWallet = abcAccount.getWallet(walletId[1]) // BCH wallet

const abcSpendInfo = {
  networkFeeOption: 'high',
  currencyCode: 'REP',
  nativeAmount: '1200000000000000000', // 1.2 REP
  spendTargets: [
    {
      destWallet,
      currencyCode: 'BCH'
    }
  ]
}

try {
  let abcTransaction = await srcWallet.makeSpend(abcSpendInfo)
  abcTransaction = await srcWallet.signBroadcastAndSave(abcTransaction)
  console.log("Sent transaction with ID = " + abcTransaction.txid)
} catch (error) {
  console.log(error)
}

```

| Param | Type | Description |
| --- | --- | --- |
| abcSpendInfo | [`AbcSpendInfo`](#abcspendinfo) | [AbcSpendInfo](#abcspendinfo) object with various parameters for a spend operation including output addresses, amounts, or payment protocol payment objects (BIP70) |
| callback | `Callback` | (Javascript) Callback function |

| Callback Param | Type | Description |
| --- | --- | --- |
| abcTransaction | [`AbcTransaction`](#abctransaction) | Unsigned [AbcTransaction](#abctransaction) object |

Creates an unsigned [AbcTransaction](#abctransaction) object which can be then be signed and broadcast to the network. Complete the spend by calling [AbcTransaction.signBroadcastAndSave](#signbroadcastandsave). Estimated fees can be determined by reading back [AbcTransaction.networkFee](#abctransaction)

May produce an [InsufficientFundsError](#insufficientfundserror) if the amount is too large, or a [DustSpendError](#dustspenderror) if the amount is too small.

### signTx

```javascript
abcCurrencyWallet.signTx(abcTransaction, callback)

// Example
abcCurrencyWallet.signTx(abcTransaction, function(error) {
  if (!error) {
    // Success, transaction signed
    console.log("Signed transaction with txId = " + abcTransaction.txid)
  }
})
```

| Param | Type | Description |
| --- | --- | --- |
| abcTransaction | [`AbcTransaction`](#abctransaction) | Unsigned [AbcTransaction](#abctransaction) object |
| callback | `Callback` | (Javascript) Callback function |

| Callback Param | Type | Description |
| --- | --- | --- |
| abcError | [`AbcError`](#abcerror) | [AbcError](#abcerror) object |

Signs this [AbcTransaction](#abctransaction) object. Does not broadcast this to the blockchain or save it in the local transaction cache. [AbcTransaction](#abctransaction).txid is null until this routine is called.

Call [AbcTransaction.broadcastTx](#broadcasttx) followed by [AbcTransaction.saveTx](#savetx) to broadcast and save the transaction.

### broadcastTx

```javascript
abcCurrencyWallet.broadcastTx(abcTransaction, callback)

// Example
abcCurrencyWallet.broadcastTx(abcTransaction, function(error) {
  if (!error) {
    // Success, transaction sent
    console.log("Sent transaction with ID = " + abcTransaction.txid)
  }
})
```

| Param | Type | Description |
| --- | --- | --- |
| abcTransaction | [`AbcTransaction`](#abctransaction) | Signed [AbcTransaction](#abctransaction) object |
| callback | `Callback` | (Javascript) Callback function |

| Callback Param | Type | Description |
| --- | --- | --- |
| abcError | [`AbcError`](#abcerror) | [AbcError](#abcerror) object |

Broadcasts transaction to the blockchain.

### saveTx

```javascript
abcCurrencyWallet.saveTx(abcTransaction, callback)

// Example
abcCurrencyWallet.saveTx(abcTransaction, function(error) {
  if (!error) {
    // Success, transaction saved
    console.log("Saved transaction with ID = " + abcTransaction.txid)
  }
})
```

| Param | Type | Description |
| --- | --- | --- |
| abcTransaction | [`AbcTransaction`](#abctransaction) | Signed [AbcTransaction](#abctransaction) object |
| callback | `Callback` | (Javascript) Callback function |

| Callback Param | Type | Description |
| --- | --- | --- |
| abcError | [`AbcError`](#abcerror) | [AbcError](#abcerror) object |

Saves transaction to local cache. This will cause the transaction to show in calls to [abcCurrencyWallet.getTransactions](#gettransactions).

### signBroadcastAndSave

```javascript
abcCurrencyWallet.signBroadcastAndSave(abcTransaction, callback)

// Example
abcCurrencyWallet.signBroadcastAndSave(abcTransaction, function(error) {
  if (!error) {
    // Success, transaction sent
    console.log("Sent transaction with ID = " + abcTransaction.txid)
  }
})
```

| Param | Type | Description |
| --- | --- | --- |
| abcTransaction | [`AbcTransaction`](#abctransaction) | Unsigned [AbcTransaction](#abctransaction) object |
| callback | `Callback` | (Javascript) Callback function |

| Callback Param | Type | Description |
| --- | --- | --- |
| abcError | [`AbcError`](#abcerror) | [AbcError](#abcerror) object |

Convenience routine to do `signTx`, `broadcastTx`, then `saveTx` in one call.

### getPaymentProtocolInfo

```javascript
// Example
const abcParsedUri = abcAccount.parseUri("bitcoin:1CsaBND4GNA5eeGGvU5PhKUZWxyKYxrFqs?amount=1.000000&r=https%3A%2F%2Fbitpay.com%2Fi%2F7TEzdBg6rvsDVtWjNQ3C3X")

const paymentProtocolInfo = await abcCurrencyWallet.getPaymentProtocolInfo(abcParsedUri.paymentProtocolURL)
const abcSpendInfo:AbcSpendInfo = {
    networkFeeOption: 'high',
    currencyCode: 'BTC',
    metadata: {
      name: paymentProtocolInfo.merchant,
      category: 'Expense:Professional Services'
    },
    spendTargets: [ paymentProtocolInfo.spendTarget ]
  }

  let abcTransaction = abcCurrencyWallet.makeSpend(abcSpendInfo)
  abcTransaction = await abcCurrencyWallet.signBroadcastAndSave(abcTransaction
  console.log("Sent transaction with ID = " + abcTransaction.txid)
}
```

| Param | Type | Description |
| --- | --- | --- |
| paymentProtocolURL | `String` | Payment protocol URL |
| callback | `Callback` | (Javascript) Callback function |

| Callback Param | Type | Description |
| --- | --- | --- |
| abcError | [`AbcError`](#abcerror) | [AbcError](#abcerror) object |
| abcPaymentProtocolInfo | [`ABCPaymentProtocolInfo`](#abcpaymentprotocolinfo) | [ABCPaymentProtocolInfo](#abcpaymentprotocolinfo) object |

Communicates over network with BIP70 payment request URL to get exact payment parameters. This returns a [ABCPaymentProtocolInfo](#abcpaymentprotocolinfo) which contains a custom, non-editable [AbcSpendTarget](#abcspendtarget) that can simply be placed into an [AbcSpendInfo](#abcspendinfo) object to send the transaction.


### getMaxSpendable

```javascript
// Example
const abcSpendInfo = {
  networkFeeOption: 'standard',
  metadata: {
    name: 'Tyra CPA',
    category: 'Expense:Professional Services',
  },
  spendTargets: [
    {
      publicAddress: '12xZEQL72YnGEbtW7bA4FPA1BUEHkxQoWN',
    }
  ]
}

abcCurrencyWallet.getMaxSpendable(abcSpendInfo, function(error, maxNativeAmount) {
  if (error === null) {
    console.log(maxNativeAmount)
  }
})
```

Get the maximum amount spendable from this wallet given the parameters of an [AbcSpendInfo](#abcspendinfo) object. The [AbcSpendInfo.spendTargets](#abcspendtarget) `nativeAmount` values are ignored when calculating the max spendable amount. This only ever returns the max spendable of the primary currency of the wallet. Any meta-tokens of the wallet will always have a max spendable equal to the number of meta-tokens in the wallet determined by [getBalance](#getbalance).

### sweepPrivateKey

coming soon...

### exportTransactionsToCSV

coming soon...

### exportTransactionsToQBO

coming soon...

## AbcSpendInfo

```javascript
// Example spend info to transfer from wallet to wallet
const walletIds = abcAccount.listWalletIds()
const srcWallet = abcAccount.getWallet(walletId[0]) // Add check for null and correct wallet type
const destWallet = abcAccount.getWallet(walletId[1]) // Add check for null and correct wallet type

abcSpendInfo = {
  networkFeeOption: 'high',
  currencyCode: 'BTC',
  metadata:  {
    name: 'Transfer to College Fund',
    category: 'Transfer:Wallet:College Fund',
  },
  spendTargets: [
    {
      destWallet,
      nativeAmount: '210000000' // 2.1 BTC
    },
  ]
}
```

Parameters

| Param | Type | Description |
| --- | --- | --- |
| currencyCode | `String` | Chooses the currency or meta-token to spend from. (Required) |
| noUnconfirmed | `Boolean` | (Optional) If set to TRUE, this will not spend from any unconfirmed funds. Default is FALSE |
| spendTargets | `Array` | Array of [AbcSpendTarget](#abcspendtarget) objects |
| networkFeeOption | `String` | Adjusts network fee amount. Must be either "low", "standard", "high", or "custom". If unspecified, the default is "standard" |
| customNetworkFee | `String` | Amount of per byte network fee if `networkFeeOption` is set to `custom`. Should be specified as smallest denomination of currency, as a string (i.e. Satoshis or Wei) |
| nativeAmount | `String` | (Optional) Only used for currency exchange (ie. Shapeshift) to specify the amount to exchange using the source currency instead of the destination currency. It is an error to specify `nativeAmount` in AbcSpendInfo if it is also specified in an AbcSpendTarget.
| metadata | [`AbcMetadata`](#abcMetadata) | [AbcMetadata](#abcMetadata) object. Outgoing transaction will have the specified metadata copied to the [AbcTransaction](#abctransaction) object |

Parameter object used for creating an unsigned [AbcTransaction](#abctransaction) object.

## AbcSpendTarget

```javascript
// Example spend target with a public addresses
const spendTarget =
  {
    currencyCode: 'BTC',
    publicAddress: '1CsaBND4GNA5eeGGvU5PhKUZWxyKYxrFqs',
    nativeAmount: '210000000' // 2.1 BTC
  }

// Example to spend to a BIP70 payment request
const abcParsedUri = abcAccount.parseUri("bitcoin:1CsaBND4GNA5eeGGvU5PhKUZWxyKYxrFqs?amount=1.000000&r=https%3A%2F%2Fbitpay.com%2Fi%2F7TEzdBg6rvsDVtWjNQ3C3X")

abcCurrencyWallet.getPaymentProtocolInfo(abcParsedUri.paymentProtocolURL, function(error, paymentProtocolInfo) {
  abcSpendInfo = {
    networkFeeOption: 'high',
    metadata: {
      name: paymentProtocolInfo.merchant,
      category: 'Expense:Professional Services'
    },
    spendTargets: [ paymentProtocolInfo.spendTarget ]
  }

  abcCurrencyWallet.makeSpend(abcSpendInfo).then(abcTransaction => {
    abcCurrencyWallet.signBroadcastAndSave(abcTransaction).then(() => {
      console.log("Sent transaction with ID = " + abcTransaction.txid)
    })
  })
}
```

Parameters

| Param | Type | Description |
| --- | --- | --- |
| currencyCode | `String` | (Optional) Chooses the currency or meta-token type for the destination receiving address or wallet. If not specified, uses the primary currency of this wallet. If the specified `currencyCode` differs from that specified in the parent [`AbcSpendInfo`](#abcspendinfo) then this spend will utilize an exchange operation (if possible) using a 3rd party service such as Shape Shift. |
| publicAddress | `String` | Public address in the format of the current wallet's currency. This requires the `nativeAmount` field to be set. Must not set both `publicAddress` and `destWallet` |
| nativeAmount | `String` | Amount to send in the smallest denomination of the source wallet's currency, as a string. ie Satoshis or Wei |
| destWallet | [`AbcCurrencyWallet`](#abccurrencywallet) | Destination wallet to transfer funds to. Must also set `nativeAmount`. Must not set both `publicAddress` and `destWallet` |
| destMetadata | [`AbcMetadata`](#abcmetadata) | [AbcMetadata](#abcmetadata) object with which will tag the transaction in the destination wallet. Must only be used when `destWallet` is set. |


## AbcParsedUri

| Property | Type | Description |
| --- | --- | --- |
| publicAddress | `String` | Public address from URI |
| privateKey | `String` | Private key |
| bitIDURI | `String` | Full BitID URI |
| bitIDDomain | `String` | Domain name portion of BitID URI |
| bitIDCallbackUri | `String` | BitID Callback URI |
| paymentProtocolUri | `String` | BIP70 Payment Request URL |
| nativeAmount | `String` | Amount in the currency's smallest denomination, as a string (i.e. Satoshis or Wei)
| metadata | [`AbcMetadata`](#abcmetadata) | [AbcMetadata](#abcmetadata) object with info extracted from URI |
| returnUri | `String` | URI to send user after URI/payment has been processed |
| bitidPaymentAddress | `Bool` | True if BitID URI is requesting a payment address (experimental)|
| bitidKycProvider | `Bool` | True if BitID URI would like to provide KYC token (experimental) |
| bitidKycRequest | `Bool` | True if BitID URI is requesting a KYC token (experimental) |


Object contains various parts of a parsed URI depending on the source URI. Any of the properties may be `null`.

## ABCPaymentProtocolInfo

Object provides basic UI displayable info about a BIP70 payment request. Also includes an [AbcSpendTarget](#abcspendtarget) for use in an [AbcSpendInfo](#abcspendinfo) to send the transaction.

| Property | Type | Description |
| --- | --- | --- |
| domain | `String` | DNS name of originator of request |
| nativeAmount | `String` | Amount of request, as a string |
| memo | `String` | Memo field returned by merchant |
| merchant | `String` | Name of merchat (may be blank) |
| abcSpendTarget | [`AbcSpendTarget`](#abcspendtarget) | [AbcSpendTarget](#abcspendtarget) object that can be used in an [AbcSpendInfo](#abcspendinfo) |

## AbcReceiveAddress

| Property | Type | Description |
| --- | --- | --- |
| publicAddress | `String` | Raw public address in native format of wallet currency type. (ie. base58 for bitcoin, base16 for ethereum) |
| nativeAmount | `String` | Amount of request denominated in the smallest unit of this wallet's currency, as a string (i.e. Satoshis or Wei) |
| metadata | `AbcMetadata` | [AbcMetadata](#abcmetadata) object corresponding to this address. Any transactions receiving funds into this address will automatically have this metadata in the [AbcTransaction](#abctransaction) object.

## AbcMetadata

Non-blockchain transaction meta data associated to an [AbcTransaction](#abctransaction) or to an [AbcReceiveAddress](#abcreceiveaddress)

| Property | Type | Description |
| --- | --- | --- |
| name | `String` | Name of external recipient or sender of funds |
| category | `String` | Transaction category of format "Expense:Food & Dining". Category must be of the form [Category]:[Sub Category] where category is one of "Income", "Expense", "Transfer", or "Exchange". Income refers to incoming funds such as payroll or business sales. Expense is the purchase of goods or services. Transfer is a transfer of funds to/from another wallet or exchange account owned by the user. Exchange is the change of funds from one type of currency to another. If abcCurrencyWallet is of type bitcoin, an incoming transaction from the purchase of bitcoin with USD should be categorized as "Exchange:Buy Bitcoin". |
| notes | `String` | Misc notes field |
| amountFiat | `Float` | Amount of transaction in the wallet's fiat currency at the time of the transaction |
| bizId | `Int` | Unique bizId associated to a business listing in the Airbitz Business Directory |
| miscJson | `String` | Generic JSON string that can be used for additional meta data |

## AbcTransaction

Object represents a signed or unsigned transaction that may or may not be broadcast to the blockchain.

| Property | Type | Description |
| --- | --- | --- |
| wallet | [`AbcCurrencyWallet`](#abccurrencywallet) | [AbcCurrencyWallet](#abccurrencywallet) this transaction is from |
| currencyCode | `String` | Which blockchain currency or token this transaction applies to. |
| metadata | [`AbcMetadata`](#abcmetadata) | [AbcMetadata](#abcmetadata) of this transaction |
| txid | `String` | Transaction ID as represented by the wallet's crypto currency. For bitcoin this is base16. This parameter is `null` until [signTx](#signtx) is called. |
| date | `Date` | Date that transaction was broadcast, detected, or confirmed on the blockchain. If the tx detection date is after the confirmation time, then this is the confirmation time. `null` if transaction has not been broadcast |
| blockHeight | `Int` | Block number that included this transaction |
| nativeAmount | `String` | Amount of transaction in denomination of smallest unit of currency, as a string. Incoming funds are positive numbers. Outgoing funds are negative. Outgoing amounts should include the providerFee and networkFee. In the case of a transaction that both spends from the current wallet to the current wallet, the amount should be the net effect on the balance of the wallet. |
| providerFee | `String` | Additional app provider fees in denomination of smallest unit of currency, as a string (i.e. Satoshis or Wei) |
| networkFee | `String` | Fee paid to network (mining fee) in denomination of smallest unit of currency, as a string (i.e. Satoshis or Wei) |
| runningBalance | `String` | Running balance of entire wallet as of this transaction in the smallest unit of currency, as a string (i.e. Satoshis or Wei) |
| signedTx |`Array` | Buffer of signed transaction data with signature. `null` if transaction is unsigned |
| ourReceiveAddresses | `Array<string>` | Contains a list of addresses that belong to this wallet and have received money in this transaction. |
| otherParams |`Object` | Crypto currency specific data |

`otherParams` has the following parameters for bitcoin wallets

| Property | Type | Description |
| --- | --- | --- |
| isReplaceByFee | `Bool` | True if this transaction is marked as RBF (BIP125) |
| isDoubleSpend | `Bool` | True if this transaction is found to be a double spend attempt |
| inputOutputList | `Array` | Array of transaction inputs and outputs |

## AbcDataDump

An object that contains an entire dump of the memory and disk cache of a wallet.

| Property | Type | Description |
| --- | --- | --- |
| walletId | `string` | The Id of the dumped wallet |
| walletType | `string` | The wallet type of the dumped wallet |
| pluginType | `string` | The plugin type (network) of the dumped wallet |
| data | `any` | An object containing the actual dump. Every key/value contains a dump of a different cache from inside the wallet. different plugins (for example Ethereum and Bitcoin) might return different data structures |

## AbcExchangeRateCache

### addSource

```javascript
const sourceBitstamp = require('airbitz-core-js-bitcoin').exchangeRateSources.bitstamp
const sourceCoinbase = require('airbitz-core-js-bitcoin').exchangeRateSources.coinbase
const sourceBitcoinAverage = require('airbitz-core-js-bitcoin').exchangeRateSources.bitcoinaverage
const sourceBraveNewCoin   = require('airbitz-core-js-bitcoin').exchangeRateSources.bravenewcoin

const abcExchangeRateCache = new AbcExchangeRateCache()
abcExchangeRateCache.addSource(sourceBitstamp)
abcExchangeRateCache.addSource(sourceCoinbase)
abcExchangeRateCache.addSource(sourceBitcoinAverage)
abcExchangeRateCache.addSource(sourceBraveNewCoin)
```

Adds an exchange rate source. Initialize with object of type [AbcExchangeRateLib](#abcexchangeratelib). `airbitz-core-js-bitcoin` includes support for Bitstamp.com, Coinbase.com, BitcoinAverage.com, and BraveNewCoin.com

### convertCurrency

```javascript
destinationCurrencyAmount = convertCurrency(amount, sourceCurrency, destinationCurrency)

// Example
const priceofBitcoin = convertCurrency(1, "BTC", "USD")
```

| Param | Type | Description |
| --- | --- | --- |
| amount | `Number` | Amount of source currency to convert |
| sourceCurrency | `String` | 3 character currency code of source currency |
| destinationCurrency | `String` | 3 character currency code of destination currency |

| Return | Type | Description |
| --- | --- | --- |
| destinationAmount | `Number` | Amount of destination currency after conversion |

Converts one currency value into another using exchange rate cache. Returns 0 if the currency pair cannot be converted due to missing support from exchange rate sources or if sources cannot be reached.

# Core Plugin API

Core Plugins are Javascript SDKs that can provide additional functionality to ABC. These include support for different exchange rate services and the ability to transact on other blockchains. Edge provides three repos which supply exchange rate and blockchain transaction functionality. Developers wishing to add support to Edge for their blockchain or exchange rate service need to implement a JS library that exposes an `AbcCorePluginFactory` class implementing their service.

Users of Core Plugins will include them via import statements such as: 

```javascript
import { CoinbaseExchangePluginFactory, ShapeShiftExchangePluginFactory } from 'edge-exchange-plugins'
import { EthereumCurrencyPluginFactory } from 'edge-currency-ethereum'
import { BitcoinCurrencyPluginFactory, LitecoinCurrencyPluginFactory, BitcoincashCurrencyPluginFactory } from 'edge-currency-bitcoin'
```

The Edge core library accepts an array of plugin factory objects in its `makeContext` function. 

```javascript
const context = makeContext({
  plugins: [CoinbaseExchangePluginFactory, EthereumCurrencyPluginFactory, BitcoinCurrencyPluginFactory, LitecoinCurrencyPluginFactory, BitcoinCashCurrencyPluginFactory]
})
```

All core plugins begin in a common format, `AbcCorePluginFactory`, which describes the type of the plugin and provides an initialization function `makePlugin`. The initialization function returns either the `AbcCurrencyPlugin` or an `AbcExchangePlugin` object that implements the actual functionality.

## AbcCorePluginFactory

```javascript
// Example exchange plugin initialization
export const MyExchangePluginFactory = {
  pluginType: 'exchange',
  pluginName: 'shapeshift',

  async makePlugin(opts) {
    return new MyExchangePlugin(opts.io)
  }
}

// Example currency plugin initialization
export const MyCurrencyPluginFactory = {
  pluginType: 'currency',
  pluginName: 'ethereum',

  async makePlugin(opts) {
    return new MyEthereumPlugin(opts.io)
  }
}
```

This is a the basic wrapper type for all plugins. A single repo is able to export multiple plugin factories which can provide different functionality.

### pluginType

Either `exchange` or `currency`

### pluginName

Simple unique string representing the currency/exchange supported by this plugin. Will be used for comparison of uniqueness
against other plugins.

ie.

```javascript
console.log(currencyPluginFactory.pluginName) // => "ethereum"
```

### makePlugin

Returns either an `AbcCurrencyPlugin` or `AbcExchangePlugin` object instance, depending on the `pluginType` property described above. This is an async method. Due to the async nature of `makePlugin`, this routine can dynamically change its functionality based on server based calls. This type of dynamic functionality change can include updating its `currencyInfo` include more supported tokens.

| Param | Type | Description |
| --- | --- | --- |
| opts | `Object` | Options for the plugin |

The options are as follows:

| Property | Type | Description |
| --- | --- | --- |
| io | `Object` | Platform-specific resources |

| Return | Type | Description |
| --- | --- | --- |
| plugin | `Promise<AbcCurrencyPlugin or AbcExchangePlugin>` | A promise that resolves to the inner plugin instance |

Note: The `io` object passed in the `options` structure contains a `folder` member. This is a [Disklet](https://www.npmjs.com/package/disklet) folder, and can be used to store app-wide information. A currency plugin might use this folder to store generic blockchain information such as the last block height or supported tokens, for example.

## AbcCurrencyPlugin

```javascript
// SDK users should do this:
import { EthereumCurrencyPluginFactory } from 'airbitz-currency-ethereum'

const context = makeContext({
  plugins: [ EthereumCurrencyPluginFactory ]
})

// Inside the `makeContext` function, this is what happens:
const currencyPlugin = await ethereumPlugin.makePlugin({ io })
```

Cryptocurrency functionality for [`AbcCurrencyWallet`](#abccurrencywallet) is provided by currency plugins. These begin life as a generic [AbcCorePluginFactory](#abccorepluginfactory) that returns an `AbcCurrencyPlugin` instance from the `createPlugin` method.

The `AbcCurrencyPlugin` object provides all the functionality needed to integrate the currency into the user interface, manage keys, and handle URI's. It also provides the ability to create `AbcTxEngine` objects, which provide the send, receive, and transaction history functionality for individual wallets.

### currencyInfo

```javascript
console.log(currencyPlugin.currencyInfo)
"
{
  walletTypes: [
    'wallet:ethereum'
  ],
  pluginName: 'ethereum',
  currencyName: 'Ethereum',
  currencyCode: 'ETH', // The 3 character code for the currency
  explorerAddress: 'https://etherscan.io/address/%s',
  explorerTransaction: 'https://etherscan.io/tx/%s',
  denominations: [
    // An array of Objects of the possible denominations for this currency
    {
      name: 'ETH',
      multiplier: '1000000000000000000',
      symbol: 'Ξ'
    }
  ],
  defaultSettings: {
    etherscanApiServers: [
      'https://api.etherscan.io'
    ],
    superethServers: [
      'http://supereth1.airbitz.co:8080'
    ]
  },
  symbolImage: 'data:image/png;base64,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', // Base64 encoded png image of the currency symbol (optional)
  symbolImageDarkMono: 'data:image/png;base64,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', // Base64 encoded png image of the currency symbol (optional)
  metaTokens: [
    // Array of objects describing the supported metatokens
    {
      currencyCode: 'REP',
      currencyName: 'Augur',
      denominations: [
        {
          name: 'REP',
          multiplier: '1000000000000000000'
        }
      ],
      contractAddress: '0xE94327D07Fc17907b4DB788E5aDf2ed424adDff6',
      symbolImage: 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADwAAAA8CAYAAAA6/NlyAAAAAXNSR0IArs4c6QAACEZJREFUaAXVmmmoVVUUx33PHDKz1CTrJYlWhhjkkEXapEXDlzTDwEBNswlDs6KCyNKEsjCHPhQRvdSKkgZ7QgUN+sQGsKwPZllkpDbQhM1ldfv979vrse++55x77n33HF8L/m+vvaa91hn2cO7r0iVnKhQKU8En4HMwK+fh8x2OAkeC/cDoX5gz880ix9Eo7nWr1Gu3wjfkmEY+Q1HUxV6RITsjnyxyGoXquoGPXZV7vWr3OF5tr5zSyX4YipnnFTnT42d4/MLsM8lhBArqC753hb1IO8jxasS/4Pq/0B6dQ0rZDkERD7iCNDsPA2HBJyD7y9k0Z5tNxtEp4jivmJUajn5JwU623BX8D+3IjNPKLjzJ2+P6I3x/V1xUwf3Q/wBEb2SXUYaRSfwsZe9ogQ1Fv6xgdyHmmzHtJLP/X7Qk3ADedQV8StvdEoePK7g7Om05RTtBN/Pp9C3J+kvPJX7C6CILlg26ScBovu/XaXmy7QVsc9EaJooutmDZot8ERHqn+4X+na5PkncqW0gHgzFhgsgqFTza+dIUVoT+napPgk3gV2UKrY5KDnliwfLBplkBIK3Px0fF6RQykntcWUK/gWOikkKepmD/wq2PinPAZRQyCugxFi2KSwhdxYLli529Gop3dly8AyYnqY3KDPoK9I5LBF3agg/B9ksgeg80xsXMXU4yk5WVo1lJCWCTqmDFwHamBaW9IilubjoS0VnXNgzvwyfeCfTHAqOmpEQxagTbnLGWukOS7HPRkcQNLiE1EysNis1cZ6/ZPPHiKBY25zh7NXdVip+pngT8TX9LpcGw16P/OxClXmOxtUOILlLiU1Ephw7plTQQ7QfD4oKhGwgeBjaLvwbfE+h1WAb0rsbuqtD5Z+bVceNkKicJHeZVqGhV1GDINdMuAfqaYfQyTPFdpF1pQlqt3avAETGx7OLqoo2OsslUxqAvAlH7WdcfEPkYoFOPkfbGc8wG/lCwCOgC2BcP2MI3YILZWYvMf31aTZ5Lq4SA0Y3hoCgmAtti7oO/DRwW2lkf3QBwP/gDiP4EZRMgshukdFRyCrNYdW8ZTEuFlh9RyVlXgyEbAX6SEvoMDE+bBLYjwRdApAt1su9L318Cy8b2bevGM+hsYDTFD4zwIPCBU35EO8DXp+HxGQx2uxi7aNs/Hsif/mSnU7MgTcyabRigN7Dt3uYwELqrgEjv5IhQn7aP7xnAZvTrQj90G4Eocv4I7WvuM8Di4jBtyZwSBkK3w+kfCHXV9oljJy/9KtHD96fvn5kjVwjfviaeQY4BWjpEa8IgyE4tatouxrGhvto+sVSU0emhPwq7IIl7gNAvdZ8B1rjRVfSg0BHZ7U6/PdTV0idWA/jWxbw+jIG8CdhK0BLqO9QnsNZUe6cWRwVD/yQQPR+lr0VGrKeLEQuFdVH+6O5yejVly1iUTyoZwVpd4NizLvoWZ1O3rR/xrnQxtcwdHCaLTDs5naJEFU9qoX9kn0BTiuHa/syONEKI2h75uv1yQMwjgG1GrokaG/1MYBSbX5RvmYwo3YEWeJHW18YyIydANw7oVLMnzqYWOfEeAyKtzX3CGMi0EdIXEVHsExj6RfYJcGMxTNufc6OMUHX15fSr3mz4/sYTp1gc7WBgh49n4UuWKNkjOwcYLbYYVbV463HSwi7a4DvT1953HtgKtCxo5t4ErgZlCfm+aXni3AOmy552LjDaBnNRGAfZemcQuYqE9mV9nB90AVTQiTJQCx4B9l7BlpEmkcvLAlYhwL8JaKL6DhSfGFodM22lgC28DS6wsPAnADt1rTV5qhZHFaZCRSr8ePAE+AcYSa+rqpl0PngH+PQMndivl3GJ4KOfara4QPtoB5ot/HigJ8knjVu847QrnEIXpmwnaHHKWow3OMe/aZ8CVrzE+veFO8CRoSOy88HHwEi/IB4V2sX1sdWZ14pVjMgvlcgngs0y8Eiv1yxgr+HmuHFK5DhM8IL4rAq9GSTeNfQHg4eAkRKYA5Jm+Ab004BmWaNlJYlFdDBUrhvNwbX+Y1/5zIzTo0EAva9LweERY8aKsNcEZh/sYItfP1bSXg7aPwbA625tB0Z6mm6JDRyhwF6nq1csgNdW3vVhfJ/noC8PNf9rIL6aSF714ol9CXRzeEQCjz6APzWiplQifC8E/oTaXNERhyOBpn6jn2FuBSUH8YqBPAN8x4Ll4C3QH+hDwTpgtAtmKqjpXxDx6wquA/puZvQ5zBAvjXgWQ90BbTq0NBjthClb/+KjxGuIozXWaC1M4rwQH6m4VGrm1j7aSEuTXsHqY+I0EKwGPrXQGZqURJIOX91dO9bphFXrXT0K3zXAp5fpxH4fT8qrREeQccD2qxpA78kSUPVvPfgMAUbjSwZK0cFRT99NwH/6dtGv73/+ELARXAu0PBnthrksRZ7tJthrIjMa265IweB0HthhzrTaRi4EPVO412ZCcE06DwF/16W18KQ0EbGrumB8BoPngE/qD04zZl1sGGwUeNPLQLuyVaBv0gDoUxeMbU+wEOhOGukOn5c0RmY6BtYOaQb4GhjpO9QcELmzQp6qYOwmgc+Akd5ZvbsH/h/WSKIPWAb2AyPtbU8LrzayxILRDwOabX3SbJx6Tx6OmVmfpIaD17xMta9tBv6JJ7JgbHqDe4HWUSNtgMZllnC9ApPkpeALy5pWx7wFQEtKWcHIpoG9wEgrgVaEyNeiXnnWNQ7J9gJ3A63ZRh/CaAtopHe91Tq0mvm1AvSvazJ5BiP5oUC7s0q0BYNReeaW6VgUcxH4JKJqnYGng5q2mJkm3dHgFNUD3Ap2Ar3jS0HZp9eOjpPk/x/aIb1a/DSlzQAAAABJRU5ErkJggg=='
    },
    {
      currencyCode: 'WINGS',
      currencyName: 'Wings',
      denominations: [
        {
          name: 'WINGS',
          multiplier: '1000000000000000000'
        }
      ],
      contractAddress: '0x667088b212ce3d06a1b553a7221E1fD19000d9aF'
    }
  ]
}
"
```

The `currencyInfo` property should be an object with the following properties describing the currency:

| Property | Type | Description |
| --- | --- | --- |
| pluginName | `String` | Unique string denoting the currency supported by this plugin. ie "ethereum" |
| currencyName | `String` | Human readable string of the currency name. ie "Ethereum" |
| currencyCode | `String` | The 3-5 character code for the currency. ie. "ETH" |
| addressExplorer | `String` | Full URL of explorer to give info on a specific address. ie. `https://etherscan.io/address/%s` |
| transactionExplorer | `String` | Full URL of explorer to give info on a specific tx. ie. `https://etherscan.io/tx/%s` |
| denominations | `Array` | An array of AbcDenomination objects of the possible denominations for this currency |
| symbolImage | `String` | Base64 encoded png or jpg image of the currency symbol designed to be placed on a dark background (optional) |
| symbolImageDarkMono | `String` | Base64 encoded png or jpg image of the currency symbol in a monochrome (grey) color over transparent (optional) |
| metaTokens | `Array` | Array of AbcMetaToken objects describing the supported metatokens |
| walletTypes | `Array` | Array of strings listing the wallet types that this currency can handle, such as `wallet:ethereum`. Please see the [AbcWalletInfo](#abcwalletinfo) documentation for information about these types. The `makeEngine`, `createPrivateKey`, and `derivePublicKey` methods can handle keys with these types. |
| defaultSettings | `object` | Default per-currency settings. This acts as a template for the settings that should be passed to [`makeEngine`](#makeengine) and [`updateSettings`](#updatesettings). Optional. |

The `AbcDenomination` object includes the following properties:

| Property | Type | Description |
| --- | --- | --- |
| name | `String` | The human readable string to describe the denomination. |
| multiplier | `Number` | The value to multiply the smallest unit of currency to get to the denomination. |
| symbol | `String` | The human readable 1-3 character symbol of the currency. ie. "Ƀ" |
| font | `String` | (Optional) The font required to display the symbol specified above. If not given, will use the default system font. |

The `AbcMetaToken` array includes the following properties:

| Property | Type | Description |
| --- | --- | --- |
| currencyCode | `String` | The human readable string to describe the denomination. |
| denominations | `Array` | An array of AbcDenomination objects of the possible denominations for this currency |
| symbolImage | `String` | Base64 encoded png or jpg image of the currency symbol (optional) |

### parseUri

```javascript
const abcParsedUri = currencyPlugin.parseUri("bitcoin:1CsaBND4GNA5eeGGvU5PhKUZWxyKYxrFqs?amount=1.2345&r=https%3A%2F%2Fbitpay.com%2Fi%2F7TEzdBg6rvsDVtWjNQ3C3X&label=Mom")

console.log(abcParsedUri.publicAddress) // -> 1CsaBND4GNA5eeGGvU5PhKUZWxyKYxrFqs
console.log(abcParsedUri.nativeAmount) // -> '123456700'
console.log(abcParsedUri.metadata.name) // -> 'Mom'
console.log(abcParsedUri.paymentProtocolURL) // -> https://bitpay.com/i/7TEzdBg6rvsDVtWjNQ3C3X
```

| Param | Type | Description |
| --- | --- | --- |
| uri | `String` | URI to parse |

| Return Param | Type | Description |
| --- | --- | --- |
| abcParsedUri | [`AbcParsedUri`](#abcparseduri) | Object with parsed parameters |

Parses a URI extracting various elements into an [AbcParsedUri](#abcparseduri) object

### encodeUri

```javascript
// Example
const abcParsedUri = {
  publicAddress: '1CsaBND4GNA5eeGGvU5PhKUZWxyKYxrFqs',
  nativeAmount: '123456700',
  metadata: {
    name: 'Mom'
  },
  paymentProtocolURL: 'https://bitpay.com/i/7TEzdBg6rvsDVtWjNQ3C3X'
}

const uri = currencyPlugin.encodeUri(abcParsedUri)

console.log(uri)
// ->  "bitcoin:1CsaBND4GNA5eeGGvU5PhKUZWxyKYxrFqs?amount=1.2345&label=Mom&r=https%3A%2F%2Fbitpay.com%2Fi%2F7TEzdBg6rvsDVtWjNQ3C3X")

```

| Param | Type | Description |
| --- | --- | --- |
| abcParsedUri | [`AbcParsedUri`](#abcparseduri) | Object with parsed parameters |

| Return Param | Type | Description |
| --- | --- | --- |
| uri | `String` | URI output |

Encodes a URI given a [AbcParsedUri](#abcparseduri) object

### createPrivateKey

```javascript
// Example
const walletKey = currencyPlugin.createPrivateKey('wallet:ethereum')

// Output:
{
  "ethereumKey": "65256374d98202d11b22d74a5d89960cf50d71f45a3d4f7641e1c2ce3b2bdc89"
}
```

Creates a new random master private key. The returned object will be used as the `keys` member of an [`AbcWalletInfo`](#abcwalletinfo) structure. Please see [`AbcWalletInfo`](#abcwalletinfo) for documentation on the various key types Edge understands. If your currency is not documented in that section, please submit a pull request to add your format to [the documentation](https://github.com/Airbitz/airbitz-docs/tree/full-api-docs).

| Param | Type | Description |
| --- | --- | --- |
| type | `string` | The type of wallet to create. See [`AbcWalletInfo`](#abcwalletinfo) for valid wallet types. |

The core will treat the returned object as the `keys` member of an [`AbcWalletInfo`](#abcwalletinfo) object. The core will generate a random `id`, and will use the same `type` as passed to `createPrivateKey`. The core will also insert the `dataKey` and `syncKey`, so the currency plugin does not need to worry about those either.

### derivePublicKey

```javascript
// Example
const walletInfo = {
  "type": "wallet:ethereum",
  "id": "lZvB9W1waiDHn52JRzUjfqAbyp1wyN5jreKbdyto4pI=",
  "keys": {
    "ethereumKey": "65256374d98202d11b22d74a5d89960cf50d71f45a3d4f7641e1c2ce3b2bdc89"
  }
}
const readOnlyWalletKey = currencyPlugin.derivePublicKey(walletInfo)

// Output:
{
  "ethereumAddress": "0x9ba8075585b5fad5c0455894a667715374f1ed63"
}
```

Converts a spending-capable [`AbcWalletInfo`](#abcwalletinfo) structure into a receive-only wallet. The core will treat the return value as the `keys` member of a new [`AbcWalletInfo`](#abcwalletinfo) structure. This has multiple uses:

1. Saving a unencrypted wallet on the local device for offline balance checks.
2. Sharing a wallet with another user is a watch-only mode.

For Bitcoin, this means converting the private seed into an xpub-format key. For Ethereum, it means converting the private key into a payment address. Please see [`AbcWalletInfo`](#abcwalletinfo) for documentation on the various key types Airbitz understands. If your currency is not documented in that section, please submit a pull request to [the documentation](https://github.com/Airbitz/airbitz-docs/tree/full-api-docs).

| Param | Type | Description |
| --- | --- | --- |
| walletInfo | [`AbcWalletInfo`](#abcwalletinfo) | The private keys to the wallet. |

The core will treat the returned object as the `keys` member of a new [`AbcWalletInfo`](#abcwalletinfo) object. The core also manages the `dataKey` and `syncKey`, so the currency plugin does not need to worry about those either.

### displayPrivateKey

```javascript
// Bitcoin bip44 wallet example
const privateKey: string = currencyPlugin.displayPrivateKey(walletInfo)
console.log(privateKey) // => "maximum head mouse racket bottle quiver button horse notebook cold simply massive"
```

Returns a user displayable string representing the master private key of this wallet. This would be a 12-24 word passphrase for Bitcoin BIP44/49 wallets and a 256bit hex string for ethereum wallets.

### makeEngine

```javascript
// Example
function onTransactionsChanged(txids) {
  // your_callback_here
}

const callbacks = {
  onAddressesChecked,
  onBalanceChanged,
  onBlockHeightChanged,
  onTransactionsChanged
}
const options = {
  callbacks,
  walletFolder,
  walletLocalFolder,
  optionalSettings
}

const currencyEngine = await currencyPlugin.makeEngine(walletInfo, options)
```

This function creates an [`AbcCurrencyEngine`](#abccurrencyengine) object to send, receive, and list transactions for an individual wallet instance.

| Param | Type | Description |
| --- | --- | --- |
| walletInfo | [`AbcWalletInfo`](#abcwalletinfo) | The keys to the wallet. This may include just the pubic key (no private key) in read-only scenarios. See the [`AbcWalletInfo`](#abcwalletinfo) documentation for details. |
| options | `Object` | Options for [`currencyPlugin.makeEngine`](#makeengine) |

| Options | Type | Description |
| --- | --- | --- |
| callbacks | [`AbcCurrencyPluginCallbacks`](#abccurrencyplugincallbacks) | Various callbacks when wallet is updated. |
| walletFolder | `Folder` | [Disklet](https://www.npmjs.com/package/disklet) folder for synced and encrypted data. May not be present in certain read-only scenarios. |
| walletLocalFolder | `Folder` | [Disklet](https://www.npmjs.com/package/disklet) folder for non-encrypted, device-only data. |
| optionalSettings | `object` | Per-currency settings. The plugin should provide a template for this in its [`currencyInfo`](#currencyinfo). Optional. |

| Return | Type | Description |
| --- | --- | --- |
| engine | `Promise<AbcCurrencyEngine>` | A promise that will resolve to the requested engine, or an error. |

The fresh [`AbcCurrencyEngine`](#abccurrencyengine) instance should either load cached transactions from disk, if available, or start with a blank transaction list if this is its first time running for a particular wallet. This makes the [`AbcCurrencyEngine`](#abccurrencyengine) usable right away, creating a fast start-up experience.

The [`AbcCurrencyEngine`](#abccurrencyengine) instance should only start querying the blockchain for transactions after the core calls `startEngine`. There are cases where the user will want to view their cached transactions without hitting the network, such as with archived walles. In these cases, the core will never call `startEngine`.

The engine should store its per-wallet transaction cache inside the provided `walletLocalFolder`. This location is unencrypted, so the plugin must never cache private keys in here.

Multiple engines may be watching the same blockchain. If these engines would like to share common chain informaiton, such as the last block height or SPV headers, they can maintain a shared cache in the `opts.io.folder` folder passed to the [AbcCorePlugin](#abccoreplugin)'s `createPlugin` method. This is an unencrypted app-wide location.

The `walletFolder` location should never be used for normal currencies. This location is for encrypted and backed-up metadata. There are proposals such as payment channels that may require metadata in the future, but normal currencies don't need this.

All these locations are [Disklet](https://www.npmjs.com/package/disklet) folders. Please see the Disklet documentation for examples of how to access them, or see the sample code for [AbcStorageWallet](#abcstoragewallet).

## AbcCurrencyEngine

An `AbcCurrencyEngine` object sends, receives, and lists transactions for a single user wallet. The [AbcCurrencyPlugin](#abccurrencyplugin) object creates `AbcCurrencyEngine` instances through its `makeEngine` method.

### updateSettings

```javascript
currencyEngine.updateSettings(settings)
```

| Param | Type | Description |
| --- | --- | --- |
| settings | `object` | A settings object in the same format as `defaultSettings` provided by [`currencyInfo`](#currencyinfo). |

If the settings change while the engine is running (due to user action), provides the new settings to the plugin.

Returns void. This method is optional if the currency does not have settings.

### startEngine

```javascript
await currencyEngine.startEngine()
```

Begins checking the blockchain for incoming transactions. Prior to this method, the engine should simply return whatever cached data it has (or nothing, if this is the first run).

| Return | Type | Description |
| --- | --- | --- |
| promise | `Promise<void>` | A promise that resolves when the engine has fully initialized. |

### killEngine

```javascript
await currencyEngine.killEngine()
```

Stops checking the blockchain for new transactions, and flushes any caches to disk. It should be safe to shut down the app once this method completes.

| Return | Type | Description |
| --- | --- | --- |
| promise | `Promise<void>` | A promise that resolves when the engine has fully shut down. |

### getDisplayPrivateSeed

```javascript
const privateSeed = abcTxEngine.getDisplayPrivateSeed()
if (privateSeed !== null) {
  console.log(privateSeed)
}
```

Gets the master private seed (or private key if wallet is initialized from key and not from seed) from the engine as a string

| Return | Type | Description |
| --- | --- | --- |
| privateSeed | `String | null` | The private seed if exists |

### getDisplayPublicSeed

```javascript
const publicKey = abcTxEngine.getDisplayPublicSeed()
if (publicKey !== null) {
  console.log(publicKey)
}
```

Gets the master public key (or address in case of the ethereum plugin) from the engine as a string

| Return | Type | Description |
| --- | --- | --- |
| publicKey | `String | null` | The public key if exists |

### resyncBlockchain

```javascript
const publicKey = await abcTxEngine.resyncBlockchain()
```

Wipes out the local disk and memory cache of all transactions and forces a refetching of all data from the network.

| Return | Type | Description |
| --- | --- | --- |
| promise | `Promise<void>` | A promise that resolves when the cache has been cleared. |

### dumpData

```javascript
abcTxEngine.dumpData()

// Example
const dataDump = abcTxEngine.dumpData()
console.log(dataDump.walletId)
console.log(dataDump.walletType)
console.log(dataDump.pluginType)
console.log(dataDump.data)
```

Returns the entire memory and disk states of the wallet for debugging purposes

| Return | Type | Description |
| --- | --- | --- |
| dataDump | [`AbcDataDump`](#abcdatadump) | An object containing the dump of the entire wallet cache and memory |

### enableTokens

```javascript
// Example
const tokens = {
  tokens: [ "XCP", "TATIANACOIN" ]
}

currencyEngine.enableTokens(tokens).catch(handleError)
```

| Param | Type | Description |
| --- | --- | --- |
| tokens | `Array` | Array of strings specifying the currency codes of tokens to enable in this wallet |

| Return | Type | Description |
| --- | --- | --- |
| promise | `Promise<void>` | A promise that resolves when the tokens are ready to use. |

Enable support for meta tokens (ie. counterparty, colored coin, ethereum ERC20). Library should begin checking the blockchain for the specified tokens and triggering the callbacks specified in [`currencyPlugin.makeEngine`](#makeengine).

### disableTokens

```javascript
// Example
const tokens = {
  tokens: [ "XCP", "TATIANACOIN" ]
}

abcTxEngine.disableTokens(tokens).catch(handleError)
```

| Param | Type | Description |
| --- | --- | --- |
| tokens | `Array` | Array of strings specifying the currency codes of tokens to disable in this wallet |

| Return | Type | Description |
| --- | --- | --- |
| promise | `Promise<void>` | A promise that resolves when the tokens are disabled. |

Disable support for meta tokens (ie. counterparty, colored coin, ethereum ERC20). Library will stop checking the blockchain for the specified tokens.

### getEnabledTokens

```javascript
getEnabledTokens():Promise<Array<string>>

// Example
try {
  const tokens = await currencyEngine.getEnabledTokens()
  console.log(tokens) // => ['WINGS', 'REP']
} catch (error) {
  console.log(error)
}
```

| Promise Return Param | Type | Description |
| --- | --- | --- |
| tokens | `Array<string>` | Array of strings of token currency codes. |

Query the library for the currently enabled tokens.

### addCustomToken

```javascript
// Example for ethereum plugin
const customTokens = {
  contractAddress: '',
  currencyCode: 'WINGS',
  multiplier: '1000000000000000000'
}

await abcTxEngine.addCustomToken(customTokens)
```

| Param | Type | Description |
| --- | --- | --- |
| customTokens | `Object` | Plugin specific object describing parameters for a token |

Enable support for a custom meta token

### getBlockHeight

```javascript
// Example
var blockHeight = currencyEngine.getBlockHeight()
console.log(blockHeight)
"455487"
```

Retrieve the current block height from the network.

### getBalance

```javascript
// Example
const balance = currencyEngine.getBalance(options)
```

| Param | Type | Description |
| --- | --- | --- |
| options | `Object` | Options parameters below |

| Option Params | Type | Description |
| --- | --- | --- |
| currencyCode | `String` | Currency code to use. ie "REP", "LTBCOIN". If not specified, uses the wallet's primary currency. ie. "BTC", or "ETH" |

| Return Param | Type | Description |
| --- | --- | --- |
| balance | `Int` | Balance in the smallest unit of the currency |

Get the current balance of this wallet in the currency's smallest denomination (ie. satoshis)

### getNumTransactions

```javascript
// Example
const numTransactions = currencyEngine.getNumTransactions(options)
```

| Param | Type | Description |
| --- | --- | --- |
| options | `Object` | Options parameters below |

| Option Params | Type | Description |
| --- | --- | --- |
| currencyCode | `String` | Currency code to use. ie "REP", "LTBCOIN". If not specified, uses the wallet's primary currency. ie. "BTC", or "ETH" |

| Return Param | Type | Description |
| --- | --- | --- |
| numTransactions | `Int` | Number of transactions in wallet |

Get the number of transactions in the wallet

### getTransactions

```javascript
const options = {
  startIndex: 5,
  numEnteries: 50
}

currencyEngine.getTransactions(options)
  .then(transaction => {
    console.log(transactions[0].txid) // => "1209befa09ab3efc039abf09490ac34fe09abc938"
  })
  .catch(handleError)
```

| Param | Type | Description |
| --- | --- | --- |
| options | `Object` | Options for getTransactions. If `null`, return all transactions |
| callback | `Callback` | (Javascript) Callback function |

| Return Param | Type | Description |
| --- | --- | --- |
| transactions | `Promise<Array<AbcTransaction>>` | A promise that resolves to an array of [AbcTransaction](#abctransaction) objects |

Returns an array of transactions matching the options specified. The plugin must fill in the following [AbcTransaction](#abctransaction) fields:

* `txid`
* `date`
* `networkFee`
* `blockHeight` (may be 0)
* `nativeAmount`
* `ourReceiveAddresses`

The remaining fields are updated by Airbitz Core.

The `options` parameter may include the following:

| Options Params | Type | Description |
| --- | --- | --- |
| currencyCode | `String` | Currency code to use. ie "REP", "LTBCOIN". If not specified, uses the wallet's primary currency. ie. "BTC", or "ETH" |
| startIndex | `Int` | The starting index into the list of transactions. 0 specifies the newest transaction |
| numEntries | `Int` |  The number of entries to return. If there aren't enough transactions to return `numEntries`, then the plugin should return the maximum possible |

### getTxidList

```javascript
const txidList: Array<string> = abcTxEngine.getTxidList()
```

Returns an array of all the txids that this wallet currently has. All txids returned in this list should be fully queried from the network with a correct `nativeAmount`.

### getFreshAddress

```javascript
const addressObj = currencyEngine.getFreshAddress(options)

console.log(addressObj) // =>

"
{ 
  'publicAddress': '1EiwDW9VjTUbLJtuPYUkbj5fJDnc36WKdz', 
  'segwitAddress': 'bc1qw508d6qejxtdg4y5r3zarvary0c5xw7kv8f3t4'
}
"
```

| Param | Type | Description |
| --- | --- | --- |
| options | `Object` | Options object documented below |

| Option Params | Type | Description |
| --- | --- | --- |
| currencyCode | `String` | Currency code to use. ie "REP", "LTBCOIN". If not specified, uses the wallet's primary currency. ie. "BTC", or "ETH" |

| Return | Type | Description |
| --- | --- | --- |
| addressObj | `Object` | Object of type `{ publicAddress: string, segwitAddress?: string }` |

Returns an address object that has never received funds. `getFreshAddress` will consider any addresses that have been sent to `addGapLimitAddresses` as having received funds and will not return such addresses in this call.

### addGapLimitAddresses

```javascript
const abcError = currencyEngine.addGapLimitAddresses(addresses, options)
```

| Param | Type | Description |
| --- | --- | --- |
| addresses | `Array` | Array of Strings containing public addresses |
| options | `Object` | Options object documented below |

The `options` parameter may include the following:

| Options Params | Type | Description |
| --- | --- | --- |
| currencyCode | `String` | Currency code to use. ie "REP", "LTBCOIN". If not specified,

| Return | Type | Description |
| --- | --- | --- |
| abcError | [`AbcError`](#abcerror) | [AbcError](#abcerror) object |

When implementing an HD wallet with multiple addresses, wallet implementations typically search for funds by going a limited number of addresses ahead of the last address that has funds received. This is usually about 10 addresses. `addGapLimitAddresses` allows ABC to specify to the plugin to treat the given addresses as if they had funds received and to forward their gap limit accordingly.

### isAddressUsed

```javascript
const isUsed = currencyEngine.isAddressUsed(address, options)
```

| Param | Type | Description |
| --- | --- | --- |
| address | `String` | String of public address to query |
| options | `Object` | Options parameters below |

The `options` parameter may include the following:

| Options Params | Type | Description |
| --- | --- | --- |
| currencyCode | `String` | Currency code to use. ie "REP", "LTBCOIN". If not specified,

| Return | Type | Description |
| --- | --- | --- |
| isUsed | `Boolean` | True if address has ever received money |

### makeSpend

```javascript
const abcTransaction = await abcTxEngine.makeSpend(abcSpendInfo)
```

Given an [AbcSpendInfo](#abcspendinfo) object, returns an unsigned [AbcTransaction](#abctransaction) object. [AbcTransaction](#abctransaction).signedTx should be NULL. `makeSpend` does not need to touch the metadata parameter in the `abcSpendInfo`. `makeSpend` only needs to support the [AbcSpendTarget](#abcspendtarget) parameters `currencyCode`, `publicAddress`, and `nativeAmount`.

Should produce an [InsufficientFundsError](#insufficientfundserror) if the amount is too large, or a [DustSpendError](#dustspenderror) if the amount is too small.

### signTx

```javascript
await abcTxEngine.signTx(abcTransaction)
```

This routine will set [AbcTransaction](#abctransaction).signedTx to an Array of bytes corresponding to the complete signed transaction. Takes an unsigned [AbcTransaction](#abctransaction) object and signs it.

### broadcastTx

```javascript
await abcTxEngine.broadcastTx(abcTransaction)
```

Saves an already signed [AbcTransaction](#abctransaction) object to the local cache so that funds are considered spent by the wallet. Any future calls to [getTransactions](#gettransactions), [getBalance](#getbalance), or [getNumTransactions](#getNumTransactions) should reflect the outcome of this saved transaction. This routine should also trigger the callback [transactionsChanged](#transactionschanged).

## AbcCurrencyPluginCallbacks

### onAddressesChecked

```javascript
onAddressesChecked(progressRatio)
```
| Param | Type | Description |
| --- | --- | --- |
| progressRatio | `Number` | 0 to 1 value indicating how far along the core is in checking all the wallet addresses for new funds. This is only meaningful after the inital call to [startEngine](#startEngine) |

Callback fires as the plugin makes progress synchronizing with the blockchain network.

### onBalanceChanged

```javascript
onBalanceChanged(currencyCode, balance)
```
| Param | Type | Description |
| --- | --- | --- |
| currencyCode | `string` | The three-letter code for which currency balance has changed. |
| balance | `string` | The new spendable balance, in the smallest units the currency supports as a string (i.e. Satoshi or Wei). |

Callback fires when the plugin detects a balance change for the supported currency.

### onBlockHeightChanged

```javascript
onBlockHeightChanged(blockHeight)
```

| Param | Type | Description |
| --- | --- | --- |
| blockHeight | `Int` | New block height value |

Callback fires when the plugin detects a blockheight change for the supported currency.

### onTransactionsChanged

```javascript
onTransactionsChanged(abcTransactions)
```

| Param | Type | Description |
| --- | --- | --- |
| abcTransactions | `Array` | Array of [AbcTransaction](#abctransaction) objects which are new or have changed. Changes may include the block height when this transaction was confirmed |

Callback fires when the plugin detects new or updated transactions from the blockchain network. The plugin must fill in the following [AbcTransaction](#abctransaction) fields:

* `txid`
* `date`
* `networkFee`
* `blockHeight` (may be 0)
* `nativeAmount`
* `ourReceiveAddresses`

## AbcExchangePlugin

```javascript
// SDK users should do this:
import { coinbasePlugin } from 'airbitz-currency-ethereum'

const context = makeContext({
  plugins: [coinbasePlugin]
})

// Inside the `makeContext` function, this is what happens:
const exchangePlugin = await coinbasePlugin.makePlugin({ io })
```

These plugins fetch price information from various exchange-rate providers. They begin life as a generic [AbcCorePlugin](#abccoreplugin) that returns an `AbcExchangePlugin` instance from is `createPlugin` method.

The `AbcExchangePlugin` object provides information about the exchange, as well as the ability to fetch rates. Exchange plugins should not spin up background tasks, but should make a fresh server connection on every `fetchExchangeRates` call.

### exchangeInfo

This property should contain information about the exchange.

```javascript
console.log(exchangePlugin.info.exchangeName) // "Coinbase"
```

| Property | Type | Description |
| --- | --- | --- |
| exchangeName | `string` | Name of the exchange as a human-friendly string |

### fetchExchangeRates

```javascript
fetchExchangeRates(pairHints)

// Example
const pairHints = [
  { fromCurrency: 'BTC', toCurrency: 'iso:EUR' },
  { fromCurrency: 'ETH', toCurrency: 'iso:USD' }
]

const pairs = await exchangePlugin.fetchExchangeRates(pairHints)

// The returned pairs might look like this:
[
  { fromCurrency: 'BTC', toCurrency: 'iso:EUR', rate: 2997.75 },
  { fromCurrency: 'ETH', toCurrency: 'iso:USD', rate: 297.12 }
]
```

Asks the plugin to fetch all available currency pairs.

The passed-in currency pairs are a hint. Normal plugins should just return all available data, but in cases where that's too expensive (such as needing a separate HTTP request per pair), the plugin can use the passed-in list as a filter.

# Edge Login UI

To ease the implementation of the Edge SDK in applications, Edge provides a base user interface capable of all the account creation, account login, password & PIN management, and password recovery.

The repo [airbitz-core-js-ui](https://github.com/Airbitz/airbitz-core-js-ui) implements a UI layer on top of [airbitz-core-js](https://github.com/Airbitz/airbitz-core-js) to provide web and React Native applications the interface required to do all the accounts management in just a small handful of Javascript API calls. For HTML apps, the UI operates in an overlay iframe on top of the current HTML view.

Install instructions are in [airbitz-core-js-ui](https://github.com/Airbitz/airbitz-core-js-ui).

## Sample webpage

A sample webpage exists in [airbitz-core-js-sample](https://github.com/Airbitz/airbitz-core-js-sample) which makes a simple page with a Login & Register button on the top bar and uses [airbitz-core-js-ui](https://github.com/Airbitz/airbitz-core-js-ui).

## Usage for web/HTML

### makeABCUIContext

```javascript
abcUiContext = abcui.makeABCUIContext({'apiKey': 'api-key-here',
                                       'accountType': 'account:repo:com.mydomain.myapp',
                                       'bundlePath': '/path-to-abcui/',
                                       'vendorName': 'My Awesome Project',
                                       'vendorImageUrl': 'https://mydomain.com/mylogo.png'})
```

Initializes the ABCUI library and returns an [ABCUIContext](#abcuicontext) object

| Param | Type | Description |
| --- | --- | --- |
| apiKey | `String` | API Key from <https://developer.airbitz.co> |
| accountType | `String` | App account type of form 'account:repo:com.mydomain.myapp' |
| bundlePath | `String` | Website path to the airbitz-core-js-ui repo |
| vendorName | `String` | Name of your app |
| vendorImageUrl | `String` | URL to image of your app's logo |

| Return Param | Type | Description |
| --- | --- | --- |
| abcUiContext | [`ABCUIContext`](#abcuicontext) | Edge account object |

## ABCUIContext

### openLoginWindow

```javascript
abcUiContext.openLoginWindow(function(error, account) {
  _account = account;
})
```

Create an overlay popup where a user can register a new account or login to a previously created account via password or PIN.

| Param | Type | Description |
| --- | --- | --- |
| callback | `Callback` | (Javascript) Callback function |

| Return Param | Type | Description |
| --- | --- | --- |
| error | [`AbcError`](#abcerror) | (Javascript) Error object. `null` if no error |
| account | [`AbcAccount`](#abcaccount) | Edge account object |

![Login UI](#https://airbitz.co/go/wp-content/uploads/2016/08/Screen-Shot-2016-08-26-at-12.50.04-PM.png)

### openManageWindow

```javascript
abcUiContext.openManageWindow(_account, function(error) {
  // your_callback_here
})
```

Launch an account management window for changing password, PIN, and recovery questions

| Param | Type | Description |
| --- | --- | --- |
| account | [`AbcAccount`](#abcaccount) | Edge account object to modify |

![Manage UI](#https://airbitz.co/go/wp-content/uploads/2016/08/Screen-Shot-2016-08-26-at-12.50.26-PM.png)

