
# 🚀 0G Storage Node Setup Guide (GCP + Screen Session)

> A complete guide to setting up an official 0G Labs Storage Node using Google Cloud Platform and screen session.

---

## 🧱 System Requirements

| Component     | Recommended |
|---------------|-------------|
| CPU           | 4+ cores    |
| RAM           | 16 GB+      |
| SSD           | 500 GB+     |
| OS            | Ubuntu 22.04|
| Bandwidth     | 500 Mbps    |

---

## 🔧 Step 1: Create a GCP VM

1. Go to Google Cloud Console → Compute Engine → VM Instances → Create Instance
2. Choose:
   - Machine: e2-standard-4 (or higher)
   - Boot Disk: Ubuntu 22.04 LTS (50GB+)
   - Allow HTTP/HTTPS
3. Click “Create”

---

## 🔓 Step 2: Open Required Firewall Ports

Open these TCP/UDP ports:

- 30333, 30334, 9000

From VPC > Firewall > Create rule:

```
Name:       0g-storage-node
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
cd 0g-storage-node
cargo build --release
```

---

## ⚙️ Step 5: Configure the Node

```bash
cd run
cp config-testnet.toml config.toml
nano config.toml
```

Edit the following fields:

- `network_enr_address = "<your-public-ip>"`
- `miner_key = "<64-char private key (no 0x)>"`
- Update contract and RPC values per latest 0G docs

---

## 🟢 Step 6: Run Node in a Screen Session

Start screen:

```bash
screen -S 0gnode
```

Run the node:

```bash
../target/release/zgs_node --config config.toml --miner-key <your_private_key>
```

Detach screen:

```bash
Ctrl + A → D
```

Reattach later:

```bash
screen -r 0gnode
```

---

## 🧪 Monitoring

```bash
tail -f logs/output.log
```

---

## 💰 Incentives

- Keep your node online 24/7
- Watch 0G social or Discord for snapshot & TGE announcements
- Use valid wallet + miner key

---

Made with ❤️ for the 0G community.
