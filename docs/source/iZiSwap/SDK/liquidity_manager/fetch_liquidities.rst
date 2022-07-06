.. _fetch_liquidities:

fetch liquidities
================================

here, we provide a simple example to fetch nft-liquidities of iZi swap from user's address

by calling **fetchLiquiditiesOfAccount()**

1. some imports
---------------

before we use function of **fetchLiquiditiesOfAccount()**, we should import some corresponding function and data structure

.. code-block:: typescript
    :linenos:

    import {BaseChain, ChainId, initialChainTable } from 'iziswap-sdk/lib/base/types'
    import {privateKey} from '../../.secret'
    import Web3 from 'web3'
    import { getLiquidityManagerContract, fetchLiquiditiesOfAccount } from 'iziswap-sdk/lib/liquidityManager/view';
    import { fetchToken } from 'iziswap-sdk/lib/base/token/token';

the detail of these imports can be viewed in following content

.. _base_obj:

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

.. _LiquidityManagerContract:

3. get web3.eth.Contract object of liquidityManager
---------------------------------------------------

.. code-block:: typescript
    :linenos:

    const liquidityManagerAddress = '0x93C22Fbeff4448F2fb6e432579b0638838Ff9581'
    const liquidityManagerContract = getLiquidityManagerContract(liquidityManagerAddress, web3)
    console.log('liquidity manager address: ', liquidityManagerAddress)

here, **getLiquidityManagerContract** is an api provided by our sdk, which returns a **web3.eth.Contract** object of **LiquidityManager**

4. cache some erc20 tokens to speed up fetching(optional)
---------------------------------------------------------

both of the function **fetchLiquiditiesOfAccount()** and **fetchLiquiditiesByTokenIds()** have a parameter called **tokenList: TokenInfoFormatted[]**, which is a cache of list of **TokenInfoFormatted**

each returned liquidity will contain fields **tokenX** and **tokenY** which are both **TokenInfoFormatted** type

if we fill tokenList with most erc20 tokens involved by these liquidities in advance, we may speed up calling of **fetchLiquiditiesOfAccount()** and **fetchLiquiditiesByTokenIds()**

ofcourse, you can skip this section and just transfer **[]** to the **tokenList** parameter in next section

here, we cache 2 erc20 tokens **testA** and **testB** in advance

.. code-block:: typescript
    :linenos:

    const testAAddress = '0xCFD8A067e1fa03474e79Be646c5f6b6A27847399'
    const testBAddress = '0xAD1F11FBB288Cd13819cCB9397E59FAAB4Cdc16F'

    const testA = await fetchToken(testAAddress, chain, web3)
    const testB = await fetchToken(testBAddress, chain, web3)

5. fetch!
---------

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


here,

**chain** is **BaseChain** obj specified in :ref:`2 <base_obj>`

**web3** is **Web3** obj specified in :ref:`2 <base_obj>`

**liquidityManagerContract** is constructed in :ref:`3 <LiquidityManagerContract>`

**account.address** is generated from private key in :ref:`2 <base_obj>`

**[testA, testB]** is parameter **tokenList** which is cache of list of possible erc20 token info needed, of course we can fill **tokenList** with **[]**

**return** of **fetchLiquiditiesOfAccount()** is list of **Liquidity** object, each has following fields

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

after this step, we have successfully fetched all liquidities of the user
