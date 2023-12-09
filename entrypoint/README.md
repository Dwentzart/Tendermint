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
mkdir -p $HOME/entrypoint && cd entrypoint
wget https://github.com/entrypoint-zone/testnets/releases/download/v1.3.0/entrypointd-v1.3.0-linux-amd64
chmod +x entrypointd-v1.3.0-linux-amd64
mv entrypointd-v1.3.0-linux-amd64 $HOME/go/bin/entrypointd

```

### Config Node
* Init Node
```
entrypointd init dwentz --chain-id entrypoint-pubtest-2
```
```
entrypointd config chain-id entrypoint-pubtest-2
entrypointd config keyring-backend test
```
* Download Genesis
```
wget -O $HOME/.entrypoint/config/genesis.json https://testnet-files.itrocket.net/entrypoint/genesis.json
```
* Custom port
```
PORT=21
```
```
entrypointd config node tcp://localhost:${PORT}657
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${PORT}660\"%" $HOME/.entrypoint/config/config.toml
sed -i -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${PORT}317\"%; s%^address = \":8080\"%address = \":${PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${PORT}091\"%; s%^address = \"0.0.0.0:8545\"%address = \"0.0.0.0:${PORT}545\"%; s%^ws-address = \"0.0.0.0:8546\"%ws-address = \"0.0.0.0:${PORT}546\"%" $HOME/.entrypoint/config/app.toml
```
* Add Peers or Seed
```
SEEDS="e1b2eddac829b1006eb6e2ddbfc9199f212e505f@entrypoint-testnet-seed.itrocket.net:34656"
PEERS="7048ee28300ffa81103cd24b2af3d1af0c378def@entrypoint-testnet-peer.itrocket.net:34656,e0bf0782c0fc1ee550d2eed0de66b0b1825776ab@167.235.39.5:46656,05419a6f8cc137c4bb2d717ed6c33590aaae022d@213.133.100.172:26878,e067ed80f78f7d343fcf90fcc6240ff3ee1441d8@65.21.69.53:36656,1a54c65244018b2484a92172113b148a8855f559@91.144.171.205:20956,7e7159e60b26508ad7b592605a79489f6c6281a0@194.163.179.176:36156,432963f1d61d0d32c9286248c4b5cfe1d89f7541@49.12.123.87:22226,75e83d67504cbfacdc79da55ca46e2c4353816e7@65.109.92.241:3106,6e38397e09a2755841e2f350ba1ff8883a66551a@[2a01:4f9:4a:2864::2]:11556,f94f05a942b987e71a92cd634915c241f58eac7c@65.109.68.87:29656,615481f262ab5862580e6e2c6aeb03a7af75f18c@45.67.216.220:12956"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.entrypoint/config/config.toml

```
* Setup minimum GasFee
```
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "0.01ibc/8A138BC76D0FB2665F8937EC2BF01B9F6A714F6127221A0E155106A45E09BCC5"|g' $HOME/.entrypoint/config/app.toml
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.entrypoint/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.entrypoint/config/config.toml
```

* Setup pruning
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.entrypoint/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.entrypoint/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.entrypoint/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.entrypoint/config/app.toml
```
* Create service
```
sudo tee /etc/systemd/system/entrypointd.service > /dev/null <<EOF
[Unit]
Description=selfchain
After=network-online.target

[Service]
User=$USER
ExecStart=$(which entrypointd) start --home $HOME/.entrypoint
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
sudo systemctl enable entrypointd
```
```
sudo systemctl restart entrypointd
```
### Check log node
```
journalctl -fu entrypointd -o cat
```
### Wallet configuration
* add wallet
```
entrypointd keys add wallet
```
* recover wallet
```
entrypointd keys add wallet --recover
```
* list wallet
```
entrypointd keys list
```
* delete wallet
```
entrypointd keys delete wallet
```
* check balances
```
entrypointd q bank balances $(entrypointd keys show wallet -a)
```
### Validator Management
* create validator
```
entrypointd tx staking create-validator \
--amount="1000000000uaum" \
--pubkey=$(entrypointd tendermint show-validator) \
--moniker=NodeName \
--identity=create account on keybase \
--website="https://" \
--details=details node validator \
--chain-id=entrypoint-pubtest-2 \
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
entrypointd tx staking edit-validator \
--new-moniker="" \
--identity="" \
--chain-id=entrypoint-pubtest-2 \
--gas-adjustment=1.2 \
--gas-prices="0.5uaum" \
--gas=auto \
--from=wallet
```
* unjail validator
```
entrypointd tx slashing unjail --from wallet --chain-id entrypoint-pubtest-2 --gas-prices 0.5uaum --gas-adjustment 1.2 --gas auto
```
* check jailed reason
```
entrypointd query slashing signing-info $(entrypointd tendermint show-validator)
```
### Token management

* withdraw rewards
```
entrypointd tx distribution withdraw-all-rewards --from wallet --chain-id entrypoint-pubtest-2  --gas-adjustment 1.2 --gas-prices 0.05uaum --gas auto -y
```
* withdraw rewards with comission
```
entrypointd tx distribution withdraw-rewards $(entrypointd keys show wallet --bech val -a) --commission --from wallet --chain-id entrypoint-pubtest-2  --gas-adjustment 1.2 --gas-prices 0.05uaum --gas auto -y
```
* delegate token to your own validator
```
entrypointd tx staking delegate $(entrypointd keys show wallet --bech val -a) 1000000uaum --from wallet --chain-id entrypoint-pubtest-2  --gas-adjustment 1.2 --gas-prices 0.05uaum --gas auto -y
```
### Node Info
* node id
```
entrypointd status 2>&1 | jq .NodeInfo
```
* validator info
```
entrypointd status 2>&1 | jq .ValidatorInfo
```

### Delete Node
```
sudo systemctl stop entrypointd &&
sudo systemctl disable entrypointd &&
sudo rm /etc/systemd/system/entrypointd.service &&
sudo systemctl daemon-reload &&
rm -f $(which entrypointd) &&
rm -rf .entrypoint
```
