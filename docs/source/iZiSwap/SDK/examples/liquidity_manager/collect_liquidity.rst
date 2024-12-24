.. _collect_liquidities:

Collect a liquidity position
=============================

In this example, we first fetch all liquidities of an account, 
then select one of it and collect fee or decreased token from it.

The full example code of this chapter can be spotted `here <https://github.com/izumiFinance/iZiSwap-sdk/blob/main/example/liquidityManager/fetchLiquidityAndCollect.ts>`_.

1. Fetch liquidity positions
-----------------------------

.. code-block:: typescript
    :linenos:

    const chain:BaseChain = initialChainTable[ChainId.BSC]
    const rpc = 'https://bsc-dataseed2.defibit.io/'
    console.log('rpc: ', rpc)
    const web3 = new Web3(new Web3.providers.HttpProvider(rpc))
    const account =  web3.eth.accounts.privateKeyToAccount(privateKey)
    console.log('address: ', account.address)

    const liquidityManagerAddress = '0x93C22Fbeff4448F2fb6e432579b0638838Ff9581'
    const liquidityManagerContract = getLiquidityManagerContract(liquidityManagerAddress, web3)

    console.log('liquidity manager address: ', liquidityManagerAddress)

    const testAAddress = '0xCFD8A067e1fa03474e79Be646c5f6b6A27847399'
    const testBAddress = '0xAD1F11FBB288Cd13819cCB9397E59FAAB4Cdc16F'

    const testA = await fetchToken(testAAddress, chain, web3)
    const testB = await fetchToken(testBAddress, chain, web3)
    const fee = 2000 // 2000 means 0.2%

    const liquidities = await fetchLiquiditiesOfAccount(
        chain, 
        web3, 
        liquidityManagerContract,
        account.address,
        [testA]
    )
    console.log('liquidity len: ', liquidities.length)
    console.log('liquidity: ', liquidities)

The code above is nearly the same as :ref:`fetch_liquidities`, you can view more detailed explains through this link

2. Select one of your liquidity positions to collect
-----------------------------------------------------------

.. code-block:: typescript
    :linenos:

    const liquidity0 = liquidities[0]

of course one can only collect his/her liquidity, cannot collect others.

.. _set_max_amount_to_collect:

3. Set max amount of tokenX and tokenY you would collect
------------------------------------------------------------------------------

.. code-block:: typescript
    :linenos:

    const tokenA = liquidity0.tokenX
    const tokenB = liquidity0.tokenY

    const maxAmountA = liquidity0.remainTokenX
    const maxAmountB = liquidity0.remainTokenY

    console.log('tokenA: ', tokenA)
    console.log('tokenB: ', tokenB)

    console.log('maxAmountA: ', maxAmountA)
    console.log('maxAmountB: ', maxAmountB)

.. _get_calling_of_getCollectLiquidityCall:

4. Get calling of getCollectLiquidityCall
------------------------------------------------------------------------------

.. code-block:: typescript
    :linenos:

    const gasPrice = '5000000000'

    const {collectLiquidityCalling, options} = getCollectLiquidityCall(
        liquidityManagerContract,
        account.address,
        chain,
        {
            tokenId: liquidity0.tokenId,
            tokenA,
            tokenB,
            maxAmountA,
            maxAmountB
        } as CollectLiquidityParam,
        gasPrice
    )

the function **getCollectLiquidityCall(...)** has following params

.. code-block:: typescript
    :linenos:

    /**
     * @param liquidityManagerContract: web3.eth.Contract, the liquidity manager contract obj
     * @param account: string, string of owner's address
     * @param chain: BaseChain, the obj describing chain we are using
     * @param params: CollectLiquidityParam, specify two tokens and max undecimal amount you want to collect
     * @param gasPrice: string| number, gas price
     */
     export const getCollectLiquidityCall = (
        liquidityManagerContract: Contract, 
        account: string,
        chain: BaseChain,
        params: CollectLiquidityParam, 
        gasPrice: number | string
    )


