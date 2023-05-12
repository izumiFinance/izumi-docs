
.. _swap_query_x2y_with_online_data:

Swap Query x2y With Online Data
===============================

In this chapter, we will do swap query of x2y mode with online data from bsc-test chain.

The full example code of this chapter can be spotted `here <https://github.com/izumiFinance/izumi-iZiSwap-sdk/blob/main/example/swapQuery/preSwapX2YWithOnlineData.ts>`_.

1. some imports
-----------------------------------------------------------

first some base imports

.. code-block:: typescript
    :linenos:
    
    import {BaseChain, ChainId, initialChainTable} from 'iziswap-sdk/lib/base/types'
    import Web3 from 'web3';
    import { getLiquidities, getLimitOrders, getPointDelta, getPoolContract, getPoolState } from 'iziswap-sdk/lib/pool/funcs';
    import { getPoolAddress, getLiquidityManagerContract } from 'iziswap-sdk/lib/liquidityManager/view';
    import { Orders } from 'iziswap-sdk/lib/swapQuery/library/Orders'
    import { LogPowMath } from 'iziswap-sdk/lib/swapQuery/library/LogPowMath'
    import { iZiSwapPool } from 'iziswap-sdk/lib/swapQuery/iZiSwapPool'
    import { decimal2Amount, fetchToken } from 'iziswap-sdk/lib/base/token/token';
    import { SwapQuery } from 'iziswap-sdk/lib/swapQuery/library/State';
    import { SwapX2YModule } from 'iziswap-sdk/lib/swapQuery/swapX2YModule'
    import JSBI from 'jsbi';


2. some basic initialization
-----------------------------------------------------------

.. code-block:: typescript
    :linenos:

    const chain:BaseChain = initialChainTable[ChainId.BSCTestnet]
    // test net
    const rpc = 'https://data-seed-prebsc-1-s3.binance.org:8545/'
    console.log('rpc: ', rpc)
    
    const web3 = new Web3(new Web3.providers.HttpProvider(rpc))

we can see in the above code, we will connect to bsc-test net.

.. _basic_query_of_swap_query_x2y:

3. some basic query
-----------------------------------------------------------

.. code-block:: typescript
    :linenos:

    const liquidityManagerAddress = '0xDE02C26c46AC441951951C97c8462cD85b3A124c'
    const liquidityManagerContract = getLiquidityManagerContract(liquidityManagerAddress, web3)
    
    const iZiAddress = '0x551197e6350936976DfFB66B2c3bb15DDB723250'.toLowerCase()
    const BNBAddress = '0xae13d989daC2f0dEbFf460aC112a837C89BAa7cd'.toLowerCase()
    const iZi = await fetchToken(iZiAddress, chain, web3)
    const BNB = await fetchToken(BNBAddress, chain, web3)
    const fee = 2000 // 2000 means 0.2%
    const poolAddress = await getPoolAddress(liquidityManagerContract, iZi, BNB, fee)
    
    const tokenXAddress = iZiAddress < BNBAddress ? iZiAddress : BNBAddress
    const tokenYAddress = iZiAddress < BNBAddress ? BNBAddress : iZiAddress
    
    const poolContract = getPoolContract(poolAddress, web3)
    const state = await getPoolState(poolContract)
    const pointDelta = await getPointDelta(poolContract)

We can see in the above code, iZi is tokenX and BNB is tokenY in bsc test net.


4. fetch order data from poolContract
-----------------------------------------------------------

.. code-block:: typescript
    :linenos:

    const leftPoint = state.currentPoint - 5000
    const rightPoint = state.currentPoint + 5000
    const batchsize = 2000

    const liquidityData = await getLiquidities(poolContract, leftPoint, rightPoint, state.currentPoint, pointDelta, state.liquidity, batchsize)
    const limitOrderData = await getLimitOrders(poolContract, leftPoint, rightPoint, pointDelta, batchsize)

    const orders: Orders.Orders = {
        liquidity: liquidityData.liquidities,
        liquidityDeltaPoint: liquidityData.point,
        sellingX: limitOrderData.sellingX,
        sellingXPoint: limitOrderData.sellingXPoint,
        sellingY: limitOrderData.sellingY,
        sellingYPoint: limitOrderData.sellingYPoint
    }


