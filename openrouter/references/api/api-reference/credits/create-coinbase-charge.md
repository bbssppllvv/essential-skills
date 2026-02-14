# Create a Coinbase Charge for Crypto Payment

## Endpoint Overview

The API provides functionality to create a Coinbase charge for cryptocurrency payments through a POST request to `https://openrouter.ai/api/v1/credits/coinbase`.

## Request Details

**Method:** POST
**Content-Type:** application/json
**Authentication:** Bearer token in Authorization header (required)

### Request Body Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| amount | number | Yes | Credit amount to charge |
| sender | string | Yes | Ethereum wallet address initiating the transaction |
| chain_id | string | Yes | Blockchain network identifier (1, 137, or 8453) |

## Response Structure

A successful 200 response returns transaction calldata containing:

- **Charge ID:** Unique identifier for the charge
- **Timestamps:** Creation and expiration times
- **Web3 Data:** Transfer intent with call data and metadata including the smart contract address and deadline

## HTTP Status Codes

- **200:** Successful charge creation
- **400:** Invalid credit amount or malformed request
- **401:** Missing or invalid authentication credentials
- **429:** Rate limit exceeded
- **500:** Server-side error

## Code Examples

The documentation provides implementation examples in Python, JavaScript, Go, Ruby, Java, PHP, C#, and Swift, all using identical request parameters of 150 credits from a sample Ethereum address on mainnet (chain_id: 1).
