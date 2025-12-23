# JWT Token Management System - Kelompok 3
## Expiration, Refresh Token, dan Manajemen Waktu

## Pembahasan Materi

### 1. Perbandingan `exp`, `iat`, dan `nbf`

JWT (JSON Web Token) memiliki *time-based claims* yang digunakan untuk mengatur masa berlaku token dan menjaga keamanan sistem autentikasi.

#### `exp` (Expiration Time)
- **Fungsi**: Menentukan kapan token akan kadaluarsa
- **Kapan Digunakan**: Wajib ada pada setiap token
- **Format**: Unix timestamp
- **Contoh**: `1703095800` (20 Desember 2024 15:30:00)
- **Validasi**: Token ditolak jika `current_time > exp`

#### `iat` (Issued At)
- **Fungsi**: Mencatat waktu pembuatan token
- **Kapan Digunakan**: Audit, tracking login, dan menghitung umur token
- **Format**: Unix timestamp
- **Contoh**: `1703094000` (20 Desember 2024 15:00:00)
- **Kegunaan**: Monitoring aktivitas login dan deteksi token lama

#### `nbf` (Not Before)
- **Fungsi**: Menentukan waktu minimum token boleh digunakan
- **Kapan Digunakan**: Scheduled access dan delayed activation
- **Format**: Unix timestamp
- **Contoh**: `1703094300` (20 Desember 2024 15:05:00)
- **Validasi**: Token ditolak jika `current_time < nbf`

**Contoh Timeline JWT:**
15:00:00 (iat) → Token dibuat
15:05:00 (nbf) → Token mulai bisa digunakan
15:30:00 (exp) → Token expired


---

### 2. Desain Alur Access Token dan Refresh Token

Sistem ini menggunakan **Dual Token Strategy** untuk menyeimbangkan keamanan dan kenyamanan pengguna.

#### Diagram Alur Access Token + Refresh Token
┌─────────────┐
│ User Login  │
└──────┬──────┘
       │
       ▼
┌─────────────────────────────┐
│ Server Mengirim:            │
│ - Access Token (30 detik)   │
│ - Refresh Token (5 menit)   │
└──────┬──────────────────────┘
       │
       ▼
┌─────────────────────────────┐
│ User Mengakses API          │
│ menggunakan Access Token    │
└──────┬──────────────────────┘
       │
       ▼ (Access Token hampir expired)
┌─────────────────────────────┐
│ User Request Refresh Token  │
│ + Verifikasi (HP & Password)│
└──────┬──────────────────────┘
       │
       ▼
┌─────────────────────────────┐
│ Server Mengirim             │
│ Access Token Baru (30 detik)│
└──────┬──────────────────────┘
       │
       ▼
       Proses Berulang
       │
       ▼ (Refresh Token expired)
┌─────────────────────────────┐
│ Refresh Token Kadaluarsa    │
│ User Harus Login Ulang      │
└─────────────────────────────┘

#### Implementasi Token pada Aplikasi

**Access Token**
- Lifetime: 30 detik
- Fungsi: Akses ke protected API/resource
- Penyimpanan: Memory (JavaScript variable)
- Keamanan: Short-lived untuk meminimalkan risiko

**Refresh Token**
- Lifetime: 5 menit
- Fungsi: Menghasilkan access token baru
- Penyimpanan: Server-side (cookie HttpOnly / memory pada demo)
- Rotation: Refresh token lama digantikan token baru

**Keuntungan Desain**
- Access token pendek meningkatkan keamanan
- User tidak perlu login ulang setiap token habis
- Refresh token dapat di-revoke kapan saja
- Aktivitas token dapat diaudit

---

### 3. Skenario Tanpa Expiration (`exp`)

#### Skenario 1: Token Dicuri (Laptop Hilang)
Situasi: Laptop mahasiswa hilang dan token tersimpan di browser.
- Tanpa expiration: Token dapat digunakan selamanya
- Dengan expiration: Token otomatis tidak valid dalam 30 detik

Risiko:
- Data akademik diubah
- Nilai dimanipulasi
- Identitas dicuri
- Kerugian bersifat permanen

---

#### Skenario 2: WiFi Public Attack (MITM)
Situasi: User menggunakan jaringan WiFi publik yang tidak aman.
- Tanpa expiration: Attacker bisa mengakses sistem kapan saja
- Dengan expiration: Token cepat kadaluarsa dan tidak dapat digunakan

Risiko:
- Account takeover
- Transaksi tidak sah
- Pencurian data pribadi
- Kerusakan reputasi sistem

---

#### Skenario 3: Employee Turnover
Situasi: Admin sistem resign atau diberhentikan.
- Tanpa expiration: Token admin lama masih dapat digunakan
- Dengan expiration: Token expired dan harus login ulang

Risiko:
- Sabotase sistem
- Data breach
- Backdoor access
- Masalah hukum

---

#### Skenario 4: Perubahan Role atau Permission
Situasi: User diturunkan dari admin menjadi user biasa.
- Tanpa expiration: Token lama masih memiliki hak admin
- Dengan expiration: Token baru mencerminkan role yang benar

Risiko:
- Privilege escalation
- Unauthorized actions
- Bypass authorization
- Kegagalan audit keamanan

---

### Dampak Bisnis dan Organisasi

**Tanpa Expiration**
- Tidak ada forced re-authentication
- Database blacklist terus membesar
- Masalah compliance (GDPR, ISO 27001)
- Audit trail tidak akurat
- Recovery dari insiden keamanan sulit

**Dengan Expiration**
- Refresh keamanan otomatis
- Dampak kebocoran token terbatas
- Forced credential verification
- Lebih mudah memenuhi compliance
- Postur keamanan sistem lebih baik

---

## Kesimpulan

1. `exp`, `iat`, dan `nbf` adalah claim waktu yang saling melengkapi
2. Strategi access token dan refresh token menyeimbangkan keamanan dan UX
3. Token tanpa expiration merupakan risiko keamanan serius
4. Best practice JWT adalah selalu menggunakan expiration dan refresh token

---

## Rekomendasi Konfigurasi Token

| Token Type | Development | Production | Critical Systems |
|------------|-------------|------------|------------------|
| Access     | 30s – 1m    | 15 – 30m   | 5 – 10m          |
| Refresh    | 5m          | 7 – 30d    | 1 – 7d           |

---

© 2025 Kelompok 3 – JWT Security Project
