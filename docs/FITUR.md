# Penjelasan Fitur

## 1. Katalog Produk (Beranda)

**File:** `src/app/page.tsx`

Halaman utama yang menampilkan semua produk dari database dalam bentuk grid kartu.

**Cara kerjanya:**
1. Saat halaman dibuka, `useEffect` dijalankan untuk mengambil data produk dari Supabase
2. Data produk disimpan di state lokal `products`
3. Ada dua filter yang bekerja secara bersamaan: **pencarian teks** dan **filter kategori**
4. Setiap kali `search` atau `category` berubah, `useEffect` kedua memfilter ulang daftar produk

**Filter kategori yang tersedia:**
- Semua, Limited Edition, Signature Series, Basics, Nuit Collection, Imperial Line

**Komponen yang dipakai:** `ProductCard` untuk setiap produk dalam grid

---

## 2. Kartu Produk

**File:** `src/components/ProductCard.tsx`

Komponen yang menampilkan satu produk dalam bentuk kartu. Dipakai di halaman beranda.

**Yang ditampilkan:**
- Gambar produk (atau ikon placeholder kalau tidak ada gambar)
- Badge "Habis" jika stok = 0 (overlay di atas gambar)
- Kategori, nama, harga, info stok
- Tombol "Tambah ke Keranjang"

**Cara kerja tombol tambah:**
- Memanggil fungsi `addItem` dari Zustand store
- Menampilkan notifikasi toast "... ditambahkan ke keranjang"
- Seluruh kartu bisa diklik untuk ke halaman detail produk

---

## 3. Detail Produk

**File:** `src/app/products/[id]/page.tsx`

Halaman detail satu produk. URL-nya dinamis — `[id]` diganti dengan ID produk yang sebenarnya (contoh: `/products/abc-123`).

**Fitur di halaman ini:**
- Gambar besar produk
- Nama, harga, kategori, deskripsi, stok
- Selector jumlah (tombol +/-)
  - Minimum: 1
  - Maximum: sesuai stok yang tersedia
- Tombol "Tambah ke Keranjang" (disabled kalau stok habis)
- Tombol kembali ke halaman sebelumnya

---

## 4. Registrasi & Login

**File:** `src/app/register/page.tsx`, `src/app/login/page.tsx`

**Alur registrasi:**
1. User isi nama, email, password (min. 6 karakter)
2. `supabase.auth.signUp()` — buat akun di Supabase Auth
3. `supabase.auth.signInWithPassword()` — langsung login untuk dapat session
4. `supabase.from('profiles').upsert()` — simpan data profil (nama, email, role) ke tabel `profiles`
5. Redirect ke beranda

**Alur login:**
1. User isi email dan password
2. `supabase.auth.signInWithPassword()` — verifikasi ke Supabase
3. Jika berhasil → redirect ke beranda
4. Jika gagal → tampilkan pesan error yang spesifik (email salah, belum konfirmasi, dll)

**Catatan:** Halaman login/register hanya bisa diakses kalau **belum login**. Middleware akan redirect ke beranda kalau user yang sudah login mencoba membuka halaman ini.

---

## 5. Navbar

**File:** `src/components/Navbar.tsx`

Navigasi yang muncul di semua halaman (didefinisikan di `src/app/layout.tsx`).

**Navbar berubah tergantung kondisi user:**

| Kondisi | Yang Tampil |
|---------|------------|
| Belum login | Produk, ikon keranjang, tombol Masuk |
| Sudah login (customer) | Produk, ikon keranjang, nama user (dropdown) |
| Sudah login (admin) | Produk, tombol Admin, ikon keranjang, nama user |

**Dropdown nama user berisi:**
- Pesanan Saya → ke `/orders`
- Keluar → logout dan redirect ke beranda

**Ikon keranjang** menampilkan badge merah dengan jumlah item jika keranjang tidak kosong.

Navbar juga **mendeteksi perubahan status login** secara real-time menggunakan `supabase.auth.onAuthStateChange()`.

---

## 6. Keranjang Belanja

**File:** `src/app/cart/page.tsx`, `src/stores/cart.ts`

Keranjang belanja dikelola oleh **Zustand store** — data disimpan di browser, bukan database. Data tetap ada walaupun halaman di-refresh karena menggunakan `localStorage`.

**Fungsi-fungsi di cart store:**

