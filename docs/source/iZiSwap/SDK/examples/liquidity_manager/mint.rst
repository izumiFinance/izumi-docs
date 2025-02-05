Mint
================================

In this section, we provide a simple example for creating a new liquidity position. Notice the liquidity position is a NFT.

The full example code of this chapter can be spotted `here <https://github.com/izumiFinance/izumi-iZiSwap-sdk/blob/main/example/liquidityManager/mint.ts>`_.


1. Some imports
---------------

.. code-block:: typescript
    :linenos:

    import Web3 from 'web3';
    import { BigNumber } from 'bignumber.js'
    import { privateKey } from '../../.secret'
    import { BaseChain, ChainId, initialChainTable, PriceRoundingType } from 'iziswap-sdk/lib/base/types'
    import { getPointDelta, getPoolContract, getPoolState } from 'iziswap-sdk/lib/pool/funcs';
    import { getPoolAddress, getLiquidityManagerContract } from 'iziswap-sdk/lib/liquidityManager/view';
    import { amount2Decimal, fetchToken } from 'iziswap-sdk/lib/base/token/token';
    import { pointDeltaRoundingDown, pointDeltaRoundingUp, priceDecimal2Point } from 'iziswap-sdk/lib/base/price';
    import { calciZiLiquidityAmountDesired } from 'iziswap-sdk/lib/liquidityManager/calc';
    import { getMintCall } from 'iziswap-sdk/lib/liquidityManager/liquidity';

Detail of these imports can be viewed in the following content.

.. _base_obj_mint:

2. Specify chain, rpc, web3, and account
--------------------------------------------------

.. code-block:: typescript
    :linenos:

    const chain:BaseChain = initialChainTable[ChainId.BSC]
    const rpc = 'https://bsc-dataseed2.defibit.io/'
    console.log('rpc: ', rpc)
    const web3 = new Web3(new Web3.providers.HttpProvider(rpc))
    const account =  web3.eth.accounts.privateKeyToAccount(privateKey)
    console.log('address: ', account.address)

where

* - **BaseChain** is a data structure to describe a chain, in this example we use **bsc** chain.
* - **ChainId** is an enum to describe **chain id**, value of the enum is equal to value of **chain id**.
* - **initialChainTable** is a mapping from some most used **ChainId** to **BaseChain**. You can fill fields of BaseChain by yourself.
* - **privateKey** is a string, which is your private key, and should be configured by your self.
* - **web3** is a public package to interact with block chain.
* - **rpc** is the rpc url on the chain you specified.

.. _LiquidityManagerContract_forMint:

3. Get web3.eth.Contract object of liquidityManager
---------------------------------------------------

.. code-block:: typescript
    :linenos:

    const liquidityManagerAddress = '0xBF55ef05412f1528DbD96ED9E7181f87d8C9F453' // example BSC address
    const liquidityManagerContract = getLiquidityManagerContract(liquidityManagerAddress, web3)
    console.log('liquidity manager address: ', liquidityManagerAddress)

Here, **getLiquidityManagerContract** is an api provided by our sdk, which returns a **web3.eth.Contract** object of **LiquidityManager**.

.. _specify_token_pair:

4. Specify Token Pair And Fee
---------------------------------------------------------

.. code-block:: typescript
    :linenos:

    const testAAddress = '0xCFD8A067e1fa03474e79Be646c5f6b6A27847399'
    const testBAddress = '0xAD1F11FBB288Cd13819cCB9397E59FAAB4Cdc16F'

    const testA = await fetchToken(testAAddress, chain, web3)
    const testB = await fetchToken(testBAddress, chain, web3)
    const fee = 2000; // 2000 means 0.2%

The function **fetchToken()** returns a **TokenInfoFormatted** obj of that token, which containing following fields.

You can fill **TokenInfoFormatted** by yourself, if you know each field correctly of the erc20-tokens related.
The TokenInfoFormatted fields used in sdk currently are only **symbol**, **address**, and **decimal**.

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

We usually set **TokenInfoFormatted.wrapTokenAddress** as undefined.

**Notice**: Here, we are ready to mint with token pair of **<testA,testB,2000>**, 
**testA** and **testB** here are both normal erc20-token.
And if you want to mint with a pair which contains chain gas token (like ETH on ethereum),
you can refer to :ref:`following section<mint_native_or_wrapped_native>`

.. _mint_native_or_wrapped_native:

