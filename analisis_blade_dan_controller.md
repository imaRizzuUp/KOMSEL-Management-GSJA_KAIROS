# Analisis Detail File Blade dan Controller

Dokumen ini memberikan rincian tentang setiap file view Blade, hubungannya dengan Controller, serta teknologi frontend yang digunakan.

---

## 1. Layout Utama

### `resources/views/layouts/app.blade.php`

Ini adalah file layout induk yang menjadi kerangka untuk hampir semua halaman setelah pengguna login.

*   **Struktur:**
    *   Mendefinisikan `<html>`, `<head>`, dan `<body>` utama.
    *   Menggunakan `@yield('konten')` untuk menyisipkan konten dari view anak.
    *   Memiliki sidebar navigasi untuk desktop dan mobile (Offcanvas).
    *   Memiliki header/navbar atas yang berisi judul halaman dinamis (`$title`) dan dropdown profil pengguna.
    *   Menyertakan `@stack('styles')` dan `@stack('scripts')` untuk memungkinkan view anak menambahkan CSS atau JS kustom.

*   **Controller Terkait:**
    *   Tidak terhubung ke satu controller spesifik. File ini digunakan oleh hampir semua metode controller yang me-render view yang memerlukan layout (misalnya, `DashboardController`, `KomselController`, dll).

*   **Teknologi & Ketergantungan:**
    *   **Bootstrap 5.3.3:** Digunakan sebagai framework CSS utama untuk layout, komponen (kartu, tombol, modal), dan sistem grid.
    *   **Bootstrap Icons 1.11.3:** Untuk ikon di seluruh aplikasi.
    *   **Google Fonts (Inter):** Font utama yang digunakan.
    *   **Chart.js:** Pustaka untuk membuat grafik, di-include di layout utama dan digunakan oleh beberapa view anak.
    *   **JavaScript Kustom:**
        *   **Theme Switcher (Dark/Light Mode):** Terdapat logika JS yang cukup kompleks untuk menangani pergantian tema dan menyimpannya di `localStorage`.
        *   **Logika Notifikasi:** Terdapat skrip untuk menangani interaksi pada dropdown notifikasi, seperti menandai notifikasi sebagai "telah dibaca" secara dinamis melalui request `fetch` ke endpoint `/notifications/mark-read/{id}`.

---

## 2. Halaman Otentikasi

File-file ini berdiri sendiri dan tidak menggunakan `layouts/app.blade.php`.

*   **Controller:** `App\Http\Controllers\AuthController`

### `resources/views/Auth/login.blade.php`
*   **Tujuan:** Menampilkan formulir login.
*   **Controller & Metode:** `AuthController@login`
*   **Teknologi:** Menggunakan Bootstrap CSS dan JS kustom untuk UI dan theme switcher, mirip dengan `app.blade.php` tapi dalam lingkup satu halaman. Form-nya mengirim data ke `route('autentikasi')` yang ditangani oleh `AuthController@authenticate`.

### `resources/views/Auth/signUp.blade.php`
*   **Tujuan:** Menampilkan formulir pendaftaran. (Tampak sangat dasar dan mungkin tidak digunakan secara aktif).
*   **Controller & Metode:** `AuthController@signup`
*   **Teknologi:** HTML dasar tanpa styling CSS framework. Form-nya mengirim data ke `route('register')` yang ditangani oleh `AuthController@register`. Metode `register` ini sendiri hanya me-redirect kembali ke halaman login, mengindikasikan fitur ini belum diimplementasikan sepenuhnya.

---

## 3. Halaman Dashboard

Halaman-halaman ini menampilkan ringkasan data berdasarkan peran pengguna.

*   **Controller:** `App\Http\Controllers\DashboardController`

### Logika Routing di Controller
Metode `DashboardController@index` bertindak sebagai "router". Metode ini pertama kali mengidentifikasi peran pengguna (Gembala, Koordinator, Leader, atau Jemaat Biasa) dan kemudian memanggil metode internal yang sesuai untuk me-render view yang relevan (`dashboardGembala`, `dashboardKordinator`, `dashboardJemaat`, atau `dashboardLeader`).

