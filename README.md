# GarminHomeAssistant

<img src="images/Actual_Venu2_Theme.jpg" width="200" title="Venu 2"/>

A Garmin application to provide a "dashboard" to control your devices via [Home Assistant](https://www.home-assistant.io/). The application will never be as fully fledged as a Home Assistant dashboard, so it is designed to be good enough for the simple and essential things. Those things that can be activated via an on/off toggle or a tap. That should cover lights, switches, and anything requiring a single press such as an automation. For anything more complicated, e.g. thermostat, it would always be quicker and simpler to reach for your phone or tablet... or the device's own remote control!

The application is designed around a simple scrollable menu where menu items have been extended to interface with the [Home Assistant API](https://developers.home-assistant.io/docs/api/rest/), e.g. to get the status of switches or lights for display on the toggle menu item. It is possible to nest menus, so there is a menu item to open a sub-menu. This can be arbitrarily deep and nested in the format of a tree of items, although you need to consider if reaching for your phone becomes quicker to select the device what you want to control.

The intended audience for this application are those comfortable with configuring a Home Assistant (e.g. editing the YAML configuration files) and debugging why URLs don't work. It does not require programming skills, but the menu is configured via JSON which feels like "coding". If you are not comfortable with this relatively low level of configuration, you may like to try other Garmin applications instead.

It is important to note that your Home Assistant instance will need to be accessible via HTTPS with public SSL or all requests from the Garmin will not work. This cannot be a self-signed certificate, it must be a public certificate. You can get one for free from [Let's Encrypt](https://letsencrypt.org/) or you can pay for [Home Assistant cloud](https://www.nabucasa.com/).

**If you are struggling with getting the application to work, please consult the [trouble shooting](Troubleshooting.md#menu-configuration-url) guide first.**


## Widget or Application?

As of version 2.0, there are now two installable versions. For older devices before applications supported 'glances', there is a now widget version. These two version must be downloaded separately due to the way the Connect IQ App Store requires them to have separate application IDs. Therefore you need to choose which you want up front. Here how they compare.

| Version                | Explanation |
|------------------------|-------------|
| Application (original) | For newer devices that allow glance views in their applications (e.g. Venu2), the GarminHomeAssistant application can be started either from a glance (with only the application name presently, no status) or from the list of applications and activities. Head over to the [GarminHomeAssistant](https://apps.garmin.com/en-US/apps/61c91d28-ec5e-438d-9f83-39e9f45b199d) application page on the [Connect IQ application store](https://apps.garmin.com/en-US/) to download the application. The application can be started two different ways, either from the glance in the carousel, or as an application from the list of applications & activities. With the latter, it is worth marking the application as a favourite.<hr><br/><img src="images/Venu2_app_start.png" width="200" title="Venu 2" style="margin:5px"/><img src="images/Vivoactive3_app_start.jpg" width="200" title="Venu 2" style="margin:5px"/><br/>If you place the application on your list of favourites, and rearrange it to appear near the top, then the item is just one button press away from the watch face. This second picture here shows the application menu on a Vivoactive 3 watch.<br/><img src="images/Venu2_glance_start.png" width="200" title="Venu 2" style="margin:5px"/><br/>On newer watches, you can also start the application from the glance carousel. The glance view here typically displays some trackable status, so ours provides some early indication of availability. Older watches will still allow you to start this application from the list of applications and activities. |
| Widget                 | For older devices that use widgets (e.g. Venu1) as opposed to applications with "glances", the GarminHomeAssistant application can instead be started from the widget carousel. This is a separate item in the Connect IQ AppStore and with this installation, the application will no longer appear in the list of applications and activities. Head over to the [GarminHomeAssistant](https://apps.garmin.com/en-US/apps/) widget page on the [Connect IQ application store](https://apps.garmin.com/en-US/) to download the widget.<hr><br/><img src="images/Venu_Widget_sim.png" width="200" title="Venu 2" style="margin:5px"/><br/>Typically the widget view implements something similar to the glance view, e.g. status, and exists in a widget carousel to allow you to select an application to launch. |

As the source code base for both is the same, the version numbers of each will be kept in step. that means that the widget version starts version numbering at 2.0 too.

## Dashboard Definition

Setup for this menu is more complicated than the Connect IQ settings menu really allows you to specify. In order to make the dashboard easily configurable and easy to change, we have provided an external mechanism for specifying the menu layout, a JSON file you write, retrieved from a URL you specify. JSON was chosen over YAML because Garmin can parse JSON HTTP GET responses into its own internal dictionary, it cannot parse YAML, hence a choice of one really. Note that JSON and YAML are essentially a 1:1 format mapping except JSON does not have comments. We recommend you take advantage of [Home Assistant's own web server](https://www.home-assistant.io/integrations/http/#hosting-files) to provide the JSON definition. The advantage here are:

1. the file is as public as you make your Home Assistant,
2. the file is editable within Home Assistant via "Studio Code Server", and
3. the schema is verifiable using [JSON Schema](https://json-schema.org/overview/what-is-jsonschema).

We have used `/config/www/garmin/<something>.json` on our Home Assistant's file system. That equates to a URL of `https://homeassistant.local/local/garmin/<something>.json`.

Schema verification is a big part of this design choice. If the application cannot read your menu definition, there's a limited amount of debug it can reasonable provide on a small screen. That responsibility now falls to you and the schema checker for help.

Example schema as shown in the images:

```json
{
  "$schema": "https://raw.githubusercontent.com/house-of-abbey/GarminHomeAssistant/main/config.schema.json",
  "title": "Home",
  "items": [
    {
      "entity": "script.food_on_table",
      "name": "Food is Ready!",
      "type": "tap",
      "tap_action": {
        "service": "script.turn_on",
        "confirm": true
      }
    },
    {
      "entity": "light.bedside_light_switch",
      "name": "Bedroom Light",
      "type": "toggle"
    },
    {
      "entity": "light.living_room_lights_all",
      "name": "Lounge Lights",
      "type": "toggle"
    },
    {
      "entity": "menu.each_lounge_light",
      "name": "Each Lounge Light",
      "title": "Lounge",
      "type": "group",
      "items": [
        {
          "entity": "light.standard_lamp",
          "name": "Standard Lamp",
          "type": "toggle"
        },
        {
          "entity": "light.bookcase_light",
          "name": "Bookcase Lamp",
          "type": "toggle"
        },
        {
          "entity": "light.corner_table_light",
          "name": "Corner Table Lamp",
          "type": "toggle"
        }
      ]
    },
    {
      "entity": "switch.bc_usbs",
      "name": "Bookcase USBs",
      "type": "toggle"
    },
    {
      "entity": "automation.garage_door_check",
      "name": "Garage Door Check",
      "type": "toggle"
    },
    {
      "entity": "automation.turn_off_usb_chargers",
      "name": "Turn off USBs",
      "type": "tap",
      "tap_action": {
        "service": "automation.trigger"
      }
    },
    {
      "entity": "scene.tv_light",
      "name": "TV Lights Scene",
      "type": "tap",
      "tap_action": {
        "service": "scene.turn_on"
      }
    }
  ]
}
```

NB. Entity names are not real in case anyone's a hacker.

The example above illustrates how to configure:

* Light or switch toggles
* Automation enable toggles
* Script invocation (tap)
* Service invocation, e.g. Scene setting, (tap)
* A sub-menu to open (tap)

The example JSON shows an example usage of each of these Home Assistance entity types. Presently, an automation is the only one that can be either a 'tap' or a 'toggle'.

| HA Type    | Tap | Toggle |
|------------|:---:|:------:|
| Switch     |  ❌ |   ✅  |
| Light      |  ❌ |   ✅  |
| Automation |  ✅ |   ✅  |
| Script     |  ✅ |   ❌  |
| Scene      |  ✅ |   ❌  |

NB. All 'tap' items must specify a 'service' tag.

Possible future extensions might include specifying the alternative texts to use instead of "On" and "Off", e.g. "Locked" and "Unlocked" (but wouldn't having locks operated from your watch be a security concern ;-))

The [schema](https://raw.githubusercontent.com/house-of-abbey/GarminHomeAssistant/main/config.schema.json) is checked by using a URL directly back to this GitHub source repository, so you do not need to install that file. You can just copy & paste your entity names from the YAML configuration files used to configure Home Assistant. With a submenu, there's a difference between "title" and "name". The "name" goes on the menu item, and the "title" at the head of the submenu. If your dashboard definition fails to meet the schema, the application will simply drop items with the wrong field names without warning.

### Old deprecated format

Version 1.5 brought in a change to the JSON schema so the follow old format remains useable but is no longer favoured. The schema now marks it as 'deprecated' to nudge people over.

```json
    {
      "entity": "scene.tv_light",
      "name": "TV Lights Scene",
      "type": "tap",
      "service": "scene.turn_on"
    }
```

The above should be replaced by the following:

```json
    {
      "entity": "scene.tv_light",
      "name": "TV Lights Scene",
      "type": "tap",
      "tap_action": {
        "service": "scene.turn_on"
      }
    }
```

This allows the `confirm` field to be accommodated in the `tap_action` along side the `service` tag, and follows the Home Assistant YAML format more closely.

## Editing the JSON file

You have options. The first is what we use.
1. **Best!** Use the [Studio Code Server](https://community.home-assistant.io/t/home-assistant-community-add-on-visual-studio-code/107863) addon for Home Assistant. You can then edit your JSON file in place.
2. Locally installed VSCode, or if not installed,
3. try the on-line version at https://vscode.dev/.

Paste in your JSON (and change the file type to JSON if not saving), it will then verify your file format and schema for you, highlighting any errors for you to fix.

A failure to get the file format right tends to mean that the response to the application errors with `INVALID_HTTP_BODY_IN_NETWORK_RESPONSE` (code of -400). This means the response did not contain JSON, it was probably an error message in plain text that could not be parsed by the Connect IQ API call. See [Toybox.Communications](https://developer.garmin.com/connect-iq/api-docs/Toybox/Communications.html) for the list of error code you might be present with on your device.

Make sure you can browse to the URL of your JSON file in a standard web browser to make sure it is accessible.

## API Key Creation

Having created your JSON definition for your dashboard, you need to create an API key for your personal account on Home Assistant. You will need a [Long-Lived Access Token](https://developers.home-assistant.io/docs/auth_api/#long-lived-access-token). This is not obvious and is bound to your own Home Assistant account.

Follow the menu sequence: `HA -> user profile -> Long-lived access tokens`. Make sure you save the generated token before dismissing it. You may like to perform this task on your phone so that you can copy and paste it (and message yourself a copy too ;-).

![Long-Lived Access Token](images/Long_Lived_Access_Tokens.png)

Having created that token, before you dismiss the dialogue box with the value you will never see again, copy it somewhere safe. You need to paste this into the Garmin Application's settings.

## API URL

If you are using Nabu Casa then your Cloud API URL can be found by looking up your URL via `HA -> Settings -> Home Assistant Cloud -> Remote Control -> Nabu Casa URL` and don't forget to add `/api` to the end of the copied string.

![Nabu Casa Remote Control](images/Nabu_Casa_Remote_Control.png)

If you have built your own infrastructure, you really don't need any assistance with the API URL!

## Settings

Unfortunately the Settings dialogue box in the Garmin IQ application times out in Android when you go to a different screen (browser for example). When you go back to the Connect IQ application (select the view again) the settings dialogue box is broken and you have to open the Settings again, so you will need to save the settings every time before you switch applications to avoid losing the information you just put in.

You can instead use an application like [Microsoft's "Phone Link"](https://apps.microsoft.com/detail/9NMPJ99VJBWV?hl=en-gb&gl=US) that allows you to copy and paste between your PC and your phone.

**Please, please, please!** Copy and paste your API key and all URLs, do not retype as it will be wrong.

<img src="images/GarminHomeAssistantSettings.png" width="400" title="Application Settings"/>

1. Copy and paste your API key you've just created into the top field.
2. Add the URL for your Home Assistant API, e.g. `https://<homeassistant>/api`. (No trailing slash `/`` character as one gets appended when creating the URL and you do not want two.)
3. Add the URL of your JSON file, e.g. `https://<homeassistant>/local/garmin/<something>.json`.

You should now have a working application on your watch and be able to operate your Home Assistant devices for as long as your watch is within Bluetooth range of your phone.

You may choose to cache your menu definition on your device in order to reduce the delay in showing the menu (as it saves waiting for an HTTP GET request). If you use this option you are responsible for managing the cache when the menu is updated at source. The Toggle below allows you to choose to refresh the cache the next time the application starts. Once the cache has been cleared, the application will reset this toggle for you, so you do not need to return to the settings to flip it.

The application timeout prevents the app running on your watch when you have forgotten to close it. It prevents the refreshing of the menu statuses and therefore excessive wear on your battery level. There is a second timeout value for confirmation views. This is intended for use with more sensitive toggles so that the confirmation view is not left open and forgotten and then confirmed accidentally without you noticing. **We cannot advise you this is safe, be careful what you toggle with the watch application!**

The menu style option selects between two menu presentations as follows:

| Menu Type       | Image                                                                  | Description |
|-----------------|------------------------------------------------------------------------|-------------|
| Icons (default) | <img src="images/Venu2_LeanUI.png" width="200" title="Venu 2"/> | "Lean User Interface" version removing the second row of text in favour of icons. Tap icons are blue with a pointing finger, menu items has three dots as favours by many graphical user interfaces. |
| Labels          | <img src="images/Venu2_Original.png" width="200" title="Venu 2"/>  | Initial version that had a second row of text. This extra text has yet to add much value. Menu and Tap actions are distinguished by text, and the toggle status is duplicated by text. A future version could possibly offer the means to customise the toggle menu item text, hence this option has not been deprecated yet. |

There is a toggle setting for "text alignment" and provides finer adjustment for right-to-left languages. Perhaps this could be made automatic based on device language?

Another toggle setting for the **Widget version only** allows the user to select a non-standard user interface behaviour. As soon as the menu is retrieved the widget view is replaced by the menu without waiting for a user selection. This has been included as requested by a user, but defaults to off which retains the expected user interactions.

Finally you may enable a background service to report the battery level to your Home Assistant. This is not available over Bluetooth like with other Bluetooth devices as Garmin did not implement it. This is more complicated than the rest of the application to [set up](#battery-level-reporting).

## Tap Item Response

Its obvious that a toggle menu item has been triggered as the visible switch changes position and colour. Less obvious is that you have successfully triggered a tap operation.

<img src="images/SimTapResponse.png" width="400" title="Tap Triggered"/>

The application will display a 'toast' showing Home Assistant's friendly name of the triggered item. The toast will disappear after a short while if not dismissed by the user.

## External Device Changes

Home Assistant will inevitably change the state of devices you are also controlling via your Garmin. The Garmin application does not maintain a web socket to listen for changes. Instead it must poll the Home Assistant API with your key. Therefore the application is not that responsive to changes. Instead there will be a delay of multiples of 100 ms per item whose status needs to be checked and amended.

The per toggle item delay is caused by a queue of responses to web requests filling up a queue and giving a [`Communications.BLE_QUEUE_FULL`](https://developer.garmin.com/connect-iq/api-docs/Toybox/Communications.html). response code.  For a Venu 2 Garmin watch an API call delay of 600 ms was found to be sustainable (500 ms was still too fast). The code now chains a sequence of updates, so as one finishes it invokes the next item's update. The more items requiring a status update that you pack into your dashboard, the slower each individual item will be updated!

The thinking here is that the watch application will only ever be open briefly not persistently, so the delay in picking up state changes won't be observed often for any race condition between two controllers.

As a consequence of this update mechanism, if you request changes too quickly you will be notified that your device cannot keep up with the rate of API responses and you will have to dismiss the error in order to continue. The is a _feature not a bug_!

## Changes to the (JSON) Dashboard Definition

When you change the JSON file defining your dashboard, you must exit the application and the reopen it. It only takes a matter of a few seconds to pick up the new definition, but it is not automatic.

## Submitting Corrections for Translations

Initially all text has been created in English, and a [Python script](https://github.com/house-of-abbey/GarminHomeAssistant/blob/main/translate.py) (Google Translate under the hood) has been used to create the first version of all translations. We have been pleased to accept better translations from native language speakers, **thank you**. If you would like to submit improved translations, our preference is you do so via a Git pull request. If you are not comfortable doing this, then just raise an issue and someone will eventually pick the request up.

In order to submit a language correction please create an XML file called `corrections.xml` in the same directory as your language containing the corrected text. The format of the XML file follows that of `strings.xml`. As an example here are some corrected French translations found in directory [`resources-fre/strings/corrections.xml`](https://github.com/house-of-abbey/GarminHomeAssistant/tree/main/resources-fre/strings/corrections.xml):

```xml
<strings>
  <string id="MenuItemOn">Activé</string>
  <string id="MenuItemTap">Clic</string>
  <string id="ApiFlood">Appels API trop rapide. Veuillez signaler cette erreur avec les détails de l'appareil.</string>
</strings>
```

The `id` attribute values are taken from the same names used in [`strings.xml`](https://github.com/house-of-abbey/GarminHomeAssistant/blob/main/resources-fre/strings/strings.xml). Not all `id` values need to be specified as missing `id`s will then use automatic translations. If the existing convention is followed then:

* The Python script will use the corrections in preference to translating, and
* Your pull request will be honoured without comment as we will take your corrections on trust.

## Battery Level Reporting

We've had [a report](https://github.com/house-of-abbey/GarminHomeAssistant/issues/39) that this feature does not work without **administrator priviledges**. We've reviewed possible fixes and come up short. We are unable to fix this at present but invite those skilled in the art of Home Assistant to suggest a solution to us!

The application and widget both now include a background service to report your watch's battery level and charging status. This requires [significant setup](BatteryReporting.md) via YAML in Home Assistant to work. This is not for the feint hearted! We are keen to received improvements, but are reluctant to provide much in the way of support. The Home Assistant community, in particular the posts on the forum at [Bluetooth Battery Levels (Android)](https://community.home-assistant.io/t/bluetooth-battery-levels-android/661525), are your best source of support for this feature. We do however offer this [trouble shooting](Troubleshooting.md#watch-battery-level-reporting) guide.


## Version History

| Version | Comment |
|:-------:|---------|
|   1.0   | Initial release for 26 devices. |
|   1.1   | Updated for 54 more devices, 80 in total. Scene support. Added vibrate acknowledgement for tap-based menu items. Falls back to a custom visual confirmation in the absence of 'toast' and vibrate support. Bug fix for large menus needing status updates. |
|   1.2   | Do not crash on zero items to update. Report unreachable URLs. Verify API URL does not have a trailing slash '/'. Increased HTTP response diagnosis. Reduced minimum API Level required from 3.3.0 to 3.1.0 to allow more device "part numbers" to be satisfied. |
|   1.3   | Tap for scripts was working in emulation but not on some phones. Decision is to make the 'service' field in the JSON compulsory for 'tap' menu items. This is a breaking change, but for many might be a fix for something not working correctly. Improve language support, we can now accept language corrections and prevent the automated translation of strings from clobbering manually refined entries. Thank you to two new contributors. |
|   1.4   | New lean user Interface with thanks to [Someone0nEarth](https://github.com/Someone0nEarth) for their contribution which is now the default. If you prefer the old style you can still select it in the settings. The provision of a 'service' tag is now not just heavily suggested by the JSON schema, it is enforced in code. With apologies to anyone suffering a breakage as a result. |
|   1.5   | <img src="images/confirmation_view.png" width="200" title="Confirmation View"/><br/>Added an optional confirmation dialogue view to prevent accidental execution of actions on mistaken tap. This also brings a change in the JSON schema to allow an optional field to specify that the confirmation should be used for a menu item. As we are now maturing and adding features we have decided to mitigate breaking changes to the JSON schema by being more careful to adopt the Home Assistant schema (noting there is a 1:1 mapping between YAML and JSON). This change does deprecate the top level `service` tag in favour of `tag_action` containing multiple fields including `service` & `confirm`. Users should migrate to the new format for the new functionality, but the timescale for actual deprecation are long and undecided. |
|   1.6   | Added a user configurable 'timeout' in seconds so that when no action is taken the application automatically closes, stopping the continuous polling for changes of status and hence saving the drain on the battery. This can be disabled with timeout=0. |
|   1.7   | Added timeout to confirmation views so that when used for security devices it does not linger when left unconfirmed. Thanks to [Jan Schneider](https://github.com/j-a-n) for the contribution. Known bug for devices not supporting [`WatchUi.getCurrentView()`](https://developer.garmin.com/connect-iq/api-docs/Toybox/WatchUi.html#getCurrentView-instance_function) API call which is only available on API Level 3.4.0, e.g. Vivoactive 4S. |
|   2.0   | A significant code base change to enable both a 'widget' version for older devices, e.g. Venu (1), and an application with a glance, e.g. Venu2. These two versions must now be distributed under separate application IDs, but they have the same code base. A further 20 more devices are now supported, the settings have been internationalised, and there's a bug fix for older devices when trying to display a helpful error message but instead the application crashed. This version has come from a significant collaboration with [Someone0nEarth](https://github.com/Someone0nEarth). |
|   2.1   | Deployment of an idea to provide Home Assistant with access to the watch battery level. Using this requires [significant setup](BatteryReporting.md) on the Home Assistant configuration and hence is detailed separately. Due to this, the default state for this battery option is _off_. Changed the application settings user interface to be more intuitive, and hence amended the way settings are managed in the background. |
|   2.2   | Adds a feature to cache the menu configuration and save the time taken for an HTTP request to fetch it. You as the user are responsible for managing the cache by clearing it when you update your configuration. Improvement to widget root display updates. Bug fix for battery level reporting when in the glance carousel. Fixed an uninternationalised string, "Execute". Unfixed issue with battery level updates when the user is not an administrator. |

## Known Issues

1. On some (old) devices (e.g. Vivoactive 3, Fenix 5s & Edge 520+), the menu does not update correctly to reflect changes in state effected by an external Home Assistant control. E.g. when the phone application changes the toggle status of a switch, the Garmin application does not reflect that change until the menu is touched or scrolled a little. This is a [known issue](https://forums.garmin.com/developer/connect-iq/i/bug-reports/menu2-doesn-t-allow-live-updates) already reported without a suggested software fix.
2. The [battery level reporting function](BatteryReporting.md) requires the user to have administrator priviledges. We are unable to fix this at present.
