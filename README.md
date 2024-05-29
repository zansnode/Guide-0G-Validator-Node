# Guide-0G-Validator-Node

## Installation Guide

### Install required packages
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget build-essential jq make lz4 gcc unzip -y
```

### Install Go
```
cd $HOME && \
ver="1.21.5" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```

### Build And Install Binary
```
cd $HOME && mkdir -p go/bin/
git clone -b v0.1.0 https://github.com/0glabs/0g-chain.git
./0g-chain/networks/testnet/install.sh
source .profile
```

### Moniker
```
0gchaind config chain-id zgtendermint_16600-1
0gchaind init YOUR-MONIKER-NAME --chain-id zgtendermint_16600-1
```

### Set up configuration
```
rm ~/.0gchain/config/genesis.json
curl -Ls https://github.com/0glabs/0g-chain/releases/download/v0.1.0/genesis.json > $HOME/.0gchain/config/genesis.json
curl -Ls https://raw.githubusercontent.com/Core-Node-Team/Testnet-TR/main/0G-Newton/addrbook.json > $HOME/.0gchain/config/addrbook.json
```
```
rm ~/.0gchain/config/genesis.json
wget -P ~/.0gchain/config https://github.com/0glabs/0g-chain/releases/download/v0.1.0/genesis.json
```

### Set up PEERS
```
PEERS="3385a1357d2ea36c93c35ee8624a9ae1b67c0300@77.237.240.30:26656,2e408c120713ddae88fe73ec47417bb039733b50@193.233.80.119:26656,2cc75d1951d3d6172aee420b713c5b2153bd3402@185.103.103.77:26656,7ad0ff034837e638e041d567c20c9f9443a2f027@135.181.2.110:26656,84f7f5739cca6312d13634ccd911cceac57b8065@193.43.147.177:12656,0f2be6b6c5db0edc217b599e9f2f9800048c4394@37.27.91.167:26656,b1b5a0999cb6e810886ceb95655e15308093bfe1@195.26.247.160:26656,bf668d127a52b8543c3b5f2a3b01f8bb79eb05a7@109.199.112.123:26656,81f13ba298ad3b8bb7eea0edefc0bfedcf947d40@84.247.131.34:26656,e359556f70f0579547dbcae8630fea6d6d07b7d2@158.220.126.40:16656,ac2a36a8a0d3bf08f10190400c5c8c3a11170de2@5.9.147.138:32656,ca0407b8b0b1e4750ef7d412d9c447f8b5458bdf@95.216.9.81:12656,e224629bf2b905628647b3cb725e20c2182e6e2f@158.220.114.24:26656,c7d87004d662c3598c5e64db9ce89db1a25b96e3@94.72.118.211:26656,79a7bc9e2a3329720b3ab3c69cea06e9359f8f8e@144.76.185.136:26656,39685773a164c8223eb694aa7e3e46e032a2113e@185.237.253.10:26656,f33aaef3ad68cf43a334adc53a731cfe77e77959@135.181.232.227:12656,0e4d302242b59508fc8b84b2c867e5f4a2befa05@173.249.23.220:12656,2d370ba469a7af6408e1cefda7a97adaf30dc81d@156.67.29.90:26656,ba35899877f815a009c513626835fd25e146b16e@37.27.120.100:26656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.0gchain/config/config.toml
```

### Set Up Gas Prices
```
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "0agnet"|g' $HOME/.0gchain/config/app.toml
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.0gchain/config/config.toml
```

### Set Up pruning
```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "19"|' \
  $HOME/.0gchain/config/app.toml
```

### Create Service A File
```
sudo tee /etc/systemd/system/0gchaind.service > /dev/null <<EOF
[Unit]
Description=0gchaind
After=network-online.target

[Service]
User=$USER
ExecStart=$(which 0gchaind) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

### Reload And Start The Service
```
sudo systemctl daemon-reload
sudo systemctl enable 0gchaind  
sudo systemctl restart 0gchaind  && sudo journalctl -fu 0gchaind  -o cat
```

## WALLET

### New Wallet
```
0gchaind keys add wallet
```
Note: dont forget to save your pharse, this is the only way to recovery your wallet!!!!

### Recovery Wallet
```
0gchaind keys add wallet --recover
```

### Check wallet balance 
```
0gchaind q bank balances $(0gchaind keys show wallet -a)
```

### Faucet
https://faucet.0g.ai/

## Validator

### Create Validator
```
0gchaind tx staking create-validator \
  --amount=900000ua0gi \
  --pubkey=$(0gchaind tendermint show-validator) \
  --moniker="YOUR-MONIKER" \
  --chain-id=zgtendermint_16600-1 \
  --commission-rate=0.05 \
  --commission-max-rate=0.1 \
  --commission-max-change-rate=0.1 \
  --min-self-delegation=1 \
  --from=wallet \
  --identity="" \
  --details="" \
  --website="" \
  --gas=500000 --gas-prices=99999ua0gi
  -y
```
Note: DONT FORGET SAVE YOUR PRIV_KEY_VALIDATOR.JSON, THIS IS THE ONLY WAY TO RESTORE YOUR VALIDATOR!!!!

## Another Command

### Edit Validator
```
0gchaind tx staking edit-validator \
--new-moniker "nama-moniker" \
--identity "keybase-id" \
--details "detailed-info" \
--website "website-link" \
--security-contact "email-address" \
--chain-id $CHAIN_ID \
--commission-rate 0.10 \
--from wallet \
--gas auto \
--gas-adjustment 1.4 \
--fees=800ua0gi \
-y
```

### Node info
```
0gchaind status 2>&1 | jq
```

### Validator info
```
0gchaind q staking validator $(0gchaind keys show wallet --bech val -a)
```

### Delegate to your validator
```
0gchaind tx staking delegate $(0gchaind keys show WALLET_NAME --bech val -a) 1000000ua0gi --from wallet -y
```

### Unjail Validator 
```
0gchaind tx slashing unjail --from WALLET_NAME --gas=500000 --gas-prices=99999neuron -y
```

Validator jail reason
```
0gchaind q slashing signing-info $(0gchaind tendermint show-validator)
```

### Stop Node
```
sudo systemctl stop 0gchaind
```

### Restart Node
```
sudo systemctl restart 0gchaind
```

### Disable Indexing
```
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.0gchain/config/config.toml
```

### Delete Node
```
sudo systemctl stop 0gchaind.service
sudo systemctl disable 0gchaind.service
sudo rm /etc/systemd/system/0gchaind.service
rm -rf $HOME/.0gchain $HOME/0g-chain
```
NOTE: You've make sure that if you already save important file before "DELETE NODE"
