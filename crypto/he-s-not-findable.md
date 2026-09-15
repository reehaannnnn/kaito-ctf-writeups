# 1. He’s Not Findable

### 1.1 Informasi Challenge

- Nama Challenge: He’s Not Findable.

- Kategori: Crypto (ECDSA / Lattice attack).

- Koneksi: nc 31.97.37.38 1338 (layanan TCP berpagar Proof-of-Work).

- Bahan: curve.py (parameter secp256k1), server.py.

- Kunci: private key ECDSA d hasil serangan Hidden Number Problem (lattice LLL, n = 42 tanda tangan).

- Flag: Kaito{c3ntr4l_f1n1t3_curv3_hnp_l4tt1c3_c137}.

### 1.2 Deskripsi Soal

The server holds a secret ECDSA private key d on curve secp256k1 and publishes the matching public key. The player may request up to 45 signed portal emissions (arbitrary throwaway messages, each freshly signed). For every signature the top 64 bits and bottom 64 bits of the 256-bit nonce are revealed; only the middle 128 bits stay secret. The goal (option [3], Execute Omega Override) is to produce a valid signature over a fixed challenge string — which requires knowing the private key.

### 1.3 Langkah-Langkah

#### Langkah 1 — Kumpulkan Tanda Tangan + Nonce Bocor

Lewati pagar Proof-of-Work, lalu minta hingga 45 emisi portal. Setiap respons memuat tanda tangan ECDSA (r, s) atas pesan yang diketahui hash-nya (h), beserta high_64 dan low_64 dari nonce ephemeral k:

high_64 = secrets.randbits(64)mid_128 = secrets.randbits(128)low_64  = secrets.randbits(64)k = (high_64 * (1 << 192) + mid_128 * (1 << 64) + low_64) % N# server mengirim: high_64, low_64 (mid_128 tetap rahasia)

Susun A = high_64*2^192 + low_64 (diketahui penuh), B = 2^64 (konstanta), dan x = mid_128 yang tidak diketahui pada rentang [0, 2^128). Dari persamaan ECDSA s*k = h + r*d (mod N), substitusi k = A + B*x lalu penataan ulang menghasilkan relasi linear bentuk Hidden Number Problem: x = c + t*d (mod N), dengan t dan c sepenuhnya dapat dihitung dari data tanda tangan publik.

#### Langkah 2 — Bangun Lattice dan Pulihkan Kunci Privat (LLL)

Dengan n = 42 tanda tangan, bangun matriks integer (n+2)x(n+2): baris 1..n berisi W*N di kolomnya masing-masing (W = 2^128 sebagai batas), baris n+1 berisi W*t_1..W*t_n, 1, 0, dan baris n+2 berisi W*c_1..W*c_n, 0, K (K = 2^128). Kombinasi baris dengan koefisien yang tepat mengevaluasi vektor (mid_1, ..., mid_n, d, K) yang koordinatnya kecil dan seimbang (~2^256 setelah penskalaan) — persis kondisi yang dibutuhkan LLL:

from fpylll import IntegerMatrix, LLLA = IntegerMatrix(dim, dim)# ... isi baris seperti di atas ...LLL.reduction(A)for row in A:    candidate_d = row[n] % N    if (candidate_d * G).x == Pub.x:        d = candidate_d        break

LLL polos (tanpa BKZ) memulihkan kunci privat yang benar pada percobaan sukses pertama; validasi dilakukan pada simulasi lokal sebelum menyerang server live (100% sukses pada banyak uji dengan 40 tanda tangan) dengan memeriksa (d*G).x == Pub.x.

#### Langkah 3 — Tempa Tanda Tangan dan Ambil Flag

Dengan d yang pulih, buat tanda tangan ECDSA standar memakai nonce acak segar atas pesan target tetap "DIMENSION_PRIME_OMEGA_OVERRIDE_TARGET", lalu kirim melalui opsi menu [3]. Server mencetak flag. Ambil tangkapan layar terminal yang menampilkan pengiriman opsi [3] beserta flag-nya.

```
python3 forge.py  # tanda tangani target dengan d, kirim via opsi [3]# Kaito{c3ntr4l_f1n1t3_curv3_hnp_l4tt1c3_c137}
```

### 1.4 Analisis

Kebocoran ini fatal karena mengubah keamanan ECDSA menjadi Hidden Number Problem: 128 dari 256 bit nonce diketahui (64 atas + 64 bawah), sehingga sisa 128 bit yang kecil dapat diselesaikan via reduksi lattice. Penskalaan W = K = 2^128 menjaga seluruh koordinat pada orde yang sama sehingga vektor target menjadi salah satu vektor pendek basis — alasan LLL polos sudah cukup tanpa BKZ. Pelajaran pertahanannya: jangan pernah membocorkan sebagian nonce; gunakan RFC 6979 (nonce deterministik) atau fresh random penuh.

### 1.5 Flag

Kaito{c3ntr4l_f1n1t3_curv3_hnp_l4tt1c3_c137}
