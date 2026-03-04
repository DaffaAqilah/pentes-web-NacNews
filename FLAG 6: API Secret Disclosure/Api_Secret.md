Baik. Berikut versi yang sudah disesuaikan.
Saya tidak menghapus bagian gambar, tetapi menggantinya dengan placeholder `{deskripsi gambar dalam word}` agar bisa kamu tambahkan manual di dokumen.

---

# FLAG [6]: API Secret Disclosure

### Description

Berapa nilai **API_SECRET** yang ditemukan setelah berhasil melewati *upload filter* dan membaca file kredensial di server?

### Hint

💡 Hint Flag 6: Upload File, Case Sensitive

---

## Analysis

Untuk menyelesaikan tantangan ini, kami mengeksploitasi kelemahan pada mekanisme **file upload validation**. Berdasarkan *hint*, kami menduga adanya celah pada pengecekan ekstensi file yang bersifat *case sensitive*.

Sistem hanya melakukan validasi sederhana terhadap ekstensi dan *Content-Type*, tanpa memverifikasi isi file secara menyeluruh. Hal ini membuka peluang untuk melakukan **Upload Filter Bypass**.

Strategi yang digunakan:

* Membuat file *web shell* sederhana berbasis PHP.
* Mengubah ekstensi menjadi kombinasi huruf besar-kecil (misalnya `.PhP`) untuk melewati filter.
* Memodifikasi header `Content-Type` menjadi `image/png`.
* Mengunggah ulang file menggunakan manipulasi request melalui Burp Suite.

Setelah file berhasil diunggah dan tersimpan di direktori `/uploads/`, kami dapat mengeksekusi perintah sistem menggunakan parameter `cmd` dan melakukan enumerasi direktori server.

Dari proses ini ditemukan folder sensitif yang berisi file kredensial internal, termasuk nilai **API_SECRET**.

---

## Solution

### 1. Pembuatan Backdoor Sederhana

Kami membuat file PHP sederhana yang dapat menjalankan perintah sistem melalui parameter URL.

Contoh akses setelah file berhasil diunggah:

```
http://target/uploads/upload_69a7bf050497e_PintuBelakang.PhP?cmd=ls
```

{Screenshot file backdoor sebelum diupload}

---

### 2. Analisis Request Menggunakan Burp Suite

Proses upload dianalisis menggunakan Burp Suite untuk melihat struktur *HTTP request* yang dikirim ke server.

Ditemukan bahwa:

* Validasi ekstensi tidak konsisten.
* Server hanya memeriksa `Content-Type`.
* Tidak ada pembatasan eksekusi file di folder `/uploads/`.

Kami kemudian:

* Mengubah ekstensi file menjadi `.PhP`
* Mengatur `Content-Type: image/png`
* Mengirim ulang request yang telah dimodifikasi

{Screenshot intercept request di Burp Suite}
{Screenshot request setelah dimodifikasi}
{Screenshot response sukses upload}

---

### 3. Enumerasi Direktori Server

Setelah file dapat dieksekusi, kami melakukan enumerasi direktori menggunakan:

```
?cmd=ls ../../
```

Dari hasil tersebut ditemukan folder:

```
Secrets
```

{Screenshot hasil ls ../../}

Kemudian dilakukan eksplorasi lebih lanjut hingga menemukan file:

```
api_credentials.txt
```

{Screenshot isi folder Secrets}
{Screenshot isi file api_credentials.txt}

---

### 4. Flag Ditemukan

Nilai **API_SECRET** yang ditemukan dalam file tersebut adalah:

```
sk_live_pharos_9x7k2m
```

**Bukti Flag Benar:**
`sk_live_pharos_9x7k2m`

{Screenshot bukti flag benar di platform CTF}

---

## Vulnerability Assessment

* **Vulnerability:** Unrestricted File Upload & Remote Code Execution (RCE)
* **Severity:** Critical
* **Impact:** Penyerang dapat mengeksekusi perintah sistem, membaca file sensitif, dan memperoleh kredensial internal.
* **Root Cause:** Validasi ekstensi berbasis *case-sensitive* dan tidak adanya pembatasan eksekusi pada direktori upload.

---

## Saran Rekomendasi Mitigasi

1. **Gunakan Whitelist Extension Validation**
   Validasi ekstensi harus dilakukan setelah dinormalisasi ke lowercase.

2. **Validasi File Signature (Magic Bytes)**
   Jangan hanya mengandalkan ekstensi atau `Content-Type`.

3. **Nonaktifkan Eksekusi di Folder Upload**
   Konfigurasi server agar direktori `/uploads/` tidak dapat mengeksekusi skrip.

4. **Simpan File di Luar Web Root**
   Gunakan mekanisme file handler untuk menampilkan file.

5. **Implementasi Web Application Firewall (WAF)**
   Deteksi dan blokir pola upload mencurigakan.

---

Struktur ini sudah rapi, formal, dan konsisten dengan format FLAG sebelumnya.
Kalau mau, saya bisa bantu buatkan template laporan final agar semua FLAG punya format identik dan siap dikumpulkan.
