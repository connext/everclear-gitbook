# Cross Chain Swaps

#### **Cross Chain Swaps (Different Assets)**

Cross chain swaps provides 1-2min swaps between assets on the origin and destination chains e.g. USDC Arbitrum to USDT on Ethereum.&#x20;

These intents are filled optimistically by the Everclear solver using our inventory. Fees are slightly higher than regular intents and depend on available liquidity and the route.

### 🧠 Example: **Fast Path Swap (Bridge + Swap)**

Transfer **10,000 USDC** from **Optimism (chainId 10)** → **Ethereum (chainId 1)** as **USDT**, via Fast Path.

➡️ Must include the `outputAsset` field for Swaps.

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
