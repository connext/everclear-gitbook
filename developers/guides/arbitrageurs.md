# Arbitrageurs

## **Process Overview**

Arbitrageurs are professional and individual actors monitoring invoices that have profitable discounts and providing liquidity when Rebalancers can not be netted against each other. By using Everclear, Arbitrageurs achieve attractive APRs on their portfolio whilst taking on low risk by purchasing discounted invoices on supported chains.

The process for Arbitrageurs is composed of these steps:

1. Monitor discounted invoices in the system to identify opportunities.
2. Calculate the amount necessary to settle the target invoice.
3. Request `newIntent` transaction data from the Everclear API on the domain that the discounted invoice is traveling to (i.e. calling `newIntent` on the destination domain specified by the intent being purchased).
4. Sign and submit the `newIntent` transaction data.

If the Arbitrageur is the first to purchase the intent on the destination domain they will receive the discounted amount as a reward.

For example:

* 100 USDT original intent sent from Optimism to Arbitrum
* Invoice discounted from 100 to 99
* Arbitraguer sends new intent of 99 USDT from Arbitrum to Optimism
* Original intent owner receives 99 USDT on Arbitrum
* Arbitraguer receives 100 USDT on Optimism

_// NOTE: These are the steps when an Arbitrageur successfully sends the first order to purchase an invoice; if the order is not first the invoice may already be purchased. In this case, the Arbitraguer’s intent would be matched with the next invoice in the queue but if there are no invoices in the queue or deposits to be netted with at processing time, the intent would be converted into an invoice and need to be purchased by a new intent._

The new intent sent by an Arbitrageur will incur the protocol fee. If the new intent is not matched and it finds itself waiting in the invoice queue for more than a whole epoch, it may begin to incur the auction discount. For information about matching, discounts, fees, and settlement visit Protocol Mechanics.

## **Monitoring Invoices**

The status of an intent can be fetched from the Everclear API which will update the intent on each domain as they are being processed. Arbitrageurs can identify intents that are being discounted (invoices) by querying the [`GET /invoices`](../api.md#get-invoices) endpoint:

This endpoint returns a paginated list of intents that have been converted into invoices and will grant rewards for Arbitrageurs.

See the full [Everclear API](../api.md) specification here.

An invoice’s `discount` is likely the most important factor in an Arbitrageur’s decision whether to purchase the invoice or not.

To optimize capital efficiency, use the [`GET /invoices/{intentId}/min-amounts`](../api.md#get-invoices-intentid-min-amounts) endpoint to calculate the minimum amount necessary to clear the target invoice. Minimum amounts are returned as an amount per domain corresponding to each of the potential destination domains that the invoice can be settled to. Arbitrageurs can choose which of these destinations domains to use as \*their\* origin chain when submitting a new intent to clear the invoice.

## Purchasing an intent

Arbitrageurs will request the `newIntent` transaction data from the Everclear API using the [`POST /intents`](../api.md#post-intents) endpoint. This transaction data should be signed and submitted onchain.

The transaction data from the API will correspond to a `newIntent` on the `FeeAdapter` contract and this should be called on the destination domain for the invoice being purchased. The netting system uses  intents from destination domains to match with existing invoices in the invoice queue on the Clearing chain i.e. creating a new intent moving USDT from Optimism to Arbitrum will purchase an existing invoice in the invoice queue moving USDT from Arbitrum to Optimism.

The `destinations` field is an array where multiple domains can be provided to allow the Arbitrageur to be settled on a preferred domain. When the invoice is purchased, the Arbitrageur’s `destinations` array will be iterated and the domain with the highest liquidity and lowest discount will be used for settlement. For example: if an invoice is traveling from Optimism to Arbitrum, the Arbitrageur’s new intent could be sent to the Spoke contract on Arbitrum with a destinations array containing multiple destinations such as \[Optimism, Polygon, BNB].

If the Arbitrageur provides multiple `destinations` and their deposit is not matched with an invoice, their invoice will be able to match with any of the provided domains when being processed. This may allow an Arbitraguer to increase the speed at which their deposit is settled and minimise the likelihood a discount will be applied to their intent as there are multiple domains for the intent to be settled on.

The `maxFee` field should **always be specified as 0** as maxFee is only applicable in cases where an order should be routed to the solver pathway and does not apply to the netting pathway.

The `ttl` input should **always be specified as 0** to indicate the order should be routed via the netting system on the Hub. When `ttl` is non-zero an order is routed via a separate solver pathway where the intent creator requires a dedicated solver to fill an intent for a fee. This pathway will not be supported at launch; Arbitrageurs must ensure all netting order fills **always specify `ttl` as 0.**

The `feeParams` consists of `fee`, `deadline`, and `signature` which would be generated through interacting with the API. The `fee` will be the amount being charged to the intent creator (levied against the input amount), the `deadline` is the period of time where the fee is valid, and the `signature` will be the signed payload from the Everclear fee signer which is used to confirm the provided inputs are valid.&#x20;

Domain IDs correspond to standard chain IDs and the protocol's supported list can be found in the [Supported Chains](../../resources/contracts/) section.

## **Monitoring Intents**

The status of an intent can be fetched from the Everclear API's [`GET  /intents/{intentId}`](../api.md#get-intents-intentid) endpoint.&#x20;

## Risk Mitigation

The main risk for an Arbitrageur will be sending a transaction to purchase a discounted invoice and not being the first in the queue. This will mean another Arbitrageur has purchased the discounted invoice and can lead to costs being incurred without receiving any rewards.
