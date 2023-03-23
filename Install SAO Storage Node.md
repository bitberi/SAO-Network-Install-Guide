# Install Storage Node for SAO Network

sudo apt update
sudo apt upgrade

sudo apt install curl make clang pkg-config libssl-dev build-essential git jq ncdu bsdmainutils htop -y < "/dev/null"

```
echo -e '\n\e[42mInstall Go\e[0m\n' && sleep 1
cd $HOME
wget -O go1.19.1.linux-amd64.tar.gz https://golang.org/dl/go1.19.1.linux-amd64.tar.gz
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.19.1.linux-amd64.tar.gz && rm go1.19.1.linux-amd64.tar.gz
echo 'export GOROOT=/usr/local/go' >> $HOME/.bash_profile
echo 'export GOPATH=$HOME/go' >> $HOME/.bash_profile
echo 'export GO111MODULE=on' >> $HOME/.bash_profile
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile && . $HOME/.bash_profile
go version
```

```
cd $HOME
echo -e '\n\e[42mInstall software\e[0m\n' && sleep 1
rm -rf $HOME/sao-node
git clone https://github.com/SAONetwork/sao-node
cd sao-node
git checkout v0.1.3
make build
#sudo cp ./saonode /usr/local/bin/ || exit
#sudo cp ./saoclient /usr/local/bin/ || exit
./saonode --chain-address https://rpc-testnet-node0.sao.network:443 init --creator sao1address
```

```
echo "[Unit]
Description=SAO Storage Node
After=network.target

[Service]
User=$USER
Type=simple
ExecStart=$HOME/sao-node/saonode run
Restart=on-failure
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target" > $HOME/saos.service
sudo mv $HOME/saos.service /etc/systemd/system
sudo tee <<EOF >/dev/null /etc/systemd/journald.conf
Storage=persistent
EOF
echo -e '\n\e[42mRunning a service\e[0m\n' && sleep 1
sudo systemctl restart systemd-journald
sudo systemctl daemon-reload
sudo systemctl enable saos
sudo systemctl restart saos
```

```
/usr/local/bin/
```
