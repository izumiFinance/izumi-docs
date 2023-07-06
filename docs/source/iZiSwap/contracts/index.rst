Contracts
==================


iZiSwap contract adopts a conventional core and periphery layered design. 

From a logical perspective, the underlying liquidity management, limit order, and swap logic are all implemented in the core. The role of the periphery is mainly to provide high-level APIs and introduce management and abstraction of users' concepts. 

In terms of assets, tokens exist within the core contract, but the periphery has a certain degree of control over assets.


.. toctree::
   core
   periphery

   