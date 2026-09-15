# 2. Showtime

### 2.1 Informasi Challenge

- Nama Challenge: Showtime.

- Kategori: Reverse.

- Author: Gojo Satoru.

- Bahan: showtime.py, program.enc (XOR berulang dengan SHA-256 kanonis).

- Kunci: kuroba:magic:showtime (unseal _ARCHIVE dengan seed 0x5A + (index*3+7)) → SHA-256, diulang modulo 32.

- Flag: KAITO{its_showtime_ladies_and_gentlemen}.

### 2.2 Deskripsi Soal

The program asks for a show seed and appears to select a random volunteer. The goal is to inspect the source and decrypt program.enc — because the randomness is only stagecraft.

### 2.3 Langkah-Langkah

#### Langkah 1 — Bedah Ilusi Acak (_pick_volunteer + _canon)

Pemilihan relawan terlihat acak, tetapi hasilnya di-hardcode; benih (seed) juga diabaikan oleh _canon():

```
def _pick_volunteer(seed):    rng = random.Random(seed)    _ = rng.randint(0, 10**9)    return "Aoko Nakamori"def _canon(seed):    _ = hashlib.sha256(str(seed).encode()).digest()    return _unseal(_ARCHIVE)
```

Masukan seed dengan demikian tidak memengaruhi kunci enkripsi; perhatian dialihkan ke _ARCHIVE.

#### Langkah 2 — Buka Nilai Kanonis

Arsip dan fungsi pembukanya (hasil terverifikasi: kuroba:magic:showtime):

```
_ARCHIVE = [54, 37, 37, 37, 43, 45, 121, 43, 36, 31, 22, 17, 75, 7, 3, 1, 26, 20, 14, 119, 124]def _unseal(payload, seed=0x5A):    return bytes(value ^ seed ^ ((index * 3 + 7) & 0xFF) for index, value in enumerate(payload))python3 -c "A=[54,37,37,37,43,45,121,43,36,31,22,17,75,7,3,1,26,20,14,119,124]; print(bytes(v^0x5A^((i*3+7)&0xFF) for i,v in enumerate(A)))"# b'kuroba:magic:showtime'
```

```
Nilai kanonis di-hash SHA-256, lalu digest dipakai ulang sebagai kunci XOR berulang (modulo 32):
```

```
python3 << 'EOF'import hashlibfrom pathlib import Patharchive = [54,37,37,37,43,45,121,43,36,31,22,17,75,7,3,1,26,20,14,119,124]canonical = bytes(v^0x5A^((i*3+7)&0xFF) for i,v in enumerate(archive))assert canonical == b"kuroba:magic:showtime"key = hashlib.sha256(canonical).digest()ct = Path("program.enc").read_bytes()print(bytes(v^key[i%len(key)] for i,v in enumerate(ct)).decode())EOF# KAITO{its_showtime_ladies_and_gentlemen}
```

Keluaran langsung menampilkan flag. Ambil tangkapan layar terminal yang menampilkan perintah beserta flag-nya.

### 2.4 Analisis

```
Fungsi _pick_volunteer membuang hasil RNG (_ = rng.randint(...)) sebelum mengembalikan nama yang tetap, dan _canon membuang digest seed (_ = sha256(...)) sebelum membuka arsip statis. Karena kedua masukan pengguna tidak pernah menjadi material kunci, satu-satunya jalan adalah reverse engineering: membuka _ARCHIVE dengan seed 0x5A + (index*3+7), lalu memakai SHA-256(kuroba:magic:showtime) sebagai keystream XOR berulang atas program.enc.
```

### 2.5 Flag

KAITO{its_showtime_ladies_and_gentlemen}
