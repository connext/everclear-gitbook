# Rebalancers

## **Process Overview**

The `rebalancing` problem occurs when a Rebalancer's funds are spread across domains and need to be moved to different domains to support user activity and/or seize opportunities. However, the rebalancing process can be costly, manual, and complex where delays can lead to missed opportunities.

Rebalancers are often centralised exchanges, solvers, and market makers using Everclear to rebalance their funds across domains. By using Everclear’s netting system they will be able to cut their costs by up to 10x and reduce the complexity of the rebalancing process.

The rebalancing process is simplified to a single transaction: calling `newIntent` on a FeeAdapter contract from the origin domain with the amount and destination specified. The transaction data is provided by the Everclear API. Once this has been executed, the Rebalancer can wait for their intent to be netted with another and funds will be received on the destination domain in their wallet (or to a specified recipient).

The steps involved in the netting process are:

1. Rebalancer requests transaction data from the Everclear API at [`POST /intents`](../api.md#post-intents). This transaction data is signed and submitted onchain. The transaction corresponds to a `newIntent` call on the `FeeAdapter` contract, funds are pulled from wallet to, fees are charged, and the call is forwarded onto the `EverclearSpoke`
2. Intents are transported from Spoke to clearing chain periodically when the queue size is more than a threshold of items or the oldest item in the queue is more than  threshold of minutes
3. Clearing chain receives intent and adds to deposit queue if it can be matched with an invoice OR adds intent to the invoice queue if there are no invoices to match with
4. Invoice and deposit queue is processed every epoch. Matched intents are added to the settlement queue and unmatched intents are added to the invoice queue
5. (Optional) If the intent is in the invoice queue for a whole epoch and is unmatched the netting auction discount begins to apply - discounting the invoice amount by a predefined BPS per epoch. This process repeats until the invoice is matched with a new intent (deposit).
6. The settlement queue is processed on the Hub and transported to the Spoke contract when the queue size is more than a threshold of item or the oldest item in the queue is more than threshold of minutes
7. Spoke contract receives SettlementMessage’s and either sends tokens or increases the virtual balance of the Rebalancer

Our estimates suggest 70% of intents will be matched with other Rebalancers in the netting system while the remaining 30% will be purchased by Arbitrageurs. For a detailed explanation of the matching process, discounts, fees, and settlement visit Protocol Mechanics section.

Rebalancers testing using our protocol are recommended to send intents with **value exceeding $250** on L2s. If the intent can't be netted against other flows, this will ensure Arbitrageurs fill the order with a reasonable BPS discount which will more accurately represent outcomes when using the system.&#x20;

## **Creating a new intent**

Rebalancers will first request the transaction data from the Everclear API at the [`POST /intents`](../api.md#post-intents) endpoint. It returns the correct parameters for a `newIntent` call on the spoke contract of the requested origin domain. This transaction data should be signed and submitted onchain.

The transaction data from the API will correspond to a `newIntent` on the `FeeAdapter` contract. The `destinations` field can be defined as a single item in an array if the Rebalancer wants to rebalance funds to a specific domain i.e. from OP to ARB. However, if the Rebalancer wants to rebalance to one of any domains they can provide a list and the system will rebalance to the domain that has the highest liquidity and lowest discount at settlement time.

The `maxFee` field should **always be specified as 0** as maxFee is only applicable in cases where an order should be routed to the solver pathway and does not apply to the netting pathway.

The `ttl` input should **always be specified as 0** to indicate the order should be routed via the netting system on the Hub. When `ttl` is non-zero an order is routed via a separate solver pathway where the intent creator requires a dedicated solver to fill the intent for a fee. This pathway will not be supported at launch; Rebalancers must ensure all netting orders **always specify `ttl` as 0.**

The `feeParams` consists of `fee`, `deadline`, and `signature` which would be generated through interacting with the API. The `fee` will be the amount being charged to the intent creator (levied against the input amount), the `deadline` is the period of time where the fee is valid, and the `signature` will be the signed payload from the Everclear fee signer which is used to confirm the inputs provided are valid.&#x20;

Domain IDs correspond to standard chain IDs and the protocol's supported list can be found in the [Supported Chains](../../resources/contracts/) section.

### Priority Settlement (Fast Path)

You can create intents that go through the priority settlement or fast path filling flow, that should settle you intent under two minutes max, to check weather the route you picked has fast path available you show first use the `/routes/quotes` , if fast is available the response payload will contain a new field called `fastPathQuote` which is the special fee being charged for the fast path. once confirmed, you can create an intent with a new query param `isFastPath: true` to the `POST /intents` endpoint.

## **Monitoring Intents**

The status of an intent can be fetched from the Everclear API's [`GET  /intents/{intentId}`](../api.md#get-intents-intentid) endpoint.&#x20;
