.. _fetch_limit_order:

fetch limit order
================================

here, we provide a simple example to fetch limit orders of iZiSwap from an user's address

by calling **fetchLimitOrderOfAccount()**

The full example code of this chapter can be spotted `here <https://github.com/izumiFinance/izumi-iZiSwap-sdk/blob/main/example/limitOrder/fetchLimitOrder.ts>`_.

1. some imports
---------------

before we use function of **fetchLimitOrderOfAccount()**, we should import some corresponding function and data structure

.. code-block:: typescript
    :linenos:

    import {BaseChain, ChainId, initialChainTable } from 'iziswap-sdk/lib/base/types'
    import {privateKey} from '../../.secret'
    import Web3 from 'web3';
    import { fetchToken } from 'iziswap-sdk/lib/base/token/token';
    import { fetchLimitOrderOfAccount, getLimitOrderManagerContract } from 'iziswap-sdk/lib/limitOrder/view';

the detail of these imports can be viewed in following content

.. _base_obj_of_fetch_limit_order:

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

.. _LimitOrderManagerContract:

3. get web3.eth.Contract object of LimitOrderManager
----------------------------------------------------

.. code-block:: typescript
    :linenos:

    const limitOrderAddress = '0x9Bf8399c9f5b777cbA2052F83E213ff59e51612B'
    const limitOrderManager = getLimitOrderManagerContract(limitOrderAddress, web3)

here, **getLimitOrderManagerContract** is an api provided by our sdk, which returns a **web3.eth.Contract** object of **LimitOrderManager**

4. cache some erc20 tokens to speed up fetching(optional)
---------------------------------------------------------

the function **fetchLimitOrderOfAccount()**  has a parameter called **tokenList: TokenInfoFormatted[]**, which is a cache of list of **TokenInfoFormatted**

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

    const {activeOrders, deactiveOrders} = await fetchLimitOrderOfAccount(
        chain, web3, limitOrderManager, account.address, [testA]
    )

    console.log('active orders len: ', activeOrders.length)
    console.log('deactive orders len: ', deactiveOrders.length)
    console.log(activeOrders)


here,

**chain** is **BaseChain** obj specified in :ref:`2 <base_obj_of_fetch_limit_order>`

**web3** is **Web3** obj specified in :ref:`2 <base_obj_of_fetch_limit_order>`

**liquidityManagerContract** is constructed in :ref:`3 <LiquidityManagerContract>`

**account.address** is generated from private key in :ref:`2 <base_obj_of_fetch_limit_order>`

**[testA, testB]** is parameter **tokenList** which is cache of list of possible erc20 token info needed, of course we can fill **tokenList** with **[]**

**return** of **fetchLimitOrderOfAccount()** is list of **LimitOrder** object, each has following fields

.. code-block:: typescript
    :linenos:

    export interface LimitOrder {
        // slot idx of the limit in use's limit order set
        idx: string,
        lastAccEarn: string,
        // original undecimal amount of token on sale when this limit order created
        // if sellXEarnY is true, the token is tokenX, otherwise tokenY
        amount: string,
        // undecimal amount of saled
        filled: string,
        // undecimal amount of 
        sellingRemain: string,
        // undecimal amount of token on sale currently
        sellingDec: string,
        accSellingDec: string,
        // undecimal amount of claimed earning token
        // if sellXEarnY is true, the token is tokenY, otherwise tokenX
        earn: string,
        // undecimal amount of unclaimed earning token
        pending: string,
        poolId: string,
        // address of pool
        poolAddress: string,

        // if sellXEarnY is true, sell tokenX
        tokenX: TokenInfoFormatted,
        tokenY: TokenInfoFormatted,
        createTime: Number,
        // point of limit order
        point: number,
        // price of this order
        priceXByY: BigNumber,
        priceXByYDecimal: number,
        // true for sellX, false for sellY
        sellXEarnY: boolean,
        // whether this limit order is active
        active: boolean
    }

after this step, we have successfully fetched all limit orders of the user.
