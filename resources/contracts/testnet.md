---
description: >-
  The contract addresses for the deployed Everclear contracts in our testnet
  environment.
---

# Testnet

{% hint style="warning" %}
This is a testnet environment, subject to frequent changes.
{% endhint %}

## Supported Chains

| Chain                | ChainID  | DomainID | Role  |
| -------------------- | -------- | -------- | ----- |
| Everclear-Sepolia    | 6398     | 6398     | Hub   |
| Sepolia              | 11155111 | 11155111 | Spoke |
| Chapel (BNB Testnet) | 97       | 97       | Spoke |
| Optimism Sepolia     | 11155420 | 11155420 | Spoke |
| Arbitrum Sepolia     | 421614   | 421614   | Spoke |

## Deployed Contracts

| Contract Name  | DomainID | Address                                     |
| -------------- | -------- | ------------------------------------------- |
| EverclearHub   | 6398     | 0x02cfc75A2C24Cf698AFb53017c1a19e16C4f14da  |
| HubGateway     | 6398     | 0x0b6241a74D521d947E02a12D1f409Bad563c7833  |
| EverclearSpoke | 11155111 | 0x83d58be7DAbd2C12df29FE80d24a0A661BC88B11  |
| SpokeGateway   | 11155111 | 0x557A0AF3F73Ee8627e60ec813d6CF29cA0f90c77  |
| EverclearSpoke | 97       | 0x83d58be7DAbd2C12df29FE80d24a0A661BC88B11  |
| SpokeGateway   | 97       | 0x557A0AF3F73Ee8627e60ec813d6CF29cA0f90c77  |
| EverclearSpoke | 11155420 | 0xd18C5E22E67947C8f3E112C622036E65a49773ab  |
| SpokeGateway   | 11155420 | 0x688a75702c4cF772164b7a448df3Db018199D414D |
| EverclearSpoke | 421614   | 0xd18C5E22E67947C8f3E112C622036E65a49773ab  |
| SpokeGateway   | 421614   | 0x688a75702c4cF772164b7a448df3Db018199D414  |

## Registered Assets

A few varieties of tokens are available to test with.

* TEST: Generic ERC20 with 18 decimals across all chains.&#x20;
* DTT: ERC20 with different decimal denominations on certain chains.
* TestxERC20: xERC20 with 18 decimals across all chains.

<table><thead><tr><th width="147">Asset Name</th><th width="89">Symbol</th><th width="104">Decimals</th><th width="109">DomainID</th><th width="175">Address</th><th width="166">Faucet</th><th width="134">Faucet Limit</th></tr></thead><tbody><tr><td>Test Token</td><td>TEST</td><td>18</td><td>11155111</td><td>0xd26e3540A0A368845B234736A0700E0a5A821bBA</td><td>(open mint)</td><td>N/A</td></tr><tr><td>Test Token</td><td>TEST</td><td>18</td><td>97</td><td>0x5f921E4DE609472632CEFc72a3846eCcfbed4ed8</td><td>(open mint)</td><td>N/A</td></tr><tr><td>Test Token</td><td>TEST</td><td>18</td><td>11155420</td><td>0x7Fa13D6CB44164ea09dF8BCc673A8849092D435b</td><td>(open mint)</td><td>N/A</td></tr><tr><td>Test Token</td><td>TEST</td><td>18</td><td>421614</td><td>0xaBF282c88DeD3e386701a322e76456c062468Ac2</td><td>(open mint)</td><td>N/A</td></tr><tr><td>DecimalsTestToken</td><td>DTT</td><td>6</td><td>11155111</td><td>0xd18C5E22E67947C8f3E112C622036E65a49773ab</td><td>0x277b67ce20c83e1ad9825c152762ba2b9b297aa6</td><td>100 * 1e6 per day</td></tr><tr><td>DecimalsTestToken</td><td>DTT</td><td>18</td><td>97</td><td>0xdef63AA35372780f8F92498a94CD0fA30A9beFbB</td><td>0xca7c45a3e5bdf9d6db5dae64c41204195879042f</td><td>100 * 1e18 per  day</td></tr><tr><td>DecimalsTestToken</td><td>DTT</td><td>6</td><td>11155420</td><td>0x294FD6cfb1AB97Ad5EA325207fF1d0E85b9C693f</td><td>0x1f150e59d87e53b6543cdaee964b7f4f074e2867</td><td>100 * 1e6 per day</td></tr><tr><td>DecimalsTestToken</td><td>DTT</td><td>6</td><td>421614</td><td>0xDFEA0bb49bcdCaE920eb39F48156B857e817840F</td><td>0xc194c96430a43ebcd6a19c13813343f019492e5b</td><td>100 * 1e6 per day</td></tr><tr><td>TestxERC20</td><td>xTEST</td><td>18</td><td>11155111</td><td>0x8F936120b6c5557e7Cd449c69443FfCb13005751</td><td>0x2de7cc4291078d1e49b41ee382ec702f5e29b6ff</td><td>1000000 * 1e18 per day</td></tr><tr><td>TestxERC20</td><td>xTEST</td><td>18</td><td>97</td><td>0x9064cD072D5cEfe70f854155d1b23171013be5c7</td><td>0x3bb905a81de6928002e79cb2dd22badca5e78e2c</td><td>1000000 * 1e18 per day</td></tr><tr><td>TestxERC20</td><td>xTEST</td><td>18</td><td>11155420</td><td>0xD3D4c6845e297e99e219BD8e3CaC84CA013c0770</td><td>0x4902cd7bacda179cd8b08f824124821c62a493fc</td><td>1000000 * 1e18 per day</td></tr><tr><td>TestxERC20</td><td>xTEST</td><td>18</td><td>421614</td><td>0xd6dF5E67e2DEF6b1c98907d9a09c18b2b7cd32C3</td><td>0xb6abe199f9256c598df0646ca9c073276d412c90</td><td>1000000 * 1e18 per day</td></tr></tbody></table>