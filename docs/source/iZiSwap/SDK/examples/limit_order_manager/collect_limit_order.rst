.. _collect_limit_order:

collect limit order
================================

here, we provide a simple example to fetch limit orders of iZiSwap from an user's address, and select one of them to collect

The full example code of this chapter can be spotted `here <https://github.com/izumiFinance/izumi-iZiSwap-sdk/blob/main/example/limitOrder/collectLimitOrder.ts>`_.


1. fetch limit orders
---------------------

.. code-block:: typescript
    :linenos:

    const chain:BaseChain = initialChainTable[ChainId.BSC]
    const rpc = 'https://bsc-dataseed2.defibit.io/'
    const web3 = new Web3(new Web3.providers.HttpProvider(rpc))
    const account =  web3.eth.accounts.privateKeyToAccount(privateKey)

    const limitOrderAddress = '0x9Bf8399c9f5b777cbA2052F83E213ff59e51612B'
    const limitOrderManager = getLimitOrderManagerContract(limitOrderAddress, web3)


    const testAAddress = '0xCFD8A067e1fa03474e79Be646c5f6b6A27847399'
    const testBAddress = '0xAD1F11FBB288Cd13819cCB9397E59FAAB4Cdc16F'

    const testA = await fetchToken(testAAddress, chain, web3)
    const testB = await fetchToken(testBAddress, chain, web3)
    const fee = 2000 // 2000 means 0.2%

    // fetch limit order
    const {activeOrders, deactiveOrders} = await fetchLimitOrderOfAccount(
        chain, web3, limitOrderManager, '0xD0B1c02E8A6CA05c7737A3F4a0EEDe075fa4920C', [testA]
    )

the code above is nearly the same as :ref:`fetch_limit_order`, you can view more detailed explains though this link

.. _select_an_order_to_collect_and_get_calling:

2. select an order to collect and get calling
-------------------------------------------------------------

first, select an order or your own

.. code-block:: typescript
    :linenos:

    const activeOrderAt2 = activeOrders[2]

second, get calling of decrease the limit order

.. code-block:: typescript
    :linenos:

    const orderIdx = activeOrderAt2.idx
    const gasPrice = '5000000000'
    const params: CollectLimOrderParam = {
        orderIdx,
        tokenX: activeOrderAt2.tokenX,
        tokenY: activeOrderAt2.tokenY,
        collectDecAmount: activeOrderAt2.sellingDec,
        collectEarnAmount: '1'
    }
    // dec limit order of orderIdx
    const {collectLimitOrderCalling, options} = getCollectLimitOrderCall(
        limitOrderManager,
        account.address,
        chain,
        params,
        gasPrice
    )

**Notice:** if tokenX or tokenY is chain gas token (like **ETH** on ethereum or **BNB** on bsc),
and you want to collect tokenX or tokenY in form of native or wrapped-native token,
you can refer to
:ref:`following section<collect_native_or_wrapped_native>`


.. _collect_native_or_wrapped_native:

3. collect native or wrapped native token
------------------------------------------------------------

In the sdk version 1.2.* or later, 

If you want to collect in form of native token(like **BNB** on bsc or **ETH** on ethereum ...),
you should replace define of **params** in :ref:`section 2<select_an_order_to_collect_and_get_calling>` with following code (here we are working on bsc chain), and 
fill **strictERC20Token** of **params** as **undefined** by default.

.. code-block:: typescript
    :linenos:

    const BNBAddress = '0xbb4CdB9CBd36B01bD1cBaEBF2De08d9173bc095c';
    const params: CollectLimOrderParam = {
        orderIdx,
        tokenX: activeOrderAt2.tokenX,
        tokenY: activeOrderAt2.tokenY,
        collectDecAmount: activeOrderAt2.sellingDec,
        collectEarnAmount: '1'
    }
    if (params.tokenX.address.toLowerCase() === BNBAddress.toLowerCase()) {
        params.tokenX.symbol = 'BNB';
    }
    if (params.tokenY.address.toLowerCase() === BNBAddress.toLowerCase()) {
        params.tokenY.symbol = 'BNB';
    }

If you want to collect in form of wrapped-native token(like **WBNB** on bsc or **WETH** on ethereum ...),
you should replace define of **params** in :ref:`section 2<select_an_order_to_collect_and_get_calling>` with following code (here we are working on bsc chain), and 
fill **strictERC20Token** of **params** as **undefined** which is the default value for that field.

.. code-block:: typescript
    :linenos:

    const BNBAddress = '0xbb4CdB9CBd36B01bD1cBaEBF2De08d9173bc095c';
    const params: CollectLimOrderParam = {
        orderIdx,
        tokenX: activeOrderAt2.tokenX,
        tokenY: activeOrderAt2.tokenY,
        collectDecAmount: activeOrderAt2.sellingDec,
        collectEarnAmount: '1'
    }
    if (params.tokenX.address.toLowerCase() === BNBAddress.toLowerCase()) {
        params.tokenX.symbol = 'WBNB'; // only difference with above code
    }
    if (params.tokenY.address.toLowerCase() === BNBAddress.toLowerCase()) {
        params.tokenY.symbol = 'WBNB'; // only difference with above code
    }

we can see that, the only difference of collection native token and wrapped-native token
is **symbol** field of **params.tokenX** or **params.tokenY**.


In the sdk version 1.1.* or before, one should specify a field named `strictERC20Token` to indicate that.
`true` for collecting token in form of `Wrapped Chain Token`, `false` for paying in form of `Chain Token`.
But we suggest you to upgrade your sdk to latest version.


4.  estimate gas (optional)
---------------------------
of course you can skip this step if you don't want to limit gas.

.. code-block:: typescript
    :linenos:

    const gasLimit = await collectLimitOrderCalling.estimateGas(options)

5. finally, send transaction!
------------------------------

for metamask or other explorer's wallet provider, you can easily write 

.. code-block:: typescript
    :linenos:

    await collectLimitOrderCalling.send({...options, gas: Number(gasLimit)})

otherwise, if you run codes in console, you could use following code

.. code-block:: typescript
    :linenos:

    const signedTx = await web3.eth.accounts.signTransaction(
        {
            ...options,
            to: limitOrderAddress,
            data: collectLimitOrderCalling.encodeABI(),
            gas: new BigNumber(Number(gasLimit) * 1.1).toFixed(0, 2),
        }, 
        privateKey
    )
    // nonce += 1;
    const tx = await web3.eth.sendSignedTransaction(signedTx.rawTransaction);

after this step, we have successfully collect a limit order (if no revert occurred).