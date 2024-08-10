
# Token Swap and Aave Deposit Script

## Overview

This script automates the process of swapping tokens and supplying them to the Aave protocol. It interacts with several DeFi protocols on the Ethereum network, including a token factory, a swap router, and the Aave lending pool. The script is designed to:

1. **Approve Tokens:** It first approves a specified amount of a token (USDC in this case) to be swapped using a decentralized exchange.
2. **Swap Tokens:** The script then swaps the approved USDC tokens for another token (LINK) using the Uniswap V3 swap router.
3. **Supply to Aave:** Finally, the swapped LINK tokens are supplied to the Aave lending pool, allowing the user to earn interest on their deposit.

The script is built using `ethers.js`, a popular library for interacting with the Ethereum blockchain, and requires a wallet with sufficient funds and the appropriate RPC URL to connect to the network.

### Workflow

The overall process involves the following steps:

1. **Token Approval:** The script first checks the allowance of the USDC tokens. If not sufficient, it sends a transaction to approve the required amount of USDC to the swap router contract.
2. **Get Pool Info:** The script retrieves the pool information for the USDC/LINK pair, including the contract address of the liquidity pool.
3. **Prepare Swap Parameters:** The script calculates the parameters necessary for the token swap, including the amount of USDC to swap, the minimum amount of LINK to receive, and other settings like the fee tier and price limit.
4. **Execute Swap:** The script sends a transaction to the Uniswap V3 router contract to execute the swap of USDC for LINK.
5. **Supply LINK to Aave:** After successfully swapping the tokens, the script sends another transaction to deposit the swapped LINK tokens into the Aave lending pool.

### Key Contracts

- **Factory Contract:** Handles the creation and management of liquidity pools for token pairs.
- **Swap Router Contract:** Facilitates the swapping of tokens through decentralized exchanges like Uniswap V3.
- **Aave Lending Pool Contract:** Manages deposits and loans in the Aave protocol, enabling users to earn interest on deposited tokens.

### Dependencies

- `ethers.js`
- `dotenv`
- Access to an Ethereum RPC endpoint
- Private key of the wallet

## Diagram Illustration

Below is a textual description of the diagram:

### Step-by-Step Workflow Diagram

1. **User Wallet:**
   - Sends approval transaction to the USDC Token Contract.
   - Signs and sends transactions to the Swap Router and Aave Lending Pool.

2. **USDC Token Contract:**
   - Handles approval for the Swap Router to spend USDC on behalf of the user.

3. **Factory Contract:**
   - Retrieves the pool address for the USDC/LINK pair.

4. **Pool Contract:**
   - Provides token pair information (USDC and LINK) and fee structure.

5. **Swap Router Contract:**
   - Executes the token swap from USDC to LINK.

6. **Aave Lending Pool Contract:**
   - Accepts the LINK deposit and credits the user's account in the Aave protocol.

The diagram could be visualized as a flowchart where arrows indicate the direction of interactions between the user’s wallet and the various smart contracts.

```
+------------------+
|                  |      Approve      +-----------------+
|   User Wallet    +------------------->   USDC Token    |
|                  |                   |    Contract     |
+------------------+                   +-----------------+
        |                                      |
        |                                      |
        v                                      v
+------------------+                   +-----------------+
|                  |                   |                 |
|  Factory Contract|<------------------+    Swap Router |
|                  |  Get Pool Address |                 |
+------------------+                   +-----------------+
        |                                      |
        |                                      |
        v                                      v
+------------------+                   +-----------------+
|                  |                   |                 |
|  Pool Contract   |                   |  Aave Lending   |
|                  |<------------------+     Pool        |
+------------------+  Execute Swap     |                 |
                                       +-----------------+
```

## How to Run the Script

1. **Clone the Repository:**
    ```bash
    git clone https://github.com/Easyisc/project-token-swap.git
    cd <repository-directory>
    ```

2. **Install Dependencies:**
    ```bash
    npm install
    ```

3. **Configure Environment Variables:**
    - Create a `.env` file in the root directory with the following content:
    ```env
    RPC_URL=<your_rpc_url>
    PRIVATE_KEY=<your_private_key>
    ```

4. **Run the Script:**
    ```bash
    node tokenSwap.js
    ```


    
## Code Explanation

The script provided is a comprehensive example of interacting with decentralized finance (DeFi) protocols on the Ethereum blockchain. Specifically, it demonstrates how to approve tokens, swap tokens using Uniswap, and supply tokens to Aave's lending pool. Below is a detailed explanation of each part of the code:

### 1. Dependencies and Setup

