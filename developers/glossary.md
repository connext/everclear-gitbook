# Glossary

**Rebalancers** - centralised exchanges, solvers, market makers, and protocols rebalancing funds across chains

**Arbitrageurs** - professional and individual actors monitoring intents that have profitable discounts and providing liquidity when Rebalancers can not be netted against each other

**Intents** - requests to move token from origin chain to destination chain(s)

**Spoke** - the entry contract on domains used to create and fulfil intents

**Hub** - the clearing chain contract used to process epochs and settlements

**SpokeGateway** - the contract on domains receiving messages from the Spoke to send via Hyperlane to the Hub and receiving messages from Hyperlane sent by the Hub that should be forwarded to the Spoke contract

**HubGateway** - the contract on domains receiving messages from the Hub to send via Hyperlane to the Spoke and receiving messages from Hyperlane sent by the Spoke that should be forwarded to the Hub contract

**Intent creator/ owner** - user creating an intent

**Intent filler** - user fulfilling an intent

**Creating an intent** - moving a token from origin chain to destination chain(s) via Everclear

**Purchasing an intent** - providing funds via Everclear to purchase an invoice in the invoice queue

**Filling an intent** - providing funds via Everclear to fill an existing intent which may include sending tokens and executing calldata

**Settling an intent** - the process of sending the settlements from the Hub to the Spoke contracts to pay the intent creators the amount they are due from the netting process

**Invoices** - intents will be converted into invoices and added to the invoiceQueue when there are no invoices in the queue or when deposits remain unmatched after the processing step

**Deposits** - intents will be converted into deposits and added to the depositQueue when there are invoices in the queue - deposits can only be used to match invoices in the epoch they are received

**Settlements** - the amount that an intent creator should receive on a domain after an invoice and deposit has been matched will be converted into a settlement and added to the settlementQueue

**InvoiceQueue** - queue of intents on the clearing chain waiting to be fulfilled by items in the depositQueue when the epoch is processed

**DepositQueue** - queue of deposits on the clearing chain waiting to be matched with items in the invoiceQueue when the epoch is processed

**SettlementQueue** - once invoices and deposits are matched they will be removed from their respective queue, converted into a settlement and added to the settlement queue

**Intent processing** - processing the intent queue on the Spoke to transport intents to Hub

**Epoch Processing** - processing the invoice and deposit queues on Hub to match intents and convert to settlements

**Settlement Processing** - processing the settlement queue on Hub and sending messages to the Spoke contract to settle the intent owner

**Batch frequency** - The frequency queues are processed depends on the size of the queue or the age of the oldest item in the queue, varies by chain

**Netting Auction** - each invoice in the system is added to the auction where its price discounts by a predefined BPS for each epoch that passes until an arbitrageur fills the intent - invoices are exempt from discounts for the first epoch they are in the invoice queue

**Exemption period** - the first epoch an invoice is in the queue where it is exempt from being discounted

**Epoch** - the auction uses epochs to increase discounts on invoices

**Epoch length** - the number of blocks each epoch lasts

**Discount formula** - the formula used to calculate the amount an invoice should be discounted by

```tsx
discountBPS = (currentEpoch - entryEpoch - 1) * discountPerEpoch
```

**Discounted intent (full) -** the amount the original intent amount is discounted by

```tsx
discountBPS = (currentEpoch - entryEpoch - 1) * discountPerEpoch
amountAfterDiscount = _invoice.amount - ((_invoice.amount * _discountBps) / Constants.BPS_FEE_DENOMINATOR);
```

**Discounted intent (partial)** - if an invoice is only partially filled, the deposit amount filling the intent will be used to calculate a partial discount

```tsx
discountBPS = (currentEpoch - entryEpoch - 1) * discountPerEpoch
amountAfterDiscount = _invoice.amount - ((depositsAmount * _discountBps) / Constants.BPS_FEE_DENOMINATOR);
```
