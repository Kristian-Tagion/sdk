# Tagion SDK Core

> **The docs are in development, please don't follow the guide just yet.**

This is back-end of the Tagion SDK. We recommend interacting with CLI tools via Docker.

## Directory Overview

``` bash
./devnet/ # CLI tool that starts Tagion Dev Network on local machine
./utils/
    ./dart/ # CLI tool to inspect DART database, and write to it directly.
    ./hibon/ # CLI tool to convert between `.hibon` and `.json` formats.
    ./tagion/ # CLI tool to create Tagion bills for devnet.
./wallet/ # CLI tool to send and receive Tagion bills
```

# FAQ [NEED POLISH]

What is Invoice?

What is Devnet?
It's a real node software running in the developer mode, spawning multiple nodes automatically.

How the nodes are communicating?
Locally, via websockets connecting to ports.

# Guide [NEED POLISH]

In this guide we will use CLI utilities to create two wallet instances, put some initial money to one of them, send some of those money to the other wallet, by sending the contract to the devnet thus modifying the database.

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

# Or mount your project folder to container workspace:
cd ./my-project
docker run -it -v $PWD/:/workspace tagion-sdk-core
```

### Step 4. Use CLI Tools

Now you are in the Devnet container. You can use all the utils, like that:

``` bash
hibonutil --help
tagionutil --help
```

You can run the Devnet from here, and use the Wallet CLI.

# Create Two Wallets

Tagion does not have the account system, like most blockchains do. Instead, we store the bills in the database. The wallet software manages the keys for those bills, so you can send them.

The `tagionwallet` util has a developer-friendly interface. You can create multiple accounts and send money from them.

Currently, in order to send money from one wallet to another, the sender must know the public key of the receiver. We use **invoices** for it. The recevier must generate the invoice, that will be fulfilled by the sender.

## Let's Create the Wallets

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

# Prepare the Devnet

To start the network we need to have the `.drt` file, this is the database file, that will be synced between the nodes and modified when the netwrok runs.

By default, there is no money in the network, so we need to create a **genesis invoice** and generate a `.drt` file from it, so the wallets we created previously has some money on them.

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

# Start the Devnet

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

# Send Tagions Between Wallets

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

