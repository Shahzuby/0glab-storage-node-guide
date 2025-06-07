
# 🚀 0G Storage Node Setup Guide (GCP + Screen Session)

> A complete guide to setting up an official 0G Labs Storage Node using Google Cloud Platform and screen session.

---

## 🧱 System Requirements

| Component     | Recommended |
|---------------|-------------|
| CPU           | 4+ cores    |
| RAM           | 16 GB+      |
| SSD           | 500 GB+     |
| OS            | Ubuntu 24.04|
| Bandwidth     | 500 Mbps    |

---

## 🔧 Step 1: Create a GCP VM

1. Go to Google Cloud Console → Compute Engine → VM Instances → Create Instance
2. Choose:
   - Machine: e2-standard-4 (or higher)
   - Boot Disk: Ubuntu 24.04 LTS (500GB+)
   - Allow HTTP/HTTPS
3. Click “Create”

---

## 🔓 Step 2: Open Required Firewall Ports

Open these TCP/UDP ports:

- 30333, 30334, 9000

From VPC > Firewall > Create rule:

```
Name:       og-storage-node
Target:     All instances
Source:     0.0.0.0/0
Protocols:  tcp:30333,30334,9000
            udp:30333,30334,9000
```
Add chain claim faucets and get RPC:

Add 0G-Galileo-Testnet chain from here: https://docs.0g.ai/run-a-node/testnet-information

Take faucet: https://faucet.0g.ai/

Get RPC: https://www.astrostake.xyz/0g-status

---

## ⚙️ Step 3: Install Dependencies

SSH into the VM and run:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y build-essential pkg-config libssl-dev curl clang cmake screen git unzip
```

Install Rust:

```bash
curl https://sh.rustup.rs -sSf | sh -s -- -y
source $HOME/.cargo/env
```

Install Go:

```bash
wget https://go.dev/dl/go1.24.3.linux-amd64.tar.gz && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf go1.24.3.linux-amd64.tar.gz && \
rm go1.24.3.linux-amd64.tar.gz && \
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc && \
source ~/.bashrc
```

---

## 📦 Step 4: Clone and Build 0G Storage Node

```bash
git clone -b v0.8.7 https://github.com/0glabs/0g-storage-node.git
```
```bash
cd 0g-storage-node && git checkout v1.0.0 && git submodule update --init
```
```bash
cargo build --release
```

---

## ⚙️ Step 5: Configure the Node

```bash
rm -rf $HOME/0g-storage-node/run/config.toml
nano config.toml
```
delete all content and paste below conent by holding cntrl+k

```bash
blockchain_rpc_endpoint = "https://evmrpc-testnet.0g.ai"
log_contract_address = "0xbD75117F80b4E22698D0Cd7612d92BDb8eaff628"
log_sync_start_block_number = 326165

mine_contract_address = "0x3A0d1d67497Ad770d6f72e7f4B8F0BAbaa2A649C"
reward_contract_address = "0xd3D4D91125D76112AE256327410Dd0414Ee08Cb4"
miner_key = "YOUR_PRIVATE_KEY"

network_boot_nodes = [
  "/ip4/47.251.117.133/udp/1234/p2p/16Uiu2HAmTVDGNhkHD98zDnJxQWu3i1FL1aFYeh9wiQTNu4pDCgps",
  "/ip4/47.76.61.226/udp/1234/p2p/16Uiu2HAm2k6ua2mGgvZ8rTMV8GhpW71aVzkQWy7D37TTDuLCpgmX",
  "/ip4/47.251.79.83/udp/1234/p2p/16Uiu2HAkvJYQABP1MdvfWfUZUzGLx1sBSDZ2AT92EFKcMCCPVawV",
  "/ip4/47.238.87.44/udp/1234/p2p/16Uiu2HAmFGsLoajQdEds6tJqsLX7Dg8bYd2HWR4SbpJUut4QXqCj",
  "/ip4/47.251.78.104/udp/1234/p2p/16Uiu2HAmSe9UWdHrqkn2mKh99b9DwYZZcea6krfidtU3e5tiHiwN",
  "/ip4/47.76.30.235/udp/1234/p2p/16Uiu2HAm5tCqwGtXJemZqBhJ9JoQxdDgkWYavfCziaqaAYkGDSfU"
]

```

Edit the following fields:

- `blockchain_rpc_endpoint = "your_rpc_endpoint"`
- `miner_key = "<64-char private key (no 0x)>"`

---
Create a Systemd Service File
```bash
sudo tee /etc/systemd/system/zgs.service > /dev/null <<EOF
[Unit]
Description=ZGS Node
After=network.target

[Service]
User=$USER
WorkingDirectory=$HOME/0g-storage-node/run
ExecStart=$HOME/0g-storage-node/target/release/zgs_node --config $HOME/0g-storage-node/run/config.toml
Restart=on-failure
RestartSec=10
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable zgs
sudo systemctl start zgs
```

---

## 🧪 Monitoring

```bash
sudo systemctl status zgs
```

complete logs
```bash
tail -f ~/0g-storage-node/run/log/zgs.log.$(TZ=UTC date +%Y-%m-%d)
```

Check block & Sync process
```bash
 while true; do     response=$(curl -s -X POST http://localhost:5678 -H "Content-Type: application/json" -d '{"jsonrpc":"2.0","method":"zgs_getStatus","params":[],"id":1}');     logSyncHeight=$(echo $response | jq '.result.logSyncHeight');     connectedPeers=$(echo $response | jq '.result.connectedPeers');     echo -e "logSyncHeight: \033[32m$logSyncHeight\033[0m, connectedPeers: \033[34m$connectedPeers\033[0m";     sleep 5; done
```


check node status: https://chainscan-galileo.bangcode.id/

View Miner Details: https://storagescan-galileo.0g.ai/miner/YOUREVMADDRESS

---

## 💰 Incentives

- Keep your node online 24/7
- Watch 0G social or Discord for snapshot & TGE announcements
- Use valid wallet + miner key

---

Made with ❤️ for the https://t.me/andhiiTGkamaii community.
