# Rebalancers

> For all integrations, follow the steps in the Creating a new Intent section using the API which will construct a transaction request with a signed payload from our fee signer. The signed payload is required to submit a valid order to the `FeeAdapter` contract when creating a new intent.

## **Process Overview**

The `rebalancing` problem occurs when a Rebalancers funds are spread across domains and need to be moved to different domains to support user activity and/or seize opportunities. However, the rebalancing process can be costly, manual, and complex where delays can lead to missed opportunities.

Rebalancers are often centralised exchanges, solvers, and market makers using Everclear to rebalance their funds across domains. By using Everclear’s netting system they will be able to cut their costs by up to 10x and reduce the complexity of the rebalancing process.

The rebalancing process is simplified to a single transaction: calling `newIntent` on a FeeAdapter contract from the origin domain with the amount and destination specified. Once this has been executed, the Rebalancer can wait for their intent to be netted with another and funds will be received on the destination domain in their wallet (or to a specified recipient).

The steps involved in the netting process are:

1. Rebalancer calls `newIntent` and funds are pulled from wallet to `FeeAdapter` contract
2. Intents are transported from Spoke to clearing chain periodically when the queue size is more than a threshold of items or the oldest item in the queue is more than  threshold of minutes
3. Clearing chain receives intent and adds to deposit queue if it can be matched with an invoice OR adds intent to the invoice queue if there are no invoices to match with
4. Invoice and deposit queue is processed every epoch. Matched intents are added to the settlement queue and unmatched intents are added to the invoice queue
5. (Optional) If the intent is in the invoice queue for a whole epoch and is unmatched the netting auction discount begins to apply - discounting the invoice amount by a predefined BPS per epoch. This process repeats until the invoice is matched with a new intent (deposit).
6. The settlement queue is processed on the Hub and transported to the Spoke contract when the queue size is more than a threshold of item or the oldest item in the queue is more than threshold of minutes
7. Spoke contract receives SettlementMessage’s and either sends tokens or increases the virtual balance of the Rebalancer



Our estimates suggest 70% of intents will be matched with other Rebalancers in the netting system while the remaining 30% will be purchased by Arbitrageurs. For a detailed explanation of the matching process, discounts, fees, and settlement visit Protocol Mechanics section.



Rebalancers testing using our protocol are recommended to send intents with **value exceeding $250** on L2s. If the intent can't be netted against other flows, this will ensure Arbitrageurs fill the order with a reasonable BPS discount which will more accurately represent outcomes when using the system.&#x20;

## **Creating a new intent**

As a Rebalancer, you will need to interact with the `FeeAdapter` contract on the origin domain to create a new intent that will be processed by the netting system.

### `newIntent` called on `FeeAdapter` contract

The entry point to rebalance is `newIntent` on the `FeeAdapter` contract of the origin domain. The `destinations` field can be defined as a single item in an array if the Rebalancer wants to rebalance funds to a specific domain i.e. from OP to ARB. However, if the Rebalancer wants to rebalance to one of any domains they can provide a list and the system will rebalance to the domain that has the highest liquidity and lowest discount at settlement time.

The `maxFee` field should **always be specified as 0** as maxFee is only applicable in cases where an order should be routed to the solver pathway and does not apply to the netting pathway.

The `ttl` input should **always be specified as 0** to indicate the order should be routed via the netting system on the Hub. When `ttl` is non-zero an order is routed via a separate solver pathway where the intent creator requires a dedicated solver to fill the intent for a fee. This pathway will not be supported at launch; Rebalancers must ensure all netting orders **always specify `ttl` as 0.**

The `feeParams` consists of `fee`, `deadline`, and `signature` which would be generated through interacting with the API. The `fee` will be the amount being charged to the user, the `deadline` is the period of time where the fee is valid, and the `signature` will be the signed payload from the Everclear fee signer to confirm the provided inputs provided are valid.&#x20;

