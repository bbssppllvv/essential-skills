# Crypto API Documentation

## Overview

OpenRouter enables credit purchases using cryptocurrency through a Coinbase integration. Users can buy credits via the UI on their credits page or programmatically through the API, paying with native chain tokens.

## Credit Purchase Process

The headless credit purchase workflow involves three steps:

1. Retrieve calldata for a new purchase
2. Execute an on-chain transaction with that data
3. Monitor balance and replenish as needed

## Getting Credit Purchase Calldata

Submit a POST request to `/api/v1/credits/coinbase` to initiate a charge. Include:
- Credit amount in USD (up to maximum limit)
- Sender wallet address
- EVM chain ID

**Supported Chains (mainnet only):**
- Ethereum
- Polygon
- Base (recommended)

### Example Request

```typescript
const response = await fetch('https://openrouter.ai/api/v1/credits/coinbase', {
  method: 'POST',
  headers: {
    Authorization: 'Bearer <OPENROUTER_API_KEY>',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    amount: 10,
    sender: '0x9a85CB3bfd494Ea3a8C9E50aA6a3c1a7E8BACE11',
    chain_id: 8453,
  }),
});
```

### Response Structure

The response includes charge details and Web3 transaction data:

```json
{
  "data": {
    "id": "...",
    "created_at": "2024-01-01T00:00:00Z",
    "expires_at": "2024-01-01T01:00:00Z",
    "web3_data": {
      "transfer_intent": {
        "metadata": {
          "chain_id": 8453,
          "contract_address": "0x03059433bcdb6144624cc2443159d9445c32b7a8",
          "sender": "0x9a85CB3bfd494Ea3a8C9E50aA6a3c1a7E8BACE11"
        },
        "call_data": {
          "recipient_amount": "...",
          "deadline": "...",
          "recipient": "...",
          "recipient_currency": "...",
          "refund_destination": "...",
          "fee_amount": "...",
          "id": "...",
          "operator": "...",
          "signature": "...",
          "prefix": "..."
        }
      }
    }
  }
}
```

## Executing the Transaction

Use an EVM client like viem to execute the transaction on-chain. The example utilizes Coinbase's `swapAndTransferUniswapV3Native()` function for native token swaps.

### Key Implementation Steps

- Extract contract address and calldata from the response
- Simulate the transaction to catch potential revert conditions
- Include sufficient ETH value (slightly above minimum) to cover gas and swap slippage
- Send the transaction and retain the hash for tracking

### Transaction Processing Timeline

- **Under $500:** Credits applied immediately upon on-chain confirmation
- **$500+:** Approximately 15-minute confirmation delay to prevent chain reorganization issues

## Monitoring Account Balance

Call `GET /api/v1/credits` periodically to check available credits and prevent service interruption.

### Using the SDK

```typescript
import { OpenRouter } from '@openrouter/sdk';

const openRouter = new OpenRouter({
  apiKey: '<OPENROUTER_API_KEY>',
});

const credits = await openRouter.credits.get();
console.log('Available credits:', credits.totalCredits - credits.totalUsage);
```

### Direct API Call

```typescript
const response = await fetch('https://openrouter.ai/api/v1/credits', {
  method: 'GET',
  headers: { Authorization: 'Bearer <OPENROUTER_API_KEY>' },
});
const { data } = await response.json();
```

### Balance Calculation

Your current balance equals total credits purchased minus total usage:

```json
{
  "data": {
    "total_credits": 50.0,
    "total_usage": 42.0
  }
}
```

**Note:** These values are cached and may lag up to 60 seconds.
