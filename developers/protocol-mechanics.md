# Protocol Mechanics

> For all integrations, follow the steps in the integration guides using the API which will construct a transaction request with a signed payload from our fee signer. The signed payload is required to submit a valid order to the `FeeAdapter` contract when creating a new intent.

## **Process Overview**

Everclear’s netting process is composed of five steps:

1. **Intent creation**: creation of a `newIntent` and transport from Spoke to Clearing chain
2. **Intent processing**: processing `newIntent` on the Clearing chain and converting to an `invoice` or `deposit`
3. **Invoice and deposit processing**: processing `invoices` and `deposits` to create `settlements`
4. **Intent settlement**: processing `settlements` and transporting the settlement information from Clearing chain to `Spoke`
5. **Intent fulfillment**: receiving the `settlementMessage` on the Spoke and crediting the recipient with funds (or virtual balance)

The two main actors in the system are:

* **Rebalancers**: centralised exchanges, solvers, market makers, and protocols rebalancing funds across chains
* **Arbitrageurs**: professional and individual actors monitoring invoices that have profitable discounts and providing liquidity when Rebalancers can not be netted against each other

### **Intent creation**

Intents can be created by anybody interacting with the Spoke contracts on supported domains. To create an intent users should call the API endpoint at `POST /intents` specifying the token, amount, and destination chain in the request body. The API response will include the contract address and transaction data that should be signed and submitted by the user.

See the [API docs](api.md) for more details.

The intent will be added to the IntentQueue on the Spoke contract and this will be periodically sent in a batch to the Hub contract on the Clearing chain when the queue size is more than a defined amount or the oldest item in the queue is older than a threshold.

### **Intent processing**

Once the intent reaches the Clearing chain, the Hub contract will check if there are invoices for the token being sent - invoices are intents waiting to be purchased/matched. If there are invoices, the new intent will be converted to a deposit and added to the deposit queue. If not, the intent will be converted to an invoice and be added to the invoice queue.&#x20;

Invoices and deposits create a queue for the token and destination domain pair such as a queue of 20x USDC invoices/deposits traveling to OP. As new intents reach the clearing chain, this process is repeated increasing the size of the queues until they are processed.

### **Invoice and deposit processing**

The intents stored in queues on the Hub contract as invoices and deposits are iterated to complete the processing step. The processing step occurs every epoch with a protocol configured epoch length - the epoch acts as a trigger to process queues and a reference point when calculating invoice discounts.

The invoice queue is iterated and the intents in the deposit queue will be checked to determine if there is enough liquidity to match the current invoice. When there is enough liquidity, the deposit queue is iterated. If the deposit can buy the rest of the invoice with surplus it will be decreased and remain in the queue to be matched with the next invoice. Whereas if the deposit will be entirely used to match the the current invoice, it will be removed from the deposit queue.

In this step, we compute the discount applied to the invoice (if any). The discount will translate to rewards for the depositor meaning they will receive the difference between the intent’s original amount and discounted amount as an additional reward in the settlement step. For example:

* Invoice of 100 USDC traveling from Optimism to Arbitrum
* Invoice discounted from 100 to 99
* Deposit of 99 traveling from Arbitrum to Optimism
* Invoice owner will receive 99 on Arbitrum
* Deposit owner will receive 100 on Optimism (+$1)

The discount applied to the intent amount is calculated as:

```solidity
discountBPS = (currentEpoch - entryEpoch) * discountPerEpoch
amountAfterDiscount = _invoice.amount - ((_amountToBeDiscounted * _discountBps) / Constants.BPS_FEE_DENOMINATOR);
```

Where:

* **amountToBeDiscounted:** the invoice amount or the deposit amount (when partially filled)
* **currentEpoch:** The current epoch
* **entryEpoch:** The epoch the invoice was created
* **discountPerEpoch:** The discount amount per epoch

When an invoice and deposit are matched, two settlement objects are added to the settlement queue on the Hub contract for each owner containing the token, amount, and destination chain. Using the example above: settlements of 99 USDC on Arb for invoice owner and 100 USDC on OP for deposit owner would be added to the settlement queue.

Once the invoice queue has been iterated and all possible matches with deposits have been made, the remaining deposits in the deposit queue are also iterated. The system checks if there is liquidity on the destination chain specified by each unmatched deposit and creates a settlement if so. In essence, the unmatched deposits in the queue are netted against each other to improve settlement efficiency and prevent them from becoming invoices.

Deposits are only eligible to be matched with invoices in the epoch they are received. After this epoch passes, an unmatched deposit that can not be netted against anything will be removed from the deposit queue, converted to an invoice, and added to the invoice queue. The process above will then repeat until a new intent (deposit) can be used to match with the invoice.

### **Intent settlement**

Once settlement objects have been added to the settlement queue on Hub, they are marked as `SETTLED` and are processed by Everclear. The settlement queue is processed by domain meaning all settlements for all assets on a domain are processed in one transaction each on Hub.

This step will iterate through the settlement queue, create SettlementMessage’s for each settlement, and remove each item from the queue. When all settlements for the assets provided have been iterated, the SettlementMessages will be batched and transported to the Spoke contract on the specified domain.

