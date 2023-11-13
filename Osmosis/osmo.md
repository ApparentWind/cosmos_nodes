# Osmosis
## Packages
```bash
sudo apt update && sudo apt upgrade -y
```
## Dependencies
```bash
sudo apt install curl build-essential git wget jq make gcc tmux chrony -y
```
## Go
```bash
if ! [ -x "$(command -v go)" ]; then
  ver="1.18.2"
  cd $HOME
  wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
  sudo rm -rf /usr/local/go
  sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
  rm "go$ver.linux-amd64.tar.gz"
  echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
  source ~/.bash_profile
fi
```
## Go
```bash
ver="1.19.1" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```
## Moniker, whrite some name
```bash
NODENAME=Copypaster
```
## Make your custom ports. You can chose from 10 to 65
```bash
OSMOSIS_PORT=44
```
## Save and import variables.

```bash
echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
if [ ! $WALLET ]; then
	echo "export WALLET=wallet" >> $HOME/.bash_profile
fi
echo "export OSMOSIS_CHAIN_ID=osmosis-1" >> $HOME/.bash_profile
echo "export OSMOSIS_PORT=${OSMOSIS_PORT}" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
## Binaries.

```bash
cd $HOME
git clone https://github.com/osmosis-labs/osmosis.git
cd osmosis
git checkout v1.0.2
make install
```

## Config app.

```bash
osmosisd config chain-id $OSMOSIS_CHAIN_ID
osmosisd config node tcp://localhost:${OSMOSIS_PORT}657
```
## For your own risk.
```bash
osmosisd config keyring-backend file
```

## Init app

```bash
osmosisd init $NODENAME --chain-id $OSMOSIS_CHAIN_ID
```
## Seeds and peers

```bash
SEEDS=""
PEERS=""
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.osmosisd/config/config.toml
```
## Genesis.

```bash
wget -O ~/.osmosisd/config/genesis.json https://github.com/osmosis-labs/networks/raw/main/osmosis-1/genesis.json
```

## Custom ports. 

```bash
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${OSMOSIS_PORT}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${OSMOSIS_PORT}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${OSMOSIS_PORT}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${OSMOSIS_PORT}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${OSMOSIS_PORT}660\"%" $HOME/.osmosisd/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${OSMOSIS_PORT}317\"%; s%^address = \":8080\"%address = \":${OSMOSIS_PORT}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${OSMOSIS_PORT}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${OSMOSIS_PORT}091\"%" $HOME/.osmosisd/config/app.toml
```
## Pruning.

```bash
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.osmosisd/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.osmosisd/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.osmosisd/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.osmosisd/config/app.toml
```
## Minimum gas price.

```bash
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.001uosmo\"/" $HOME/.osmosisd/config/app.toml
```

## Prometheus.

```bash
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.osmosisd/config/config.toml
```
## Reset chain data.

```bash
osmosisd unsafe-reset-all --home $HOME/.osmosisd 
```

## AddrBook.

```bash
wget -O addrbook.json https://snapshots.polkachu.com/addrbook/osmosis/addrbook.json --inet4-only
mv addrbook.json ~/.osmosisd/config
```

## Service.

```bash
sudo tee /etc/systemd/system/osmosisd.service > /dev/null <<EOF
[Unit]
Description=Fire_starter
After=network-online.target

[Service]
User=$USER
ExecStart=$(which osmosisd) start --home $HOME/.osmosisd
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Register and start service.

```bash
sudo systemctl daemon-reload
sudo systemctl enable osmosisd
sudo systemctl restart osmosisd && sudo journalctl -u osmosisd -f -o cat
