# Edge Wallet Plugin API

Edge Wallet plugins are a way to include additional functionality into Edge Wallet on iOS and Android. They are currently used to buy and sell bitcoin through Glidera & Bity.com, and purchase discounted Starbucks and Target cards through Foldapp.

The Wallet Plugin API provides a subset of the entire Edge SDK which allows the plugin to access the currently logged in user's wallet to request sends and receives.

These pages serve as an introduction on writing your own plugins. You only need basic web development skills to build your own plugin. If you can write HTML, CSS and Javascript, you can easily write a plugin and run it on Android and iOS.

## How it works

Edge plugins are just single page HTML files, with all the resources "compiled" in. That means that the javascript libraries, stylesheets and whatever else are all included in one monolithic HTML file.

The code is loaded by the Edge Wallet into a sandboxed webview. A bridge is created between the webview and the native-app via the [`edge-libplugin`][libplugin] package.  Using the [`edge-libplugin`][libplugin] package, plugins can easily:

- Read and write encrypted data
- Request Sends
- Generate receive addresses for coins and tokens

## Creating a plugin

```javascript
git clone https://github.com/teneighty/edge-plugin-skeleton
cd edge-plugin-skeleton
npm install
```

To get started you can clone the skeleton project. This already includes [`edge-libplugin`][libplugin] as a dependency. If you look in the `package.json` you'll also find the command `edge-ify`. This command is resonsible for taking your code and creating the monolithic HTML file which the native app will load.

## Load plugin into Edge

First you'll have to have the Edge Wallet building locally. Please follow the instructions in the Edge Wallet [README][edge-readme].

Once you are able to build Edge Wallet, you can make your plugin a dependency and add it to the `plugins` array at the bottom of the `package.json`. See [here][package-plugin-list]. 

At this point you can build Edge Wallet, and use your plugin. If you need to debug your plugin, you can use Safari to debug iOS and [Chrome][chrome-debugging] to debug on Android.

## Submit Your Plugin

To submit your plugin for inclusion in Edge Wallet, submit a pull request for the changes to the `package.json` for `edge-react-gui` repo. Now just wait for the Edge team to accept your PR.

# Edge Plugin API Reference

## PluginWallet object

```javascript
{
    "id": "wallet-id",
    "name": "My Wallet",
    "type": "wallet:bitcoin",
    "currencyCode": "ETH",
    "primaryNativeBalance": "ETH",
}
```

The `PluginWallet` object represents a wallet from the user's Edge account.
Users can switch which wallet they currently use for requests and sends.
Developers can fetch current `PluginWallet` object by using `selectedWallet`.

| Property     | Type     | Description                 | 
| ---          | ---      | ---                         | 
| id           | `String` | Wallet ID                   | 
| name         | `String` | Wallet text name            | 
| type         | `String` |                             | 
| currencyCode | `Number` | Wallet currency code number | 

## Core

### wallets

```javascript
core.wallets()

// Example
core.wallets().then(wallets => {
})
```

Returns a promise with a list of the wallets for this account. The success type is an array of `PluginWallet` objects.

### chooseWallet

```javascript
core.chooseWallet()

// Example
core.chooseWallet().then(wallet => {
})
```

Launches UI modal to select a wallet

### selectedWallet

```javascript
core.selectedWallet()

// Example
core.selectedWallet().then(wallet => {
})
```

Returns a promise with the user's currently selected wallet 

### getAddress

```javascript
core.getAddress(walletId, currencyCode)

// Example
core.getAddress('wallet-id', 'BTC').then(publicAddress => {
})
```

Returns a promise with a public address for the specified wallet

> Response


### requestSpend

```javascript
core.requestSpend(wallet, address, amount, options)

// Example
// Send 1.23 BTC to address 12xZEQL72YnGEbtW7bA4FPA1BUEHkxQoWN
core.requestSpend(wallet, "12xZEQL72YnGEbtW7bA4FPA1BUEHkxQoWN", 123000000, {
    label: "Roger Mark",
    category: "Income:Consulting",
    notes: "Web development project from 2016-04",
  })
  .then(response => {
    if (response.back) {
      console.log("User pressed back button. Funds not sent")
    } else {
      console.log("Bitcoin sent")
    }
  })
  .catch(error => {
    console.log("Error sending funds")
  })
```

Request that the user spends. This takes the user to the native spend confirmation screen so they can confirm the spend.

The optional metadata given in 'options' such as 'label', 'category', and 'notes' will also be written for this transaction.

| Param    | Type        | Description                                            | 
| ---      | ---         | ---                                                    | 
| wallet   | `ABCWallet` | Wallet to create a receive request/address from        | 
| address  | `String`    | Bitcoin address or BIP21 URI                           | 
| amount   | `Int`       | Amount to send in satoshis                             | 
| options  | `Object`    | JS Object of options for receive request               | 

| Options  | Type        | Description                                            | 
| ---      | ---         | ---                                                    | 
| label    | `String`    | (Optional) Transaction 'Payee/Payer' name              | 
| category | `String`    | (Optional) Transaction category such as "Expense:Rent" | 
| notes    | `String`    | (Optional) Transaction misc notes field                | 
| success  | `Function`  | Success callback                                       | 
| error    | `Function`  | Error callback                                         | 

### requestSign

```javascript
core.requestSign(wallet, address, amount, options)

// Example
// Send 1.23 BTC to address 12xZEQL72YnGEbtW7bA4FPA1BUEHkxQoWN
let signedTx = await core.requestSign(wallet, "12xZEQL72YnGEbtW7bA4FPA1BUEHkxQoWN", 123000000, {
    label: "Roger Mark",
    category: "Income:Consulting",
    notes: "Web development project from 2016-04",
  })
```

