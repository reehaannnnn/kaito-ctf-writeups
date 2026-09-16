# 2. The Omega Trigger

### 2.1 Informasi Challenge

- Nama Challenge: The Omega Trigger (Omega Device).

- Kategori: CRYPTO (Matsumoto–Imai, multivariate public-key).

- Target: 31.97.37.38:1339 (oracle transformasi MI).

- Teknik: differential attack atas oracle (1190 query) → pulihkan secret_x = 0xa1949da.

- Flag: Kaito{0m3g4_d3v1c3_matsumoto_imai_patarin_differential}.

### 2.2 Deskripsi Soal

The challenge uses the Matsumoto–Imai (MI) scheme, a multivariate public-key construction. The server exposes an oracle that accepts queries and returns transformed outputs. The source shows the main goal is to recover the internal secret of the MI implementation through a structural/differential weakness in the oracle. The end target is the secret value secret_x, which then yields the flag.

### 2.3 Langkah-Langkah

#### Langkah 1 — Recon Oracle (input → output)

Karena challenge berbasis oracle, pendekatan pertama adalah memetakan pola input -> oracle -> output dan membandingkan bagaimana output berubah ketika input dimodifikasi sedikit. Kuncinya: transformasi MI tidak berperilaku seperti fungsi black-box acak — ada struktur aljabar yang dapat dieksploitasi.

```
nc 31.97.37.38 1339# uji: kirim x, catat f(x); kirim x+delta, catat f(x+delta)
```

#### Langkah 2 — Differential Attack (1190 query)

Alih-alih brute-force secret_x, kirim banyak input yang berhubungan secara terstruktur lalu analisis selisih f(x + Δ) − f(x); perbedaan tersebut membocorkan informasi transformasi internal. Eksploit menjalankan differential queries otomatis hingga terkumpul 1190 pasangan input/output, yang kemudian membentuk sistem persamaan untuk mempersempit kemungkinan secret.

```
python3 exploit_mi.py  # differential queries otomatis -> 1190 query
```

#### Langkah 3 — Pulihkan secret_x dan Ambil Flag

Eksploit menghasilkan nilai secret internal secret_x = 0xa1949da — milestone yang menuntaskan bagian kriptografi (MI oracle → differential queries → recover algebraic structure → recover secret_x → decrypt/derive challenge secret). Dengan secret tersebut flag diturunkan:

```
python3 exploit_mi.py  # lanjutan: derive secret -> flag# Kaito{0m3g4_d3v1c3_matsumoto_imai_patarin_differential}
```

### 2.4 Analisis

Serangan berhasil karena MI memiliki struktur diferensial yang tidak dimiliki fungsi acak: selisih keluaran atas input berkorelasi mengungkap parameter internal sedikit demi sedikit. Dengan 1190 query, sistem persamaan menjadi cukup untuk mengisolasi secret_x; setelah itu flag hanya persoalan derivasi. Pelajaran pertahanannya: oracle kriptografi harus membatasi laju/kuota query dan memakai konstruksi yang tahan analisis diferensial (seperti yang mematahkan MI aslinya, Patarin).

### 2.5 Flag

Kaito{0m3g4_d3v1c3_matsumoto_imai_patarin_differential}
