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
git clone https://github.com/BlockXLabs/BlockX-Genesis-Mainnet1.git
cd BlockX-Genesis-Mainnet1
make install

```

### Config Node
* Init Node
```
blockxd init dwentz --chain-id --chain-id blockx_100-1
```
```
blockxd config chain-id blockx_100-1
blockxd config keyring-backend file
```
* Download Genesis
```
wget -O $HOME/.blockxd/config/genesis.json "https://raw.githubusercontent.com/BlockXLabs/networks/master/chains/blockx_100-1/genesis.json"
```
* Custom port
```
PORT=21
```
```
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${PORT}660\"%" $HOME/.blockxd/config/config.toml
sed -i -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${PORT}317\"%; s%^address = \":8080\"%address = \":${PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${PORT}091\"%; s%^address = \"0.0.0.0:8545\"%address = \"0.0.0.0:${PORT}545\"%; s%^ws-address = \"0.0.0.0:8546\"%ws-address = \"0.0.0.0:${PORT}546\"%" $HOME/.blockxd/config/app.toml
```
* Add Peers or Seed
```
seeds="cd462b62d54296ab4550d7c1ed5baafe5653faa6@137.184.7.64:26656,fbaf65d8f2732cb19269569763de4b75d84f5f52@147.182.238.235:26656,5f21477b66cce124fc61167713243d8de30a9572@137.184.38.212:26656,abddf4491980d5e6c31b44e3640610c77d475d89@146.190.153.165:26656"
sed -i.bak -e "s/^seed *=.*/seed = \"$seed\"/" ~/.blockxd/config/config.toml
peers="dfc2886dd41cc063ac0e962df490e94bc4aa6e43@65.108.206.74:19656,85d0069266e78896f9d9e17915cdfd271ba91dfd@146.190.153.165:26656,adcd9c90cc9fba509fb283e365ecd31bd5c37ff5@49.13.166.213:26656,9b84b33d44a880a520006ae9f75ef030b259cbaf@137.184.38.212:26656,3045517c28ad1965f68c47fa04f08b042834f2f8@143.198.130.3:26656,e15f4d31281036c69fa17269d9b26ff8733511c6@147.182.238.235:26656,34d08633547fc406095ff6d730fdfe65d34b96d0@158.69.125.73:11356,8ebf5e70dad7268a66a9198dbe9006f9140415b6@217.182.211.81:26656,dc240d568509fa275cb870b93de4db1869d7187a@5.78.103.187:26656,72639ce4ce7e0260d7ae129e6acc07dcb54d6af1@167.235.102.45:20656,bc152258668e673a3b63f964fa75afdd478078c7@185.246.85.48:39656,bbe679ddc774dc91b962985c7339a2e7934b8451@207.180.250.5:26656,724b268dbb274e7d4b26503129604a968c9e226b@37.120.189.81:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.blockxd/config/config.toml
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0abcx\"|" $HOME/.blockxd/config/app.toml

```
* Setup minimum GasFee
```
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "0.01ibc/8A138BC76D0FB2665F8937EC2BF01B9F6A714F6127221A0E155106A45E09BCC5"|g' $HOME/.entrypoint/config/app.toml
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.entrypoint/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.entrypoint/config/config.toml
```

* Setup pruning
```
sed -i 's|^pruning *=.*|pruning = "custom"|g' $HOME/.blockxd/config/app.toml
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $HOME/.blockxd/config/app.toml
sed -i 's|^pruning-interval *=.*|pruning-interval = "10"|g' $HOME/.blockxd/config/app.toml
sed -i 's|^snapshot-interval *=.*|snapshot-interval = 2000|g' $HOME/.blockxd/config/app.toml
sed -i 's|^prometheus *=.*|prometheus = true|' $HOME/.blockxd/config/config.toml
* Create service
```
sudo tee /etc/systemd/system/blockxd.service > /dev/null <<EOF
[Unit]
Description=blockx
After=network-online.target

[Service]
User=$USER
ExecStart=$(which blockxd) start --home $HOME/.blockxd
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
sudo systemctl enable blockxd
```
```
sudo systemctl restart blockxd
```
### Check log node
```
journalctl -fu blockxd-o cat
```
### Wallet configuration
* add wallet
```
blockxd keys add wallet
```
* recover wallet
```
blockxd keys add wallet --recover
```
* list wallet
```
blockxd keys list
```
* delete wallet
```
blockxd keys delete wallet
```
* check balances
```
blockxd q bank balances $(blockxd keys show wallet -a)
```
### Validator Management
* create validator
```
blockxd tx staking create-validator \
--amount 9999999999999999990000abcx \
--from wallet \
--commission-max-change-rate "0.01" \
--commission-max-rate "0.2" \
--commission-rate "0.1" \
--min-self-delegation "1" \
--pubkey  $(blockxd tendermint show-validator) \
--moniker \
--chain-id blockx_100-1 \
--identity="" \
--details="" \
--website="" \
--gas=auto \
-y
```
* edit validator
```
blockxd tx staking edit-validator \
--new-moniker= \
--identity= \
--website= \
--details=satsetsatseterror \
--chain-id=blockx_100-1 \
--from=wallet \
--gas=auto \
-y
```
* unjail validator
```
blockxdx slashing unjail --from wallet --chain-id blockx_100-1 --gas-prices 0.5uaum --gas-adjustment 1.2 --gas auto
```
* check jailed reason
```
blockxd query slashing signing-info $(blockxd tendermint show-validator)
```
### Token management

* withdraw rewards
```
blockxd tx distribution withdraw-all-rewards --from wallet --chain-id blockx_100-1  --gas-adjustment 1.2 --gas auto -y
```
* withdraw rewards with comission
```
blockxd tx distribution withdraw-rewards $(blockxd keys show wallet --bech val -a) --commission --from wallet --chain-id blockx_100-1  --gas-adjustment 1.2 --gas auto -y
```
* delegate token to your own validator
```
blockxd tx staking delegate $(blockxd keys show wallet --bech val -a) 1000000abcx --from wallet --chain-id blockx_100-1 --gas-adjustment 1.2 --gas-prices 0.05uaum --gas auto -y
```
### Node Info
* node id
```
blockxd status 2>&1 | jq .NodeInfo
```
* validator info
```
blockxd status 2>&1 | jq .ValidatorInfo
```

### Delete Node
```
sudo systemctl stop blockxd &&
sudo systemctl disable blockxd &&
sudo rm /etc/systemd/system/eblockxd.service &&
sudo systemctl daemon-reload &&
rm -f $(which blockxd) &&
rm -rf .blockxd
```

