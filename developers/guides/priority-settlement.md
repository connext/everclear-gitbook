# Priority Settlement

#### **Priority Settlement (Same Asset)**

Priority settlement provides 1-2min cross chain transfers for the same asset on the origin and destination chains e.g. USDC Arbitrum to Ethereum.&#x20;

These intents are filled optimistically by the Everclear solver using our inventory. Fees are slightly higher than regular intents.

### 🧭 Intent Creation Flow

1. **Get Quote** → `/routes/quotes`\
   Preview available routes, estimated output, and fees.
2. **Create Intent** → `/intents/`\
   Submit chosen parameters (origin, destination, input/output assets).
3. **Execute Transaction** → On-chain\
   Use the returned calldata + target address from the API response.

### 🔹 Endpoints

Get quote ([Documentation](../api.md#post-intents)):

```
GET https://api.everclear.org/routes/quotes
```

Create intent ([Documentation](../api.md#post-routes-quotes)):

```
POST https://api.everclear.org/intents/
```

### 🧠 Example: **Fast Path (Same Asset)**

Transfer **10,000 USDT** from **Optimism** → **Ethereum**, same token (no swap).\
➡️ Remove the `outputAsset` field.

```js
const axios = require('axios');

// Construct intent
const data = JSON.stringify({
  "amount": "10000000000",
  "origin": "10",
  "destinations": ["1"],
  "inputAsset": "0xdAC17F958D2ee523a2206206994597C13D831ec7", // USDT on OP
  "to": "0xade09131C6f43fe22C2CbABb759636C43cFc181e",
  "from": "0xade09131C6f43fe22C2CbABb759636C43cFc181e",
  "isFastPath": true
});

const config = {
  method: 'post',
  url: 'https://api.everclear.org/routes/quotes',
  headers: { 
    'Content-Type': 'application/json'
  },
  data
};

// Get Quote
axios.request(config)
  .then((response) => console.log(JSON.stringify(response.data, null, 2)))
  .catch((error) => console.error(error));
```
