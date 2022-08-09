mint
================================

here, we provide a simple example for creating a new liquidity


1. some imports
---------------

.. code-block:: typescript
    :linenos:

    import {BaseChain, ChainId, initialChainTable, PriceRoundingType} from 'iziswap-sdk/lib/base/types'
    import {privateKey} from '../../.secret'
    import Web3 from 'web3';
    import { getPointDelta, getPoolContract, getPoolState } from 'iziswap-sdk/lib/pool/funcs';
    import { getPoolAddress, getLiquidityManagerContract } from 'iziswap-sdk/lib/liquidityManager/view';
    import { amount2Decimal, fetchToken } from 'iziswap-sdk/lib/base/token/token';
    import { pointDeltaRoundingDown, pointDeltaRoundingUp, priceDecimal2Point } from 'iziswap-sdk/lib/base/price';
    import { BigNumber } from 'bignumber.js'
    import { calciZiLiquidityAmountDesired } from 'iziswap-sdk/lib/liquidityManager/calc';
    import { getMintCall } from 'iziswap-sdk/lib/liquidityManager/liquidity';

the detail of these imports can be viewed in following content

.. _base_obj_mint:

2. specify which chain, rpc url, web3, and account
--------------------------------------------------

.. code-block:: typescript
    :linenos:

    const chain:BaseChain = initialChainTable[ChainId.BSC]
    const rpc = 'https://bsc-dataseed2.defibit.io/'
    console.log('rpc: ', rpc)
    const web3 = new Web3(new Web3.providers.HttpProvider(rpc))
    const account =  web3.eth.accounts.privateKeyToAccount(privateKey)
    console.log('address: ', account.address)

here

**BaseChain** is a data structure to describe a chain, in this example we use **bsc** chain.

**ChainId** is an enum to describe **chain id**, value of the enum is equal to value of **chain id**

**initialChainTable** is a mapping from some most used **ChainId** to **BaseChain**, of course you can fill fields of BaseChain by yourself

**privateKey** is a string, which is your private key, and should be configured by your self

**web3** package is a public package to interact with block chain

**rpc** is the rpc url on the chain you specified

.. _LiquidityManagerContract_forMint:

3. get web3.eth.Contract object of liquidityManager
---------------------------------------------------

.. code-block:: typescript
    :linenos:

    const liquidityManagerAddress = '0x93C22Fbeff4448F2fb6e432579b0638838Ff9581'
    const liquidityManagerContract = getLiquidityManagerContract(liquidityManagerAddress, web3)
    console.log('liquidity manager address: ', liquidityManagerAddress)

here, **getLiquidityManagerContract** is an api provided by our sdk, which returns a **web3.eth.Contract** object of **LiquidityManager**

4. fetch 2 erc20-tokens' infomations
---------------------------------------------------------

.. code-block:: typescript
    :linenos:

    const testAAddress = '0xCFD8A067e1fa03474e79Be646c5f6b6A27847399'
    const testBAddress = '0xAD1F11FBB288Cd13819cCB9397E59FAAB4Cdc16F'

    const testA = await fetchToken(testAAddress, chain, web3)
    const testB = await fetchToken(testBAddress, chain, web3)

**fetchToken()** returns a **TokenInfoFormatted** obj of that token, which containing following fields,

of course you can fill TokenInfoFormatted by yourself if you have known each field correctly of the erc20-token
the TokenInfoFormatted fields used in sdk currently are only **symbol**, **address**, and **decimal**.

.. code-block:: typescript
    :linenos:

    export interface TokenInfoFormatted {
        // chain id of chain
        chainId: number;
        // name of token
        name: string;
        // symbol of token
        symbol: string;
        // img url, not necessary for sdk, you can fill any string or undefined
        icon: string;
        // address of token
        address: string;
        // decimal value of token, acquired by calling 'decimals()'
        decimal: number;
        // not necessary for sdk, you can fill any date or undefined
        addTime?: Date;
        // not necessary for sdk, you can fill either true/false/undefined
        custom: boolean;
        // this field usually undefined.
        // wrap token address of this token if this token has transfer fee.
        // this field only has meaning when you want to use sdk of box to deal with problem of transfer fee
        wrapTokenAddress?: string;
    }

notice that, usually we set **TokenInfoFormatted.wrapTokenAddress** as undefined.

following paragraph corresponding to box and wrap token you can just **skip** it if you do not consider token with transfer fee.

