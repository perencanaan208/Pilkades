# Panduan: Hosting "Pilkades Baleharjo 2027" via Vercel (mode iframe)

## Cara kerjanya

- **Google Apps Script (GAS)** tetap jadi **satu-satunya tempat** aplikasi
  hidup: UI (`index.html`), backend (`KODE.gs`), dan datanya (Google Sheet).
  Semua fitur, tampilan, dan logika diedit di sini seperti biasa.
- **Vercel** hanya menampung **satu file `index.html` kecil** yang isinya
  sebuah `<iframe>` mengarah ke URL Web App GAS Anda. Vercel tidak
  menyimpan salinan aplikasi, jadi tidak akan pernah "ketinggalan versi".
- Karena `doGet()` di `KODE.gs` sudah memakai
  `setXFrameOptionsMode(HtmlService.XFrameOptionsMode.ALLOWALL)`, GAS
  mengizinkan halamannya ditampilkan di dalam iframe dari domain manapun
  — termasuk domain Vercel Anda.

Alur update ke depan: **edit di Apps Script → Deploy versi baru → selesai.**
Tidak perlu sentuh repo GitHub/Vercel lagi, kecuali URL Web App-nya berubah.

---

## Bagian A — Pastikan Web App GAS sudah aktif

1. Buka project Apps Script Anda (berisi `KODE.gs` dan `index.html`).
2. Klik **Deploy → New deployment** (atau **Manage deployments** jika sudah
   pernah deploy sebelumnya, lalu buat versi baru).
3. Pilih tipe **Web app**, dengan pengaturan:
   - **Execute as:** Me
   - **Who has access:** Anyone
4. Klik **Deploy**, lalu salin URL yang diberikan. Pastikan URL berakhiran
   `/exec` (bukan `/dev`), contoh:
   ```
   https://script.google.com/macros/s/AKfycbxXXXXXXXXXXXX/exec
   ```

Simpan URL ini — dipakai di Bagian C.

---

## Bagian B — Push folder ini ke GitHub

Folder ini berisi 2 file:
- `index.html` — halaman wrapper iframe
- `PANDUAN.md` — file ini

Langkah dari terminal (di folder ini):

```bash
git init
git add index.html PANDUAN.md
git commit -m "Setup halaman iframe Pilkades Baleharjo"
git branch -M main
git remote add origin https://github.com/USERNAME/NAMA-REPO.git
git push -u origin main
```

Ganti `USERNAME/NAMA-REPO` dengan repo GitHub Anda. Jika repo belum ada,
buat dulu lewat github.com → **New repository** (boleh kosong, tanpa
README) sebelum menjalankan `git push`.

---

## Bagian C — Isi URL GAS ke `index.html`

Sebelum push (atau setelahnya, lalu commit ulang), buka `index.html` dan
cari baris ini:

```js
const GAS_APP_URL = "GANTI_DENGAN_URL_WEB_APP_GAS_ANDA";
```

Ganti dengan URL `/exec` dari Bagian A, misalnya:

```js
const GAS_APP_URL = "https://script.google.com/macros/s/AKfycbxXXXXXXXXXXXX/exec";
```

Simpan, lalu commit & push perubahan:

```bash
git add index.html
git commit -m "Set URL Web App GAS"
git push
```

> Tips: ada juga mode override sementara tanpa edit file — buka
> `https://domain-vercel-anda.vercel.app/?src=URL_GAS_ANDA` untuk uji coba
> URL lain tanpa mengubah kode.

---

## Bagian D — Deploy ke Vercel

1. Buka [vercel.com](https://vercel.com) → login (bisa pakai akun GitHub).
2. Klik **Add New → Project**.
3. Pilih repo GitHub yang tadi Anda push.
4. Pada pengaturan build:
   - **Framework Preset:** pilih **Other** (karena ini file statis, tanpa
     proses build).
   - **Build Command:** kosongkan.
   - **Output Directory:** kosongkan / biarkan default (root).
5. Klik **Deploy**. Setelah selesai, Vercel memberi URL seperti
   `https://nama-repo.vercel.app`.
6. Buka URL tersebut — aplikasi Pilkades akan tampil di dalamnya lewat
   iframe.

Opsional: di menu **Settings → Domains** pada project Vercel, Anda bisa
menambahkan domain kustom (mis. `pilkades-baleharjo.desa.id`).

---

## Bagian E — Alur kerja setelah ini

- **Ubah fitur / tampilan aplikasi** → edit `index.html` dan `KODE.gs` di
  Apps Script seperti biasa, lalu **Deploy → Manage deployments → edit
  (ikon pensil) → New version → Deploy**. Perubahan otomatis muncul di
  Vercel karena hanya memuat ulang iframe — **tidak perlu push apa pun**.
- **Ganti URL Web App GAS** (misalnya redeploy dari awal dan dapat URL
  baru) → update `GAS_APP_URL` di `index.html`, commit, push ulang ke
  GitHub. Vercel otomatis redeploy setiap kali ada push baru ke `main`.

---

## Troubleshooting

| Gejala | Kemungkinan penyebab & solusi |
|---|---|
| Layar putih / loading terus lalu muncul pesan error | URL GAS salah, belum diganti, atau memakai `/dev` bukan `/exec`. Periksa kembali Bagian A & C. |
| Muncul pesan "refused to connect" di console browser | Pastikan `setXFrameOptionsMode(HtmlService.XFrameOptionsMode.ALLOWALL)` masih ada di `doGet()` pada `KODE.gs`, dan deployment memakai akses **Anyone**. |
| Aplikasi tampil tapi fitur login/simpan data gagal | Ini murni masalah di sisi GAS (Sheet, permission, quota), bukan di Vercel — cek log di Apps Script Editor (**Executions**). |
| Perubahan di Apps Script tidak muncul | Pastikan Anda membuat **versi deployment baru** (bukan hanya menyimpan draf), karena URL `/exec` selalu menunjuk ke versi yang sedang di-deploy. |
