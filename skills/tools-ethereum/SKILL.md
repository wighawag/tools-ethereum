---
name: tools-ethereum
description: Query and interact with Ethereum and EVM-compatible blockchains using the ecli CLI. Get balances, read blocks and transactions, call smart contract functions, send transactions, encode/decode calldata, and query event logs.
compatibility: Requires Node.js and npm. Network access needed for RPC calls.
---

# tools-ethereum CLI

Use `ecli` to query blockchain data and execute transactions on any EVM network.

## Quick Start

```bash
# Option 1: Install globally
npm install -g tools-ethereum
ecli get_balance --address 0x... --rpc-url $RPC_URL

# Option 2: Use npx (no install required)
npx tools-ethereum get_balance --address 0x... --rpc-url $RPC_URL
```

### Configuration

```bash
# Set RPC endpoint (required for all commands)
export ECLI_RPC_URL=https://eth-mainnet.g.alchemy.com/v2/YOUR_KEY

# For transactions, also set private key
export ECLI_PRIVATE_KEY=0x...

# Query example
ecli get_balance --address 0x742d35Cc6634C0532925a3b844Bc9e7595f5a321
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
ecli get_balance --address 0x... --rpc-url $RPC_URL
# Output: { "address": "0x...", "balance": "1000000000000000000", "balanceInEther": 1.0 }
```

### Get Block Info
```bash
# By number
ecli get_block --blockNumber 18000000 --rpc-url $RPC_URL

# By hash  
ecli get_block --blockHash 0x... --rpc-url $RPC_URL

# Latest block
ecli get_latest_block --rpc-url $RPC_URL

# Include full transaction list
ecli get_block --blockNumber latest --includeTransactions --rpc-url $RPC_URL
```

### Get Transaction Details
```bash
ecli get_transaction --txHash 0x... --rpc-url $RPC_URL
# Returns: hash, from, to, value, gas, gasPrice, blockNumber, nonce, input
```

### Check if Address is Contract
```bash
ecli get_code --address 0x... --rpc-url $RPC_URL
# Returns bytecode. If "0x", it's an EOA (not a contract)
```

### Read Storage Slot
```bash
ecli get_storage_at --address 0x... --slot 0x0 --rpc-url $RPC_URL
```

---

## Calling Smart Contracts

### Read Contract State (No Gas Required)

Use human-readable ABI format for the `--abi` parameter.

```bash
# Call with no arguments
ecli call_contract \
  --address 0x... \
  --abi "function totalSupply() returns (uint256)" \
  --rpc-url $RPC_URL

# Call with arguments (comma-separated)
ecli call_contract \
  --address 0x... \
  --abi "function balanceOf(address) returns (uint256)" \
  --args 0x742d35Cc6634C0532925a3b844Bc9e7595f5a321 \
  --rpc-url $RPC_URL

# Call with JSON array args
ecli call_contract \
  --address 0x... \
  --abi "function allowance(address,address) returns (uint256)" \
  --args '["0xOwner...","0xSpender..."]' \
  --rpc-url $RPC_URL
```

**Common ERC20 calls:**
```bash
# Token name
ecli call_contract --address $TOKEN --abi "function name() returns (string)" --rpc-url $RPC_URL

# Token symbol
ecli call_contract --address $TOKEN --abi "function symbol() returns (string)" --rpc-url $RPC_URL

# Decimals
ecli call_contract --address $TOKEN --abi "function decimals() returns (uint8)" --rpc-url $RPC_URL

# Balance
ecli call_contract --address $TOKEN --abi "function balanceOf(address) returns (uint256)" --args $HOLDER --rpc-url $RPC_URL
```

### Encode/Decode Calldata

```bash
# Encode function call to calldata
ecli encode_calldata \
  --abi "function transfer(address,uint256)" \
  --args "0xRecipient...,1000000000000000000" \
  --rpc-url $RPC_URL
# Output: { "calldata": "0xa9059cbb..." }

# Decode calldata back to function + args
ecli decode_calldata \
  --calldata 0xa9059cbb... \
  --abi "function transfer(address,uint256) returns (bool)" \
  --rpc-url $RPC_URL
```