The interface of `getLiquidities` can be viewed as following:

.. code-block:: typescript
    :linenos:

    export const getLiquidities = async (
        pool: Contract, 
        leftPoint: number, 
        rightPoint: number, 
        targetPoint: number, 
        pointDelta: number, 
        targetLiquidity: string, 
        batchSize: number
    ): Promise<{
        liquidities: JSBI[], 
        point: number[]
    }>

When we call `getLiquidities`, we should transfer a `targetPoint` with its liquidity value
`targetLiquidity` (usually `state.currentPoint` and `state.liquidity`).

5. form pool data
------------------

.. code-block:: typescript
    :linenos:

    const sqrtRate_96 = LogPowMath.getSqrtPrice(1)

    const swapQueryState: SwapQuery.State = {
        currentPoint: state.currentPoint,
        liquidity: JSBI.BigInt(state.liquidity),
        sqrtPrice_96: LogPowMath.getSqrtPrice(state.currentPoint),
        liquidityX: JSBI.BigInt(state.liquidityX)
    }

    const pool = new iZiSwapPool(swapQueryState, orders, sqrtRate_96, pointDelta, fee)

Here, state object is obtained :ref:`some basic query<basic_query_of_swap_query_x2y>`.

6. do swap query
-------------------------------------------------------------

.. code-block:: typescript
    :linenos:

    const lowPt = state.currentPoint - 1500;

    const tokenX = {
        address: '0x551197e6350936976DfFB66B2c3bb15DDB723250',
        decimal: 18
    } as TokenInfoFormatted
    const inputAmountStr = decimal2Amount(5, tokenX).toFixed(0)

    const {amountX, amountY} = SwapX2YModule.swapX2Y(pool, JSBI.BigInt(inputAmountStr), lowPt)
    
    console.log('cost: ', amountX.toString())
    console.log('acquire: ', amountY.toString())

Here, `lowPt` means lower bound of point or `undecimal_price_x_by_y` during swap.
When we call interface like `swapY2X` or `swapY2XDesireX`,
the parameter `highPt` means higher bound of point or `undecimal_price_x_by_y` during swap.

After we run codes above, amountX will store undecimal amount of tokenX during this swap and
amountY will store undecimal amount of tokenY during this swap.

**Notice**, when we use those order data to call `swapX2Y` or `swapX2YDesireY`, we should garrentee following non-equalities:

.. code-block:: typescript
    :linenos:

    max(sellingYPoint[0], liquidityDeltaPoint[0]) <= lowPoint 
    lowPoint <= currentPoint
    currentPoint <= liquidityDeltaPoint.last()

and if we want to call `swapY2X` or `swapY2XDesireX`, we should garrentee that.

.. code-block:: typescript
    :linenos:

    liquidityDeltaPoint[0] <= currentPoint
    currentPoint < highPoint 
    highPoint <= min(sellingXPoint.last(), liquidityDeltaPoint.last())

otherwise, an **iZiSwapError** with corresponding **errcode** and infomation will be throwed in this interface.

If you are sure your swap will not exceed following range by limit amount or desire parameter.

.. code-block:: typescript
    :linenos:

    [max(sellingYPoint[0], liquidityDeltaPoint[0]), min(sellingXPoint.last(), liquidityDeltaPoint.last())]
   
you can add some fake data with very left or right point as guards to order data.

**Notice** that, when we call `swapX2Y` and `swapX2YDesireY`, `amountX` is amount of tokenX actually payed.
When we call `swapY2X` and `swapY2XDesireX`, `amountY` is amount of tokenY actually payed.