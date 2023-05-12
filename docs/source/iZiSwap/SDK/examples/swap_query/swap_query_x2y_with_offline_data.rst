
Swap Query x2y With Offline Data
=======================================

Example of **swap query** usage of x2y with offline data.

The full example code of this chapter can be spotted `here <https://github.com/izumiFinance/izumi-iZiSwap-sdk/blob/main/example/swapQuery/preSwapX2YWithOfflineData.ts>`_.

1. some imports
-----------------------------------------------------------

first some base imports

.. code-block:: typescript
    :linenos:

    import { TokenInfoFormatted } from 'iziswap-sdk/lib/base/types'
    import { Orders } from 'iziswap-sdk/lib/swapQuery/library/Orders'
    import { LogPowMath } from 'iziswap-sdk/lib/swapQuery/library/LogPowMath'
    import { iZiSwapPool } from 'iziswap-sdk/lib/swapQuery/iZiSwapPool'
    import { decimal2Amount } from 'iziswap-sdk/lib/base/token/token';
    import { SwapQuery } from 'iziswap-sdk/lib/swapQuery/library/State';
    import { SwapX2YModule } from 'iziswap-sdk/lib/swapQuery/swapX2YModule'
    import JSBI from 'jsbi';
    import { State } from 'iziswap-sdk/lib/pool/types';


2. construct order data
-----------------------

second, construct order data for swap, we should notice that the range of order data should
cover currentPoint and lowPoint (low boundary of calling `swapX2Y` or `swapX2YDesireY`)

.. code-block:: typescript
    :linenos:

    const liquidity = [
        '26728378980016527',
        '1918546082716318',
        '7295067906501307',
        '523230926682255801',
        '517854404858470812',
        '1918546082716318'
    ].map((e:string)=>JSBI.BigInt(e))

    const liquidityDeltaPoint = [ -98560, -93920, -93760, -93720, -93360, -93280 ]

    const sellingX = [] as JSBI[]
    const sellingXPoint = [] as number[]

    const sellingY = [ '0', '100000000000000000', '1200000000000000', '422993654872524085' ].map((e:string)=>JSBI.BigInt(e))
    const sellingYPoint = [ -98560, -94760, -93920, -93560 ]

    const orders: Orders.Orders = {
        liquidity,
        liquidityDeltaPoint,
        sellingX,
        sellingXPoint,
        sellingY,
        sellingYPoint,
    }

in the above code, `liquidityDeltaPoint` is an strict ascending array of points where liquidity changed cross that point from left to right.
and `liquidity[i]` is liquidity value at `liquidityDeltaPoint[i]`.
`sellingX` and `sellingXPoint` is data of limit orders which sell tokenX in this pool.
`sellingY` and `sellingYPoint` is data of limit orders which sell tokenY in this pool.
`sellingXPoint` and `sellingYPoint` should be strict ascending order.
if you want to know how to fetch those data from pool, we can see :ref:`this page <swap_query_x2y_with_online_data>`


3. construct pool data
-----------------------------------------------------------

.. code-block:: typescript
    :linenos:

    const sqrtRate_96 = LogPowMath.getSqrtPrice(1)

    const swapQueryState: SwapQuery.State = {
        currentPoint: -93548,
        liquidity: JSBI.BigInt('523230926682255801'),
        sqrtPrice_96: LogPowMath.getSqrtPrice(-93548),
        liquidityX: JSBI.BigInt('25090486294305875')
    }

    const fee = 2000
    const pool = new iZiSwapPool(swapQueryState, orders, sqrtRate_96, pointDelta, fee)
    

4. do swap query
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