# xERC20

> For all integrations, follow the steps in the API section which will construct an order that is signed by our fee signer and can be submitted to the `FeeAdapter` contract to create a new intent. Orders submitted directly to the `EverclearSpoke` will revert as well as any orders submitted to the `FeeAdapter` without a validly signed payload.

## Process Overview

The XERC20 open token standard for a multi-chain ERC20 which can be transferred across chains with no slippage and without compromising security. Token issuers can control the chains their token is deployed to and set rate limits on a per-bridge basis - making the token natively multichain without compromises.

The process of setting up XERC20 infrastructure can be found in this guide; token issuers should contact our team once they would like their XERC20 to be supported by Everclear and provide a list of chains they would like to be supported (where the XERC20 token exists).

`XERC20` bridging **should only be used** once:

* XERC20 infrastructure deployed by the token issuer
* XERC20 token approved on the Hub&#x20;
* XERC20 token status stored on Spokes (as per request from token issuer)

The process of bridging an XERC20 with the Spoke is as follows:

1. User approves the `XERC20Module` contract to pull `XERC20` from wallet
2. User calls `newIntent` on the Spoke, funds are pulled from wallet to `XERC20Module` contract, and tokens are burned by the `XERC20Module`
3. Intents are transported from Spoke to clearing chain periodically when the queue size is more than a threshold of items or the oldest item in the queue is more than threshold of minutes
4. Clearing chain receives intent and adds to the invoice queue if it’s the only intent for the xERC20 tickerHash on Hub OR adds the intent to the deposit queue
5. Invoice and deposit queues are processed every epoch. As the `XERC20` does not need to be matched with another invoice, a settlement is automatically created and added to the settlement queue
6. The settlement queue is processed on the Hub and transported to the Spoke contract when the queue size is more than a threshold of item or the oldest item in the queue is more than threshold of minutes
7. Spoke contract receives SettlementMessage’s and calls the `XERC20Module` contract to mint tokens to the recipient

_If the minting limit has been reached, the `XERC20Module` will store the amount and recipient. The tokens can be minted when the limit resets by calling `mintDebt` ._

## Creating an xERC20 Intent

As a user, you will need to interact with the Spoke contract on the origin domain to create a new intent that will be processed by the netting system.

### `newIntent` called on `Spoke` contract

The entry point to bridge an XERC20 is newIntent on the Spoke contract of the origin domain. The `destinations` field can be defined as a single item or an array - the Hub will select the first supported domain to bridge to so we recommend solely providing a supported domain.

The `maxFee` **must be set to 0** as `maxFee` is not applicable to the XERC20 order type. The `ttl` must **be set to 0** to indicate the order should be routed via the native XERC20 route.

```solidity
/**
 * @notice Creates a new intent
 * @param _destinations The possible destination chains of the intent
 * @param _receiver The destinantion address of the intent
 * @param _inputAsset The asset address on origin
 * @param _outputAsset The asset address on destination
 * @param _amount The amount of the asset
 * @param _maxFee The maximum fee that can be taken by solvers
 * @param _ttl The time to live of the intent
 * @param _data The data of the intent
 * @return _intentId The ID of the intent
 * @return _intent The intent object
*/
  function newIntent(
    uint32[] memory _destinations,
    address _receiver,
    address _inputAsset,
    address _outputAsset,
    uint256 _amount,
    uint24 _maxFee,
    uint48 _ttl,
    bytes calldata _data
  ) external returns (bytes32 _intentId, Intent memory _intent);
```

When the `Spoke` contract completes the `newIntent` call, the intent will be added to the `intentQueue`, and periodically sent to the `Hub` contract on the clearing chain - depending on the configuration of the max queue size and age for the origin domain.

An intent can be created by interacting directly with the contract. The following is a simple example sending 100 xTOKEN via the XERC20 route from Sepolia Testnet to BNB Testnet.

```tsx
import { ethers, BigNumberish } from 'ethers'

// Wallet and contract configuration //
const PRIVATE_KEY = "<PRIVATE_KEY>";
const RPC_URL_ARB = "<RPC_URL>";
const SPOKE_ADDRESS = ""; // Spoke on origin chain
const SPOKE_ABI = ["function newIntent(uint32[] _destinations,address _to, address _inputAsset, address _outputAsset, uint256 _amount, bytes _data) external"];
const ERC20_ABI = ["function approve(address spender, uint256 amount) external"]

// Function inputs //
const DEST = ["97"] // Single item - Sending to BNB testnet
const DESTS = ["97","xyz"] // Multiple items - Sepolia and xyz testnet
const TO = "0x..." // Receiver address on destination domain
const XTOKEN_SEPOLIA_TEST = "0x...."; // XTOKEN on Sepolia testnet
const XTOKEN_BNB_TEST = "0x...."; // XTOKEN on BNB testnet
const AMOUNT_IN = ethers.toBigInt(100_000_000); // Amount being transferred
const MAX_FEE = 0; // No max fee applicable to XERC20 orders
const TTL = 0; // Specifying ttl as 0 for an XERC20 bridges via native XERC20 route

async function newIntent(): Promise<void> {
	// Configuring the provider and wallet for the solver
	const provider = new ethers.JsonRpcProvider(RPC_URL_ARB);
	const wallet = new ethers.Wallet(PRIVATE_KEY, provider);
	
	// Configuring the contract instance
	const xTokenContract = new ethers.Contract(XTOKEN_SEPOLIA_TEST, ERC20_ABI, wallet);
	const spokeContract = new ethers.Contract(SPOKE_ADDRESS, SPOKE_ABI, wallet);
	
	// Approving and waiting for tx to be mined
	const approveTx = await xTokenContract.approve(SPOKE_ADDRESS, amount);
	await approveTx.wait(5);
	
	// Calling new intent to create an intent on OP
	const newIntentTx = await spokeContract.newIntent(DEST, TO, XTOKEN_SEPOLIA_TEST, USDT_BNB_TEST, AMOUNT_IN, MAX_FEE, TTL, ""); 
	await newIntentTx.wait(5);
}

newIntent(); 
```

