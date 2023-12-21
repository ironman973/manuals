```bash
sudo apt update
sudo apt install -y curl git jq lz4 build-essential unzip

bash <(curl -s "https://raw.githubusercontent.com/nodejumper-org/cosmos-scripts/master/utils/go_install.sh")
source .bash_profile

NODE_MONIKER="Your Node Name"

wget https://snapshots-testnet.nodejumper.io/arkeonetwork-testnet/arkeod
chmod +x arkeod
mv arkeod $HOME/go/bin/

arkeod config keyring-backend test
arkeod config chain-id arkeo
arkeod init "$NODE_MONIKER" --chain-id arkeo

curl -s http://seed.arkeo.network:26657/genesis | jq '.result.genesis' > $HOME/.arkeo/config/genesis.json
curl -s https://snapshots-testnet.nodejumper.io/arkeonetwork-testnet/addrbook.json > $HOME/.arkeo/config/addrbook.json

SEEDS="20e1000e88125698264454a884812746c2eb4807@seeds.lavenderfive.com:22856"
PEERS=""
sed -i 's|^seeds *=.*|seeds = "'$SEEDS'"|; s|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.arkeo/config/config.toml

sed -i 's|^pruning *=.*|pruning = "custom"|g' $HOME/.arkeo/config/app.toml
sed -i 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|g' $HOME/.arkeo/config/app.toml
sed -i 's|^pruning-interval *=.*|pruning-interval = "17"|g' $HOME/.arkeo/config/app.toml
sed -i 's|^snapshot-interval *=.*|snapshot-interval = 0|g' $HOME/.arkeo/config/app.toml

sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0001uarkeo"|g' $HOME/.arkeo/config/app.toml
sed -i 's|^prometheus *=.*|prometheus = true|' $HOME/.arkeo/config/config.toml

sudo tee /etc/systemd/system/arkeod.service > /dev/null << EOF
[Unit]
Description=Arkeo Network Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which arkeod) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF

arkeod tendermint unsafe-reset-all --home $HOME/.arkeo --keep-addr-book

SNAP_NAME=$(curl -s https://snapshots-testnet.nodejumper.io/arkeonetwork-testnet/info.json | jq -r .fileName)
curl "https://snapshots-testnet.nodejumper.io/arkeonetwork-testnet/${SNAP_NAME}" | lz4 -dc - | tar -xf - -C "$HOME/.arkeo"

sudo systemctl daemon-reload
sudo systemctl enable arkeod
sudo systemctl start arkeod

sudo journalctl -u arkeod -f --no-hostname -o cat
```
