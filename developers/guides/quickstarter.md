# Quickstarter

## 🧩 Everclear Intents Overview

The **Everclear system** supports multiple types of **intents**, depending on user needs — speed, cost, and liquidity availability.\
Each intent represents a user’s instruction to transfer or swap assets across chains.

***

### 🔹 Intent Types

#### 1. **Basic Intent (Netting Path)**

* **Description:** The standard route for intent execution.
* **Speed:** \~20 minutes
* **Fees:** Lowest among all intent types
* **Path:** Uses the **regular netting path** where settlement happens after batching.

***

#### 2. **Priority Settlement (Same Asset)**

* **Description:** Faster settlement for transfers of the **same asset** across chains.
* **Speed:** < 2 minutes
* **Fees:** Slightly higher than Basic
* **Execution:** Filled directly by the **solver** from inventory.

***

#### 3. **Priority Settlement Swap (Bridge + Swap)**

* **Description:** Enables **cross-chain swaps** (different assets between chains).
* **Speed:** < 2 minutes
* **Fees:** Depends on liquidity and route (includes swap fees)
* **Execution:** Solver fulfills both **bridge + swap** in a single fast operation.

***

### ⚙️ Creating Intents

All intents are created through the **API**.\
Before creation, it’s recommended to **fetch quotes** to understand fees, expected output, and available routes.

***

### 💬 Getting Quotes

Use the **`/routes/quotes`** endpoint to get an estimate of:

* Expected output amount after fees
* Fee breakdown (fixed + variable)
* Route limits
* Settlement time estimates
* Availability of fast path options

***

#### 🧠 Example: Get Quote for Sending 10K USDC Optimism → USDT Ethereum

```js
const axios = require('axios');

const data = JSON.stringify({
  "origin": "10", // Origin chain ID (Optimism)
  "destinations": ["1"], // Destination chain IDs (Ethereum)
  "inputAsset": "0x0b2C639c533813f4Aa9D7837CAf62653d097Ff85", // USDC on OP
  "amount": "10000000000", // Amount in asset decimals (10,000 USDC = 1e10)
  "to": "0xade09131C6f43fe22C2CbABb759636C43cFc181e", // Receiver address
  "from": "0xade09131C6f43fe22C2CbABb759636C43cFc181e" // Sender address
});

const config = {
  method: 'post',
  url: 'https://api.everclear.org/routes/quotes',
  headers: { 
    'Content-Type': 'application/json'
  },
  data
};

axios.request(config)
  .then((response) => console.log(JSON.stringify(response.data, null, 2)))
  .catch((error) => console.error(error));
```

***

#### 💡 For Priority Settlement Swap

To request a quote for **Priority Settlement Swap** (different input/output assets), include `outputAsset` on the destination chain:

```js
const data = JSON.stringify({
  "origin": "10",
  "destinations": ["1"],
  "inputAsset": "0x0b2C639c533813f4Aa9D7837CAf62653d097Ff85", // USDC on OP
  "outputAsset": "0xdAC17F958D2ee523a2206206994597C13D831ec7", // USDT on ETH
  "amount": "10000000000",
  "to": "0xade09131C6f43fe22C2CbABb759636C43cFc181e",
  "from": "0xade09131C6f43fe22C2CbABb759636C43cFc181e"
});
```

***

### 📦 Sample Response

```json
{
  "fixedFeeUnits": "537600",
  "variableFeeBps": 0,
  "totalFeeBps": 0.5376,
  "expectedAmount": "9999462400",
  "currentLimit": "4999959549397",
  "settlementEstimate": {
    "p25Minutes": 11.75,
    "p50Minutes": 15,
    "p75Minutes": 19.75
  },
  "splitCount": 1,
  "fastPathQuote": {
    "fixedFeeUnits": "537600",
    "variableFeeBps": 3,
    "totalFeeBps": 3.5376,
    "expectedAmount": "9996462400"
  },
  "reqType": "bridge"
}
```

***

#### 🧾 Response Fields Explained

| Field                | Description                                       |
| -------------------- | ------------------------------------------------- |
| `fixedFeeUnits`      | Flat fee for the route                            |
| `variableFeeBps`     | Variable fee (in basis points) based on liquidity |
| `totalFeeBps`        | Combined total fee in basis points                |
| `expectedAmount`     | Expected output after all fees                    |
| `currentLimit`       | Current route capacity                            |
| `settlementEstimate` | Estimated time (in minutes) for completion        |
| `splitCount`         | Number of route splits (if multi-path)            |
| `fastPathQuote`      | Fee + output estimate for Fast Path execution     |
| `reqType`            | Request type (`bridge` / `swap`)                  |

***

#### 🧭 Notes

* Always **check `currentLimit`** to ensure your intent can be routed immediately.
* Fast path routes may be unavailable if solver liquidity is low.
* Basic path provides more predictable execution but slower settlement.



