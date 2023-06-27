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



Grouped Limit Order
------------------------------------






.. figure:: ../../_static/images/content/limit-order2.png
   :width: 800
   :align: center
   :alt: An example of concentrated liquidity.
   :name: figure-limit-order2


Key Technology
------------------------------------


