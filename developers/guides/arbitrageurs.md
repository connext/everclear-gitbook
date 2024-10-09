# Arbitrageurs

## **Process Overview**

Arbitrageurs are professional and individual actors monitoring invoices that have profitable discounts and providing liquidity when Rebalancers can not be netted against each other. By using Everclear, Arbitrageurs achieve attractive APRs on their portfolio whilst taking on low risk by purchasing discounted invoices on supported chains.

The process for Arbitrageurs is composed of two steps:

1. Monitoring discounted invoices in the system to identify opportunities.
2. Calling `newIntent` on the Spoke contract that a discounted invoice is travelling to (i.e. calling `newIntent` on the destination domain specified by the intent being purchased).

If the Arbitrageur is the first to purchase the intent on the destination domain they will receive the discounted amount as a reward.

For example:

* 100 USDT original intent sent from Optimism to Arbitrum
* Invoice discounted from 100 to 99
* Arbitraguer sends new intent of 99 USDT from Arbitrum to Optimism
* Original intent owner receives 99 USDT on Arbitrum
* Arbitraguer receives 100 USDT on Optimism

The steps involved in the arbitrage process are:

1. Monitor discounted invoice
2. Call `newIntent` on Spoke contract (destination domain of discounted invoice) and funds will be pulled from wallet to the contract
3. Intent transported to the Clearing chain periodically when the queue size is more than threshold of items or the oldest item in the queue is more than thresold of minutes
4. Clearing chain receives intent and adds to deposit queue
5. Invoice and deposit queue are processed every epoch and matched intents are added to the settlement queue
6. Settlement queue processed and transported to the Spoke when the queue size is more than threshold of items or the oldest item in the queue is more than threshold od minutes
7. Settlement on Spoke sends funds to the Arbitraguer

_// NOTE: These are the steps when an Arbitrageur successfully sends the first order to purchase an invoice; if the order is not first the invoice may already be purchased. In this case, the Arbitraguer’s intent would be matched with the next invoice in the queue but if there are no invoices in the queue or deposits to be netted with at processing time, the intent would be converted into an invoice and need to be purchased by a new intent._

The new intent sent by an Arbitrageur will incur the protocol fee. If the new intent is not matched and it finds itself waiting in the invoice queue for more than a whole epoch, it may begin to incur the auction discount. Visit the Risk section (coming soon) for recommendations on how avoid this situation and for information about matching, discounts, fees, and settlement visit Protocol Mechanics.

## **Monitoring discounted invoices**

The status of an intent can be fetched from the Everclear API which will update the intent on each domain as they are being processed. Arbitrageurs can identify intents that are being discounted (invoices) by querying the `/invoices` endpoint:

```
GET {{baseUrl}}/invoices?destinations=97&limit=100
```

This endpoint returns a paginated list of intents that have been converted into invoices and will grant rewards for Arbitrageurs that can supply liquidity _from_ domain 97. A sample response may look like:

{% code fullWidth="true" %}
```json
[
  {
    "intent_id": "0xDEADBEEF",
    "discount": "100", // in BPS
    "owner": "0xade09131C6f43fe22C2CbABb759636C43cFc181e",
    "entry_epoch": 1,
    "amount": 1000000000000000000
  },
  ...
]
```
{% endcode %}

See the full Everclear API specification here (coming soon).

An invoice’s `discount` is likely the most important factor in an Arbitrageur’s decision whether to purchase the invoice or not.

## Purchasing an intent

As an Arbitrageur, you will need to interact with the `Spoke` contract on the domain that the intent being purchased has specified as its destination domain.

### `newIntent` called on `Spoke` contract

The entry point to purchase an invoice (intent) is `newIntent` on the `Spoke` contract and this should be called on the destination domain for the invoice being purchased. `newIntent` is called because the netting system uses new intents from destination domains to match with existing invoices in the invoice queue on the Clearing chain i.e. creating a new intent moving USDT from Optimism to Arbitrum will purchase an existing invoice in the invoice queue moving USDT from Arbitrum to Optimism.

The `destinations` field is an array where multiple domains can be provided to allow the Arbitrageur to be settled on a preferred domain. When the invoice is purchased, the Arbitrageur’s `destinations` array will be iterated and the domain with the highest liquidity and lowest discount will be used for settlement. For example: if an invoice is travelling from Optimism to Arbitrum, the Arbitrageur’s new intent could be sent to the Spoke contract on Arbitrum with a destinations array containing multiple destinations such as \[Optimism, Polygon, BNB].

If the Arbitrageur provides multiple `destinations` and their deposit is not matched with an invoice, their invoice will be able to match with any of the provided domains when being processed. This may allow an Arbitraguer to increase the speed at which their deposit is settled and minimise the likelihood a discount will be applied to their intent as there are multiple domains for the intent to be settled on.

The `maxFee` field should **always be specified as 0** as maxFee is only applicable in cases where an order should be routed to the solver pathway and does not apply to the netting pathway.

