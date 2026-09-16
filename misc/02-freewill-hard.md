# 2. Freewill-Hard

### 2.1 Informasi Challenge

- Nama Challenge: Freewill-Hard.

- Kategori: Misc.

- Author: Gojo Satoru.

- Bahan: freewill.py, ledger.enc (nonce || ciphertext || tag GCM), vault.enc, oracle.enc.

- Kunci: kaito:branch:omega (unseal _ARCHIVE) → SHA-256 sebagai kunci AES-256-GCM.

- Flag: KAITO{authorship_was_never_yours}.

### 2.2 Deskripsi Soal

The challenge presents several choices and paths, but the source code determines whether those choices actually affect the result. The decisive functions are _unseal() and _canon(): the former opens a fixed archive, while the latter demonstrably discards the reader's path before returning.

### 2.3 Langkah-Langkah

#### Langkah 1 — Buktikan Pilihan Dibuang (_canon)

Fungsi _canon() menghitung hash dari label pilihan, lalu membuangnya dan selalu membuka arsip statis:

```
def _canon(labels):    digest = hashlib.sha256("|".join(labels).encode()).digest()    _ = int.from_bytes(digest[:4], "big")    return _unseal(_ARCHIVE)
```

Arsipnya (_ARCHIVE, 18 byte) dan fungsi pembukanya:

_ARCHIVE = [193, 212, 217, 207, 233, 187, 238, 229, 243, 243, 251, 139, 212, 134, 153, 154, 157, 164]def _unseal(payload, seed=0xA7):    return bytes(value ^ seed ^ ((index * 5 + 13) & 0xFF) for index, value in enumerate(payload))

#### Langkah 2 — Buka Nilai Kanonis

```
Jalankan unseal (hasil terverifikasi: kaito:branch:omega):
```

```
python3 -c "A=[193,212,217,207,233,187,238,229,243,243,251,139,212,134,153,154,157,164]; print(bytes(v^0xA7^((i*5+13)&0xFF) for i,v in enumerate(A)))"# b'kaito:branch:omega'
```

```
Nilai kanonis di-hash SHA-256 menjadi kunci AES 32 byte. Berkas ledger.enc berlayout 12 byte nonce || ciphertext || 16 byte tag GCM (pasang dahulu bila perlu: pip install cryptography):
```

```
python3 << 'EOF'import hashlibfrom pathlib import Pathfrom cryptography.hazmat.primitives.ciphers.aead import AESGCMarchive = [193,212,217,207,233,187,238,229,243,243,251,139,212,134,153,154,157,164]canonical = bytes(v^0xA7^((i*5+13)&0xFF) for i,v in enumerate(archive))assert canonical == b"kaito:branch:omega"key = hashlib.sha256(canonical).digest()enc = Path("ledger.enc").read_bytes()print(AESGCM(key).decrypt(enc[:12], enc[12:], None).decode())EOF# KAITO{authorship_was_never_yours}
```

Keluaran dekripsi langsung menampilkan flag. Ambil tangkapan layar terminal yang menampilkan perintah beserta flag-nya.

### 2.4 Analisis

```
Fungsi _canon secara eksplisit membuang digest pilihan (_ = int.from_bytes(...)) sebelum memanggil _unseal(_ARCHIVE), sehingga jalur yang dijalani pembaca tidak pernah menjadi material kunci. Karena kunci AES = SHA-256(kaito:branch:omega) dan layout berkas terautentikasi (nonce || ciphertext || tag), dekripsi hanya berhasil bila material kanonis tepat. Berkas vault.enc dan oracle.enc tidak diperlukan untuk flag ini; keduanya dicatat sebagai artefak terbuka untuk pendalaman (kemungkinan membutuhkan kunci dari cerita yang dijalani, bukan dari cerita yang tertulis).
```

### 2.5 Flag

KAITO{authorship_was_never_yours}
