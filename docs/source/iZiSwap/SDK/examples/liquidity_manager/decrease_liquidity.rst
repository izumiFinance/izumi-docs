decrease a liquidity
====================

in this example, we first fetch all liquidities of an account, 
then select one of it and decrease (or called withdraw) some liquidity

The full example code of this chapter can be spotted `here <https://github.com/izumiFinance/izumi-iZiSwap-sdk/blob/main/example/liquidityManager/fetchLiquidityAndDec.ts>`_.

We should notice that, after decrease liquidity, the tokens are still in the pool contract.
To collect your decreased or fee token from pool. you can see this example :ref:`collect_liquidities`.

1. fetch liquidities
--------------------

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

the code above is nearly the same as :ref:`fetch_liquidities`, you can view more detailed explains though this link

1. select one of your liquidities to decrease (or withdraw)
-----------------------------------------------------------

.. code-block:: typescript
    :linenos:

    const liquidity0 = liquidities[0]

    const decRate = 0.1
    const originLiquidity = new BigNumber(liquidity0.liquidity)
    const decLiquidity = originLiquidity.times(decRate)

of course one can only withdraw his/her liquidity, cannot withdraw others.

here, **decLiquidity** is value of liquidity to withdraw, and it is type of BigNumber

3. pre-compute undecimal amount of tokenX and tokenY after withdraw (optional)
------------------------------------------------------------------------------
the market may change very fast, our frontend app should allow user to select a **slippery value** before withdraw.

setting **slippery value** can be viewed on many apps such as **uniswapv3**, **pancake**...

of course you can skip this step and fill **minAmountX** and **minAmountY** as '0' if you donot care

.. code-block:: typescript
    :linenos:

    const {amountX, amountY} = getWithdrawLiquidityValue(
        liquidity0,
        liquidity0.state,
        decLiquidity
    )
    // here the slippery is 1.5%
    const minAmountX = amountX.times(0.985).toFixed(0)
    const minAmountY = amountY.times(0.985).toFixed(0)

here, **getWithdrawLiquidityValue** is the function provide by our sdk to pre compute **undecimal_amount** of tokenX and tokenY withdrawed from liquidity. 

.. code-block:: typescript
    :linenos:

    /**
     * @param liquidity: Liquidity, the liquidity object describe the liquidity you want to withdraw
     * @param state: State, the state queried from the pool, can be obtained by liquidity.state
     * @param withdrawLiquidity: BigNumber, value of liquidity you want to withdraw, could not larger than liquidity.liquidity
     * @return amountX: BigNumber, estimated undecimal amount of tokenX acquired after withdraw
     * @return amountY: BigNumber, estimated undecimal amount of tokenY acquired after withdraw
     * @return amountXDecimal: number, estimated decimal amount of tokenX acquired after withdraw
     * @return amountYDecimal: number, estimated decimal amount of tokenY acquired after withdraw
     */
     getWithdrawLiquidityValue(liquidity, state, withdrawLiquidity)

4. get calling of decreaseLiquidity (or we say withdraw)

.. code-block:: typescript
    :linenos:

    const gasPrice = '5000000000'

    const {decLiquidityCalling, options} = getDecLiquidityCall(
        liquidityManagerContract,
        account.address,
        chain,
        {
            tokenId: liquidity0.tokenId,
            liquidDelta: decLiquidity.toFixed(0),
            minAmountX,
            minAmountY
        } as DecLiquidityParam,
        gasPrice
    )

the function **getDecLiquidityCall(...)** has following params

.. code-block:: typescript
    :linenos:

    /**
     * @param liquidityManagerContract: web3.eth.Contract, the liquidity manager contract obj
     * @param accountAddress: string, string of owner's address
     * @param chain: BaseChain, the obj describing chain we are using
     * @param gasPrice: string| number, gas price
     */
     getDecLiquidityCall(liquidityManagerContract, accountAddress, chain, params, gasPrice)

5. estimate gas (optional)
--------------------------

of course you can skip this step if you don't want to limit gas

.. code-block:: typescript
    :linenos:

    const gasLimit = await decLiquidityCalling.estimateGas(options)
    console.log('gas limit: ', gasLimit)

6. send transaction!
--------------------

for metamask or other explorer's wallet provider, you can easily write

.. code-block:: typescript
    :linenos:

    await decLiquidityCalling.send({...options, gas: gasLimit})

otherwise, you could use following code

.. code-block:: typescript
    :linenos:

    // sign transaction
    const signedTx = await web3.eth.accounts.signTransaction(
        {
            ...options,
            to: liquidityManagerAddress,
            data: decLiquidityCalling.encodeABI(),
            gas: new BigNumber(gasLimit * 1.1).toFixed(0, 2),
        }, 
        privateKey
    )
    // send transaction
    const tx = await web3.eth.sendSignedTransaction(signedTx.rawTransaction);
    console.log('tx: ', tx);

after sending transaction, we will successfully decrease the liquidity (if no revert occurred)