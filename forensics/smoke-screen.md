# 2. Smoke Screen

### 2.1 Informasi Challenge

- Nama Challenge: Smoke Screen.

- Kategori: Forensics / Steganography.

- Poin: 1000.

- Author: Gojo Satoru.

- Bahan: smoke-screen.zip (cocoon_1.bin sampai cocoon_5.bin).

- Teknik: identifikasi berkas anomali + decode Base64 + dekompresi zlib.

- Flag: Kaito{5m0k3_4nd_m1rr0r5_1l1u510n_0f_3sc4p3}.

### 2.2 Deskripsi Soal

Kaito Kid escaped the rooftop inside a cocoon of smoke — one of five charges deployed that night. Four are empty misdirection. One carries what he left behind. Of the five cocoon files, only one holds the flag; the other four are decoys.

### 2.3 Langkah-Langkah

#### Langkah 1 — Ekstrak Berkas dan Periksa Ukuran

Ekstrak arsip lalu bandingkan ukuran kelima berkas:

```
unzip smoke-screen.zipls -la cocoon_*.bin# cocoon_1.bin  132 byte# cocoon_2.bin  132 byte# cocoon_3.bin   92 byte  <-- berbeda# cocoon_4.bin  132 byte# cocoon_5.bin  132 byte
```

Empat berkas berukuran 132 byte, sedangkan cocoon_3.bin hanya 92 byte. Perbedaan ukuran ini merupakan petunjuk pertama — berkas yang ukurannya berbeda kemungkinan memuat data yang sebenarnya.

#### Langkah 2 — Periksa Tipe dan String Berkas

Periksa tipe dan string yang terbaca dari kelima berkas:

file cocoon_*.bin# semuanya: data (binary mentah), tanpa header khususstrings cocoon_*.bin# cocoon_1,2,4,5: NOISENOISENOISE... (berulang)# cocoon_3.bin: eJzzdvQM8a9Ozk/Oz8+Lz0+LL87Nz06NT8/MSUlNiU8tTk4sSK0FAP8lDjw=

Berkas cocoon_1, 2, 4, dan 5 hanya berisi string NOISE yang berulang (decoy), sedangkan cocoon_3.bin berisi string Base64 yang diawali eJ — ciri khas data yang dikompresi dengan zlib.

#### Langkah 3 — Decode Base64 dan Decompress zlib

Uraikan string Base64 lalu dekompresi hasil binernya:

```
python3 << 'EOF'import base64, zlibb64 = "eJzzdvQM8a9Ozk/Oz8+Lz0+LL87Nz06NT8/MSUlNiU8tTk4sSK0FAP8lDjw="print(zlib.decompress(base64.b64decode(b64)).decode())EOF# Kaito{5m0k3_4nd_m1rr0r5_1l1u510n_0f_3sc4p3}
```

Keluaran langsung menampilkan flag yang valid.

### 2.4 Analisis

Seleksi bekerja melalui dua lapis eliminasi: ukuran berkas menyisihkan empat decoy berukuran identik (132 byte berekor NOISE), sedangkan awalan eJ pada string cocoon_3.bin menandakan kompresi zlib sehingga jalur decode-nya pasti (Base64 lalu zlib, bukan cipher). Struktur ini menjelaskan mengapa cocoon ketiga — sebagaimana petunjuk Conan — bukanlah asap, melainkan pesan.

### 2.5 Flag

Kaito{5m0k3_4nd_m1rr0r5_1l1u510n_0f_3sc4p3}
