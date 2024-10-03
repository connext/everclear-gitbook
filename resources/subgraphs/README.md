# Subgraphs

Y can find deployed subgraphs by environment in the sub-sections. Below, the main entities are described.

## **Spoke Entities**

The main entities that can be queried in the Spoke Subgraph are:

* **OriginIntent**: tracks status and information related to an intent created on this domain
* **DestinationIntent**: tracks status and information related to an intent settled on this domain
* **IntentSettleEvent**: created when an intent is settled on destination via `_handleSettlement`
* **Queue**: the intent queue on the domain

**IntentStatus**

```graphql
enum IntentStatus {
  NONE
  ADDED # signifies added to the message queue
  DISPATCHED # signifies the batch containing the message has been sent
  SETTLED # signifies settlement has arrived on spoke domain for intent
  SETTLED_AND_MANUALLY_EXECUTED # settlement has arrived & calldata executed
}
```

**OriginIntent**

```graphql
type OriginIntent @entity {
  id: Bytes! # intent id
  queueIdx: BigInt!
  message: Message
  settlement: SpokeSettlement

  status: IntentStatus!

  initiator: Bytes!
  receiver: Bytes!
  inputAsset: Bytes!
  outputAsset: Bytes!
  maxFee: BigInt!
  origin: BigInt!
  nonce: BigInt!
  timestamp: BigInt!
  ttl: BigInt!
  amount: BigInt!
  destinations: [BigInt!]
  data: Bytes!

  # Add Intent Transaction
  addEvent: IntentAddEvent!
}
```

**DestinationIntent**

```graphql
type DestinationIntent @entity {
  id: Bytes! # intent id
  queueIdx: BigInt!
  message: Message
  status: IntentStatus!
  settlement: SpokeSettlement

  initiator: Bytes!
  receiver: Bytes!
  inputAsset: Bytes!
  outputAsset: Bytes!
  maxFee: BigInt!
  origin: BigInt!
  nonce: BigInt!
  timestamp: BigInt!
  ttl: BigInt!
  amount: BigInt!
  destinations: [BigInt!]
  data: Bytes!

  fillEvent: IntentFillEvent!
  calldataExecutedEvent: ExternalCalldataExecutedEvent
}
```

**IntentSettleEvent**

```graphql
type IntentSettleEvent @entity(immutable: true) {
  id: Bytes!
  intentId: Bytes!

  account: Bytes!
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

**Queue**

```graphql
type Queue @entity {
  id: Bytes! # 'INTENT' or 'FILL'
  type: QueueType!
  lastProcessed: BigInt
  size: BigInt!
  first: BigInt!
  last: BigInt!
}
```

## **Hub Entities**

The main entities that can be queried in the Hub Subgraph are:

* **HubIntent:** tracks the status of an intent on the Hub
* **SettlementQueue:** tracks settlements in the queue
* **SettlementMessage:** settlement message being sent to the Spoke
* **Token:** information related to a token including fees, feeRecipients, and related assets
* **Asset:** information related to an asset on a domain
* **Solver:** information related to a Solver's configuration

**HubIntentStatus**

```graphql
enum HubIntentStatus {
  NONE
  ADDED
  DEPOSIT_PROCESSED
  FILLED
  ADDED_AND_FILLED
  INVOICED
  SETTLED
  SETTLED_AND_MANUALLY_EXECUTED
  UNSUPPORTED
  UNSUPPORTED_RETURNED
}
```

**HubIntent**

```graphql
type HubIntent @entity {
  id: Bytes!
  status: HubIntentStatus!
  pendingRewards: BigInt!

  settlement: HubSettlement

  addEvent: IntentAddEvent
  fillEvent: IntentFillEvent
  message: SettlementMessage
  pendingRewardsAddedEvents: [PendingRewardsAddedEvent!]!
}
```

**SettlementQueue**

```graphql
type SettlementQueue @entity {
  id: Bytes! # domain
  domain: BigInt!
  lastProcessed: BigInt
  size: BigInt!
  first: BigInt!
  last: BigInt!
}
```

**SettlementMessageType**

```graphql
enum SettlementMessageType {
  SETTLED
  UNSUPPORTED_RETURNED
}
```

**SettlementMessage**

```graphql
type SettlementMessage @entity(immutable: true) {
  id: Bytes!
  quote: BigInt!
  domain: BigInt!
  intentIds: [Bytes!]!
  type: SettlementMessageType!

  txOrigin: Bytes!
  transactionHash: Bytes!
  timestamp: BigInt!
  blockNumber: BigInt!
  txNonce: BigInt!
  gasLimit: BigInt!
  gasPrice: BigInt!
}
```

**Token**

```graphql
type Token @entity {
  id: Bytes! # Ticker Hash
  feeRecipients: [Bytes!]
  feeAmounts: [BigInt!]
  maxDiscountBps: BigInt!
  assets: [Asset!]! @derivedFrom(field: "token")
}
```

**Asset**

```graphql
type Asset @entity(immutable: true) {
  id: Bytes! # Asset hash
  token: Token
  domain: BigInt
  adopted: Bytes!
  approval: Boolean!
}
```

#### Solver

```graphql
type Solver @entity {
  id: Bytes! # solver address
  supportedDomains: [BigInt!]
  updateVirtualBalance: Boolean!
}
```

For more information visit the Github repo: _Coming soon._
