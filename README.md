# Taring Naga Lab — Café & Roastery Operations Suite

Aplikasi POS + manajemen roastery all-in-one. PWA installable, multi-user dengan role,
open bill café, multi-printer (kitchen/bar), sync via Google Spreadsheet, dan pembukuan.

**Stack:** Single-file HTML + React 18 + Chart.js + ExcelJS (semua via CDN). Tanpa build step.
Data tersimpan di browser (localStorage) + opsional sync ke Google Sheet.

---

## 1. ISI PAKET

```
tn-lab/
├── index.html              ← Aplikasi utama (buka ini)
├── manifest.webmanifest    ← Konfigurasi PWA
├── sw.js                   ← Service worker (offline + auto-update)
├── apps-script.gs          ← Backend Google Apps Script (untuk sync, opsional)
├── icons/                  ← Icon PWA (PNG/SVG/ICO)
└── README.md               ← Dokumen ini
```

---

## 2. CARA DEPLOY (pilih salah satu)

App ini **static** — tidak butuh server backend. Wajib **HTTPS** agar PWA install,
Web Bluetooth printer, dan service worker berfungsi.

### Opsi A — GitHub Pages (gratis, paling mudah)
1. Buat repo GitHub baru, upload semua file (index.html, sw.js, manifest, icons/)
2. Settings → Pages → Source: `main` branch, folder `/root` → Save
3. Tunggu ~1 menit, app live di `https://username.github.io/repo-name/`

### Opsi B — Netlify / Vercel / Cloudflare Pages (gratis, drag-drop)
1. Login ke netlify.com (atau vercel/cloudflare)
2. Drag folder `tn-lab` ke area deploy
3. Selesai — dapat URL HTTPS otomatis

