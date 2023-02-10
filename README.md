# canto-infra
---

## Install Dependencies

### Update and Upgrade System

```bash
sudo apt update
sudo apt upgrade
```

### Install golang

```bash
sudo snap install go --classic
# Test Installation
go version
```

### Install apt packages

```bash
sudo apt-get install git
sudo apt-get install gcc
sudo apt-get install make
```

## Setup Sync until First Chain Upgrade

### Install cantod

```bash
git clone https://github.com/Canto-Network/Canto.git
cd Canto
git checkout genesis
make install
sudo mv $HOME/go/bin/cantod /usr/bin/
# Test Installation
cantod version --log_level error --long | head -n6
```

### Initialize cantod

```bash
cantod init <MONIKER> --chain-id canto_7700-1
cd ~/.cantod/config
rm genesis.json
wget https://github.com/Canto-Network/Canto/raw/genesis/Networks/Mainnet/genesis.json
```

### Set Config

```bash
# Set config.toml
vim ~/.cantod/config/config.toml
# Add peers to persistent_peers list. This is under the P2P section.
persistent_peers = "<node_id>@<ip>:26656"

# IF NEEDED: Can enable prometheus. This is under the Instrumentation section.
prometheus = true
# Metrics exposed on Port: 26660

# Set app.toml
vim ~/.cantod/config/app.toml
# Set minimum-gas-prices
minimum-gas-prices = "0.0001acanto"

# IF NEEDED: Can setup unpruned node. This is under the Base Configuration section.
pruning = "nothing"
```

### Setup Systemd

```bash
# Create systemd service
sudo vim /etc/systemd/system/cantod.service
# Paste below text
[Unit]
Description=Canto Test Node
After=network.target

[Service]
Type=simple
User=ubuntu
WorkingDirectory=/home/ubuntu/
ExecStart=/usr/bin/cantod start --trace --log_level info --json-rpc.api eth,txpool,personal,net,debug,web3 --rpc.laddr "tcp://0.0.0.0:26657" --api.enable --json-rpc.enable
Restart=on-failure
StartLimitInterval=0
RestartSec=3
LimitNOFILE=65535
LimitMEMLOCK=209715200

[Install]
WantedBy=multi-user.target
# Close and Save file

# Reload the service files
sudo systemctl daemon-reload
# Create the symlinlk
sudo systemctl enable cantod.service
```

## Run Node

### Start Node

```bash
sudo systemctl start cantod
# Watch logs
journalctl -u cantod -f
```

### Chain Upgrade

```bash
# Stop Node
sudo systemctl stop cantod

# Inside Canto repository
git checkout <version_tag>
sudo rm /usr/bin/cantod
make install
sudo mv $HOME/go/bin/cantod /usr/bin/

# Restart Node
sudo systemctl start cantod
```

#### Chain Upgrades to `Version @ Block Height`

1. `v2.0.0 @ 218225`
2. `v3.0.0 @ 1231500`
3. `v4.0.0 @ 1274863`