# Setting up ETH node on mini-PC


## Node Structure

|                   |             |     |
|-------------------|-------------|-----|
| Hardware         |   Beelink SEi mini-PC | intel i5  |
|  OS              |   ubuntu     | C/C++   | 
| consensus client |  lighthouse | Rust |
| Execution client | go-ethereum | Go   |

For instructions on setting up a miniPC machine see [setup](https://github.com/Ramzgate/node_setup/blob/main/miniPC_machine.md)

---------------------------

## clef setup

1. __Data directory setup__
    - Create a data directory `mkdir sepolia-data` (_sepolia-data_, _mainnet_data_, or just _data_)
    - Go to directory `cd sepolia-data`
    - Create subdirectories for different keys:
        - `mkdir clef`
        - `mkdir keystore`
        - `mkdir jwt_secret`
    - Create file _sepolia-env-setup.sh_ with: 
        ```
        export DATA_PATH=<path to .../sepolia-data>
        export CLEF_PATH=$DATA_PATH/clef
        export KEYSTORE_PATH=$DATA_PATH/keystore
        export JWT_PATH=$DATA_PATH/jwt_secret

        export CLEF_FILE=$CLEF_PATH/clef.ipc
        export JWT_SECRET=$JWT_PATH/jwt.hex

        echo "DATA_PATH="$DATA_PATH
        echo "CLEF_FILE="$CLEF_FILE
        echo "KEYSTORE_PATH="$KEYSTORE_PATH
        echo "JWT_SECRET="$JWT_SECRET
        ```
        
    - `source sepolia-env-setup.sh`

2. __Create a JWT secret file__
    - A JWT secret file is used to secure the communication between the execution client and the consensus client
    - create a JWT secret file which will be used in later steps
        - `openssl rand -hex 32 | tr -d "\n" | sudo tee $JWT_PATH/jwt.hex`

3. __Initialize clef__
    - `go-ethereum/build/bin/clef init`
    - copy `masterseed.json` to `...clef/`
        - `cp <path and file given on init> .../clef/.`
4. __New account__
    - `go-ethereum/build/bin/clef newaccount --keystore $KEYSTORE_PATH`

5. __Launch clef in proper chain__
```
go-ethereum/build/bin/clef --keystore $KEYSTORE_PATH --configdir $CLEF_PATH --chainid <chain id>
```
 - _chainid_ flag:
     - _mainet_ - 1
    - _sepolia_ - 11155111

## lighthouse setup

### Installing a lighthouse client

1. Instructions [source](https://lighthouse-book.sigmaprime.io/installation-source.html)
2. Install Rust using rustup：
    - `curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh`
        - During installation, when prompted, enter 1 for the default installation
        - After Rust installation run `cargo version` 
            - If it cannot be found, run `source $HOME/.cargo/env` and `cargo version` should return the version
            - append `source $HOME/.cargo/env to ~/.bashrc`
3. Install dependencies 
    - `sudo apt install -y git gcc g++ make cmake pkg-config llvm-dev libclang-dev clang`
4. Download source:
    - `git clone https://github.com/sigp/lighthouse.git`
5. Build from source:
    - `cd lighthouse`
    - `git checkout stable`
    - `make`

### Running a lighthouse client

2. __Launch a beacon node using Lighthouse__

    Use the following command to start a non-staking beacon nod:

    ```
    lighthouse bn \
    --network mainnet \
    --execution-endpoint http://localhost:8551 \
    --execution-jwt /secrets/jwt.hex \
    --checkpoint-sync-url https://mainnet.checkpoint.sigp.io \
    --disable-deposit-contract-sync
    ```

3. __Notable flags__:
    - `--network`: selects network:
        - `lighthouse` (no flag): Mainnet
        - `lighthouse --network mainnet`: Mainnet
        - `lighthouse --network goerli`: Goerli (testnet)
        - `lighthouse --network sepolia`: Sepolia (testnet)
        - `lighthouse --network gnosis`: Gnosis chain

            >_Note: Using the correct --network flag is very important; using the wrong flag can result in penalties, slashings or lost deposits. As a rule of thumb, always provide a --network flag instead of relying on the default_

    - `--execution-endpoint`: the URL of the execution engine API
        - If the execution engine is running on the same computer with the default port, this will be `http://localhost:8551`
    - `--execution-jwt`: the path to the JWT secret file shared by Lighthouse and the execution engine. This is a mandatory form of authentication which ensures that Lighthouse has the authority to control the execution engine
    - `--checkpoint-sync-url`: Lighthouse supports fast sync from a recent finalized checkpoint. Checkpoint sync is optional; however, we highly recommend it since it is substantially faster than syncing from genesis while still providing the same functionality. The checkpoint sync is done using public endpoints provided by the Ethereum community. For example, in the above command, we use the URL for Sigma Prime's checkpoint sync server for mainnet `https://mainnet.checkpoint.sigp.io`
Additional checkpoint sync servers can be found at `https://eth-clients.github.io/checkpoint-sync-endpoints/`
    - `--http`: to expose an HTTP server of the beacon chain. The default listening address is `http://localhost:5052`
        - The HTTP API is required for the beacon node to accept connections from the validator client, which manages keys

4. __Example__
```
lighthouse bn --network sepolia --execution-endpoint http://localhost:8551 --execution-jwt $JWT_SECRET --checkpoint-sync-url https://sepolia.checkpoint-sync.ethpandaops.io --disable-deposit-contract-sync --http
```

## go-ethereum setup

- Download source code from [go-ethereum](https://geth.ethereum.org/downloads)
- Instructions below are based on [this](https://geth.ethereum.org/docs/getting-started/installing-geth) webpage

1. Build from source code (Linux and Mac) 
    - The go-ethereum repository should be cloned locally. Then, the command make geth configures everything for a temporary build and cleans up afterwards. This method of building only works on UNIX-like operating systems, and a Go installation is still required.
        - `git clone https://github.com/ethereum/go-ethereum.git`
        - `cd go-ethereum`
        - `make geth`
        - `make all` - generates `geth`, `clef`, `devp2p`, `abidump`, `abigen`, `bootnode`,\
         `ethkey`, `evm`, `faucet`, `p2psim`, `rlpdump`
    - These commands create a Geth executable file in `go-ethereum/build/bin` folder that can be moved and run from another directory if required
        >> The binary is standalone and doesn't require any additional files

2. To update an existing Geth installation simply stop the node, navigate to the project root directory and pull the latest version from the Geth GitHub repository. then rebuild and restart the node
    - `cd go-ethereum`
    - `git pull`
    - `make geth`



- `echo '{"id": 1, "jsonrpc": "2.0", "method": "account_list"}' | nc -U ~/.clef/clef.ipc`

- `/Users/eyal42/Work/Entrepreneurship/Ramzgate/Technical/Node/go-ethereum/sepolia-data/jwt.hex`
- `export JWT_SECRET_PATH=/Users/eyal42/Work/Entrepreneurship/Ramzgate/Technical/Node/go-ethereum/sepolia-data/jwt.hex`

- `openssl rand -hex 32`
- `openssl rand -hex 32`

```
build/bin/geth --sepolia --datadir sepolia-data --authrpc.addr localhost --authrpc.port 8551 --authrpc.vhosts localhost --authrpc.jwtsecret $JWT_SECRET --http --http.api eth,net,txpool --signer=$CLEF_FILE --http --http.port 8545
```

## Launching a Node

