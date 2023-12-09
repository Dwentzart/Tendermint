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
wget https://snapshots.indonode.net/selfchain/selfchaind
```
```
sudo chmod +x selfchaind
sudo mv selfchaind /usr/local/bin
```

### Config Node
* Init Node
```
selfchaind init dwentz --chain-id self-dev-1
```
```
selfchaind config chain-id self-dev-1
selfchaind config keyring-backend test
```
* Download Genesis
```
curl -Ls https://raw.githubusercontent.com/hotcrosscom/selfchain-genesis/main/networks/devnet/genesis.json > $HOME/.selfchain/config/genesis.json

```
* Custom port
```
PORT=19
```
```
selfchaind config node tcp://localhost:${PORT}657
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${PORT}660\"%" $HOME/.selfchain/config/config.toml
sed -i -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${PORT}317\"%; s%^address = \":8080\"%address = \":${PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${PORT}091\"%; s%^address = \"0.0.0.0:8545\"%address = \"0.0.0.0:${PORT}545\"%; s%^ws-address = \"0.0.0.0:8546\"%ws-address = \"0.0.0.0:${PORT}546\"%" $HOME/.selfchain/config/app.toml
```
* Add Peers or Seed
```
PEERS="94a7baabb2bcc00c7b47cbaa58adf4f433df9599@157.230.119.165:26656,d3b5b6ca39c8c62152abbeac4669816166d96831@165.22.24.236:26656,35f478c534e2d58dc2c4acdf3eb22eeb6f23357f@165.232.125.66:26656"
SEEDS="94a7baabb2bcc00c7b47cbaa58adf4f433df9599@157.230.119.165:26656,d3b5b6ca39c8c62152abbeac4669816166d96831@165.22.24.236:26656,35f478c534e2d58dc2c4acdf3eb22eeb6f23357f@165.232.125.66:26656"
sed -i -e "s|^persistent_peers *=.*|persistent_peers = \"$PEERS\"|" $HOME/.selfchain/config/config.toml
sed -i.bak -e "s/^seeds =.*/seeds = \"$SEEDS\"/" $HOME/.selfchain/config/config.toml
```
* Setup minimum GasFee
```
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.005uself\"/" $HOME/.selfchain/config/app.toml
```

* Setup pruning
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.selfchain/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.selfchain/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.selfchain/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.selfchain/config/app.toml
```
* Create service
```
sudo tee /etc/systemd/system/selfchaind.service > /dev/null <<EOF
[Unit]
Description=selfchain
After=network-online.target

[Service]
User=$USER
ExecStart=$(which selfchaind) start --home $HOME/.selfchain
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
sudo systemctl enable selfchaind
```
```
sudo systemctl restart selfchaind
```
### Check log node
```
journalctl -fu selfchaind -o cat
```
### Wallet configuration
* add wallet
```
selfchaind keys add wallet
```
* recover wallet
```
selfchaind keys add wallet --recover
```
* list wallet
```
selfchaind keys list
```
* delete wallet
```
selfchaind keys delete wallet
```
* check balances
```
selfchaind q bank balances $(selfchaind keys show wallet -a)
```
### Validator Management
* create validator
```
selfchaind tx staking create-validator \
--amount="1000000000uself" \
--pubkey=$(selfchaind tendermint show-validator) \
--moniker=NodeName \
--identity=create account on keybase \
--website="https://" \
--details=details node validator \
--chain-id=self-dev-1 \
--commission-rate="0.1" \
--commission-max-rate="0.15" \
--commission-max-change-rate="0.05" \
--min-self-delegation=1000000000 \
--broadcast-mode block \
--gas-adjustment=1.2 \
--gas-prices="0.5uself" \
--gas=auto \
--from=wallet
```
* edit validator
```
selfchaind tx staking edit-validator \
--new-moniker="" \
--identity="" \
--chain-id=self-dev-1 \
--gas-adjustment=1.2 \
--gas-prices="0.5uself" \
--gas=auto \
--from=wallet
```
* unjail validator
```
selfchaind tx slashing unjail --from wallet --chain-id self-dev-1 --gas-prices 0.5uself --gas-adjustment 1.2 --gas auto
```
* check jailed reason
```
selfchaind query slashing signing-info $(selfchaind tendermint show-validator)
```
### Token management

* withdraw rewards
```
selfchaind tx distribution withdraw-all-rewards --from wallet --chain-id self-dev-1  --gas-adjustment 1.2 --gas-prices 0.05uself --gas auto -y
```
* withdraw rewards with comission
```
selfchaind tx distribution withdraw-rewards $(selfchaind keys show wallet --bech val -a) --commission --from wallet --chain-id self-dev-1  --gas-adjustment 1.2 --gas-prices 0.05uself --gas auto -y
```
* delegate token to your own validator
```
selfchaind tx staking delegate $(selfchaind keys show wallet --bech val -a) 1000000uself --from wallet --chain-id self-dev-1  --gas-adjustment 1.2 --gas-prices 0.05uself --gas auto -y
```
### Node Info
* node id
```
selfchaind status 2>&1 | jq .NodeInfo
```
* validator info
```
selfchaind status 2>&1 | jq .ValidatorInfo
```

### Delete Node
```
sudo systemctl stop selfchaind &&
sudo systemctl disable selfchaind &&
sudo rm /etc/systemd/system/selfchaind.service &&
sudo systemctl daemon-reload &&
rm -f $(which selfchaind) &&
rm -rf .selfchain &&
rm -rf selfchain
```
