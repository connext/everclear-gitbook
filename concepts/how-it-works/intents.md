---
description: >-
  Everclear supports the direct integration of solver networks and makes no
  assumptions about signaling, discovery, matching, or execution of the intent.
---

# Intents

## Lifecycle

### Creation

Intents are created by calling `newIntent` on the `FeeAdapter` contract:

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
   * @param _destinations The destination chains of the intent
   * @param _to The destinantion address of the intent
   * @param _inputAsset The asset address on origin
   * @param _outputAsset The asset address on destination
   * @param _amount The amount of the asset
   * @param maxFee The maximum fee that can be taken by solvers
   * @param ttl The time to live of the intent
   * @param _data The data of the intent
   * @param _feeParams The fee parameters
   * @return _intentId The ID of the intent
   * @return _intent The intent object
   */
  function newIntent(
    uint32[] memory _destinations,
    address _to,
    address _inputAsset,
    address _outputAsset,
    uint256 _amount,
    uint24 _maxFee,
    uint48 _ttl,
    bytes calldata _data,
    IFeeAdapter.FeeParams calldata _feeParams
  ) external whenNotPaused returns (bytes32 _intentId, Intent memory _intent) {
```

This method will debit funds from the caller, charge a fee, and forward the call onto the EverclearSpoke contract. The Spoke will then emit an `IntentAdded` event, and insert an `Intent` message into the queue.&#x20;

### Intent Queue Dispatch

Periodically, messages within this queue are dispatched to the clearing chain by offchain agents.

{% hint style="info" %}
Offchain agents respect configured thresholds for queue size and age of oldest item when evaluating whether or not to dispatch messages.
{% endhint %}

Once the message arrives on the clearing chain, the intent is marked as `Added`.

### Intent Fulfillment

At any point after intent creation, solvers can fill the intent by calling `fillIntent` on the destination `Spoke` contract if they have sufficient balance stored on-chain:

{% hint style="info" %}
Everclear makes no assumptions about solver selection or intent execution. Adding liquidity can be done at the time of intent fulfillment.
{% endhint %}

```solidity
  /**
   * @notice fills an intent
   * @param _intent The intent structure
   * @return _fillMessage The enqueued fill message
   */
  function fillIntent(
    Intent calldata _intent,
    uint24 _fee
  ) external whenNotPaused returns (FillMessage memory _fillMessage) {

```

This method will debit funds from the stored balance of the caller, emit an `IntentFilled` event, and insert a `Fill` message into the queue.

### Fill Queue Dispatch

Periodically, messages within this queue is dispatched to the clearing chain by offchain agents.

{% hint style="info" %}
Offchain agents respect configured thresholds for queue size and age of oldest item when evaluating whether or not to dispatch messages.
{% endhint %}

Once the message arrives on the clearing chain, the intent is marked as `Filled` if the `Intent` message is not present. If both messages have arrived, the intent is marked as `ADDED_AND_FILLED` and is enqueued for settlement.

When intents are marked for settlement, their solver has balance on the clearing chain that can be claimed on any of the supported spokes with sufficient liquidity.

### Settlement Queue Dispatch

Periodically, offchain agents will dispatch the enqueued settlements. Offchain agents will by default settle to the intent's specified destination chain(s) if there is sufficient available liquidity, otherwise will settle to the solver-configured domain with the highest available liquidity.

{% hint style="info" %}
Offchain agents respect configured thresholds for queue size and age of oldest item when evaluating whether or not to dispatch messages.
{% endhint %}

Once an intent is dispatched for settlement, it is marked as `SETTLED`.

### Settlement Processing

Once the dispatched settlements arrive on the spoke contracts, funds are either sent to the solver directly or their virtual balance is incremented depending on their preference.

{% hint style="info" %}
This preference is set via the `setUpdateVirtualBalance` function on the `Hub` contract.
{% endhint %}

## How can I reduce my settlement latency?

There is a tradeoff between batch latency and system base messaging costs. Offchain agents hosted by Everclear will be subject to configured thresholds, but solvers can always make their own cost vs. latency tradeoffs by dispatching the queues themselves and including the estimated Hyperlane fee in the `msg.value` of the following functions:

**Intent and Fill Queue Methods**

```solidity
interface IGateway {
  /**
   * @notice Quotes cost of sending a message to the transport layer
   * @param _message The message to send
   * @return _fee The fee to send the message
   */
  function quoteMessage(uint32 _chainId, bytes memory _message) external view returns (uint256 _fee);
}

interface IEverclearSpoke {
  /**
   * @notice Process the intent queue messages to send a batched message to the transport layer
   * @param _intents The intents to process, must respect the intent queue order
   */
  function processIntentQueue(Intent[] calldata _intents) external payable;

  /**
   * @notice Process the fill queue messages to send a batching message to the transport layer
   * @param _amount The amount of messages to process and batch
   */
  function processFillQueue(uint32 _amount) external payable;
}
```

#### Settlement Queue Methods

```solidity
interface IGateway {
  /**
   * @notice Quotes cost of sending a message to the transport layer
   * @param _message The message to send
   * @return _fee The fee to send the message
   */
  function quoteMessage(uint32 _chainId, bytes memory _message) external view returns (uint256 _fee);
}

interface ISettler {
  /**
   * @notice Dispatches batch settlements to the transport layer for a domain and amount
   * @param _domain The domain which settlements queue is going to be processed
   * @param _amount The amount of settlements to be batched
   */
  function processSettlementQueue(uint32 _domain, uint32 _amount) external payable;
}
```
