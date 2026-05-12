# Arsitektur Sistem

## Gambaran Besar

Halume adalah aplikasi **full-stack** — artinya satu project ini mengurus tampilan (frontend) sekaligus logika bisnis dan data (backend). Ini dimungkinkan oleh Next.js yang bisa menjalankan kode di sisi server maupun browser.

```
Browser Pengguna
      │
      ▼
┌─────────────────────┐
│   Next.js (Vercel)  │  ← Frontend + Backend dalam satu project
│                     │
│  - Halaman (React)  │
│  - Middleware       │
│  - Server Actions   │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│      Supabase       │  ← Database + Autentikasi
│                     │
│  - PostgreSQL DB    │
│  - Auth (login)     │
│  - Row Level Sec.   │
└─────────────────────┘
```

---

## Tech Stack Dijelaskan

### Next.js — Framework Utama
Next.js adalah framework berbasis React. Bedanya dengan React biasa, Next.js bisa menjalankan kode di **server** (bukan hanya di browser). Ini berguna untuk keamanan dan performa.

Di project ini Next.js dipakai versi **App Router** (folder `src/app/`), di mana setiap folder menjadi sebuah halaman URL secara otomatis.

### Supabase — Database & Autentikasi
Supabase adalah layanan database berbasis PostgreSQL yang dilengkapi fitur login/logout siap pakai. Kita tidak perlu membuat sistem login dari nol.

Supabase juga punya fitur **Row Level Security (RLS)** — aturan di level database yang menentukan siapa boleh baca/tulis data apa. Contoh: user hanya bisa melihat pesanannya sendiri, bukan milik orang lain.

### Zustand — State Keranjang Belanja
Zustand adalah library untuk menyimpan data sementara di browser. Di project ini dipakai untuk keranjang belanja — saat user tambah produk ke keranjang, data itu disimpan di browser (bukan database) sampai user checkout.

Data keranjang juga disimpan di `localStorage` browser, jadi tidak hilang saat halaman di-refresh.

### Tailwind CSS + shadcn/ui — Tampilan
Tailwind adalah framework CSS yang ditulis langsung di dalam HTML/JSX menggunakan class. shadcn/ui adalah kumpulan komponen siap pakai (tombol, kartu, dialog, dll) yang dibangun di atas Tailwind.

---

## Struktur Folder

```
halume/
├── src/
│   ├── app/                    ← Semua halaman aplikasi
│   │   ├── page.tsx            ← Halaman beranda (/)
│   │   ├── layout.tsx          ← Layout utama (Navbar ada di sini)
│   │   ├── globals.css         ← CSS global
│   │   ├── login/page.tsx      ← Halaman login
│   │   ├── register/page.tsx   ← Halaman daftar akun
│   │   ├── cart/page.tsx       ← Halaman keranjang
│   │   ├── checkout/page.tsx   ← Halaman checkout
│   │   ├── orders/page.tsx     ← Riwayat pesanan user
│   │   ├── products/
│   │   │   └── [id]/page.tsx   ← Detail produk (URL dinamis)
│   │   └── admin/
│   │       ├── layout.tsx      ← Layout khusus admin
│   │       ├── page.tsx        ← Dashboard admin
│   │       ├── products/       ← Kelola produk
│   │       └── orders/         ← Kelola pesanan
│   │
│   ├── components/             ← Komponen yang dipakai ulang
│   │   ├── Navbar.tsx          ← Navigasi atas
│   │   ├── ProductCard.tsx     ← Kartu produk di katalog
│   │   └── ui/                 ← Komponen shadcn/ui
│   │
│   ├── lib/
│   │   ├── supabase/
│   │   │   ├── client.ts       ← Koneksi Supabase untuk browser
│   │   │   └── server.ts       ← Koneksi Supabase untuk server
│   │   └── utils.ts            ← Helper function
│   │
│   ├── stores/
│   │   └── cart.ts             ← State keranjang belanja (Zustand)
│   │
│   ├── types/
│   │   └── index.ts            ← Definisi tipe data TypeScript
│   │
│   └── middleware.ts           ← Proteksi halaman (cek login)
│
├── supabase/
│   └── schema.sql              ← SQL untuk membuat database
│
├── public/                     ← File statis (gambar, ikon)
├── .env.local.example          ← Contoh konfigurasi environment
└── next.config.ts              ← Konfigurasi Next.js
```

---

## Alur Kerja Aplikasi

### Alur User Belanja
```
1. Buka halaman beranda (/)
   → Produk diambil dari database Supabase
   → Ditampilkan dalam grid kartu produk

2. Klik produk → halaman detail (/products/:id)
   → Pilih jumlah → klik "Tambah ke Keranjang"
   → Data produk disimpan di Zustand (browser)

3. Buka keranjang (/cart)
   → Lihat semua item, ubah jumlah, hapus item
   → Klik "Lanjut ke Checkout"

4. Halaman checkout (/checkout)
   → Middleware cek: sudah login? kalau belum → redirect ke /login
   → Isi nama, telepon, alamat
   → Klik "Buat Pesanan" → data dikirim ke Supabase
   → Keranjang dikosongkan, halaman sukses muncul

5. Lihat riwayat di /orders
```

### Alur Admin
```
1. Login dengan akun yang punya role 'admin'
2. Navbar otomatis tampilkan tombol "Admin"
3. Masuk ke /admin → lihat statistik
4. /admin/products → tambah/edit/hapus produk
5. /admin/orders → lihat semua pesanan, ubah status
```

---

## Sistem Autentikasi

Autentikasi ditangani oleh **Supabase Auth**. Saat user login, Supabase menyimpan **session** (semacam tanda pengenal) di cookie browser.

File `src/middleware.ts` berjalan di setiap request halaman dan memeriksa:
- Apakah user sudah login?
- Apakah user punya role 'admin'?

Berdasarkan hasil pemeriksaan, middleware akan memperbolehkan akses atau redirect ke halaman lain.

```
Request ke /checkout
       │
       ▼
  middleware.ts
       │
  Cek session ──── Belum login? ──→ Redirect ke /login
       │
  Sudah login ──────────────────→ Lanjut ke halaman
```

---

## Dua Jenis Koneksi Supabase

Project ini punya dua file koneksi Supabase yang berbeda:

| File | Dipakai di | Kenapa Berbeda |
|------|-----------|----------------|
| `lib/supabase/client.ts` | Komponen browser (`'use client'`) | Berjalan di browser, pakai cookie browser |
| `lib/supabase/server.ts` | Server Components & middleware | Berjalan di server, pakai cookie request/response |

Ini penting karena cara mengakses cookie di browser dan server berbeda secara teknis.
