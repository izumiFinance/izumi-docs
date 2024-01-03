.. _fetch_liquidities:

Fetch liquidity positions
================================

Here, we provide an example to fetch liquidity positions of a user address.

The function is realized by calling **fetchLiquiditiesOfAccount()**,

The full example code of this chapter can be spotted `here <https://github.com/izumiFinance/izumi-iZiSwap-sdk/blob/main/example/liquidityManager/fetchLiquidity.ts>`_.

1. Some imports
----------------

Before we use function of **fetchLiquiditiesOfAccount()**, we should import some corresponding function and data structure.

.. code-block:: typescript
    :linenos:

    import {BaseChain, ChainId, initialChainTable } from 'iziswap-sdk/lib/base/types'
    import {privateKey} from '../../.secret'
    import Web3 from 'web3'
    import { getLiquidityManagerContract, fetchLiquiditiesOfAccount } from 'iziswap-sdk/lib/liquidityManager/view';
    import { fetchToken } from 'iziswap-sdk/lib/base/token/token';

Details of these imports can be viewed in following content.

.. _base_obj:

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

Where

* - **BaseChain** is a data structure to describe a chain, in this example we use **bsc** chain.
* - **ChainId** is an enum to describe **chain id**, value of the enum is equal to value of **chain id**.
* - **initialChainTable** is a mapping from some most used **ChainId** to **BaseChain**. You can fill fields of BaseChain by yourself.
* - **privateKey** is a string, which is your private key, and should be configured by yourself.
* - **web3**  is a public package to interact with block chain.
* - **rpc** is the rpc url on the chain you specified.

.. _LiquidityManagerContract:

3. Get the web3.eth.Contract object of liquidityManager
-------------------------------------------------------

.. code-block:: typescript
    :linenos:

    const liquidityManagerAddress = '0xBF55ef05412f1528DbD96ED9E7181f87d8C9F453' // example BSC address
    const liquidityManagerContract = getLiquidityManagerContract(liquidityManagerAddress, web3)
    console.log('liquidity manager address: ', liquidityManagerAddress)

Here, **getLiquidityManagerContract** is an api provided by our sdk, which returns a **web3.eth.Contract** object of **LiquidityManager**.

4. Cache some erc20 tokens to speed up fetching(optional)
---------------------------------------------------------

Both of the function **fetchLiquiditiesOfAccount()** and **fetchLiquiditiesByTokenIds()** have a parameter called **tokenList: TokenInfoFormatted[]**, which is a cache list of **TokenInfoFormatted**.

Each returned liquidity position contains fields **tokenX** and **tokenY**,  which are both of **TokenInfoFormatted** type.

If you fill tokenList with most erc20 tokens involved by these liquidities in advance, you may speed up the calling of **fetchLiquiditiesOfAccount()** and **fetchLiquiditiesByTokenIds()**.

You can also skip this section and just transfer **[]** to the **tokenList** parameter in next section.

Here, we cache 2 erc20 tokens **testA** and **testB** in advance.

.. code-block:: typescript
    :linenos:

    const testAAddress = '0xCFD8A067e1fa03474e79Be646c5f6b6A27847399'
    const testBAddress = '0xAD1F11FBB288Cd13819cCB9397E59FAAB4Cdc16F'

    const testA = await fetchToken(testAAddress, chain, web3)
    const testB = await fetchToken(testBAddress, chain, web3)

5. Fetch!
----------

.. code-block:: typescript
    :linenos:

    const liquidities = await fetchLiquiditiesOfAccount(
        chain, 
        web3, 
        liquidityManagerContract,
        account.address,
        [testA, testB]
    )
    console.log('liquidity len: ', liquidities.length)
    console.log('liquidtys: ', liquidities)


Here,

* - **chain** is a **BaseChain** obj specified in :ref:`2 <base_obj>`.
* - **web3** is a **Web3** obj specified in :ref:`2 <base_obj>`.
* - **liquidityManagerContract** is constructed in :ref:`3 <LiquidityManagerContract>`.
* - **account.address** is generated from private key in :ref:`2 <base_obj>`.
* - **[testA, testB]** is parameter **tokenList** which is cache of list of possible erc20 token info needed, of course we can fill **tokenList** with **[]**.

The function **return** of **fetchLiquiditiesOfAccount()** is list of **Liquidity** object, each has following fields.

.. code-block:: typescript
    :linenos:

    export interface Liquidity {
        // value of nft-id, a int value, but may be too large, so transformed into decimal system string
        tokenId: string;
        // left_point_on_pool of liquidity
        // describe min_undecimal_price_X_by_Y of this liquidity
        leftPoint: number;
        // right_point_on_pool of liquidity
        // describe max_undecimal_price_X_by_Y of this liquidity
        rightPoint: number;
        // value of liquidity on each point in [leftPoint, rightPoint),
        // a int value, but may be too large, so transformed into decimal system string
        liquidity: string;
        lastFeeScaleX_128: string;
        lastFeeScaleY_128: string;
        // undecimal amount of uncollected tokenX fee or withdrawed tokenX,
        remainTokenX: string;
        // undecimal amount of uncollected tokenY fee or withdrawed tokenY
        remainTokenY: string;
        // undecimal amount of tokenX in the liquidity (after latest withdraw or add or mint)
        amountX: string;
        // undecimal amount of tokenY in the liquidity (after latest withdraw or add or mint)
        amountY: string;
        poolId: string;
        poolAddress: string;
        tokenX: TokenInfoFormatted;
        tokenY: TokenInfoFormatted;
        // 2000 means 0.2%
        fee: number;
        // state() of pool
        state: State;
    }

Finally, we have successfully fetched all liquidity positions of an address.
