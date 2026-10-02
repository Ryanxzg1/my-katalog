# PRD — My-Katalog

**Versi:** 1.0 (MVP) · **Status:** Draft untuk review · **Stack:** Laravel + Inertia.js + React (Breeze)

---

## 1. Overview

My-Katalog adalah web app untuk mencatat barang yang dimiliki, memantau statusnya (kapan diperoleh, kapan bisa dipakai, kapan habis masa pakai), merencanakan pembelian berbasis data riil, dan menerima notifikasi otomatis.

Dipakai pribadi oleh pembuat, lalu dibuka untuk publik (via LinkedIn) sebagai portfolio project. Artinya: **multi-user, public-facing**.

## 2. Problem Statement

- Tidak ada daftar terpusat barang yang dimiliki.
- Tidak ada histori kapan barang diperoleh.
- Tidak ada informasi kapan barang bisa dipakai dan kapan masa pakainya berakhir.
- Rencana pembelian hanya asumsi, tanpa data riil (stok, usia pakai, histori).

## 3. Goals & Success Metrics

| Goal | Metric | Target |
| --- | --- | --- |
| Semua barang termonitor | Barang tercatat lengkap dengan tanggal diperoleh | 100% barang milik pembuat tercatat dalam 2 minggu setelah launch |
| Plan pembelian berbasis data | Plan dibuat dari barang existing (convert 1 klik) | Fitur berfungsi end-to-end |
| Notifikasi berguna | Notifikasi terkirim tepat waktu, tanpa duplikat | 0 notifikasi duplikat per barang per threshold |
| Kualitas portfolio | Feature test untuk flow kritis | Auth, CRUD barang, ownership, plan, export ter-cover |
| Siap publik | Isolasi data antar user | 0 celah akses lintas user (diverifikasi test) |

## 4. Target Users

1. **Pembuat (primary):** developer yang mencatat barang pribadi dan butuh reminder.
2. **Pengunjung LinkedIn (secondary):** orang awam yang mencoba sekilas. Harus paham cara pakai tanpa dokumentasi; kemungkinan besar hanya mencoba 1–5 menit.

## 5. Scope MVP

1. Auth User (register, verifikasi email, login, reset password)
2. Dashboard / Summary
3. Halaman Input & Kelola Barang
4. Halaman Setting Jenis Barang (kategori)
5. Halaman Pembuatan Plan Pembelian
6. Notifikasi (in-app + email)
7. Ekspor Excel / PDF

> Catatan: Notifikasi ada di Goal awal tetapi belum masuk daftar fitur di draft sebelumnya. Di versi ini dimasukkan sebagai fitur MVP.

## 6. Functional Requirements

### 6.1 Auth

- **FR-AUTH-1** Register dengan nama, email, password.
- **FR-AUTH-2** Email wajib diverifikasi sebelum mengakses dashboard dan fitur lain.
- **FR-AUTH-3** Login email/password; reset password via email.
- **FR-AUTH-4** Saat register, sistem membuat kategori default untuk user tersebut (lihat 6.4).
- **FR-AUTH-5** Rate limit pada login, register, dan reset password.
- **FR-AUTH-6** User dapat menghapus akunnya; seluruh data user ikut terhapus (cascade).

### 6.2 Dashboard

- **FR-DASH-1** Ringkasan: total barang, total barang per kategori, jumlah plan aktif.
- **FR-DASH-2** Widget "Perlu Perhatian": barang yang masa pakainya berakhir dalam 30 hari ke depan atau sudah lewat.
- **FR-DASH-3** Widget "Plan Terdekat": plan berstatus *planned* dengan target tanggal terdekat.
- **FR-DASH-4** Empty state yang jelas untuk user baru (ajakan menambah barang pertama).
- **FR-DASH-5** Semua data hanya milik user yang login.

### 6.3 Barang

- **FR-ITEM-1** Create, read, update, delete barang.
- **FR-ITEM-2** Field: nama, kategori, kuantitas, satuan, tanggal diperoleh, tanggal mulai bisa dipakai, tanggal akhir masa pakai (opsional), harga (opsional), catatan.
- **FR-ITEM-3** List dengan pagination, pencarian nama, filter kategori, dan sorting (tanggal diperoleh, tanggal akhir masa pakai).
- **FR-ITEM-4** Validasi: nama wajib; kuantitas bilangan bulat ≥ 0; tanggal akhir masa pakai tidak boleh sebelum tanggal diperoleh.
- **FR-ITEM-5** Aksi pada barang: "Buat Plan Pembelian" (prefill nama, kategori, catatan ke form plan).
- **FR-ITEM-6** Batas jumlah barang per user (default 500) untuk mencegah abuse.

### 6.4 Kategori (Jenis Barang)

- **FR-CAT-1** CRUD kategori milik user.
- **FR-CAT-2** Kategori default per user dibuat saat register (contoh: Elektronik, Consumable, Dokumen, Lainnya).
- **FR-CAT-3** Nama kategori unik per user.
- **FR-CAT-4** Kategori yang masih dipakai barang tidak bisa dihapus tanpa memindahkan barangnya terlebih dahulu (atau ditolak dengan pesan jelas).