**Notice:** if tokenX(tokenA) or tokenY(tokenB) is chain gas token (like **ETH** on ethereum or **BNB** on bsc),
and you want to collect tokenX or tokenY in form of native or wrapped-native token,
you can refer to
:ref:`following section<liquidity_collect_native_or_wrapped_native>`


.. _liquidity_collect_native_or_wrapped_native:

5. collect native or wrapped native token
------------------------------------------------------------

In the sdk version 1.2.* or later, 

If you want to collect in form of native token(like **BNB** on bsc or **ETH** on ethereum ...),
you should replace define code of **tokenA** and **tokenB** in :ref:`section 2<set_max_amount_to_collect>` with following code (here we are working on bsc chain), and 
fill **strictERC20Token** of **CollectLiquidityParam** in :ref:`section above<get_calling_of_getCollectLiquidityCall>` as **undefined** by default.
And the **options** calculated in :ref:`section above<get_calling_of_getCollectLiquidityCall>` will contain the corresponding **msg.value**.

.. code-block:: typescript
    :linenos:

    const BNBAddress = '0xbb4CdB9CBd36B01bD1cBaEBF2De08d9173bc095c';
    
    const tokenA = liquidity0.tokenX
    const tokenB = liquidity0.tokenY

    if (params.tokenA.address.toLowerCase() === BNBAddress.toLowerCase()) {
        params.tokenA.symbol = 'BNB';
    }
    if (params.tokenB.address.toLowerCase() === BNBAddress.toLowerCase()) {
        params.tokenB.symbol = 'BNB';
    }

If you want to collect in form of wrapped-native token(like **WBNB** on bsc or **WETH** on ethereum ...),
you should replace define code of **tokenA** and **tokenB** in :ref:`section 2<set_max_amount_to_collect>` with following code (here we are working on bsc chain), and 
fill **strictERC20Token** of **CollectLiquidityParam** in :ref:`section above<get_calling_of_getCollectLiquidityCall>` as **undefined** by default.

.. code-block:: typescript
    :linenos:

    const BNBAddress = '0xbb4CdB9CBd36B01bD1cBaEBF2De08d9173bc095c';
    
    const tokenA = liquidity0.tokenX
    const tokenB = liquidity0.tokenY

    if (params.tokenA.address.toLowerCase() === BNBAddress.toLowerCase()) {
        params.tokenA.symbol = 'WBNB'; // only difference to above code
    }
    if (params.tokenB.address.toLowerCase() === BNBAddress.toLowerCase()) {
        params.tokenB.symbol = 'WBNB'; // only difference to above code
    }

we can see that, the only difference of collection native token and wrapped-native token
is **symbol** field of **tokenA** or **tokenB**.


In the sdk version 1.1.* or before, one should specify a field named `strictERC20Token` to indicate that.
`true` for collecting token in form of `Wrapped Chain Token`, `false` for paying in form of `Chain Token`.
But we suggest you to upgrade your sdk to latest version.


6. Estimate gas (optional)
--------------------------

of course you can skip this step if you don't want to limit gas

.. code-block:: typescript
    :linenos:

    const gasLimit = await collectLiquidityCalling.estimateGas(options)
    console.log('gas limit: ', gasLimit)

7. Send transaction!
--------------------

for metamask or other explorer's wallet provider, you can easily write

.. code-block:: typescript
    :linenos:

    await collectLiquidityCalling.send({...options, gas: Number(gasLimit)})

otherwise, you could use following code

.. code-block:: typescript
    :linenos:

    // sign transaction
    const signedTx = await web3.eth.accounts.signTransaction(
        {
            ...options,
            to: liquidityManagerAddress,
            data: collectLiquidityCalling.encodeABI(),
            gas: new BigNumber(Number(gasLimit) * 1.1).toFixed(0, 2),
        }, 
        privateKey
    )
    // send transaction
    const tx = await web3.eth.sendSignedTransaction(signedTx.rawTransaction);
    console.log('tx: ', tx);

after sending transaction, we will successfully collect token from the liqudity (if no revert occurred)