### Opsi C — Hosting sendiri (cPanel / Nginx / Apache)
1. Upload semua file ke folder public (mis. `public_html/tn-lab/`)
2. Pastikan domain pakai HTTPS (Let's Encrypt gratis)
3. Akses `https://domain.com/tn-lab/`

> **Testing lokal:** `python -m http.server 8000` lalu buka `http://localhost:8000`.
> Catatan: di localhost, Web Bluetooth & PWA install terbatas — pakai HTTPS untuk fitur penuh.

---

## 3. SETUP PERTAMA KALI (WAJIB — yang harus Anda lakukan manual)

### ✅ Langkah 1 — Buat akun Admin
1. Buka app pertama kali → muncul layar **Setup Pertama Kali**
2. Isi username + password (min 4 karakter) → role otomatis **Administrator**
3. Klik Sign Up → masuk dashboard

### ✅ Langkah 2 — Set identitas bisnis
1. Buka **Settings (Pengaturan)** dari menu
2. **N° 02 Business Info**: isi nama bisnis, tagline, alamat, telepon
3. **N° 05 Branding**: upload logo (opsional) + favicon
4. Klik **Simpan Settings**

### ✅ Langkah 3 — Set kategori & menu café (kalau pakai POS café)
1. Settings → **N° 07 Café POS**: tambah/hapus kategori (Coffee, Food, dll)
2. Inventory → **Products** → tambah menu/produk dengan harga & kategori
   - Untuk minuman made-to-order: kosongkan stok (= unlimited/∞)
   - Untuk barang fisik (pastry, beans): isi stok angka

### ✅ Langkah 4 — Tambah pegawai/kasir (kalau ada tim)
1. Settings → **Users → Roles & Permissions**: atur izin tiap role
2. Tab **Users → Add User**: buat akun kasir/barista dengan role yang sesuai
3. Pegawai login dengan akun masing-masing — menu otomatis terbatas sesuai role

---

## 4. SYNC MULTI-DEVICE (opsional — kalau pakai banyak HP/komputer)

Tanpa setup ini, app tetap jalan penuh tapi data **hanya di 1 device**.
Untuk sync antar device + backup otomatis ke Google Sheet:

### ✅ Setup backend (sekali saja, ~10 menit)
1. Buka https://docs.google.com/spreadsheets → buat sheet baru → copy **ID** dari URL
   (`docs.google.com/spreadsheets/d/`**INI_ID**`/edit`)
2. Buka https://script.google.com → New Project
3. Hapus kode default, **tempel seluruh isi `apps-script.gs`**
4. Ganti 2 baris di atas:
   ```js
   const SHEET_ID = 'TEMPEL_ID_SHEET_DISINI';
   const SHARED_SECRET = 'buat-password-acak-panjang-min-24-karakter';
   ```
5. Save → **Deploy → New deployment** → type: **Web app**
   - Execute as: **Me**
   - Who has access: **Anyone with the link**
   - Deploy → Authorize (login Google, izinkan akses)
6. Copy URL Web App (berakhiran `/exec`)

### ✅ Hubungkan app
1. Settings → **N° 11 Sync & Database**
2. Tempel **URL** + **Secret** yang sama
3. Klik **Test Connection** → harus "Connected"
4. Klik **Force Sync Now** (otomatis mengaktifkan sync)
5. Set interval (rekomendasi 60 detik)

> Lakukan langkah "Hubungkan app" yang sama di tiap device dengan URL+Secret yang sama.

---

## 5. INTEGRASI WEB ORDER (opsional — kalau punya website jualan)

Order dari website otomatis masuk ke POS:
1. Settings → **N° 12 Webhook** → copy **Webhook URL**
2. Di website Anda, saat checkout, kirim POST ke URL itu:
   ```js
   fetch('WEBHOOK_URL', {
     method:'POST',
     body: JSON.stringify({ order: {
       items:[{name:'Latte', price:28000, qty:2}],
       customer:'Budi', total:56000, channel:'Web'
     }})
   });
   ```
3. App polling otomatis → order masuk POS + auto-print (kalau printer terhubung)

---

## 6. PRINTER THERMAL (opsional)

### Struk customer
1. Pasang printer Bluetooth (Bixolon/Eppos/Xprinter/generic 58mm/80mm)
2. Pair di Bluetooth OS dulu
3. Settings → **N° 13 Thermal Printer** → Connect Printer → Test Print
4. Settings → **N° 08 Receipt** → atur logo, header, footer, WiFi QR

### Kitchen/Bar (multi-printer)
1. Settings → **N° 07 Café POS → Print Stations**
2. Tambah station (Kitchen, Bar) → pilih kategori item yang dicetak
3. Connect printer per station → saat order, ticket otomatis ke station relevan

> Web Bluetooth hanya jalan di **Chrome/Edge desktop & Chrome Android**.
> Safari iOS tidak support → otomatis fallback ke browser print.

---

## 7. INSTALL KE HP (PWA)

- **Android (Chrome):** menu ⋮ → "Install app" / "Add to Home Screen"
- **iOS (Safari):** Share → "Add to Home Screen" (wajib Safari)
- **Desktop (Chrome/Edge):** ikon install di address bar

---

## 8. ROLE & VISIBILITAS MENU (default)

| Menu | Admin | Owner | Pegawai |
|---|:---:|:---:|:---:|
| Dashboard | ✓ | ✓ | ✓ |
| Roast Log | ✓ | ✓ | ✓ |
| Inventory | ✓ | ✓ | ✗ |
| POS | ✓ | ✓ | ✓ |
| Pembukuan (Reports) | ✓ | ✓ | ✗ |
| Settlement | ✓ | ✓ | lihat saja |
| Users | ✓ | ✗ | ✗ |
| Settings | ✓ | ✓ | ✗ |

Semua izin **bisa diubah** di Users → Roles & Permissions. Bisa buat role baru
(kasir, supervisor, barista, dll).

---

## 9. OPERASI HARIAN

1. **Buka hari** — Settlement → Open Day (input kas awal)
2. **Jualan** — POS → pilih menu → Bayar Sekarang ATAU Simpan Bill (open tab per customer/meja)
3. **Open bill** — tab Open Bill: lihat tagihan belum bayar, tambah item, finalize
4. **Tutup hari** — Settlement → Close Day (input kas akhir). App peringatkan kalau ada open bill belum bayar
5. **Laporan** — Pembukuan → Export Excel (lengkap dengan grafik)

---

## 10. BACKUP & KEAMANAN DATA

- **Backup manual:** Settings → N° 14 Data → Export JSON (simpan rutin!)
- **Restore:** Import JSON
- Data lokal di browser — **jangan clear browser data** tanpa backup
- Kalau pakai sync, data juga aman di Google Sheet
- Customer tidak aktif > 2 bulan (configurable) terhapus otomatis

---

## 11. TROUBLESHOOTING

| Masalah | Solusi |
|---|---|
| Layar putih/crash | Tombol "Reload" atau "Reset Data Lokal" di error screen |
| Sync "unauthorized" | Secret di app ≠ Apps Script. Samakan persis |
| Sync "Unexpected token <" | URL salah — pakai URL `/exec`, bukan URL editor |
| Printer tak muncul | Wajib HTTPS + Chrome/Edge. Pair di OS dulu |
| PWA tak bisa install iOS | Wajib Safari + HTTPS |
| "Penyimpanan penuh" | Export backup, lalu kurangi data lama / hapus logo besar |
| Versi lama muncul | Hard refresh (Ctrl+Shift+R), atau tutup semua tab & buka lagi |

---

## 12. WHITE-LABEL (untuk dijual ke bisnis lain)

Setiap klien tinggal:
1. Settings → Branding: ganti nama app, logo, favicon
2. Settings → Business Info: ganti identitas
3. Pakai Google account & secret mereka sendiri (data terpisah & milik mereka)

Tidak perlu edit kode. Biaya operasional **nol** (hosting gratis + Apps Script gratis).

---

> "Kopi yang baik punya gigi." — Taring Naga
> v1.7 · Production Build
