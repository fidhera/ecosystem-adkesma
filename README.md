<div align="center">

<img src="Logo%20ECA.png" alt="Ecosystem Adkesma Logo" width="550"/>

<br/>
<br/>

<!-- Identity & Community -->
[![Built by fidhera](https://img.shields.io/badge/Built%20by-fidhera-E50914?style=for-the-badge&logo=github)](https://github.com/fidhera)
[![Instagram](https://img.shields.io/badge/Instagram-Instagram-E4405F?style=for-the-badge&logo=instagram&logoColor=white)](https://instagram.com/fidhera)
[![Discord](https://img.shields.io/badge/Discord-ECA%20Server-5865F2?style=for-the-badge&logo=discord&logoColor=white)](https://discord.gg/fHxRMvgj4) 
[![License](https://img.shields.io/badge/License-MIT-F59E0B?style=for-the-badge)](LICENSE)

<br/>

<!-- Core & Automation Tech Stack -->
[![Python Version](https://img.shields.io/badge/Python-3.12-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![GitHub Actions](https://img.shields.io/badge/CI%2FCD-GitHub%20Actions-2088FF?style=for-the-badge&logo=githubactions&logoColor=white)](https://github.com/features/actions)
[![Selenium](https://img.shields.io/badge/Automation-Selenium-43B02A?style=for-the-badge&logo=selenium&logoColor=white)](https://www.selenium.dev/)
[![BeautifulSoup](https://img.shields.io/badge/Scraping-BeautifulSoup4-4B8BBE?style=for-the-badge)](https://www.crummy.com/software/BeautifulSoup/)
[![Discord Webhook API](https://img.shields.io/badge/Delivery-Discord%20Webhook-5865F2?style=for-the-badge&logo=discord&logoColor=white)](https://discord.com/developers/docs/resources/webhook)

<br/>

**ECA (Ecosystem Adkesma) - Gunadarma News Scraper** adalah bot otomatisasi pemantauan informasi kampus terpadu berbasis *multi-portal scraper*. Sistem ini mengekstraksi pengumuman akademik dari domain **BAAK**, **LePKom**, **Studentsite**, dan **V-Class**, memvalidasi riwayat duplikasi (*anti-spam*), serta menyiarkan notifikasi *rich embed* secara langsung ke kanal Discord melalui orkestrasi **GitHub Actions**.

---

[Deskripsi](#deskripsi) • [Fitur Utama](#fitur-utama) • [Arsitektur Sistem](#arsitektur-sistem) • [Panduan Instalasi](#panduan-instalasi) • [Struktur Direktori](#struktur-direktori) • [Penulis](#penulis)
</div>

---

## Deskripsi

Mahasiswa sering kali terlambat menerima pengumuman penting akademik dikarenakan informasi kampus terfragmentasi di berbagai domain terpisah. Selain itu, sebagian situs menerapkan proteksi ketat (*Cloudflare*) atau menggunakan perenderan dinamis berbasis JavaScript yang menyulitkan penarikan data secara konvensional.

**ECA News Scraper** mengatasi permasalahan tersebut melalui alur otomatisasi modular:
1. **Pemisahan Modul Pengikis (Modular Scraping):** Pemisahan skrip ekstraksi per domain memudahkan pemeliharaan selektor DOM jika terjadi perubahan antarmuka web.
2. **Penanganan DOM Dinamis & Cloudflare:** Menggunakan Selenium WebDriver (Headless/Headed) dan BeautifulSoup untuk mengeksekusi JavaScript di latar belakang, serta mekanisme *fallback* data lokal untuk situs dengan proteksi protektif.
3. **Penyaringan Duplikasi (State Persistence):** Menyimpan status riwayat pengumuman di `data/last_updates.json` menggunakan algoritma pembanding judul guna menjamin saluran komunikasi bebas dari pesan berulang (*anti-spam*).
4. **Distribusi Terjadwal Tanpa Peladen Mandiri (Serverless CI/CD):** Dijalankan otomatis menggunakan *Cron Scheduler* di GitHub Actions setiap 2 jam sekali pada jam aktif operasional akademik.

---

## Fitur Utama

- **Pemantauan Terpusat 4 Portal Kampus:**
  - **BAAK:** Pemantauan informasi KRS, kalender akademik, dan surat edaran.
  - **LePKom:** Pemantauan jadwal kursus pengulangan, ujian praktikum, dan transfer praktikan.
  - **Studentsite (v4):** Pemantauan rekrutmen asisten laboratorium dan lowongan karier.
  - **V-Class:** Pemantauan pengumuman forum perkuliahan berbasis Moodle.
- **Lapisan Anti-Spam (State Buffer):** Menyimpan indeks 50 riwayat judul terakhir per portal ke dalam `last_updates.json`.
- **Eksekusi Terjadwal Otomatis:** Workflow GitHub Actions berjalan pada rentang jam kerja (07:00 s.d 21:00 WIB / UTC: 0, 2, 4, 6, 8, 10, 12, 14).
- **Distribusi Multi-Webhook:** Memetakan setiap portal ke kanal Discord masing-masing dengan pewarnaan tema kartu (*embed*) yang spesifik.
- **Pencegahan Limit API:** Menerapkan jeda waktu (*throttling*) 2 detik antar transmisi data ke endpoint Discord API.

---

## Arsitektur Sistem

```text
               [ GitHub Actions Scheduler (Cron / Manual Dispatch) ]
                                         │
                                         ▼
                            [ main.py (Orchestrator) ]
                                         │
                 ┌───────────────────────┴───────────────────────┐
                 ▼                                               ▼
     [ Muat .env / Secrets ]                         [ load_history() JSON ]
                 │                                               │
                 └───────────────────────┬───────────────────────┘
                                         ▼
                 [ Pipeline Eksekusi Multi-Portal Scraper ]
    ┌────────────────┬─────────────────────┬──────────────────┬─────────────────┐
    │                │                     │                  │                 │
    ▼                ▼                     ▼                  ▼                 ▼
[ BAAK Scraper ] [ LePKom Scraper ] [ Studentsite Scraper ] [ V-Class Scraper ]
(Selenium / CSV) (Selenium Headless)   (Selenium DOM v4)    (Moodle Selector)
    │                │                     │                  │                 │
    └────────────────┴─────────────────────┼──────────────────┴─────────────────┘
                                           ▼
                       [ Ekstraksi Berita Baru (reversed) ]
                                           │
                                           ▼
                     [ Validasi: Judul Sudah Ada di History? ]
                                   │               │
                               (Tidak)            (Ya)
                                   │               │
                                   ▼               ▼
                        [ send_to_discord() ]   [ Lewati / Skip ]
                                   │
                                   ▼
                       [ Respon HTTP: 200 / 204 ]
                                   │
                                   ▼
                    [ Simpan ke last_updates.json ]
                                   │
                                   ▼
                    [ Auto-Commit & Push ke GitHub ]

```

---

## Panduan Instalasi
Ikuti langkah-langkah berikut secara berurutan untuk menjalankan proyek di lingkungan komputer lokal.

### 1. Kloning Repositori

Buka Terminal / Command Prompt / PowerShell, lalu jalankan:

```bash
git clone [https://github.com/fidhera/ecosystem-adkesma.git](https://github.com/fidhera/ecosystem-adkesma.git)
cd ecosystem-adkesma
```

### 2. Konfigurasi Lingkungan Virtual (Virtual Environment)

Membuat lingkungan isolasi dependensi Python.

**Windows (PowerShell):**

```powershell
python -m venv venv
Set-ExecutionPolicy -Scope Process -ExecutionPolicy RemoteSigned
.\venv\Scripts\Activate.ps1
```

**Windows (Command Prompt):**

```cmd
python -m venv venv
.\venv\Scripts\activate.bat
```

**macOS / Linux:**

```bash
python3 -m venv venv
source venv/bin/activate
```

### 3. Pemasangan Pustaka Dependensi

Pastikan `venv` telah aktif (terdapat tanda `(venv)` di awal baris perintah), kemudian jalankan:

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

### 4. Konfigurasi Kredensial Lingkungan (.env)
Buat berkas .env pada direktori utama proyek, lalu isi URL Webhook Discord saluran terkait:

```bash
BAAK_WEBHOOK=[https://discord.com/api/webhooks/.../](https://discord.com/api/webhooks/.../)...
LEPKOM_WEBHOOK=[https://discord.com/api/webhooks/.../](https://discord.com/api/webhooks/.../)...
STUDENTSITE_WEBHOOK=[https://discord.com/api/webhooks/.../](https://discord.com/api/webhooks/.../)...
VCLASS_WEBHOOK=[https://discord.com/api/webhooks/.../](https://discord.com/api/webhooks/.../)...
```

### 5. Pengujian dan Menjalankan Program

Uji koneksi ke kanal Discord:

```text
python test_vclass_conn.py
```

Atau jalankan seluruh siklus pemantauan::

```text
python main.py
```

---

## Struktur Direktori

```text
ecosystem-adkesma/
│
├── .github/
│   └── workflows/
│       └── scraper_news.yml      # Definisi pipeline CI/CD GitHub Actions & cron scheduler
│
├── data/
│   └── last_updates.json         # Penyimpanan lokal riwayat pengumuman (state persistence)
│
├── scrapers/
│   ├── local_data/
│   │   └── baak_data.csv         # Cadangan dataset pengumuman BAAK
│   ├── baak.py                   # Modul ekstraksi berita BAAK Gunadarma
│   ├── lepkom.py                 # Modul ekstraksi berita LePKom Virtual Mandiri
│   ├── studentsite.py            # Modul ekstraksi berita portal Studentsite v4
│   ├── vclass.py                 # Modul ekstraksi forum pengumuman V-Class Moodle
│   └── utils.py                  # Konfigurasi WebDriver Selenium & anti-bot options
│
├── .env                          # Konfigurasi variabel lingkungan lokal (default penulis)
├── .gitignore                    # Berkas pengecualian pelacakan Git
├── Dockerfile                    # Spesifikasi container engine
├── Logo ECA.png                  # Identitas grafis proyek
├── main.py                       # Skrip orkestrator sentral pengikisan & pengiriman
├── requirements.txt              # Daftar dependensi modul Python
└── test_vclass_conn.py           # Skrip uji konektivitas endpoint webhook V-Class
```

---

## Penulis
Dikembangkan dan dikelola oleh: [Raffael Fidhera](https://instagram.com/fidhera)
