# FRM Tips

1. [Basics](#basics)
* 1.1 [Order types](#orders--executions)

#### Basics
###### Orders & executions
Order types:
* limit (can be seen by the market) - execute at specific price (buy at 9k, sell at 11k)
* stop (also called stop market, can't be seen by the market, until stop price triggered) - execute order at market price when price would be stop price (buy once price would be 9k, actual buy price can be different from 9k)
* stop limit - (can't be seen by the market) - once stop price is reached, limit order would be put into orderbook. Useful when you want to create buy limit order with 11k. If you add just limit order with 11k, it would be executed immediately at 10k (cause market price is 10k). So you create stop order at 11k, with limit price - can be same as stop price like 11k, or you can add any other limit price, like 11.5k. So basically if you want buy limit with above market price - use stop limit order (simple limit won't work cause there is better market price & such limir order would be executed immediately). stop limit buy - if limit price above stop price, order would be executed for price between stop & limit price.
* stop loss - you create stop market order where based on stop price (can be fixed amount or percentile, like 5% of current price) market order executed to prevent loss. stop loss can't be stop limit - cause in case of stop limit there is a risk that order won't be executed at limit price (either price goes too deep, or it would be not enough liquidity for this limit price)
* take profit - execute order when price goes some percent & you made this profit. 10% take profit order - would be sold when stock price goes up 10%. You can use it with stop loss to manage risk exposure. 5:15 risk-to-reward ratio - put 2 orders, take-order - 15%, and stop-loss - 5%.
* trailing stop - lock in profit order, that would trigger opposite execution in case drop of certain percent. If you open 10% long position, than when price fall 10% from peak price, automatic sell would fire. Also peak price is constantly moving, so if you buy order for 100 with 10% trail, then at 90 would be sale, but if price goes to 200, then sale price would be re-adjusted to 180. Trailing stop more flexible than stop order, cuase sell price is always re-adjusted from max peak price.
* reduce-only (make sense only for contracts) - will reduce current position size or close position completely. If reduce order is increasing position size, such order would be rejected. So this type ensures that your position won't be increased unintentionally.

Time-in-force - how long order remains active before it get executed/expired (by setting time-in-force you don't need to worry about cancelling old orders):
* DAY (active during day) - cancelled if order is not executed during trading day (common for traditional exchanges)
* GTC (good till cancel) - order is active until executed or explicitly cancelled by trader
* FOK (fill or kill) - execute whole order or cancel it (no partial fills allowed)
* IOC (immediate or cancel) - execute immediately all or partial and cancel all unfilled quantity
* OPC, IOC, GTD and DTC
* post only, profit stop