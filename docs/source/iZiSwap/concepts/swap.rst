Swap
=============================

.. figure:: ../../_static/images/content/swap.png
   :width: 600
   :align: center
   :alt: An example of swap frontend.
   :name: figure-swap



|br|

**Swap** is the process of exchanging a certain amount of tokenX for a certain amount of tokenY. 

From the current price point, swaps will consume the liquidity of the other token depending on the direction of exchange. This liquidity includes that provided by limit orders and normal liquidity positions.
When the liquidity at a price point is exhausted, the procedure continues to consume the liquidity at the *next price point* until all the liquidity at each price point is depleted or the specified quantity of tokens has been exchanged.


*Here we abuse the term "next price point" a little bit since as mentioned in the previous section, the liquidity exists in intervals. In this situation, we can quickly determine whether the liquidity within each interval is sufficient.*


At each price point, the swap procedure follows the **Constant Sum Rule**, that is, 

.. math::
    L = x \sqrt{p} + y /\sqrt{p}  = (x + \Delta x) \sqrt{p} + (y + \Delta y) / \sqrt{p},


where :math:`L` is the liquidity on the price point, including normal range liquidity positions and the one-time limit order, :math:`\Delta` represents the variation of two tokens during the swap process.



.. |br| raw:: html

      <br>