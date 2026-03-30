# Blockchain Skripsi — SKT Kelas A

## Spesifikasi Jaringan

| Parameter | Value |
|-----------|-------|
| Platform | Ethereum Private (Geth Clique PoA) |
| Chain ID | 20260315 |
| Konsensus | Clique PoA |
| Block Period | 5 detik |
| Network ID | 20260315 |

---

## Struktur Repository

```
├── genesis.json        # Konfigurasi genesis block (dibuat K1)
├── contracts/          # Smart contract (dikerjakan K2)
├── backend/            # REST API & plagiarism engine (dikerjakan K3)
├── monitoring/         # Dashboard monitoring (dikerjakan K4)
└── README.md
```

---

## Persiapan Awal — Semua Kelompok (K2, K3, K4)

> Lakukan langkah berikut **sebelum** setup node. K1 membutuhkan address validator kalian untuk mengkonfigurasi genesis block.

### 1. Login ke VM Masing-Masing Kelompok

Masuk ke VM yang sudah disiapkan menggunakan SSH atau akses langsung.

### 2. Install go-ethereum (Geth) Versi 1.13.15

```bash
# Download binary Geth v1.13.15
wget https://gethstore.blob.core.windows.net/builds/geth-linux-amd64-1.13.15-c5ba367e.tar.gz

# Ekstrak
tar -xvf geth-linux-amd64-1.13.15-c5ba367e.tar.gz

# Pindahkan ke /usr/local/bin
sudo mv geth-linux-amd64-1.13.15-c5ba367e/geth /usr/local/bin/

# Verifikasi instalasi
geth version
```

> Pastikan output `geth version` menunjukkan **1.13.15**.

### 3. Buat Folder Project

```bash
mkdir ~/case-base && cd ~/case-base
```

### 4. Buat Akun Baru

```bash
geth account new --datadir ./project
```

Setelah menjalankan perintah di atas, kamu akan diminta memasukkan password. Simpan password ini baik-baik.

Output-nya akan menampilkan sesuatu seperti ini:

```
Your new key was locked with a password. Please remember this password.
Public address of the key: 0xABCD...1234
```

### 5. Kirim Public Address ke K1

Salin **Public Address** yang muncul (format `0x...`) dan kirimkan ke K1 agar bisa didaftarkan sebagai validator di `genesis.json`.

> ⚠️ Jangan jalankan node sebelum K1 membagikan `genesis.json` dan enode URL!

---

## Panduan Setup Node untuk K2, K3, K4

> `genesis.json` sudah disiapkan oleh K1 dan tersedia di repo ini.
> Ikuti langkah berikut di VM masing-masing kelompok.

### 1. Clone Repo & Ambil genesis.json

```bash
git clone https://github.com/JBeees/blockchain-skripsi.git
```

### 2. Inisialisasi Blockchain

```bash
geth init --datadir ./project genesis.json
```

### 3. Simpan Password

```bash
echo "password_kamu" > password.txt
chmod 600 password.txt
```

### 4. Jalankan Node

```bash
# Tunggu K1 bagikan enode URL dulu sebelum jalankan ini
geth \
  --datadir ./project \
  --networkid 20260315 \
  --bootnodes "enode://XXXX...@10.34.100.173:30303" \
  --port 30303 \
  --http --http.addr "0.0.0.0" --http.port 8545 \
  --http.api "eth,net,web3,personal,miner" \
  --mine --miner.etherbase <ADDR_KELOMPOK_KAMU> \
  --unlock <ADDR_KELOMPOK_KAMU> --password ./password.txt \
  --allow-insecure-unlock \
  --verbosity 3
```

> Ganti `enode://XXXX...` dengan enode URL dari K1
> Ganti `<ADDR_KELOMPOK_KAMU>` dengan address validator kamu (dengan 0x)      
