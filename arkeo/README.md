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
rm -rf arkeod
wget https://testnet-files.itrocket.net/arkeo/arkeod
chmod +x $HOME/arkeod
mv $HOME/arkeod $HOME/go/bin/arkeod

```

### Config Node
* Init Node
```
arkeod init dwentz --chain-id arkeo
```
```
arkeod config chain-id arkeo
arkeod config keyring-backend test
```
* Download Genesis
```
wget -O $HOME/.arkeo/config/genesis.json https://testnet-files.itrocket.net/arkeo/genesis.json

```
* Custom port
```
PORT=22
```
```
arkeod config node tcp://localhost:${PORT}657
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${PORT}660\"%" $HOME/.arkeo/config/config.toml
sed -i -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${PORT}317\"%; s%^address = \":8080\"%address = \":${PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${PORT}091\"%; s%^address = \"0.0.0.0:8545\"%address = \"0.0.0.0:${PORT}545\"%; s%^ws-address = \"0.0.0.0:8546\"%ws-address = \"0.0.0.0:${PORT}546\"%" $HOME/.arkeo/config/app.toml
```
* Add Peers or Seed
```
SEEDS="df0561c0418f7ae31970a2cc5adaf0e81ea5923f@arkeo-testnet-seed.itrocket.net:18656"
PEERS="5c3ca78b11bbd746f950c198cac51d4e5d4c0750@arkeo-testnet-peer.itrocket.net:18656,769de3fabb66d2dcbae7550ce7252f6f469c5d3f@65.108.126.188:26856,e033753cac027fc6605a95dab3b3fc5550d4b9bf@65.109.84.33:40656,6ae2136893a08a412f0c02eab8d595d502cd5457@65.108.206.118:36656,be71f456a7aa3da953db899298b53d28b75f4676@65.108.229.93:37656,465600bad30995e46124d5ec23021f4845be2ece@38.242.210.137:26656,844c7132a9f4550bf619edb5f59c20789357748b@[2a01:4f8:262:1d64::2]:18656,2aa2c1f2e52b3fab89a4c5c0058d53ca52dd96b9@[2a01:4f9:3a:2de9::2]:18656,db1d6df90a60d408fa638c0670cee1931d4b730d@65.108.72.253:18656,e0d147e64b92af644d704644834224ce3e3a5681@212.220.45.177:26656,8c2d799bcc4fbf44ef34bbd2631db5c3f4619e41@213.239.207.175:60656"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.arkeo/config/config.toml

```
* Setup minimum GasFee
```
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "0.001uarkeo"|g' $HOME/.arkeo/config/app.toml
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.arkeo/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.arkeo/config/config.toml

```

* Setup pruning
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.arkeo/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.arkeo/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.arkeo/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.arkeo/config/app.toml
```
* Create service
```
sudo tee /etc/systemd/system/arkeod.service > /dev/null <<EOF
[Unit]
Description=selfchain
After=network-online.target

[Service]
User=$USER
ExecStart=$(which arkeod) start --home $HOME/.arkeo
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
sudo systemctl enable arkeod
```
```
sudo systemctl restart arkeod
```
### Check log node
```
journalctl -fu arkeod -o cat
```
### Wallet configuration
* add wallet
```
arkeod keys add wallet
```
* recover wallet
```
arkeod keys add wallet --recover
```
* list wallet
```
arkeod keys list
```
* delete wallet
```
arkeod keys delete wallet
```
* check balances
```
arkeod q bank balances $(arkeod keys show wallet -a)
```
### Validator Management
* create validator
```
arkeod tx staking create-validator \
--amount="1000000000uarkeo" \
--pubkey=$(arkeod tendermint show-validator) \
--moniker=NodeName \
--identity=create account on keybase \
--website="https://" \
--details=details node validator \
--chain-id=arkeo \
--commission-rate="0.1" \
--commission-max-rate="0.15" \
--commission-max-change-rate="0.05" \
--min-self-delegation=1000000000 \
--broadcast-mode block \
--gas-adjustment=1.2 \
--gas-prices="0.5uarkeo" \
--gas=auto \
--from=wallet
```
* edit validator
```
arkeod tx staking edit-validator \
--new-moniker="" \
--identity="" \
--chain-id=arkeo \
--gas-adjustment=1.2 \
--gas-prices="0.5uarkeo" \
--gas=auto \
--from=wallet
```
* unjail validator
```
arkeod tx slashing unjail --from wallet --chain-id arkeo --gas-prices 0.5uarkeo --gas-adjustment 1.2 --gas auto
```
* check jailed reason
```
arkeod query slashing signing-info $(arkeod tendermint show-validator)
```
### Token management

* withdraw rewards
```
arkeod tx distribution withdraw-all-rewards --from wallet --chain-id arkeo  --gas-adjustment 1.2 --gas-prices 0.05uarkeo --gas auto -y
```
* withdraw rewards with comission
```
arkeod tx distribution withdraw-rewards $(arkeod keys show wallet --bech val -a) --commission --from wallet --chain-id arkeo  --gas-adjustment 1.2 --gas-prices 0.05uarkeo --gas auto -y
```
* delegate token to your own validator
```
arkeod tx staking delegate $(arkeod keys show wallet --bech val -a) 1000000uarkeo --from wallet --chain-id arkeo  --gas-adjustment 1.2 --gas-prices 0.05uarkeo --gas auto -y
```
### Node Info
* node id
```
arkeod status 2>&1 | jq .NodeInfo
```
* validator info
```
arkeod status 2>&1 | jq .ValidatorInfo
```

### Delete Node
```
sudo systemctl stop arkeod &&
sudo systemctl disable arkeod &&
sudo rm /etc/systemd/system/arkeod.service &&
sudo systemctl daemon-reload &&
rm -f $(which arkeod) &&
rm -rf .arkeo
```