```javascript
const ethers = require("ethers");
const FACTORY_ABI = require("./abis/factory.json");
const SWAP_ROUTER_ABI = require("./abis/swaprouter.json");
const POOL_ABI = require("./abis/pool.json");
const TOKEN_IN_ABI = require("./abis/token.json");
const dotenv = require("dotenv");
dotenv.config();
```

- **`ethers.js`**: A library for interacting with the Ethereum blockchain. It's used to manage wallets, send transactions, and interact with smart contracts.
- **ABIs**: Application Binary Interfaces (ABIs) define how to interact with the smart contracts. They are JSON files that describe the functions and events of a contract.
- **`dotenv`**: This package loads environment variables from a `.env` file, allowing you to securely store sensitive information like your RPC URL and private key.

### 2. Contract Addresses

```javascript
const POOL_FACTORY_CONTRACT_ADDRESS = "0x0227628f3F023bb0B980b67D528571c95c6DaC1c";
const SWAP_ROUTER_CONTRACT_ADDRESS = "0x3bFA4769FB09eefC5a80d6E87c3B9C650f7Ae48E";
const AAVE_LENDING_POOL_ADDRESS = "0x7d2768dE32b0b80b7a3454c06BdAc94A69DDc7A9"; // Replace with actual Aave Lending Pool contract address
```

These variables store the addresses of the relevant smart contracts on the Ethereum network:
- **Factory Contract**: Used to get the pool address for the token pair.
- **Swap Router Contract**: Used to perform token swaps.
- **Aave Lending Pool**: Used to supply tokens to Aave.

### 3. Provider and Signer Setup

```javascript
const provider = new ethers.JsonRpcProvider(process.env.RPC_URL);
const factoryContract = new ethers.Contract(POOL_FACTORY_CONTRACT_ADDRESS, FACTORY_ABI, provider);
const signer = new ethers.Wallet(process.env.PRIVATE_KEY, provider);
```

- **`provider`**: Connects to the Ethereum network using the RPC URL specified in the `.env` file.
- **`factoryContract`**: A contract instance to interact with the pool factory.
- **`signer`**: Represents the user's wallet and is used to sign transactions.

### 4. Token Definitions

```javascript
const USDC = {
  chainId: 11155111,
  address: "0x1c7D4B196Cb0C7B01d743Fbc6116a902379C7238",
  decimals: 6,
  symbol: "USDC",
  name: "USD//C",
  isToken: true,
  isNative: true,
  wrapped: false,
};

const LINK = {
  chainId: 11155111,
  address: "0x779877A7B0D9E8603169DdbD7836e478b4624789",
  decimals: 18,
  symbol: "LINK",
  name: "Chainlink",
  isToken: true,
  isNative: true,
  wrapped: false,
};
```

- **USDC and LINK**: These objects define the properties of the tokens involved in the swap and subsequent supply to Aave. Each token has attributes like its address, decimals, and symbol.

### 5. Token Approval Function

