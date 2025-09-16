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
Langkah pertama adalah memastikan semua paket yang diperlukan terinstal.
> 1. Perbarui Daftar Paket:
```
opkg update
```
> 2. Instal Layanan Bluetooth:
```
opkg install kmod-bluetooth bluez-utils bluez-daemon
```
> 3. Instal Layanan Audio (PulseAudio):
```
opkg install pulseaudio pulseaudio-tools pulseaudio-module-bluetooth
```
<br>

## 2. 🛠️ Konfigurasi PulseAudio
Modifikasi file konfigurasi PulseAudio untuk mengaktifkan modul Bluetooth.<br>

> 1. Buka file `system.pa`:

```
vi /etc/pulse/system.pa
```
> 2. Tambahkan Modul Bluetooth:<br>
>    Tambahkan baris berikut di akhir file atau di bagian yang sesuai:<br>
```
### Bluetooth modules
load-module module-bluetooth-discover
load-module module-bluez5-discover
```
> 3. Nonaktifkan Modul yang Tidak Perlu:<br>
>    Pastikan modul `module-detect` dan `module-console-kit` dinonaktifkan dengan menambahkan `#` di depannya untuk menghindari kesalahan. Bagian yang relevan akan terlihat seperti ini:
```
# .ifexists module-detect.so
# load-module module-detect
# .endif
# .ifexists module-console-kit.so
# load-module module-console-kit
# .endif
```
> Simpan dan tutup file dengan menekan `Esc`, lalu ketik `:wq` dan tekan Enter.
> atau kamu bisa melakukan perubahan isi file `system.pa` melalui filemanager
<br>

## 3. 🛠️ Tes Koneksi dan Pemutaran Audio







