# 2. Fear Hole

### 2.1 Informasi Challenge

- Nama Challenge: Fear Hole.

- Kategori: WEB (1000 pts).

- Stack: Flask + PyJWT (EdDSA/Ed25519), tema The Hole (admission → interview → discharge via JWT).

- Celah: JWT kid confusion (KeyResolver cache lintas-authority saat partitioned=False) + redeem_recovery tanpa ikatan realm (bound_recovery=False).

- Flag: Kaito{th3_d00r_w45_n3v3r_1n_th3_r00m_58e4c12a}.

### 2.2 Deskripsi Soal

A Flask + PyJWT (EdDSA/Ed25519) web challenge themed as The Hole, which tests visitors' fears and then releases them through a JWT-verified discharge process. The goal is to forge a valid discharge token and recover the real flag.

### 2.3 Langkah-Langkah

#### Langkah 1 — Admission dan Eskalasi ke Staff

Buka admission untuk memperoleh cookie, csrf, dan arrival_slip (JWT yang dapat di-decode tanpa verifikasi untuk mengambil nonce). Kemudian naikkan hak menjadi staff: buat practice realm dengan alias r.sanchez (akun staff asli), minta recovery ticket untuk alias tersebut, lalu redeem di realm asli. Karena bound_recovery=False, kecocokan realm/subject tidak benar-benar diperiksa sehingga role berubah menjadi staff.

```
curl -k -c jar.txt https://fear-xxxx.instance.tbf1.online/admission# catat cookie, csrf, arrival_slip (decode ambil nonce)# buat practice realm alias r.sanchez -> minta recovery ticket -> redeem di realm asli
```

#### Langkah 2 — Racuni Cache KeyResolver

Inti bug ada di tokens.py kelas KeyResolver: saat partitioned=False (default di server.py), cache diindeks hanya berdasarkan kid, bukan (authority, kid) — klasik JWT kid confusion karena satu resolver dipakai bersama untuk authority berbeda (controller/reception vs examiner). Sebagai staff, mulai interview agar server menyiapkan next_key epoch berikutnya; kid-nya dapat diintip lebih dulu dari /api/reception/manifest (scheduled_key) sebelum key aktif. Daftarkan examiner dengan keypair Ed25519 buatan sendiri tetapi kid dipaksa sama dengan scheduled_key, lalu kirim token attestation bertanda tangan private key sendiri ke /api/examiners/test — terverifikasi lolos (kid belum pernah di-cache) sehingga cache kini menyimpan public key penyerang di bawah kid tersebut.

```
python3 -c "from cryptography.hazmat.primitives.asymmetric import ed25519; sk=ed25519.Ed25519PrivateKey.generate(); open('atk_sk.bin','wb').write(sk.private_bytes_raw()); open('atk_pk.bin','wb').write(sk.public_key().public_bytes_raw())"# daftar examiner kid=<scheduled_key> + POST /api/examiners/test (attestation self-signed)
```

#### Langkah 3 — Aktifkan Key Beracun dan Tempa Discharge

Selesaikan interview (/api/interviews/return) agar controller_key server berpindah ke next_key yang kid-nya sudah tercemar di cache. Buat JWT baru (sub, nonce, admission, epoch sesuai data admission) yang ditandatangani private key sendiri dengan kid yang sama, lalu kirim ke /api/reception/discharge bersama arrival_slip asli. Server memverifikasi memakai cache tercemar → cocok dengan public key penyerang → dianggap sah → flag asli dikembalikan.

```
python3 forge_discharge.py  # JWT self-signed kid=<poisoned> + POST /api/reception/discharge# Kaito{th3_d00r_w45_n3v3r_1n_th3_r00m_58e4c12a}
```

### 2.4 Analisis

Dua kelemahan saling mengunci: redeem_recovery yang longgar memberi jalan eskalasi ke staff (tanpa ini penyerang tak dapat memicu rotasi key), sedangkan cache KeyResolver yang diindeks kid saja memungkinkan public key penyerang menetap lintas-authority. Karena rotasi key (scheduled_key terlihat di manifest sebelum aktif) memberi jendela untuk meracuni cache lebih dulu, token tempaan lolos verifikasi. Perbaikan yang tepat: indeks cache per (authority, kid) atau resolver terpisah per authority, ikat recovery ke realm (bound_recovery=True), dan jangan pernah mengekspos kid terjadwal sebelum aktif.

### 2.5 Flag

Kaito{th3_d00r_w45_n3v3r_1n_th3_r00m_58e4c12a}
