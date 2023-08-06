# Setting up ETH node on mini-PC


## Node Structure

|                   |             |     |
|-------------------|-------------|-----|
| Hardware         |   Beelink SEi mini-PC | intel i5  |
|  OS              |   ubuntu     | C/C++   | 
| consensus client |  lighthouse | Rust |
| Execution client | go-ethereum | Go   |

For instructions on seting up a miniPC machine see [setup](https://github.com/Ramzgate/node_setup/blob/main/miniPC_machine.md)


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

1. __Create a JWT secret file__
    - A JWT secret file is used to secure the communication between the execution client and the consensus client
    - create a JWT secret file which will be used in later steps
        - `sudo mkdir -p /secrets`
        - `openssl rand -hex 32 | tr -d "\n" | sudo tee /secrets/jwt.hex`

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
    - `--http`: to expose an HTTP server of the beacon chain. The default listening address is `http://localhost:5052`
        - The HTTP API is required for the beacon node to accept connections from the validator client, which manages keys

## go-ethereum setup


## clef setup

## Launching a Node