only if we want to use **box** and the token has transfer fee, we should set the **wrapTokenAddress** field.
if we donot want to use **box** or the token has no transfer fee, **TokenInfoFormatted.wrapTokenAddress** should be undefined.
:ref:`box<box>` is designed to deal with problem of erc20 token with ":ref:`transfer fee<transfer_fee>`".
there is a problem that in iZiSwap we can not mint or trade or add limit order with tokens which have transfer fee.
to deal with this problem, we can deploy a :ref:`Wrap Token<wrap_token>` which can be transformed from origin erc20 token.
wrap token has no transfer fee, transfer fee only charged when user transform origin token to wrap token or wrap token to origin token.
and we can mint or add limit order or trade with such wrap tokens instead of origin token in iZiSwap.
for sdk of box, see :ref:`here<box>` for more infomation.


5. get state of corresponding swap pool
---------------------------------------------------------

first get pool address of token pair (testA, testB, fee)

.. code-block:: typescript
    :linenos:

    const poolAddress = await getPoolAddress(liquidityManagerContract, testA, testB, fee)

function **getPoolAddress(...)** queries **liquidityManagerContract** to get iZiSwap pool address of token pair **(testA, testB, fee)**


.. code-block:: typescript

   - liquidityManagerContract: liquidity manager contract, acquired in '4. get web3.eth.Contract object of liquidityManager'
   
   - testA: an erc20 token in type of TokenInfoFormatted, acquired in '5. fetch 2 erc20-tokens' infomations'
   
   - testB: another erc20 token in type of TokenInfoFormatted, also acquired in '5. fetch 2 erc20-tokens' infomations'
   
   - fee: an int number, fee/1e6 is fee rate of pool, etc, 2000 means 0.2% fee rate
  
after acquire **poolAddress**, calling **getPoolContract(...)** to get pool contract object

.. code-block:: typescript
    :linenos:

    const pool = getPoolContract(poolAddress, web3)

thirdly, query state of pool

.. code-block:: typescript
    :linenos:

    const state = await getPoolState(pool)

state is a **State** obj which extends from **BaseState**

only fields in **BaseState** are used in this example


.. code-block:: typescript
    :linenos:

    export interface BaseState {
        // current point on the pool, see document in concepts(price/decimalPrice/undecimalPrice/point)
        // ranging from (-800000, 800000)
        currentPoint: number,
        // liquidity value on currentPoint, a decimal system format string
        liquidity: string,
        // value of liquidity of tokenX on currentPoint, a decimal system format string
        // liquidityY = liquidity - liquidityX
        liquidityX: string
    }

to compute undecimal-amount of token in minting, we will take use of **state.currentPoint**

6.  compute boundary point of liquidity on the pool
---------------------------------------------------------

boundary point is **leftPoint** and **rightPoint** of liquidity, according to :ref:`price` , we know that **point** on the pool and **decimal price** can be transformed from each other

first we determine the minimal and maximum **decimal price** of our liquidity ready to mint

assume the desired minimal **decimal price** of **A_by_B** is **0.099870** (this decimal price means 0.099870 testB to buy 1.0 testA, here, number 0.099870 and 1.0 are both **decimal amount**).
assume the max **decimal price** of  `A_by_B` is `0.29881`

then, we can get 2 **point**s on the pool of min and max **decimal prices** though following code

.. code-block:: typescript
    :linenos:

    const point1 = priceDecimal2Point(testA, testB, 0.099870, PriceRoundingType.PRICE_ROUNDING_NEAREST)
    const point2 = priceDecimal2Point(testA, testB, 0.29881, PriceRoundingType.PRICE_ROUNDING_NEAREST)

**priceDecimal2Point(...)** is a function to transform **decimal price** to the **point** on the pool, the function has following params

.. code-block:: typescript

    /**
     * @param tokenA: TokenInfoFormatted, one erc20 token of pool
     * @param tokenB: TokenInfoFormatted, another erc20 token of pool
     * @param priceDecimalAByB: number,  decimal price of A_by_B (A_by_B means how much tokenB to buy 1 tokenA)
     * @param roundingType: PriceRoundingType, rounding type when transform price to point
     * @return point: number, point on the pool transformed from decimal price
     */
    priceDecimal2Point(tokenA, tokenB, priceDecimalAByB, roundingType)

because we do not ensure that tokenA's address is smaller than tokenB

so here point1 may be larger than point2, and we could not simply specify leftPoint as point1 and rightPoint as point2

instead we take min(point1, point2) as leftPoint and max(point1, point2) as rightPoint

.. code-block:: typescript
    :linenos:

    let leftPoint = Math.min(point1, point2)
    let rightPoint = Math.max(point1, point2)

also, when we mint, the boundary point of liquidity must be times of `pointDelta`

so we should rounding `leftPoint` and `rightPoint` to times of `pointDelta` throw following codes

.. code-block:: typescript
    :linenos:

    const pointDelta = await getPointDelta(pool)
    
    leftPoint = pointDeltaRoundingDown(leftPoint, pointDelta)
    rightPoint = pointDeltaRoundingUp(rightPoint, pointDelta)

