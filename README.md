# Computer-Networking

# Proyek Ping Python di Raspberry Pi

Proyek ini adalah implementasi dari program ping menggunakan Python yang dapat dijalankan pada Raspberry Pi dengan sistem operasi Linux. Program ini mengirimkan paket ICMP (ping) ke alamat IP tujuan, dan menghitung waktu round-trip dari paket yang dikirim.

## Persyaratan

- Raspberry Pi dengan sistem operasi berbasis Linux (misalnya Raspberry Pi OS).
- Python 3.x terinstal pada Raspberry Pi.
- Hak akses root atau kemampuan untuk menjalankan socket raw (untuk mengirim paket ICMP).
- Modul Python berikut:
  - `socket` (bawaan Python)
  - `os` (bawaan Python)
  - `sys` (bawaan Python)
  - `struct` (bawaan Python)
  - `time` (bawaan Python)
  - `select` (bawaan Python)

## Langkah-langkah Menjalankan Proyek

### 1. Persiapkan Raspberry Pi

Sebelum menjalankan script, pastikan Raspberry Pi Anda sudah terinstal Python 3. Anda dapat mengeceknya dengan perintah berikut:

```bash
python3 --version
