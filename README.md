# EsikatCeisa - Sistem Integrasi Kepabeanan (TPB & PLB)

**EsikatCeisa** adalah aplikasi sistem integrasi berbasis web yang menjembatani dan mengotomatisasi proses pengelolaan dokumen kepabeanan untuk Tempat Penimbunan Berikat (TPB) dan Pusat Logistik Berikat (PLB). Fokus utama sistem ini adalah integrasi **Host-to-Host (H2H)** dengan sistem CEISA (Customs-Excise Information System and Automation) milik Direktorat Jenderal Bea dan Cukai (DJBC).

Aplikasi ini ditujukan untuk mempermudah perusahaan (Eksportir / Importir / Pusat Logistik Berikat) dalam melakukan perekaman, pengeditan, serta pengiriman dokumen pabean elektronik seperti BC 2.5, BC 2.7, dan BC 3.0.

---

## 🎯 Tujuan Pembuatan
1. **Efisiensi:** Mengurangi penginputan manual berulang dengan sistem *auto-populate* dari data master referensi Bea Cukai.
2. **Akurasi Pabean:** Meminimalisir *human-error* melalui fitur kalkulasi pungutan otomatis (Bea Masuk, PPN, PPh, dll) berdasarkan kurs, tarif, dan status/fasilitas barang (Dibayar, Dibebaskan, Ditangguhkan, dll).
3. **Kepatuhan (Compliance):** Memastikan format payload (JSON) sesuai standar skema CEISA (v.0.5.29) sebelum di-*submit* via fitur pra-validasi dan generator JSON.
4. **Sentralisasi:** Menggabungkan pengelolaan berbagai form dokumen, rincian logistik (kemasan, kontainer, pengangkut), dan rincian barang (termasuk daftar bahan baku) dalam satu antarmuka modern.

---

## 💻 Tech Stack (Teknologi yang Digunakan)

