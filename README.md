🇮🇩 Bahasa Indonesia | [🇬🇧 English](README.en.md)

# ESP32-C6 OTA Weather Station

![ESP32-C6](https://img.shields.io/badge/ESP32-C6-blue)
![Version](https://img.shields.io/badge/version-1.1.0-green)
![License](https://img.shields.io/badge/license-MIT-orange)

Stasiun cuaca IoT lengkap dengan update firmware OTA, provisioning WiFi, dan pemantauan cuaca real-time untuk Jakarta, Indonesia.

![Dashboard Preview](docs/images/dashboard-preview.png)
![Dashboard OTA](docs/images/dashboard-ota.png)
---

## 🌟 Fitur

### Fungsi Utama
- 🌐 **Provisioning WiFi** - Konfigurasi WiFi mudah lewat antarmuka web (mode AP)
- 🔄 **Update Firmware OTA** - Update over-the-air yang aman dengan sistem dual partition
- ⏰ **Real-Time Clock** - Sinkronisasi NTP dengan zona waktu WIB (GMT+7)
- 🌦️ **Pemantauan Cuaca** - Data suhu & kelembapan langsung dari Open-Meteo API
- 💡 **Indikator LED** - Umpan balik visual untuk status operasi sistem
- 📱 **Web UI Responsif** - Desain gradien yang menarik dan ramah perangkat mobile

### Fitur Teknis
- **Dual Partition OTA** - Update firmware yang aman dengan rollback otomatis
- **Factory Recovery** - Partisi cadangan untuk pemulihan sistem
- **Mode APSTA** - Mode AP dan Station berjalan bersamaan
- **Dukungan HTTPS** - Komunikasi API aman dengan validasi sertifikat
- **JSON REST API** - Mudah diintegrasikan dengan sistem eksternal

---

## 📋 Daftar Isi

- [Kebutuhan Hardware](#kebutuhan-hardware)
- [Kebutuhan Software](#kebutuhan-software)
- [Mulai Cepat](#mulai-cepat)
- [Struktur Project](#struktur-project)
- [Antarmuka Web](#antarmuka-web)
- [Dokumentasi API](#dokumentasi-api)
- [Indikator LED](#indikator-led)
- [Konfigurasi](#konfigurasi)
- [Proses Update OTA](#proses-update-ota)
- [Pemecahan Masalah](#pemecahan-masalah)
- [Arsitektur](#arsitektur)
- [Kontribusi](#kontribusi)
- [Lisensi](#lisensi)

---

## 🔧 Kebutuhan Hardware

- **Board ESP32-C6 Development** (varian apa pun dengan flash 4MB)
- **3x LED** (warna bebas, dengan resistor yang sesuai)
- **Breadboard & Kabel Jumper**
- **Kabel USB-C** untuk pemrograman

### Diagram Wiring
```
ESP32-C6          LED
GPIO 4  ────────  LED Status Sistem (WiFi/OTA/Recovery)
GPIO 5  ────────  LED Fetch Cuaca
GPIO 6  ────────  LED Mode AP
GND     ────────  Common Cathode LED (lewat resistor)
```

**Resistor yang disarankan:** 220Ω - 330Ω per LED

---

## 💻 Kebutuhan Software

- **ESP-IDF v5.4+** ([Panduan Instalasi](https://docs.espressif.com/projects/esp-idf/en/latest/esp32c6/get-started/))
- **Python 3.8+**
- **Git**

### Setup ESP-IDF
```bash
# Install ESP-IDF
git clone -b v5.4 --recursive https://github.com/espressif/esp-idf.git
cd esp-idf
./install.sh esp32c6
python -m venv venv
.\venv\Scripts\activate
# Aktifkan environment
. ./export.sh
```

---

## 🚀 Mulai Cepat

### 1. Clone Repository
```bash
git clone https://github.com/yourusername/esp32c6-ota-weather.git
cd esp32c6-ota-weather
```

### 2. Konfigurasi Project
```bash
# Set target
idf.py set-target esp32c6

# (Opsional) Konfigurasi menuconfig
idf.py menuconfig
```

### 3. Build & Flash
```bash
# Build project
idf.py build

# bersihkan build
idf.py fullclean

# Flash ke perangkat
 python -m esptool --chip esp32c6 -b 460800 --before default_reset --after hard_reset write_flash --flash_mode dio --flash_size 4MB --flash_freq 80m 0x0 build/bootloader/bootloader.bin 0x8000 build/partition_table/partition-table.bin 0x10000 build/esp32c6-ota-weather.bin

# hapus memori flash
python -m esptool --chip esp32c6 --port COM5 erase_flash

# monitor
python -m serial.tools.miniterm "COM5" 115200
```

### 4. Setup Pertama Kali

1. **Hubungkan ke AP:**
   - SSID: `ESP32-C6-Setup`
   - Password: `12345678`

2. **Buka Browser:**
   - Kunjungi: `http://192.168.4.1`

3. **Konfigurasi WiFi:**
   - Masukkan kredensial WiFi kamu
   - Klik "Connect to WiFi"
   - Perangkat akan restart dan terhubung

4. **Akses Dashboard:**
   - Cari IP perangkat di serial monitor
   - Buka `http://[IP-PERANGKAT]/` di browser

---

## 📁 Struktur Project
```
esp32c6-ota-weather/
├── CMakeLists.txt              # Konfigurasi CMake root
├── sdkconfig.defaults          # Konfigurasi default project
├── partitions.csv              # Tabel partisi (Factory + OTA)
├── README.md                   # File ini
├── docs/
│   └── architecture.md         # Dokumentasi arsitektur
├── components/
│   ├── led_indicator/          # Komponen kontrol LED
│   │   ├── led_indicator.c
│   │   ├── include/led_indicator.h
│   │   └── CMakeLists.txt
│   ├── wifi_manager/           # Manajemen koneksi WiFi
│   │   ├── wifi_manager.c
│   │   ├── include/wifi_manager.h
│   │   └── CMakeLists.txt
│   ├── sntp_sync/              # Sinkronisasi waktu NTP
│   │   ├── sntp_sync.c
│   │   ├── include/sntp_sync.h
│   │   └── CMakeLists.txt
│   ├── ota_manager/            # Logika update OTA
│   │   ├── ota_manager.c
│   │   ├── include/ota_manager.h
│   │   └── CMakeLists.txt
│   ├── weather_client/         # Klien API cuaca
│   │   ├── weather_client.c
│   │   ├── include/weather_client.h
│   │   └── CMakeLists.txt
│   └── web_server/             # HTTP server & web UI
│       ├── web_server.c
│       ├── include/web_server.h
│       └── CMakeLists.txt
└── main/
    ├── main.c                  # Aplikasi utama
    └── CMakeLists.txt
```

---

## 🌐 Antarmuka Web

### Dashboard Utama

Akses di: `http://[IP-PERANGKAT]/`

**Fitur:**
- ✅ Tampilan waktu saat ini (zona waktu WIB)
- ✅ Informasi cuaca (Suhu & Kelembapan)
- ✅ Status koneksi WiFi
- ✅ Informasi jaringan (IP, Gateway)
- ✅ Form konfigurasi ulang WiFi
- ✅ Tautan ke halaman update OTA

### Halaman Update OTA

Akses di: `http://[IP-PERANGKAT]/ota`

**Fitur:**
- 📤 Upload firmware dengan drag & drop
- 📊 Progres upload real-time
- ℹ️ Versi firmware & informasi partisi saat ini
- ⚠️ Peringatan keamanan

---

## 🔌 Dokumentasi API

### Base URL
```
http://[IP-PERANGKAT]/api
```

### Endpoint

#### 1. Ambil Status Sistem
```http
GET /api/status
```

**Respons:**
```json
{
  "connected": true,
  "ssid": "IoT_M2M",
  "ip": "192.168.8.136",
  "subnet": "255.255.255.0",
  "gateway": "192.168.8.1",
  "ap_ip": "192.168.4.1"
}
```

#### 2. Ambil Waktu Saat Ini
```http
GET /api/time
```

**Respons:**
```json
{
  "synced": true,
  "time": "16.02.2026 23:45:30",
  "year": 2026,
  "month": 2,
  "day": 16,
  "hour": 23,
  "minute": 45,
  "second": 30,
  "epoch": 1771259130
}
```

#### 3. Ambil Data Cuaca
```http
GET /api/weather
```

**Respons:**
```json
{
  "valid": true,
  "temperature": 26.4,
  "humidity": 91,
  "last_update": 1771259073,
  "last_update_str": "16.02.2026 23:34:33"
}
```

#### 4. Simpan Konfigurasi WiFi
```http
POST /api/wifi/save
Content-Type: application/json

{
  "ssid": "MyWiFi",
  "password": "mypassword"
}
```

**Respons:**
```json
{
  "success": true
}
```

#### 5. Ambil Info OTA
```http
GET /api/ota/info
```

**Respons:**
```json
{
  "version": "1.0.0",
  "partition": "ota_0",
  "free_space": 1376256
}
```

#### 6. Upload Firmware
```http
POST /api/ota/update
Content-Type: multipart/form-data

file: [file .bin biner]
```

**Respons:**
```json
{
  "success": true
}
```

---

## 💡 Indikator LED

### GPIO 4 - LED Status Sistem

| Pola | Arti |
|---------|---------|
| **Menyala terus** | WiFi terhubung (operasi normal) |
| **Kedip 200ms** | Update OTA sedang berjalan |
| **Kedip 1000ms** | Mode recovery / Error sistem |
| **Mati** | WiFi terputus |

### GPIO 5 - LED Fetch Cuaca

| Pola | Arti |
|---------|---------|
| **Kedip** | Mengambil data cuaca dari API |
| **Menyala (2 detik)** | Fetch berhasil diselesaikan |
| **Mati** | Idle |

### GPIO 6 - LED Mode AP

| Pola | Arti |
|---------|---------|
| **Menyala terus** | Mode AP aktif (provisioning tersedia) |
| **Mati** | Hanya mode STA |

---

## ⚙️ Konfigurasi

### Pengaturan WiFi

Kredensial AP default (ubah di `wifi_manager.h`):
```c
#define WIFI_AP_SSID        "ESP32-C6-Setup"
#define WIFI_AP_PASSWORD    "12345678"
#define WIFI_AP_IP          "192.168.4.1"
```

### Pengaturan Cuaca

Lokasi (ubah di `weather_client.h`):
```c
#define WEATHER_LATITUDE    "-6.1818"
#define WEATHER_LONGITUDE   "106.8223"
#define WEATHER_FETCH_INTERVAL_MS   (3600000)  // 1 jam
```

### Versi Firmware

Ubah di `ota_manager.h`:
```c
#define FIRMWARE_VERSION "1.0.0"
```

### Tabel Partisi

Edit `partitions.csv` untuk ukuran partisi kustom:
```csv
factory,  app,  factory, 0x10000, 0x150000,  # 1.31 MB
ota_0,    app,  ota_0,   ,        0x150000,  # 1.31 MB
ota_1,    app,  ota_1,   ,        0x150000,  # 1.31 MB
```

---

## 🔄 Proses Update OTA

### Metode 1: Lewat Antarmuka Web (Direkomendasikan)

1. Build firmware baru:
```bash
   idf.py build
```

2. Cari file biner:
```
   build/esp32c6-ota-weather.bin
```

3. Buka halaman OTA:
```
   http://[IP-PERANGKAT]/ota
```

4. Upload file `.bin`

5. Tunggu upload & verifikasi selesai

6. Perangkat reboot otomatis

### Metode 2: Lewat Command Line
```bash
# Memakai curl
curl -X POST http://[IP-PERANGKAT]/api/ota/update \
  -F "file=@build/esp32c6-ota-weather.bin"
```

### Proteksi Rollback

- Sistem dual partition (ota_0 ↔ ota_1)
- Rollback otomatis jika boot gagal
- Partisi factory sebagai pemulihan terakhir

---

## 🐛 Pemecahan Masalah

### Perangkat tidak menampilkan mode AP

**Solusi:**
- Tekan tombol reset
- Periksa LED 6 (harus menyala saat mode AP)
- Cari `ESP32-C6-Setup` di daftar jaringan WiFi

### Tidak bisa terhubung ke WiFi

**Solusi:**
- Periksa SSID & password sudah benar
- Pastikan jaringan 2.4GHz (ESP32 tidak mendukung 5GHz)
- Periksa router mengizinkan perangkat baru
- Lihat log serial: `idf.py monitor`

### Data cuaca menampilkan "unavailable"

**Solusi:**
- Periksa koneksi internet
- Pastikan waktu sudah tersinkronisasi (NTP)
- Periksa endpoint API di log
- Certificate bundle mungkin perlu diperbarui

### Update OTA gagal

**Solusi:**
- Pastikan file `.bin` adalah firmware yang benar
- Periksa ukuran file < ukuran partisi (1.31 MB)
- Pastikan koneksi WiFi stabil
- Periksa log serial untuk detail error

### Waktu tidak tersinkronisasi

**Solusi:**
- Periksa akses internet WiFi
- Pastikan server NTP dapat dijangkau
- Tunggu hingga 30 detik untuk sinkronisasi pertama
- Periksa firewall mengizinkan NTP (UDP port 123)

---

## 🏗️ Arsitektur

Lihat dokumentasi arsitektur lengkap: [docs/architecture.md](docs/architecture.md)

**Gambaran umum:**
```
┌─────────────────────────────────────────────────────┐
│                   Web Browser                        │
│          (Dashboard & Antarmuka OTA)                 │
└──────────────────┬──────────────────────────────────┘
                   │ HTTP/HTTPS
                   ↓
┌─────────────────────────────────────────────────────┐
│                Komponen Web Server                   │
│  (Rendering HTML, Endpoint REST API, Handler)       │
└──┬────────┬─────────┬──────────┬────────────────┬───┘
   │        │         │          │                │
   ↓        ↓         ↓          ↓                ↓
┌──────┐ ┌────────┐ ┌──────┐ ┌──────────┐ ┌──────────┐
│ WiFi │ │  SNTP  │ │ OTA  │ │ Weather  │ │   LED    │
│ Mgr  │ │  Sync  │ │ Mgr  │ │  Client  │ │Indicator │
└──────┘ └────────┘ └──────┘ └──────────┘ └──────────┘
```

---

## 🤝 Kontribusi

Kontribusi sangat diterima! Ikuti langkah berikut:

1. Fork repository
2. Buat branch fitur (`git checkout -b feature/AmazingFeature`)
3. Commit perubahan (`git commit -m 'Add some AmazingFeature'`)
4. Push ke branch (`git push origin feature/AmazingFeature`)
5. Buka Pull Request

---

## 📝 Lisensi

Project ini dilisensikan di bawah MIT License - lihat file [LICENSE](LICENSE) untuk detailnya.

---

## 🙏 Ucapan Terima Kasih

- **Espressif Systems** - Framework ESP-IDF
- **Open-Meteo** - API cuaca gratis
- **Kontributor komunitas** - Laporan bug dan permintaan fitur

---


**Dibuat dengan ❤️ menggunakan ESP32-C6 oleh MJ Mokhtar**
