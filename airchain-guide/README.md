### **Recommended Hardware:**

4 Cores, 8GB RAM, 200GB of storage (NVME)

# Installation

```bash
# install dependencies, if needed
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip -y

# install go, if needed
cd $HOME
VER="1.21.3"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
 
# set vars
echo "export WALLET="wallet"" >> $HOME/.bash_profile
echo "export MONIKER="test"" >> $HOME/.bash_profile
echo "export JUNCTION_CHAIN_ID="junction"" >> $HOME/.bash_profile
echo "export JUNCTION_PORT="18"" >> $HOME/.bash_profile
source $HOME/.bash_profile
 
# download binary
cd $HOME
rm -rf $HOME/.junction
wget https://github.com/airchains-network/junction/releases/download/v0.1.0/junctiond
sudo chmod +x junctiond
sudo mv junctiond /usr/local/bin/
 
# config and init app
junctiond init $MONIKER
sed -i -e "s|^node *=.*|node = \"tcp://localhost:${JUNCTION_PORT}657\"|" $HOME/.junction/config/client.toml
 
# download genesis and addrbook
wget -O $HOME/.junction/config/genesis.json https://snapshot.hypechain.org/airchain-testnet/genesis.json
wget -O $HOME/.junction/config/addrbook.json https://snapshot.hypechain.org/airchain-testnet/addrbook.json
 
# set seeds and peers
SEEDS="aeaf101d54d47f6c99b4755983b64e8504f6132d@airchain-testnet-peer.hypechain.org:28656"
PEERS="48887cbb310bb854d7f9da8d5687cbfca02b9968@35.200.245.190:26656,2d1ea4833843cc1433e3c44e69e297f357d2d8bd@5.78.118.106:26656,de2e7251667dee5de5eed98e54a58749fadd23d8@34.22.237.85:26656,1918bd71bc764c71456d10483f754884223959a5@35.240.206.208:26656,ddd9aade8e12d72cc874263c8b854e579903d21c@178.18.240.65:26656,eb62523dfa0f9bd66a9b0c281382702c185ce1ee@38.242.145.138:26656,0305205b9c2c76557381ed71ac23244558a51099@162.55.65.162:26656,086d19f4d7542666c8b0cac703f78d4a8d4ec528@135.148.232.105:26656,3e5f3247d41d2c3ceeef0987f836e9b29068a3e9@168.119.31.198:56256,8b72b2f2e027f8a736e36b2350f6897a5e9bfeaa@131.153.232.69:26656,6a2f6a5cd2050f72704d6a9c8917a5bf0ed63b53@93.115.25.41:26656,e09fa8cc6b06b99d07560b6c33443023e6a3b9c6@65.21.131.187:26656"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.junction/config/config.toml
 
# set custom ports in app.toml
sed -i.bak -e "s|:1317|:${JUNCTION_PORT}317|g;
s|:8080|:${JUNCTION_PORT}080|g;
s|:9090|:${JUNCTION_PORT}090|g;
s|:9091|:${JUNCTION_PORT}091|g;
s|:8545|:${JUNCTION_PORT}545|g;
s|:8546|:${JUNCTION_PORT}546|g;
s|:6065|:${JUNCTION_PORT}065|g" $HOME/.junction/config/app.toml
 
# set custom ports in config.toml file
sed -i.bak -e "s|:26658|:${JUNCTION_PORT}658|g;
s|:26657|:${JUNCTION_PORT}657|g;
s|:6060|:${JUNCTION_PORT}060|g;
s|:26656|:${JUNCTION_PORT}656|g;
s|^external_address = \"\"|external_address = \"$(wget -qO- eth0.me):${JUNCTION_PORT}656\"|g;
s|:26660|:${JUNCTION_PORT}660|g" $HOME/.junction/config/config.toml
 
# config pruning
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.junction/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.junction/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"50\"/" $HOME/.junction/config/app.toml
 
# set minimum gas price, enable prometheus and disable indexing
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "0.00025amf"|g' $HOME/.junction/config/app.toml
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.junction/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.junction/config/config.toml
 
# create service file
sudo tee /etc/systemd/system/junctiond.service > /dev/null <<EOF
[Unit]
Description=Warden node
After=network-online.target
 
[Service]
User=$USER
WorkingDirectory=$HOME/.junction
ExecStart=$(which junctiond) start
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
 
[Install]
WantedBy=multi-user.target
EOF
 
# reset and download snapshot
junctiond tendermint unsafe-reset-all --home $HOME/.junction
 
# enable and start service
sudo systemctl daemon-reload
sudo systemctl enable junctiond
sudo systemctl restart junctiond && sudo journalctl -u junctiond -f

```

---

---

---

# Wallet

### **Create a wallet**

```bash
junctiond keys add $WALLET
```

### **To restore your keys you have created, do this**

```bash
junctiond keys add $WALLET --recover
```