### 6.5 Plan Pembelian

- **FR-PLAN-1** CRUD plan: nama barang, kategori (opsional), target tanggal beli, estimasi budget (opsional), catatan.
- **FR-PLAN-2** Status: `planned`, `bought`, `cancelled`. Transisi: `planned → bought | cancelled`.
- **FR-PLAN-3** Plan dapat dibuat manual atau dari barang existing (FR-ITEM-5).
- **FR-PLAN-4** Saat status berubah ke `bought`, user ditawari membuat barang baru dari plan tersebut (prefill). Tidak otomatis, agar tidak membuat data tanpa konfirmasi.
- **FR-PLAN-5** List plan dengan filter status dan sorting target tanggal.

### 6.6 Notifikasi

- **FR-NOTIF-1** Scheduled job harian memeriksa barang yang akhir masa pakainya jatuh pada H-7 dan H-0 (hari-H), serta barang yang sudah lewat tanpa pernah dinotifikasi.
- **FR-NOTIF-2** Channel: in-app (badge + daftar notifikasi, tandai sudah dibaca) dan email.
- **FR-NOTIF-3** Idempotent: satu barang hanya mendapat satu notifikasi per threshold. Job yang dijalankan ulang tidak boleh membuat duplikat.
- **FR-NOTIF-4** User dapat mematikan notifikasi email di pengaturan akun (in-app tetap aktif).
- **FR-NOTIF-5** Jika pengiriman email gagal, job di-retry dengan batas tertentu; kegagalan akhir di-log dengan konteks (user_id, item_id, jenis notifikasi). Kegagalan satu user tidak menghentikan proses user lain.
- **FR-NOTIF-6** Notifikasi plan (target tanggal beli mendekat) tidak masuk MVP.

### 6.7 Ekspor

- **FR-EXP-1** Ekspor daftar barang dan daftar plan ke Excel (.xlsx) dan PDF.
- **FR-EXP-2** Hanya mengekspor data milik user yang login; hormati filter aktif di list.
- **FR-EXP-3** Sel teks yang diawali `=`, `+`, `-`, `@` harus dinetralkan pada ekspor Excel untuk mencegah formula injection.
- **FR-EXP-4** Ekspor sinkron untuk MVP, aman karena jumlah data per user dibatasi (FR-ITEM-6).
- **FR-EXP-5** Rate limit pada endpoint ekspor.

## 7. User Flow

**A. Onboarding:** Landing → Register → verifikasi email → login → dashboard kosong (empty state) → "Tambah Barang".

**B. Tambah barang:** Dashboard/List → "Tambah Barang" → isi form → simpan → muncul di list dan dashboard.

**C. Barang hampir habis → plan:** Dashboard widget "Perlu Perhatian" → buka barang → "Buat Plan Pembelian" → form plan prefill → simpan.

**D. Plan manual:** Menu Plan → "Tambah Plan" → isi form → simpan (status `planned`).

**E. Plan selesai:** Plan → ubah status `bought` → tawaran "Tambahkan ke daftar barang?" → ya (form prefill) / tidak.

**F. Notifikasi:** Job harian → notifikasi in-app + email → user klik → halaman detail barang → (opsional) flow C.

**G. Ekspor:** List Barang/Plan → "Export" → pilih Excel/PDF → file terunduh.

## 8. Data Model (konseptual)

**users** — id, name, email (unique), password, email_verified_at, email_notifications_enabled (default true), timestamps.

**categories** — id, user_id (FK, cascade), name · unique(user_id, name).

**items** — id, user_id (FK, cascade), category_id (FK), name, quantity, unit, acquired_at, usable_from, expires_at (nullable), price (nullable), notes (nullable), timestamps. Index: (user_id, expires_at), (user_id, category_id).

**plans** — id, user_id (FK, cascade), category_id (nullable), source_item_id (nullable, FK), name, target_date, estimated_budget (nullable), status, notes (nullable), timestamps. Index: (user_id, status, target_date).

**item_notification_logs** — id, item_id (FK, cascade), type (`h7`/`h0`/`overdue`), notified_at · unique(item_id, type). Dasar idempotency FR-NOTIF-3.

**notifications** — memakai tabel notifikasi bawaan Laravel (database channel).

Aturan umum: seluruh tabel milik user memiliki `user_id` dan seluruh query harus ter-scope ke user login.

## 9. Non-Functional Requirements

**Security**

- Policy untuk setiap resource (item, category, plan) pada setiap aksi; tidak mengandalkan pengecekan manual per controller.
- Validasi lewat Form Request; mass assignment dilindungi.
- Perlindungan IDOR: akses resource lain mengembalikan 404/403, bukan data.
- CSRF dan XSS: gunakan proteksi bawaan framework; hindari render HTML mentah dari input user.
- Password di-hash dengan mekanisme bawaan; secret hanya lewat environment variable.
- Rate limiting: login, register, reset password, ekspor.
- Log error dengan konteks (user_id, resource, aksi) tanpa menyimpan password atau token.

**Performance**

