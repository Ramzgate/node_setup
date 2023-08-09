# Setting up ETH node on Raspberry-Pi

[Start Here](https://ethereum.org/en/developers/tutorials/run-node-raspberry-pi/)

## Dual Machines
The Raspberry Pi 4B the most advanced Raspberry Pi to date has 8G RAM and performance that falls a bit short of what would required to run a full node, therefore we opt to run the consensus client and the execution clients on separate machines connected via Ethernet 

## Node Structure

|                   |             |     |
|-------------------|-------------|-----|
| Hardware         |   Raspberry Pi 4B |   |
|  OS              |   ubuntu     | C/C++   | 
| consensus client |  lighthouse | Rust |
| Execution client | go-ethereum | Go   |

For instructions on setting up Raspberry Pi machines see [setup](https://github.com/Ramzgate/node_setup/blob/main/raspberry_pi_machine.md)


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

3. __Example__
```
lighthouse bn --network sepolia --execution-endpoint http://localhost:8551 --execution-jwt $JWT_SECRET_PATH --checkpoint-sync-url https://sepolia.checkpoint-sync.ethpandaops.io --disable-deposit-contract-sync --http
```
