# Detail Relasi Database Aplikasi GSJA KAIROS

Dokumen ini menjelaskan Entity Relationship Diagram (ERD) dari aplikasi GSJA KAIROS, berdasarkan analisis file migrasi dan model Eloquent.

---

## 1. Entitas: `users`

Tabel ini menyimpan data pengguna utama aplikasi, seperti pelayan, koordinator, dan admin yang memiliki akses login. Tabel ini tampaknya merupakan tabel pengguna yang lebih modern dan terintegrasi dengan fitur-fitur baru.

**Atribut/Kolom:**

*   `id`: Primary Key (BigInt, Unsigned)
*   `old_api_id`: ID unik dari API lama untuk sinkronisasi (BigInt, Unsigned, Nullable)
*   `name`: Nama pengguna (String)
*   `email`: Alamat email pengguna, harus unik (String)
*   `no_hp`: Nomor Handphone pengguna untuk notifikasi WA (String, Nullable)
*   `komsel_id`: ID komsel tempat pengguna berada (BigInt, Unsigned, Nullable)
*   `password`: Password yang sudah di-hash (String, Nullable)
*   `is_coordinator`: Flag boolean jika pengguna adalah seorang koordinator (Boolean, default: false)
*   `roles`: Menyimpan peran pengguna dalam format JSON (JSON, Nullable)
*   `status`: Status akun pengguna, misal 'aktif' (String, default: 'aktif')
*   `api_token`: Token untuk otentikasi API (Text, Nullable)
*   `scheduling_unlock_until`: Batas waktu izin untuk input jadwal di luar jadwal (Datetime, Nullable)
*   `remember_token`: Token untuk fitur "Ingat Saya" (String)
*   `timestamps`: `created_at` dan `updated_at`

**Kode Migrasi:**

*File: `database/migrations/0001_01_01_000003_create_users_table.php`*
```php
Schema::create('users', function (Blueprint $table) {
    $table->id(); // ID lokal
    $table->unsignedBigInteger('old_api_id')->unique()->nullable(); // ID dari API lama
    
    $table->string('name');
    $table->string('email')->unique();
    $table->string('password')->nullable(); // Dibuat nullable, kita tidak pakai
    $table->json('roles')->nullable();      // Untuk menyimpan roles dari API
    $table->string('status')->default('aktif'); // <-- KOLOM YANG HILANG
    $table->text('api_token')->nullable(); // Untuk menyimpan token (jika API nanti diamankan)

    $table->rememberToken();
    $table->timestamps();
});
```
*File Tambahan:*
- `2025_11_18_130621_add_is_coordinator_to_users_table.php`: Menambah `is_coordinator`.
- `2025_11_29_121817_add_scheduling_unlock_to_users_table.php`: Menambah `scheduling_unlock_until`.
- `2025_11_30_133650_add_no_hp_to_users_table.php`: Menambah `no_hp`.
- `2025_11_30_145337_add_komsel_id_to_users_table.php`: Menambah `komsel_id`.

---

## 2. Entitas: `user_kairos`

Tabel ini tampaknya merupakan versi awal atau tabel "warisan" untuk data jemaat yang disinkronkan dari sistem lama. Tabel ini masih digunakan, terutama untuk relasi absensi.

**Atribut/Kolom:**

*   `id`: Primary Key (BigInt, Unsigned)
*   `origin_id`: ID asli dari aplikasi lama untuk sinkronisasi (BigInt, Unsigned, Unique, Nullable)
*   `email`: Alamat email (String, Nullable)
*   `password`: Password yang sudah di-hash (String)
*   `nama`: Nama jemaat (String)
*   `komsel_id`: ID komsel tempat jemaat berada (BigInt, Unsigned, Nullable)
*   `status`: Status jemaat, misal 'aktif' (String, default: 'aktif')
*   `roles`: Menyimpan peran dalam format JSON (JSON, Nullable)
*   `remember_token`: Token untuk fitur "Ingat Saya" (String)
*   `timestamps`: `created_at` dan `updated_at`

**Kode Migrasi:**

*File: `database/migrations/2025_09_19_065246_create_user_kairos_table.php`*
```php
Schema::create('user_kairos', function (Blueprint $table) {
    $table->id();
    $table->unsignedBigInteger('origin_id')->unique()->nullable();
    $table->string('email')->nullable();
    $table->string('password');
    $table->string('nama');
    $table->unsignedBigInteger('komsel_id')->nullable(); 
    $table->string('status')->default('aktif');
    $table->rememberToken();
    $table->timestamps();
});
```
*File Tambahan:*
- `2025_11_06_115648_add_roles_column_to_user_kairos_table.php`: Menambah `roles`.

