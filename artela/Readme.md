### Install prequirement
```
sudo apt update && sudo apt upgrade -y
```
```
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

### Install go
```
ver="1.20.4"
cd $HOME
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
rm "go$ver.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
source ~/.bash_profile
go version
```
### Download binary
```
cd $HOME
git clone https://github.com/artela-network/artela
cd artela
git checkout v0.4.7-rc4
make install
```

### Config Node
* Init Node
```
artelad init dwentz --chain-id artela_11822-1
```
```
artelad config chain-id artela_11822-1
artelad config keyring-backend test
```
* Download Genesis
```
wget -O $HOME/.artelad/config/genesis.json https://docs.artela.network/assets/files/genesis-314f4b0294712c1bc6c3f4213fa76465.json
```
* Custom port
```
PORT=25
```
```
artelad config node tcp://localhost:${PORT}657
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${PORT}660\"%" $HOME/.artelad/config/config.toml
sed -i -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${PORT}317\"%; s%^address = \":8080\"%address = \":${PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${PORT}091\"%; s%^address = \"0.0.0.0:8545\"%address = \"0.0.0.0:${PORT}545\"%; s%^ws-address = \"0.0.0.0:8546\"%ws-address = \"0.0.0.0:${PORT}546\"%" $HOME/.artelad/config/app.toml
```
* Add Peers or Seed
```
seed="43b0a791ba5a2cd500c615ccc88304f3f4ab2501@rpc-t.artela.nodestake.org:666"
sed -i.bak -e "s/^seed *=.*/seed = \"$seed\"/" ~/.artelad/config/config.toml
peers="a996136dcb9f63c7ddef626c70ef488cc9e263b8@144.217.68.182:22256,978dee673bd447147f61aa5a1bdaabdfb8f8b853@47.88.57.107:26656,aa416d3628dcce6e87d4b92d1867c8eca36a70a7@47.254.93.86:26656,30fb0055aced21472a01911353101bc4cd356bb3@47.89.230.117:26656,32d0e4aec8d8a8e33273337e1821f2fe2309539a@47.88.58.36:26656,9e2fbfc4b32a1b013e53f3fc9b45638f4cddee36@47.254.66.177:26656,a03ae11a093c67e2554b73d174c4168fe715af10@57.128.103.184:26656,b23bc610c374fd071c20ce4a2349bf91b8fbd7db@65.108.72.233:11656,04fe1c36f8481649101bff41485e66287e40b136@170.64.140.116:26656,0188a9bcff4f411b29dbddda527d77803396e1c6@185.245.182.180:26656,bec6934fcddbac139bdecce19f81510cb5e02949@47.254.24.106:26656,146d6011cce0423f564c9277c6a3390657c53730@157.90.226.23:26656,1b73ac616d74375932fb6847ec67eee4a98174e9@116.202.85.52:25556,35ce36af33e289a29787eedb3127d21bf10edcff@81.0.218.194:45656,de5612c035bd1875f0bd36d7cbf5d660b0d1e943@5.78.64.11:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.artelad/config/config.toml


```
* Setup minimum GasFee
```
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0uart\"/" $HOME/.artelad/config/app.toml
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.artelad/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.artelad/config/config.toml

```

* Setup pruning
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.artelad/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.artelad/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.artelad/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.artelad/config/app.toml
```
* Create service
```
sudo tee /etc/systemd/system/artelad.service > /dev/null <<EOF
[Unit]
Description=artela
After=network-online.target

[Service]
User=$USER
ExecStart=$(which artelad) start --home $HOME/.artelad
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
### Launch Node
```
sudo systemctl daemon-reload 
```
```
sudo systemctl enable artelad
```
```
sudo systemctl restart artelad
```
### Check log node
```
journalctl -fu artelad -o cat
```
### Wallet configuration
* add wallet
```
artelad keys add wallet
```
* recover wallet
```
artelad keys add wallet --recover
```
* list wallet
```
artelad keys list
```
* delete wallet
```
artelad keys delete wallet
```
* check balances
```
artelad q bank balances $(artelad keys show wallet -a)
```
### Validator Management
* create validator
```
artelad tx staking create-validator \
--amount="1000000000uart" \
--pubkey=$(artelad tendermint show-validator) \
--moniker=NodeName \
--identity=create account on keybase \
--website="https://" \
--details=details node validator \
--chain-id=artela_11822-1 \
--commission-rate="0.1" \
--commission-max-rate="0.15" \
--commission-max-change-rate="0.05" \
--min-self-delegation=1000000000 \
--broadcast-mode block \
--gas-adjustment=1.2 \
--gas-prices="0.5uart" \
--gas=auto \
--from=wallet
```
* edit validator
```
artelad tx staking edit-validator \
--new-moniker="" \
--identity="" \
--chain-id=artela_11822-1 \
--gas-adjustment=1.2 \
--gas-prices="0.5uart" \
--gas=auto \
--from=wallet
```
* unjail validator
```
artelad tx slashing unjail --from wallet --chain-id artela_11822-1 --gas-prices 0.5uart --gas-adjustment 1.2 --gas auto
```
* check jailed reason
```
artelad query slashing signing-info $(artelad tendermint show-validator)
```
### Token management

* withdraw rewards
```
artelad tx distribution withdraw-all-rewards --from wallet --chain-id artela_11822-1  --gas-adjustment 1.2 --gas-prices 0.05uart --gas auto -y
```
* withdraw rewards with comission
```
artelad tx distribution withdraw-rewards $(artelad keys show wallet --bech val -a) --commission --from wallet --chain-id artela_11822-1  --gas-adjustment 1.2 --gas-prices 0.05uart --gas auto -y
```
* delegate token to your own validator
```
artelad tx staking delegate $(artelad keys show wallet --bech val -a) 1000000uart --from wallet --chain-id artela_11822-1  --gas-adjustment 1.2 --gas-prices 0.05uart --gas auto -y
```
### Node Info
* node id
```
artelad status 2>&1 | jq .NodeInfo
```
* validator info
```
artelad status 2>&1 | jq .ValidatorInfo
```

### Delete Node
```
sudo systemctl stop artelad &&
sudo systemctl disable artelad &&
sudo rm /etc/systemd/system/artelad.service &&
sudo systemctl daemon-reload &&
rm -f $(which artelad) &&
rm -rf .artelad
```