## 🚀 Creating Intents

After obtaining a quote via the `/routes/quotes` endpoint, you can create an **Intent** using the `/intents/` endpoint.\
The intent defines the exact on-chain transaction parameters required to initiate a transfer or swap through Everclear.

***

### 🧭 Intent Creation Flow

1. **Get Quote** → `/routes/quotes`\
   Preview available routes, estimated output, and fees.
2. **Create Intent** → `/intents/`\
   Submit chosen parameters (origin, destination, input/output assets).
3. **Execute Transaction** → On-chain\
   Use the returned calldata + target address from the API response.

***

### 🔹 Endpoint

```
POST https://api.everclear.org/intents/
```

***

### ⚙️ Common Request Parameters

| Field          | Type       | Required | Description                                           |
| -------------- | ---------- | -------- | ----------------------------------------------------- |
| `amount`       | `string`   | ✅        | Amount to send (in token decimals).                   |
| `origin`       | `string`   | ✅        | Chain ID of the source network.                       |
| `destinations` | `string[]` | ✅        | Array of destination chain IDs.                       |
| `inputAsset`   | `string`   | ✅        | Asset address on the origin chain.                    |
| `outputAsset`  | `string`   | Optional | Destination asset address (only for swaps).           |
| `to`           | `string`   | ✅        | Receiver wallet address.                              |
| `from`         | `string`   | ✅        | Sender wallet address.                                |
| `callData`     | `string`   | Optional | Additional encoded calldata (if any).                 |
| `isFastPath`   | `boolean`  | Optional | `true` for Fast Path, omit or `false` for Basic Path. |

***

### 🧠 Example 1: **Basic Intent (Netting Path)**

For the **regular 15–20 minute netting route** (lowest fees),\
➡️ **Remove both `outputAsset` and `isFastPath`**.

```js
const data = JSON.stringify({
  "amount": "10000000000",
  "origin": "10",
  "destinations": ["1"],
  "inputAsset": "0x0b2C639c533813f4Aa9D7837CAf62653d097Ff85", // USDC on OP
  "to": "0xade09131C6f43fe22C2CbABb759636C43cFc181e",
  "from": "0xade09131C6f43fe22C2CbABb759636C43cFc181e"
});
```

This will route the intent through the **standard netting system**, ideal for larger or lower-fee transfers.

***

### 🧠 Example 2: **Fast Path (Same Asset)**

Transfer **10,000 USDT** from **Optimism** → **Ethereum**, same token (no swap).\
➡️ Remove the `outputAsset` field.

```js
const data = JSON.stringify({
  "amount": "10000000000",
  "origin": "10",
  "destinations": ["1"],
  "inputAsset": "0xdAC17F958D2ee523a2206206994597C13D831ec7", // USDT on OP
  "to": "0xade09131C6f43fe22C2CbABb759636C43cFc181e",
  "from": "0xade09131C6f43fe22C2CbABb759636C43cFc181e",
  "isFastPath": true
});
```

***

### 🧠 Example 3: **Fast Path Swap (Bridge + Swap)**

Transfer **10,000 USDC** from **Optimism (chainId 10)** → **Ethereum (chainId 1)** as **USDT**, via Fast Path.

```js
const axios = require('axios');

const data = JSON.stringify({
  "amount": "10000000000", // 10,000 USDC (in decimals)
  "origin": "10", // Optimism
  "destinations": ["1"], // Ethereum
  "inputAsset": "0x0b2C639c533813f4Aa9D7837CAf62653d097Ff85", // USDC on OP
  "outputAsset": "0xdAC17F958D2ee523a2206206994597C13D831ec7", // USDT on ETH
  "to": "0xade09131C6f43fe22C2CbABb759636C43cFc181e",
  "from": "0xade09131C6f43fe22C2CbABb759636C43cFc181e",
  "callData": "",
  "isFastPath": true // Fast Path = under ~2 mins
});

const config = {
  method: 'post',
  url: 'https://api.everclear.org/intents/',
  headers: { 
    'Content-Type': 'application/json'
  },
  data
};

axios.request(config)
  .then((response) => console.log(JSON.stringify(response.data, null, 2)))
  .catch((error) => console.error(error));
```

***

### 📦 Response

```json
{
  "to": "0xd0185bfb8107c5b2336bC73cE3fdd9Bfb504540e",
  "data": "0x...",
  "chainId": 10,
  "value": "0"
}
```

***

### 🧾 Response Fields

| Field     | Type     | Description                                                                 |
| --------- | -------- | --------------------------------------------------------------------------- |
| `to`      | `string` | The **Fees Adapter** contract address — transaction target.                 |
| `data`    | `string` | Encoded calldata required for execution.                                    |
| `chainId` | `number` | Origin chain ID where transaction must be broadcast.                        |
| `value`   | `string` | Native token (in wei) to send along with the transaction (typically `"0"`). |
