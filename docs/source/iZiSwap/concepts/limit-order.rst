Limit Order
=============================



.. figure:: ../../_static/images/content/limit-order1.png
   :width: 300
   :align: center
   :alt: An example of concentrated liquidity.
   :name: figure-limit-order1

   Illustration of Liquidity conditioned combined with normal positions and limit orders.  




**Limit Order** refers to providing liquidity at a specified price point as a counterparty, and the transaction is executed when the price reaches the specified price point and a trader try to take a swap action. 
Unlike normal liquidity positions, the liquidity provided by limit orders is one-time only and will not be repeatedly traded. 

For example, for the ETH/USDC/0.2% pool, the current price is 1 ETH = 2000 USDC. 
A user placed a sell limit order of 1 ETH at a price of 2500 USDC. When the price reaches or crosses 2500, this limit order will be filled. 
However, if the price drops again to, for example, 2000 USDC, the token in this limit order still exists as 2500 USDC instead of 1 ETH.

The core principle behind iZiSwap's design of limit orders is **fully non-custodial**, meaning that users' assets and trading activities are entirely carried out on the contract chain without relying on third-party relays or fund custody.

From the perspective of swap, there is no difference in liquidity provided by both limit orders and normal liquidity positions. They are processed uniformly. 
However, from an implementation perspective, we need new algorithms because on-chain operations require low time and space complexity. In traditional CEX matching engines, 
the execution of limit orders requires traversal of order placers in chronological order. However, this is not acceptable on the blockchain. We propose the following two key designs for this purpose.



Grouped Limit Order
------------------------------------
Group the limit orders of different users at the same point into a grouped value, which is the single target object in the swap process. 
The status updates of different users are lazy, meaning that their own parts will only be processed from this grouped value when they update their own limit orders.


*Limitation: The time complexity of traversal did not disappear, but shifted from the swap operation to the limit order traders. Unlike traditional CEX models, limit order providers need to claim tokens traded on their own.*


.. figure:: ../../_static/images/content/limit-order2.png
   :width: 800
   :align: center
   :alt: An example of concentrated liquidity.
   :name: figure-limit-order2


Legacy Design
------------------------------------
The most crucial aspect of a limit order is to ensure chronological correctness. 
That is, if the price passes through the target price, then regardless of subsequent price fluctuations, the portion that has been traded is locked in and can be claimed at any time.

As a result, iZiSwap proposes a legacy locking mechanism as shown in the above figure. When the price crosses the target price, that is, when all limit orders on the target price are filled, 
any remaining but unclaimed portion of the order will be moved to the legacyEarn space and recorded with a timestamp. 
Due to the monotonicity of time, any user who placed an order before this time can retrieve their executed portion from the legacy space.


*Limitation: The locking mechanism of Legacy is designed for the situation where the price passes through and all trades are completed. 
For partially completed transactions, iZiSwap adheres to the principle of fairness and adopts a first-come, first-served claim method.*



