# Mongoose OS WiFi Setup

[![Gitter](https://badges.gitter.im/cesanta/mongoose-os.svg)](https://gitter.im/cesanta/mongoose-os?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge)

- [Mongoose OS WiFi Setup](#mongoose-os-wifi-setup)
  - [Captive Portal Stack](#captive-portal-stack)
  - [Author](#author)
  - [Features](#features)
  - [Settings](#settings)
      - [`cportal.setup.copy` Setting](#cportalsetupcopy-setting)
      - [`cportal.setup.disable` Setting](#cportalsetupdisable-setting)
      - [`cportal.redirect_file` Setting](#cportalredirect_file-setting)
  - [Installation/Usage](#installationusage)
    - [Use specific branch of library](#use-specific-branch-of-library)
  - [Required Libraries](#required-libraries)
  - [How it works](#how-it-works)
  - [Available Functions/Methods](#available-functionsmethods)
    - [C Functions](#c-functions)
    - [Usage in mJS](#usage-in-mjs)
  - [Changelog](#changelog)
  - [License](#license)

This library is for testing, saving, and setting up Mongoose OS device's WiFi.  The main feature of this library is the ability to test WiFi credentials, and then save or update them in the configuration.

## Captive Portal Stack

This is the **WiFi Setup** library from the [Captive Portal WiFi Full Stack](https://github.com/tripflex/captive-portal-wifi-stack), a full stack (frontend web ui & backend handling) library for implementing a full Captive Portal WiFi with Mongoose OS

## Author
Myles McNamara ( https://smyl.es )

## Features
- Test WiFi SSID and Passwords
- Automatically save SSID and Password after validating SSID and Password
- Automatically disable AP after successful WiFi test
- Automatically reboot the device after saving and successful test
- Set which STA index to save configuration to `wifi.sta` `wifi.sta1` or `wifi.sta2`
- Set custom timeout for testing connection and credentials (`30` seconds by default `cportal.setup.timeout`)
- Mongoose OS successful, failed, and started test events
- Callback for failed/successful test for use in `C` or `mJS`
- Disable Captive Portal setting after successful test

## Settings
Check the `mos.yml` file for latest settings, all settings listed below are defaults

```yaml
  - [ "cportal.setup.copy", "i", 0, {title: "Copy SSID and Password to this STA ID after succesful test ( 0 - wifi.sta | 1 - wifi.sta1 | 2 - wifi.sta2 )"}]
  - [ "cportal.setup.timeout", "i", 30, {title: "Timeout, in seconds, before considering a WiFi connection test as failed"}]
  - [ "cportal.setup.disable", "i", 2, {title: "Action to perform after successful test and copy/save values -- 0 - do nothing, 1 - Disable AP (wifi.ap.enable), 2 - Disable AP and Captive Portal (cportal.enable)"}]
  - [ "cportal.setup.reboot", "i", 0, {title: "0 to disable, or value (in seconds) to wait and then reboot device, after successful test (and copy/save values)"}]

```
#### `cportal.setup.copy` Setting
Set this equal to the STA ID you want to save the values to (if you want to save them after successful test).  Use `-1` to disable saving after successful test.

#### `cportal.setup.disable` Setting
Action to perform after successful test and copy/save values (if enabled)
`0` - Do Nothing
`1` - Disable AP `wifi.ap.enable`
`2` - Disable AP `wifi.ap.enable` and Captive Portal `cportal.enable`

#### `cportal.redirect_file` Setting
This setting if for if you want to use your own custom HTML file as the "redirect" file sent that includes a `meta` refresh tag.  This setting is optional, and when not defined, a dynamically generated response will be sent, that looks similar to this:

## Installation/Usage
Add this lib your `mos.yml` file under `libs:`

```yaml
  - origin: https://github.com/tripflex/captive-portal-wifi-setup
```

### Use specific branch of library
To use a specific branch of this library (as example, `dev`), you need to specify the `version` below the library

```yaml
  - origin: https://github.com/tripflex/captive-portal-wifi-setup
   version: dev
```

## Required Libraries
*These libraries are already defined as dependencies of this library, and is just here for reference (you're probably already using these anyways)*
- [wifi](https://github.com/mongoose-os-libs/wifi)
  
## How it works
This library adds `C` functions you can use to test wifi credentials (see below).  For a complete solution, use the Captive Portal WiFi Stack which adds easy methods for calling this (web ui, etc).

## Available Functions/Methods

### C Functions
```C
bool mgos_captive_portal_wifi_setup_test(char *ssid, char *pass, wifi_setup_test_cb_t cb, void *userdata );
```

The `mgos_captive_portal_wifi_setup_test` will return `true` if the test was started, `false` if it was not

The `wifi_setup_test_cb_t` is for a callback (optional) after a successful/failed wifi test.
```C
typedef void (*wifi_setup_test_cb_t)(bool result, char *ssid, char* password, void *userdata);
```

It will return the result `false` for failed `true` for success, the tested SSID, Password, and any userdata if you passed it when originally calling the function.

### Usage in mJS
To keep library size to a minimum, no mjs file is included with this library, but you can easily call it using the built in **ffi** for mjs, like this:
```javascript
let testWiFi = ffi('bool mgos_captive_portal_wifi_setup_test(char*,char*,void(*)(bool,char*,char*,userdata),userdata)')
testWiFi( "My SSID", "somePassword", function( success, ssid, pass, userdata){
  print( 'Test Completed!');
}, NULL );
```

## Changelog

**1.0.0** TBD - Initial release

## License
Apache 2.0