# ESP32 Nordic OTA DFU Uploader

![Project Banner](docs/project-banner.png)

ESP32 firmware package uploader + BLE DFU runner for Nordic OTA-compatible targets.

This release repo contains binaries and usage docs only.

## Latest Releases

Download the latest firmware artifacts from:

- https://github.com/cra0/esp32-nordic-ota-dfu/releases

Use a flasher such as ESPTool
- https://github.com/espressif/esptool/releases

## Flash Instructions

You can use esptool to flash the merged.bin firmware (which contains everything) into the target device like so:

`esptool.exe --chip esp32 --port COM12 write-flash 0x0 esp32\blecent-esp32-merged.bin`

Where chip sets the target chipset, port is the connected port and the path at the end points to the bin.


## What This Does

1. Creates a local Wi-Fi AP so you can upload a Nordic DFU `.zip` package.
2. Stores the package on the ESP32.
3. Reboots into BLE update mode and automatically scans for DFU targets.
4. Performs DFU and keeps logs you can review later from the web UI.

## Step 0: It is important you have the OTAFix installed as the bootloader. [Please see here](https://github.com/oltaco/Adafruit_nRF52_Bootloader_OTAFIX)

## Step 1: Connect to ESP32 AP

On your phone, connect to:

- SSID: `drone-updater`
- Password: `meshcore123`

![Connect to AP](docs/esp32-ap-connect.jpg)

## Step 2: Open the Upload UI

Open this URL in your browser:

- `http://192.168.4.1/ui`

![Upload UI](docs/upload-ui.jpg)

## Package Upload Workflow

1. Tap **Choose file** and select your Nordic DFU `.zip`.
2. Tap **Upload Zip**.
3. Wait for upload + package validation to complete.
4. After a valid package is present:
   - **Upload Zip** is hidden.
   - **Delete Package** appears.
   - **Reboot to BLE** appears.

### Buttons in AP UI

- **Upload Zip**: uploads and validates the package.
- **Delete Package**: removes the stored package.
- **Reboot / Reboot to BLE**:
  - With valid package: reboots into BLE DFU mode.
  - Without package: normal reboot.
- **View Logs**: opens detailed run history.

## View Flash Logs Later

From `/ui`, tap **View Logs** (or open `/logs`) to inspect previous DFU runs.

You can:

- read the latest logs,
- download raw logs,
- clear logs.

![Logs Page](docs/view-logs.jpg)

## Hardware Button Controls

The left button on the ESP32-C6 board supports short/long press behavior:

- **Single press (short)**:
  - In BLE mode: toggles scan pause/resume (when idle).
  - In AP mode: ignored.
  - During active BLE transfer/connection: ignored.
- **Long press (~2.5s)**:
  - Toggles next boot mode (AP <-> BLE),
  - then reboots immediately.
  - Ignored while a DFU transfer is in progress.

The reset button is always a hardware reboot.

## Recommended Operator Flow

1. When the esp32 starts up for the first time it checks if it has a valid package, if not it starts in AP (Access Point) mode where there is a WI-FI access point for you to connect to (for now called `drone-updater` with `meshcore123` as the password).
2. You connect to that AP and go to the url `http://192.168.4.1/ui` and upload your zip package for the target meshcore node you want to flash.
3. You then can press reboot to bluetooth or powercycle the esp32 it will then start its scanning constantly. You can pause the scan at any time by pressing the program/boot button. Holding down that button for 5 seconds will force it back into AP mode.
4. You then yes login to the repeaters admin panel and go into the console and write `start ota` it will then give you the MAC address of the bluetooth advert.
5. You then fly the drone or bring the device close to the repeater (if you have an external antenna it can work up to 10 meters)
6. Typically the update takes 10 seconds but you can wait as long as you like. Right now there is no way to tell what happened *this is a problem for later for me to solve* 
7. If you then try to login again to the device on the meshcore app and it worked and the clock was reset you can tell it worked.
8. **To confirm it worked you type `ver` in the console**
9. Then when you get the drone back you can always put it back into AP mode by holding the program/boot button for 5secs then view the logs to see what happened.

## Notes

- AP credentials are fixed in this release:
  - SSID: `drone-updater`
  - Password: `meshcore123`
- BLE scanning and DFU are autonomous in BLE mode.
- If no valid package is present, device defaults to AP mode for upload.
