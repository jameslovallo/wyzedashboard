# Wyze Desktop

![](src/assets/screenshot.png)

This is a desktop application for power users to control/view their Wyze bulbs, plugs and cameras from a Mac or PC. (For linux support, please download and build yourself.)

Because Wyze doesn't provide a developer API, this is only possible by integrating with IFTTT webhooks and [IPCamLive](https://www.ipcamlive.com). I am investigating ways to stream RTSP video without IPCamLive but RTSP is hard to display in web/electron applications. If you know anything about the subject and wish to help, please [contact me](https://patrickonparker.com).

**I cannot stress enough that this app is not intended for everyone: it is difficult to set up and is meant for power users who want a lighter alternative to BlueStacks on desktop, or who need to be able to view multiple cameras at once.** It is in-development and relies on several 3rd-party services and hacks in order to work. Use at your own risk. Also, know that I am a new dad and don't have a ton of time to provide support in the forums or on Reddit, please support each other and help answer each other's questions.

### [Download](https://github.com/patrickonparker/wyzedashboard/releases/tag/0.0.1)

All future releases will be found at the project's [releases](https://github.com/patrickonparker/wyzedashboard/releases) page.

---

## Getting Started

Download the app for your platform from the [releases page](https://github.com/patrickonparker/wyzedashboard/releases), extract the package, open the app and ignore whatever Windows/Mac security prompts usually pop up for downloaded apps.

### Adding bulbs & plugs

The app uses IFTTT to turn devices on or off, since Wyze does not provide a developer API. Sadly, there is no way to know if the device is already on or off and display its current status.

1. Go to [https://ifttt.com/maker_webhooks](https://ifttt.com/maker_webhooks) and click the "Documentation" button. It will tell you your unique webhook key at the top of the page. Copy this key, open the app, click the settings (gear) icon in the toolbar and past the key into the "IFTTT Webhook Key" field. You only have to do this once.
2. Now go back to IFTTT and create two new applets where 'THIS' = Webhook and 'THAT' = WYZE. The first applet will be to turn your device on, the second to turn it off. You will be asked to enter a webhook 'action' for each. Remember the name of the actions, you will need them in step 4.
3. Click the plus button at the bottom right corner and choose 'Add a Device'. Choose the 'Bulb' or 'Plug' tab at the top.
4. Give your device a name and enter your 'on' action and 'off' action in the fields below, then click 'Add Device.'
5. Test your device.

### Adding Cameras

This app embeds IPCamLive camera feeds, which must be set up through their service. On average I experience 5-12 seconds of latency, which I think is pretty tolerable. IPCamLive provides a valuable service and this app uses several hacks to embed their feeds, even using a free tier, including refreshing the feed every 4 minutes so that it doesn't time out. If you use their service with this app, please support them by paying for a subscription.

1. Get a dynamic DNS provider (I use [noip](https://www.noip.com) through my Netgear router) and set up a dynamic dns hostname. (You only have to do this once.)
2. Install the [RTSP firmware](https://support.wyzecam.com/hc/en-us/articles/360026245231-Wyze-Cam-RTSP) for your camera.
3. Get your device's RTSP url.
4. Forward port 554 to a unique port for your device in your router's settings. I have 3 cameras and use external ports 554, 555 and 556.
5. Go to [IPCamLive](https://www.ipcamlive.com), create an account and add your camera. You will need to enter your rtsp url in the following format:
   - rtsp://YOUR_RTSP_USERNAME:YOUR_RTSP_PASSWORD@YOUR_DYNAMIC_DNS_HOSTNAME:YOUR_CAMS_EXTERNAL_PORT/live
6. Go to your cam's 'Live' tab and copy the link.
7. In the app, click 'Add New Device', choose 'Camera' at the top, and enter your device's name and IPCamLive 'live' link.
8. You can optionally add an IFTTT webhook action to trigger a recording on the device.

## Known Issues

- Importing saved devices is crashing the Mac app. For now, if you use both the Mac and Windows versions of the app, please manage your devices from the Mac app, then export your devices and import them in the Windows app.

## Build from Source (i.e. for Linux)

### Install Dependencies

```bash
yarn
```

### Start the app in development mode (hot-code reloading, error reporting, etc.)

```bash
quasar dev
```

### Build the app for production

```bash
quasar build
```
