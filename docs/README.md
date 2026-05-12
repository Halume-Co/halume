# Dokumentasi Halume

Selamat datang di dokumentasi project **Halume** — aplikasi e-commerce penjualan parfum eksklusif berbasis web.

## Daftar Isi

| Dokumen | Isi |
|---------|-----|
| [ARSITEKTUR.md](./ARSITEKTUR.md) | Gambaran besar sistem, tech stack, dan alur kerja aplikasi |
| [FITUR.md](./FITUR.md) | Penjelasan setiap fitur dari sudut pandang kode |
| [DATABASE.md](./DATABASE.md) | Struktur tabel database dan relasi antar tabel |
| [PANDUAN_LOKAL.md](./PANDUAN_LOKAL.md) | Cara menjalankan project di laptop sendiri |

## Ringkasan Project

**Halume** adalah aplikasi web e-commerce single-company untuk penjualan parfum. Pengguna bisa melihat katalog produk, menambahkan ke keranjang, dan melakukan checkout. Admin bisa mengelola produk dan memantau pesanan.

### Halaman yang Tersedia

| Halaman | URL | Akses |
|---------|-----|-------|
| Beranda / Katalog | `/` | Semua orang |
| Detail Produk | `/products/:id` | Semua orang |
| Login | `/login` | Tamu |
| Daftar | `/register` | Tamu |
| Keranjang | `/cart` | Semua orang |
| Checkout | `/checkout` | User login |
| Riwayat Pesanan | `/orders` | User login |
| Dashboard Admin | `/admin` | Admin saja |
| Kelola Produk | `/admin/products` | Admin saja |
| Kelola Pesanan | `/admin/orders` | Admin saja |

### Tech Stack

- **Next.js 16** — Framework utama (React)
- **Supabase** — Database + autentikasi
- **Zustand** — State management keranjang belanja
- **Tailwind CSS + shadcn/ui** — Tampilan antarmuka
- **TypeScript** — Bahasa pemrograman
- **Vercel** — Hosting & deployment

### Link Penting

- Website: [halume.vercel.app](https://halume.vercel.app)
- Repository: [github.com/Halume-Co/halume](https://github.com/Halume-Co/halume)