### **Save wallet and validator address**

```bash
WALLET_ADDRESS=$(junctiond keys show $WALLET -a)
VALOPER_ADDRESS=$(junctiond keys show $WALLET --bech val -a)
echo "export WALLET_ADDRESS="$WALLET_ADDRESS >> $HOME/.bash_profile
echo "export VALOPER_ADDRESS="$VALOPER_ADDRESS >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### **Check sync status, once your node is fully synced, the output from above will print "false"**

```bash
junctiond status 2>&1 | jq .sync_info
```

### **After creating a validator, make sure the fund has been funded sucessfully**

```bash
junctiond query bank balances $WALLET_ADDRESS
```

---

---

---

# **Validator**

To create a validator at airchain node, you should import the data of your validator to the **validator.json** file

first of all, you have to execute this to see `validator-pub-key`

```bash
junctiond comet show-validator
```

then, you can see the result such a following output below

```bash
{"@type":"/cosmos.crypto.ed25519.PubKey","key":"xxxx"}
```

after getting the `<validator-pub-key>`, you can insert it to the **validator.json** file

```bash
sudo tee $HOME/.junction/validator.json > /dev/null <<EOF
{
	"pubkey": {"@type":"/cosmos.crypto.ed25519.PubKey","key":"xxxx"},
	"amount": "1000000amf",
	"moniker": "<validator-name>",
	"identity": "optional identity signature (ex. UPort or Keybase)",
	"website": "validator's (optional) website",
	"security": "validator's (optional) security contact email",
	"details": "validator's (optional) details",
	"commission-rate": "0.1",
	"commission-max-rate": "0.2",
	"commission-max-change-rate": "0.01",
	"min-self-delegation": "1"
}
EOF
 
# Create a validator using the JSON configuration
junctiond tx staking create-validator $HOME/.junction/validator.json \
    --from $WALLET \
    --chain-id junction \
	--gas auto \
	--gas-adjustment 1.5 \
	--fees 500amf

```

---

---

---

# Usefull commads

## **Service operations**

```bash
#Check Logs
sudo journalctl -f -u junctiond
 
#Start service
sudo systemnctl start junctiond
 
#Restart Service
sudo systemnctl restart junctiond
```

withdraw all reward

```bash
junctiond tx distribution withdraw-all-rewards --from $WALLET --chain-id junction --gas auto --gas-adjustment 1.5 --fees 500amf 
```

Withdraw rewards and commission from your validator

```
junctiond tx distribution withdraw-rewards $VALOPER_ADDRESS --from $WALLET --commission --chain-id junction --gas auto --gas-adjustment 1.5 --fees 500amf -y 
```

Check wallet balance

```bash
junctiond query bank balances $WALLET
```

Delegate to the validator itself

```bash
junctiond tx staking delegate $(junctiond keys show $WALLET --bech val -a) 1000000amf --from $WALLET --chain-id junction --gas auto --gas-adjustment 1.5 --fees 500amf -y 
```

Redelegate to other validators

```bash
junctiond tx staking redelegate $VALOPER_ADDRESS <TO_VALOPER_ADDRESS> 1000000amf --from $WALLET --chain-id junction --gas auto --gas-adjustment 1.5 --fees 500amf -y 
```

Unbond

```bash
junctiond tx staking unbond $(junctiond keys show $WALLET --bech val -a) 1000000amf --from $WALLET --chain-id junction --gas auto --gas-adjustment 1.5 --fees 500amf -y 
```

Transfer Funds

```bash
junctiond tx bank send $WALLET_ADDRESS <TO_WALLET_ADDRESS> 1000000amf --gas auto --gas-adjustment 1.5 --fees 500amf -y 
```

### **KEY MANAGEMENT**

Create Wallet

```bash
junctiond keys add $WALLET
```

Recover Existing Wallet

```bash
junctiond keys add $WALLET --recover
```

Check All Available Wallet

```bash
junctiond keys list
```

Check wallet funds

```bash
junctiond q bank balances $WALLET_ADDRESS
```

### **VALIDATOR INFORMATION**

create a validator

```bash
junctiond tx staking create-validator \
--amount 1000000amf \
--from $WALLET \
--commission-rate 0.1 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.01 \
--min-self-delegation 1 \
--security-contact ""\
--pubkey $(junctiond tendermint show-validator) \
--moniker $MONIKER \
--identity "" \
--details "a validator blockchain" \
--chain-id junction \
--gas auto \
--gas-adjustment 1.5 \
--fees 500amf
```

edit existing validator

```bash
junctiond tx staking edit-validator \
--commission-rate 0.1 \
--new-moniker $MONIKER \
--identity "" \
--details "a validator blockchain" \
--from $WALLET \
--chain-id junction \
--gas auto \
--gas-adjustment 1.5 \
--fees 500amf \
-y
```

validator status/info

```bash
junctiond status 2>&1 | jq
```

Unjail validator
