# FRM Tips

1. [Basics](#basics)
* 1.1 [Order types](#order-types)
2. [Derivatives](#derivatives)
* 2.1 [Basic Concepts](#basic-concepts)
* 2.2 [Future Contract](#future-contract)
* 2.3 [Spread Contract](#spread-contract)
* 2.4 [Liquidation vs ADL](#liquidation-vs-adl)
* 2.5 [Option: put vs call](#option-put-vs-call)


#### Basics
###### Order types
Order types:
* limit (can be seen by the market) - execute at specific price (buy at 9k, sell at 11k)
* stop (also called stop market, can't be seen by the market, until stop price triggered) - execute order at market price when price would be stop price (buy once price would be 9k, actual buy price can be different from 9k)
* stop limit - (can't be seen by the market) - once stop price is reached, limit order would be put into orderbook. Useful when you want to create buy limit order with 11k. If you add just limit order with 11k, it would be executed immediately at 10k (cause market price is 10k). So you create stop order at 11k, with limit price - can be same as stop price like 11k, or you can add any other limit price, like 11.5k. So basically if you want buy limit with above market price - use stop limit order (simple limit won't work cause there is better market price & such limir order would be executed immediately). stop limit buy - if limit price above stop price, order would be executed for price between stop & limit price.
* stop loss - you create stop market order where based on stop price (can be fixed amount or percentile, like 5% of current price) market order executed to prevent loss. stop loss can't be stop limit - cause in case of stop limit there is a risk that order won't be executed at limit price (either price goes too deep, or it would be not enough liquidity for this limit price)
* take profit (also called bracket order) - execute order when price goes some percent & you made this profit. 10% take profit order - would be sold when stock price goes up 10%. You can use it with stop loss to manage risk exposure. 5:15 risk-to-reward ratio - put 2 orders, take-order - 15%, and stop-loss - 5%.
    * bracketed buy order - 3 orders in one: buy order + sell limit + sell stop (buy btc for 10k, create sell limit for 15k, and sell stop for 9k). In such case you lock-in profit & limit your loss. Keep in mind that sell stop - stop-loss order, which turned into market order, so actual sell price may be even lower then stop price.
    * bracketed sell order - short sell order with 3 orders: (sell order + buy limit + stop buy). You sell btc for 10k (current price), add buy limit for 8k, and stop buy for 12k. Again this type of order help to cover some caveat of short selling
* trailing stop - lock in profit order, that would trigger opposite execution in case drop of certain percent. If you open 10% long position, than when price fall 10% from peak price, automatic sell would fire. Also peak price is constantly moving, so if you buy order for 100 with 10% trail, then at 90 would be sale, but if price goes to 200, then sale price would be re-adjusted to 180. Trailing stop more flexible than stop order, cuase sell price is always re-adjusted from max peak price.
* reduce-only (make sense only for contracts) - will reduce current position size or close position completely. If reduce order is increasing position size, such order would be rejected. So this type ensures that your position won't be increased unintentionally.

Time-in-force - how long order remains active before it get executed/expired (by setting time-in-force you don't need to worry about cancelling old orders):
* DAY (active during day) - cancelled if order is not executed during trading day (common for traditional exchanges)
* GTC (good till cancel) - order is active until executed or explicitly cancelled by trader
* FOK (fill or kill) - execute whole order or cancel it (no partial fills allowed)
* IOC (immediate or cancel) - execute immediately all or partial and cancel all unfilled quantity
* Post Only (only for limit order) - order accepted only if it not executed immediately, otherwise order would be rejected. So if you quote buy limit with post only with limit price below market - it won't execute immediately, but would be rejected.
* MOO/MOC (market-on-open, market-on-close) - for traditional stock, where market open/closed daily. Trader submit order when market opens or close.

#### Derivatives
###### Basic Concepts
* Derivative - financial contract between 2 parties that derive it price from underlying asset (btc derivative - will derive price from spot bitcoin price)
* Future - type derivative - agreement to buy/sell asset at predetermined future date & price
* Basis (funding rate) - difference in price between spot & future market
* Funding - series of continues payments between longs & shorts, tethers perpetual price to spot price
###### Future Contract
There are 3 types of future contract:
* dated future (vanilla future) - have pre-defined maturity date on which they would be executed, settled in quote currency
* inverse dated future - same as dated futures, but calculated at inverse
  p&l calculation:
    * dated future      => n * m * (exitPrice - entryPrice)
    * inverse future    => n * m * (1/entryPrice - 1/exitPrice)
  Don't confuse it with `inverse futures contract` - which settled in base currency
* perpetual future (perpetuals) - future contract without maturity date. Every 8 hours basis calculated & either one (long or short) payed to counterparty:
    * perpPrice > spotPrice (contract trades at `premium` to spot) - funding is positive (longs pay to shorts)
    * perpPrice < spotPrice (contract trades at `discount` to spot) - funding is negative (shorts pay to longs)
###### Spread Contract
###### Liquidation vs ADL
Don't confuse these 2:
  * liquidation - attempt to re-sell you position (either long or short) to someone else
  * ADL (Auto de-leveraging) - closing of your position and the opposite position of counter-party
Exchange - is nothing more but a facilitator between 2 parties. One opens short position and another long.
For each number of open short position - equal number of long position should be kept.
If user1 open long position and user2 open short position, and they were matched, they both have a contract.
If price goes up, that means user1 will earn money from user2. If user2 runs out of money, his position would be liquidated.
But user1 still has his position open. So we need urgently someone else to take short position that was kept by user2 before liquidation.
If nobody would like to take it that means, nobody will pay to user1, in case price still going up. So what should we do.
Unfortunately exchanges has nothing better, rather then just close 2 position. short - cause nobody taking it, and long - cause there is no short counterpaty.
Basically what happens is that although user1 want to keep his position open and make money, exchange will forcefully close his position.
Liquidation goes into 3 steps:
  * liquidation orderbook - first order get into this special orderbook for liquidated positions
  * publis orderbook - if nobody takes order from first orderbook, then exchange will try to execute order in public orderbook
  * liqudation reserve - special reserve that takes order that nobody want to take. This reserve:
      * take position if nobody else want and pay to user
      * take maintenanceMargin from liquidated user
So once liquidation reserve is depleted and nobody can pay to user, then ADL would happen:
    * liquidation reserve will generate new ADL order and submit it to public orderbook
    * this order would be automatically executed and decrease user position
    * exchanges use ADL order rather then just change user position to have full trailing
###### Option: put vs call