When the message arrives on the Spoke, funds will be transferred from the contract to the recipient or the recipient's virtual balance will be increased depending on the settlement strategy.

## Netting Auction

The netting auction is used to discount invoices that have not been matched via the netting system.&#x20;

New intents received by the Hub contract will have a one epoch exemption period before a discount is applied. Once the invoice has been in the invoice queue for more than one epoch, a BPS discount will be applied for every passing epoch. The invoice can be discounted by a maximum threshold which is defined by the user in BPS when `newIntent` is called.&#x20;

As the discount increases, the system relies on Arbitrageurs to fill the invoice when it becomes profitable for them to do so. By fostering a healthy ecosystem of Arbitrageurs, invoices that can not be netted should be filled at efficient prices as the Arbitrageurs race to fill invoices and risk missing out on rewards if they try to take too much profit. We plan to reduce the barriers to entry for new Arbitrageurs to create an efficient marketplace that will provide optimal pricing for users.

Invoices in the invoice queue can also be filled when new intents from Rebalancers are received. If the invoice has been discounted the new intent from a Rebalancer will receive their intent amount plus the discounted amount as a reward; meaning they will receive more than they transferred. This provides additional pressure for Arbitrageurs to fill intents because Rebalancer may equally fill intents in the meantime.

## **Fees**

There are a few costs that will reduce the amount a user will receive when using Everclear:

* Dynamic protocol fee
* Gas
* Solver fee (if applicable)
* Discounts (if intents become invoices)

When creating a `newIntent`, the user will be charged the protocol fee to use the netting system currently calculated off-chain and charged when submitting an order to the `FeeAdapter` contract plus the gas fee for submitting a transaction onchain.

In addition, a solver fee should be supplied if the user specifies `ttl > 0` when creating a new intent. This is the gas cost for a solver to execute the intent on destination. It should be high enough to incentivize solvers to fill the intent.

Finally, if an intent is not filled within the `ttl` and no other intents are netted against it, it will be converted into an invoice and get discounted.&#x20;

For users transferring xERC20s, view the xERC20 section for more information.&#x20;

## **Batch frequency**

There are three points where queues are batch processed:

* Intent queues sent from Spoke(s) to Hub
* Invoice and deposit queues processed on Hub
* Settlement queues sent from Hub to Spoke

The queue processing speeds impacts how quickly intents are settled when using the netting system. Increasing the pace of processing will lead to a faster settlement experience for Rebalancers and higher APRs for Arbers as they can recycle funds in more epochs to purchase invoices. However, all queues are processed by Everclear and increasing the speed will lead to higher protocol costs and consequently a higher protocol fee applied to all intents.

All processing functions for queues are public meaning they could be executed by Rebalancers or Arbers who would like to speed up the process. The examples below will describe each queue process and reference how a user could execute a transaction to speed up a queue.

### Intent Queue

The intent queues holds the intents that have been submitted to the Spoke contracts that are waiting to be transported to the Hub. The processing speed for this queue may vary by chain based on the whether there are more than a number of items in the queue OR the oldest item is older than an amount of minutes.

The `processIntentQueue` function is public, allowing anybody to process the intentQueue by calling the function with a list of intents. To successfully call this function:

1. Query the Subgraph on the origin domain to fetch the intentQueue
2. Provide the list of intents in the correct order they are stored in the intentQueue
3. Provide msg.value that will pay Hyperlane to send the message from Spoke to Clearing chain

```solidity
/**
 * @notice Process the intent queue messages to send a batched message to the transport layer
 * @param _intents The intents to process, must respect the intent queue order
*/
function processIntentQueue(Intent[] calldata _intents) external payable;
```

### Invoice & Deposit Queue

The invoice and deposit queues hold intents received by the Hub, which are categorised depending on whether they can be matched with an existing invoice or not. This will be processed when there are more than a defined number of items in the queue or the oldest item is older than a threshold.

The `processDepositsAndInvoices` function is public, allowing anybody to process the deposit and invoice queue by calling the function with a ticker hash.

```solidity
/**
 * @notice Process the epochs for a specific asset
 * @param _tickerHash The asset for which epochs are going to be processed
 *
*/
function processDepositsAndInvoices(bytes32 _tickerHash) external;
```

### Settlement Queue

The settlement queue holds the invoices and deposits that have been matched on the Hub, where two settlements are generated to pay both of the owners on their corresponding chains. This will be processed when there are more than a defined number items in the queue or the oldest item is older than a threshold.

The `processSettlementQueue` function is public, allowing anybody to process the settlement queue by calling the function with a domain and amount. To successfully call the function:

1. Provide the domain that is being settled to
2. Provide the amount of settlements that should be processed

```solidity
/**
 * @notice Dispatches batch settlements to the transport layer for a domain and amount
 * @param _domain The domain which settlements queue is going to be processed
 * @param _amount The amount of settlements to be batched
*/
function processSettlementQueue(uint32 _domain, uint32 _amount) external payable;
```
