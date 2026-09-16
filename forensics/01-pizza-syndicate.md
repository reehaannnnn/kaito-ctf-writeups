# 1. Pizza Syndicate

### 1.1 Informasi Challenge

- Nama Challenge: Pizza Syndicate.

- Kategori: Forensics.

- Author: LTFZP.

- Bahan: kitchen_tap.pcap (tangkapan MQTT) + oven_core_ram.raw (dump RAM oven pintar).

- Kunci: NP-9000-8F3A-44C1-229E::CrustCartelVendetta1924::peperoni_heritage_1924 → SHA-256 sebagai kunci AES-256-CBC.

- Flag: Kaito{n4p0l1_s4uc3_m4f14_m3m0ry_c4rv1ng_p1zz4_1924}.

### 1.2 Deskripsi Soal

The smart oven at Ristorante Don Peperoni was compromised by an in-memory implant. The network capture and RAM dump hold everything needed to recover the sauce formula. The PCAP yields the vault identifier and encryption mode, while the RAM dump stores the reflective implant, the hardware identifier, and the staged payload.

### 1.3 Langkah-Langkah

#### Langkah 1 — Bedah PCAP (MQTT)

Buka kitchen_tap.pcap di Wireshark dan saring trafik MQTT. Pesan yang relevan berada di bawah topik /oven/napoli/v1/override: perintah extract_sauce_formula dengan vault_id peperoni_heritage_1924 dan cipher aes-256-cbc, serta pesan implan (agent reflective_stub, target_pid 842, action hook_recipe_dossier). PCAP dengan demikian memberikan identifier vault dan mode enkripsi.

tshark -r kitchen_tap.pcap -Y mqtt -T fields -e mqtt.topic -e mqtt.msg 2>/dev/null | grep -i -E "override|agent" | head -10

#### Langkah 2 — Bedah Dump RAM (Implan + HWID)

Cari peringatan kernel di dalam dump untuk menemukan implan ELF reflektif dan identifier perangkat keras:

```
strings -a oven_core_ram.raw | grep -i -E "ELF|DMI|anonymous|thermal"
```

```
Keluaran yang relevan: DMI Officine Meccaniche Napoli S.p.A. Fornace-Napoli-9000-SmartPro/Rev 3.2-Industrial, region executable anonim pada 0x00520000, stub ELF reflektif in-memory, dan override suhu darurat 500C. ELF reflektif berada pada 0x520000, sedangkan tabel DMI memuat identifier NP-9000-8F3A-44C1-229E. Konstanta implan yang terobfuskasi di sekitar offset 0x522090 dibuka dengan XOR 0x5C:
```

```
python3 << 'EOF'from pathlib import Pathram = Path("oven_core_ram.raw").read_bytes()obfuscated = ram[0x522090:0x522090 + 27]print(bytes(value ^ 0x5C for value in obfuscated).decode())EOF# ::CrustCartelVendetta1924::
```

Material kunci lengkap dengan demikian adalah HWID + salt + vault_id: NP-9000-8F3A-44C1-229E::CrustCartelVendetta1924::peperoni_heritage_1924.

Cari penanda payload dan petakan layout-nya:

```
strings -a -t x oven_core_ram.raw | grep PIZZA_SAUCE_V2
```

```
Buffer bermula pada 0x880000 dengan layout: marker PIZZA_SAUCE_V2, panjang ciphertext big-endian 4 byte (688 byte), IV 16 byte, lalu ciphertext AES-CBC. Dekripsi dengan kunci SHA-256 dari material langkah 2 (pasang dahulu bila perlu: pip install cryptography):
```

```
python3 << 'EOF'import hashlib, structfrom pathlib import Pathfrom cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modesfrom cryptography.hazmat.primitives.padding import PKCS7ram = Path("oven_core_ram.raw").read_bytes()marker = b"PIZZA_SAUCE_V2"start = ram.index(marker) + len(marker)length = struct.unpack_from(">I", ram, start)[0]iv = ram[start + 4:start + 20]ciphertext = ram[start + 20:start + 20 + length]key = hashlib.sha256(b"NP-9000-8F3A-44C1-229E::CrustCartelVendetta1924::peperoni_heritage_1924").digest()dec = Cipher(algorithms.AES(key), modes.CBC(iv)).decryptor()padded = dec.update(ciphertext) + dec.finalize()unpad = PKCS7(128).unpadder()print((unpad.update(padded) + unpad.finalize()).decode())EOF
```

Keluaran dekripsi memuat formula saus Don Peperoni beserta flag. Ambil tangkapan layar terminal yang menampilkan perintah beserta flag-nya.

### 1.4 Analisis

```
Rantai bukti tersusun dari tiga sumber yang saling mengunci: PCAP memberi vault_id dan mode cipher, dump RAM memberi HWID (DMI) dan salt (XOR 0x5C pada 0x522090), dan marker PIZZA_SAUCE_V2 memberi lokasi payload beserta IV. Karena kunci AES = SHA-256(HWID+salt+vault_id), setiap komponen wajib tepat — satu byte salah maka padding PKCS7 gagal. Struktur marker + panjang + IV + ciphertext menegaskan bahwa payload dipentaskan secara sadar, bukan sisa memori acak.
```

### 1.5 Flag

Kaito{n4p0l1_s4uc3_m4f14_m3m0ry_c4rv1ng_p1zz4_1924}