5. mint with native or wrapped native
------------------------------------------------------------

In the sdk version 1.2.* or later, 

If you want to mint in form of native token(like **BNB** on bsc or **ETH** on ethereum ...),
for simplification, you are ready to mint with pair **<BNB, testB, 2000>**,
just simply replace **testA** in :ref:`section 4<specify_token_pair>` as **BNB** and 
fill **strictERC20Token** of **mintParams** in :ref:`section 9<liquidity_manager_mint_calling>` as **undefined** by default.
And the **options** calculated in :ref:`section 9<liquidity_manager_mint_calling>` will contain the corresponding **msg.value**.

.. code-block:: typescript
    :linenos:

    const testA = {
        chainId: ChainId.BSC,
        symbol: 'BNB', 
        // address of wbnb on bsc mainnet
        address: '0xbb4CdB9CBd36B01bD1cBaEBF2De08d9173bc095c',
        decimal: 18,
    } as TokenInfoFormatted;

If you want to mint in form of  wrapped-native token(like **WBNB** on bsc or **WETH** on ethereum ...),
for simplification, you are ready to mint with pair **<WBNB, testB, 2000>**,
just simply replace **testA** in :ref:`section 4<specify_token_pair>` as **WBNB** and 
fill **strictERC20Token** of **mintParams** in :ref:`section 9<liquidity_manager_mint_calling>` as **undefined** by default.

.. code-block:: typescript
    :linenos:

    const testA = {
        chainId: ChainId.BSC,
        symbol: 'WBNB', // only difference with above code
        // address of wbnb on bsc mainnet
        address: '0xbb4CdB9CBd36B01bD1cBaEBF2De08d9173bc095c',
        decimal: 18,
    } as TokenInfoFormatted;

we can see that, the only difference of mining native token and wrapped-native token
is **symbol** field of **sellToken**.


In the sdk version 1.1.* or before, one should specify a field named `strictERC20Token` to indicate that.
`true` for paying token in form of `Wrapped Chain Token`, `false` for paying in form of `Chain Token`.
But we suggest you to upgrade your sdk to latest version.


6. Get state of the corresponding pool
---------------------------------------------------------

First get the pool address of token pair (testA, testB, fee):

.. code-block:: typescript
    :linenos:

    const poolAddress = await getPoolAddress(liquidityManagerContract, testA, testB, fee)

The function **getPoolAddress(...)** queries **liquidityManagerContract** to get iZiSwap pool address of token pair **(testA, testB, fee)**, where

 * - **liquidityManagerContract**: liquidity manager contract, acquired in step 4.
 * - **testA**: an erc20 token in type of TokenInfoFormatted, acquired in step 5.
 * - **testB**: another erc20 token in type of TokenInfoFormatted, also acquired in step 5.
 * - **fee**: an int number, fee/1e6 is fee rate of pool, etc, 2000 means 0.2% fee rate
  
When **poolAddress** is ready, you can call **getPoolContract(...)** to get the pool contract object.

.. code-block:: typescript
    :linenos:

    const pool = getPoolContract(poolAddress, web3)

Then we can get the state of the pool:

.. code-block:: typescript
    :linenos:

    const state = await getPoolState(pool)

where state is a **State** obj which extends from **BaseState**, with only fields in **BaseState** are used in this example.


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

.. _compute_boundary_point:

7.  Compute boundary point of liquidity on the pool
---------------------------------------------------------

The boundary point is **leftPoint** and **rightPoint** of a liquidity position, according to :ref:`price` , we know that **point** in the pool and **decimal price** can be transformed from each other.

We first determine the minimal and maximum **decimal price** of our liquidity ready to mint.

Assume the desired minimal **decimal price** of **A_by_B** is **0.099870** (this decimal price means 0.099870 testB to buy 1.0 testA, here, number 0.099870 and 1.0 are both **decimal amount**).
and the max **decimal price** of  `A_by_B` is `0.29881`

We can get 2 **point**s on the pool of min and max **decimal prices** though following code:

.. code-block:: typescript
    :linenos:

    const point1 = priceDecimal2Point(testA, testB, 0.099870, PriceRoundingType.PRICE_ROUNDING_NEAREST)
    const point2 = priceDecimal2Point(testA, testB, 0.29881, PriceRoundingType.PRICE_ROUNDING_NEAREST)

where **priceDecimal2Point(...)** is a function to transform **decimal price** to the **point** on the pool, the function has following params:

