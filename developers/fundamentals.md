# Fundamentals

**Architecture**

Everclear uses a Spoke and Hub model to transport intents and settlements between supported domains. Functions are triggered on the Spoke or Hub contracts, sent to the SpokeGateway or HubGateway, and transported between domains using Hyperlane. The Spoke domains are chains such as Ethereum Mainnet, Arbitrum, or Optimism whilst the Hub domain is the "Clearing chain" where the protocol's core logic is executed.&#x20;

Everclear is composed of Rebalancers using the system to rebalance funds across domains and Arbitrageurs monitoring the system to purchase discounted intents. Rebalancer intents will be netted against intents traveling in the opposing direction and when there are no intents to be matched with the transfer amount will be discounted by BPS every epoch. The amount will be discounted until a Rebalancer or Arbitrageur sends a matching new intent; up to a maximum discount threshold.

**Flow of Funds**

When a Rebalancer or Arbitraguer creates an intent, funds are pulled from their wallet and deposited in the Spoke contract. The unclaimed balance in the Spoke contract is increased by the intent amount, and it will be used to settle the user that purchases the original intent with a new intent.

New intents are added to the intent queue which is transported from the Spoke to the Clearing chain periodically. On the Hub domain, intents that cannot be matched with existing intents become invoices in the invoice queue whilst intents that can be matched become deposits in the deposit queue. The invoice and deposit queues are processed at the end of every epoch and matched items create settlements that are added to the settlement queue.

The settlements in the queue can be settled through different strategies depending on the intent type. An intent being netted would be settled with the netting strategy whereas xERC20 tokens being bridged would be settled with the xERC20 strategy. Two strategies that will be supported at launch: (1) Netting, and (2) xERC20.

Finally, the settlement queue is processed sending settlement messages from the Hub to the Spoke domains. Once the message has been transported to the corresponding Spoke domains, the destination domains for the matched intents, the user will receive tokens in their wallet or have an increased virtual balance depending on their settlement preference.

### **Components**

There are several on and off-chain components in the system.

#### **Supported Domains**

* `Spoke`. This spoke contract holds funds from created intents (used as liquidity for settlement). It formats messages from the `intentQueue` and dispatches them via the `SpokeGateway` contract.
* `SpokeGateway`. This spoke contract is responsible for dispatching messages to the transport layer, and formatting the inbound settlement messages to properly call the functions on the transport layer.
* `XERC20Module`. This spoke contract is responsible for burning and minting XERC20 tokens when interacting with the system.&#x20;
* `FeeAdapter`. This spoke contract is responsible for charging dynamic fees to users via the submission of a signed payload from our fee signer.

#### **Clearing Layer (Hub)**

* `Hub`. This clearing chain contract handles inbound intent messages and dispatches settlement via the gateway contract.
* `HubGateway`. This clearing chain contract is responsible for dispatching messages to the transport layer, and formatting the inbound message payloads to properly call the functions on the `Hub`.

#### Agents

* _Lighthouse._ This agent is responsible for managing the queues and executing cron jobs in the the network. Queues are processed according to configured age and size thresholds per chain.
* _Cartographer._ This agent is responsible for creating a natively cross-chain view of the network state. This view is derived from an indexing layer (i.e. subgraphs) on each chain.

#### **Transport Layer (Hyperlane)**

The transport layer is responsible for sending messages throughout the network. The messages are dispatched and processed by the `HubGateway` and `SpokeGateway` contracts.
