System Specs
Hardware	Requirement
CPU	4 Cores
RAM	8 GB
Disk	200 GB
Bandwidth	10 MBit/s
https://docs.story.foundation/docs/node-setup

Install dependencies
Terminal window
sudo apt update
sudo apt-get update
sudo apt install curl git make jq build-essential gcc unzip wget lz4 aria2 -y

Download Story-Geth binary v0.9.3
Terminal window
cd $HOME
wget https://story-geth-binaries.s3.us-west-1.amazonaws.com/geth-public/geth-linux-amd64-0.9.3-b224fdf.tar.gz
tar -xzvf geth-linux-amd64-0.9.3-b224fdf.tar.gz
[ ! -d "$HOME/go/bin" ] && mkdir -p $HOME/go/bin
if ! grep -q "$HOME/go/bin" $HOME/.bash_profile; then
  echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
fi
sudo cp $HOME/geth-linux-amd64-0.9.3-b224fdf/geth $HOME/go/bin/story-geth
source $HOME/.bash_profile
story-geth version

banner
Download Story binary v0.10.1
Terminal window
cd $HOME
wget https://story-geth-binaries.s3.us-west-1.amazonaws.com/story-public/story-linux-amd64-0.10.1-57567e5.tar.gz
tar -xzvf story-linux-amd64-0.10.1-57567e5.tar.gz
[ ! -d "$HOME/go/bin" ] && mkdir -p $HOME/go/bin
if ! grep -q "$HOME/go/bin" $HOME/.bash_profile; then
  echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
fi
sudo cp $HOME/story-linux-amd64-0.10.1-57567e5/story $HOME/go/bin
source $HOME/.bash_profile
story version

banner
Init Iliad node
Terminal window
story init --network iliad --moniker "Your_moniker_name"

banner
Create story-geth service file
Terminal window
sudo tee /etc/systemd/system/story-geth.service > /dev/null <<EOF
[Unit]
Description=Story Geth Client
After=network.target

[Service]
User=root
ExecStart=/root/go/bin/story-geth --iliad --syncmode full
Restart=on-failure
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF

Create story service file
Terminal window
sudo tee /etc/systemd/system/story.service > /dev/null <<EOF
[Unit]
Description=Story Consensus Client
After=network.target

[Service]
User=root
ExecStart=/root/go/bin/story run
Restart=on-failure
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF

Use snapshot
Snapshot link

Reload and start story-geth
Terminal window
sudo systemctl daemon-reload && \
sudo systemctl start story-geth && \
sudo systemctl enable story-geth && \
sudo systemctl status story-geth

banner
Reload and start story
Terminal window
sudo systemctl daemon-reload && \
sudo systemctl start story && \
sudo systemctl enable story && \
sudo systemctl status story

banner
Check logs
Terminal window
sudo journalctl -u story-geth -f -o cat

banner
Wait a minute for connect peers

Terminal window
sudo journalctl -u story -f -o cat

banner
Check sync status
Terminal window
curl localhost:26657/status | jq

Check block sync left:
Terminal window
while true; do
    local_height=$(curl -s localhost:26657/status | jq -r '.result.sync_info.latest_block_height');
    network_height=$(curl -s https://rpc-story.josephtran.xyz/status | jq -r '.result.sync_info.latest_block_height');
    blocks_left=$((network_height - local_height));
    echo -e "\033[1;38mYour node height:\033[0m \033[1;34m$local_height\033[0m | \033[1;35mNetwork height:\033[0m \033[1;36m$network_height\033[0m | \033[1;29mBlocks left:\033[0m \033[1;31m$blocks_left\033[0m";
    sleep 5;
done

Create validator
Export validator Public Key & Private key
By default, when you run story init a validator key is created for you. To view your validator key, run the following command:

Terminal window
story validator export

In addition, if you want to export the derived EVM private key of your validator into the default data config directory, please run the following:

Terminal window
story validator export --export-evm-key

Note that to participate in consensus, at least 1 IP must be staked (equivalent to 1000000000000000000 wei)!
Faucet link: https://faucet.story.foundation/

banner
Create validator
Terminal window
story validator create --stake 1000000000000000000 --private-key "your_private_key"

banner
Caution

Backup the validator key:

File location: /root/.story/story/config/priv_validator_key.json
Copy this file to your local machine. Store it carefully; this is the most crucial key for your validator.
Directoryroot
Directory.story
Directorystory
Directoryconfig
priv_validator_key.json
important file
…
Validator Staking
Terminal window
story validator stake \
   --validator-pubkey "VALIDATOR_PUB_KEY_IN_BASE64" \
   --stake 1000000000000000000 \
   --private-key xxxxxxxxxxxxxx

Replace VALIDATOR_PUB_KEY_IN_BASE64 Amount: 1000000000000000000=1 IP Token

banner
Check your Validator on Explorer
Get your validator info:
Terminal window
curl -s localhost:26657/status | jq -r '.result.validator_info'

Result:

Note

{
    "address": "D6F92FD7D0460AA9E4CF4D299FE479E93395DCF3",
    "pub_key": {
          "type": "tendermint/PubKeySecp256k1",
          "value": "A+46wEmBx5QQscNOKhmJgaAQjdr85s1OzvNimMiaysp3"
    },
     "voting_power": "15000"
}

Paste HEX Validator Address: D6F92FD7D0460AA9E4CF4D299FE479E93395DCF3 to search
https://testnet.story.explorers.guru/
banner
# Delete node
Caution

Backup your data, private key, validator key before remove node.

Terminal window
sudo systemctl stop story-geth
sudo systemctl stop story
sudo systemctl disable story-geth
sudo systemctl disable story
sudo rm /etc/systemd/system/story-geth.service
sudo rm /etc/systemd/system/story.service
sudo systemctl daemon-reload
sudo rm -rf $HOME/.story
sudo rm $HOME/go/bin/story-geth
sudo rm $HOME/go/bin/story
