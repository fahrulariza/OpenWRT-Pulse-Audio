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
  1. Persiapan Awal<br>
  2. Konfigurasi PulseAudio<br>
3. Tes Koneksi dan Pemutaran Audio
4. Otomatisasi dengan Skrip Startup
</p>
<br>

## 1. Persiapan Awal
Langkah pertama adalah memastikan semua paket yang diperlukan terinstal.
Perbarui Daftar Paket:
```
opkg update
```