---

## 3. Entitas: `schedules` (Jadwal)

Menyimpan data jadwal komsel yang dibuat oleh para *leader*.

**Atribut/Kolom:**

*   `id`: Primary Key (BigInt, Unsigned)
*   `komsel_id`: ID komsel dari API lama (BigInt, Unsigned)
*   `day_of_week`: Hari pelaksanaan komsel (String)
*   `time`: Waktu pelaksanaan komsel (Time)
*   `location`: Lokasi pelaksanaan komsel (String)
*   `description`: Deskripsi tambahan (Text, Nullable)
*   `status`: Status jadwal, misal 'Menunggu' (String, default: 'Menunggu')
*   `timestamps`: `created_at` dan `updated_at`

**Kode Migrasi:**

*File: `database/migrations/2025_10_21_053321_create_schedules_table.php`*
```php
Schema::create('schedules', function (Blueprint $table) {
    $table->id();
    $table->unsignedBigInteger('komsel_id'); 
    $table->string('day_of_week');
    $table->time('time');
    $table->string('location');
    $table->text('description')->nullable();
    $table->string('status')->default('Menunggu');
    $table->timestamps();
});
```

---

## 4. Entitas: `attendances` (Absensi)

Tabel pivot yang mencatat kehadiran jemaat (`user_kairos`) pada sebuah jadwal komsel (`schedules`).

**Atribut/Kolom:**

*   `id`: Primary Key (BigInt, Unsigned)
*   `schedule_id`: Foreign Key ke `schedules.id` (BigInt, Unsigned)
*   `user_kairos_id`: Foreign Key ke `user_kairos.id` (BigInt, Unsigned)
*   `timestamps`: `created_at` dan `updated_at`

**Kode Migrasi:**

*File: `database/migrations/2025_10_28_104350_create_attendances_table.php`*
```php
Schema::create('attendances', function (Blueprint $table) {
    $table->id();
    $table->foreignId('schedule_id')->constrained('schedules')->onDelete('cascade');
    $table->foreignId('user_kairos_id')->constrained('user_kairos')->onDelete('cascade');
    $table->timestamps();
});
```

---

## 5. Entitas: `guest_attendances` (Absensi Tamu)

Mencatat kehadiran tamu (yang tidak terdaftar di `user_kairos`) pada sebuah jadwal komsel.

**Atribut/Kolom:**

*   `id`: Primary Key (BigInt, Unsigned)
*   `schedule_id`: Foreign Key ke `schedules.id` (BigInt, Unsigned)
*   `name`: Nama tamu (String)
*   `timestamps`: `created_at` dan `updated_at`

**Kode Migrasi:**

*File: `database/migrations/2025_10_28_105152_create_guest_attendances_table.php`*
```php
Schema::create('guest_attendances', function (Blueprint $table) {
    $table->id();
    $table->foreignId('schedule_id')->constrained('schedules')->onDelete('cascade');
    $table->string('name');
    $table->timestamps();
});
```

---

## 6. Entitas: `kunjungans` (Kunjungan)

Mencatat data kunjungan pastoral yang dilakukan oleh seorang PIC (dari tabel `users`) kepada seorang anggota jemaat.

**Atribut/Kolom:**

*   `id`: Primary Key (BigInt, Unsigned)
*   `pic_id`: Foreign Key ke `users.id` (PIC yang melakukan kunjungan)
*   `member_id`: ID Jemaat dari API lama (BigInt, Unsigned)
*   `nama_anggota_snapshot`: Salinan nama jemaat untuk arsip (String)
*   `tanggal`: Waktu kunjungan (DateTime)
*   `jenis_kunjungan`: Jenis kunjungan, misal 'Pastoral', 'Sakit' (String)
*   `catatan`: Catatan dari kunjungan (Text, Nullable)
*   `photo_path`: Path foto bukti kunjungan (String, Nullable)
*   `status`: Status kunjungan, misal 'Terjadwal', 'Selesai' (String)
*   `timestamps`: `created_at` dan `updated_at`

**Kode Migrasi:**

