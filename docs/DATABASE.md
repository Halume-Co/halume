# Database

Project ini menggunakan **PostgreSQL** melalui layanan Supabase. Schema lengkap ada di file `supabase/schema.sql`.

---

## Diagram Relasi Tabel

```
auth.users (dikelola Supabase)
     │
     │ 1
     ▼
  profiles ──── menyimpan data profil & role user
     
auth.users
     │
     │ 1
     ▼
  orders ──────────────────── 1 pesanan milik 1 user
     │
     │ 1
     ▼
order_items ─────────────────── bisa banyak item per pesanan
     │
     │ N
     ▼
  products ────────────────── 1 item merujuk ke 1 produk
```

---

## Tabel: `profiles`

Menyimpan data profil pengguna. Dibuat otomatis saat user mendaftar via trigger database.

| Kolom | Tipe | Keterangan |
|-------|------|-----------|
| `id` | uuid | Primary key, sama dengan ID di `auth.users` |
| `email` | text | Email user |
| `full_name` | text | Nama lengkap |
| `role` | text | `'customer'` atau `'admin'` (default: customer) |
| `created_at` | timestamptz | Waktu dibuat |

**Catatan:** Kolom `role` menentukan level akses user di seluruh aplikasi. Untuk membuat akun admin, jalankan SQL di file `supabase/set_admin.sql`.

---

## Tabel: `products`

Menyimpan semua data produk parfum.

| Kolom | Tipe | Keterangan |
|-------|------|-----------|
| `id` | uuid | Primary key, auto-generate |
| `name` | text | Nama produk |
| `description` | text | Deskripsi (boleh kosong) |
| `price` | numeric | Harga dalam Rupiah |
| `stock` | integer | Jumlah stok tersedia |
| `image_url` | text | URL gambar produk (boleh kosong) |
| `category` | text | Nama kategori (boleh kosong) |
| `created_at` | timestamptz | Waktu dibuat |
| `updated_at` | timestamptz | Waktu terakhir diubah (auto-update) |

---

## Tabel: `orders`

Menyimpan setiap transaksi pembelian.

| Kolom | Tipe | Keterangan |
|-------|------|-----------|
| `id` | uuid | Primary key, auto-generate |
| `user_id` | uuid | Referensi ke `auth.users` |
| `status` | text | Status pesanan (lihat di bawah) |
| `total` | numeric | Total harga keseluruhan |
| `shipping_address` | text | Alamat pengiriman lengkap |
| `created_at` | timestamptz | Waktu pesanan dibuat |
| `updated_at` | timestamptz | Waktu terakhir diubah |

**Status pesanan yang valid:**

| Status | Arti |
|--------|------|
| `pending` | Baru dibuat, menunggu konfirmasi |
| `processing` | Sedang diproses |
| `shipped` | Sudah dikirim |
| `delivered` | Sudah diterima |
| `cancelled` | Dibatalkan |

---

## Tabel: `order_items`

Menyimpan rincian produk dalam setiap pesanan. Satu pesanan bisa punya banyak baris di tabel ini.

| Kolom | Tipe | Keterangan |
|-------|------|-----------|
| `id` | uuid | Primary key, auto-generate |
| `order_id` | uuid | Referensi ke tabel `orders` |
| `product_id` | uuid | Referensi ke tabel `products` |
| `quantity` | integer | Jumlah produk yang dipesan |
| `price` | numeric | **Harga saat transaksi** (bukan harga produk saat ini) |
| `created_at` | timestamptz | Waktu dibuat |

**Kenapa harga disimpan di sini?**
Harga produk bisa berubah kapan saja. Dengan menyimpan harga di `order_items`, riwayat pesanan lama tetap menampilkan harga yang benar pada saat transaksi terjadi.

---

## Row Level Security (RLS)

Supabase menggunakan **Row Level Security** — aturan keamanan langsung di level database. Ini memastikan user hanya bisa mengakses data yang memang miliknya, bahkan jika ada bug di aplikasi.

### Ringkasan Aturan RLS

| Tabel | Siapa yang bisa SELECT | Siapa yang bisa INSERT/UPDATE/DELETE |
|-------|----------------------|-------------------------------------|
| `profiles` | User sendiri + Admin | User sendiri (update only) |
| `products` | Semua orang | Admin saja |
| `orders` | User pemilik + Admin | User (insert), Admin (update) |
| `order_items` | User pemilik + Admin | User pemilik |

---

## Auto-Generated Features

### Trigger: Buat Profil Otomatis
Saat user baru mendaftar via Supabase Auth, sebuah **trigger** (`on_auth_user_created`) otomatis membuat baris baru di tabel `profiles` dengan role default `'customer'`.

### Trigger: Update `updated_at` Otomatis
Tabel `products` dan `orders` punya trigger yang otomatis memperbarui kolom `updated_at` setiap kali ada perubahan data.

---

## Cara Reset Database

Jika ingin membuat ulang database dari awal:
1. Buka Supabase Dashboard → SQL Editor
2. Jalankan isi file `supabase/schema.sql`
3. Data sample produk sudah termasuk di akhir file tersebut
