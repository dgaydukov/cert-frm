# FRM Tips

1. [Basics](#basics)
* 1.1 [Order types](#order-types)
2. [Derivatives](#derivatives)
* 2.1 [Basic Concepts](#basic-concepts)
* 2.2 [Futures](#futures)
* 2.3 [Spreads](#spreads)
* 2.4 [Liquidation vs ADL](#liquidation-vs-adl)
* 2.5 [Options](#options)


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
###### Futures
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
###### Spreads
Spread - type of contract where underlying not real asset (like for futures/options) but difference between 2 assets or 2 derivatives
Calendar Spread - price difference of 2 futures with same underlying but different maturity date.
Let's say we have 2 futures one expired in September another in December, then:
* long - long `BTC/USDC/SEP` + short `BTC/USDC/DEC`
* short - short `BTC/USDC/SEP` + long `BTC/USDC/DEC`
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
Liquidation fund (liquidation reserve) - special fund that can pay to user for some time until ADL happen. 
when liquidation happen & liquidation order is matched in public orderbook below bunkruptcy price, the remaining goes to fund
this is the way fund is growing
###### Options
There are 2 types of options (Keep in mind that you can buy/sell any of these types):
* call option - right (but not obligation) to buy a stock at specified price (called strike price) before or at specified date
* put option - right (but not obligation) to sell a stock at specified price (called strike price) before or at specified date
Premium - price of the option contract, based option pricing model:
  Black-Scholes model - developed in 1973, one of the best way to estimate the price of option contract
  It designed for european options, cause it doesn't take into account that american options can be executed at any moment prior to expiration date
  It requires 5 input variables:
    * strike price of an option
    * current stock price
    * expiration time
    * risk-free rate
    * volatility
There are 3 concepts of option price:
* ATM (at-the-money) - option strike price is identical to the current market price of underlying asset
* ITM (in-the-money) - option with intrinsic value:
    * call - marketPrice=30, strikePrice=25, so 30-25=5 is the premium
    * put - marketPrice=30, strikePrice=35, so 5 - premium or intrinsic price of an option contract
* OTM (out-of-the-money) - option with only intrinsic value (opposite of ITM):
    * call - marketPrice=30, strikePrice=35, premium - 0.5. It make no sense to execute such option, cause what the point to buy stock at 35, when market price is 30
    yet since there is potential to grow, option itself has some intrinsic value of 0.5
    * put - marketPrice=30, strikePrice=25, premium - 0.5. Again there is no point to sell at 25, when you can sell at 30, 
    but still cause stock may fall, option cost some money
There are 2 styles of options:
* european style - option may be executed only on expiry date
* american style - option may be executed at any time prior to expiry date
There are several types of options:
* vanilla - standard options
* move - direction of price movement not relevant (this is relevant for vanilla), size of price movement matter
  So it tied to price volatility of underlying asset, so you can speculate on price volatility instead of direction
  price of move contract reflects expectation about price volatility. So if you beleive price will move up or down a lot you long position
  otherwise short. Same rules as with straddle if premium (how much you pay) is less then price movement - you win.
  You open long position (pay 100), current price - 1000, if tomorrow price 1200 - you make 100.
  For longs - loss can never exceed premium, so longs can never be liquidated
  Shorts - required margin & can be liquidated
* turbo - exotic option.
Option chain (option matrix) - list of all available option contracts (both put & call) for single underlying asset within given maturity period
Option straddle - strategy buy both call & put option with same strike price and expiration date. Profitable when stock rise/fall from strike price
  byt more then premium paid. Use it if you anticipate significant move in stock price, but not sure about direction.
  Let's say stock price is 50%, and call & put option cost 2.5. You buy both for 5 totally. Then 5/50 * 100% = 10%. So price movement should be at least 10% so you can make profit.
Option greeks - set of risks that can affect the price, named after greek letters:
* vega - impact of change in volatility
* theta - impact of change in time remaining
* delta - impact of change of underlying asset price
* gamma - range of change in delta
Minor greeks - in addition to main greeks, there is also not so common minor:
* rho - rate of change between option value and 1% change in interest rate
* lambda/epsilon/vomma/vera/speed/zomma/color/ultima.