*File: `database/migrations/2025_11_21_020421_create_kunjungans_table.php`*
```php
Schema::create('kunjungans', function (Blueprint $table) {
    $table->id();
    $table->unsignedBigInteger('pic_id'); // ID User (Leader) yang login
    $table->unsignedBigInteger('member_id'); // ID Jemaat dari API Lama
    $table->string('nama_anggota_snapshot');
    $table->dateTime('tanggal');
    $table->string('jenis_kunjungan');
    $table->text('catatan')->nullable();
    $table->string('status')->default('Terjadwal');
    $table->timestamps();

    $table->foreign('pic_id')->references('id')->on('users')->onDelete('cascade');
});
```
*File Tambahan:* `2025_11_21_032147_add_photo_to_kunjungans_table.php` menambah `photo_path`.

---

## 7. Entitas: `oikos_visits` (Kunjungan OIKOS)

Mencatat jadwal dan laporan kunjungan OIKOS (program penjangkauan).

**Atribut/Kolom:**

*   `id`: Primary Key
*   `oikos_name`: Nama target OIKOS (String)
*   `pelayan_user_id`: Foreign Key ke `users.id` (Pelayan yang ditugaskan)
*   `jemaat_id`: ID jemaat dari API (BigInt, Nullable)
*   `start_date`, `end_date`: Periode penjangkauan (Date)
*   `status`: Status kunjungan, misal 'Direncanakan' (String)
*   `...`: Kolom-kolom laporan lainnya (`realisasi_date`, `is_doa_5_jari`, `catatan`, dll.)
*   `timestamps`: `created_at` dan `updated_at`

**Kode Migrasi:**

*File: `database/migrations/2025_11_07_014850_create_oikos_visits_table.php`*
```php
Schema::create('oikos_visits', function (Blueprint $table) {
    $table->id();
    $table->string('oikos_name'); 
    $table->foreignId('pelayan_user_id')
          ->nullable()
          ->constrained('users')
          ->onDelete('set null');
    $table->unsignedBigInteger('jemaat_id')->nullable();
    $table->date('start_date');
    $table->date('end_date');
    $table->string('status')->default('Direncanakan');
    // ... Kolom laporan lainnya
    $table->text('catatan')->nullable();
    $table->timestamps();
});
```

---

## 8. Entitas: `notifications` (Notifikasi)

Menyimpan notifikasi yang akan ditampilkan kepada pengguna.

**Atribut/Kolom:**

*   `id`: Primary Key
*   `user_id`: Foreign Key ke `users.id` (Penerima notifikasi)
*   `title`, `message`, `type`: Detail notifikasi (String)
*   `action_url`: Link tujuan saat notifikasi diklik (String, Nullable)
*   `is_read`: Status dibaca (Boolean)
*   `read_at`: Waktu dibaca (Timestamp, Nullable)
*   `timestamps`: `created_at` dan `updated_at`

**Kode Migrasi:**

*File: `database/migrations/2025_11_29_121848_create_notifications_table.php`*
```php
Schema::create('notifications', function (Blueprint $table) {
    $table->id();
    $table->foreignId('user_id')->constrained('users')->onDelete('cascade');
    $table->string('title');
    $table->text('message');
    $table->string('type')->default('info');
    $table->string('action_url')->nullable(); 
    $table->boolean('is_read')->default(false);
    $table->timestamp('read_at')->nullable();
    $table->timestamps();
});
```
---

## Ringkasan Relasi

*   **`users` (One-to-Many):**
    *   Satu `User` bisa memiliki banyak `Notification` (`users.id` -> `notifications.user_id`).
    *   Satu `User` (PIC) bisa melakukan banyak `Kunjungan` (`users.id` -> `kunjungans.pic_id`).
    *   Satu `User` (Pelayan) bisa ditugaskan ke banyak `OikosVisit` (`users.id` -> `oikos_visits.pelayan_user_id`).

*   **`schedules` (One-to-Many):**
    *   Satu `Schedule` bisa memiliki banyak `Attendance` (absensi jemaat terdaftar) (`schedules.id` -> `attendances.schedule_id`).
    *   Satu `Schedule` bisa memiliki banyak `GuestAttendance` (absensi tamu) (`schedules.id` -> `guest_attendances.schedule_id`).

*   **`user_kairos` (Many-to-Many dengan `schedules` melalui `attendances`):**
    *   Seorang `user_kairos` bisa hadir di banyak `schedules`, dan satu `schedule` bisa dihadiri banyak `user_kairos`. Relasi ini dijembatani oleh tabel `attendances`.

*   **`notifications`, `kunjungans`, `oikos_visits` (Many-to-One):**
    *   Banyak `Notification`, `Kunjungan`, atau `OikosVisit` dimiliki oleh satu `User`.