.. code-block:: typescript

    /**
     * @param tokenA: TokenInfoFormatted, one erc20 token of pool
     * @param tokenB: TokenInfoFormatted, another erc20 token of pool
     * @param priceDecimalAByB: number,  decimal price of A_by_B (A_by_B means how much tokenB to buy 1 tokenA)
     * @param roundingType: PriceRoundingType, rounding type when transform price to point
     * @return point: number, point on the pool transformed from decimal price
     */
    priceDecimal2Point(tokenA, tokenB, priceDecimalAByB, roundingType)

Since we do not ensure that tokenA's address is smaller than tokenB, point1 may be larger than point2. We could not simply specify leftPoint as point1 and rightPoint as point2.
Instead, we take min(point1, point2) as leftPoint and max(point1, point2) as rightPoint.

.. code-block:: typescript
    :linenos:

    let leftPoint = Math.min(point1, point2)
    let rightPoint = Math.max(point1, point2)

When we mint, the boundary point of liquidity must be times of `pointDelta`.
Thus we should rounding `leftPoint` and `rightPoint` to times of `pointDelta` throw following codes:

.. code-block:: typescript
    :linenos:

    const pointDelta = await getPointDelta(pool)
    
    leftPoint = pointDeltaRoundingDown(leftPoint, pointDelta)
    rightPoint = pointDeltaRoundingUp(rightPoint, pointDelta)

where **pointDelta** is a number value queried from pool contract.

For fee rate of 0.2%, pointDelta usually equals to **40** (0.3% -> **60**, 1% -> **200**).

Besides, for **leftPoint** and **rightPoint** we must guarantee following inequality:

.. code-block:: typescript

    leftPoint >= pool.leftMostPt()
    rightPoint <= pool.rightMostPt()
    rightPoint - leftPoint < 400000

8. Specify or compute tokenA's and tokenB's max undecimal amount (optional)
----------------------------------------------------------------------------------------

Sometimes, a user wants to know the amount of tokenA when he fill amount of tokenB or vise versa.

Here we provide a function named `calciZiLiquidityAmountDesired()` in sdk to do this calculation.

Suppose we want to specify max decimal amount of tokenA ( token named testA) to be 100,

.. code-block:: typescript
    :linenos:

    const maxTestA = new BigNumber(100).times(10 ** testA.decimal)

then we can compute the corresponding undecimal amount of tokenB ( token named testB).

.. code-block:: typescript
    :linenos:

    const maxTestB = calciZiLiquidityAmountDesired(
        leftPoint, rightPoint, state.currentPoint,
        maxTestA, true, testA, testB
    )

Here, `calciZiLiquidityAmountDesired(...)` is a function provided by sdk,
which is used for computing one erc20-token's undecimal amount of a liquidity after
given `leftPoint` `rightPoint` `currentPoint` and  the other erc20-token's undecimal amount.

The params are as follows:

.. code-block:: typescript

   /**
    * @param leftPoint: number, left point of the liquidity
    * @param rightPoint: number, right point of the liquidity
    * @param currentPoint: number, current point on the swap pool
    * @param amount: BigNumber, undecimal amount of one token
    * @param amountIsTokenA: boolean, true for amount is tokenA's undecimal amount, false for tokenB
    * @param tokenA: TokenInfoFormatted, tokenA information
    * @param tokenB: TokenInfoFormatted, tokenB information
    */
   calciZiLiquidityAmountDesired(leftPoint, rightPoint, currentPoint, amount, amountIsTokenA, tokenA, tokenB):


After the call to `calciZiLiquidityAmountDesired`, we get a `BigNumber` stored in `maxTestB`, which is corresponding undecimal amount of tokenB ( token named testB).

.. _liquidity_manager_mint_calling:

9. Get the mint calling
-----------------------

First, construct necessary params and gasPrice for the mint calling.

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

Then, get mint calling by:

.. code-block:: typescript
    :linenos:

    const { mintCalling, options } = getMintCall(
        liquidityManagerContract,
        account.address,
        chain,
        mintParams,
        gasPrice
    )

where **mintParams** of type MintParam,  and **maxAmountA**, **maxAmountB**, **minAmountA**, **minAmountB**
is required min-max undecimal amount of tokenA and tokenB deposited in this mint procedure.
You can fill **maxAmountA**, **maxAmountB**, **minAmountA**, **minAmountB** to arbitrary value as you want.

