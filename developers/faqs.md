# FAQs

### **What is an intent?**

An intent is a transaction sent from an origin domain that should be delivered to a destination domain. In Everclear’s system, intents are rebalancing transactions where funds are moving between domains via our netting system or xERC20 bridge transactions.

### What is a Rebalancer?

Centralised exchanges, solvers, market makers, and protocols rebalancing funds across chains.

### What is an Arbitrageur?

Arbitrageurs are professional and individual actors monitoring intents that have profitable discounts. The Arbitrageurs in Everclear act as a source of funds when there are no Rebalancers to net intents against and improve the efficiency of our netting system.

### What is rebalancing?

Rebalancing is a core task professional actors such as market makers, solvers, intent protocols, and CEXs use to support the crypto ecosystem. Funds held by these actors are moved between different domains to ensure user orders can be filled, withdrawals can be supported, and risk can be adequately managed.

### **What is netting an intent?**

Intents are netted when they are processed successfully by our netting system. Netting is the simple process of taking multiple intents moving in opposing directions and matching them with each other so the intent owners receive what they desire but their assets do not need to move across chains.&#x20;

## Processes - Overview

### **How does the netting system work?**

The netting system creates queues of invoices and deposits from intents received on the Spoke domains on the Clearing chain. These queues are then matched with each other to enable users to travel between domains using our netting system. Invoices wait in the invoice queue for deposits moving in the opposing direction and they are netted against each other to achieve a low cost rebalancing transaction.

Our network is composed of professional actors who want to rebalance large sums of funds between domains at a low cost. The netting system provides a plug-and-play solution to achieve rebalancing goals - widespread adoption will enable participants to access an institutional grade transport mechanism between domains similar to the interbank infrastructure in traditional finance.

## Processes - Intent Creation

### **What is the process of a creating a new intent?**

To create an intent, the newIntent function on a `Spoke` contract is called which will add an intent to the intentQueue on the Spoke domain.

### How is the intentId created?

The `intentId` is created by taking the function inputs plus the initiator (msg.sender), origin domain, timestamp, and a nonce - encoding and then hashing this information. This approach ensures each `intentId` is unique even if the intent is identical to a previous requests due to the uniqueness of the domain, iterating nonce, and timestamp.

### What happens to ERC20 funds in an active intent?

The `Spoke` will pull funds from the `msg.sender` when calling `newIntent` and custody them on the contract. This allows Everclear to use the funds to finalise the settlement of the intent i.e. pay the user that is netted against the intent on the origin domain.

### How does an intent reach the Hub?

The intents created on a `Spoke` will be added to the `intentQueue` for that contract. This queue is processed by our relayers periodically based on whether (a) the size of the queue is larger than a threshold or (b) the oldest item in the queue is older than a threshold.

The `processIntentQueue` function takes an array of intents; the array is iterated upon and each item is hashed to and checked against an intentId in the queue (in order of the queue). Once all of the items are validly checked, the intents are encoded with the message type corresponding to process intents, and sent to the `SpokeGateway` which will send the message to the clearing chain.

The `processIntentQueue` function is publicly accessible, meaning any EOA can call it and if the intents provided are in the same order as the queue then the call will succeed. The Subgraph can be queried for this information and the example query for this can be found in the Rebalancer or Arbitrageur integration guides.

## Processes - Intent Netting

### How is an intent netted?

Intents in the system are netted against new intents created on the opposing domain that is being travelled to i.e. an existing intent moving USDC from Optimism to Arbitrum would be netted against a new intent moving USDC from Arbitrum to Optimism. The new intent is added to the `intentQueue` and processed as seen above.

### What is the process of netting an intent?

The first intent is delivered to the Hub domain and converted to an invoice, which waits to be netted against a deposit. When a new intent from the destination domain is delivered to the Hub domain and there are invoices it can be matched with, it will be converted to a deposit.

The invoices and deposits are then iterated matching the invoice with deposit(s) until the full amount has been netted. As this happens, the netted invoice will be added to the settlement queue as a settlement object and each deposit used will also be added to the settlement queue as a settlement object.

### How are multiple domains used?

Multiple domains can be provided as an input when creating a newIntent. If there are multiple domains, they will be iterated at the invoice/deposit queue processing time and the domain with the highest liquidity and lowest discount will be selected. Deposits from this domain will then be used to net against the invoice until it has been filled. Deposits will also follow a similar process, where the highest liquidity and lowest discount domain is used at settlement.

### What happens to the funds of the intent used as a deposit?

Just like the original intent, the funds from an intent that is used as a deposit will he custodied on the Spoke contract on the domain the `newIntent` was sent from. These funds will then be used to repay intents that have been netted through our system when they reach the domain.

### How does an intent reach the Hub?

Intents reach the hub through the `intentQueue` being processed in the `processIntentQueue` function on the `Spoke` contract. This queue is processed by our relayers periodically based on whether (a) the size of the queue is larger than a threshold or (b) the oldest item in the queue is older than a threshold.

The `processIntentQueue` function takes an array of Intent objects and the Spoke will check the intent exists before transporting it to the Hub. Each of the items are dequeued and the intents are sent to the `SpokeGateway` which will use Hyperlane to send the message to the clearing chain.

The `processIntentQueue` function is publicly accessible, meaning any EOA can call it by simply providing the current list of intents in order that the would like to process. Everclear will execute this process periodically based on a threshold related to the size of the queue and oldest item in the queue.

## Netting Auction Pricing

### What is the netting auction discount approach?

The netting auction discount, is the discount applied to an invoice when it has been in the invoice queue for more than one epoch. The invoice will be discounted for every passing epoch until there is a new intent that is received to match with it.

### When are invoices exempt from the discount?

Invoices will be exempt from being discounted for the first epoch they are in the invoice queue.&#x20;

### What is the max discount?

The max discount applicable to an invoice is provided by the user when they create a new intent.

### When would an invoice be discounted?

The discount would apply to an invoice when it is in the invoice queue for more than one epoch.

### **What is the discount formula?**

```solidity
discountBPS = (currentEpoch - entryEpoch - 1) * discountPerEpoch
amountAfterDiscount = _invoice.amount - ((_amountToBeDiscounted * _discountBps) / Constants.BPS_FEE_DENOMINATOR);
```

_Where the amountToBeDiscounted is the value of the invoice when it is being fulled netted or the value of the deposit when it is being partially filled._

## Batch Frequency

### How often are batches sent from Spoke to Hub and Hub to Spoke?

Batches are sent to the hub by relayers periodically based on whether (a) the size of the queue is larger than a threshold or (b) the oldest item in the queue is older than a threshold.

Specific timings coming soon.

### Can I make the timeframes faster?

Yes, the process function on the `Spoke` that sends intents to the `Hub` are public. This means it can be called to increase the speed of queue processing. To do this, an array of intents would need to be provided in the correct order to call the `processIntentQueue` function successfully.

The same applies to processing functions from Hub to Spoke - view the Fundamental Mechanics section for more information about this.

## Costs & Pricing

### What fees are incurred sending an intent via Everclear?

An intent will expire three possible fees:

1. Protocol fee
2. Gas Fee

### What is the protocol fee?

Coming soon.
