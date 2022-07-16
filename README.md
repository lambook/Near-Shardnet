
# near-node-setup-shardnet

In this guide, we will show how to setup a running node in Near Shardnet network.

## Hardware

**[Stake Wars](https://medium.com/nearprotocol/join-stake-wars-to-become-a-chunk-only-producer-52cb67b19f19)** main objective is to set up validator nodes called Chunk-Only Producers whose main purpose is to produce [chunks](https://near.org/papers/nightshade/#nightshade) in a single shard (a network partition).
These validators can mount their nodes on machines with less constraining specifications compared to a full validating node [hardware requirement](https://near-nodes.io/validator/hardware) on the Near mainnet network.
For this guide, we are using a [Hetzner](https://hetzner.com) cloud server with the following specifications :

- Server AMD Ryzen™ 5 3600
- CPU:  6 cores / 12 threads @ 3.6 GHz
- 64 GB DDR4 RAM
- 2 x 512 GB NVMe SSD (Raid1)
- price : 36 euros/month


![server](images/node/server_hetzner.png "server")



## Create your Shardnet wallet

In this section, we will learn how to create a Shardnet wallet

1- Navigate to  https://wallet.shardnet.near.org and press Create account

![server](images/wallet/wallet_01.png "server")

2-In the next page, Enter your Account-ID and press 'Reserve my Account-ID'

![server](images/wallet/wallet_02.png "server")

3- Choose a security method, 'Secure paraphrase' is the safest and the recommended one

![server](images/wallet/wallet_03.png "server")

4-Set up you security paraphrase (seed), you should keep it for yourself and never share it. Copy the paraphrase.
You will be asked to verify it ine the next step

![server](images/wallet/wallet_04.png "server")

5- You are asked to verify a given work from your secure paraphrase, enter it and press verify and complete

![server](images/wallet/wallet_05.png "server")

6-Now , you are asked to verify the whole 12 work of teh paraphrase, enter them all and press Find my Account

![server](images/wallet/wallet_07.png "server")

7-Your account has been created.You can see your Account-ID in the top right corner of the browser

![server](images/wallet/wallet_08.png "server")




## How Setup a non root user


In the following sections, **It's recommended** to run the commands  as a non-root user

* Login as root and create a user

   ```bash
   root@Ubuntu-2004-focal-64-minimal ~ # sudo adduser nearuser
    ```
    add the user to the sudoers (nano or vim)
   ```bash
   root@Ubuntu-2004-focal-64-minimal ~ # EDITOR=vim visudo  
   ```
* Scroll down next to the line :
   ```bash
    root    ALL=(ALL:ALL) ALL
   ```
* Add the line :

   ```bash
   nearuser  ALL=(ALL) NOPASSWD:ALL
   ```
* Save a file and quit the editor.


# Deploy the node

## setup NEAR-CLI

Sync and install the newest versions of all installed packages on the linux machine

   ```bash
   ouiouane@Ubuntu-2004-focal-64-minimal:~$ sudo apt update && sudo apt upgrade -y
   ```

Install Node.js and npm

   ```bash
   curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -  
   sudo apt install build-essential nodejs
   PATH="$PATH"
   ```

Check Node.js and npm versions:
  ```bash
  ouiouane@Ubuntu-2004-focal-64-minimal:~$ node -v
  v18.6.0
  ```

   ```bash
  ouiouane@Ubuntu-2004-focal-64-minimal:~$ npm -v
  8.13.2   
  ```

Install NEAR-CLI
  ```bash
   ouiouane@Ubuntu-2004-focal-64-minimal:~$ sudo npm install -g near-cli
  ```

Set Network to Shardnet
In order to make it persistent, add it to ~/.bashrc.

  ```bash
  ouiouane@Ubuntu-2004-focal-64-minimal:~$ echo 'export NEAR_ENV=shardnet' >> ~/.bashrc
  ```

Near-Cli is now installed, you can check by running the following commands for example:


  ```bash
  ouiouane@Ubuntu-2004-focal-64-minimal:~$ near validators current 
  ```   

lists the current validators set


![node_01](images/node/node_01.png "node_01")



###Setup node validator

#### Install dependencies  and set the configuration

first, check that you have the required hardware specifications to run a node

  ```bash
  ouiouane@Ubuntu-2004-focal-64-minimal:~$ lscpu | grep -P '(?=.*avx )(?=.*sse4.2 )(?=.*cx16 )(?=.*popcnt )' > /dev/null \
  >   && echo "Supported" \
  >   || echo "Not supported"
  Supported
  ```
Install developer tools:

   ```bash
  ouiouane@Ubuntu-2004-focal-64-minimal:~$ sudo apt install -y git binutils-dev libcurl4-openssl-dev zlib1g-dev libdw-dev libiberty-dev cmake gcc g++ python docker.io protobuf-compiler libssl-dev pkg-config clang llvm cargo
  ```
Install Python pip

  ```bash
  ouiouane@Ubuntu-2004-focal-64-minimal:~$ sudo apt install python3-pip
  ```

Set the configuration

  ```bash
  ouiouane@Ubuntu-2004-focal-64-minimal:~$ USER_BASE_BIN=$(python3 -m site --user-base)/bin
  ouiouane@Ubuntu-2004-focal-64-minimal:~$ export PATH="$USER_BASE_BIN:$PATH"

  ```
Install Building env

  ```bash
  ouiouane@Ubuntu-2004-focal-64-minimal:~$ sudo apt install clang build-essential make
  ```

Install Rust & Cargo

  ```bash
  ouiouane@Ubuntu-2004-focal-64-minimal:~$ curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
  ```

you will see the following output , press 1 and press enter :

 ```bash
  1) Proceed with installation (default)
  2) Customize installation
  3) Cancel installation
  >1

  info: profile set to 'default'
  info: default host triple is x86_64-unknown-linux-gnu
  info: syncing channel updates for 'stable-x86_64-unknown-linux-gnu'
  info: latest update on 2022-06-30, rust version 1.62.0 (a8314ef7d 2022-06-27)
  info: downloading component 'cargo'
  info: downloading component 'clippy'
  info: downloading component 'rust-docs'
  info: downloading component 'rust-std'
  info: downloading component 'rustc'
  info: downloading component 'rustfmt'
  info: installing component 'cargo'
  info: installing component 'clippy'
  info: installing component 'rust-docs'
  18.3 MiB /  18.3 MiB (100 %)  13.9 MiB/s in  1s ETA:  0s
  info: installing component 'rust-std'
  26.0 MiB /  26.0 MiB (100 %)  16.5 MiB/s in  1s ETA:  0s
  info: installing component 'rustc'
  54.1 MiB /  54.1 MiB (100 %)  19.1 MiB/s in  2s ETA:  0s
  info: installing component 'rustfmt'
  info: default toolchain set to 'stable-x86_64-unknown-linux-gnu'

  stable-x86_64-unknown-linux-gnu installed - rustc 1.62.0 (a8314ef7d 2022-06-27)
  Rust is installed now. Great!

  To get started you may need to restart your current shell.
  This would reload your PATH environment variable to include
  Cargo's bin directory ($HOME/.cargo/bin).

  To configure your current shell, run:
  source "$HOME/.cargo/env"
  ```

Source the environment
  ```bash
  ouiouane@Ubuntu-2004-focal-64-minimal:~$ source $HOME/.cargo/env
  ``` 

Clone nearcore project from GitHub :

  ```bash
  ouiouane@Ubuntu-2004-focal-64-minimal:~$ git clone https://github.com/near/nearcore
   ```  
then

   ```bash
   ouiouane@Ubuntu-2004-focal-64-minimal:~$ cd nearcore
   ouiouane@Ubuntu-2004-focal-64-minimal:~/nearcore$ git fetch
   ``` 

checkout to the master branch

   ```bash
   ouiouane@Ubuntu-2004-focal-64-minimal:~/nearcore$ git checkout master
   Already on 'master'
   Your branch is up to date with 'origin/master'.
   ``` 
Compile nearcore binary, run the following commands in the nearcore folder :

   ```bash
   ouiouane@Ubuntu-2004-focal-64-minimal:~/nearcore$ cargo build -p neard --release --features shardnet
   ``` 
The binary path is set to target/release/neard

Initialize working directory
Generate the initial required working directory and a couple of configuration files:

* config.json : contains needed information for a node to run on the network, how to communicate with peers, and how to reach consensus
* genesis.json : contains initial accounts, contracts, access keys, and other records which represents the initial state of the blockchain
* node_key.json : contains a public and private key for the node. Also includes an optional account_id parameter which is required to run a validator node.
* data/ : Near node state.

Run the following command to generate working directory

   ```bash
   ouiouane@Ubuntu-2004-focal-64-minimal:~/nearcore$./target/release/neard --home ~/.near init --chain-id shardnet --download-genesis 
   ```
produces the following output :
   ```bash
    2022-07-15T19:19:26.243764Z  INFO neard: version="trunk" build="crates-0.14.0-216-g96f13d239" latest_protocol=100
    2022-07-15T19:19:26.244006Z  INFO near: Using key ed25519:14Pbi1rtMo4s9PqxfxB3mLxz4FQdKQaDVm1KYKU7sT1S for node
    2022-07-15T19:19:26.244057Z  INFO near: Downloading genesis file from: https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/shardnet/genesis.json.xz ...
    [00:00:00] [####################################################################################################################################################################################] 1.11KB/1.11KB [5.82MB/s] (0s)
    2022-07-15T19:19:27.068464Z  INFO near: Saved the genesis file to: /home/ouiouane/.near/genesis.json ...
    2022-07-15T19:19:27.082573Z  INFO near: Generated for shardnet network node key and genesis file in /home/ouiouane/.near
   ```

Replace the config.json
in config.json, modify the 2 parameters : boot_nodes and tracked_shards

   ````bash
   ouiouane@Ubuntu-2004-focal-64-minimal:~/nearcore$ rm ~/.near/config.json
   ouiouane@Ubuntu-2004-focal-64-minimal:~/nearcore$ wget -O ~/.near/config.json https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/shardnet/config.json
   --2022-07-15 21:22:48--  https://s3-us-west-1.amazonaws.com/build.nearprotocol.com/nearcore-deploy/shardnet/config.json
   Resolving s3-us-west-1.amazonaws.com (s3-us-west-1.amazonaws.com)... 52.219.116.232
   Connecting to s3-us-west-1.amazonaws.com (s3-us-west-1.amazonaws.com)|52.219.116.232|:443... connected.
   HTTP request sent, awaiting response... 200 OK
   Length: 3647 (3.6K) [application/json]
   Saving to: ‘/home/ouiouane/.near/config.json’

   /home/ouiouane/.near/config.json                         100%[===============================================================================================================================>]   3.56K  --.-KB/s    in 0s

   2022-07-15 21:22:49 (112 MB/s) - ‘/home/ouiouane/.near/config.json’ saved [3647/3647]
   ````

Get latest snapshot

Install aws Cli
  ````bash
  ouiouane@Ubuntu-2004-focal-64-minimal:~/nearcore$ sudo apt-get install awscli -y
  ````
then

  ````bash
  ouiouane@Ubuntu-2004-focal-64-minimal:~/nearcore$ cd ~/.near
  ouiouane@Ubuntu-2004-focal-64-minimal:~/.near$ aws s3 --no-sign-request cp s3://build.openshards.io/stakewars/shardnet/data.tar.gz .
  tar -xzvf data.tar.gz
   ````

Run the node
You can start your node by  running the following command under nearcore folder:
  ````bash
  ouiouane@Ubuntu-2004-focal-64-minimal:~/nearcore$./target/release/neard --home ~/.near run
  ````
The command produces the following output:

  ````bash
  ouiouane@Ubuntu-2004-focal-64-minimal:~/nearcore$ sudo ./target/release/neard --home ~/.near run
  2022-07-15T19:34:43.519846Z  INFO neard: version="trunk" build="crates-0.14.0-216-g96f13d239" latest_protocol=100
  2022-07-15T19:34:43.527357Z  INFO db: Created a new RocksDB instance. num_instances=1
  2022-07-15T19:34:43.527828Z  INFO db: Dropped a RocksDB instance. num_instances=0
  2022-07-15T19:34:43.527836Z  INFO near: Opening RocksDB database path=/home/ouiouane/.near/data
  2022-07-15T19:34:43.594753Z  INFO db: Created a new RocksDB instance. num_instances=1
  2022-07-15T19:34:43.612921Z  INFO near_network::peer_manager::peer_manager_actor: Bandwidth stats total_bandwidth_used_by_all_peers=0 total_msg_received_count=0 max_max_record_num_messages_in_progress=0
  2022-07-15T19:34:43.620805Z  INFO stats: #  941069 Waiting for peers 0 peers ⬇ 0 B/s ⬆ 0 B/s 0.00 bps 0 gas/s CPU: 0%, Mem: 60.1 MB
  2022-07-15T19:34:43.620858Z DEBUG stats: EpochId(`Bz9n5HbzLd5Bmsuoc15xbKkJnuuMgNJm8YYb2qMngT2E`) Blocks in progress: 0 Chunks in progress: 0 Orphans: 0
  2022-07-15T19:34:53.621893Z  INFO stats: #  941069 Waiting for peers 0 peers ⬇ 0 B/s ⬆ 0 B/s 0.00 bps 0 gas/s CPU: 92%, Mem: 155 MB
  2022-07-15T19:34:53.621944Z DEBUG stats: EpochId(`Bz9n5HbzLd5Bmsuoc15xbKkJnuuMgNJm8YYb2qMngT2E`) Blocks in progress: 0 Chunks in progress: 0 Orphans: 0
  2022-07-15T19:35:03.622780Z  INFO stats: #  941069 Downloading headers 1.73% (29355 left; at 941585) 5 peers ⬇ 109 kB/s ⬆ 109 kB/s 0.00 bps 0 gas/s CPU: 51%, Mem: 278 MB
  2022-07-15T19:35:03.622832Z DEBUG stats: EpochId(`Bz9n5HbzLd5Bmsuoc15xbKkJnuuMgNJm8YYb2qMngT2E`) Blocks in progress: 0 Chunks in progress: 0 Orphans: 0
  2022-07-15T19:35:15.148486Z  INFO stats: #  941069 Downloading headers 3.45% (28843 left; at 942101) 7 peers ⬇ 349 kB/s ⬆ 328 kB/s 0.00 bps 0 gas/s CPU: 52%, Mem: 318 MB
  2022-07-15T19:35:15.148520Z DEBUG stats: EpochId(`Bz9n5HbzLd5Bmsuoc15xbKkJnuuMgNJm8YYb2qMngT2E`) Blocks in progress: 0 Chunks in progress: 0 Orphans: 0
  2022-07-15T19:35:27.476642Z  INFO stats: #  941069 Downloading headers 6.90% (27821 left; at 943131) 8 peers ⬇ 596 kB/s ⬆ 493 kB/s 0.00 bps 0 gas/s CPU: 48%, Mem: 291 MB
  2022-07-15T19:35:27.476676Z DEBUG stats: EpochId(`Bz9n5HbzLd5Bmsuoc15xbKkJnuuMgNJm8YYb2qMngT2E`) Blocks in progress: 0 Chunks in progress: 0 Orphans: 0
  2022-07-15T19:35:37.478760Z  INFO stats: #  941069 Downloading headers 12.09% (26284 left; at 944684) 9 peers ⬇ 780 kB/s ⬆ 551 kB/s 0.00 bps 0 gas/s CPU: 60%, Mem: 320 MB
  2022-07-15T19:35:37.478812Z DEBUG stats: EpochId(`Bz9n5HbzLd5Bmsuoc15xbKkJnuuMgNJm8YYb2qMngT2E`) Blocks in progress: 0 Chunks in progress: 0 Orphans: 0
  ````

Authorize Wallet Locally  
in order to run  transactions via NEAR-CLI, a full access key needs to be installed locally.

run the command :
   ````bash
    ouiouane@Ubuntu-2004-focal-64-minimal:~$ near login
   ````

You will see the following outpu :

 ```bash
  Please help us to collect data on near-cli usage to improve developer experience.
  We will never send private information. We collect which commands are run with attributes, your account ID, and your country
  Note that your account ID and all associated on-chain transactions are already being recorded on public blockchain.

  Would you like to opt in (y/n)? y

  Please authorize NEAR CLI on at least one of your accounts.

  If your browser doesn't automatically open, please visit this URL
  https://wallet.shardnet.near.org/login/?referrer=NEAR+CLI&public_key=ed25519%3AATjazxxxxxxxxxxxxxxxxss_url=http%3A%2F%2F127.0.0.1%3A5000
  Please authorize at least one account at the URL above.

  Which account did you authorize for use with NEAR CLI?
  Enter it here (if not redirected automatically):

  ```
Copy the url and paste it to you browser
Grant Access to Near CLI

![node_02](images/node/node_02.png "node_02")

![node_03](images/node/node_03.png "node_03")



After granting access you will see this page:

![node_04](images/node/node_04.png "node_04")

Enter your account-id and press Enter:

  ````bash
  Which account did you authorize for use with NEAR CLI?
  Enter it here (if not redirected automatically):
  ouiouane-01.shardnet.near
  Logged in as [ ouiouane-01.shardnet.near ] with public key [ ed25519:ATjazw... ] successfully
 ````

Check the validator_key.json
Run the following command :

  ```bash
  ouiouane@Ubuntu-2004-focal-64-minimal:~$ cat ~/.near/validator_key.json
  cat: /home/ouiouane/.near/validator_key.json: No such file or directory
  ```
If the file is not present, generate one like this :

   ```bash
    near generate-key <pool_id> , (pool_id = account-id = ouiouane-01 in our case)
   ```

Copy the file generated to shardnet folder and be sure to replace <pool_id> by your accountId

   ```bash
   ouiouane@Ubuntu-2004-focal-64-minimal:~$ cp ~/.near-credentials/shardnet/ouiouane-01.json ~/.near/validator_key.json
   ```

Edit ~/.near/validator_key.json file and change the following :

- “account_id” : xxxx.factory.shardnet.near, where xxx is the PoolName (ouiouane-01 in our case)
- private_key to secret_key

Start the node
   ```bash
   ouiouane@Ubuntu-2004-focal-64-minimal:~/nearcore$ target/release/neard run
   ```

Install the node Service

   ```bash
    ouiouane@Ubuntu-2004-focal-64-minimal:~/nearcore$ sudo vi /etc/systemd/system/neard.service
   ```

Copy and paste the following :

 ```bash   
   [Unit]
   Description=NEARd Daemon Service
   [Service]
   Type=simple
   User=<USER>
   #Group=near
   WorkingDirectory=/home/<USER>/.near
   ExecStart=/home/<USER>/nearcore/target/release/neard run
   Restart=on-failure
   RestartSec=30
   KillSignal=SIGINT
   TimeoutStopSec=45
   KillMode=mixed
   [Install]
   WantedBy=multi-user.target
  ```

repalce <USER> by a non root user

Enable neard service

  ```bash
  ouiouane@Ubuntu-2004-focal-64-minimal:~/nearcore$ sudo systemctl enable neard
  ```
Start the node :

  ```bash 
  ouiouane@Ubuntu-2004-focal-64-minimal:~/nearcore$ sudo systemctl start neard
  ```

You can see the node logs by runing the following command :

  ```bash
  ouiouane@Ubuntu-2004-focal-64-minimal:~/.near$ journalctl -n 100 -f -u neard
  ```
Which will give the following output:

![node_04](images/node/node_logs_01.png "node_04")


You can install ccze to print pretty logs
  ```bash
  ouiouane@Ubuntu-2004-focal-64-minimal:~/.near$ sudo apt install ccze
  ```
Running the command

  ```bash
   ouiouane@Ubuntu-2004-focal-64-minimal:~/.near$ journalctl -n 100 -f -u neard | ccze -A
  ```
will print the following

![node_04](images/node/node_logs_02.png "node_04")



## Configure a staking pool

### Deploy a Staking Pool Contract
create a new staking pool with the specified name, and deploys it to the indicated accountId by calling the staking pool factory

  ```bash
  near call factory.shardnet.near create_staking_pool '{"staking_pool_id": "<pool id>", "owner_id": "<accountId>", "stake_public_key": "<public key>", "reward_fee_fraction": {"numerator": 5, "denominator": 100}, "code_hash":"DD428g9eqLL8fWUxv8QSpVFzyHi1Qd16P8ephYCTmMSZ"}' --accountId="<accountId>" --amount=30 --gas=300000000000000
  ```
repalace  :

- <pool id> with your account-id monikor 
- <accountId> with your complete account-id, ie : xxxxx.shardnet.near
- <public key> with your node public key (the one in file .near/validator_key.json)

In our case :

  ```bash
  near call factory.shardnet.near create_staking_pool '{"staking_pool_id": "ouiouane-01", "owner_id": "ouiouane-01.shardnet.near", "stake_public_key": "ed25519:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx", "reward_fee_fraction": {"numerator": 3, "denominator": 100}, "code_hash":"DD428g9eqLL8fWUxv8QSpVFzyHi1Qd16P8ephYCTmMSZ"}' --accountId="ouiouane-01.shardnet.near" --amount=30 --gas=300000000000000
  ```

if successful, you will see the following :

![node_04](images/staking/staking_01.png "node_04")



You have  configured your Staking pool. You can see if your node is visible on https://explorer.shardnet.near.org/nodes/validators

Deposit and Stake NEAR

You can stake to your node by running the following command :

 ```bash
   ouiouane@Ubuntu-2004-focal-64-minimal:~/nearcore$ near call ouiouane-01.factory.shardnet.near deposit_and_stake --amount 1395 --accountId ouiouane-01.shardnet.near --gas=300000000000000
 ```
if the staking transaction is successful , you will see the following output  :

![node_04](images/staking/staking_02.png "node_04")

Ping

A ping should be issued each epoch to keep reported rewards current.

to run a ping, use the following command:

   ```bash
   ouiouane@Ubuntu-2004-focal-64-minimal:~/nearcore$ near call ouiouane-01.factory.shardnet.near ping '{}' --accountId ouiouane-01.shardnet.near --gas=300000000000000
   ```

if successful, you will see the following:

![node_04](images/staking/ping_01.png "node_04") 
 


