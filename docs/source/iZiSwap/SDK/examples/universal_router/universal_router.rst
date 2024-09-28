

Universal Quoter and Swap
----------------------------

Origin examples for **Swap** and **Quoter** can only deal with paths without V2 pool.

To help you deal with paths which may contain both 
V2 pool(or we say iZi Classic Pool) and V3 pool, 
we newly implement **UniversalQuoter** and **UniversalSwapRouter** contracts.

In this section, we provide examples of using the iZiSwap SDK to implement price inquiry (through **UniversalQuoter** contract) and swap (through **UniversalSwapRouter** contract).
These are two of the most common and frequent operations in DEX.

Price inquiry means pre-querying amount of token acquired or token needed to pay. For example, if you want to swap 1 ETH to USDC, you can use the Quoter contract to get how much USDC 
can you get based on the current liquidity conditions.

Swap means invoking real trading with given exact paying amount or exact acquiring amount. 

Based on exact paying amount (amount mode) or exact acquiring amount (desired mode), they are processed by different interfaces. For example, if you want to swap M ETH to N USDC, 
if M is pre-determined, and N is acquired by the UniversalQuoter contract, the amount mode is invoked. Otherwise, if N is pre-determined and M is acquired by the UniversalQuoter contract, this is the desired mode situation.

**UniversalQuoter** and **UniversalSwap** are 2 different contracts and the deployed contracts can be found in the corresponding section.


.. toctree::
   :maxdepth: 1
   
   universal_quoter_swap_chain_with_exact_input
   universal_quoter_swap_chain_with_exact_output