Domain Ids will correspond to standard chain Ids.

## **Monitoring Intents**

The status of an intent can be fetched from the Subgraph which will update the intent on each domain as they are being processed. Token issuers should be able to query the Subgraph using the `intentId` to monitor the progress it has made through the system whilst users bridging via the UI will be able to find all of the required information there.

For token issuers, intents created on the `Spoke` contract will be added to the origin domain Subgraph as an `OriginIntent` entity. The status of the processing on the hub domain will be added to the Subgraph as a `HubIntent` entity. The settlement of the intent will be added to the destination domain Subgraph under `IntentSettleEvent` entity.

There are 3 statuses used for `OriginIntent` entities:

* NONE: does not exist
* ADDED: signifies added to the message queue
* DISPATCHED: signifies the batch containing the message has been sent

There are 9 statuses used for `HubIntent` entities:

* NONE: does not exist
* ADDED: intent has been added to the Hub
* DEPOSIT\_PROCESSED: has been added to the depositQueue
* FILLED: deposit has purchased an invoice in the queue
* COMPLETED: has been added to settlement queue on Hub
* INVOICED: has been added to the invoiceQueue
* SETTLED: settlement has been sent from Hub to Spoke
* UNSUPPORTED: the asset is not supported on the origin/destination domains or the wrong output asset has been provided
* UNSUPPORTED\_RETURNED: the unsupported intent has been returned to the origin domain

The existence of the `IntentSettleEvent` entity for an intentId implies it has been settled on the destination domain.

### 1 - Monitoring intent status

The Subgraph can be queried on the origin domain to pull information about the intent using the `intentId`; Id will remain the same across the domains. Schema for `OriginIntent`:

```graphql
type OriginIntent @entity {
  id: Bytes! # intent id
  queueIdx: BigInt!
  message: Message

  status: IntentStatus!

  initiator: Bytes!
  receiver: Bytes!
  inputAsset: Bytes!
  outputAsset: Bytes!
  maxRoutersFee: BigInt!
  origin: BigInt!
  destination: BigInt!
  nonce: BigInt!
  amount: BigInt!
  data: Bytes

  isTransfer: Boolean!

  # Add Intent Transaction
  addEvent: IntentAddEvent!

  # Bumps
  bumps: [IntentBumpedEvent!]! @derivedFrom(field: "intent")
}
```

For example, if the origin domain Subgraph was being queried to fetch the status of an intent, it could be constructed as follows using its `intentId`:

```tsx
import { gql, request } from 'graphql-request'

const intentStatus = gql`
  query GetIntentStatus($id: Bytes!) {
    originIntents(where: { id: $id }) {
      status
    }
  }
`;

await request('EVERCLEAR_GRAPH_URL_ORIGIN', intentStatus, { id: 'YOUR_INTENT_ID' });
```

### **2 - Monitoring purchase status**

The Subgraph on the hub domain can be queried to pull information about the intent’s processing status using the `intentId`. Schema for `HubIntent`:

```graphql
type HubIntent @entity {
  id: Bytes!
  status: HubIntentStatus!

  settlement: SettlementMessage

  addEvent: IntentAddEvent
  fillEvent: IntentFillEvent

  queue: SettlementQueue
  queueNode: Bytes

  bumps: [BumpProcessedEvent!]! @derivedFrom(field: "intent")
}
```

For example, if the hub domain Subgraph was queried to fetch the status of the intent, it could be constructed as follows using its `intentId`:

```tsx
import { gql, request } from 'graphql-request'

const purchaseStatus = gql`
  query GetPurchaseStatus($id: Bytes!) {
    hubIntents(where: { id: $id }) {
      status
    }
  }
`;

await request('EVERCLEAR_GRAPH_URL_HUB', purchaseStatus, { id: 'YOUR_INTENT_ID' });
```

### 3 - Monitoring settlement status

The Subgraph on the destination domain can be queried to pull information about the settlement using the `intentId`. Schema for `IntentSettledEvent`:

```graphql
type IntentSettleEvent @entity(immutable: true) {
  id: Bytes!
  intentId: Bytes!

  filler: Bytes!
  asset: Bytes!
  amount: BigInt!

  # Settle Transaction
  transactionHash: Bytes!
  timestamp: BigInt!
  gasPrice: BigInt!
  gasLimit: BigInt!
  blockNumber: BigInt!
  txOrigin: Bytes!

  txNonce: BigInt!
}
```

For example, if the destination domain Subgraph was queried to fetch the status of the settlement, it could be constructed as follows using its `intentId`:

```tsx
import { gql, request } from 'graphql-request'

const settleStatus = gql`
  query getSettledStatus($intentId: Bytes!) {
    intentSettleEvents(where: { intentId: $intentId }) {
      asset
      amount
    }
  }
`;

await request('EVERCLEAR_GRAPH_URL_DEST', settleStatus, { intentId: 'YOUR_INTENT_ID' });
```
