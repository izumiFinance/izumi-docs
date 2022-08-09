decrease liquidity and collect
================================

here, we provide a simple example for decreasing some liquidity from existing liquidity and withdraw tokens through box's interface


1. some imports
---------------

.. code-block:: typescript
    :linenos:

    import {BaseChain, ChainId, initialChainTable, TokenInfoFormatted} from 'iziswap-sdk/lib/base/types'
    import {privateKey} from '../../.secret'
    import Web3 from 'web3';
    import { getPoolContract, getPoolState } from 'iziswap-sdk/lib/pool/funcs';
    import { getSwapTokenAddress } from 'iziswap-sdk/lib/base/token/token';
    import { BigNumber } from 'bignumber.js'
    import { getLiquidityManagerContract, getPoolAddress, getWithdrawLiquidityValue, Liquidity } from 'iziswap-sdk/lib/liquidityManager';
    import { getBoxContract, DecLiquidityAndCollectParams, getDecLiquidityAndCollectCall } from 'iziswap-sdk/lib/box';

the detail of these imports can be viewed in following content


2. specify which chain, rpc url, web3, and account
--------------------------------------------------

.. code-block:: typescript
    :linenos:

    const chain:BaseChain = initialChainTable[ChainId.BSCTestnet]
    const rpc = 'https://data-seed-prebsc-2-s3.binance.org:8545/'
    const web3 = new Web3(new Web3.providers.HttpProvider(rpc))
    const account =  web3.eth.accounts.privateKeyToAccount(privateKey)

we use bsc test net in this example.

**BaseChain** is a data structure to describe a chain, in this example we use **bsc** chain.

**ChainId** is an enum to describe **chain id**, value of the enum is equal to value of **chain id**

**initialChainTable** is a mapping from some most used **ChainId** to **BaseChain**, of course you can fill fields of BaseChain by yourself

**privateKey** is a string, which is your private key, and should be configured by your self

**web3** package is a public package to interact with block chain

**rpc** is the rpc url on the chain you specified

.. _BoxContract_forDecAndCollect:

3. get web3.eth.Contract object of box
---------------------------------------------------

.. code-block:: typescript
    :linenos:

    const boxAddress = '0x904C130a8bf933f5c11Ea58CAA306f2296db22af'
    const boxContract = getBoxContract(boxAddress, web3)

here, **getBoxContract** is an api provided by our sdk, which returns a **web3.eth.Contract** object of **Box**

4. specify token pair
---------------------------------------------------------

.. code-block:: typescript
    :linenos:

    const feeB = {
        chainId: chain.id,
        name: '',
        symbol: 'FeeB',
        icon: '',
        address: '0x0C2CE63c797190dAE219A92AeBE4719Dc83AADdf',
        wrapTokenAddress: '0x5a2FEa91d21a8D53180020F8272594bf0D6F36DC',
        decimal: 18,
    } as TokenInfoFormatted
    
    const wBNB = {
        chainId: chain.id,
        name: '',
        symbol: 'BNB',
        icon: '',
        address: '0xa9754f0D9055d14EB0D2d196E4C51d8B2Ee6f4d3',
        wrapTokenAddress: undefined,
        decimal: 18,
    } as TokenInfoFormatted

    const fee = 2000 // 2000 means 0.2%

in the above code, the token pair is **feeB/wBNB**, fee is 0.2%.
here, **feeB** is a token which has transfer fee (about 5% during each transfer).
we can see in the feeB, we fill a field named **wrapTokenAddress** which is address of its **Wrap Token**
for token wBNB, its **wrapTokenAddress** field is undefined.

our sdk will check **TokenInfoFormatted.wrapTokenAddress**, if it is undefined, we will regard it as token with no transfer fee.
if it is not undefined, we will assume that this token has transfer fee, and we will take use of the its wrap token address.

so, for token with transfer fee, we should fill **TokenInfoFormatted.wrapTokenAddress** with corresponding **Wrap Token** address.
for token with no transfer fee, we should set **wrapTokenAddress** with undefined.

5. pre compute amount of withdrawed token
------------------------------------------------------------------

we use sdk's interface **getWithdrawLiquidityValue** to precompute amount of withdrawed token.

the returns and params of this interface can be seen as following code.

.. code-block:: typescript
    :linenos:

    export const getWithdrawLiquidityValue = (
        liquidity: Liquidity,
        state: BaseState,
        withdrawLiquidity: BigNumber
    ): {amountXDecimal: number, amountYDecimal: number, amountX: BigNumber, amountY: BigNumber} 

in the above code, **liquidity** is **Liquidity** obj, which can be obtained from interface **fetchLiquiditiesOfAccount** or **fetchLiquiditiesByTokenIds**.
fetching liquidities can be viewed :ref:`here<fetch_liquidities>`.
**state** is **BaseState** obj, which can be obtained from pool contract.
**withdrawLiquidity** is a BigNumber, describing how much liquidity we want to decrease (or we say withdraw).

to get **liquidity** param, we can call **fetchLiquiditiesByTokenIds**.
but, the interface **getWithdrawLiquidityValue** only use a few fields of **Liquidity**,
so we can easily construct the liquidity obj if we only want to call **getWithdrawLiquidityValue**.

.. code-block:: typescript
    :linenos:

    const tokenId = '121'
    const liquidityRaw = await liquidityManagerContract.methods.liquidities(tokenId).call()

    const liquidity = {
        leftPoint: Number(liquidityRaw.leftPt),
        rightPoint: Number(liquidityRaw.rightPt),
        liquidity: liquidityRaw.liquidity.toString(),
        tokenX: getSwapTokenAddress(feeB).toLowerCase() < getSwapTokenAddress(wBNB).toLocaleLowerCase() ? {...feeB} : {...wBNB},
        tokenY: getSwapTokenAddress(feeB).toLowerCase() > getSwapTokenAddress(wBNB).toLocaleLowerCase() ? {...feeB} : {...wBNB},
	} as Liquidity

