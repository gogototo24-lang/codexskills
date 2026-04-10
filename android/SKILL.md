---
name: Android Device Access
description: Access camera, microphone, calendar, location, clipboard, notifications, battery, contacts, SMS, TTS, Shizuku privileged shell, and run Java on the Android host via bsh and termux- commands
---

# Android Device Access

You run inside proot on Android. Three ways to interact with the device:
1. **termux-\* commands** — simple shell commands for common tasks
2. **bsh** — BeanShell interpreter for direct Android Java API access
3. **shizuku** — privileged shell commands via ADB/root identity

## termux-* Commands (simple device access)

| Command | Usage |
|---------|-------|
| `termux-camera-photo [-c 0\|1] <file.jpg>` | Take photo |
| `termux-clipboard-get` / `termux-clipboard-set "text"` | Clipboard |
| `termux-location` | GPS/network location (JSON) |
| `termux-device-info` | Model, manufacturer, SDK |
| `termux-notification -t "Title" -c "Content" [--id N]` | Show notification |
| `termux-calendar-list` | List calendar events |
| `termux-calendar-insert -t "Title" -d "Description" [-b start_ms] [-e end_ms] [-a "Location"]` | Insert calendar event |
| `termux-battery-status` | Battery level, status, temperature |
| `termux-vibrate [-d ms]` | Vibrate device (default 200ms) |
| `termux-torch [on\|off]` | Toggle flashlight |
| `termux-volume [level]` | Get/set media volume |
| `termux-wifi-connectioninfo` | WiFi SSID, IP, signal |
| `termux-tts-speak "text"` | Text-to-speech |
| `termux-toast "text"` | Show Android toast |
| `termux-share "text"` | Android share intent |

For contacts and SMS, use Shizuku: `shizuku content query --uri content://contacts/phones` or `shizuku service call isms ...`

## Shizuku — Privileged Shell (ADB/root)

Shizuku lets you run commands with ADB shell (UID 2000) or root identity via the `rish` binary. Requires the Shizuku app to be installed and running on the device.

```bash
shizuku ls /data/data
shizuku pm list packages
shizuku settings put global adb_enabled 1
shizuku dumpsys battery
shizuku am force-stop com.example.app
shizuku cmd package install-existing com.example.app
```

Shizuku runs commands as the shell user (same as `adb shell`), giving access to:
- Protected paths (`/data`, `/system`)
- Package management (`pm install/uninstall`)
- System settings (`settings put/get`)
- Service management (`dumpsys`, `cmd`)
- App control (`am force-stop`, `am start`)
- UI automation (`uiautomator dump`, `input tap/swipe`)
- Grant permissions: `shizuku appops set gptos.intelligence.assistant MANAGE_EXTERNAL_STORAGE allow`

### UI Automation via Shizuku

```bash
shizuku uiautomator dump /sdcard/ui.xml && cat /sdcard/ui.xml
shizuku input tap 500 800
shizuku input swipe 500 1500 500 500 300
shizuku input text "hello"
shizuku input keyevent KEYCODE_HOME
shizuku screencap -p /sdcard/screen.png
```

If Shizuku is not installed, the connector UI shows a dialog with install link.

## BeanShell (bsh) — Java on Android Host

```bash
bsh -c '<code>'          # Execute inline Java
bsh -e '<expression>'    # Evaluate and print result
bsh <file.bsh>           # Execute file
```

### Built-in Variables

`context`, `app`, `activity`, `handler`, `bridge`, `prootManager`, `pm`, `contentResolver`, `runtime`

### Callback Helpers

| Helper | Methods |
|--------|---------|
| `camera` | `info()`, `takePhoto(path)`, `takePhoto(path, cameraId)` |
| `audio` | `record(path)`, `record(path, durationSec)` |
| `location` | `getCurrentLocation()`, `getCurrentLocation(timeoutSec)` |
| `sensor` | `read(sensorType)` — 1=accel, 2=magnetic, 4=gyro |
| `clipboard` | `get()`, `set(text)` |

```bash
bsh -e 'camera.takePhoto(context.getCacheDir().getAbsolutePath() + "/photo.jpg")'
bsh -e 'camera.info()'
bsh -e 'audio.record(context.getCacheDir().getAbsolutePath() + "/rec.m4a", 5)'
bsh -e 'sensor.read(1)'
bsh -e 'clipboard.get()'
bsh -e 'Build.MODEL + " " + Build.MANUFACTURER'
```