```solidity
  /**
  * @param fee The fee being charged on the inputAsset
  * @param deadline The deadline timestamp after which the sig is no longer valid
  * @param sig The signed payload from the fee signer for the intent 
  */
  struct FeeParams {
    uint256 fee;
    uint256 deadline;
    bytes sig;
  }
  
/**
 * @notice Creates a new intent
 * @param _destinations The possible destination chains of the intent
 * @param _to The destinantion address of the intent
 * @param _inputAsset The asset address on origin
 * @param _outputAsset The asset address on destination
 * @param _amount The amount of the asset
 * @param _maxFee The maximum fee that can be taken by solvers
 * @param _ttl The time to live of the intent
 * @param _data The data of the intent
 * @param _feeParams The fee parameters
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
    bytes calldata _data,
    IFeeAdapter.FeeParams calldata _feeParams
  ) external returns (bytes32 _intentId, Intent memory _intent);
```

When the `FeeAdapter` contract completes the `newIntent` call, it will charge the user fees, and forward the call to the `EverclearSpoke`. The intent will then be added to the `intentQueue`, and periodically sent to the `Hub` contract on the Clearing chain - depending on the configuration of the max queue size and age for the origin domain.

An intent can be created by interacting directly with the contract. The following is a simple example sending 100 USDT via the netting system from Sepolia Testnet to BNB Testnet - `ttl` and `maxFee` are specified as 0 to indicate the netting system should be used.

```tsx
import { ethers, BigNumberish } from 'ethers'

// Wallet and contract configuration //
const PRIVATE_KEY = "<PRIVATE_KEY>";
const RPC_URL_ARB = "<RPC_URL>";
const FEE_ADAPTER = ""; // FeeAdapter on origin chain
const SPOKE_ABI = ["function newIntent(uint32[] _destinations, address _receiver, address _inputAsset, address _outputAsset, uint256 _amount, uint24 _maxFee, uint48 _ttl, bytes _data) external"];
const ERC20_ABI = ["function approve(address spender, uint256 amount) external"]

// Function inputs //
const ORIGIN = "11155111"
const DEST = ["97"] // Single item - Sending to BNB testnet
const DESTS = ["97","xyz"] // Multiple items - Sepolia and xyz testnet
const TO = "0x..." // Receiver address on destination domain
const USDT_SEPOLIA_TEST = "0x7169D38820dfd117C3FA1f22a697dBA58d90BA06"; // USDT on Sepolia testnet
const USDT_BNB_TEST = "0x337610d27c682E347C9cD60BD4b3b107C9d34dDd"; // USDT on BNB testnet
const AMOUNT_IN = ethers.toBigInt(100_000_000); // Amount being transferred
const MAX_FEE = 0; // No max fee applicable to netting orders
const TTL = 0; // Specifying ttl as 0 indicates this is a netting order

async function newIntent(): Promise<void> {
    // Configuring the provider and wallet for the solver
    const provider = new ethers.JsonRpcProvider(RPC_URL_ARB);
    const wallet = new ethers.Wallet(PRIVATE_KEY, provider);
    
    // Configuring the contract instance
    const usdtContract = new ethers.Contract(USDT_SEPOLIA_TEST, ERC20_ABI, wallet);
    
    // Approving and waiting for tx to be mined
    const approveTx = await usdtContract.approve(FEE_ADAPTER, amount);
    await approveTx.wait(5);
    
    // Constructing the payload
    const payload = {
        origin: ORIGIN, // Example: Arbitrum Sepolia
        destinations: DEST, // Array of destination domains
        to: TO,
        inputAsset: USDT_SEPOLIA_TEST,
        amount: AMOUNT_IN.toString(),
        callData: "0x", // empty callData for netting orders
        maxFee: MAX_FEE.toString(),
        permit2Params: {
          nonce: "0",        // Placeholder values
          deadline: "0",
          signature: "0x"
        }
    };
    
    // Using API to generate TransactionRequest for a newIntent
    const txRequest = await fetch('https://api.testnet.everclear.org/intents', {
        method: 'POST',
        headers: {
          "Content-Type": "application/json"
        },
        body: JSON.stringify(payload)
    });
    
    // Submitting the order
    const txResponse = await wallet.sendTransaction(txRequest);
    const receipt = await txResponse.wait();
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
