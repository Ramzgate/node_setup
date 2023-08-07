# Setting up ETH node on Raspberry-Pi

[Start Here](https://ethereum.org/en/developers/tutorials/run-node-raspberry-pi/)

## Dual Machines
The Raspverry Pi 4B the most advanced Raspberry Pi to date has 8G RAM and performance that falls a bit short of what would required to run a full node, therefore we opt to run the consensus client and the execution clients on seperate machines connected via Ethernet 

## Node Structure

|                   |             |     |
|-------------------|-------------|-----|
| Hardware         |   Raspberry Pi 4B |   |
|  OS              |   ubuntu     | C/C++   | 
| consensus client |  lighthouse | Rust |
| Execution client | go-ethereum | Go   |

For instructions on seting up Raspberry Pi machines see [setup](https://github.com/Ramzgate/node_setup/blob/main/raspberry_pi_machine.md)


---------------------------

## Machine 1 - lighthouse

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
    - `--http`: to expose an HTTP server of the beacon chain. The default listening address is `http://localhost:5052`
        - The HTTP API is required for the beacon node to accept connections from the validator client, which manages keys

4. __Example__
```
lighthouse bn --network sepolia --execution-endpoint http://localhost:8551 --execution-jwt $JWT_SECRET_PATH --checkpoint-sync-url https://sepolia.checkpoint-sync.ethpandaops.io --disable-deposit-contract-sync --http
```
