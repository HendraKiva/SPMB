# SPMB SDI Abu Hurairah Lahat — Panduan Deploy (Gratis)

Aplikasi ini terdiri dari:
- **Backend** (Node.js + Express) — menyimpan data pendaftar & konten halaman
- **Database** (MongoDB Atlas — gratis selamanya, 512 MB)
- **Hosting** (Render.com — gratis)

Ikuti langkah di bawah secara berurutan. Total waktu ±20–30 menit untuk yang pertama kali.

---

## BAGIAN 1 — Siapkan Database (MongoDB Atlas)

1. Buka https://www.mongodb.com/cloud/atlas/register dan daftar akun gratis (bisa pakai akun Google).
2. Setelah masuk, pilih **"Build a Database"** → pilih paket **M0 (Free)**.
3. Pilih provider & region bebas (misalnya AWS - Singapore agar lebih dekat), klik **Create**.
4. Saat diminta membuat **Database User**:
   - Isi username, misalnya `spmb-admin`
   - Klik **Autogenerate Secure Password**, lalu **salin dan simpan password ini** (akan dipakai nanti).
5. Pada bagian **Network Access / IP Access List**, klik **Add IP Address** → pilih **Allow Access from Anywhere** (`0.0.0.0/0`). Ini diperlukan agar Render bisa terhubung ke database Anda.
6. Setelah cluster selesai dibuat (biasanya 1–3 menit), klik **Connect** → pilih **Drivers** → pilih **Node.js**.
7. Salin **Connection String** yang muncul, bentuknya seperti:
   ```
   mongodb+srv://spmb-admin:<password>@cluster0.xxxxx.mongodb.net/?retryWrites=true&w=majority
   ```
8. Ganti `<password>` dengan password yang Anda simpan di langkah 4. Tambahkan nama database setelah `.net/`, misalnya:
   ```
   mongodb+srv://spmb-admin:PASSWORD_ANDA@cluster0.xxxxx.mongodb.net/spmb-abu-hurairah?retryWrites=true&w=majority
   ```
   Simpan string ini — ini adalah nilai **MONGO_URI** Anda.

---

## BAGIAN 2 — Unggah Proyek ke GitHub

1. Buat akun di https://github.com jika belum punya.
2. Klik **New repository**, beri nama misalnya `spmb-abu-hurairah`, pilih **Public** atau **Private**, lalu **Create repository**.
3. Unggah seluruh isi folder proyek ini (kecuali folder `node_modules` jika ada) ke repository tersebut:
   - Cara termudah: di halaman repository GitHub, klik **"uploading an existing file"**, lalu seret semua file & folder proyek ke sana, lalu klik **Commit changes**.
   - (Alternatif untuk yang terbiasa command line: `git init`, `git add .`, `git commit -m "init"`, `git remote add origin <url-repo>`, `git push -u origin main`)

---

## BAGIAN 3 — Deploy ke Render (Backend + Website jadi satu link)

1. Buka https://render.com dan daftar (bisa langsung pakai akun GitHub Anda agar terhubung otomatis).
2. Klik **New +** → **Web Service**.
3. Pilih repository `spmb-abu-hurairah` yang tadi diunggah.
4. Isi konfigurasi:
   - **Name**: `spmb-abu-hurairah` (akan menjadi bagian dari link Anda)
   - **Region**: Singapore (atau terdekat)
   - **Build Command**: `npm install`
   - **Start Command**: `npm start`
   - **Instance Type**: **Free**
5. Scroll ke bagian **Environment Variables**, klik **Add Environment Variable**, lalu tambahkan tiga berikut satu per satu:

   | Key | Value |
   |---|---|
   | `MONGO_URI` | connection string dari Bagian 1 |
   | `ADMIN_PASSWORD` | kata sandi admin pilihan Anda sendiri (gunakan yang kuat) |
   | `JWT_SECRET` | string acak bebas, contoh: `spmb-abu-hurairah-rahasia-2027-xyz` |

6. Klik **Create Web Service**. Render akan mulai build & deploy otomatis — tunggu sampai statusnya **Live** (biasanya 2–5 menit).
7. Setelah selesai, Render akan memberi Anda link publik seperti:
   ```
   https://spmb-abu-hurairah.onrender.com
   ```
   **Inilah link yang bisa Bapak/Ibu bagikan ke media sosial, WhatsApp, brosur, dll.**

---

## BAGIAN 4 — Uji Coba

1. Buka link yang diberikan Render di browser.
2. Coba isi formulir pendaftaran — pastikan muncul pesan "Pendaftaran berhasil disimpan".
3. Klik tombol **Admin** di pojok kanan atas, masukkan `ADMIN_PASSWORD` yang tadi Anda buat.
4. Cek tab **Data Pendaftar** — pastikan data uji coba tadi muncul di sana. Anda bisa menghapusnya nanti langsung dari database Atlas jika perlu.
5. Coba ubah Visi/Misi atau Prestasi di panel Admin, klik **Simpan Perubahan**, lalu refresh halaman untuk memastikan perubahan tersimpan.

---

## Catatan Penting

- **Render paket gratis akan "tidur"** setelah 15 menit tanpa ada pengunjung. Saat ada pengunjung pertama setelah tidur, halaman akan terasa lambat sekitar 30–50 detik sebelum aktif kembali — ini normal untuk paket gratis, bukan error.
- **Kata sandi admin** (`ADMIN_PASSWORD`) sebaiknya diganti secara berkala. Untuk mengubahnya, buka Render → Settings → Environment → ubah nilainya → simpan (Render akan otomatis redeploy).
- **Domain sendiri (opsional)**: jika sekolah punya domain sendiri (misalnya `spmb.sdiabuhurairah.sch.id`), domain tersebut dapat dihubungkan lewat menu **Settings → Custom Domain** di Render.
- **Backup data**: sesekali unduh data pendaftar lewat tombol **Unduh CSV** di panel Admin sebagai cadangan.
- Jika ingin upgrade agar tidak "tidur" dan lebih cepat, Render menyediakan paket berbayar mulai sekitar USD 7/bulan — opsional, tidak wajib.

---

## Struktur Proyek

```
spmb-app/
├── package.json
├── .env.example        # contoh variabel lingkungan (untuk lokal)
├── server/
│   ├── index.js         # entry point Express
│   ├── db.js             # koneksi MongoDB
│   ├── defaultContent.js # data awal visi/misi/jalur
│   ├── models/            # skema database
│   ├── routes/             # endpoint API
│   └── middleware/          # proteksi admin (JWT)
└── public/
    ├── index.html         # halaman utama
    ├── styles.css
    └── app.js               # logika frontend (fetch API, form, admin panel)
```

## Menjalankan di Komputer Sendiri (opsional, untuk uji coba lokal)

```bash
npm install
cp .env.example .env
# isi .env dengan MONGO_URI, ADMIN_PASSWORD, JWT_SECRET
npm start
# buka http://localhost:3000
```
