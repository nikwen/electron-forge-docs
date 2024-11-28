---
description: Create a Windows installer for your Electron app using Electron Forge.
---

# Squirrel.Windows

The Squirrel.Windows target builds your application using the [Squirrel.Windows](https://github.com/Squirrel/Squirrel.Windows) framework. It generates three files:

<table><thead><tr><th width="258">File</th><th>Description</th></tr></thead><tbody><tr><td><code>{appName} Setup.exe</code></td><td>The main executable installer for your application</td></tr><tr><td><code>{appName}-full.nupkg</code></td><td>The NuGet package file used for updates</td></tr><tr><td><code>RELEASES</code></td><td>Metadata file used to check if an update is available</td></tr></tbody></table>

Squirrel.Windows is a no-prompt, no-hassle, no-admin method of installing Windows applications, and is therefore the most user friendly you can get.

## Requirements

You can only build the Squirrel.Windows target on a Windows machine or on a Linux machine with [`mono`](https://www.mono-project.com/) and [`wine`](https://www.winehq.org/) installed.

## Installation

```bash
npm install --save-dev @electron-forge/maker-squirrel
```

## Usage

Add this module to the [makers](./) section of your [Forge configuration](../configuration.md):

{% code title="forge.config.js" %}
```javascript
module.exports = {
  makers: [
    {
      name: '@electron-forge/maker-squirrel',
      config: {
        certificateFile: './cert.pfx',
        certificatePassword: process.env.CERTIFICATE_PASSWORD
      }
    }
  ]
};
```
{% endcode %}

The Squirrel.Windows maker inherits all of its config options from the [`electron-winstaller`](https://github.com/electron/windows-installer) module, _except_ for `appDirectory` and `outputDirectory`, which are set by the maker.

Complete configuration options are documented in the [`MakerSquirrelConfig`](https://js.electronforge.io/modules/\_electron\_forge\_maker\_squirrel.html#MakerSquirrelConfig) types.

### Mandatory metadata

Squirrel.Windows requires mandatory package metadata to satisfy the [`.nuspec`](https://learn.microsoft.com/en-us/nuget/reference/nuspec) manifest format. There are two ways to specify this information in Electron Forge.

#### In package.json

By default, the Squirrel.Windows maker fetches the `author` and `description` fields in the project's package.json file.

{% code title="package.json" %}
```jsonc
{
  // ...
  "author": "Alice and Bob",
  "description": "An example Electron app"
  // ...
}
```
{% endcode %}

#### In your Forge config

Alternatively, you can also override these values directly in your Squirrel.Windows maker config.

{% code title="forge.config.js" %}
```javascript
module.exports = {
  makers: [
    {
      name: '@electron-forge/maker-squirrel',
      config: {
        authors: 'Alice and Bob',
        description: 'An example Electron app'
      }
    }
  ]
};
```
{% endcode %}

{% hint style="warning" %}
Note that the Forge config field is **"authors"** while the package.json field is called **"author".**
{% endhint %}

### Handling startup events

When first running your app, updating it, and uninstalling it, Squirrel.Windows will spawn your app an additional time with some special arguments. You can read more about these arguments on the [`electron-winstaller`](https://github.com/electron/windows-installer) README.

The easiest way to handle these arguments and stop your app launching multiple times during these events is to use the [`electron-squirrel-startup`](https://github.com/mongodb-js/electron-squirrel-startup) module as one of the first things your app does.

{% code title="main.js" %}
```javascript
const { app } = require('electron');

// run this as early in the main process as possible
if (require('electron-squirrel-startup')) app.quit();
```
{% endcode %}

### Spaces in the app name

Squirrel.Windows has bugs when certain identifiers contain spaces (e.g. [#713](https://github.com/Squirrel/Squirrel.Windows/issues/713)). The following is a configuration that works well even if your app name contains spaces:

{% code title="package.json" %}
```json5
{
  // Hyphenated version
  "name": "app-name",
  // The app name with spaces (will be shown to your users)
  "productName": "App Name",
  // ...
}
```
{% endcode %}

{% code title="forge.config.ts" %}
```typescript
const config: ForgeConfig = {
  makers: [
    new MakerSquirrel({
      // CamelCase version
      name: "AppName",
      // ...
    }),
  ],
  // ...
}
```
{% endcode %}

Alternatively, instead of using CamelCase in the `MakerSquirrel` config, you could also use the hyphenated version `app-name` (as [Electron Fiddle](https://github.com/electron/fiddle) does).

## Debugging

For advanced debug logging for this maker, add the `DEBUG=electron-windows-installer*` environment variable.
