---
description: >-
  The contract addresses for the deployed Everclear contracts in our mainnet
  environment.
---

# Mainnet

## Supported Chains

| Chain     | ChainID | DomainID | Role  |
| --------- | ------- | -------- | ----- |
| Everclear | 25327   | 25327    | Hub   |
| Ethereum  | 1       | 1        | Spoke |
| Optimism  | 10      | 10       | Spoke |
| BSC       | 56      | 56       | Spoke |
| Base      | 8453    | 8453     | Spoke |
| Arbitrum  | 421614  | 421614   | Spoke |

## Deployed Contracts

| Contract Name  | DomainID | Address                                    |
| -------------- | -------- | ------------------------------------------ |
| EverclearHub   | 25327    | 0xa05A3380889115bf313f1Db9d5f335157Be4D816 |
| HubGateway     | 25327    | 0xEFfAB7cCEBF63FbEFB4884964b12259d4374FaAa |
| EverclearSpoke | 1        | 0xa05A3380889115bf313f1Db9d5f335157Be4D816 |
| SpokeGateway   | 1        | 0x9ADA72CCbAfe94248aFaDE6B604D1bEAacc899A7 |
| EverclearSpoke | 10       | 0xa05A3380889115bf313f1Db9d5f335157Be4D816 |
| SpokeGateway   | 10       | 0x9ADA72CCbAfe94248aFaDE6B604D1bEAacc899A7 |
| EverclearSpoke | 56       | 0xa05A3380889115bf313f1Db9d5f335157Be4D816 |
| SpokeGateway   | 56       | 0x9ADA72CCbAfe94248aFaDE6B604D1bEAacc899A7 |
| EverclearSpoke | 8453     | 0xa05A3380889115bf313f1Db9d5f335157Be4D816 |
| SpokeGateway   | 8453     | 0x9ADA72CCbAfe94248aFaDE6B604D1bEAacc899A7 |
| EverclearSpoke | 42161    | 0xa05A3380889115bf313f1Db9d5f335157Be4D816 |
| SpokeGateway   | 42161    | 0x9ADA72CCbAfe94248aFaDE6B604D1bEAacc899A7 |

## Registered Assets

<table><thead><tr><th width="147">Asset Name</th><th width="89">Symbol</th><th width="104">Decimals</th><th width="109">DomainID</th><th width="175">Address</th><th width="166">Faucet</th><th width="134">Faucet Limit</th></tr></thead><tbody><tr><td>Wrapped Ether</td><td>WETH</td><td>18</td><td>1</td><td>0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2</td><td>(open mint)</td><td>N/A</td></tr><tr><td>Wrapped Ether</td><td>WETH</td><td>18</td><td>10</td><td>0x4200000000000000000000000000000000000006</td><td>(open mint)</td><td>N/A</td></tr><tr><td>Wrapped Ether</td><td>WETH</td><td>18</td><td>56</td><td>0x2170Ed0880ac9A755fd29B2688956BD959F933F8</td><td>(open mint)</td><td>N/A</td></tr><tr><td>Wrapped Ether</td><td>WETH</td><td>18</td><td>8453</td><td>0x4200000000000000000000000000000000000006</td><td>(open mint)</td><td>N/A</td></tr><tr><td>Wrapped Ether</td><td>WETH</td><td>18</td><td>42161</td><td>0x82aF49447D8a07e3bd95BD0d56f35241523fBab1</td><td>0x277b67ce20c83e1ad9825c152762ba2b9b297aa6</td><td>100 * 1e6 per day</td></tr><tr><td>USD Coin</td><td>USDC</td><td>6</td><td>1</td><td>0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48</td><td>0xca7c45a3e5bdf9d6db5dae64c41204195879042f</td><td>100 * 1e18 per  day</td></tr><tr><td>USD Coin</td><td>USDC</td><td>6</td><td>10</td><td>0x0b2C639c533813f4Aa9D7837CAf62653d097Ff85</td><td>0x1f150e59d87e53b6543cdaee964b7f4f074e2867</td><td>100 * 1e6 per day</td></tr><tr><td>USD Coin</td><td>USDC</td><td>18</td><td>56</td><td>0x8AC76a51cc950d9822D68b83fE1Ad97B32Cd580d</td><td>0xc194c96430a43ebcd6a19c13813343f019492e5b</td><td>100 * 1e6 per day</td></tr><tr><td>USD Coin</td><td>USDC</td><td>6</td><td>8453</td><td>0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913</td><td>0x2de7cc4291078d1e49b41ee382ec702f5e29b6ff</td><td>1000000 * 1e18 per day</td></tr><tr><td>USD Coin</td><td>USDC</td><td>6</td><td>42161</td><td>0xaf88d065e77c8cC2239327C5EDb3A432268e5831</td><td>0x3bb905a81de6928002e79cb2dd22badca5e78e2c</td><td>1000000 * 1e18 per day</td></tr><tr><td>Tether USD</td><td>USDT</td><td>6</td><td>1</td><td>0xdAC17F958D2ee523a2206206994597C13D831ec7</td><td>0x4902cd7bacda179cd8b08f824124821c62a493fc</td><td>1000000 * 1e18 per day</td></tr><tr><td>Tether USD</td><td>USDT</td><td>6</td><td>10</td><td>0x94b008aA00579c1307B0EF2c499aD98a8ce58e58</td><td>0xb6abe199f9256c598df0646ca9c073276d412c90</td><td>1000000 * 1e18 per day</td></tr><tr><td>Tether USD</td><td>USDT</td><td>18</td><td>56</td><td>0x55d398326f99059fF775485246999027B3197955</td><td></td><td></td></tr><tr><td>Tether USD</td><td>USDT</td><td>6</td><td>42161</td><td>0x3f3f5dF88dC9F13eac63DF89EC16ef6e7E25DdE7</td><td></td><td></td></tr></tbody></table>