### `resources/views/dashboard/dashboard.blade.php`
*   **Tujuan:** Dashboard utama untuk **Leader/Partner/OTR**.
*   **Controller & Metode:** `DashboardController@dashboardLeader`
*   **Relasi Blade:** `@extends('layouts.app')`
*   **Data & Tampilan:**
    *   Menampilkan kartu KPI (Total Anggota, OIKOS Bulan Ini, Total KOMSEL, Rata-rata Hadir).
    *   Menampilkan daftar anggota yang berulang tahun bulan ini, dengan link untuk membuat kunjungan HUT.
    *   Menampilkan daftar jadwal ibadah yang akan datang.
    *   Menampilkan notifikasi jika ada laporan OIKOS yang perlu direvisi.
    *   Menggunakan data yang di-pass seperti `$totalAnggota`, `$oikosBulanIni`, `$birthdayMembers`, `$upcomingSchedules`.

### `resources/views/dashboard/gembala.blade.php`
*   **Tujuan:** Dashboard level tertinggi untuk **Gembala (Super Admin)**.
*   **Controller & Metode:** `DashboardController@dashboardGembala`
*   **Relasi Blade:** `@extends('layouts.app')`
*   **Data & Tampilan:**
    *   Fokus pada struktur organisasi.
    *   Menampilkan kartu statistik global (Total Jiwa, Komsel Aktif, Leaders, Total OIKOS).
    *   Menampilkan kartu highlight untuk Koordinator Utama.
    *   Menampilkan daftar nama dalam kolom-kolom terpisah untuk Leaders, Partners, dan Orang Tua Rohani (OTR).
    *   Menggunakan data seperti `$coordinator`, `$leaders`, `$partners`, `$otr`.

### `resources/views/dashboard/kordinator.blade.php`
*   **Tujuan:** Dashboard untuk **Koordinator**.
*   **Controller & Metode:** `DashboardController@dashboardKordinator`
*   **Relasi Blade:** `@extends('layouts.app')`
*   **Data & Tampilan:**
    *   Fokus pada monitoring dan evaluasi kinerja semua komsel.
    *   Menampilkan KPI global.
    *   Menampilkan chart "Tren Kehadiran Global" menggunakan Chart.js (`<canvas id="coordChart">`).
    *   Menampilkan tabel "Monitoring Kesehatan Komsel" yang berisi data performa setiap komsel (rata-rata hadir, rate, dll).
    *   Menggunakan data seperti `$monitoringData`, `$attendanceChartLabels`, `$attendanceChartData`.

### `resources/views/dashboard/jemaat.blade.php`
*   **Tujuan:** Dashboard paling sederhana untuk **Jemaat Biasa**.
*   **Controller & Metode:** `DashboardController@dashboardJemaat`
*   **Relasi Blade:** `@extends('layouts.app')`
*   **Data & Tampilan:**
    *   Menampilkan informasi "Rumah Rohani" mereka.
    *   Menampilkan nama KOMSEL mereka, nama Leader yang menggembalakan, dan jadwal ibadah berikutnya.
    *   Menampilkan "empty state" jika jemaat belum tergabung dalam komsel.
    *   Menggunakan data seperti `$myKomsel`, `$myLeader`, `$nextSchedule`.

---

## 4. Manajemen KOMSEL

*   **Controller:** `App\Http\Controllers\KomselController`

### `resources/views/KOMSEL/daftarKomsel.blade.php`
*   **Tujuan:** Menampilkan daftar semua anggota jemaat yang bisa di-filter berdasarkan komsel.
*   **Controller & Metode:** `KomselController@daftar`
*   **Relasi Blade:** `@extends('layouts.app')`
*   **Teknologi:** Menggunakan Select2 untuk dropdown filter komsel yang interaktif dan searchable. Data untuk tabel di-pass dari controller sebagai variabel `$users` dan `$komsels`.

### `resources/views/KOMSEL/jadwalKomsel.blade.php`
*   **Tujuan:** Menampilkan, membuat, mengedit, dan menghapus jadwal ibadah komsel.
*   **Controller & Metode:** `KomselController@jadwal`
*   **Relasi Blade:** `@extends('layouts.app')`
*   **Teknologi:**
    *   Menampilkan tabel berisi daftar jadwal.
    *   Menggunakan Modal Bootstrap untuk form tambah dan edit jadwal.
    *   Menggunakan JavaScript untuk mengisi data form edit saat tombol "Edit" diklik.
    *   Terdapat form hapus yang tersembunyi untuk setiap jadwal.
    *   Terdapat logika kompleks untuk absensi di mana data anggota diambil melalui `fetch` API ke `api/komsel/{komselId}/users`.

---

## 5. Manajemen OIKOS

