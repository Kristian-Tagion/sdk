# Tagion SDK Core

This is back-end of the Tagion SDK. We recommend interacting with CLI tools via Docker.

## Directory Overview

``` bash
./devnet/ # CLI tool that simulates Tagion Network on local machine
./utils/
    ./dart/ # CLI tool to inspect DART database, and write to it directly.
    ./hibon/ # CLI tool to convert between `.hibon` and `.json` formats.
    ./tagion/ # CLI tool to create Tagion bills for devnet.
./wallet/ # CLI tool to send and receive Tagion bills
```

## Running Devnet Container

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

# Or, if you need, mount current volume to the container:
docker run -it -v $PWD/:/tagion tagion-sdk-core

# Or any other volume via absolute path to it:
docker run -it -v /any-other-volume/:/some-container-voume tagion-sdk-core
```

### Step 4. Use CLI Tools

Now you are in the Devnet container. You can use all the utils, like that:

```bash
hibonutil --help
tagionutil --help
```

You can run the Devnet from here, and use the Wallet CLI.

## Running the Devnet

First, we need to print some tagions.

[This section is in development]

## Using Wallet CLI

To send and receive money in the network, we can use Wallet CLI.

[This section is in development]