```javascript
async function approveToken(tokenAddress, tokenABI, amount, wallet) {
  try {
    const tokenContract = new ethers.Contract(tokenAddress, tokenABI, wallet);
    const approveAmount = ethers.parseUnits(amount.toString(), USDC.decimals);
    const approveTransaction = await tokenContract.approve.populateTransaction(SWAP_ROUTER_CONTRACT_ADDRESS, approveAmount);
    const transactionResponse = await wallet.sendTransaction(approveTransaction);
    console.log(`Approval Transaction Sent: ${transactionResponse.hash}`);
    const receipt = await transactionResponse.wait();
    console.log(`Approval Transaction Confirmed: https://sepolia.etherscan.io/tx/${receipt.transactionHash}`);
  } catch (error) {
    console.error("An error occurred during token approval:", error);
    throw new Error("Token approval failed");
  }
}
```

- **Purpose**: Approves the specified amount of a token (USDC) to be spent by the swap router contract.
- **Logic**:
  - Creates a contract instance for the token using its ABI and the wallet.
  - Sends a transaction to approve the swap router to spend the specified amount of tokens.
  - Waits for the transaction to be mined and logs the transaction hash.

### 6. Get Pool Info Function

```javascript
async function getPoolInfo(factoryContract, tokenIn, tokenOut) {
  const poolAddress = await factoryContract.getPool(tokenIn.address, tokenOut.address, 3000);
  if (!poolAddress) {
    throw new Error("Failed to get pool address");
  }
  const poolContract = new ethers.Contract(poolAddress, POOL_ABI, provider);
  const [token0, token1, fee] = await Promise.all([
    poolContract.token0(),
    poolContract.token1(),
    poolContract.fee(),
  ]);
  return { poolContract, token0, token1, fee };
}
```

- **Purpose**: Retrieves the liquidity pool address and details for the USDC/LINK pair from the factory contract.
- **Logic**:
  - Queries the factory contract for the pool address.
  - If the pool address is valid, it creates a contract instance for the pool.
  - Retrieves information about the tokens in the pool and the fee structure.

### 7. Prepare Swap Parameters Function

```javascript
async function prepareSwapParams(poolContract, signer, amountIn) {
  return {
    tokenIn: USDC.address,
    tokenOut: LINK.address,
    fee: await poolContract.fee(),
    recipient: signer.address,
    amountIn: amountIn,
    amountOutMinimum: 0,
    sqrtPriceLimitX96: 0,
  };
}
```

- **Purpose**: Prepares the parameters required for executing the token swap.
- **Logic**:
  - Specifies the input token (USDC) and the output token (LINK).
  - Retrieves the pool fee from the pool contract.
  - Sets the recipient to the signer’s address.
  - Specifies the amount of USDC to swap and other swap settings.

### 8. Execute Swap Function

```javascript
async function executeSwap(swapRouter, params, signer) {
  const transaction = await swapRouter.exactInputSingle.populateTransaction(params);
  const receipt = await signer.sendTransaction(transaction);
  console.log(`Swap Transaction Confirmed: https://sepolia.etherscan.io/tx/${receipt.hash}`);
}
```

- **Purpose**: Executes the token swap from USDC to LINK using the swap router contract.
- **Logic**:
  - Calls the `exactInputSingle` function on the swap router with the prepared parameters.
  - Sends the transaction and waits for it to be mined.
  - Logs the transaction hash.

### 9. Supply to Aave Function

```javascript
async function supplyToAave(tokenAddress, amount, signer) {
  try {
    // Step 1: Approve Aave to spend your tokens
    const tokenContract = new ethers.Contract(tokenAddress, TOKEN_IN_ABI, signer);
    const approveTransaction = await tokenContract.approve.populateTransaction(AAVE_LENDING_POOL_ADDRESS, amount);
    const approvalResponse = await signer.sendTransaction(approveTransaction);
    await approvalResponse.wait();

    // Step 2: Supply tokens to Aave Lending Pool
    const aaveLendingPool = new ethers.Contract(AAVE_LENDING_POOL_ADDRESS, AAVE_LENDING_POOL_ABI, signer);
    const supplyTransaction = await aaveLendingPool.deposit(tokenAddress, amount, signer.address, 0);
    const supplyResponse = await supplyTransaction.wait();

    console.log(`Successfully supplied ${ethers.formatUnits(amount, LINK.decimals)} LINK to Aave.`);
    console.log(`Transaction Hash: https://sepolia.etherscan.io/tx/${supplyResponse.transactionHash}`);
  } catch (error) {
    console.error("An error occurred while supplying tokens to Aave:", error.message);
    throw new Error("Aave supply failed");
  }
}
```

- **Purpose**: Supplies the swapped LINK tokens to the Aave lending pool.
- **Logic**:
  - **Approval**: Approves the Aave lending pool contract to spend the user's LINK tokens.
  - **Deposit**: Calls the `deposit` function on the Aave lending pool contract to supply the LINK tokens.
  - Logs the transaction hash of the deposit.

### 10. Main Function

```javascript
async function main(swapAmount) {
  const inputAmount = swapAmount;
  const amountIn = ethers.parseUnits(inputAmount.toString(), USDC.decimals);

  try {
    await approveToken(USDC.address, TOKEN_IN_ABI, inputAmount, signer);
    const { poolContract } = await getPoolInfo(factoryContract, USDC, LINK);
    const params = await prepareSwapParams(poolContract, signer, amountIn);
    const swapRouter = new ethers.Contract(SWAP_ROUTER_CONTRACT_ADDRESS, SWAP_ROUT

ER_ABI, signer);
    await executeSwap(swapRouter, params, signer);
    const linkBalance = await signer.getBalance(LINK.address);
    await supplyToAave(LINK.address, linkBalance, signer);
  } catch (error) {
    console.error("An error occurred:", error);
  }
}

const swapAmount = "0.01";
main(swapAmount).catch((error) => {
  console.error("Main function error:", error);
});
```

- **Purpose**: The `main` function orchestrates the entire process from token approval to the final supply of tokens to Aave.
- **Logic**:
  - Calls the `approveToken` function to approve the USDC tokens.
  - Retrieves the pool information for the USDC/LINK pair.
  - Prepares the swap parameters and executes the swap.
  - Finally, supplies the LINK tokens to Aave.
  - Logs any errors that occur during the process.

## 11. Running the Script

To execute the script, simply run the following command in your terminal:

```bash
node tokenSwap.js
```

This will initiate the entire process of token approval, swapping, and supplying to Aave.
