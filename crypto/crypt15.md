# 4. Crypt15

### 4.1 Informasi Challenge

- Nama Challenge: Crypt15.

- Kategori: CRYPTO (Forensics, 900 pts).

- Bahan: msgstore_db.crypt15 (backup lokal WhatsApp terenkripsi) + tiga hints yang dapat dibeli.

- Kunci: 64 hex (32 byte) gabungan dua shard 32-hex + derivasi HMAC-SHA256 ganda + AES-256-GCM + gzip.

- Flag: KAITO{signal_protocol_crypt15_backup_decrypted}.

### 4.2 Deskripsi Soal

After a Kaito Kid heist, police seized a rooted Android phone. The given msgstore_db.crypt15 file is a WhatsApp encrypted local-backup file. Unlike the older crypt12, the crypt15 format is not password-based: it is encrypted with AES-256-GCM keyed from a locally-stored 64-hex-character backup key (normally shown once on the phone, or exfiltrated from /data/data/com.whatsapp/files/key). Without that key the ciphertext is computationally infeasible to break.

### 4.3 Langkah-Langkah

#### Langkah 1 — Pahami Format crypt15

Format backup crypt15 WhatsApp memakai AES-256-GCM dengan kunci backup 64-hex yang tersimpan lokal, bukan berbasis kata sandi. Tanpa kunci tersebut ciphertext tidak dapat dipecahkan secara komputasional, sehingga perhatian dialihkan pada pemulihan kunci dari hints.

file msgstore_db.crypt15# data (backup terenkripsi, bukan SQLite mentah)

#### Langkah 2 — Pulihkan Kunci dari Hints

Dua hints bersama-sama memberikan kunci yang terbelah menjadi dua paruh 32-hex: field notes Agent Kaito (Local shard: deadbeefcafebabe1337133713371337) dan Unlock Hint (0123456789abcdef0123456789abcdef). Penggabungan shard + unlock menghasilkan root key 64-hex (32 byte) yang penuh:

```
ROOT=deadbeefcafebabe13371337133713370123456789abcdef0123456789abcdefpython3 -c "print(len(bytes.fromhex('$ROOT'.replace('$ROOT','deadbeefcafebabe13371337133713370123456789abcdef0123456789abcdef'))))"# 32
```

```
Hint "WhatsApp E2E Encryption — Brief" menjabarkan rantai derivasi persis backup crypt15 asli: intermediate = HMAC-SHA256(key = 0x00*32, msg = root_key), lalu aes_key = HMAC-SHA256(key = intermediate, msg = "backup encryption\x01"), dilanjutkan dekripsi AES-256-GCM dan dekompresi gzip menjadi SQLite msgstore.db mentah. Alih-alih merakit manual (parsing header protobuf, ekstraksi IV, tag GCM), dipakai paket open-source wa-crypt-tools (pip install wa-crypt-tools) yang mengimplementasikan spesifikasi persis dan menyediakan CLI wadecrypt:
```

```
pip install wa-crypt-toolswadecrypt <64-hex-key> msgstore_db.crypt15 out.dbpython3 -c "import sqlite3; print(sqlite3.connect('out.db').execute(\"select name from sqlite_master where type='table'\").fetchall())"# header 'SQLite format 3' valid; tabel: jid, chat, message
```

#### Langkah 3 — Baca Chat dan Ambil Flag

DB hasil dekripsi memuat tiga tabel (jid, chat, message). Pembacaan tabel message menampilkan percakapan in-universe tentang kunci backup E2E yang dipecah menjadi dua shard — dan pesan terakhir memuat flag secara langsung:

```
python3 -c "import sqlite3; [print(r) for r in sqlite3.connect('out.db').execute('select * from message')]"# ... KAITO{signal_protocol_crypt15_backup_decrypted}
```

### 4.4 Analisis

Keamanan crypt15 bertumpu seluruhnya pada kerahasiaan kunci backup 32 byte; formatnya sendiri terdokumentasi (HMAC ganda + GCM + gzip) dan didukung tooling publik. Challenge ini memodelkan kegagalan operasional klasik: kunci yang dipecah menjadi shard justru dibocorkan melalui hints, sehingga pemulihan menjadi masalah rekonstruksi (shard + unlock), bukan kriptanalisis. Setelah root key utuh, dekripsi bersifat mekanis. Pelajaran pertahanannya: kunci backup tidak boleh meninggalkan perangkat dalam bentuk apa pun yang dapat dibeli/disadap.

### 4.5 Flag

KAITO{signal_protocol_crypt15_backup_decrypted}