in the above codes, pointDelta is a number value queried from pool contract

for fee rate of 0.2%, pointDelta usually equals to **40**

besides, about **leftPoint** and **rightPoint** we must garrentee following inequality

.. code-block:: typescript

    leftPoint >= pool.leftMostPt()
    rightPoint <= pool.rightMostPt()
    rightPoint - leftPoint < 400000

7. specify or compute tokenA's and tokenB's max undecimal amount in this mint (optional)
----------------------------------------------------------------------------------------

sometimes, our app's user wants to know the amount of tokenA when he fill amount of tokenB or amount of tokenB when he fill tokenA.

so, we provide a function named `calciZiLiquidityAmountDesired()` in sdk to do this calculation

suppose we want to specify max decimal amount of tokenA ( token named testA) is 100

.. code-block:: typescript
    :linenos:

    const maxTestA = new BigNumber(100).times(10 ** testA.decimal)

and we can compute corresponding undecimal amount of tokenB ( token named testB)

.. code-block:: typescript
    :linenos:

    const maxTestB = calciZiLiquidityAmountDesired(
        leftPoint, rightPoint, state.currentPoint,
        maxTestA, true, testA, testB
    )

here, `calciZiLiquidityAmountDesired(...)` is a function provided by sdk,
which is used for computing one erc20-token's undecimal amount of a liquidity after
given `leftPoint` `rightPoint` `currentPoint` and  the other erc20-token's undecimal amount

the params are following:

.. code-block:: typescript

   /**
    * @param leftPoint: number, left point of the liquidity
    * @param rightPoint: number, right point of the liquidity
    * @param currentPoint: number, current point on the swap pool
    * @param amount: BigNumber, undecimal amount of one token
    * @param amountIsTokenA: boolean, true for amount is tokenA's undecimal amount, false for tokenB
    * @param tokenA: TokenInfoFormatted, tokenA infomation
    * @param tokenB: TokenInfoFormatted, tokenB infomation
    */
   calciZiLiquidityAmountDesired(leftPoint, rightPoint, currentPoint, amount, amountIsTokenA, tokenA, tokenB):


here, after we calling `calciZiLiquidityAmountDesired`,
we get a `BigNumber` stored in `maxTestB`,
which is corresponding undecimal amount of tokenB ( token named testB)

.. _liquidity_manager_mint_calling:

8. get mint calling
-------------------

first, construct necessary params and gasPrice for mint calling

.. code-block:: typescript
    :linenos:

    const mintParams = {
        tokenA: testA,
        tokenB: testB,
        fee,
        leftPoint,
        rightPoint,
        maxAmountA: maxTestA.toFixed(0),
        maxAmountB: maxTestB.toFixed(0),
        minAmountA: maxTestA.times(0.985).toFixed(0),
        minAmountB: maxTestB.times(0.985).toFixed(0),
    }

    const gasPrice = '5000000000'

then, get mint calling

.. code-block:: typescript
    :linenos:

    const { mintCalling, options } = getMintCall(
        liquidityManagerContract,
        account.address,
        chain,
        mintParams,
        gasPrice
    )

mintParams is type of MintParam, **maxAmountA**, **maxAmountB**, **minAmountA**, **minAmountB**
is required min-max undecimal amount of tokenA and tokenB deposited in this mint

of course, you can fill **maxAmountA**, **maxAmountB**, **minAmountA**, **minAmountB** to arbitrary value as you want

function **getMintCall** returns 2 object, **mintCalling** and **options**

after get **mintCalling** and **options**, we can estimate gas for mint

9. estimate gas (optional)
---------------------------
of course you can skip this step if you donot want to limit gas

.. code-block:: typescript
    :linenos:

    const gasLimit = await mintCalling.estimateGas(options)
    console.log('gas limit: ', gasLimit)

10. finally, send transaction!
------------------------------

for metamask or other explorer's wallet provider, you can easily write 

.. code-block:: typescript
    :linenos:

    await mintCalling.send({...options, gas: gasLimit})

otherwise, if you are runing codes in console, you could use following code

.. code-block:: typescript
    :linenos:

    // sign transaction
    const signedTx = await web3.eth.accounts.signTransaction(
        {
            ...options,
            to: liquidityManagerAddress,
            data: mintCalling.encodeABI(),
            gas: new BigNumber(gasLimit * 1.1).toFixed(0, 2),
        }, 
        privateKey
    )
    // send transaction
    const tx = await web3.eth.sendSignedTransaction(signedTx.rawTransaction);
    console.log('tx: ', tx)

after this step, we have successfully minted the liquidity (if no revert occured)