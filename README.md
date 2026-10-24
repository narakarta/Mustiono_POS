# MUSTIONO COMPUTER — POS Kasir & Servis

Aplikasi kasir (POS) & servis dengan arsitektur:

- **Tampilan (HTML/JS)** → di-host gratis di **GitHub Pages**
- **Database & backend** → **Google Sheets + Google Apps Script** (API JSON)

---

## Isi Folder

| File | Fungsi |
|---|---|
| `index.html` | Aplikasi utama — **upload ke GitHub** |
| `chart.umd.min.js` | Grafik dashboard (offline, ada cadangan CDN) |
| `logo.png` | Logo tampilan |
| `Code.gs` | Backend — **tempel ke editor Apps Script** |
| `appsscript.json` | Manifest Apps Script |

> `html5-qrcode.min.js` **tidak dipakai lagi** (fitur scan kamera sudah dihapus). Boleh dihapus dari repo GitHub.

---

## BAGIAN 1 — Pasang Backend (Google Sheets + Apps Script)

1. Buka [Google Drive](https://drive.google.com) → **New → Google Sheets**, beri nama mis. `MUSTIONO POS Database`.
   *(Sudah punya spreadsheet aplikasi lama? Pakai itu saja — datanya tetap terbaca.)*
2. Menu **Extensions → Apps Script**.
3. Hapus isi `Code.gs` bawaan, lalu **tempel seluruh isi `Code.gs`** dari folder ini.
4. Buka **Project Settings** (ikon gerigi ⚙) → centang **"Show 'appsscript.json' manifest file in editor"** → kembali ke **Editor** → buka `appsscript.json` → ganti isinya dengan file `appsscript.json` dari folder ini.
5. Simpan (**Ctrl+S**). Di toolbar pilih fungsi **`initializeDatabase`** → **Run**.
   - Muncul izin akses: **Review permissions → pilih akun → Advanced → Go to … (unsafe) → Allow**.
   - Fungsi ini otomatis membuat semua sheet, akun default **superadmin / admin123**, dan 2 cabang contoh.
6. **Deploy → New deployment** → pilih tipe **Web app**:
   - *Execute as*: **Me**
   - *Who has access*: **Anyone** ← **WAJIB** (kalau tidak, aplikasi GitHub tidak bisa akses)
   - Klik **Deploy** → **salin URL yang berakhiran `/exec`**

> **Update backend nanti:** setelah mengubah `Code.gs`, jangan deploy baru dari nol — buka **Deploy → Manage deployments → ikon pensil → Version: New version → Deploy**. URL `/exec` tetap sama.

---

## BAGIAN 2 — Upload Tampilan ke GitHub Pages

1. Buat akun [GitHub](https://github.com) (kalau belum) → **New repository**, mis. `mustiono-pos` (boleh Public).
2. Di halaman repo: **Add file → Upload files** → upload ke-3 file ini:
   - `index.html`
   - `chart.umd.min.js`
   - `logo.png`
3. **Commit changes**.
4. Masuk **Settings → Pages** → *Source*: **Deploy from a branch** → *Branch*: **main** + **/ (root)** → **Save**.
5. Tunggu 1–2 menit. Situs jadi di:
   `https://<username-github>.github.io/<nama-repo>/`

---

## BAGIAN 3 — Menghubungkan Aplikasi dengan Server

1. Buka situs GitHub Pages Anda.
2. Di halaman login, klik tulisan kecil **"Server: …"** di bawah tombol **MASUK**.
3. **Tempel URL `/exec`** dari Bagian 1 → OK.
4. Login: **superadmin / admin123** → segera ganti password di menu **Setting Akun**.

URL server tersimpan di perangkat itu saja (localStorage). Kalau ingin URL permanen untuk semua pengguna, isi variabel `DEFAULT_API_URL` di baris atas file `index.html`.

---

## Tips & Catatan

- **Pesan "Tidak bisa menghubungi server"** → biasanya akses Web App belum diset **Anyone**, atau URL bukan yang berakhiran `/exec`.
- **Fitur scan kamera sudah dihapus** → tombol 📷 Scan, kamera live, dan scan dari foto tidak ada lagi, begitu juga kolom input barcode di halaman kasir. Halaman POS hanya memakai kolom **cari nama produk**, filter kategori, dan filter jenis. (Barcode/SKU tetap tersimpan di data barang dan muncul di tabel Input Barang.)
- **POS 15 item per halaman** → grid produk menampilkan **15 produk per halaman** dengan tombol **‹ Sebelumnya / Next ›** dan info "Halaman 1 / 3". Berganti filter atau mencari produk otomatis kembali ke halaman 1.
- **Printer** → mode *dialog print browser* mendukung semua printer; mode Bluetooth/Serial (thermal) perlu Chrome di HP/Laptop. Pengaturan **metode printer & ukuran kertas sekarang menyatu di menu 🧾 Setting Nota** (global, bukan per cabang).
- **Printer Bluetooth (mode langsung)** → perangkat terakhir **disimpan otomatis**, cetak berikutnya langsung terkirim tanpa pilih ulang. Ganti perangkat lewat tombol **"Ganti printer"** di jendela cetak.
- **Hasil cetak Bluetooth beda antara desktop & HP?** Tiga penyebabnya, semua sudah ditangani:
  1. **Ukuran paket data (MTU).** HP Android sering pakai MTU kecil (20 byte) sedangkan desktop bisa jauh lebih besar. Aplikasi sekarang memakai *write-with-response* (pecah otomatis oleh sistem BLE) dengan chunk 100 byte, dan hanya jatuh ke chunk 20 byte bila printer tidak mendukung mode itu — jadi nota tidak lagi terpotong/acak di HP.
  2. **Karakter non-ASCII.** Printer thermal hanya paham ASCII (CP437). Semua teks nota sekarang dinormalkan (`—` → `-`, spasi khusus → spasi biasa, dll) dan tanggal/angka diformat manual tanpa `toLocale*`, sehingga hasilnya identik di semua perangkat.
  3. **Popup cetak diblokir HP.** Bila popup diblokir, aplikasi sekarang menampilkan panduan mengizinkan popup (lengkap dengan tombol *Sudah Diizinkan — Coba Lagi* dan *Cetak Lewat Dialog Browser*) — bukan lagi pindah diam-diam ke dialog print browser. Inilah penyebab paling umum hasil cetak "beda".
  Catatan: **mode langsung memang teks polos** (tanpa logo, lebar tetap 32 karakter kertas 58mm / 48 karakter kertas 80mm), jadi wajar berbeda dengan hasil **Dialog Print Browser** yang mencetak nota HTML lengkap.
- **🔓 Izin popup untuk cetak mode langsung** → mode Bluetooth/Serial membuka **jendela cetak terpisah**, dan banyak browser (terutama HP) memblokir popup secara bawaan. Bila diblokir, aplikasi menampilkan panduan mengizinkan popup **sesuai browser yang dipakai** (Chrome/Edge desktop, Chrome Android, Samsung Internet, plus catatan khusus iPhone/iPad) dengan tombol **🔓 Sudah Diizinkan — Coba Lagi** dan **🖨️ Cetak Lewat Dialog Browser**. Untuk memastikan izinnya sebelum melayani pembeli: menu **🧾 Setting Nota** → **🔓 Cek Izin Popup** (status izin juga ditampilkan di halaman itu). Metode **Dialog Print Browser tidak butuh izin popup sama sekali**.
- **Keamanan URL server** → di halaman login URL ditampilkan **tersensor** (contoh: `…/s/AKfycbTE••••••••qrst/exec`). Saat mengganti koneksi, kotak input selalu **kosong** — URL lama tidak pernah ditampilkan; kosongkan lalu OK untuk menghapus koneksi.
- **Laporan & Dashboard** → pemfilteran memakai **hari kalender zona waktu toko** (`Asia/Jakarta`), bukan jam UTC. Transaksi yang dibuat jam **00:00–06:59 WIB** tetap masuk ke hari yang benar dan tidak hilang dari Laporan/Dashboard.
- **"🏬 Semua Cabang"** → khusus **superadmin**; dipakai untuk melihat rekap gabungan. Transaksi kasir tetap wajib punya 1 cabang aktif (otomatis dipilih bila kosong).
- **Hak akses menu (baru)** → Super Admin bisa mengatur menu apa saja yang boleh dibuka **Kasir** dan **Kepala Toko** di menu **Setting Akun → 🔐 Hak Akses Menu**. Pilihan yang bisa dicentang: Dashboard, POS Kasir, Input Barang, Laporan. Menu **Setting Nota** & **Setting Akun** selalu khusus Super Admin (berisi pengaturan sistem). Bawaan: `kasir` = POS Kasir saja, `kepala` = Dashboard + Input Barang + Laporan. Bila semua centang dimatikan untuk suatu role, akun role itu akan melihat halaman "🚫 Belum Ada Akses Menu" dan tidak bisa membuka halaman apa pun. Perubahan berlaku setelah pengguna **memuat ulang halaman / login ulang**. Tombol **↩️ Kembalikan Bawaan** mengembalikan pengaturan awal.
- **🧾 Setting Nota & Printer (global)** → desain nota (judul, header/footer, logo, awalan nomor) **dan** pengaturan printer (metode Dialog Browser / Bluetooth / Serial + ukuran kertas 80mm/58mm) ada dalam satu menu **Setting Nota**. Semuanya **global** (berlaku semua cabang) dan hanya bisa diubah **Super Admin**. Menu "Setting Printer" per cabang sudah dihapus.
- **Update tampilan** → upload ulang `index.html` ke GitHub (Commit) — tanpa perlu deploy ulang Apps Script.
- **⚠ Update versi 24 Okt 2026 mengubah backend** → wajib lakukan **dua-duanya**: (1) upload ulang `index.html` ke GitHub, (2) tempel ulang `Code.gs` di Apps Script lalu **Deploy → Manage deployments → ikon pensil → Version: New version → Deploy**. Perubahan: fitur scan kamera & kolom barcode dihapus, POS 15 item per halaman, Setting Printer digabung ke Setting Nota, dan **Hak Akses Menu** baru di Setting Akun. Bila hanya `index.html` yang di-update, halaman **Hak Akses Menu** akan gagal menyimpan (server lama masih menolak simpan setting dari selain Super Admin / masih memakai setting per cabang).
- **Ganti password default & tambah kasir** lewat menu **Setting Akun**.
