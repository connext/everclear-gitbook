# Getting Started

## Everclear Overview

Everclear (prev Connext) is the first Clearing Layer.

More protocols such as UniswapX, 1Inch, CoWSwap, and Across are adopting intent-based order systems where users provide an intent and the protocol’s network of solvers compete to fill the request. This approach can provide optimal fulfillments and is becoming widely popular as more protocols aim to support account abstraction. However, once solvers are reimbursed for filling these orders, their funds are spread across chains and need to be moved back to the chain(s) they aim to provide their services from.

This is called `rebalancing` and for solvers it’s costly and complex to rebalance funds back to the chains they want to operate from. To do this today, solvers uses CEXs and bridges but close to 80% of `rebalancing` activity could be netted against each other. This would lead to a 10x cost reduction, simplify the process, and create opportunities to programmatically rebalance. For users, it would make intent fulfilment cheaper and enable liquidity to reach chains with ease.

We believe the future of on-chain activity will be intent based which will require solver networks to grow exponentially. Combined with the explosion of layer 2 and 3 chains, the biggest blocker to success of our ecosystems will be liquidity. By solving the `rebalancing` problem, we believe the potential of the multi-chain future can be unlocked.

#### How does Everclear work?

Rebalancers (solvers) deposit funds into a Spoke contract on any supported chain and specify the destination chain(s) they'd like to receive them on. As Rebalancers submit their intents, the Hub contract deployed on the Clearing chain will net their intents against others moving in the opposite direction.

When there are no Rebalancers to net the intent against, the Hub will discount the intent by a configured BPS per epoch. The intent will be discounted (up to a max threshold) until it is purchased by a new intent and the owner of the new intent will receive the discount as a bonus. For example:

* Origin intent of $100 discounted to $99 (moving from Arb to Op)
* New intent of $99 received (moving from Op to Arb)
* Origin intent owner receives $99 on Op
* New intent owner receives $100 on Arb

The discount approach is similar to a dutch auction and has been implemented to encourage Arbitrageurs to support the system. Arbitrageurs will monitor new intents on supported chains and purchase them once the discount applied reaches a profitable rate - competition between Arbitraguers should lead to optimal pricing for Rebalancers.

## Developing

Dedicated integration guides have been provided in the [Guides](guides/) section for Rebalancers and Arbitrageurs. The [Everclear API](api.md)'s full OpenAPI specification is also available to reference. Finally, deployments for contracts and subgraphs can be found in [Resources](broken-reference).
