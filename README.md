<div align="center">
<img src="https://upload.wikimedia.org/wikipedia/commons/9/92/Openwrt_Logo.svg" alt="OpenWrt - Bluetooth Audio di OpenWrt" width="200"/>

![License](https://img.shields.io/github/license/fahrulariza/OpenWRT-Pulse-Audio)
[![GitHub All Releases](https://img.shields.io/github/downloads/fahrulariza/OpenWRT-Pulse-Audio/total)](https://github.com/fahrulariza/OpenWRT-Pulse-Audio/releases)
![Total Commits](https://img.shields.io/github/commit-activity/t/fahrulariza/OpenWRT-Pulse-Audio)
![Top Language](https://img.shields.io/github/languages/top/fahrulariza/OpenWRT-Pulse-Audio)
[![Open Issues](https://img.shields.io/github/issues/fahrulariza/OpenWRT-Pulse-Audio)](https://github.com/fahrulariza/OpenWRT-Pulse-Audio/issues)

<h1>Bluetooth Audio di OpenWrt</h1>
<p>Kelola router OpenWrt Anda dengan mudah dan kreatif!</p>
</div>

Instalasi dan Konfigurasi Bluetooth Audio di OpenWrt
Panduan ini menjelaskan langkah-langkah untuk menginstal dan mengkonfigurasi driver dan layanan audio di OpenWrt, serta skrip untuk otomatisasi koneksi ke speaker Bluetooth saat startup.

<p>
Daftar Isi
<br>
>  1. ⚙️ Persiapan Awal<br>
>  2. 🛠️ Konfigurasi PulseAudio<br>
>  3. 🛠️ Tes Koneksi dan Pemutaran Audio
>  4. Otomatisasi dengan Skrip Startup
</p>
<br>

## 1. ⚙️ Persiapan Awal
cek dulu di terminal melalui `lsusb` apakah module dongle bluetooth yang akan digunakan terdeteksi dan support.<br>
```
root@wow-wrt:/# lsusb
Bus 001 Device 003: ID 2717:ff88 Google Pixel
Bus 002 Device 001: ID 1d6b:0003 Linux 5.15.162-ophub xhci-hcd xHCI Host Controller
Bus 001 Device 004: ID 0bda:8153 Realtek USB 10/100/1000 LAN
Bus 001 Device 005: ID 33fa:0001  USB2.0-BT
Bus 001 Device 002: ID 05e3:0610 GenesysLogic USB2.1 Hub
Bus 001 Device 001: ID 1d6b:0002 Linux 5.15.162-ophub xhci-hcd xHCI Host Controller
root@wow-wrt:/# 
```
Ini menunjukkan bahwa dongle Bluetooth USB Anda sudah dikenali oleh sistem OpenWrt. Langkah selanjutnya adalah menginstal dan mengkonfigurasi paket perangkat lunak yang diperlukan untuk mengaktifkan fungsionalitas Bluetooth.terlihat diatas ada terdeteksi `USB2.0-BT` dengan ID `33fa:0001` maka bisa dilanjutkan ketahap selanjutnya.<br>

Langkah pertama adalah memastikan semua paket yang diperlukan terinstal.
<br>
<br>

1. Perbarui Daftar Paket:
```
opkg update
```
```
opkg install dos2unix
```
<br>

2. Instal Layanan Bluetooth:
```
opkg install kmod-bluetooth bluez-utils bluez-daemon
```
<br>

3. Instal Layanan Audio (PulseAudio) secara berurutan:
```
opkg install pulseaudio-daemon-avahi
```
```
opkg install pulseaudio pulseaudio-tools pulseaudio-module-bluetooth
```
<br>

>   bila Ada konflik file saat menginstal `pulseaudio-daemon-avahi` karena paket `pulseaudio-daemon` sudah terinstal dan keduanya mencoba menyediakan file yang sama. <br>
>   Anda harus menghapus `pulseaudio-daemon` terlebih dahulu sebelum menginstal versi avahi.<br>
>   Solusi <br>
>   Untuk mengatasi ini, gunakan opsi `--force-depends` untuk menghapus paket yang saling bergantung.<br>
>   Hapus Paket Lama secara Paksa: <br>
>   Hapus paket `pulseaudio-daemon` dan dependensinya secara paksa. <br>
>   Ini akan menghapus `pulseaudio-daemon`, `pulseaudio-tools`, dan `pulseaudio-profiles` dalam satu perintah. <br>

```
opkg remove --force-removal-of-dependent-packages pulseaudio-daemon
```
>   lalu ulangi Instal Layanan Audio (PulseAudio) secara berurutan:
<br>
<br>

## 2. 🛠️ Konfigurasi PulseAudio
<br>

Modifikasi file konfigurasi PulseAudio untuk mengaktifkan modul Bluetooth.<br>

1. Buka file `system.pa`:

```
vi /etc/pulse/system.pa
```
<br>

2. Tambahkan Modul Bluetooth:<br>
>    Tambahkan baris berikut di akhir file atau di bagian yang sesuai:<br>
```
### Bluetooth modules
load-module module-bluetooth-discover
load-module module-bluez5-discover
```
<br>

3. Nonaktifkan Modul yang Tidak Perlu:<br>
>    Pastikan modul `module-detect` dan `module-console-kit` dinonaktifkan dengan menambahkan `#` di depannya untuk menghindari kesalahan. Bagian yang relevan akan terlihat seperti ini:<br>
```
# .ifexists module-detect.so
# load-module module-detect
# .endif

# .ifexists module-console-kit.so
# load-module module-console-kit
# .endif
```
> Simpan dan tutup file dengan menekan `Esc`, lalu ketik `:wq` dan tekan Enter.<br>
> atau kamu bisa melakukan perubahan isi file `system.pa` melalui filemanager
<br>

4. Buat direktori PulseAudio dan atur izin<br>

```
mkdir -p /var/run/pulse/.config/pulse
chown -R pulse:pulse /var/run/pulse
```
<br>
<br>

## 3. 🛠️ Tes Koneksi dan Pemutaran Audio
Uji semua layanan secara manual sebelum membuat skrip otomatisasi.
<br>

1. Mulai Ulang Layanan:
>   Hentikan PulseAudio:<br>
```
killall pulseaudio
```
>   Periksa Konfigurasi D-Bus:<br>
>   Pastikan layanan dbus berjalan dengan benar, karena bluetoothd dan pulseaudio sangat bergantung padanya.<br>
```
/etc/init.d/dbus start
```
>   Mulai ulang layanan Bluetooth:<br>
```
/etc/init.d/bluetoothd restart
```
>   Mulai ulang lagi PulseAudio:<br>
>   Jalankan PulseAudio lagi, sekarang dengan konfigurasi yang diperbarui.<br>
```
sudo -u pulse pulseaudio --daemonize --disallow-exit --disable-shm --exit-idle-time=-1
```
<br>

2. Identifikasi Adaptor Bluetooth:
```
hciconfig
```

>   Pastikan adaptor Anda (misalnya hci0 atau hci1) memiliki BD Address yang valid (bukan 00:00...) dan status `UP RUNNING`.<br>
>   contoh hciconfig belum `UP RUNNING` atau status `DOWN`<br>
> LEWATKAN dulu hciconfig jika belum ada koneksi ke perangkat melalui bluetooth.
> lanjut ke tahap selanjutnya 3. Uji Koneksi Bluetooth:
```
root@riza-wrt:/# hciconfig

hci0:   Type: Primary  Bus: UART

        BD Address: 00:00:00:00:00:00  ACL MTU: 0:0  SCO MTU: 0:0

        DOWN 

        RX bytes:0 acl:0 sco:0 events:0 errors:0

        TX bytes:42 acl:0 sco:0 commands:6 errors:0
```
>   contoh hciconfig status  `UP RUNNING` terkoneksi ke alamat bluetooth `D0:53:58:F4:98:08`
```
root@wow-wrt:/# hciconfig
hci1:   Type: Primary  Bus: UART
        BD Address: 00:00:00:00:00:00  ACL MTU: 0:0  SCO MTU: 0:0
        DOWN 
        RX bytes:0 acl:0 sco:0 events:0 errors:0
        TX bytes:14 acl:0 sco:0 commands:2 errors:0

hci0:   Type: Primary  Bus: USB
        BD Address: D0:53:58:F4:98:08  ACL MTU: 1021:9  SCO MTU: 255:4
        UP RUNNING 
        RX bytes:27432458 acl:367 sco:0 events:3917831 errors:0
        TX bytes:-1883463402 acl:3915131 sco:0 commands:331 errors:0
```
<br>

3. Uji Koneksi Bluetooth:
>  Di dalam `bluetoothctl`, pastikan speaker Anda dalam mode pairing, lalu jalankan perintah berikut:
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
>   trust bluetooth jika tampil bluetooth yang ingin di koneksikan
```
bluetoothctl# trust D0:85:73:E4:98:08
```
>   scan bluetooth jika tampil bluetooth yang ingin di koneksikan
```
[bluetooth]# connect D0:53:58:F4:98:08
```
>   Jika berhasil, Anda akan melihat pesan yang mengonfirmasi bahwa perangkat terhubung akan muncul pesan `Connection successful`.
<br>

4. Identifikasi lagi Adaptor Bluetooth:
```
hciconfig
```

>   Pastikan adaptor Anda (misalnya hci0 atau hci1) memiliki BD Address yang valid (bukan 00:00...) dan status `UP RUNNING`.<br>
>   bluetooth berhasil terkoneksi ke alamat bluetooth `D0:53:58:F4:98:08` ke `hci0` tapi hciconfig status  `DOWN` 
```
root@wow-wrt:/# hciconfig
hci1:   Type: Primary  Bus: UART
        BD Address: 00:00:00:00:00:00  ACL MTU: 0:0  SCO MTU: 0:0
        DOWN 
        RX bytes:0 acl:0 sco:0 events:0 errors:0
        TX bytes:14 acl:0 sco:0 commands:2 errors:0

hci0:   Type: Primary  Bus: USB
        BD Address: D0:53:58:F4:98:08  ACL MTU: 1021:9  SCO MTU: 255:4
        DOWN 
        RX bytes:27432458 acl:367 sco:0 events:3917831 errors:0
        TX bytes:-1883463402 acl:3915131 sco:0 commands:331 errors:0
```
>   jalankan perintah `hciconfig hci0 up` jika posisinya di hci0 atau hci1 maka gunakan hci sesuai interface MAC bluetooth.<br>
>   lalu cek kembali `hciconfig` apakah berhasil. <br>
>   contoh hciconfig status  `UP RUNNING` terkoneksi ke alamat bluetooth `D0:53:58:F4:98:08` <br>
```
root@wow-wrt:/# hciconfig
hci1:   Type: Primary  Bus: UART
        BD Address: 00:00:00:00:00:00  ACL MTU: 0:0  SCO MTU: 0:0
        DOWN 
        RX bytes:0 acl:0 sco:0 events:0 errors:0
        TX bytes:14 acl:0 sco:0 commands:2 errors:0

hci0:   Type: Primary  Bus: USB
        BD Address: D0:53:58:F4:98:08  ACL MTU: 1021:9  SCO MTU: 255:4
        UP RUNNING 
        RX bytes:27432458 acl:367 sco:0 events:3917831 errors:0
        TX bytes:-1883463402 acl:3915131 sco:0 commands:331 errors:0
```
<br>

5. Uji Pemutaran Audio:
> untuk melihat daftar sink audio. Cek apakah audio dialihkan ke aliran bluetooth
```
sudo -u pulse pactl list short sinks
```
> jika belum maka set pengaturan ke alamat bluetooth. Jika ingin memastikan speaker Bluetooth adalah output default, gunakan perintah ini:
```
sudo -u pulse pactl set-default-sink bluez_sink.D0_53:58:F4:98_08.a2dp_sink
```
> Cek lagi apakah audio sudah dialihkan ke aliran bluetooth
```
sudo -u pulse pactl list short sinks
```
jika hasilnya seperti ini maka berhasil
```
root@wow-wrt:/# sudo -u pulse pactl list short sinks
1       bluez_sink.D0_85_73_E4_98_08.a2dp_sink  module-bluez5-device.c  s16le 2ch 44100Hz       SUSPENDED
root@wow-wrt:/#
```
>   Output yang berikan menunjukkan bahwa semua konfigurasi sudah benar.<br>
>   pactl list short sinks: Menunjukkan speaker Bluetooth (bluez_sink...) terdeteksi sebagai sink. Statusnya `SUSPENDED` karena tidak ada audio yang diputar saat itu, ini normal.<br>
>   pactl set-default-sink: Mengatur speaker Bluetooth sebagai output audio default.<br>

Jika sink Bluetooth muncul, uji pemutaran audio:
>   file audio harus berformat `wav`
```
sudo -u pulse paplay /path/ke/file.wav
```
>   Jika terdengar suara melalui Speaker Bluetooth.
Selamat, audio berhasil diputar!
<br>
<br>

6. Otomatis terkoneksi saat OpenWRT dihidupkan.
<br>

> Gunakan script ini jika ingin otomatis [here](https://github.com/fahrulariza/OpenWRT-Pulse-Audio/releases) <br>
> simpan ditempat yang mudah diakses.<br>
> edit pada script ubah MAC sesuai perangkat Speaker Bluetooth yang ingin di koneksikan.<br>
```
# Hubungkan ke speaker Bluetooth
echo "Menghubungkan ke D0:53:58:F4:98:08..."
bluetoothctl power on &
bluetoothctl agent on &
bluetoothctl trust D0:53:58:F4:98:08 &
bluetoothctl connect D0:53:58:F4:98:08 &
```
> lalu sediakan audio efek startup pada bagian ini. yang mana bila berhasil terkoneksi ke perangkat Speaker Bluetooth yang dituju maka akan diputar.
```
sudo -u pulse /usr/bin/paplay --volume=45536 /www/audio/startup.wav
```
> lalu ubah perizinan dan konversi
```
chmod +x /path/ke/audio-auto-connect.sh
```
```
dos2unix /path/ke/audio-auto-connect.sh
```
> disable startup init.d pada pulseaudio
```
/etc/init.d/pulseaudio disable
```
> lalu tambahkan di `/etc/rc.local` untuk script `audio-auto-connect.sh` berjalan otomatis
```
# Mulai layanan absensi-kerja di background
/path/ke/audio-auto-connect.sh &
```
> lalu restart OpenWRT dan Tunggu apakah berhasil terdengar Suara terkoneksi di Bluetooth Speaker.