Request that the user signs a transaactions. This takes the user to the native spend confirmation screen so they can confirm.

The optional metadata given in 'options' such as 'label', 'category', and 'notes' will also be written for this transaction.

| Param    | Type        | Description                                            | 
| ---      | ---         | ---                                                    | 
| wallet   | `ABCWallet` | Wallet to create a receive request/address from        | 
| address  | `String`    | Bitcoin address or BIP21 URI                           | 
| amount   | `Int`       | Amount to send in satoshis                             | 
| options  | `Object`    | JS Object of options for receive request               | 

| Options  | Type        | Description                                            | 
| ---      | ---         | ---                                                    | 
| label    | `String`    | (Optional) Transaction 'Payee/Payer' name              | 
| category | `String`    | (Optional) Transaction category such as "Expense:Rent" | 
| notes    | `String`    | (Optional) Transaction misc notes field                | 
| success  | `Function`  | Success callback                                       | 
| error    | `Function`  | Error callback                                         | 


### broadcastTx

```javascript
core.broadcastTx(wallet, signedTx)
```

Broadcast the transaction.

### saveTx

```javascript
core.saveTx(wallet, signedTx)
```

Save the transaction in the wallets history. This is useful if the transaction is being broadcasted by a third party.

### requestFile

```javascript
core.requestFile()

// Example
let base64file = core.requestFile()
```

Launches the native OS's camera or file browser so the user can select a file. The file is returned as base64encoded string.

### bitidAddress

```javascript
core.bitidAddress(uri, message)

// Example
core.bitidAddress('bitid://airbitz.co/developer?nonce=12345', 'my message').then(address => {
  console.log(address)
})
```

Returns a bitid address for the given uri and message

### bitidSignature

```javascript
core.bitidSignature(uri, message)

// Example
core.bitidAddress('bitid://airbitz.co/developer?nonce=12345', 'my message').then(signature => {
  console.log(signature)
})
```

Returns a bitid signature for the given uri and message


### writeData

```javascript
core.writeData(key, data)

// Example
let data = JSON.stringify({
  "name": "Joe",
  "phone": "619-800-1234",
  "age:" 69 
})
core.writeData("userInfo", data)
  .then(() => {
    console.log('success!')
  })
  .catch(error => {
    console.log(error)
  })
```

Securely persist data into the Edge core under this user's account. Only the current plugin will have access to that data. Data is fully encrypted and synchronized between devices that the user logs into.

| Param | Type     | Description                      | 
| ---   | ---      | ---                              | 
| key   | `String` | Key string to reference the data | 
| data  | `String` | Value to store                   | 

### readData

```javascript
// Example
let data = JSON.stringify({
  "name": "Joe",
  "phone": "619-800-1234",
  "age:" 69 
})
let res = await core.writeData("userInfo", data)
data = await core.readData("userInfo")
console.log(data)
```

Read back data from the Edge core under this user's account. Only the current plugin will have access to that data. Data is fully encrypted and synchronized between devices that the user logs into.

| Param    | Type     | Description                                   | 
| ---      | ---      | ---                                           | 
| key      | `String` | Key string to reference the data              | 

| Response | Type     | Description                                   | 
| ---      | ---      | ---                                           | 
| data     | `String` | Data stored from a previous call to writeData | 

### clearData

```javascript
core.clearData()
```

Clear all data in the Edge core, for the current plugin.

### debugLevel

```javascript
core.debugLevel (level, text)

// Example
core.debugLevel (0, "Something happened, I don't want to talk about it")
```

Log messages to the Edge core at a particular level.

| Param | Type      | Description                                 | 
| ---   | ---       | ---                                         | 
| level | `integer` | ERROR = 0, WARNING = 1, INFO = 2, DEBUG = 3 | 
| text  | `String`  | Message to log                              | 


## Config

### get

```javascript
config.get(key)

// Example
let value = await config.get("API_KEY")
```

Fetch a configuration value. These are set in the native iOS/Android code, before the webview is loaded. Useful to pass in parameters such as API keys embedded in the native app.

| Param    | Type     | Description                      | 
| ---      | ---      | ---                              | 
| key      | `String` | Key string to reference the data | 

| Response | Type     | Description                      | 
| ---      | ---      | ---                              | 
| value    | `String` | Data passed in from native app   | 

## UI

### title

```javascript
ui.title(title)

// Example
ui.title("Bob's Awesome Plugin");
```

Set the title of the current view. This updates the native apps titlebar.

| Param | Type     | Description     | 
| ---   | ---      | ---             | 
| title | `String` | Title on navbar | 

### showAlert

```javascript
ui.showAlert(title, message)

// Example
ui.showAlert("Account creation error", "Error creating account. Please try again later");

ui.showAlert("Creating account", "Please wait...");
```

Launches a native alert dialog. Alert will automatically fade when tapped or after ~6 seconds unless the showSpinner option is given.

| Param       | Type      | Description                                                                    | 
| ---         | ---       | ---                                                                            | 
| title       | `String`  | Title of alert                                                                 | 
| message     | `String`  | Body of alert                                                                  | 

### hideAlert

```javascript
ui.hideAlert()
```

Hides any currently displaying alert from showAlert

[libplugin]: https://gitub.com/teneighty/edge-libplugin
[edge-readme]: https://github.com/Airbitz/edge-react-gui
[package-plugin-list]: https://github.com/Airbitz/edge-react-gui/blob/webview-plugins/package.json#L189-L196
[chrome-debugging]: https://developers.google.com/web/tools/chrome-devtools/remote-debugging/webviews