- Waktu muat halaman utama (dashboard, list) \< 3 detik pada koneksi broadband standar, diukur dengan data sampai 500 barang.
- Tidak ada N+1 query pada list dan dashboard (eager loading); semua list ber-pagination.

**Reliability**

- Job notifikasi idempotent dan toleran terhadap kegagalan parsial.
- Kegagalan layanan email tidak boleh memblokir flow utama user.
- Backup database sesuai kemampuan hosting yang dipilih.

**Maintainability**

- Business logic di Service/Action class, bukan di controller.
- Penamaan deskriptif, tanpa magic number (threshold notifikasi dan batas jumlah data di config).
- Migration dan seeder untuk data demo.

## 10. Technical Constraints & Stack

| Layer | Pilihan |
| --- | --- |
| Backend | Laravel |
| Frontend | React via Inertia.js (starter kit Laravel Breeze) |
| Auth | Breeze (register, login, verifikasi email, reset password) |
| Authorization | Laravel Policies (tanpa package role, karena hanya ada ownership) |
| Ekspor Excel | maatwebsite/laravel-excel |
| Ekspor PDF | barryvdh/laravel-dompdf (atau alternatif sejenis) |
| Scheduler & Queue | Task Scheduling + Queue bawaan Laravel |
| Testing | PHPUnit/Pest untuk feature test |

Kelemahan Inertia: tidak menyediakan REST API yang reusable untuk mobile app; jika dibutuhkan kelak, perlu API layer terpisah.

## 11. Acceptance Criteria

1. User A tidak dapat melihat, mengubah, menghapus, atau mengekspor data User B, termasuk dengan memanipulasi ID di URL (dibuktikan feature test).
2. User yang belum verifikasi email tidak dapat mengakses fitur utama.
3. User baru dapat menambah barang pertama dari empty state tanpa instruksi tambahan.
4. Dashboard dan list termuat \< 3 detik pada dataset 500 barang.
5. Notifikasi H-7 dan H-0 terkirim sekali per barang per threshold; menjalankan job dua kali tidak menghasilkan duplikat.
6. Ekspor Excel dan PDF menghasilkan file valid; sel berawalan `=`, `+`, `-`, `@` tidak dieksekusi sebagai formula.
7. Rate limit aktif pada login, register, reset password, dan ekspor.
8. Semua fitur MVP (bagian 5) berfungsi end-to-end.
9. Feature test lulus untuk: auth, CRUD barang, CRUD kategori, plan, ownership, notifikasi.

## 12. Out of Scope (Fase 1)

- Admin panel dan moderasi user
- Fitur sosial (katalog publik, sharing antar user)
- Barcode/QR scanner dan upload foto barang
- Push notification dan mobile app
- Notifikasi untuk plan
- Prediksi konsumsi berbasis pola/ML
- Integrasi marketplace
- Ekspor asinkron untuk data besar
- Login OAuth (Google) — kandidat fase 1.5
- Multi-bahasa dan multi-currency

## 13. Timeline (estimasi kasar, 1 developer)

| Minggu | Fokus |
| --- | --- |
| 1 | Setup project, Breeze (Inertia + React), data model, migration, kategori default |
| 2 | CRUD barang & kategori, Policy, Form Request, test ownership |
| 3 | Dashboard, empty state, scheduler + job notifikasi (idempotent), notifikasi in-app |
| 4 | Plan pembelian, convert barang → plan, plan → barang, email notifikasi |
| 5 | Ekspor Excel/PDF, rate limiting, hardening keamanan |
| 6 | Test menyeluruh, README + screenshot, deploy, monitoring dasar |

## 14. Risks & Mitigation

| Risk | Dampak | Mitigasi |
| --- | --- | --- |
| Celah ownership (IDOR) | Kebocoran data antar user | Policy di semua resource + feature test lintas user |
| Spam register / bot | Database penuh sampah, biaya naik | Verifikasi email, rate limit, batas data per user |
| Abuse ekspor | Beban server | Rate limit, batas jumlah data |
| Deliverability email | Notifikasi tidak sampai | Gunakan layanan email transaksional; pantau kegagalan lewat log |
| Scope creep | Timeline molor | Patuhi daftar Out of Scope |
| Biaya hosting saat ramai | Biaya tak terduga | Pilih hosting dengan batas biaya jelas; batasi data per user |

## 15. Open Questions / Asumsi

1. **Hosting & email provider** belum ditentukan; memengaruhi deployment, queue worker, dan scheduler (cron).
2. **Timezone:** diasumsikan satu timezone tetap (WIB) untuk perhitungan H-7/H-0. Jika pengunjung dari timezone lain, perlu keputusan lanjutan.
3. **Barang tanpa masa pakai** (misal furnitur) diasumsikan tidak memicu notifikasi.
4. **Batas 500 barang per user** adalah angka asumsi; sesuaikan setelah ada data pemakaian.
5. **Kategori default per user** (disalin saat register) dipilih agar sederhana; alternatifnya kategori global + custom, lebih hemat data tetapi query lebih kompleks.
6. **Penghapusan akun** dengan cascade diasumsikan cukup; belum ada kebutuhan retensi data.