*   **Controller:** `App\Http\Controllers\OikosController`

### `resources/views/OIKOS/formInput.blade.php`
*   **Tujuan:** Form untuk membuat jadwal kunjungan OIKOS baru.
*   **Controller & Metode:** `OikosController@formInputOikos`
*   **Relasi Blade:** `@extends('layouts.app')`
*   **Teknologi:** Menggunakan JavaScript untuk menampilkan/menyembunyikan input nama OIKOS (manual) atau dropdown anggota jemaat (terdaftar) berdasarkan pilihan. Terdapat logika UI yang menampilkan peringatan jika form dibuka di luar hari yang diizinkan.

### `resources/views/OIKOS/daftarOIKOS.blade.php`
*   **Tujuan:** Menampilkan daftar semua jadwal OIKOS yang sudah dibuat.
*   **Controller & Metode:** `OikosController@daftarOikos`
*   **Relasi Blade:** `@extends('layouts.app')`
*   **Teknologi:**
    *   Menampilkan tabel OIKOS.
    *   Banyak menggunakan Modal Bootstrap untuk berbagai aksi:
        *   Modal untuk mengisi laporan (`laporanModal`).
        *   Modal untuk melihat detail laporan (`detailModal`).
        *   Modal untuk mendelegasikan tugas (`delegasiModal`).
        *   Modal untuk meminta revisi (`revisiModal`).
    *   Sangat bergantung pada JavaScript dan `fetch` API untuk mengisi data modal secara dinamis dengan mengambil data dari `api/oikos-visits/{oikosVisit}`.

---

## 6. Manajemen Kunjungan

*   **Controller:** `App\Http\Controllers\KunjunganController`

### `resources/views/kunjungan/index.blade.php`
*   **Tujuan:** Menampilkan daftar kunjungan pastoral.
*   **Controller & Metode:** `KunjunganController@index`
*   **Relasi Blade:** `@extends('layouts.app')`
*   **Teknologi:** Menampilkan tabel kunjungan. Terdapat modal Bootstrap untuk mengisi atau melihat laporan hasil kunjungan.

### `resources/views/kunjungan/create.blade.php`
*   **Tujuan:** Form untuk mencatat kunjungan pastoral baru.
*   **Controller & Metode:** `KunjunganController@create`
*   **Relasi Blade:** `@extends('layouts.app')`
*   **Teknologi:** Menggunakan Select2 untuk dropdown "Pilih Anggota" dan "Pilih PIC" (jika admin) agar mudah dicari.

---

## 7. Laporan Statistik

*   **Controller:** `App\Http\Controllers\StatistikController`

### `resources/views/statistik.blade.php`
*   **Tujuan:** Menampilkan dashboard statistik performa komsel dan OIKOS.
*   **Controller & Metode:** `StatistikController@statistik`
*   **Relasi Blade:** `@extends('layouts.app')`
*   **Teknologi:**
    *   Memiliki form filter berdasarkan bulan dan tahun.
    *   Menampilkan berbagai chart menggunakan Chart.js:
        *   `kehadiranPerKomselChart` (Bar chart)
        *   `topAttendeesChart` (Bar chart)
        *   `attendanceTrendChart` (Line chart)
    *   Data untuk chart di-pass dari controller sebagai variabel JSON (`$komselChartLabels`, `$komselChartData`, dll).

### `resources/views/statistik_pdf.blade.php`
*   **Tujuan:** Template khusus untuk generate laporan PDF.
*   **Controller & Metode:** `StatistikController@exportPdf`
*   **Relasi Blade:** Tidak menggunakan layout, ini adalah file HTML mandiri.
*   **Teknologi:** Didesain dengan styling CSS `inline` atau di dalam tag `<style>` agar kompatibel dengan library `barryvdh/laravel-dompdf`. Menampilkan data statistik dalam format tabel sederhana yang ringkas untuk dicetak.

---

## 8. Halaman Lainnya

### `resources/views/welcome.blade.php`
*   **Tujuan:** Halaman selamat datang publik.
*   **Controller & Metode:** Ditampilkan oleh sebuah Closure Route di `routes/web.php`.
*   **Teknologi:** Halaman statis sederhana.

### `resources/views/errors/500.blade.php`
*   **Tujuan:** Halaman error kustom untuk server error (500).
*   **Controller & Metode:** Ditampilkan secara otomatis oleh Laravel saat terjadi exception yang tidak tertangani.
*   **Teknologi:** Halaman statis.
