# Dynamic and Static Inflation in Cosmos SDK

## Step 1: Install Cosmos SDK
1. Clone the Cosmos SDK repository:
   ```sh
   git clone https://github.com/cosmos/cosmos-sdk
   cd cosmos-sdk
   ```

2. Go to the SimApp directory and install dependencies:
   ```sh
   cd simapp
   make install
   ```



## Step 2: Setting Up a Local Test Application Node

1. If you have previously run simd, reset it:
    ```sh
    simd comet unsafe-reset-all
    ```
   and delete the contents of the ~/.simapp directory:
    ```sh
    rm -rf ~/.simapp/
    ```

2. Initialize a new blockchain:
    ```sh
    simd init inflation --chain-id my-test-inflation-chain
    ```

3. Choose the inflation mechanism:
    - Static Inflation: Make changes to ~/.simapp/config/genesis.json
    - Here we set the inflation rate change to 0%
    ```json
    "mint": {
      "minter": {
        "inflation": "0.010000000000000000",
        "annual_provisions": "0.000000000000000000"
      },
      "params": {
        "mint_denom": "stake",
        "inflation_rate_change": "0.000000000000000000", 
        "inflation_max": "0.010000000000000000",
        "inflation_min": "0.010000000000000000",
        "goal_bonded": "0.670000000000000000",
        "blocks_per_year": "6311520",
        "max_supply": "0"
      }
    }
    ```

    - Dynamic Inflation: Make changes to ~/.simapp/config/genesis.json
    - Here we set the inflation rate change to 1%
    - As I understand it, for a more interesting change in inflation, you can write your own custom mint module, and there, for example, change inflation depending on the number of transactions
   ```json
    "mint": {
      "minter": {
        "inflation": "0.013000000000000000",
        "annual_provisions": "0.000000000000000000"
      },
      "params": {
        "mint_denom": "stake",
        "inflation_rate_change": "0.010000000000000000",
        "inflation_max": "0.020000000000000000",
        "inflation_min": "0.007000000000000000",
        "goal_bonded": "0.670000000000000000",
        "blocks_per_year": "6311520",
        "max_supply": "0"
      }
    }
    ```

4. Add an account and create transactions:
    ```sh
    simd keys add my_validator --keyring-backend test
    MY_VALIDATOR_ADDRESS=$(simd keys show my_validator -a --keyring-backend test)
    simd genesis add-genesis-account $MY_VALIDATOR_ADDRESS 1000000000stake
   
    simd genesis gentx my_validator 500000000stake --chain-id my-test-inflation-chain --keyring-backend test
    simd genesis collect-gentxs
    ```

5. Start the node:
    ```sh
    simd start
    ```

## Step 3: Executing Transactions

1. Open a new terminal tab and create a new account:
    ```sh
    simd keys add recipient --keyring-backend test
    export RECIPIENT=$(simd keys show recipient -a --keyring-backend test)
    ```

2. Verify the recipient and my_validator addresses:
    ```sh 
   echo $RECIPIENT
   echo $MY_VALIDATOR_ADDRESS
    ```
   If they are not set::
   ```sh
    export RECIPIENT=$(simd keys show recipient -a --keyring-backend test)
    export MY_VALIDATOR_ADDRESS=$(simd keys show my_validator -a --keyring-backend test)
   ```

3. Send tokens to the recipient from my_validator:
    ```sh
   simd tx bank send $MY_VALIDATOR_ADDRESS $RECIPIENT 1000000stake --chain-id my-test-inflation-chain --keyring-backend test
    ```

4. Check the balance of the recipient and the sender:
    ```sh
    simd query bank balances $RECIPIENT
    simd query bank balances $MY_VALIDATOR_ADDRESS
    ```

## Step 4: Checking Inflation

1. Check the inflation rate:
    ```sh
   simd query mint inflation
    ```

