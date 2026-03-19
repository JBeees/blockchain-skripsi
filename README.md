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

