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

## Bagian F — Ikon aplikasi (agar tidak muncul logo Chrome bawaan)

Folder ini berisi ikon logo "E-Pilkades Baleharjo", dan **semuanya sengaja
ditaruh rata di satu folder yang sama dengan `index.html` — tanpa
subfolder** — supaya aman kalau Anda upload lewat GitHub web (upload file
satu-satu lewat browser bisa "meratakan" struktur folder, jadi lebih aman
kalau memang tidak ada folder sejak awal):

- `favicon-16.png`
- `favicon-32.png`
- `icon-192.png`
- `icon-512.png`
- `apple-touch-icon.png`
- `manifest.json`

**Penting:** upload/`git add` **ke-6 file di atas + `index.html`, semuanya
rata di root repo** (bukan di dalam folder apa pun). Nama file harus persis
sama seperti di atas — kalau GitHub menambahkan akhiran seperti
`(1)` karena dianggap duplikat, hapus file lama lebih dulu lalu upload
ulang dengan nama yang benar.

Cara push lewat `git` (disarankan, lebih aman daripada upload manual satu-satu):
```bash
git add index.html manifest.json favicon-16.png favicon-32.png icon-192.png icon-512.png apple-touch-icon.png PANDUAN.md
git commit -m "Tambah ikon aplikasi"
git push
```

Efeknya kalau sudah benar:
- **Tab browser** menampilkan ikon logo, bukan ikon default.
- **"Add to Home Screen" / "Install app"** di HP (Android & iPhone) dan
  "Install" di Chrome desktop akan memakai logo ini sebagai ikon aplikasi,
  bukan logo Chrome / huruf inisial generik.

Setelah push, di Vercel tunggu deployment baru selesai (cek tab
**Deployments**), lalu buka situsnya dengan **hard refresh**
(`Ctrl+Shift+R`) — Chrome sering menyimpan cache manifest lama.

Catatan: file `index.html` di project **Google Apps Script** Anda (yang
dibuka lewat iframe) juga sudah saya sisipkan ikon yang sama secara inline
(base64), jadi kalau ada orang yang membuka **langsung** URL `.../exec`
GAS tanpa lewat Vercel, ikonnya tetap konsisten — file itu terpisah, tidak
perlu ikut diedit di sini.

---

## Troubleshooting

| Gejala | Kemungkinan penyebab & solusi |
|---|---|
| Layar putih / loading terus lalu muncul pesan error | URL GAS salah, belum diganti, atau memakai `/dev` bukan `/exec`. Periksa kembali Bagian A & C. |
| Muncul pesan "refused to connect" di console browser | Pastikan `setXFrameOptionsMode(HtmlService.XFrameOptionsMode.ALLOWALL)` masih ada di `doGet()` pada `KODE.gs`, dan deployment memakai akses **Anyone**. |
| Aplikasi tampil tapi fitur login/simpan data gagal | Ini murni masalah di sisi GAS (Sheet, permission, quota), bukan di Vercel — cek log di Apps Script Editor (**Executions**). |
| Perubahan di Apps Script tidak muncul | Pastikan Anda membuat **versi deployment baru** (bukan hanya menyimpan draf), karena URL `/exec` selalu menunjuk ke versi yang sedang di-deploy. |
| Ikon belum muncul / masih huruf inisial saat "Install app" | Cek satu per satu: (1) semua file ikon ada di **root repo**, bukan di dalam folder `icons/`; (2) nama file persis `icon-192.png`, `icon-512.png`, dst, tanpa `(1)`; (3) `manifest.json` menunjuk ke nama file tanpa awalan folder (`"icon-192.png"`, bukan `"icons/icon-192.png"`); (4) sudah hard refresh (`Ctrl+Shift+R`) setelah deployment Vercel terbaru selesai. |
