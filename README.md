# Jobsheet 12 — Integrasi Modul Peminjaman

**Sub-CPMK:** Mengintegrasikan front-end dan back-end proyek SIMPUS-Mini secara utuh.

## Perubahan dari Jobsheet 11
- `sql/03_peminjaman.sql` — tabel `peminjaman` (`buku_id` → `buku(id)`, `anggota_id` → `anggota(id)`, `tanggal_pinjam DATE DEFAULT CURRENT_DATE`, `tanggal_kembali`, `status`). Kalau database lama masih memakai kolom `id_buku`/`id_anggota`, file ini otomatis mengganti namanya.
- Modul `peminjaman/`:
  - `tambah.php` + `proses_tambah.php` — pilih anggota + buku (dropdown hanya `stok > 0`), simpan transaksi dan kurangi stok dalam satu transaction (`beginTransaction` / `commit` / `rollBack`) dengan `SELECT ... FOR UPDATE`.
  - `kembali.php` + `proses_kembali.php` — daftar transaksi aktif (JOIN 3 tabel, bisa dicari), tombol Kembalikan menambah stok lagi dalam transaction serupa.
  - `riwayat.php` — histori peminjaman per anggota (JOIN `peminjaman` + `buku`).
- `includes/header.php` — menu dropdown **Peminjaman** (Peminjaman Baru, Pengembalian, Riwayat), hanya saat login.
- `index.php` — kartu "Sedang Dipinjam" dari `COUNT(*) ... WHERE status = 'dipinjam'`, ditambah Aksi Cepat dan tabel Transaksi Terbaru untuk petugas.
- `buku/hapus.php`, `anggota/hapus.php` — pesan jelas kalau data masih punya riwayat peminjaman (foreign key).
- **Tugas mandiri:** anggota yang masih meminjam buku lebih dari 14 hari tidak boleh meminjam buku baru (`proses_tambah.php`).
- **Latihan 6.4:** kolom Jatuh Tempo (+14 hari, label Terlambat) dan No. HP anggota di halaman Pengembalian; uji race condition otomatis di `tests/uji_peminjaman.php`.

## Cara menjalankan
```bash
psql -d simpus_mini -f sql/03_peminjaman.sql
php -S localhost:8000
```
Atau buka `sql/setup_db.php` sekali untuk menjalankan ketiga file SQL. Laragon: arahkan virtual host ke folder `Jobsheet-12/`.

## Pengujian end-to-end
Registrasi petugas → Login → Tambah Buku & Anggota → Peminjaman Baru → stok buku berkurang di Daftar Buku → kartu "Sedang Dipinjam" bertambah → Pengembalian → stok kembali dan transaksi hilang dari daftar aktif → Riwayat → status "Selesai" → Logout.

Uji otomatis (server dijalankan dengan beberapa worker supaya request paralel benar-benar bersamaan):
```bash
PHP_CLI_SERVER_WORKERS=4 php -S localhost:8000
php tests/uji_peminjaman.php <username> <password>
```

## Struktur Folder
```text
Jobsheet-12/
├── index.php                  # Kartu "Sedang Dipinjam" dari database + transaksi terbaru
├── includes/header.php        # + menu Peminjaman
├── peminjaman/
│   ├── tambah.php, proses_tambah.php    # transaction + FOR UPDATE + cek terlambat
│   ├── kembali.php, proses_kembali.php  # transaction kebalikan
│   └── riwayat.php                      # JOIN
├── sql/03_peminjaman.sql      # tabel peminjaman + foreign key
├── tests/uji_peminjaman.php   # uji guard, CSRF, race condition, klik ganda, keterlambatan
├── buku/, anggota/, auth/     # sama seperti Jobsheet 11 (+ pesan foreign key di hapus.php)
└── docs/                      # identik dengan Jobsheet 11
```