here, we only fill 5 fields in **Liquidity**, because **getWithdrawLiquidityValue** only read those 5 fields of **Liquidity** object.
we should notice that, when we fill tokenX and tokenY, we should ensure **getSwapTokenAddress(tokenX).toLowerCase() < getSwapTokenAddress(tokenY).toLowerCase()**

to get **state** param, we need to query the pool

.. code-block:: typescript
    :linenos:

    const liquidityManagerAddress = '0x6bEae78975e561fDF27AaC6f09F714E69191DcfD'
    const liquidityManagerContract = getLiquidityManagerContract(liquidityManagerAddress, web3)

    const poolAddress = await getPoolAddress(liquidityManagerContract, feeB, wBNB, fee)
    const pool = getPoolContract(poolAddress, web3)

    const state = await getPoolState(pool)

in the above code, **state** is an object of **State**, which inherits from **BaseState**.

then, we determine how much liquidity to decrease.

.. code-block:: typescript
    :linenos:

    const liquidityDelta = new BigNumber(liquidity.liquidity).div(10)

after above steps, we can call **getWithdrawLiquidityValue** to pre compute withdrawed tokens.

.. code-block:: typescript
    :linenos:

    const {amountX, amountY} = getWithdrawLiquidityValue(liquidity, state, liquidityDelta)

in the above code, **amountX** is undecimal amount of withdrawed **tokenX**, **amountY** is undecimal amount of withdrawed **tokenY**

to determine amount of **FeeB** and amount of **wBNB**, we should determine which token is tokenX and which is tokenY.
etc, compare dictionary order of 2 token address.

.. code-block:: typescript
    :linenos:

    const amountFeeB = getSwapTokenAddress(feeB).toLowerCase() < getSwapTokenAddress(wBNB).toLocaleLowerCase() ? amountX : amountY
    const amountWBNB = getSwapTokenAddress(feeB).toLowerCase() < getSwapTokenAddress(wBNB).toLocaleLowerCase() ? amountY : amountX


6. construct params and get calling
------------------------------------------------------------------

.. code-block:: typescript
    :linenos:

    const decLiquidityAndCollectParams = {
        tokenId,
        tokenA: wBNB,
        tokenB: feeB,
        liquidDelta: liquidityDelta.toFixed(0),
        minAmountA: minAmountWBNB.times(0.98).toFixed(0),
        minAmountB: minAmountFeeB.times(0.98).toFixed(0),
    } as DecLiquidityAndCollectParams

    const gasPrice = '15000000000'

    const { calling, options } = getDecLiquidityAndCollectCall(
        boxContract,
        account.address,
        chain,
        decLiquidityAndCollectParams,
        gasPrice
    )

in the above code, notice the field **addLiquidityParams.minAmountA** and **addLiquidityParams.minAmountB**.
we fill these fields with **"MaxValue" * 0.98**, which are significantly higher than that in :ref:`box mint params <box_mint_params>` or :ref:`box add liquidity params <box_add_liquidity_params>`.
in those examples, user mint or add through **box** paying "transfer fee" token directly, the transform from origin token to wrap token is done within the box contract's interfaces.
transform from origin token (which has transfer fee) to wrap token will cause transfer fee,
and amount of wrap token is amount of origin token minus transfer fee.
when the box contract call **liquidityManager**'s mint or addLiquidity interfaces to mint for user, it deposits wrap token.
and due to the fact that amount of wrap token is less than amount of origin token, the actual amount deposited will be reduced. 
to pass the checking of **minAmountA** and **minAmountB** in **liquidityManager**, we fill these 2 fields with lower value **"MaxValue * 0.8"**.
but in this case, when user decrease and collect, the token withdrawed from iZiSwap is wrap token, not origin token (which has transfer fee).
no transfer fee occured inside box's calling of liquidityManger, so higher value **"MaxValue * 0.8"** can pass checking of **liquidityManager**.

for more detail you can read corresponding contract code.

7.  estimate gas (optional)
---------------------------
of course you can skip this step if you donot want to limit gas.

notice that you should should approve box to operate your liquidity nft before estimate gas or send transaction,
because **box** will call **liquidityManager** to decrease and collect your nft liquidity, the box need your approve.
you can view interfaces corresponding to approve or approval in erc721's interfaces for more information.

.. code-block:: typescript
    :linenos:

    const gasLimit = await calling.estimateGas(options)

8.  finally, send transaction!
------------------------------

notice that you should should approve box to operate your liquidity nft before estimate gas or send transaction,
because **box** will call **liquidityManager** to decrease and collect your nft liquidity, the box need your approve.
you can view interfaces corresponding to approve or approval in erc721's interfaces for more information.

for metamask or other explorer's wallet provider, you can easily write 

.. code-block:: typescript
    :linenos:

    await calling.send({...options, gas: gasLimit})

otherwise, if you are runing codes in console, you could use following code

.. code-block:: typescript
    :linenos:

    // sign transaction
    const signedTx = await web3.eth.accounts.signTransaction(
        {
            ...options,
            to: boxAddress,
            data: calling.encodeABI(),
            gas: new BigNumber(gasLimit * 1.1).toFixed(0, 2),
        }, 
        privateKey
    )
    // send transaction
    const tx = await web3.eth.sendSignedTransaction(signedTx.rawTransaction);

after this step, we have successfully add liquidity on existing liqudity through **Box** (if no revert occured)