# OGDC PPA CamScanner Pro (PWA Edition)

Aplikasi Web Scanner Dokumen Digital berstandar **CamScanner** yang terintegrasi langsung dengan **Google Apps Script**, **Google Drive**, dan **Google Sheets**. Dilengkapi dengan kemampuan **Progressive Web App (PWA)** sehingga dapat langsung diinstal di layar utama perangkat Android, iPhone/iPad, Windows, dan MacOS.

![OGDC CamScanner](https://img.icons8.com/fluency/192/scanner.png)

---

## 🌟 Fitur Utama

- **PWA (Progressive Web App) Installable**:
  - Tombol **"Pasang Aplikasi"** langsung di bilah navigasi dan pop-up banner otomatis.
  - Berjalan dalam mode *standalone* layar penuh tanpa bilah URL browser, persis seperti aplikasi native Android/iOS.
  - Dilengkapi panduan khusus untuk pengguna iOS Safari (*Add to Home Screen*).
  - Caching aset melalui Service Worker (`sw.js`) dan Web App Manifest (`manifest.json`).

- **Otsu Auto-Crop & Convex Hull Edge Detection**:
  - Deteksi 4 sudut fisik lembar kertas secara otomatis (tidak memotong acak atau mengambil latar meja).
  - Menghilangkan gangguan bayangan tangan, jari penahan, dan garis tabel internal.
  - Dilengkapi fitur *Automatic Inset* (3.5%) untuk mengunci sudut persis di atas kertas putih.

- **Post-Warp Auto Desk Stripper**:
  - Otomatis memindai 4 tepi terluar gambar hasil warp dan memangkas sisa meja kayu / mousepad hitam yang tersisa.
  - Tombol manual *"Kikis Sisa Meja"* (4.5% sekali klik) di layar filter.

- **Magic Color Filter & Anti-Fade Engine**:
  - *Morphological Background Illumination Division*: Menghapus bayangan tangan, bayangan ponsel, dan gradasi pencahayaan ruangan secara merata.
  - *Anti-Fade Curve*: Memutihkan kertas menjadi putih murni (`#ffffff`), sementara teks judul, tulisan tangan, garis tabel, dan nomor BA tetap hitam pekat dan tajam (tidak pudar).
  - *Color Seal & Stamp Preservation*: Tinta stempel merah dan tanda tangan pulpen biru tetap cerah dan tajam dengan saturasi yang diperkuat.

- **Form Input Transaksi Bersih (Clean Blank)**:
  - Seluruh kolom form (Tanggal, Shift, Jenis Transaksi, Vendor, Berkas, Nomer BA) hadir dalam keadaan **kosong polos** saat halaman dibuka.
  - Dilengkapi tombol cepat **Reset Form** (<i class="fa-solid fa-eraser"></i>).
  - `autocomplete="off"` mencegah browser mengisi data lama secara otomatis.

---

## 📲 Cara Memasang sebagai PWA

### 🤖 Android (Google Chrome / Samsung Internet)
1. Buka tautan Web App di browser ponsel.
2. Banner **"Pasang Aplikasi PWA"** akan muncul di bagian bawah, atau tekan tombol **"PASANG"** di pojok kanan atas.
3. Tekan **"Pasang" / "Install"**.
4. Ikon **OGDC Scanner** akan muncul di layar utama (*Home Screen*) HP Anda.

### 🍎 iPhone / iPad (Safari)
1. Buka tautan Web App di Safari.
2. Tekan tombol **Bagikan (Share)** (ikon kotak dengan panah ke atas) di bilah bawah.
3. Gulir ke bawah dan pilih **"Tambah ke Layar Utama" (Add to Home Screen)**.
4. Tekan **Tambah (Add)** di pojok kanan atas.

### 💻 Windows / MacOS (Chrome / Edge)
1. Buka Web App di Google Chrome atau Microsoft Edge.
2. Klik ikon instalasi (<i class="fa-solid fa-download"></i>) di sebelah kanan *address bar* browser.
3. Klik **Install**. Aplikasi akan terbuka di jendela terpisah tanpa frame browser.

---

## 🛠️ Struktur File Proyek

```text
├── Index.html          # Antarmuka Pengguna (UI), Logika Scanner & PWA Engine
├── Code.js             # Backend Google Apps Script (Drive & Sheets Handler)
├── manifest.json       # Web App Manifest untuk PWA
├── sw.js               # Service Worker untuk Caching & PWA
├── appsscript.json     # Manifest Konfigurasi Google Apps Script
├── .clasp.json         # Konfigurasi Google Clasp CLI
└── README.md           # Dokumentasi Proyek
```

---

## 🚀 Deployment ke Google Apps Script via Clasp

Proyek ini telah dikonfigurasi untuk deployment instan menggunakan `@google/clasp`:

1. **Login Clasp**:
   ```bash
   npx @google/clasp login
   ```
2. **Push Perubahan**:
   ```bash
   npx @google/clasp push --force
   ```
3. **Deploy Versi Terbaru**:
   ```bash
   npx @google/clasp deploy -i <DEPLOYMENT_ID> -d "Deskripsi Update"
   ```

---

## 📄 Lisensi
Hak Cipta © 2026 PT Putra Perkasa Abadi (PPA) - Unit OGDC.
Seluruh hak cipta dilindungi undang-undang.