### BeanShell Callbacks

BeanShell CAN implement Java interfaces — use simplified syntax without types:

```bash
bsh -c 'cb = new Runnable() { run() { print("callback!"); } }; cb.run();'
```

### Permissions

```bash
bsh -e 'androidx.core.content.ContextCompat.checkSelfPermission(context, "android.permission.CAMERA") == PackageManager.PERMISSION_GRANTED ? "granted" : "denied"'
# Grant via adb: adb shell pm grant gptos.intelligence.assistant android.permission.RECORD_AUDIO
```

### Notifications (bsh)

```bash
bsh -c 'NotificationManager nm = (NotificationManager)context.getSystemService("notification"); String ch = "bsh_ch"; nm.createNotificationChannel(new NotificationChannel(ch, "BSH", NotificationManager.IMPORTANCE_DEFAULT)); nm.notify(42, new Notification.Builder(context, ch).setSmallIcon(android.R.drawable.ic_dialog_info).setContentTitle("Title").setContentText("Body").build());'
```

### Device Info (bsh)

```bash
bsh -c 'BatteryManager bm = (BatteryManager)context.getSystemService("batterymanager"); print(bm.getIntProperty(4) + "%");'
bsh -e 'pm.getInstalledPackages(0).size() + " packages"'
```

Auto-imports: `android.os.*`, `android.content.*`, `android.provider.*`, `android.app.*`, `android.net.*`, `android.media.*`, `android.hardware.*`, `android.location.*`, `java.io.*`, `java.util.*`, `java.net.*`. Built-in `print()`, `runOnUi(Runnable)`.

## Communication (am intents)

```bash
am start -a android.intent.action.SENDTO -d "mailto:user@example.com" --es android.intent.extra.SUBJECT "Subj" --es android.intent.extra.TEXT "Body"
am start -a android.intent.action.SENDTO -d "smsto:+1234567890" --es sms_body "Msg"
am start -a android.intent.action.DIAL -d "tel:+1234567890"
am start -a android.intent.action.SEND -t "text/plain" --es android.intent.extra.TEXT "Content"
am start -a android.intent.action.VIEW -d "geo:0,0?q=Tokyo+Tower"
am start -a android.intent.action.VIEW -d "https://example.com"
am start -a android.settings.SETTINGS
am start -a android.intent.action.SET_ALARM --ei android.intent.extra.alarm.HOUR 8 --ei android.intent.extra.alarm.MINUTES 30
```

am extra types: `--es` String | `--ei` Int | `--el` Long | `--ez` Boolean | `--ef` Float

## Storage

`/root/` persists inside proot.

**Play Store version**: Cannot request full file access. Use Shizuku or download the APK version instead.

### APK version (full access)

Download APK: https://github.com/friuns2/openclaw-android-assistant/releases

The APK version has `MANAGE_EXTERNAL_STORAGE` permission. After enabling via Storage connector (Settings toggle), `/sdcard/` is directly accessible:

```bash
ls /sdcard/Download/
cp /sdcard/DCIM/Camera/photo.jpg /root/photo.jpg
cp /root/output.pdf /sdcard/Download/
find /sdcard/ -name "*.pdf" -maxdepth 2
```

### Shizuku Storage (Play Store or APK)

Install Shizuku: https://shizuku.rikka.app/download/

With Shizuku connected, full filesystem access is available regardless of version:

```bash
shizuku ls /sdcard/Download/
shizuku cat /sdcard/Documents/notes.txt
shizuku cp /sdcard/DCIM/Camera/photo.jpg /data/local/tmp/
shizuku find /sdcard/ -name "*.pdf" -maxdepth 2
shizuku mv /sdcard/old.txt /sdcard/new.txt
```

Copy files between proot and sdcard via Shizuku:
```bash
shizuku cat /sdcard/file.txt > /root/file.txt
cp /root/file.txt /tmp/file.txt
shizuku cp /data/user/0/gptos.intelligence.assistant/cache/file.txt /sdcard/Download/
```
if shizuku is not available, reply with link to Shizuku download: https://shizuku.rikka.app/download/

sdcard/ full storage access only possible with github version, google play limits storage access - APK download: https://friuns2.github.io/openclaw-android-assistant/