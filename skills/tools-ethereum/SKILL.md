---
name: tools-ethereum
description: Query and interact with Ethereum and EVM-compatible blockchains using the ecli CLI. Get balances, read blocks and transactions, call smart contract functions, send transactions, encode/decode calldata, and query event logs.
compatibility: Requires Node.js and npm. Network access needed for RPC calls.
---

# tools-ethereum CLI

Use `ecli` to query blockchain data and execute transactions on any EVM network.

## Quick Start

```bash
# Option 1: Use npx (no install required)
npx tools-ethereum --rpc-url $RPC_URL get_balance --address 0x... 

# Option 2: Install globally
npm install -g tools-ethereum # or use pnpm/volta/...
ecli --rpc-url $RPC_URL get_balance --address 0x...
```

### Configuration

```bash
# Set RPC endpoint (required for all commands)
export ECLI_RPC_URL=https://eth-mainnet.g.alchemy.com/v2/YOUR_KEY

# For transactions, also set private key
export ECLI_PRIVATE_KEY=0x...

# Query example
ecli --rpc-url $RPC_URL get_balance --address 0x742d35Cc6634C0532925a3b844Bc9e7595f5a321
```

All commands output JSON. Parse with `jq` or process programmatically.

---

## Commands Overview

| Command | Purpose |
|---------|---------|
| `get_balance` | ETH balance for address |
| `get_block_number` | Latest block number |
| `get_block` | Block by number or hash |
| `get_latest_block` | Latest block details |
| `get_transaction` | Transaction by hash |
| `get_transaction_count` | Nonce for address |
| `get_gas_price` | Current gas price |
| `get_fee_history` | Historical fee data |
| `get_chain_id` | Network chain ID |
| `get_code` | Contract bytecode |
| `get_storage_at` | Raw storage slot |
| `call_contract` | Read contract (view/pure) |
| `encode_calldata` | Encode function call |
| `decode_calldata` | Decode calldata |
| `get_contract_logs` | Query events |
| `get_transaction_logs` | Logs from tx |
| `estimate_gas` | Estimate gas cost |
| `send_transaction` | Send tx (requires key) |
| `wait_for_transaction_confirmation` | Wait for receipt |
| `sign_message` | Sign message (requires key) |

---

## Reading Blockchain Data

### Get Balance
```bash
ecli --rpc-url $RPC_URL get_balance --address 0x...
# Output: { "address": "0x...", "balance": "1000000000000000000", "balanceInEther": 1.0 }
```

### Get Block Info
```bash
# By number
ecli --rpc-url $RPC_URL get_block --blockNumber 18000000 

# By hash  
ecli --rpc-url $RPC_URL get_block --blockHash 0x... 

# Latest block
ecli --rpc-url $RPC_URL get_latest_block 

# Include full transaction list
ecli --rpc-url $RPC_URL get_block --blockNumber latest --includeTransactions 
```

### Get Transaction Details
```bash
ecli --rpc-url $RPC_URL get_transaction --txHash 0x... 
# Returns: hash, from, to, value, gas, gasPrice, blockNumber, nonce, input
```

### Check if Address is Contract
```bash
ecli --rpc-url $RPC_URL get_code --address 0x... 
# Returns bytecode. If "0x", it's an EOA (not a contract)
```

### Read Storage Slot
```bash
ecli --rpc-url $RPC_URL get_storage_at --address 0x... --slot 0x0 
```

---

## Calling Smart Contracts

### Read Contract State (No Gas Required)

Use human-readable ABI format for the `--abi` parameter.

```bash
# Call with no arguments
ecli --rpc-url $RPC_URL call_contract \
  --address 0x... \
  --abi "function totalSupply() returns (uint256)" \
  

# Call with arguments (comma-separated)
ecli --rpc-url $RPC_URL call_contract \
  --address 0x... \
  --abi "function balanceOf(address) returns (uint256)" \
  --args 0x742d35Cc6634C0532925a3b844Bc9e7595f5a321 \
  

# Call with JSON array args
ecli --rpc-url $RPC_URL call_contract \
  --address 0x... \
  --abi "function allowance(address,address) returns (uint256)" \
  --args '["0xOwner...","0xSpender..."]' \
  
```

**Common ERC20 calls:**
```bash
# Token name
ecli --rpc-url $RPC_URL call_contract --address $TOKEN --abi "function name() returns (string)" 

# Token symbol
ecli --rpc-url $RPC_URL call_contract --address $TOKEN --abi "function symbol() returns (string)" 

# Decimals
ecli --rpc-url $RPC_URL call_contract --address $TOKEN --abi "function decimals() returns (uint8)" 

# Balance
ecli --rpc-url $RPC_URL call_contract --address $TOKEN --abi "function balanceOf(address) returns (uint256)" --args $HOLDER 
```

### Encode/Decode Calldata

```bash
# Encode function call to calldata
ecli --rpc-url $RPC_URL encode_calldata \
  --abi "function transfer(address,uint256)" \
  --args "0xRecipient...,1000000000000000000" \
  
# Output: { "calldata": "0xa9059cbb..." }

# Decode calldata back to function + args
ecli --rpc-url $RPC_URL decode_calldata \
  --calldata 0xa9059cbb... \
  --abi "function transfer(address,uint256) returns (bool)" \
  
```

---

