# Tagion SDK Core


This is back-end of the Tagion SDK. There are command-line utilities and the devnet program.

To quickly test the money transfer process on the devnet you just need to follow two guides:

1. [Run the Devnet Container](#run-the-devnet-container)
2. [Bootstrap Configuration](#bootstrap-configuration)

## Table of Contents
- [Tagion SDK Core](#tagion-sdk-core)
  - [Table of Contents](#table-of-contents)
  - [Directory Overview](#directory-overview)
- [Run the Devnet Container](#run-the-devnet-container)
    - [Step 1. Clone SDK Repository](#step-1-clone-sdk-repository)
    - [Step 2. Build the Docker Image](#step-2-build-the-docker-image)
    - [Step 3. Run the Docker Image](#step-3-run-the-docker-image)
    - [Step 4. Try CLI Tools](#step-4-try-cli-tools)
- [Bootstrap Configuration](#bootstrap-configuration)
- [Manual Devnet Configuration](#manual-devnet-configuration)
  - [Create Two Wallets](#create-two-wallets)
  - [Prepare the Devnet](#prepare-the-devnet)
  - [Start the Devnet](#start-the-devnet)
  - [Send Tagions Between Wallets](#send-tagions-between-wallets)



## Directory Overview

``` bash
./devnet/ # CLI tool that starts Tagion Dev Network on local machine
./utils/
    ./dart/ # CLI tool to inspect DART database, and write to it directly.
    ./hibon/ # CLI tool to convert between `.hibon` and `.json` formats.
    ./tagion/ # CLI tool to create Tagion bills for devnet.
./wallet/ # CLI tool to send and receive Tagion bills
```

# Run the Devnet Container

If you don't have Docker installed, please follow [official installation guide](https://docs.docker.com/get-started/).

### Step 1. Clone SDK Repository

``` bash
git clone https://github.com/tagion/sdk.git ./tagion-sdk
cd ./tagion-sdk
```

### Step 2. Build the Docker Image

``` bash
cd ./core
docker build -t tagion-sdk-core .
```

### Step 3. Run the Docker Image

``` bash
docker run -it tagion-sdk-core

# Or mount your project folder to container workspace,
# to keep the state of the container on your machine
cd ./my-project
docker run -it -v $PWD/:/workspace tagion-sdk-core
```

### Step 4. Try CLI Tools

Now you are in the devnet container. You can use all the utils, like that:

``` bash
hibonutil --help
tagionutil --help
```

You can run the devnet from here, and use the Wallet CLI.

# Bootstrap Configuration

We prepared a quick configuration for you, so you can try wallet CLI, sending some money back and forth, without going through the whole process of preparing the devnet.

In the bootstrap setup we have 4 wallets in the directories from `w1` to `w4`, each one has the pin `1111`. The `w1` has some money on it, the other 3 do not. The directory `node0` has initial genesis DART database, with the Tagion bills (money) for the `w1`.

In this simple guide, we will transfer money from `w1` to `w2` and check the balances of each wallet.

**Let's begin!**

We will need two terminal windows for this guide - one for the devnet and the other for interacting with Wallet CLI. 

**In the first terminal**, run the Docker container that you have built in the previous step:

``` bash
# We are not mounting the volume this time,
# and we are naming the container for later use
docker run -it --name tagion tagion-sdk-core 
```

Now, use `bootstrap` script to get all the needed files in the `/workspace` directory and start the devnet.

``` bash
bootstrap # Will copy all the needed files
devnet # Will start the devnet
```

Next, open another terminal session and ssh into the same container, where the devnet is running. Leave the devnet running in the first terminal.

**In the second terminal:**

``` bash
# We can use 'tagion' as a name, because 
# we named the container in the first step:
docker exec -it tagion bash 
```

Use this terminal to interact with Wallet CLI going further.

--- 

First, let's check the balance of the `w1`, it should have 50000 TGN.

``` bash
cd ./w1
tagionwallet -g # Opening GUI to see the balance
```

Now you are ready to send some money around.

Use `demo_send` script to send money from one waller to another:

``` bash
demo_send w1 w2 500 # We are sending 500 TGN from w1 to w2
```

You should get output like this:

```
0] b.owner        025a87708536aa5bec75d635bf162a6334e416f14334c251fb045b2525b6c5e3d6
0] account        025a87708536aa5bec75d635bf162a6334e416f14334c251fb045b2525b6c5e3d6
signed  true pkey=025a87708536aa5bec75d635bf162a6334e416f14334c251fb045b2525b6c5e3d6
500 TGN sent from w1 to w2
```

If you look closely at the logs of the devnet that is running in another terminal session, you should notice the logs about this money transfer. 

Under the hood, `w1` just generated a contract and sent it to one of the devnet nodes. The node then gossiped the transaction to other nodes and eventually, every node got the same graph of events, without any synchronization, thanks to Hashgrpah algorithm. Once the epoch in the Hashgraph was completed, each node updated its local DART file, creating a new Tagion bill, with public key, managed by `w2`. Now only `w2` can spend that new Tagion bill.

Wait for at least 30 seconds and check the balance of `w2`:

``` bash
cd ./w2
tagionwallet --update # Sync wallet with the database of devnet
tagionwallet -g # Enter the GUI to see the balance  
```

If you don't see the balance changed, wait for another 20-30 seconds and repeat the process. You can do the same with `w1` to see the amount changed.

**Congratulations!** You just made the money transfer on the devnet with very early versions of dev tools. If you'd like to get your hands even more dirty, follow the manual devnet configuration guide below.

# Manual Devnet Configuration

If you want to go deep and configure the devnet manually, follow this guide. We will create two wallets, generate genesis file that puts some money on one of the wallets and will start the devnet. After that, you can use `tagionwallet` directly to create invoices and fulfill them via another wallet.

## Create Two Wallets

Tagion does not have the account system, like most blockchains do. Instead, we store the bills in the database. The wallet software manages the keys for those bills, so you can send them.

The `tagionwallet` util has a developer-friendly interface. You can create multiple accounts and send money from them.

Currently, in order to send money from one wallet to another, the sender must know the public key of the receiver. We use **invoices** for it. The recevier must generate the invoice, that will be fulfilled by the sender.

**Let's create some Wallets!**

First, make sure you are in the `tagion-sdk-core` container:

``` bash
tagionwallet --help # Should output the help
```

Then we create two folders, one for each wallet. Let's name them `w1` and `w2` accordingly.

``` bash
mkdir w1 w2
```

Now we will go in the `tagionwallet` ~~kinda~~ GUI mode to create the wallets. We need to repeat the process two times for each wallet.

``` bash
tagionwallet -g # Will enter the GUI mode
```

Press `c` to enter interactive mode and create the wallet. You will be asked to answer at least 3 security questions and enter a PIN code afterwards. With this security questions you can restore your wallet, in case you forgot the PIN code.

Finally, your directory should look like this:

```
./w1/
    ./tagionwallet.hibon
./w2/
    ./tagionwallet.hibon
```

## Prepare the Devnet

To start the network we need to have the `.drt` file, this is the database file, that will be synced between the nodes and modified when the netwrok runs.

By default, there is no money in the network, so we need to create a **genesis invoice** and generate a `.drt` file from it, so the wallets we created previously has some money on them.

```bash
cd ./w1
# Enter GUI mode and generate an invoice for any sum
tagionwallet -g 
# Enter Pincode
# Press 'a' to enter account
# Press 'i' to enter invoice mode
# Type invoice label, press enter and fill the sum of money to generate
# Quit the GUI by pressing ctrl+c
ls # Now you should see added invoice.hibon
# You see the contents of this file by typing
hibonutil ./invoice.hibon -p
```

Now we need to convert the invoice to the genesis file:

``` bash
cd ../ # Exit the w1 folder
pwd # Should output '/workspace'
tagionutil -i ./w1/invoice.hibon ./genesis.hibon
```

Now we will generate a dart database based on the genesis file:

``` bash
dartutil --initialize --dartfilename dart.drt -i genesis.hibon --modify
```

Then, we create a folder for the first node, and move the generated dart file there:

``` bash
mkdir node0
cp dart.drt ./node0
```

We need to manually move the `genesis.hibon` to the first wallet, so it knows its bills.

``` bash
mv genesis.hibon w1/bills.hibon
```

Finally, your directory should look like this:

```
./genesis.hibon
./dart.drt

./node0/
    ./dart.drt 
./w1/
    ./tagionwallet.hibon
    ./bills.hibon
./w2/
    ./tagionwallet.hibon
```

Now everything is ready to start a devnet.

## Start the Devnet

Now, we have the `.drt` database file, and we can start `tagionwave` that will spawn multiple nodes and sync the database between them. Once the devnet is started, we can send money between the wallets.

The devnet will create a folder for each node, and keep the `.drt` database file in each of them. But we need to create the first folder, so at least one node does have the database.

Open a separate console and ssh into the container we created:
```bash
docker container ls # Will show you the list of continaer

# Replace <CONTAINER_HASH> with the hash you
# get from previous command (first colmn)
docker exec -it <CONTAINER_HASH> bash # Will move you to the container
```

``` bash
pwd # Let's make sure we are in the correct folder
# Output: /workspace/

ls # At this point we should have the .drt file generated
# Output: 

mkdir node0 # Creating a directory for the first node
cp 
```

## Send Tagions Between Wallets

The network is running, good job making all this way! Now let's what we came here to do - move millions of tagions around.

Now you have two wallets, and 1 million tagions on one of them. Let's send 500k to the other wallet.



You can see the contents of the `hibon` files using `hibonutil` :

``` bash
# Will convert to JSON and output formatted in the console
hibonutil genesis.hibon -p 
```
  

Now, we need to create invoice from the second wallet (it will be fulfilled by the first wallet).

``` bash
cd w2
tagionwallet -g # Enter GUI to generate invoice
```

You need to enter the pincode of the second wallet, in this example we used **2222**, and press `i` to generate invoice. Enter invoice name, amount and then quit the `tagionwallet` by pressing `ctrl + c` 

Now let's pay the invoice:

``` bash
cd w1
tagionwallet --pay ../w2/invoice.hibon --pin 1111 -s
```

This command will create the executable contract, sign it and send to the devnet that should be running in your container by now. The devnet node will gossip the transaction to the rest of the nodes and in a few seconds, after consensus is reached, the database will be modified.

Let's check the ballances of both wallets now.

