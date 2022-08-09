collect
================================

here, we provide a simple example for collecting earned or decreased token from an existing liquidity through box's interface


1. some imports
---------------

.. code-block:: typescript
    :linenos:

    import {BaseChain, ChainId, initialChainTable, TokenInfoFormatted} from 'iziswap-sdk/lib/base/types'
    import {privateKey} from '../../.secret'
    import Web3 from 'web3';
    import { BigNumber } from 'bignumber.js'
    import { getBoxContract, CollectLiquidityParams, getCollectCall } from 'iziswap-sdk/lib/box';

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

.. _BoxContract_forCollect:

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

5. determine params for collecting
------------------------------------------------------------------

.. code-block:: typescript
    :linenos:

    const collectLiquidityParams = {
        tokenId: '121',
        tokenA: wBNB,
        tokenB: feeB,
        maxAmountA: '1000000000000000000',
        maxAmountB: '1000000000000000000',
    } as CollectLiquidityParams

in the above code, field **tokenId** is nft id of liquidity you want to collect, **maxAmountA** and **maxAmountB** describe maximum amount (undecimal) of token you want to collect.

6. get collect calling
-----------------------------------

.. code-block:: typescript
    :linenos:

    const gasPrice = '15000000000'

    const { collectCalling, options } = getCollectCall(
        boxContract,
        account.address,
        chain,
        collectLiquidityParams,
        gasPrice
    )

in the above code, function **getCollectCall** returns 2 object, **collectCalling** and **options**

after acquiring **collectCalling** and **options**, we can estimate gas

7.  estimate gas (optional)
---------------------------

of course you can skip this step if you donot want to limit gas.

notice that you should should approve box to operate your liquidity nft before estimate gas or send transaction,
because **box** will call **liquidityManager** to decrease and collect your nft liquidity, the box need your approve.
you can view interfaces corresponding to approve or approval in erc721's interfaces for more information.

.. code-block:: typescript
    :linenos:

    const gasLimit = await collectCalling.estimateGas(options)

8.  finally, send transaction!
------------------------------

notice that you should should approve box to operate your liquidity nft before estimate gas or send transaction,
because **box** will call **liquidityManager** to decrease and collect your nft liquidity, the box need your approve.
you can view interfaces corresponding to approve or approval in erc721's interfaces for more information.

for metamask or other explorer's wallet provider, you can easily write 

.. code-block:: typescript
    :linenos:

    await collectCalling.send({...options, gas: gasLimit})

otherwise, if you are runing codes in console, you could use following code

.. code-block:: typescript
    :linenos:

    // sign transaction
    const signedTx = await web3.eth.accounts.signTransaction(
        {
            ...options,
            to: boxAddress,
            data: collectCalling.encodeABI(),
            gas: new BigNumber(gasLimit * 1.1).toFixed(0, 2),
        }, 
        privateKey
    )
    // send transaction
    const tx = await web3.eth.sendSignedTransaction(signedTx.rawTransaction);

after this step, we have successfully add liquidity on existing liqudity through **Box** (if no revert occured)