---

## Querying Event Logs

### Get Contract Events
```bash
ecli get_contract_logs \
  --contractAddress 0x... \
  --fromBlock 18000000 \
  --toBlock 18001000 \
  --eventAbis "event Transfer(address indexed from, address indexed to, uint256 value)" \
  --rpc-url $RPC_URL
```

### Get Logs from Transaction
```bash
ecli get_transaction_logs \
  --txHash 0x... \
  --eventAbis "event Transfer(address indexed from, address indexed to, uint256 value)" \
  --rpc-url $RPC_URL
```

---

## Sending Transactions

**Requires:** `ECLI_PRIVATE_KEY` or `PRIVATE_KEY` environment variable (hex string starting with `0x`).

### Send ETH
```bash
ecli send_transaction \
  --to 0xRecipient... \
  --value "1000000000000000000" \
  --rpc-url $RPC_URL
# value is in wei (1 ETH = 1000000000000000000)
```

### Call Contract Function
```bash
# Using ABI + args
ecli send_transaction \
  --to 0xContract... \
  --abi "function transfer(address,uint256)" \
  --args "0xRecipient...,1000000" \
  --rpc-url $RPC_URL

# Using raw calldata
ecli send_transaction \
  --to 0xContract... \
  --data 0xa9059cbb... \
  --rpc-url $RPC_URL
```

### Gas Configuration
```bash
ecli send_transaction \
  --to 0x... \
  --value "1000000000000000000" \
  --gas "21000" \
  --maxFeePerGas "50000000000" \
  --maxPriorityFeePerGas "2000000000" \
  --rpc-url $RPC_URL
```

### Wait for Confirmation
```bash
# Send and capture hash
result=$(ecli send_transaction --to $TO --value "1000000000000000000" --rpc-url $RPC_URL)
txHash=$(echo $result | jq -r '.transactionHash')

# Wait for confirmation
ecli wait_for_transaction_confirmation --txHash $txHash --rpc-url $RPC_URL
```

---

## Gas Estimation

Estimate before sending to avoid failures.

```bash
# Simple transfer
ecli estimate_gas --to 0x... --value "1000000000000000000" --rpc-url $RPC_URL

# Contract call
ecli estimate_gas \
  --to 0xContract... \
  --abi "function transfer(address,uint256)" \
  --args "0xRecipient...,1000000" \
  --rpc-url $RPC_URL
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
balance=$(ecli call_contract --address $TOKEN --abi "function balanceOf(address) returns (uint256)" --args $SENDER --rpc-url $RPC_URL | jq -r '.result')

# Send transfer
ecli send_transaction \
  --to $TOKEN \
  --abi "function transfer(address,uint256) returns (bool)" \
  --args "$RECIPIENT,$AMOUNT" \
  --rpc-url $RPC_URL
```

### ERC20 Approve + TransferFrom
```bash
# Approve spender
ecli send_transaction \
  --to $TOKEN \
  --abi "function approve(address,uint256) returns (bool)" \
  --args "$SPENDER,$AMOUNT" \
  --rpc-url $RPC_URL
```

### Estimate Then Send
```bash
gas=$(ecli estimate_gas --to $CONTRACT --abi "function mint()" --rpc-url $RPC_URL | jq -r '.gasUsed')
# Add 20% buffer
ecli send_transaction --to $CONTRACT --abi "function mint()" --gas $((gas * 120 / 100)) --rpc-url $RPC_URL
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
ecli get_chain_id --rpc-url https://eth-mainnet.g.alchemy.com/v2/KEY

# Polygon
ecli get_gas_price --rpc-url https://polygon-rpc.com

# Arbitrum
ecli get_block_number --rpc-url https://arb1.arbitrum.io/rpc

# Local (Anvil/Hardhat)
ecli get_balance --address 0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266 --rpc-url http://localhost:8545
```
