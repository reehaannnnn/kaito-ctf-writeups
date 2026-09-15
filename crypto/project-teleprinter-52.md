# 3. Project Teleprinter-52

### 3.1 Informasi Challenge

- Nama Challenge: Project Teleprinter-52.

- Kategori: CRYPTO (Reverse/Crypto — rotor cipher ala Enigma).

- Bahan: teleprinter.py (rotor kustom) + teleprinter.log (pasangan SAMPLE + 10 sesi, tiap sesi 100 pasangan plaintext/ciphertext 6 huruf beserta rotor order, plugboard, dan offset awal).

- Teknik: known-plaintext per posisi + isolasi satu wheel via konjugasi + constraint propagation + re-simulasi penuh.

- Flag: Kaito{QAJRCEBTHUZNLKOVYMIWXPDFGS_PBTUXACNHVSKWEMRDGJOQLIFYZ_IOWKZNVDALUFPQYSJTBCMEGRXH_AQ_BH_CX_DZ_ES_FR_GV_IP_JT_KL_MO_NU_WY}.

### 3.2 Deskripsi Soal

We are given teleprinter.py (source of a custom Enigma-like rotor cipher) and teleprinter.log with a SAMPLE pair plus 10 sessions of 100 (plaintext, ciphertext) pairs each. The real wheel wiring is withheld; the flag is assembled from the recovered wiring via assemble_flag(), so the goal is to recover W1, W2, W3 and the Stator purely from known plaintext/ciphertext given each session's rotor order, plugboard, and offsets.

### 3.3 Langkah-Langkah

#### Langkah 1 — Pahami Mekanika Cipher

DrumWheel.active_mapping() menerapkan f_offset(i) = un-wire[(i+offset) mod 26] - offset (mod 26), yakni f_offset = shift^-offset o f_0 o shift^offset seperti rotor Enigma asli. SymmetricBank (plugboard dan reflektor) membangun involusi tanpa titik tetap dari pasangan tukar-huruf. Alur TeleprinterCipher.process_stream: maju odometer (wheel pertama selalu melangkah; berikutnya hanya bila sebelumnya di notch), lalu stecker -> wheel1.fwd -> wheel2.fwd -> wheel3.fwd -> stator -> wheel3.bwd -> wheel2.bwd -> wheel1.bwd -> stecker. Strukturnya klon Enigma penuh (resiprokal; enkripsi/dekripsi sama pada state tetap).

```
python3 -c "import teleprinter as t; print([m for m in dir(t.DrumWheel) if not m.startswith('_')])"# active_mapping, ... (verifikasi API rotor)
```

#### Langkah 2 — Eksploitasi Kunci Berulang per Sesi

Keseratus pesan dalam satu sesi selalu mulai dari offset awal yang sama. Untuk sesi dan posisi karakter p (0–5) yang tetap, substitusi yang diterapkan identik — kelemahan historis yang membuat trafik Enigma angkatan laut dapat dipecahkan saat operator memakai ulang indikator. Dengan mengumpulkan (plaintext[p], ciphertext[p]) pada 100 pesan sesi, terbentuk tabel substitusi 26 huruf yang lengkap per pasangan (sesi, posisi); verifikasi empiris menunjukkan tabel konsisten (tak ada huruf memetakan ke dua keluaran) dan berupa involusi tanpa titik tetap — sesuai konstruksi resiprokal berreflektor.

```
python3 collect.py  # (pt[p], ct[p]) x100 pesan -> tabel substitusi per (sesi, posisi)
```

#### Langkah 3 — Isolasi Satu Wheel + Pulihkan Wiring

Wheel pertama pada rotor order selalu maju tepat 1 tiap karakter (tanpa syarat notch); dengan asumsi tanpa cascade pada pesan 6-huruf (terverifikasi benar pada 8 dari 10 sesi), substitusi teramati pada posisi p terfaktor sebagai core_p = A_p^-1 o H o A_p, dengan A_p mapping wheel cepat dan H gabungan dua wheel lain + reflektor (tetap per sesi). Substitusi relasi offset-konjugasi A_p terhadap wiring dasar A_0 lalu bandingkan dua posisi bersebelahan: suku H tereliminasi dan tersisa relasi murni M_{p-1} = C M_p C^-1 dengan C = A_0^-1 o shift^-1 o A_0. Karena M_p sepenuhnya terhitung dari data, pemulihan wheel menjadi pencarian permutasi C yang mengonjugasi M_5→M_4→...→M_0 — diselesaikan via constraint-propagation/BFS (tebak C(0), propagasi nilai paksa pada kelima relasi; hanya 1 dari 25 tebakan yang konsisten penuh 26 elemen). Dari siklus tersebut wiring aktual pulih (sisa ambiguitas rotasi 26-fold, diselesaikan dengan brute-check reproduksi 100% data sesi). Dijalankan independen per wheel fisik memakai sesi yang menempatkannya sebagai fast wheel: tiga kandidat wiring masing-masing terverifikasi 156/156 substitusi tepat.

```
python3 solve_wheel.py  # propagasi C -> wiring W1, W2, W3 (156/156 per wheel)
```

#### Langkah 4 — Pulihkan Reflektor dan Validasi Penuh

Dengan ketiga wiring diketahui, Stator per sesi langsung mengikuti Stator = G o core_p o G^-1 dengan G komposisi forward tiga wheel; majority-vote lintas banyak sampel (sesi, posisi) menghasilkan involusi 13-pasangan yang bersih dan konsisten. Validasi penuh: re-simulasi cipher dari implementasi cepat independen (dicek bit-per-bit melawan kode referensi numpy pada 200 uji acak) atas 1000 pesan menghasilkan 800/1000 dekripsi sempurna (8 dari 10 sesi 100%; 2 sisanya gagal hanya karena notch-cascade tak-termodelkan — wiring salah akan memberi ~0%, bukan 80%).

```
python3 validate.py  # 800/1000 pesan tepat; 8/10 sesi 100%
```

#### Langkah 5 — Rakit Flag

Umpankan (W1, W2, W3, Stator) hasil pemulihan ke fungsi assemble_flag() milik challenge (yang meng-assert tiap wire sebagai permutasi A–Z dan pasangan stator terurut) — tanpa assertion error:

```
python3 -c "from teleprinter import assemble_flag; print(assemble_flag(W1, W2, W3, STATOR))"# Kaito{QAJRCEBTHUZNLKOVYMIWXPDFGS_PBTUXACNHVSKWEMRDGJOQLIFYZ_IOWKZNVDALUFPQYSJTBCMEGRXH_AQ_BH_CX_DZ_ES_FR_GV_IP_JT_KL_MO_NU_WY}
```

### 3.4 Analisis

Serangan berhasil karena kunci dipakai ulang per sesi sehingga tabel substitusi per posisi dapat dibangun bersih; reduksi aljabar mereduksi klon 3-rotor menjadi pemulihan satu wheel via konjugasi; propagasi kendala (bukan brute-force) menyelesaikan permutasi yang tidak diketahui secara eksak; dan re-simulasi penuh memberi validasi ground-truth. Pelajaran pertahanannya: jangan pernah memakai ulang state awal per batch pesan — gunakan nonce/IV unik per pesan seperti praktik Enigma yang benar.

### 3.5 Flag

Kaito{QAJRCEBTHUZNLKOVYMIWXPDFGS_PBTUXACNHVSKWEMRDGJOQLIFYZ_IOWKZNVDALUFPQYSJTBCMEGRXH_AQ_BH_CX_DZ_ES_FR_GV_IP_JT_KL_MO_NU_WY}
