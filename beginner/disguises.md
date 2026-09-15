# 4. Disguises

### 4.1 Informasi Challenge

- Nama Challenge: Disguises.

- Kategori: Beginner.

- Author: Gojo Satoru.

- Bahan: disguise.zip (case_summary.txt + locker.enc 47 byte).

- Kunci: north_tower:2103 → SHA-256 (keystream 32 byte, diulang modulo 32).

- Flag: KAITO{the_disguise_fools_eyes_not_the_timeline}.

### 4.2 Deskripsi Soal

As the story goes, the police read every printed word while Conan reads what is not printed in ink. The notice.txt file looks like an ordinary letter, yet every line ends with invisible characters. In total 384 hidden bits (177 U+200B and 207 U+200C) are spread over 16 lines, 24 bits (three characters) each.

```
unzip disguise.zipcat case_summary.txt
```

Ringkasan kasus memerintahkan untuk menemukan saksi yang waktu dan lokasinya sesuai dengan pencurian.

### 4.3 Analisis Kesaksian

```
Keempat kesaksian dibandingkan terhadap resital yang berakhir pukul 21:00. Hanya laporan Officer Takagi (21:03, akses atap menara utara, glider putih ke timur laut) yang konsisten secara waktu dan lokasi. Tiga lainnya gugur masing-masing karena perawakan terlalu berat (Inspector Megure, 21:05), monokel di mata kiri yang tidak presisi (Kurator, 21:12), serta lorong belakang yang seharusnya sudah steril pada 21:18 (Stagehand). Nilai yang berguna dengan demikian adalah lokasi north tower dan waktu 21:03.
```

### 4.4 Penyusunan Kunci

Lokasi dinormalisasi menjadi huruf kecil dengan spasi diganti garis bawah, titik dua pada jam dihilangkan, kemudian kedua nilai digabungkan dengan titik dua:

```
north tower → north_tower21:03 → 2103Material kunci: north_tower:2103
```

```
Digest SHA-256 dari material tersebut diverifikasi di terminal:
```

```
python3 -c "import hashlib; print(hashlib.sha256(b'north_tower:2103').hexdigest())"# 188828a36816e79c7de148d197977526252b6560df926eb84d7f58408f15ef7d
```

### 4.5 Dekripsi locker.enc

```
Berkas locker.enc berukuran 47 byte sedangkan digest SHA-256 hanya 32 byte, sehingga digest dipakai ulang sebagai kunci XOR berulang:
```

```
from pathlib import Pathimport hashlibciphertext = Path("locker.enc").read_bytes()key = hashlib.sha256(b"north_tower:2103").digest()plaintext = bytes(value ^ key[index % len(key)] for index, value in enumerate(ciphertext))print(plaintext.decode())# KAITO{the_disguise_fools_eyes_not_the_timeline}
```

### 4.6 Flag

KAITO{the_disguise_fools_eyes_not_the_timeline}