## Querying Event Logs

### Get Contract Events
```bash
ecli --rpc-url $RPC_URL get_contract_logs \
  --contractAddress 0x... \
  --fromBlock 18000000 \
  --toBlock 18001000 \
  --eventAbis "event Transfer(address indexed from, address indexed to, uint256 value)" \
  
```

### Get Logs from Transaction
```bash
ecli --rpc-url $RPC_URL get_transaction_logs \
  --txHash 0x... \
  --eventAbis "event Transfer(address indexed from, address indexed to, uint256 value)" \
  
```

---

## Sending Transactions

**Requires:** `ECLI_PRIVATE_KEY` or `PRIVATE_KEY` environment variable (hex string starting with `0x`).

### Send ETH
```bash
ecli --rpc-url $RPC_URL send_transaction \
  --to 0xRecipient... \
  --value "1000000000000000000" \
  
# value is in wei (1 ETH = 1000000000000000000)
```

### Call Contract Function
```bash
# Using ABI + args
ecli --rpc-url $RPC_URL send_transaction \
  --to 0xContract... \
  --abi "function transfer(address,uint256)" \
  --args "0xRecipient...,1000000" \
  

# Using raw calldata
ecli --rpc-url $RPC_URL send_transaction \
  --to 0xContract... \
  --data 0xa9059cbb... \
  
```

### Gas Configuration
```bash
ecli --rpc-url $RPC_URL send_transaction \
  --to 0x... \
  --value "1000000000000000000" \
  --gas "21000" \
  --maxFeePerGas "50000000000" \
  --maxPriorityFeePerGas "2000000000" \
  
```

### Wait for Confirmation
```bash
# Send and capture hash
result=$(ecli --rpc-url $RPC_URL send_transaction --to $TO --value "1000000000000000000" )
txHash=$(echo $result | jq -r '.transactionHash')

# Wait for confirmation
ecli --rpc-url $RPC_URL wait_for_transaction_confirmation --txHash $txHash 
```

---

## Gas Estimation

Estimate before sending to avoid failures.

```bash
# Simple transfer
ecli --rpc-url $RPC_URL estimate_gas --to 0x... --value "1000000000000000000" 

# Contract call
ecli --rpc-url $RPC_URL estimate_gas \
  --to 0xContract... \
  --abi "function transfer(address,uint256)" \
  --args "0xRecipient...,1000000" \
  
# Output: { "gasUsed": "65000", "gasEstimateInGwei": 0.000065 }
```

---

## ABI Format Reference

The CLI uses human-readable Solidity ABI format:

```
function name() returns (uint256)
function balanceOf(address owner) view returns (uint256)
function transfer(address to, uint256 amount) returns (bool)
function approve(address spender, uint256 value) returns (bool)
event Transfer(address indexed from, address indexed to, uint256 value)
event Approval(address indexed owner, address indexed spender, uint256 value)
```

**Types:** `address`, `uint256`, `int256`, `bool`, `bytes`, `bytes32`, `string`, arrays (`address[]`)

---

## Common Patterns

### ERC20 Token Transfer
```bash
# Check balance first
balance=$(ecli --rpc-url $RPC_URL call_contract --address $TOKEN --abi "function balanceOf(address) returns (uint256)" --args $SENDER  | jq -r '.result')

# Send transfer
ecli --rpc-url $RPC_URL send_transaction \
  --to $TOKEN \
  --abi "function transfer(address,uint256) returns (bool)" \
  --args "$RECIPIENT,$AMOUNT" \
  
```

### ERC20 Approve + TransferFrom
```bash
# Approve spender
ecli --rpc-url $RPC_URL send_transaction \
  --to $TOKEN \
  --abi "function approve(address,uint256) returns (bool)" \
  --args "$SPENDER,$AMOUNT" \
  
```

### Estimate Then Send
```bash
gas=$(ecli --rpc-url $RPC_URL estimate_gas --to $CONTRACT --abi "function mint()"  | jq -r '.gasUsed')
# Add 20% buffer
ecli --rpc-url $RPC_URL send_transaction --to $CONTRACT --abi "function mint()" --gas $((gas * 120 / 100)) 
```

---

## Environment Variables

| Variable | Description |
|----------|-------------|
| `ECLI_RPC_URL` | Primary RPC endpoint |
| `RPC_URL` | Fallback RPC endpoint |
| `ECLI_PRIVATE_KEY` | Primary private key (0x-prefixed) |
| `PRIVATE_KEY` | Fallback private key |

The `--rpc-url` flag overrides environment variables.

---

## Error Handling

All commands return JSON. Errors include an `error` field:

```json
{ "error": "Transaction not found", "stack": "..." }
```

Exit codes: `0` = success, `1` = error.

---

## Multi-Network Usage

```bash
# Ethereum Mainnet
ecli --rpc-url $RPC_URL get_chain_id --rpc-url https://eth-mainnet.g.alchemy.com/v2/KEY

# Polygon
ecli --rpc-url $RPC_URL get_gas_price --rpc-url https://polygon-rpc.com

# Arbitrum
ecli --rpc-url $RPC_URL get_block_number --rpc-url https://arb1.arbitrum.io/rpc

# Local (Anvil/Hardhat)
ecli --rpc-url $RPC_URL get_balance --address 0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266 --rpc-url http://localhost:8545
```
