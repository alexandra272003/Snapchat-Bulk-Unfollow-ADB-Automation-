# Snapchat Bulk Unfollow (ADB Automation)

A Windows batch script that automates bulk-unfollowing accounts on Snapchat using `adb` (Android Debug Bridge) and coordinate-based UI taps — no root, no third-party app, no API access required.

Snapchat has no built-in "unfollow all" button and no public API for this action. This project simulates the exact manual steps a human would do (open the Following list → tap the ✕ next to a name → confirm removal) but automates the repetition.

---

## ⚠️ Disclaimer

- This automates actions on **your own account** using standard Android debugging tools. It is not illegal.
- It **does violate Snapchat's Terms of Service**, which prohibit automated/bot-like interaction with the app. Snapchat's systems may detect unusual tap patterns and could rate-limit, flag, or temporarily lock the account.
- Every account removed this way is **instant and irreversible** — there is no bulk "undo." Use with intention.
- This project is not affiliated with, endorsed by, or connected to Snap Inc. in any way.
- Use at your own risk.

---

## How it works

Snapchat's "Manage accounts I follow" screen doesn't expose individual buttons (like the ✕ icon) as distinct, addressable UI elements to accessibility tools — everything in each row is drawn as one flattened view. Because of this, standard automation frameworks (Appium, Selenium-style element-finding) can't reliably target the ✕ button by ID.

Instead, this script uses **fixed-coordinate tapping** via `adb shell input tap X Y`:

1. Tap the ✕ icon position (always the top visible row, since removed rows shift the list up)
2. Wait briefly for the "Remove?" confirmation popup
3. Tap "Yes" to confirm
4. Repeat
5. Every N removals, force-close and relaunch Snapchat, then re-navigate back to the Edit screen — this works around occasional in-app glitches/freezes

---

## Requirements

- A Windows PC
- An Android phone with Snapchat installed and logged in
- A USB cable (must support data transfer, not charge-only)
- [Android Platform Tools (adb)](https://developer.android.com/tools/releases/platform-tools) downloaded and extracted

---

## Setup

### 1. Enable Developer Options on your phone
- Settings → About phone → tap **Build number** 7 times

### 2. Enable USB Debugging
- Settings → Developer options → toggle **USB debugging** ON

### 3. Install adb on your PC
- Download platform-tools from the link above
- Extract the ZIP anywhere, e.g. `C:\platform-tools\`

### 4. Connect your phone
- Plug in via USB
- Tap **Allow** on the "Allow USB debugging?" popup on your phone

### 5. Verify the connection
Open Command Prompt inside the `platform-tools` folder and run:
```
adb devices
```
You should see your device listed as `device` (not `unauthorized` or empty).

### 6. Check your screen resolution
```
adb shell wm size
```
The coordinates in this script are calibrated for **1080x2400**. If your resolution differs, see [Calibrating coordinates](#calibrating-coordinates-for-your-phone) below.

---

## Usage

### 1. Turn on Airplane Mode
This prevents incoming snap notifications/banners from interrupting taps mid-run.

### 2. Manually navigate to the Edit screen
Open Snapchat → **Stories** tab → **3-dot menu** (top right) → **Manage accounts I follow** → **Edit** (top right)

You should now see the list with ✕ icons next to every account.

### 3. Run the script
Save `unfollow_all.bat` (below) into your `platform-tools` folder, then in Command Prompt:
```
unfollow_all.bat
```

### 4. Let it run
It will remove accounts in batches, restarting the app periodically to avoid glitches, until it hits the max count or runs out of accounts.

---

## The Script (`unfollow_all.bat`)

```bat
@echo off
setlocal enabledelayedexpansion

set /a total=0
set /a batchSize=15
set /a maxTotal=500

:restart_app
echo Restarting Snapchat to clear any glitches...
adb shell am force-stop com.snapchat.android
timeout /t 2 /nobreak >nul
adb shell monkey -p com.snapchat.android -c android.intent.category.LAUNCHER 1
timeout /t 6 /nobreak >nul

echo Navigating back to Edit screen...
adb shell input tap 735 2195
timeout /t 2 /nobreak >nul
adb shell input tap 1000 160
timeout /t 2 /nobreak >nul
adb shell input tap 270 1975
timeout /t 2 /nobreak >nul
adb shell input tap 1004 150
timeout /t 2 /nobreak >nul

echo Starting removal batch...
for /L %%i in (1,1,%batchSize%) do (
    adb shell input tap 985 357
    timeout /t 2 /nobreak >nul
    adb shell input tap 540 1320
    set /a total+=1
    echo Removed account number !total!
    timeout /t 4 /nobreak >nul
)

if !total! LSS %maxTotal% (
    echo Batch of %batchSize% done, restarting app fresh before next batch...
    timeout /t 5 /nobreak >nul
    goto restart_app
)

echo All done! Total removed: !total!
pause
```

---

## Calibrating coordinates for your phone

The tap coordinates were derived from a **1080x2400** display (Vivo V20). If your phone has a different resolution, screen density, or UI scale, you'll need to recalibrate:

1. Get your resolution: `adb shell wm size`
2. Take a screenshot while on the target screen:
   ```
   adb exec-out screencap -p > screen.png
   ```
3. Open `screen.png` in Paint (or any image viewer that shows pixel coordinates)
4. Hover over the element you want to tap (✕ icon, "Yes" button, Stories tab, 3-dot menu, "Manage accounts I follow" text, "Edit" text) and note the x/y pixel position
5. Replace the corresponding coordinates in the script

| Element | Approx. coordinate (1080x2400) |
|---|---|
| Stories tab (bottom nav) | `735, 2195` |
| 3-dot menu (top right) | `1000, 160` |
| "Manage accounts I follow" | `270, 1975` |
| "Edit" (top right) | `1004, 150` |
| ✕ icon (top row) | `985, 357` |
| "Yes" confirm button | `540, 1320` |

---

## Known limitations

- **Coordinate-based, not element-based** — if Snapchat updates its UI layout, these coordinates will break and need recalibration.
- **No confirmation replay** — once removed, the script cannot re-follow accounts. There is no built-in "dry run" mode.
- **Screen must stay on and unlocked** — if the phone locks mid-run, taps will land on the lock screen instead.
- **Notifications can still interfere** even with Airplane Mode if you have other apps generating local alerts — close background apps if issues persist.
- **This is UI automation, not an API integration** — it is inherently more fragile than a proper API call would be, because none exists for this action.

---

## License

Do whatever you want with this. No warranty, no liability, use at your own risk.
