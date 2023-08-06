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

__Installing a lighthouse client__:

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

__Running a lighthouse client__:

1. Create a JWT secret file
    - A JWT secret file is used to secure the communication between the execution client and the consensus client
    - create a JWT secret file which will be used in later steps
        - `sudo mkdir -p /secrets`
        - `openssl rand -hex 32 | tr -d "\n" | sudo tee /secrets/jwt.hex`
2. Non-staking
```
lighthouse bn \
  --network mainnet \
  --execution-endpoint http://localhost:8551 \
  --execution-jwt /secrets/jwt.hex \
  --checkpoint-sync-url https://mainnet.checkpoint.sigp.io \
  --disable-deposit-contract-sync
  ```


## go-ethereum setup


## clef setup

## Launching a Node

