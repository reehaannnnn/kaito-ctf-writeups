# 1. Berglas

### 1.1 Informasi Challenge

- Nama Challenge: Berglas.

- Kategori: REVERSE.

- Author: Gojo Satoru.

- Bahan: berglas.py, deck.enc (798 byte, AES-256-GCM), witness/*.log (5 berkas kesaksian).

- Kunci: berglas:acaan:effect (unmask _MASKED dengan XOR 0xBE + (index*7+11)) → SHA-256 sebagai kunci AES.

- Flag: KAITO{any_card_any_number_no_method_known}.

### 1.2 Deskripsi Soal

This challenge simulates the Berglas effect: the spectator names any card and any position, yet the result always succeeds. Five witness logs record different choices — parlor (KD at 4), mirage (7H at 39), midnight (TC at 52), glasshouse (2C at 23), elysium (AS at 17) — but all share one stack fingerprint: 8eade2b59b78b709. The hint says to ask what was chosen before the choosing began.

### 1.3 Langkah-Langkah

```
Fungsi _load_stack() hanya mengembalikan 52 placeholder (["\?\?"] * 52) dan _resolve() langsung mengembalikan kartu masukan pengguna tanpa pernah membaca tumpukan. Dengan demikian rutin interaktifnya hanyalah ilusi; flag yang sebenarnya tersimpan di dalam berkas deck.enc yang terenkripsi. Verifikasi bahwa kelima segel saksi konsisten dengan rumus SHA-256("CARD@POSISI:fingerprint")[:20]:
```

```
cat berglas.pycat witness/*.logpython3 -c "import hashlib; fp='8eade2b59b78b709'; print(hashlib.sha256(('AS@17:'+fp).encode()).hexdigest()[:20])"# 9c155011778567601fec (cocok dengan elysium.log)
```

Kelima segel terverifikasi cocok, sehingga kesaksian hanya berfungsi sebagai kamuflase dan perhatian dialihkan ke deck.enc.

Program memuat array byte _MASKED yang tidak pernah dipakai. Pengujian masker XOR 0xBE dengan geseran (index*7+11) mengungkapkan teks yang dapat dibaca:

```
python3 -c "M=[215,201,213,249,245,241,248,184,156,151,142,135,143,226,182,172,163,89,84,90]; print(bytes(v^0xBE^((i*7+11)&0xFF) for i,v in enumerate(M)).decode())"# berglas:acaan:effect
```

Material berglas:acaan:effect inilah yang menjawab petunjuk “apa yang dipilih sebelum pemilihan dimulai” — tumpukan Acaan yang sudah tertulis sebelum penonton menyebut apa pun.

```
Berkas deck.enc berlayout 12 byte nonce || ciphertext || 16 byte authentication tag, dan kunci AES-nya adalah SHA-256 digest dari material langkah 2. Jalankan dekripsi (pasang dahulu pustakanya bila belum ada: pip install cryptography):
```

```
python3 << 'EOF'import hashlib, jsonfrom pathlib import Pathfrom cryptography.hazmat.primitives.ciphers.aead import AESGCMmasked = [215,201,213,249,245,241,248,184,156,151,142,135,143,226,182,172,163,89,84,90]material = bytes(v^0xBE^((i*7+11)&0xFF) for i,v in enumerate(masked))key = hashlib.sha256(material).digest()enc = Path("deck.enc").read_bytes()result = json.loads(AESGCM(key).decrypt(enc[:12], enc[12:], None))print(result["flag"])EOF# KAITO{any_card_any_number_no_method_known}
```

JSON hasil dekripsi memuat performer, tumpukan 52 kartu, dan flag. Ambil tangkapan layar terminal yang menampilkan perintah beserta flag-nya.

### 1.4 Analisis

Ilusi efek Berglas dibangun melalui tiga lapisan: rutin interaktif yang selalu mengembalikan input penonton (tidak pernah membaca tumpukan), lima segel saksi yang valid tetapi tidak mengungkapkan apa pun tentang tumpukan, dan material kunci yang disamarkan sebagai array tak terpakai. Karena kunci AES hanya dapat direkonstruksi dari _MASKED, satu-satunya jalan adalah reverse engineering: membedah sumber, membuka material, lalu mendekripsi deck.enc. Struktur nonce || ciphertext || tag pada berkas 798 byte menegaskan pemakaian AES-256-GCM yang terautentikasi.

### 1.5 Flag

KAITO{any_card_any_number_no_method_known}
