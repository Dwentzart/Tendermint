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
wget https://snapshots.indonode.net/mantra/mantrachaind
```
```
sudo chmod +x mantrachaind
sudo mv mantrachaind /usr/local/bin
```

### Config Node
* Init Node
```
mantrachaind init dwentz --chain-id mantrachain-testnet-1
```
```
mantrachaind config chain-id mantrachain-testnet-1
mantrachaind config keyring-backend test
```
* Download Genesis
```
curl -Ls https://snapshots.indonode.com/mantra/genesis.json > $HOME/.mantrachain/config/genesis.json

```
* Custom port
```
PORT=20
```
```
mantrachaind config node tcp://localhost:${PORT}657
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${PORT}660\"%" $HOME/.mantrachain/config/config.toml
sed -i -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${PORT}317\"%; s%^address = \":8080\"%address = \":${PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${PORT}091\"%; s%^address = \"0.0.0.0:8545\"%address = \"0.0.0.0:${PORT}545\"%; s%^ws-address = \"0.0.0.0:8546\"%ws-address = \"0.0.0.0:${PORT}546\"%" $HOME/.mantrachain/config/app.toml
```
* Add Peers or Seed
```
PEERS="$(curl -sS https://rpc.mantra-t.indonode.net/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}' | sed -z 's|\n|,|g;s|.$||')"
sed -i -e "s|^persistent_peers *=.*|persistent_peers = \"$PEERS\"|" $HOME/.mantrachain/config/config.toml
```
* Setup minimum GasFee
```
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.005uaum\"/" $HOME/.mantrachain/config/app.toml
```

* Setup pruning
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.mantrachain/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.mantrachain/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.mantrachain/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.mantrachain/config/app.toml
```
* Create service
```
sudo tee /etc/systemd/system/mantrachaind.service > /dev/null <<EOF
[Unit]
Description=selfchain
After=network-online.target

[Service]
User=$USER
ExecStart=$(which mantrachaind) start --home $HOME/.mantrachain
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
sudo systemctl enable mantrachaind
```
```
sudo systemctl restart mantrachaind
```
### Check log node
```
journalctl -fu mantrachaind -o cat
```
### Wallet configuration
* add wallet
```
mantrachaind keys add wallet
```
* recover wallet
```
mantrachaind keys add wallet --recover
```
* list wallet
```
mantrachaind keys list
```
* delete wallet
```
mantrachaind keys delete wallet
```
* check balances
```
mantrachaind q bank balances $(mantrachaind keys show wallet -a)
```
### Validator Management
* create validator
```
mantrachaind tx staking create-validator \
--amount="1000000000uaum" \
--pubkey=$(mantrachaind tendermint show-validator) \
--moniker=NodeName \
--identity=create account on keybase \
--website="https://" \
--details=details node validator \
--chain-id=mantrachain-testnet-1 \
--commission-rate="0.1" \
--commission-max-rate="0.15" \
--commission-max-change-rate="0.05" \
--min-self-delegation=1000000000 \
--broadcast-mode block \
--gas-adjustment=1.2 \
--gas-prices="0.5uaum" \
--gas=auto \
--from=wallet
```
* edit validator
```
mantrachaind tx staking edit-validator \
--new-moniker="" \
--identity="" \
--chain-id=mantrachain-testnet-1 \
--gas-adjustment=1.2 \
--gas-prices="0.5uaum" \
--gas=auto \
--from=wallet
```
* unjail validator
```
mantrachaind tx slashing unjail --from wallet --chain-id mantrachain-testnet-1 --gas-prices 0.5uaum --gas-adjustment 1.2 --gas auto
```
* check jailed reason
```
mantrachaind query slashing signing-info $(mantrachaind tendermint show-validator)
```
### Token management

* withdraw rewards
```
mantrachaind tx distribution withdraw-all-rewards --from wallet --chain-id mantrachain-testnet-1  --gas-adjustment 1.2 --gas-prices 0.05uaum --gas auto -y
```
* withdraw rewards with comission
```
mantrachaind tx distribution withdraw-rewards $(mantrachaind keys show wallet --bech val -a) --commission --from wallet --chain-id mantrachain-testnet-1  --gas-adjustment 1.2 --gas-prices 0.05uaum --gas auto -y
```
* delegate token to your own validator
```
mantrachaind tx staking delegate $(mantrachaind keys show wallet --bech val -a) 1000000uaum --from wallet --chain-id mantrachain-testnet-1  --gas-adjustment 1.2 --gas-prices 0.05uaum --gas auto -y
```
### Node Info
* node id
```
mantrachaind status 2>&1 | jq .NodeInfo
```
* validator info
```
mantrachaind status 2>&1 | jq .ValidatorInfo
```

### Delete Node
```
sudo systemctl stop mantrachaind &&
sudo systemctl disable mantrachaind &&
sudo rm /etc/systemd/system/mantrachaind.service &&
sudo systemctl daemon-reload &&
rm -f $(which mantrachaind) &&
rm -rf .mantrachain 
```
