# Install Storage Node for SAO Network
The Storage Node is a component in the SAO Network that allows you to provide storage for the blockchain. Storage Node will be rewarded based on contribution to the network

This Tutorial for SAO Storage Node Testnet v0.1.3 on Ubuntu 22.04 (Recommend-Tested), Ubuntu 20.04

Reference: https://docs.sao.network/participate-in-sao-network/run-storage-node
# 1. Q & A
Requirements
```
IP: Requires stable public IP
Ports open: 5153, 4001 (Customizable)
Hardware:  Low, No specific requirements
```
Is it possible to run Storage Node and Validator Node on the same server?
```
Yes
```
Is it required to install Consensus Node ?
```
Not sure. According to the document, the SAO team says it's necessary. But testing shows it looks like it can run standalone. This issue will be further updated
```
# 2. Installation Instructions
Update and install support libraries
```
sudo apt update -y
sudo apt upgrade -y
sudo apt install curl make clang pkg-config libssl-dev build-essential git jq ncdu bsdmainutils -y
```
Install Go v1.19.1
```
cd $HOME
wget -O go1.19.1.linux-amd64.tar.gz https://golang.org/dl/go1.19.1.linux-amd64.tar.gz
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.19.1.linux-amd64.tar.gz && rm go1.19.1.linux-amd64.tar.gz
echo 'export GOROOT=/usr/local/go' >> $HOME/.bash_profile
echo 'export GOPATH=$HOME/go' >> $HOME/.bash_profile
echo 'export GO111MODULE=on' >> $HOME/.bash_profile
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.bash_profile && . $HOME/.bash_profile
go version
```
If you want to clear the previous setting data run the following line
```
rm -rf $HOME/.sao-node
```
### Install SAO Storage Node
```
cd $HOME
rm -rf $HOME/sao-node
git clone https://github.com/SAONetwork/sao-node
cd sao-node
git checkout v0.1.3
make
sudo cp ./saonode /usr/local/bin/
sudo cp ./saoclient /usr/local/bin/
saonode -v
saoclient -v
```
Current version
```
#saonode version 0.0.1+git.9dbd3b3
#saoclient version 0.0.1+git.9dbd3b3
```
Service installation
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
sudo systemctl restart systemd-journald
sudo systemctl daemon-reload
sudo systemctl enable saos
```
Done ! Storage Node is ready 

## Init & start node
To launch Storage Node you first need to associate a wallet address for saonode. You can create a new SAO wallet with saonode following the instructions below or reuse an existing wallet address from SAO Consensus (See section 3.1)

### Create a new wallet with saonode
```
saonode account create --key-name wallet
```
Result (Remember to save your wallet address and seed phrase)
```
#Account:  wallet
#Address:  sao1fnslurzu2c5rakume2g70qlzkhtf242qz4nh0r
#Mnemonic:  kick canal follow option unable prison that robot year about first cereal boil still approve blouse crazy fold clerk march daughter silent autumn cement
```
Next, to be able to use this SAO address, you need to have a balance to pay gas fees when joining the network. You can transfer money from another wallet to this wallet or get 1 SAO per day from faucet on Testnet

### Register your Storage Node with the network
```
saonode --chain-address https://rpc-testnet-node0.sao.network:443 init --creator ****YourSaoAddress****
```
enter "yes" to agree to create. If successful you will receive a Hash
### Test your Storage Node
```
saonode --vv run
```
It will take about a few minutes for Storage to connect with other Nodes in the network, At this point your Node has not yet joined the network. Pay attention to the status line when your node starts joining the network.
```
repo: /root/.sao-node, Remote: https://rpc-testnet-node0.sao.network:443, WsEndpoint： /websocket
node[sao1fnslurzu2c5r****YourSaoAddress****242qz4nh0r] is joining SAO network...
```
If you pass this line successfully without error, your Storage Node has officially joined successfully and started working in the network. We have finished testing, You can turn it off to switch to starting Storage Node Service.

### Start Storage Node Service (saos)
```
sudo systemctl start saos
```
### Done ! 

# 3. Management and supervision
Check Storage Node status and reward 
```
saonode info
```
* Note: status 1111 means everything works fine, Node Pledge and Reward status will be available in a few hours from launch


Check Node service status
```
sudo systemctl status saos
```
Restart Node service
```
sudo systemctl restart saos
```
Check Node log
```
journalctl -u saos -f
```
Check the actual amount of data SAO Network is storing on your node
```
du -sh ~/.sao-node/ipfs
```
## 3.1 Export & Import Wallet
saonode currently only supports importing and exporting "private keys", does not support Seed Phrase like Consensus Node

Export private key from Consensus Node(Validator)
```
saod keys export yourWalletName
```
Export private key from saonode (Storage Node)
```
saonode account export --key-name yourWalletName
```
Import private key into saonode (Storage Node)
```
saonode account import --key-name yourWalletName
```
# 4. Error handling
Error while registering Node
```
Error: repo at '~/.sao-node' is already initialized
```
* -> Just delete it "rm -rf $HOME/.sao-node"

Error when starting Node, can't join the network
```
node not found: unknown request: chain error: code: (11012) desc: failed to process the tx
```
* -> You have not successfully registered Node, misconfigured the network, or some other reason. To fix it delete the folder '~/.sao-node' and proceed to re-register node

### 4.1 Too many failed stream creation errors while node is running
```
dial backoff: network error: code: (15005) desc: failed to create the stream
node    node/node.go:392        invalid transport server addressa=/ip4/***.***.***.***
```
-> If you do not have a public IP address or do not open Port 5153, then other computers on the network will not be able to connect to you. After a while your Node will be banned by the network. To resolve this error, check the following instructions:
* First of all, check if your IP address and Port when reporting to the network are correct. 
```
saonode info
```
If you are using a home network without a direct connection to the internet, by default saonode will not recognize your public IP, instead it will be a Lan IP of the form 192.168.1.x. To fix this, you will need to declare the --multiaddr \<multiaddr\> parameter when registering node. Regarding ports, if using home network you will need to configure port forwarding on your internet modem.
  
* Next, check to make sure both you and the other party have opened the port. (can be checked at: https://www.yougetsignal.com/tools/open-ports/)
If you've opened the Port and the other party doesn't open it, it's not your fault, and you don't need to worry about it. If otherwise they are open they may have blocked you because of too many failed connections

* After solving the problem, restart the "saos" service and wait for other Nodes in the network to accept you again
  
# 5. Remove Storage Node
```
rm -rf $HOME/sao-node
rm -rf $HOME/.sao-node
rm -rf /usr/local/bin/saonode
rm -rf /usr/local/bin/saoclient
```
# The end !
A Tutorial By BitBeri.com
