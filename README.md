
# Andromeda Testnet galileo-2 Validator Node Setup

## Install Ubuntu 20.04 on a new server and login as root

## Install ``ufw`` firewall and configure the firewall

```
apt-get update
apt-get install ufw
ufw default deny incoming
ufw default allow outgoing
ufw allow 22
ufw allow 26656
ufw enable
```

## Create a new User

```
# add user
adduser node

# add user to sudoers
usermod -aG sudo node

# login as user
su - node
```

## Install Prerequisites

```
sudo apt update
sudo apt install make pkg-config build-essential libssl-dev curl jq git chrony libleveldb-dev -y
sudo apt-get install manpages-dev -y

# Increase the default open files limit (This is to make sure that the nodes won't crash once the network grows larger and larger.)
sudo su -c "echo 'fs.file-max = 65536' >> /etc/sysctl.conf"
sudo sysctl -p

# Install go
curl https://dl.google.com/go/go1.18.3.linux-amd64.tar.gz | sudo tar -C/usr/local -zxvf -

# Update environment variables to include go
cat <<'EOF' >>$HOME/.profile
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
EOF

source $HOME/.profile

# check go version
go version

```

## Install Ignite CLI

```
sudo curl https://get.ignite.com/cli! | sudo bash
```


## Install Andromeda Testnet Node

```
git clone https://github.com/andromedaprotocol/chain.git
cd chain
git checkout galileo-2-final
ignite chain build
cd
```

## Initialise your Validator node
```
#Choose a name for your validator and use it in place of “<moniker-name>” in the following command:
andromedad init <moniker-name> --chain-id galileo-2

#Example
acred init Synergy_Nodes --chain-id galileo-2
```

## Download genesis.json file
```
git clone https://github.com/andromedaprotocol/testnets.git
rm $HOME/.andromedad/config/genesis.json
cp $HOME/testnets/galileo-2/genesis.json $HOME/.andromedad/config/
```

## Add Peers to config.toml

```
cd

PEERS="6f39c4e2f38ebffc09e7221f33c4a64626f20b7b@107.191.40.4:26656,e34a7f0750b2692c02b3d072a51250fd5560f1e8@46.101.82.159:26656,be7d935eaab980319b3ca1b3ca189f1c7ee8c334@213.239.213.149:26656,77c04ed628aa56322c45b8da14b8567c1bf322e5@65.108.98.53:11056,d22bf273fc96fd6b184333271aa1e979e10876ff@188.40.140.51:21000,b307ba31eab05fd93b52eead9b61923f6382a8bd@188.40.140.51:21001"

sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.acred/config/config.toml
```

## Start the node and let it Sync
```
andromedad start

# Press Ctrl+c to exit

```

## State Sync

For State Sync, please follow the instructions provided at the following link:

https://testnet-ping.wildsage.io/andromeda/statesync

## Running the validator as a systemd unit
```
cd /etc/systemd/system
sudo nano andromedad.service
```
Copy the following content into ``andromedad.service`` and save it.

```
[Unit]
Description=Andromedad Daemon
After=network.target

[Service]
Type=simple
User=node
ExecStart=/home/node/go/bin/andromedad start
Restart=on-abort

[Install]
WantedBy=multi-user.target

[Service]
LimitNOFILE=1048576
```

```
sudo systemctl daemon-reload
sudo systemctl enable andromedad

# Start the service
sudo systemctl start andromedad

# Stop the service
sudo systemctl stop andromedad

# Restart the service
sudo systemctl restart andromedad


# For Entire log
journalctl -t andromedad

# For Entire log reversed
journalctl -t andromedad -r

# Latest and continuous
journalctl -t andromedad -f
```

## Execute the folloiwng command to get the node id

```
andromedad tendermint show-node-id
```

## Create a Wallet for your Validator Node

Make sure to copy the 24 words Mnemonics Phrase, save it in a file and store it on a safe location.

```
andromedad keys add <wallet-name>

#Example
andromedad keys add my_wallet

```

Get ANDR tokens to the above created wallet from Faucet present in Andromeda Discord.


## Create and Register Your Validator Node
```
andromedad tx staking create-validator -y \
  --amount=5000000uandr \
  --pubkey=$(andromedad tendermint show-validator) \
  --moniker=<moniker-name> \
  --identity="<Keybase.io ID>" \
  --website="<website-address>" \
  --chain-id=galileo-2 \
  --commission-rate="0.05" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
  --gas="700000" \
  --from=<wallet-name>
  
#Example

andromedad tx staking create-validator -y \
  --amount=5000000uandr \
  --pubkey=$(andromedad tendermint show-validator) \
  --moniker="My Node" \
  --identity "AF9D7EF7CC70CE24" \
  --website "https://www.yourwebsite.com" \
  --chain-id=galileo-2 \
  --commission-rate="0.05" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
  --gas="700000" \
  --from=my_wallet
  
```

## Get Validator Operator Address (Valoper Address)

Make sure to change ``<wallet-name>`` to correct values.

```
andromedad keys show <wallet-name> --bech val --output json | jq -r .address

#Example
andromedad keys show my_wallet --bech val --output json | jq -r .address
```

## Delegate ``ANDR`` to Your Node
```
andromedad tx staking delegate <validator-address> 1000000uandr --from <wallet-name> --chain-id galileo-2 -y

#Example
andromedad tx staking delegate andrvaloper1xesqr8vjvy34jhu027zd70ypl0nnev5ehskd2u 1000000uandr --from my_wallet --chain-id galileo-2 -y
```
## Backup Validator node file

Take a backup of the following files after you have created and registered your validator node successfully.

```
/home/node/.andromedad/config/node_key.json
/home/node/.andromedad/config/priv_validator_key.json
/home/node/.andromedad/data/priv_validator_state.json
```

## Withdraw Rewards

Make sure to change ``<validator-operator-address>``, ``<wallet-name>`` to correct values.

```
andromedad tx distribution withdraw-rewards <validator-address> --from <wallet-name> --chain-id=galileo-2 -y

#Example
andromedad tx distribution withdraw-rewards andrvaloper1xesqr8vjvy34jhu027zd70ypl0nnev5ehskd2u --from synergy --chain-id=galileo-2 -y
```

## Check Balance of an Address

```
andromedad query bank balances <wallet-address> --chain-id galileo-2

#Example:
andromedad query bank balances andr1xesqr8vjvy34jhu027zd70ypl0nnev5e7kqmjx --chain-id galileo-2
```