### **Backend & Database**
- **Framework Utama**: [Laravel 11.x](https://laravel.com/) (menggunakan PHP 8.3+)
- **Database**: MariaDB / MySQL. 
  - *Catatan:* Proses akses data transaksi dan master CEISA menggunakan **PDO & Query Builder** secara langsung (tanpa Eloquent ORM) demi fleksibilitas kueri dan optimasi kecepatan pengolahan struktur dokumen yang masif.
- **PDF Generator**: `mpdf/mpdf` (digunakan saat mencetak draf/respon dokumen dari sistem CEISA).

### **Frontend & Styling**
- **UI & Styling**: 
  - **Blade Templates**: Dengan struktur modular *partials* (memecah UI kompleks menjadi komponen kecil).
  - **TailwindCSS (v3.4)**: *Utility-first framework* untuk antarmuka elegan, ringkas, dan responsif.
  - `@tailwindcss/forms`: Plugin untuk perapian elemen *form*.
- **JavaScript & Interaktivitas**:
  - **Vanilla JS & jQuery**: Menangani logika *client-side* (pengisian *state* otomatis antar-tab, kalkulasi nilai pajak secara *real-time*, AJAX API calls).
  - **Select2**: Untuk *dropdown* cerdas (sangat krusial untuk pencarian ribuan data referensi pelabuhan, valuta, dll).
  - **Feather Icons**: SVG icons *lightweight*.
  - **Notyf / SweetAlert2**: Untuk menampilkan pop-up interaktif & *toast notifications*.

---

## 🏗️ Struktur Arsitektur ("Procedural in Laravel")

Proyek ini mengadopsi pendekatan arsitektur spesifik: **"Procedural in Laravel"** pada modul CEISA. Hal ini berarti alur *request/routing* mengarah langsung ke eksekusi skrip `.php` fungsional di dalam direktori `app/Http/Controllers/Ceisa/` (seperti `store.php`, `update.php`, `get_detail.php`, `send.php`).

### Flow Arsitektur Singkat:
- **View Layer**: Menggunakan modular partials Blade untuk membentuk *Single-page-like form* (berpindah antar tab Header, Entitas, Barang, Pungutan tanpa *reload*).
- **Client-Side State (`bc25.js`, `bc27.js`, dll)**: Menjalankan "Business Logic" di sisi peramban pengguna—mulai dari menghitung Pabean (FOB, CIF, PDRI) hingga merakit draf *JSON Payload*.
- **API & Caching**: Data referensi yang masif ditarik dari *endpoint* internal dan dicache ke `localStorage` (sehingga *dropdown* langsung siap tanpa membebani server terus menerus).
- **Gateway Script (`send.php`)**: Berfungsi sebagai *middleware* untuk merakit *final request*, memuat token H2H, dan mengirimkannya ke API CEISA.

---

## ✨ Fitur Utama (Berdasarkan Modul BC)

### 1. Form Input Berjenjang (Tab Navigation)
Pembuatan dokumen sangat kompleks sehingga dipecah menjadi beberapa tab: **Header**, **Entitas** (Pengusaha/Importir/Pemasok/Pemilik), **Dokumen**, **Pengangkut**, **Kemasan & Kontainer**, **Transaksi**, **Barang & Bahan Baku**, **Pungutan**, dan **Pernyataan**.

### 2. Modul Barang Terintegrasi
- Input Spesifikasi Khusus (kode spesifik untuk jenis barang tertentu).
- Terhubung langsung dengan formulir khusus lampiran **Bahan Baku Impor & Lokal** menggunakan antarmuka *Offcanvas* (side-panel).

### 3. Modul Auto-Pungutan (Tarif & Pajak)
Otomatis mengkalkulasi Nilai Pabean, mendistribusikan BM, BMAD, BMTP, PPN, PPnBM, dan PPh ke dalam matriks status sesuai kode fasilitas tarif barang (Misal: 1=Dibayar, 2=Ditanggung Pemerintah, 3=Ditangguhkan, 5=Dibebaskan, 6=Tidak Dipungut, 7=Sudah Dilunasi).

### 4. Manajemen Entitas Dinamis
Setiap kode entitas (Pengusaha TPB, Pemasok, Pembeli, dll) di-mapping spesifik dengan isian yang disesuaikan (nomor identitas, NITKU, alamat, status NIB/API) lalu digabungkan saat JSON dibentuk.

### 5. Live JSON Inspector
Fitur inspeksi JSON (*Cek JSON*) untuk melihat pra-tinjau stuktur data persis yang akan dikirimkan ke Bea Cukai secara *real-time* saat sedang mengisi form.

---

## 🔄 Flowchart Aplikasi (Alur Kerja Pengguna)

```mermaid
graph TD
    A[Mulai Pembuatan/Edit Dokumen BC] --> B[Sistem Cek Cache Referensi]
    B -->|Tersedia| C[Load Form & Dropdown dari Cache]
    B -->|Tidak Ada| D[Fetch API & Simpan ke Cache]
    D --> C
    
    C --> E[Isi Tab: Header, Entitas, Transaksi, Pengangkut]
    E --> F[Isi Tab: Detail Barang]
    
    F --> G{Punya Bahan Baku?}
    G -->|Ya| H[Input via Offcanvas Bahan Baku]
    G -->|Tidak| I[Lanjutkan]
    H --> I
    
    I --> J[Auto-kalkulasi Pungutan & Pajak]
    J --> K[Lihat Rekap di Tab Pungutan]
    
    K --> L[Tombol Validasi]
    L --> M{Atribut Mandatory Terisi?}
    M -->|Belum| N[Tampilkan Notifikasi Error]
    N --> E
    M -->|Sudah| O[Generate Struktur JSON H2H]
    
    O --> P[Klik Simpan & Kirim]
    P --> Q[Execute: store.php / update.php (Save to Database)]
    Q --> R[Execute: send.php (POST ke Server CEISA)]
    R --> S[Sistem Menunggu Respons/Status CEISA]
```

---

## 🚀 Instalasi & Development

Jalankan perintah berikut untuk mengoperasikan aplikasi dalam mode *development*:

```bash
# Menjalankan Backend Laravel Server
php artisan serve

# Kompilasi Aset Frontend (Tailwind & JavaScript)
npm run dev
```

*Dokumen ini merupakan pedoman arsitektur sistem EsikatCeisa (TPB/PLB) agar para developer memahami struktur "Procedural in Laravel" dan konsep flow H2H CEISA di dalamnya.*
