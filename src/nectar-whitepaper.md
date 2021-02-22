# [nectar.fund](https://nectar.fund)

<img src="./images/nectar.png" alt="nectar.fund">

## What is Nectar?

On-chain index pools of your favourite Binance Smart Chain tokens. 
 - Invest in many tokens at once in a fully decentralised manner. 
 - Track indices of other investors and replicate their gains. 

nectar is managed by the holders of its governance token [NCTR](https://github.com/nectar-fund/contracts), which is used to vote on proposals for protocol updates and high level index management such as the definition of market sectors and the creation of new management strategies. 

## Nectar Pools
The first product developed by nectar is a set of capitalization-weighted index pools designed to replicate the behavior of index funds, which historically [have returned better and more consistent returns](https://www.cnbc.com/2019/03/15/active-fund-managers-trail-the-sp-500-for-the-ninth-year-in-a-row-in-triumph-for-indexing.html) than actively managed funds on the stock market.

Nectar pools simplify asset management on Binance Smart Chain the way that index funds do for the stock market: by creating a single asset which represents ownership in a diverse portfolio that tracks the market sector the index represents. Each index pool has an BEP-20 index token which anyone can mint by providing the underlying assets in the pool, burn to claim the underlying assets, or swap with exchanges to easily manage their exposure to specific markets.

Nectar pools regularly rebalance their underlying assets in order to better represent the market sectors they track. Portfolio targets are set using on-chain data from Pancake Swap. As with index funds, the only roles for humans in managing index pools are the initial determination of weighting and asset selection rules, the definition of market sectors and the classification of assets into those sectors. These roles are carried out by NCTR governance, which has mandatory time-locks for all governance decisions.

### Summary

Nectar pools are tokenized portfolios that double as AMMs. The pool contract is designed to be able to radically change the composition of its portfolio without needing to access external liquidity.

The Nectar Pool contract is a fork of the [Balancer Pool](https://github.com/balancer-labs/balancer-core/blob/master/contracts/BPool.sol). The primary changes made to the contract were to enable more dynamic pool management so that assets can be bound, rebound and reweighed gradually and without the need to access external liquidity.

It may be useful to read the Balancer [Whitepaper](https://balancer.finance/whitepaper/) or [documentation](https://docs.balancer.finance/) for additional context on the pool contract.

### Rebalancing

The typical method for rebalancing a token portfolio is to sell and purchase sufficient amounts of each asset to reach the desired weights. This typically involves trading with on-chain exchanges or using an auction system. Any method of swapping on-chain to rebalance will cause some amount of loss for the pool, potentially quite a lot. On-chain exchanges are illiquid, and auctions on Ethereum [have a history of being exploited](https://forum.makerdao.com/t/black-thursday-response-thread/1433).

Nectar pools rebalance themselves over time by creating small arbitrage opportunities that incentivize traders to gradually adjust token balances and weights. As tokens are swapped, their weights move slightly toward the targets set by the pool controller. This weight adjustment occurs at a maximum of once per hour in order to create small arbitrage opportunities over time that eventually bring the portfolio composition in line with the targets.

While this rebalancing process is not instantaneous, it is permissionless, it works for arbitrarily large pools, it is generally more gas efficient and it does not assume that the pool or its controller can access external liquidity to execute rebalances.

### Limitations

**Abnormal token implementations**

Tokens that have internal transfer fees or other non-standard balance updates may create arbitrage opportunities. For now, these tokens should not be used in nectar pools. Nectar pools do not have the same ERC20 restrictions on return values as Balancer pools, as the pool contract uses methods from OpenZeppelin's `SafeERC20` library.

**Permanent loss for some liquidity providers due to unbound token handling**

While the selling of a pool's unbound tokens is restricted to a small range around their moving average prices, it is still possible for a liquidity provider to experience permanent loss due to the way that unbound tokens are handled. If an LP exits a pool after a token is removed from the pool, but before its balance is swapped to the other underlying assets, the LP will suffer a loss of around 1% of their pool tokens' value (as 1% is the minimum weight of the pool).

**Swap input amount**

When a token is sold to a pool, the input amount can not exceed half of the pool's current balance in that token. This restriction applies to swaps and single-asset liquidity providing functions, but does not apply to all-asset liquidity providing. This only applies to an individual call to the contract, and can be bypassed with multiple calls.

**Swap output amount**

When a token is purchased from a pool, the output amount can not exceed one third of the pool's current balance in that token. This restriction applies to swaps and single-asset liquidity removal functions, but does not apply to all-asset liquidity removal. This only applies to an individual call to the contract, and can be bypassed with multiple calls.

**Minimum balance**

When a pool is first initialized, it must have a balance of at least 1e6 wei. This does not apply to tokens after the pool is initialized