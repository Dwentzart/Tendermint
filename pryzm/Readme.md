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
curl -L https://storage.googleapis.com/pryzm-resources/pryzmd-0.9.0-linux-amd64.tar.gz | tar -xvzf - -C $HOME
chmod +x pryzmd
mv pryzmd $HOME/go/bin/
```

### Config Node
* Init Node
```
pryzmd init dwentz --chain-id indigo-1
```
```
pryzmd config chain-id indigo-1
pryzmd config keyring-backend test
```
* Download Genesis
```
wget -O $HOME/.pryzm/config/genesis.json https://raw.githubusercontent.com/vinjan23/Testnet.Guide/main/Pryzm/genesis.json
```
* Custom port
```
PORT=24
```
```
pryzmd config node tcp://localhost:${PORT}657
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${PORT}660\"%" $HOME/.pryzm/config/config.toml
sed -i -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${PORT}317\"%; s%^address = \":8080\"%address = \":${PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${PORT}091\"%; s%^address = \"0.0.0.0:8545\"%address = \"0.0.0.0:${PORT}545\"%; s%^ws-address = \"0.0.0.0:8546\"%ws-address = \"0.0.0.0:${PORT}546\"%" $HOME/.pryzm/config/app.toml
```
* Add Peers or Seed
```
seeds="ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@testnet-seeds.polkachu.com:24856,d1d43cc7c7aef715957289fd96a114ecaa7ba756@testnet-seeds.nodex.one:23210"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.pryzm/config/config.toml
```
* Setup minimum GasFee
```
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "0.001upryzm"|g' $HOME/.pryzm/config/app.toml
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.pryzm/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.pryzm/config/config.toml

```

* Setup pruning
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.pryzm/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.pryzm/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.pryzm/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.pryzm/config/app.toml
```
* Create service
```
sudo tee /etc/systemd/system/pryzmd.service > /dev/null <<EOF
[Unit]
Description=pryzm
After=network-online.target

[Service]
User=$USER
ExecStart=$(which pryzmd) start --home $HOME/.pryzm
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
sudo systemctl enable pryzmd
```
```
sudo systemctl restart pryzmd
```
### Check log node
```
journalctl -fu pryzmd -o cat
```
### Wallet configuration
* add wallet
```
pryzmd keys add wallet
```
* recover wallet
```
pryzmd keys add wallet --recover
```
* list wallet
```
pryzmd keys list
```
* delete wallet
```
pryzmd keys delete wallet
```
* check balances
```
pryzmd q bank balances $(pryzmd keys show wallet -a)
```
### Validator Management
* create validator
```
pryzmd tx staking create-validator \
--amount="1000000000upryzm" \
--pubkey=$(pryzmd tendermint show-validator) \
--moniker=NodeName \
--identity=create account on keybase \
--website="https://" \
--details=details node validator \
--chain-id=indigo-1 \
--commission-rate="0.1" \
--commission-max-rate="0.15" \
--commission-max-change-rate="0.05" \
--min-self-delegation=1 \
--gas-adjustment=1.4 \
--gas-prices="0.5upryzm" \
--gas=auto \
--from=wallet
```
* edit validator
```
pryzmd tx staking edit-validator \
--new-moniker="" \
--identity="" \
--chain-id=indigo-1 \
--gas-adjustment=1.2 \
--gas-prices="0.5upryzm" \
--gas=auto \
--from=wallet
```
* unjail validator
```
pryzmd tx slashing unjail --from wallet --chain-id indigo-1 --gas-prices 0.5upryzm --gas-adjustment 1.2 --gas auto
```
* check jailed reason
```
pryzmd query slashing signing-info $(pryzmd tendermint show-validator)
```
### Token management

* withdraw rewards
```
pryzmd tx distribution withdraw-all-rewards --from wallet --chain-id indigo-1  --gas-adjustment 1.2 --gas-prices 0.05upryzm --gas auto -y
```
* withdraw rewards with comission
```
pryzmd tx distribution withdraw-rewards $(pryzmd keys show wallet --bech val -a) --commission --from wallet --chain-id indigo-1  --gas-adjustment 1.2 --gas-prices 0.05upryzm --gas auto -y
```
* delegate token to your own validator
```
pryzmd tx staking delegate $(pryzmd keys show wallet --bech val -a) 1000000upryzm --from wallet --chain-id indigo-1  --gas-adjustment 1.2 --gas-prices 0.05upryzm --gas auto -y
```
### Node Info
* node id
```
pryzmd status 2>&1 | jq .NodeInfo
```
* validator info
```
pryzmd status 2>&1 | jq .ValidatorInfo
```

### Delete Node
```
sudo systemctl stop pryzmd &&
sudo systemctl disable pryzmd &&
sudo rm /etc/systemd/system/pryzmd.service &&
sudo systemctl daemon-reload &&
rm -f $(which pryzmd) &&
rm -rf .pryzm
```