The function **getMintCall** returns 2 object, **mintCalling** and **options**.
When **mintCalling** and **options** are ready, we can estimate gas.


Notice that, to mint with pair contains chain gas token (like `ETH` on ethereum or `BNB` on bsc),
you can refer to :ref:`this section<mint_native_or_wrapped_native>`.



10.  Approve (skip if you mint with native token directly)
-------------------------------------------------------------

Before estimate gas or send transaction, you need approve contract **liquidityManager** to have authority to spend your token,
since you need transfer some tokenA and some tokenB to pool.

.. code-block:: typescript
    :linenos:

    // the approve interface abi of erc20 token
    const erc20ABI = [{
      "inputs": [
        {
          "internalType": "address",
          "name": "spender",
          "type": "address"
        },
        {
          "internalType": "uint256",
          "name": "amount",
          "type": "uint256"
        }
      ],
      "name": "approve",
      "outputs": [
        {
          "internalType": "bool",
          "name": "",
          "type": "bool"
        }
      ],
      "stateMutability": "nonpayable",
      "type": "function"
    }];

    // if tokenA is not chain token (BNB on bsc chain or ETH on eth chain...), we need transfer tokenA to pool
    // otherwise we can skip following codes
    if (maxTestA.gt(0)) {
        const tokenAContract = new web3.eth.Contract(erc20ABI, testAAddress);
        // you could approve a very large amount (much more greater than amount to transfer),
        // and don't worry about that because liquidityManager only transfer your token to pool with amount you specified and your token is safe
        // then you do not need to approve next time for this user's address
        const approveCalling = tokenAContract.methods.approve(
            liquidityManagerAddress, 
            "0xffffffffffffffffffffffffffffffff"
        );
        // estimate gas
        const gasLimit = await approveCalling.estimateGas({from: account})
        // then send transaction to approve
        // you could simply use followiing line if you use metamask in your frontend code
        // otherwise, you should use the function "web3.eth.accounts.signTransaction"
        // notice that, sending transaction for approve may fail if you have approved the token to liquidityManager before
        // if you want to enlarge approve amount, you should refer to interface of erc20 token
        await approveCalling.send({gas: Number(gasLimit)})
    }
    
    // if tokenB is not chain token (BNB on bsc chain or ETH on eth chain...), we need transfer tokenA to pool
    // otherwise we can skip following codes
    if (mexTestB.gt(0)) {
        const tokenBContract = new web3.eth.Contract(erc20ABI, testBAddress);
        // you could approve a very large amount (much more greater than amount to transfer),
        // and don't worry about that because liquidityManager only transfer your token to pool with amount you specified and your token is safe
        // then you do not need to approve next time for this user's address
        const approveCalling = tokenBContract.methods.approve(
            liquidityManagerAddress, 
            "0xffffffffffffffffffffffffffffffff"
        );
        // estimate gas
        const gasLimit = await approveCalling.estimateGas({from: account})
        // then send transaction to approve
        // you could simply use followiing line if you use metamask in your frontend code
        // otherwise, you should use the function "web3.eth.accounts.signTransaction"
        // notice that, sending transaction for approve may fail if you have approved the token to liquidityManager before
        // if you want to enlarge approve amount, you should refer to interface of erc20 token
        await approveCalling.send({gas: Number(gasLimit)})
    }


11.  Estimate gas (optional)
-----------------------------
You can skip this step if you do not want to limit gas.

.. code-block:: typescript
    :linenos:

    const gasLimit = await mintCalling.estimateGas(options)
    console.log('gas limit: ', gasLimit)

12. Finally, send transaction!
------------------------------

Now, we can send transaction to mint a new liquidity position.

For metamask or other injected wallet provider, you can easily write 

.. code-block:: typescript
    :linenos:

    await mintCalling.send({...options, gas: Number(gasLimit)})

Otherwise, if you are running codes in console, you could use the following code

.. code-block:: typescript
    :linenos:

    // sign transaction
    const signedTx = await web3.eth.accounts.signTransaction(
        {
            ...options,
            to: liquidityManagerAddress,
            data: mintCalling.encodeABI(),
            gas: new BigNumber(Number(gasLimit) * 1.1).toFixed(0, 2),
        }, 
        privateKey
    )
    // send transaction
    const tx = await web3.eth.sendSignedTransaction(signedTx.rawTransaction);
    console.log('tx: ', tx)

Finally, we have successfully minted a liquidity position (if no revert occurred).