The `ttl` input should **always be specified as 0** to indicate the order should be routed via the netting system on the Hub. When `ttl` is non-zero an order is routed via a separate solver pathway where the intent creator requires a dedicated solver to fill an intent for a fee. This pathway will not be supported at launch; Arbitrageurs must ensure all netting order fills **always specify `ttl` as 0.**

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

When the `Spoke` contract completes the `newIntent` call, the intent will be added to the `intentQueue`, and periodically sent to the `Hub` contract on the Clearing chain - depending on the configuration of the max queue size and age for the origin domain.

An intent can be created by interacting directly with the contract. The following is a simple example sending 100 USDT via the netting system from Sepolia Testnet to BNB Testnet -  `ttl` and `maxFee` are specified as 0 to indicate the netting system should be used.

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
const USDT_SEPOLIA_TEST = "0x7169D38820dfd117C3FA1f22a697dBA58d90BA06"; // USDT on Sepolia testnet
const USDT_BNB_TEST = "0x337610d27c682E347C9cD60BD4b3b107C9d34dDd"; // USDT on BNB testnet
const AMOUNT_IN = ethers.toBigInt(100_000_000); // Amount being transferred
const MAX_FEE = 0; // No max fee applicable to netting orders
const TTL = 0; // Specifying ttl as 0 indicates a netting order

async function newIntent(): Promise<void> {
  // Configuring the provider and wallet for the solver
  const provider = new ethers.JsonRpcProvider(RPC_URL_ARB);
  const wallet = new ethers.Wallet(PRIVATE_KEY, provider);

  // Configuring the contract instance
  const usdtContract = new ethers.Contract(USDT_SEPOLIA_TEST, ERC20_ABI, wallet);
  const spokeContract = new ethers.Contract(SPOKE_ADDRESS, SPOKE_ABI, wallet);

  // Approving and waiting for tx to be mined
  const approveTx = await usdtContract.approve(SPOKE_ADDRESS, amount);
  await approveTx.wait(5);

  // Calling new intent to purchase an intent on OP
  const newIntentTx = await spokeContract.newIntent(DEST, TO, USDT_SEPOLIA_TEST, USDT_BNB_TEST, AMOUNT_IN, MAX_FEE, TTL, ""); 
  await newIntentTx.wait(5);
}

newIntent(); 
```

Domain Ids will correspond to standard chain Ids.

## **Monitoring Intents**

The status of an intent sent by the Arbitrageur can be fetched from the Subgraph which will update the intent on each domain as they are being processed. Arbitrageurs should be able to query the Subgraph using the `intentId` to monitor the progress it has made through the system.

The intents created on the `Spoke` contract will be added to the origin domain Subgraph as an `OriginIntent` entity. The status of the purchase on the Clearing chain will be added to the Subgraph as a `HubIntent` entity. The settlement of the intent will be added to the destination domain Subgraph under `IntentSettleEvent` entity.

There are 3 statuses used for `OriginIntent` entities:

* NONE: does not exist
* ADDED: signifies added to the message queue
* DISPATCHED: signifies the batch containing the message has been sent

There are 9 statuses used for `HubIntent` entities:

* NONE: does not exist
* ADDED: intent has been added to the Clearing chain
* DEPOSIT\_PROCESSED: has been added to the depositQueue
* FILLED: deposit has purchased an invoice in the queue
* COMPLETED: has been added to settlement queue on Clearing chain
* INVOICED: has been added to the invoiceQueue
* SETTLED: settlement has been sent from Clearing chain to Spoke
* UNSUPPORTED: the asset is not supported on the origin/destination domains or the wrong output asset has been provided
* UNSUPPORTED\_RETURNED: the unsupported intent has been returned to the origin domain

The existence of the `IntentSettleEvent` entity for an intentId implies it has been settled on the destination domain.

### **1 - Monitoring intent status**

Once the new intent has been sent to the Spoke by the Arbitrageur, the Subgraph can be queried on the origin domain to pull information about it using the `intentId`; Id will remain the same across domains. Schema for `OriginIntent`:

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

The Subgraph on the Clearing chain (Hub) can be queried to pull information about the purchase using the `intentId`. Schema for `HubIntent`:

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

For example, if the Clearing chain domain Subgraph was queried to fetch the status of the purchase, it could be constructed as follows using its `intentId`:

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

### **3 - Monitoring settlement status**

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

For example, if the Clearing chain domain Subgraph was queried to fetch the status of the settlement, it could be constructed as follows using its `intentId`:

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

## Risk Mitigation

The main risk for an Arbitrageur will be sending a transaction to purchase a discounted invoice and not being the first in the queue. This will mean another Arbitrageur has purchased the discounted invoice and can lead to costs being incurred without receiving any rewards.

Examples will be provided to reduce risks as an Arbitrageur once updates to the smart contract implementation has been completed.
