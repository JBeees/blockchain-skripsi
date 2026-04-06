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

# Tambahkan ke PATH
echo 'export PATH=/usr/local/bin:$PATH' >> ~/.bashrc
source ~/.bashrc

# Verifikasi instalasi — harus muncul 1.13.15
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
cd ~/case-base
git clone https://github.com/JBeees/blockchain-skripsi.git
cp blockchain-skripsi/genesis.json ~/case-base/
```

### 2. Inisialisasi Blockchain

```bash
cd ~/case-base
geth init --datadir ./project genesis.json
# Harus muncul: Successfully wrote genesis state
# Konfirmasi ke grup bahwa init berhasil
```

### 3. Simpan Password

```bash
echo "password_kamu" > password.txt
chmod 600 password.txt
```

### 4. Generate config.toml

> ⚠️ `static-nodes.json` sudah tidak didukung di Geth v1.13.15. Gunakan `config.toml` untuk static nodes.

```bash
cd ~/case-base
geth --datadir ./project dumpconfig > config.toml
```

### 5. Edit StaticNodes di config.toml

```bash
nano config.toml
```

Cari baris `StaticNodes` di bagian `[Node.P2P]` lalu isi dengan enode URL semua kelompok **kecuali diri sendiri**:

```toml
StaticNodes = ["enode://ENODE_K1@10.34.100.173:30303", "enode://ENODE_K3@10.34.100.177:30303", "enode://ENODE_K4@10.34.100.176:30303"]
```

> Ganti `ENODE_Kx` dengan enode URL asli masing-masing kelompok (tanpa `?discport=0` di akhir)
> Jangan masukkan enode diri sendiri ke dalam list

Simpan: `Ctrl+X` → `Y` → `Enter`

### 6. Jalankan Node

> Tunggu K1 bagikan enode URL dulu sebelum jalankan ini

```bash
cd ~/case-base
geth \
  --config config.toml \
  --networkid 20260315 \
  --bootnodes "enode://ENODE_K1@10.34.100.173:30303" \
  --port 30303 \
  --http --http.addr "0.0.0.0" --http.port 8545 \
  --http.api "eth,net,web3,personal,miner" \
  --mine --miner.etherbase <ADDR_KELOMPOK_KAMU> \
  --unlock <ADDR_KELOMPOK_KAMU> --password ./password.txt \
  --allow-insecure-unlock \
  --http.corsdomain="*" \
  --metrics \
  --metrics.addr 0.0.0.0 \
  --metrics.port 6060 \
  --verbosity 3
```

> Ganti `enode://ENODE_K1...` dengan enode URL dari K1 (IP: 10.34.100.173)
> Ganti `<ADDR_KELOMPOK_KAMU>` dengan address validator kamu (dengan 0x)

### 7. Verifikasi Jaringan

```bash
# Buka terminal baru, jangan tutup terminal node
geth attach --datadir ~/case-base/project
```

```javascript
net.peerCount       // harus: jumlah kelompok - 1
clique.getSigners() // harus muncul semua address validator
eth.blockNumber     // harus: > 0
admin.peers         // lihat detail IP yang terhubung
```

---

## Catatan Penting

> - Folder `project/` berisi private key — **jangan di-push ke GitHub**
> - File `password.txt` — **jangan di-push ke GitHub**
> - File `config.toml` — **jangan di-push ke GitHub** (berisi konfigurasi lokal)
> - File `genesis.json` harus **sama persis** di semua VM
> - Node harus **terus berjalan** selama testing dan demo
> - Gunakan Geth versi **1.13.15** — versi lebih baru tidak support Clique PoA

---

## Prosedur Init Ulang genesis.json

> Lakukan ini jika `genesis.json` diupdate oleh K1 — **semua kelompok wajib melakukan ini tanpa terkecuali.**

### 1. Stop Node

```bash
pkill geth

# Verifikasi sudah mati
ps aux | grep geth
```

### 2. Backup Keystore

```bash
cp -r ~/case-base/project/keystore ~/keystore-backup
```

> ⚠️ Jangan lewati langkah ini — keystore berisi private key validator kamu!

### 3. Hapus Data Lama

```bash
rm -rf ~/case-base/project
```

### 4. Pull genesis.json Terbaru dari GitHub

```bash
cd ~/case-base/blockchain-skripsi
git pull
cp genesis.json ~/case-base/
```

### 5. Init Ulang

```bash
cd ~/case-base
geth init --datadir ./project genesis.json
# Harus muncul: Successfully wrote genesis state
```

### 6. Kembalikan Keystore

```bash
mkdir -p ~/case-base/project/keystore
cp -r ~/keystore-backup/* ~/case-base/project/keystore/

# Verifikasi keystore ada
ls ~/case-base/project/keystore/
# Harus muncul file UTC--...
```

### 7. Generate config.toml Ulang

```bash
cd ~/case-base
geth --datadir ./project dumpconfig > config.toml
```

Edit StaticNodes seperti sebelumnya:

```bash
nano config.toml
```

### 8. Jalankan Node Lagi

```bash
cd ~/case-base
nohup geth \
  --config config.toml \
  --networkid 20260315 \
  --bootnodes "enode://ENODE_K1@10.34.100.173:30303" \
  --port 30303 \
  --http --http.addr "0.0.0.0" --http.port 8545 \
  --http.api "eth,net,web3,personal,miner" \
  --mine --miner.etherbase <ADDR_KELOMPOK_KAMU> \
  --unlock <ADDR_KELOMPOK_KAMU> --password ./password.txt \
  --allow-insecure-unlock \
  --http.corsdomain="*" \
  --metrics \
  --metrics.addr 0.0.0.0 \
  --metrics.port 6060 \
  --verbosity 3 > geth.log 2>&1 &
```

### 9. Verifikasi

```bash
geth attach --datadir ~/case-base/project
```

```javascript
net.peerCount       // harus: jumlah kelompok - 1
clique.getSigners() // harus muncul semua address validator
eth.blockNumber     // harus: > 0
```

---

*Tugas Kelas A — Sistem Komputasi Terdistribusi*
