# Panduan Menjalankan di Lokal

Ikuti langkah-langkah ini untuk menjalankan project Halume di laptop kamu sendiri.

---

## Prasyarat

Pastikan sudah terinstal di laptop:

| Software | Versi Minimum | Cek dengan |
|---------|--------------|-----------|
| Node.js | 18 atau lebih | `node --version` |
| npm | 9 atau lebih | `npm --version` |
| Git | Terbaru | `git --version` |

---

## Langkah 1 — Clone Repository

Buka terminal, lalu jalankan:

```bash
git clone https://github.com/Halume-Co/halume.git
cd halume
```

---

## Langkah 2 — Install Dependencies

```bash
npm install
```

Perintah ini akan mendownload semua library yang dibutuhkan project (sekitar 200+ package). Tunggu sampai selesai.

---

## Langkah 3 — Setup Environment Variables

Project ini membutuhkan koneksi ke Supabase. Konfigurasinya disimpan di file `.env.local` yang **tidak boleh di-upload ke GitHub** (sudah ada di `.gitignore`).

**3a.** Buat file `.env.local` di root project:

```bash
cp .env.local.example .env.local
```

**3b.** Buka Supabase Dashboard → pilih project Halume → **Settings → API**

**3c.** Copy dua nilai berikut ke file `.env.local`:

```
NEXT_PUBLIC_SUPABASE_URL=https://xxxxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbGci...
```

Ganti nilai setelah `=` dengan nilai asli dari Supabase Dashboard.

---

## Langkah 4 — Jalankan Development Server

```bash
npm run dev
```

Buka browser dan akses: **http://localhost:3000**

Website Halume akan berjalan di laptop kamu dan terhubung ke database Supabase yang sama dengan versi live.

---

## Langkah 5 — Membuat Akun Admin (Opsional)

Akun yang baru dibuat otomatis menjadi `customer`. Untuk membuat akun admin:

1. Daftar akun baru lewat `/register`
2. Buka Supabase Dashboard → **SQL Editor**
3. Jalankan query berikut (ganti email sesuai akun yang ingin dijadikan admin):

```sql
UPDATE profiles
SET role = 'admin'
WHERE email = 'emailkamu@contoh.com';
```

4. Logout dan login ulang di aplikasi
5. Tombol "Admin" akan muncul di navbar

---

## Perintah yang Tersedia

| Perintah | Fungsi |
|---------|--------|
| `npm run dev` | Jalankan server development (untuk coding) |
| `npm run build` | Build untuk production |
| `npm run start` | Jalankan versi production (setelah build) |
| `npm run lint` | Cek error di kode |

---

## Masalah Umum

### "Cannot find module" saat `npm run dev`
Jalankan ulang `npm install` lalu coba lagi.

### Produk tidak muncul / halaman blank
Cek file `.env.local` — pastikan URL dan anon key Supabase sudah benar dan tidak ada spasi di awal/akhir.

### Error "Email not confirmed" saat login
Masuk ke Supabase Dashboard → **Authentication → Providers → Email** → matikan opsi "Confirm email". Ini untuk kemudahan development.

### Port 3000 sudah dipakai
Tambahkan port di perintah: `npm run dev -- --port 3001`

---

## Deployment ke Vercel

Project ini sudah terdeploy di [halume.vercel.app](https://halume.vercel.app). Jika ingin deploy ulang:

1. Push ke branch `main` di GitHub
2. Vercel otomatis mendeteksi perubahan dan melakukan deploy baru

Pastikan environment variables (`NEXT_PUBLIC_SUPABASE_URL` dan `NEXT_PUBLIC_SUPABASE_ANON_KEY`) sudah diset di **Vercel Dashboard → Settings → Environment Variables**.
