# Storage KV Setup Zero Gravity 0G

Hardware Requirements 

Memory: 32 GB RAM  
CPU: 8 cores  
Disk: Size matches the KV streams it maintains  

#

### Update packages and Install dependencies:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make gcc tar clang pkg-config libssl-dev ncdu cmake -y
```
Install Go  
```bash
cd $HOME
VER="1.22.0"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
go version
```
Install Rust  
```bash
cd $HOME
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```
Download source code
```bash
cd $HOME
rm -rf 0g-storage-kv
git clone -b v1.4.0 https://github.com/0glabs/0g-storage-kv.git 
```
Build  
```bash
cd 0g-storage-kv
git submodule update --init --recursive
cargo build --release
```
Download the config file
```bash
rm -rf $HOME//0g-storage-kv/run/config.toml
wget -P $HOME//0g-storage-kv/run/ https://github.com/fallblight/Storage-KV-Setup-Zero-Gravity-0G/blob/main/config.toml
```
You can use community RPC or your own RPC
```bash
#######################################################################
###                     Log Sync Config Options                     ###
#######################################################################

blockchain_rpc_endpoint = "https://evmrpc-testnet.0g.ai"
log_contract_address = "0x0460aA47b41a66694c0a73f667a1b795A5ED3556"
log_sync_start_block_number = 595059
```
Create service file
```bash
sudo tee /etc/systemd/system/0gkv.service > /dev/null <<EOF
[Unit]
Description=0G-KV Node
After=network.target

[Service]
User=$USER
WorkingDirectory=$HOME/0g-storage-kv/run
ExecStart=$HOME/0g-storage-kv/target/release/zgs_kv --config $HOME/0g-storage-kv/run/config.toml
Restart=always
RestartSec=10
LimitNOFILE=65535
StandardOutput=journal
StandardError=journal
SyslogIdentifier=zgs_kv

[Install]
WantedBy=multi-user.target
EOF
```
Enable and start service
```bash
sudo systemctl daemon-reload && \
sudo systemctl enable 0gkv && \
sudo systemctl start 0gkv && \
sudo systemctl status 0gkv
```

Check logs
```bash
sudo journalctl -u 0gkv -f -o cat
```

## Stop and Delete node
```bash
sudo systemctl stop 0gkv
sudo systemctl disable 0gkv
sudo rm /etc/systemd/system/0gkv.service
rm -rf $HOME/0g-storage-kv
```
