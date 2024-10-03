# Rebalancers

## **Process Overview**

The `rebalancing` problem occurs when a Rebalancers funds are spread across domains and need to be moved to different domains to support user activity and/or seize opportunities. However, the rebalancing process can be costly, manual, and complex where delays can lead to missed opportunities.

Rebalancers are often centralised exchanges, solvers, and market makers using Everclear to rebalance their funds across domains. By using Everclear’s netting system they will be able to cut their costs by up to 10x and reduce the complexity of the rebalancing process.

The rebalancing process is simplified to a single transaction: calling `newIntent` on a Spoke contract from the origin domain with the amount and destination specified. Once this has been executed, the Rebalancer can wait for their intent to be netted with another and funds will be received on the destination domain in their wallet (or to a specified recipient).

The steps involved in the netting process are:

1. Rebalancer calls `newIntent` and funds are pulled from wallet to Spoke contract
2. Intents are transported from Spoke to clearing chain periodically when the queue size is more than a threshold of items or the oldest item in the queue is more than  threshold of minutes
3. Clearing chain receives intent and adds to deposit queue if it can be matched with an invoice OR adds intent to the invoice queue if there are no invoices to match with
4. Invoice and deposit queue is processed every epoch. Matched intents are added to the settlement queue and unmatched intents are added to the invoice queue
5. (Optional) If the intent is in the invoice queue for a whole epoch and is unmatched the netting auction discount begins to apply - discounting the invoice amount by a predefined BPS per epoch. This process repeats until the invoice is matched with a new intent (deposit).
6. The settlement queue is processed on the Hub and transported to the Spoke contract when the queue size is more than a threshold of item or the oldest item in the queue is more than threshold of minutes
7. Spoke contract receives SettlementMessage’s and either sends tokens or increases the virtual balance of the Rebalancer



Our estimates suggest 70% of intents will be matched with other Rebalancers in the netting system while the remaining 30% will be purchased by Arbitrageurs. For a detailed explanation of the matching process, discounts, fees, and settlement visit Protocol Mechanics section.

## **Creating a new intent**

As a Rebalancer, you will need to interact with the `Spoke` contract on the origin domain to create a new intent that will be processed by the netting system.

### `newIntent` called on `Spoke` contract

The entry point to rebalance is `newIntent` on the `Spoke` contract of the origin domain. The `destinations` field can be defined as a single item in an array if the Rebalancer wants to rebalance funds to a specific domain i.e. from OP to ARB. However, if the Rebalancer wants to rebalance to one of any domains they can provide a list and the system will rebalance to the domain that has the highest liquidity and lowest discoutn at settlement time.

```solidity
/**
 * @notice Creates a new intent
 * @param _destinations The possible destination chains of the intent
 * @param _to The destinantion address of the intent
 * @param _inputAsset The asset address on origin
 * @param _outputAsset The asset address on destination
 * @param _amount The amount of the asset
 * @param _data The data of the intent
 * @return _intentId The ID of the intent
 * @return _intent The intent object
*/
  function newIntent(
    uint32[] memory _destinations,
    address _to,
    address _inputAsset,
    address _outputAsset,
    uint256 _amount,
    bytes calldata _data
  ) external returns (bytes32 _intentId, Intent memory _intent);
```

When the `Spoke` contract completes the `newIntent` call, the intent will be added to the `intentQueue`, and periodically sent to the `Hub` contract on the Clearing chain - depending on the configuration of the max queue size and age for the origin domain.

An intent can be created by interacting directly with the contract. The following is a simple example sending 100 USDT via the netting system from Sepolia Testnet to BNB Testnet.

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
    
    // Calling new intent to create an intent on OP
    const newIntentTx = await spokeContract.newIntent(DEST, TO, USDT_SEPOLIA_TEST, USDT_BNB_TEST, AMOUNT_IN, ""); 
    await newIntentTx.wait(5);
}

newIntent(); 
```

Domain Ids will correspond to standard chain Ids.

## **Monitoring Intents**

The status of an intent can be fetched from the Subgraph which will update the intent on each domain as they are being processed. Rebalancers should be able to query the Subgraph using the `intentId` to monitor the progress it has made through the system.

The intents created on the `Spoke` contract will be added to the origin domain Subgraph as an `OriginIntent` entity. The status of the purchase on the Hub domain will be added to the Subgraph as a `HubIntent` entity. The settlement of the intent will be added to the destination domain Subgraph under `IntentSettleEvent` entity.

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

The existence of the `IntentSettleEvent` entity for an intentId on the destination chain implies it has been settled.

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

The Subgraph on the Clearing chain (Hub) can be queried to pull information about the intent’s purchase status using the `intentId`. Schema for `HubIntent`:

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

For example, if the Hub domain Subgraph was queried to fetch the status of the intent, it could be constructed as follows using its `intentId`:

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