| Fungsi | Kegunaan |
|--------|---------|
| `addItem(product, qty)` | Tambah produk. Kalau sudah ada, tambah jumlahnya (tidak melebihi stok) |
| `removeItem(productId)` | Hapus produk dari keranjang |
| `updateQuantity(id, qty)` | Ubah jumlah. Kalau qty ≤ 0, produk dihapus otomatis |
| `clearCart()` | Kosongkan seluruh keranjang |
| `total()` | Hitung total harga semua item |
| `itemCount()` | Hitung total jumlah item (untuk badge di navbar) |

**Tampilan keranjang:**
- Daftar produk dengan gambar, nama, harga, kontrol jumlah, tombol hapus
- Panel ringkasan pesanan dengan total harga
- Tombol "Lanjut ke Checkout"

---

## 7. Checkout

**File:** `src/app/checkout/page.tsx`

**Proses checkout (urutan eksekusi):**
1. Cek keranjang tidak kosong
2. Cek user sudah login (kalau belum → redirect ke login)
3. User isi form pengiriman (nama penerima, telepon, alamat)
4. Klik "Buat Pesanan":
   - Insert data ke tabel `orders` (total, alamat, status 'pending')
   - Insert semua item ke tabel `order_items` (produk, jumlah, harga saat itu)
   - Update stok setiap produk di tabel `products` (dikurangi sesuai jumlah)
   - `clearCart()` — kosongkan keranjang
5. Tampilkan halaman sukses dengan tombol "Lihat Pesanan" dan "Belanja Lagi"

**Kenapa harga disimpan di `order_items`?**
Harga produk bisa berubah di masa depan. Dengan menyimpan harga saat transaksi di `order_items`, riwayat pesanan tetap akurat walaupun harga produk sudah diubah admin.

---

## 8. Riwayat Pesanan

**File:** `src/app/orders/page.tsx`

Menampilkan semua pesanan milik user yang sedang login.

**Fitur:**
- Daftar pesanan diurutkan dari yang terbaru
- Setiap pesanan menampilkan: nomor order, tanggal, jumlah item, total, status
- Klik chevron untuk expand → tampilkan detail item dan alamat pengiriman

**Status pesanan:**

| Status | Tampilan |
|--------|---------|
| pending | Abu-abu — Menunggu |
| processing | Biru — Diproses |
| shipped | Biru — Dikirim |
| delivered | Hijau outline — Selesai |
| cancelled | Merah — Dibatalkan |

---

## 9. Dashboard Admin

**File:** `src/app/admin/page.tsx`

Halaman pertama yang dilihat admin setelah masuk ke `/admin`.

**Menampilkan 4 statistik:**
- Total parfum (produk) di database
- Total pesanan yang masuk
- Total user terdaftar
- Total pendapatan (dari pesanan berstatus 'delivered')

Di bawahnya ada tabel **5 pesanan terbaru**.

Halaman ini menggunakan **Server Component** (tanpa `'use client'`) — data diambil di server sebelum halaman dikirim ke browser, sehingga lebih cepat dan aman.

---

## 10. Kelola Produk (Admin)

**File:** `src/app/admin/products/page.tsx`

CRUD lengkap untuk produk.

**Fitur:**
- Tabel daftar semua produk
- Tombol **Tambah Produk** → buka dialog form
- Tombol **Edit** (ikon pensil) → buka dialog form dengan data produk terisi
- Tombol **Hapus** (ikon tong sampah) → konfirmasi → hapus dari database

**Field produk:**
- Nama produk (wajib)
- Harga dalam Rupiah (wajib)
- Stok (wajib)
- Kategori (dropdown)
- URL gambar
- Deskripsi

---

## 11. Kelola Pesanan (Admin)

**File:** `src/app/admin/orders/page.tsx`

Daftar semua pesanan dari seluruh user.

**Fitur:**
- Tabel semua pesanan dengan nomor order, tanggal, total, status
- Expand baris untuk lihat detail item dan alamat pengiriman
- Dropdown untuk **mengubah status pesanan** langsung dari tabel
  - Perubahan langsung tersimpan ke database tanpa perlu klik tombol simpan

---

## 12. Proteksi Halaman (Middleware)

**File:** `src/middleware.ts`

Middleware berjalan **sebelum** setiap halaman dimuat dan mengatur siapa boleh akses halaman apa.

**Aturan proteksi:**

| Kondisi | Halaman yang diproteksi | Aksi |
|---------|------------------------|------|
| Belum login | `/checkout`, `/orders`, `/admin` | Redirect ke `/login` |
| Sudah login | `/login`, `/register` | Redirect ke `/` |
| Login tapi bukan admin | `/admin` | Redirect ke `/` |
