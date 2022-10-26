add liquidity
================================

here, we provide a simple example for adding some liquidity to existing liquidity through box's interface


1. some imports
---------------

.. code-block:: typescript
    :linenos:

    import {BaseChain, ChainId, initialChainTable, PriceRoundingType, TokenInfoFormatted} from 'iziswap-sdk/lib/base/types'
    import {privateKey} from '../../.secret'
    import Web3 from 'web3';
    import { getPointDelta, getPoolContract, getPoolState } from 'iziswap-sdk/lib/pool/funcs';
    import { amount2Decimal, fetchToken } from 'iziswap-sdk/lib/base/token/token';
    import { pointDeltaRoundingDown, pointDeltaRoundingUp, priceDecimal2Point } from 'iziswap-sdk/lib/base/price';
    import { BigNumber } from 'bignumber.js'
    import { calciZiLiquidityAmountDesired, getLiquidityManagerContract, getPoolAddress } from 'iziswap-sdk/lib/liquidityManager';
    import { getBoxContract, getAddLiquidityCall, AddLiquidityParams } from 'iziswap-sdk/lib/box';

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

.. _BoxContract_forAdd:

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

.. _box_add_liquidity_params:

5. determine params for adding liquidity
------------------------------------------------------------------

first, to compute amount of mint token, we need current point (price) of swap pool.

.. code-block:: typescript
    :linenos:

    const liquidityManagerAddress = '0x6bEae78975e561fDF27AaC6f09F714E69191DcfD'
    const liquidityManagerContract = getLiquidityManagerContract(liquidityManagerAddress, web3)

    const poolAddress = await getPoolAddress(liquidityManagerContract, feeB, wBNB, fee)
    const pool = getPoolContract(poolAddress, web3)

    const state = await getPoolState(pool)

**state.currentPoint** is current point we want.

secondly, we need to know **leftPoint** and **rightPoint** of the liquidity, and in the example of :ref:`box mint<box_mint>`.
the left point and right point of that minned liquidity is following

.. code-block:: typescript
    :linenos:

    const leftPoint = 4680
    const rightPoint = 8760

thirdly, we determine to pay 1.0 feeB, set **AddLiquidityParams**

.. code-block:: typescript

    
    const maxFeeB = new BigNumber(1).times(10 ** feeB.decimal)
    const maxWBNB = calciZiLiquidityAmountDesired(
        leftPoint, rightPoint, state.currentPoint,
        maxFeeB, true, feeB, wBNB
    )

    const maxWBNBDecimal = amount2Decimal(maxFeeB, feeB)

    const addLiquidityParams = {
        tokenId: '121',
        tokenA: wBNB,
        tokenB: feeB,
        fee,
        leftPoint,
        rightPoint,
        maxAmountA: maxWBNB.toFixed(0),
        maxAmountB: maxFeeB.toFixed(0),
        minAmountA: maxWBNB.times(0.8).toFixed(0),
        minAmountB: maxFeeB.times(0.8).toFixed(0),
    } as AddLiquidityParams

in the above code, notice the field **addLiquidityParams.minAmountA** and **addLiquidityParams.minAmountB**.
we fill these fields with **"MaxValue" * 0.8**, which are significantly lower than that in :ref:`another mint example <liquidity_manager_mint_calling>`.
in that mint example, user mint directly through **liquidityManager**, and cannot mint with "transfer fee" token, so we fill them with higher value **"MaxValue" * 0.985"**.
but in this case, token **FeeB** will charge transfer fee when we mint with **FeeB** through **Box**.
So we select values to fill **mintParams.minAmountA** and **mintParams.minAmountB**.

6. get add liquidity calling
-----------------------------------

.. code-block:: typescript
    :linenos:

    const gasPrice = '15000000000'

    const { addLiquidityCalling, options } = getAddLiquidityCall(
        boxContract,
        account.address,
        chain,
        addLiquidityParams,
        gasPrice
    )

in the above code, function **getAddLiquidityCall** returns 2 object, **addLiquidityCalling** and **options**

after acquiring **addLiquidityCalling** and **options**, we can estimate gas

7.  estimate gas (optional)
---------------------------
of course you can skip this step if you donot want to limit gas.

notice that you should do following steps before estimate gas or send transaction in this "add liquidity" case.

first, you should should approve box to operate your liquidity nft before estimate gas or send transaction,
because **box** will call **liquidityManager** to add some liquidity to your nft liquidity, the box need your approve.
you can view interfaces corresponding to approve or approval in erc721's interfaces for more information.

second, you should approve box to deposit your **FeeB** token to corresponding **WrapToken**, 
because box will call **deposit** interface of **WrapToken** to help you deposit your **FeeB**, the box needs your approve.
you can view **depositApprove** interface of **WrapToken** contract for more information.

third, you should approve **WrapToken** to transfer your **FeeB** token, because in **deposit** interface of **WrapToken**,
the **WrapToken** contract call transfer interface of **FeeB** to transfer your **FeeB** token, and **WrapToken** needs your approve.

forthly, if the token pair is "FeeB-USDT" or "FeeB-iZi" or FeeB with other normal erc20 token instead of wbnb/weth,
you should approve **Box** to transfer your corresponding erc20 token,
you can view interfaces corresponding to approve or approval in erc20's interfaces for more information.

after above steps, you can estimate or send the transaction

.. code-block:: typescript
    :linenos:

    const gasLimit = await addLiquidityCalling.estimateGas(options)

8.  finally, send transaction!
------------------------------

notice that you should do following steps before estimate gas or send transaction in this "add liquidity" case.

first, you should should approve box to operate your liquidity nft before estimate gas or send transaction,
because **box** will call **liquidityManager** to add some liquidity to your nft liquidity, the box need your approve.
you can view interfaces corresponding to approve or approval in erc721's interfaces for more information.

second, you should approve box to deposit your **FeeB** token to corresponding **WrapToken**, 
because box will call **deposit** interface of **WrapToken** to help you deposit your **FeeB**, the box needs your approve.
you can view **depositApprove** interface of **WrapToken** contract for more information.

third, you should approve **WrapToken** to transfer your **FeeB** token, because in **deposit** interface of **WrapToken**,
the **WrapToken** contract call transfer interface of **FeeB** to transfer your **FeeB** token, and **WrapToken** needs your approve.

forthly, if the token pair is "FeeB-USDT" or "FeeB-iZi" or FeeB with other normal erc20 token instead of wbnb/weth,
you should approve **Box** to transfer your corresponding erc20 token,
you can view interfaces corresponding to approve or approval in erc20's interfaces for more information.

after above steps, you can estimate or send the transaction

for metamask or other explorer's wallet provider, you can easily write 

.. code-block:: typescript
    :linenos:

    await addLiquidityCalling.send({...options, gas: gasLimit})

otherwise, if you are runing codes in console, you could use following code

.. code-block:: typescript
    :linenos:

    // sign transaction
    const signedTx = await web3.eth.accounts.signTransaction(
        {
            ...options,
            to: boxAddress,
            data: addLiquidityCalling.encodeABI(),
            gas: new BigNumber(gasLimit * 1.1).toFixed(0, 2),
        }, 
        privateKey
    )
    // send transaction
    const tx = await web3.eth.sendSignedTransaction(signedTx.rawTransaction);

after this step, we have successfully add liquidity on existing liqudity through **Box** (if no revert occured)