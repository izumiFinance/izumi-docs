.. _fetch_limit_order_and_dec:

fetch limit order and dec
================================

here, we provide a simple example to fetch limit orders of iZiSwap from an user's address, and select one of them to decrease(or called cancel)

The full example code of this chapter can be spotted `here <https://github.com/izumiFinance/izumi-iZiSwap-sdk/blob/main/example/limitOrder/fetchLimitOrderAndDec.ts>`_.

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

2. select an order and get calling of decrease
--------------------------------------------------

first, select an order or your own

.. code-block:: typescript
    :linenos:

    const activeOrderAt2 = activeOrders[2]

second, get calling for decreasing the limit order

.. code-block:: typescript
    :linenos:

    const orderIdx = activeOrderAt2.idx
    const gasPrice = '5000000000'
    // dec limit order of orderIdx
    const {decLimOrderCalling, options} = getDecLimOrderCall(
        limitOrderManager,
        orderIdx,
        activeOrderAt2.sellingRemain,
        '0xffffffff',
        account.address,
        chain,
        gasPrice
    )


3.  estimate gas (optional)
---------------------------
of course you can skip this step if you don't want to limit gas.

.. code-block:: typescript
    :linenos:

    const gasLimit = await decLimOrderCalling.estimateGas(options)

4. finally, send transaction!
------------------------------

for metamask or other explorer's wallet provider, you can easily write 

.. code-block:: typescript
    :linenos:

    await decLimOrderCalling.send({...options, gas: gasLimit})

otherwise, if you run codes in console, you could use following code

.. code-block:: typescript
    :linenos:

    const signedTx = await web3.eth.accounts.signTransaction(
        {
            ...options,
            to: limitOrderAddress,
            data: decLimOrderCalling.encodeABI(),
            gas: new BigNumber(gasLimit * 1.1).toFixed(0, 2),
        }, 
        privateKey
    )
    // nonce += 1;
    const tx = await web3.eth.sendSignedTransaction(signedTx.rawTransaction);

after this step, we have successfully decrease a limit order (if no revert occurred).