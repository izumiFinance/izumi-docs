Pool 
=============================



.. image:: ../../_static/images/content/pool.png
   :width: 700
   :align: center


A **POOL** in iZiSwap is distinguished with tuple *<tokenA, tokenB, fee>*, e.g., *<BNB, USDT, 2000>* means the pool with token *ETH* and *USDT* with fee rate *0.2%*, where *fee* means the transaction fee a trader will pay if he initiate a swap action. *tokenA* and *tokenB* are tokens that must 
implement the ERC-20 standard.

A pool is similar to a trading market in CEX. For example, if you want to trade BNB and USDT on Binance, you go to 
the BNB-USDT spot market, while here you interact with the *<BNB,USDT,2000>* pools.  Liquidity are added into it and organized in a certain structure, with the help of the following design.


Point
-------------------------------------------



Liquidity
-------------------------------------------


Pool Status
-------------------------------------------



