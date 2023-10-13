## Install Defund node


```bash
cd $HOME
sudo apt update && sudo apt upgrade -y
sudo apt install make clang pkg-config libssl-dev build-essential git jq ncdu bsdmainutils htop -y < "/dev/null"
```

```bash
cd $HOME
wget -O go1.19.1.linux-amd64.tar.gz https://golang.org/dl/go1.19.1.linux-amd64.tar.gz
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.19.1.linux-amd64.tar.gz && rm go1.19.1.linux-amd64.tar.gz
echo 'export GOROOT=/usr/local/go' >> $HOME/.bash_profile
echo 'export GOPATH=$HOME/go' >> $HOME/.bash_profile
echo 'export GO111MODULE=on' >> $HOME/.bash_profile
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile && . $HOME/.bash_profile
go version
```

```bash
cd $HOME
git clone https://github.com/defund-labs/defund
cd defund
git checkout v0.1.0
make install
cd $HOME
```

```bash
defundd init <<name>> --chain-id defund-private-2
wget -qO $HOME/.defund/config/genesis.json "https://raw.githubusercontent.com/defund-labs/testnet/main/defund-private-2/genesis.json"
wget -O $HOME/.defund/config/addrbook.json https://github.com/88Mikhail88/My_Testnets/blob/main/Defund/addrbook.json
```

```bash
seeds="85279852bd306c385402185e0125dffeed36bf22@38.146.3.194:26656"
peers="d9184a3a61c56b803c7b317cd595e83bbae3925e@194.163.174.231:26677,5e7853ec4f74dba1d3ae721ff9f50926107efc38@65.108.6.45:60556,f114c02efc5aa7ee3ee6733d806a1fae2fbfb66b@65.108.46.123:56656,aa2c9df37e372c7928435075497fb0fb7ff9427e@38.129.16.18:26656,f2985029a48319330b99767d676412383e7061bf@194.163.155.84:36656,daff7b8cbcae4902c3c4542113ba521f968cc3f8@213.239.217.52:29656"
sed -i.default "s/^seeds *=.*/seeds = \"$seeds\"/;" $HOME/.defund/config/config.toml
sed -i "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/;" $HOME/.defund/config/config.toml
```

```bash
recent=100
every=0
interval=10

sed -i.back "s/pruning *=.*/pruning = \"custom\"/g" $HOME/.defund/config/app.toml
sed -i "s/pruning-keep-recent *=.*/pruning-keep-recent = \"$recent\"/g" $HOME/.defund/config/app.toml
sed -i "s/pruning-keep-every *=.*/pruning-keep-every = \"$every\"/g" $HOME/.defund/config/app.toml
sed -i "s/pruning-interval *=.*/pruning-interval = \"$interval\"/g" $HOME/.defund/config/app.toml
```

```bash
echo "[Unit]
Description=defund
After=network.target

[Service]
User=$USER
Type=simple
ExecStart=$(which defundd) start --home $HOME/.defund
Restart=on-failure
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target" > $HOME/defund.service
sudo mv $HOME/defund.service /etc/systemd/system
sudo tee <<EOF >/dev/null /etc/systemd/journald.conf
Storage=persistent
EOF
```

```bash
sudo systemctl restart systemd-journald
sudo systemctl daemon-reload
sudo systemctl enable defund 

sudo systemctl restart defund
```

```bash
defundd keys add <<wallet>>
```

```bash
defundd keys add <<wallet>> --recover
```

```bash
curl -s localhost:26657/status | jq .result.sync_info.catching_up
```

```bash
defundd q bank balances <<address>>
```

```bash
defundd tx staking create-validator \
--moniker="<<name>>" \
--amount=1000000ufetf \
--pubkey=$(defundd tendermint show-validator) \
--chain-id=defund-private-2 \
--commission-max-change-rate=0.01 \
--commission-max-rate=0.20 \
--commission-rate=0.05 \
--min-self-delegation=1 \
--from=<<wallet>> \
--yes 
```
