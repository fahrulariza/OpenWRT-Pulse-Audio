<div align="center">
<img src="https://upload.wikimedia.org/wikipedia/commons/9/92/Openwrt_Logo.svg" alt="OpenWrt - Bluetooth Audio di OpenWrt" width="200"/>
<br>
<img src="https://upload.wikimedia.org/wikipedia/commons/b/bf/Armbianlogo.png" alt="OpenWrt - Bluetooth Audio di Armbian" width="200"/>

![License](https://img.shields.io/github/license/fahrulariza/OpenWRT-Pulse-Audio)
[![GitHub All Releases](https://img.shields.io/github/downloads/fahrulariza/OpenWRT-Pulse-Audio/total)](https://github.com/fahrulariza/OpenWRT-Pulse-Audio/releases)
![Total Commits](https://img.shields.io/github/commit-activity/t/fahrulariza/OpenWRT-Pulse-Audio)
![Top Language](https://img.shields.io/github/languages/top/fahrulariza/OpenWRT-Pulse-Audio)
[![Open Issues](https://img.shields.io/github/issues/fahrulariza/OpenWRT-Pulse-Audio)](https://github.com/fahrulariza/OpenWRT-Pulse-Audio/issues)

<h1>Bluetooth Audio in OpenWrt / Armbian</h1>
<p>Manage your OpenWrt and Armbian routers easily and creatively!</p>
</div>

Installing and Configuring Bluetooth Audio in OpenWrt
This guide explains the steps for installing and configuring audio drivers and services in OpenWrt, as well as a script to automate connecting to Bluetooth speakers at startup.

<p>
  
## Table of Contents

<br>
1. ⚙️ Initial Preparation**<br>
2. 🛠️ PulseAudio Configuration**<br>
3. 🛠️ Connection Test and Audio Playback**<br>
4. ⚙️ Automation with Startup Scripts**<br>
<br>

## 🚀 Future development features can be used for
- **Sound Alarm**
- **Azan Audio**
- **Internet Connection Alarm**
- **Smart IoT**
- **and others**

</p>
<br>

## 1. ⚙️ Initial Preparation
First, check in the terminal using `lsusb` whether the Bluetooth dongle module you will be using is detected and supported.
```
root@wow-wrt:/# lsusb
Bus 001 Device 003: ID 2717:ff88 Google Pixel
Bus 002 Device 001: ID 1d6b:0003 Linux 5.15.162-ophub xhci-hcd xHCI Host Controller
Bus 001 Device 004: ID 0bda:8153 Realtek USB 10/100/1000 LAN
Bus 001 Device 005: ID 33fa:0001 USB2.0-BT
Bus 001 Device 002: ID 05e3:0610 GenesysLogic USB2.1 Hub
Bus 001 Device 001: ID 1d6b:0002 Linux 5.15.162-ophub xhci-hcd xHCI Host Controller
root@wow-wrt:/#
```
This indicates that your USB Bluetooth dongle has been recognized by the OpenWrt system. The next step is to install and configure the necessary software packages to enable Bluetooth functionality. As seen above, `USB2.0-BT` with ID `33fa:0001` is detected, so you can proceed to the next step.

The first step is to ensure all necessary packages are installed.

<br>
<br>

1. Update the Package List:
```
opkg update
```
```
opkg install dos2unix
```
<br>

2. Install the Bluetooth Service:
```
opkg install kmod-bluetooth bluez-utils bluez-daemon
```
<br>

3. Install the Audio Service (PulseAudio) sequentially:
```
opkg install pulseaudio-daemon-avahi
```
```
opkg install pulseaudio pulseaudio-tools pulseaudio-module-bluetooth
```
<br>

> There is a file conflict when installing `pulseaudio-daemon-avahi` because the `pulseaudio-daemon` package is already installed and both are trying to provide the same files. <br>
> You must remove `pulseaudio-daemon` first before installing the avahi version.<br>
> Solution <br>
> To resolve this, use the `--force-depends` option to remove dependent packages.<br>
> Forcefully Remove Old Packages: <br>
> Forcefully remove the `pulseaudio-daemon` package and its dependencies. <br>
> This will remove `pulseaudio-daemon`, `pulseaudio-tools`, and `pulseaudio-profiles` in one command. <br>

```
opkg remove --force-removal-of-dependent-packages pulseaudio-daemon
```
> then repeat Install Audio Services (PulseAudio) in sequence:
<br>
<br>

## 2. 🛠️ PulseAudio Configuration
<br>

Modify the PulseAudio configuration file to enable the Bluetooth module.<br>
1. Open the `system.pa` file:

```
vi /etc/pulse/system.pa
```
<br>

2. Add the Bluetooth Module:<br>
> Add the following lines at the end of the file or in the appropriate section:<br>
```
### Bluetooth modules
load-module module-bluetooth-discover
load-module module-bluez5-discover
```
<br>

3. Disable Unnecessary Modules:<br>
> Ensure the `module-detect` and `module-console-kit` is disabled by prefixing it with a `#` to avoid errors. The relevant section will look like this:

```
# .ifexists module-detect.so
# load-module module-detect
# .endif

# .ifexists module-console-kit.so
# load-module module-console-kit
# .endif
```
> Save and close the file by pressing `Esc`, then type `:wq` and press Enter.
> Or you can modify the contents of the `system.pa` file via the provided `filemanager`.
> Here is the modified `system.pa` [here](https://github.com/fahrulariza/OpenWRT-Pulse-Audio/blob/master/etc/pulse/system.pa)
<br>

4. Create a PulseAudio directory and set permissions.

```
mkdir -p /var/run/pulse/.config/pulse
chown -R pulse:pulse /var/run/pulse
```
<br>
<br>

## 3. 🛠️ Test Connection and Audio Playback
Test all services manually before creating automation scripts. <br>

1. Restart the Service:
> Stop PulseAudio:

```
killall pulseaudio
```
> Check the D-Bus Configuration:
> Ensure the dbus service is running properly, as bluetoothd and pulseaudio heavily depend on it.

```
/etc/init.d/dbus start
```
> Restart the Bluetooth service:

```
/etc/init.d/bluetoothd restart
```
> Restart PulseAudio:
<br>
> Run PulseAudio again, now with the updated configuration.

```
sudo -u pulse pulseaudio --daemonize --disallow-exit --disable-shm --exit-idle-time=-1
```
<br>

2. Identify the Bluetooth Adapter:
```
hciconfig
```

> Make sure your adapter (e.g. hci0 or hci1) has a valid BD Address (not 00:00...) and the status is `UP RUNNING`.<br>
> example hciconfig is not `UP RUNNING` or status is `DOWN`<br>
> SKIP hciconfig if there is no connection to the device via Bluetooth.
> go to next stage 3. Test Bluetooth Connection:
```
root@riza-wrt:/# hciconfig

hci0:   Type: Primary  Bus: UART 

BD Address: 00:00:00:00:00:00  ACL MTU: 0:0  SCO MTU: 0:0 

DOWN 

RX bytes:0 acl:0 sco:0 events:0 errors:0 

TX bytes:42 acl:0 sco:0 commands:6 errors:0
```
> example hciconfig status `UP RUNNING` connected to bluetooth address `D0:53:58:F4:98:08`
```
root@wow-wrt:/#hciconfig
hci1: Type: Primary Bus: UART 
BD Address: 00:00:00:00:00:00 ACL MTU: 0:0 SCO MTU: 0:0 
DOWN 
RX bytes:0 acl:0 sco:0 events:0 errors:0 
TX bytes:14 acl:0 sco:0 commands:2 errors:0

hci0: Type: Primary Bus: USB 
BD Address: D0:53:58:F4:98:08 ACL MTU: 1021:9 SCO MTU: 255:4 
UP RUNNING 
RX bytes:27432458 acl:367 sco:0 events:3917831 errors:0 
TX bytes:-1883463402 acl:3915131 sco:0 commands:331 errors:0
```
<br>

3. Test Bluetooth Connection:
> Inside `bluetoothctl`, make sure your speaker is in pairing mode, then run the following command:
```
bluetoothctl
```
```
bluetoothctl# power on
```
```
bluetoothctl# agent on
```
```
bluetoothctl# scan on
```
> trust bluetooth if the Bluetooth device you want to connect to appears
```
bluetoothctl# trust D0:85:73:E4:98:08
```
> scan bluetooth if the Bluetooth device you want to connect to appears
```
[bluetooth]# connect D0:53:58:F4:98:08
```
> If successful, you will see a message confirming the device is connected. `Connection successful` will appear.

4. Identify the Bluetooth adapter again:
```
hciconfig
```

> Make sure your adapter (e.g., hci0 or hci1) has a valid BD Address (not 00:00...) and is in `UP RUNNING` status.<br>
> Bluetooth successfully connected to Bluetooth address `D0:53:58:F4:98:08` to `hci0`, but hciconfig status is `DOWN`
```
root@wow-wrt:/# hciconfig
hci1: Type: Primary Bus: UART
BD Address: 00:00:00:00:00:00 ACL MTU: 0:0 SCO MTU: 0:0
DOWN
RX bytes: 0 acl: 0 sco: 0 events: 0 errors: 0
TX bytes: 14 acl: 0 sco: 0 commands: 2 errors: 0

hci0: Type: Primary Bus: USB
BD Address: D0: 53: 58:F4: 98: 08 ACL MTU: 1021:9 SCO MTU: 255:4
DOWN
RX bytes: 27432458 acl: 367 sco: 0 events: 3917831 errors: 0
TX bytes: -1883463402 acl: 3915131 sco: 0 commands: 331 errors: 0
```
> Run the command `hciconfig hci0 up`. If the position is hci0 or hci1, use the hci according to the Bluetooth MAC interface.<br>
> Then check `hciconfig` again to see if it works. <br>
> example hciconfig status `UP RUNNING` connected to bluetooth address `D0:53:58:F4:98:08` <br>
```
root@wow-wrt:/#hciconfig
hci1: Type: Primary Bus: UART 
BD Address: 00:00:00:00:00:00 ACL MTU: 0:0 SCO MTU: 0:0 
DOWN 
RX bytes:0 acl:0 sco:0 events:0 errors:0 
TX bytes:14 acl:0 sco:0 commands:2 errors:0

hci0: Type: Primary Bus: USB 
BD Address: D0:53:58:F4:98:08 ACL MTU: 1021:9 SCO MTU: 255:4 
UP RUNNING 
RX bytes:27432458 acl:367 sco:0 events:3917831 errors:0
TX bytes:-1883463402 acl:3915131 sco:0 commands:331 errors:0
```
<br>

5. Test Audio Playback:
> to see the list of audio sinks. Check if the audio is routed to the Bluetooth stream.
```
sudo -u pulse pactl list short sinks
```
> if not, set the settings to the Bluetooth address. To ensure the Bluetooth speaker is the default output, use this command:
```
sudo -u pulse pactl set-default-sink bluez_sink.D0_53:58:F4:98_08.a2dp_sink
```
> Double-check that the audio has been redirected to the Bluetooth stream.
```
sudo -u pulse pactl list short sinks
```
If the output looks like this, it's successful.
```
root@wow-wrt:/# sudo -u pulse pactl list short sinks
1 bluez_sink.D0_85_73_E4_98_08.a2dp_sink module-bluez5-device.c s16le 2ch 44100Hz SUSPENDED
root@wow-wrt:/#
```
> The output shows that all configurations are correct.
> pactl list short sinks: Shows Bluetooth speakers (bluez_sink...) is detected as a sink. Its status is `SUSPENDED` because no audio is currently playing, which is normal.<br>
> pactl set-default-sink: Sets the Bluetooth speaker as the default audio output.<br>

If the Bluetooth sink appears, test audio playback:
> The audio file must be in `wav` format
```
sudo -u pulse paplay /path/to/file.wav
```
> If you hear sound through the Bluetooth speaker.
Congratulations, audio is playing successfully!

<br>
<br>

6. Automatically connects when OpenWRT is turned on.
<br>

> Use this script if you want it to be automated [here](https://github.com/fahrulariza/OpenWRT-Pulse-Audio/releases) <br>
> Save it somewhere easily accessible.<br>
> Edit the script and change the MAC address to match the Bluetooth speaker device you want to connect to.<br>
```
# Connect to Bluetooth speaker
echo "Connecting to D0:53:58:F4:98:08..."
bluetoothctl power on &
bluetoothctl agent on &
bluetoothctl trust D0:53:58:F4:98:08 &
bluetoothctl connect D0:53:58:F4:98:08 &
```
> Then provide a startup effect audio in this section. This will play when the Bluetooth speaker is successfully connected.
```
sudo -u pulse /usr/bin/paplay --volume=45536 /www/audio/startup.wav
```
> then change permissions and convert
```
chmod +x /path/to/audio-auto-connect.sh
```
```
dos2unix /path/to/audio-auto-connect.sh
```
> disable init.d startup on pulseaudio
```
/etc/init.d/pulseaudio disable
```
> then add it to `/etc/rc.local` so the `audio-auto-connect.sh` script runs automatically
```
# Start the work-attendance service in the background
/path/to/audio-auto-connect.sh &
```
> then restart OpenWRT and wait to see if the connected sound is heard on the Bluetooth speaker.
