.. _swap_mode:

Swap mode
=====================

In this section, we simply discuss 4 swap mode in core contracts.

Background
----------

iZiSwap consists of core contracts (`iZiSwapPool`, `iZiSwapFactory`) and periphery contracts (`LiquidityManager`, `LimitOrderManager`, `Swap`, `Quoter`).
In our sdk or frontend-app, we usually call interfaces of periphery contracts, and the periphery contracts will call corresponding core-contracts interfaces according to our params.

The most frequently used function calls are `swapAmount()` and `swapDesire()` in the `Swap` contract from periphery contracts set.
Here **desire** means whether the the amount of the output token is pre-determined.  For example, if you want to swap M tokenA - > N tokenB,
when M is pre-determined, `swapAmount()` should be invoked. If N is pre-determined, use `swapDesire()`.


When we call the `swapAmount()` interface in `Swap`,
the `Swap` contract will first split `swap path` acquired from parameter into several 
token pairs (etc, list of tuples like `(payedToken, acquiredToken, fee)`). The `path[0].payedToken` is user's payed token.
And `path[path.length-1].acquiredToken` is then token which user finally would buy.

On the contrary, when calling `swapDesire()`, user's payed token is not `path[0].payedToken`,
and is `path[path.length-1].payedToken` instead. (We will clarify the construction of `path` in the following sections.)

When `Swap` contract gets the path (list of tokenPair) parameter, it will traversal each token pair in order and call corresponding
swap interface of corresponding pool contract each to swap.

iZiSwapPool contract provides 4 types of swap interfaces for 4 modes.


Swap mode in core contracts
---------------------------

In the core contract, we implement 4 swap interface for 4 different modes,
which consists of `swapX2Y`, `swapX2YDesireY`, `swapY2X` and `swapY2XDesireX`.

each `token pair` has an `iZiSwapPool` contract if this pair has been established through iZiSwapFactory before, 
all options which involves `liquidity`, `limit order` and `swap` of this pair will be delivered from 
`periphery` to this `pool`.

Suppose the token pair is (`tokenA`, `tokenB`, `fee`), and we ignore `fee` here.

If the address of `tokenA` is smaller than `tokenB`, then in this pool, we call `tokenA` as `tokenX`
and call `tokenB` as `tokenY`, otherwise, we call `tokenA` as `tokenY` and `tokenB` as tokenX.

**swapX2Y**

when we call pool's `swapX2Y(...)` interface, it means that we want to pay some `tokenX` to get `tokenY` and the limit undecimal amount
of tokenX is specified as a parameter in this interface.


**swapX2YDesireY**

when we call pool's `swapX2YDesireY(...)` interface, it means that we want to pay some `tokenX` to get `tokenY` and the at least undecimal amount
of tokenY is specified as a parameter in this interface. 
In this interface, the amount is desired amount of tokenY and that's why we named this interface as `swapX2YDesireY`.

**swapY2X**

when we call pool's `swapY2X(...)` interface, it means that we want to pay some `tokenY` to get `tokenX` and the limit undecimal amount
of payed tokenY is specified as a parameter in this interface.

**swapY2XDesireX**

when we call pool's `swapY2XDesireX(...)` interface, it means that we want to pay some `tokenY` to get `tokenX` and the at least undecimal amount
of tokenX is specified as a parameter in this interface. 
In this interface, the at least amount is desired amount of tokenX and that's why we named this interface as `swapY2XDesireX`.

For instance, we are calling `Swap.swapAmount(...)` for paying certain amount of tokenA to get tokenB.

And we assume that the path contains no hop tokens, i.e. swap tokenA -> tokenB directly.
Then, the logic of `swapAmount()` can be simply viewed as following pseudo code.

.. code-block:: typescript
    :linenos:

    // we want to pay amountA (undecimal amount) of tokenA to get 
    // some tokenB
    const pool = getiZiSwapPoolByPair(tokenA.address, tokenB.address, fee)
    if (tokenA.address.toLowerCase() < tokenB.address.toLowerCase()) {
        // in the pool, tokenA is tokenX
        // tokenB is tokenY
        // because each pool only serves one pair, so we donot need to specify
        // tokenA or tokenB or fee when calling swap interface of pool
        pool.swapX2Y(amountA)
    } else {
        // in the pool, tokenA is tokenB
        // tokenB is tokenX
        // because each pool only serves one pair, so we donot need to specify
        // tokenA or tokenB or fee when calling swap interface of pool
        pool.swapY2X(amountA)
    }

