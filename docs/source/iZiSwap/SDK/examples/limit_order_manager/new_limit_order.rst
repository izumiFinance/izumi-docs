.. _new_limit_order:

new limit order
================================

here, we provide a simple example for creating a new limit order


1. some imports
---------------

.. code-block:: typescript
    :linenos:

    import {BaseChain, ChainId, initialChainTable, PriceRoundingType } from 'iziswap-sdk/lib/base/types'
    import {privateKey} from '../../.secret'
    import Web3 from 'web3';
    import { decimal2Amount, fetchToken, getSwapTokenAddress } from 'iziswap-sdk/lib/base/token/token'
    import { getDeactiveSlot, getLimitOrderManagerContract, getPoolAddress } from 'iziswap-sdk/lib/limitOrder/view';
    import { getNewLimOrderCall } from 'iziswap-sdk/lib/limitOrder/limitOrder';
    import { AddLimOrderParam } from 'iziswap-sdk/lib/limitOrder/types'
    import { pointDeltaRoundingDown, pointDeltaRoundingUp, priceDecimal2Point } from 'iziswap-sdk/lib/base/price'
    import { BigNumber } from 'bignumber.js'
    import { getPointDelta, getPoolContract } from 'iziswap-sdk/lib/pool/funcs';

the detail of these imports can be viewed in following content

.. _limit_order_base_obj_mint:

2. specify which chain, rpc url, web3, and account
--------------------------------------------------

.. code-block:: typescript
    :linenos:

    const chain:BaseChain = initialChainTable[ChainId.BSC]
    const rpc = 'https://bsc-dataseed2.defibit.io/'
    const web3 = new Web3(new Web3.providers.HttpProvider(rpc))
    const account =  web3.eth.accounts.privateKeyToAccount(privateKey)
    const accountAddress = account.address

here

**BaseChain** is a data structure to describe a chain, in this example we use **bsc** chain.

**ChainId** is an enum to describe **chain id**, value of the enum is equal to value of **chain id**

**initialChainTable** is a mapping from some most used **ChainId** to **BaseChain**, of course you can fill fields of BaseChain by yourself

**privateKey** is a string, which is your private key, and should be configured by your self

**web3** package is a public package to interact with block chain

**rpc** is the rpc url on the chain you specified

.. _LimitOrderManagerContract_forNew:

3. get web3.eth.Contract object of limitOrderManager
----------------------------------------------------

.. code-block:: typescript
    :linenos:

    const limitOrderAddress = '0x9Bf8399c9f5b777cbA2052F83E213ff59e51612B'
    const limitOrderManager = getLimitOrderManagerContract(limitOrderAddress, web3)

here, **getLiquidityManagerContract** is an api provided by our sdk, which returns a **web3.eth.Contract** object of **LiquidityManager**

4. fetch 2 erc20-tokens' infomations and specify selltoken,earntoken and fee
----------------------------------------------------------------------------

.. code-block:: typescript
    :linenos:

    const testAAddress = '0xCFD8A067e1fa03474e79Be646c5f6b6A27847399'
    const testBAddress = '0xAD1F11FBB288Cd13819cCB9397E59FAAB4Cdc16F'

    const testA = await fetchToken(testAAddress, chain, web3)
    const testB = await fetchToken(testBAddress, chain, web3)
    const fee = 2000 // 2000 means 0.2%

    const sellToken = testA
    const earnToken = testB

**fetchToken()** returns a **TokenInfoFormatted** obj of that token, which containing following fields,

of course you can fill TokenInfoFormatted by yourself if you have known each field correctly of the erc20-token
the TokenInfoFormatted fields used in sdk currently are only **symbol**, **address**, and **decimal**.

.. code-block:: typescript
    :linenos:

    export interface TokenInfoFormatted {
        // chain id of chain
        chainId: number;
        // name of token
        name: string;
        // symbol of token
        symbol: string;
        // img url, not necessary for sdk, you can fill any string or undefined
        icon: string;
        // address of token
        address: string;
        // decimal value of token, acquired by calling 'decimals()'
        decimal: number;
        // not necessary for sdk, you can fill any date or undefined
        addTime?: Date;
        // not necessary for sdk, you can fill either true/false/undefined
        custom: boolean;
        // this field usually undefined.
        // wrap token address of this token if this token has transfer fee.
        // this field only has meaning when you want to use sdk of box to deal with problem of transfer fee
        wrapTokenAddress?: string;
    }

notice that, usually we set **TokenInfoFormatted.wrapTokenAddress** as undefined.

following paragraph corresponding to box and wrap token you can just **skip** it if you do not consider token with transfer fee.

only if we want to use **box** and the token has transfer fee, we should set the **wrapTokenAddress** field.
if we donot want to use **box** or the token has no transfer fee, **TokenInfoFormatted.wrapTokenAddress** should be undefined.
:ref:`box<box>` is designed to deal with problem of erc20 token with ":ref:`transfer fee<transfer_fee>`".
there is a problem that in iZiSwap we can not mint or trade or add limit order with tokens which have transfer fee.
to deal with this problem, we can deploy a :ref:`Wrap Token<wrap_token>` which can be transformed from origin erc20 token.
wrap token has no transfer fee, transfer fee only charged when user transform origin token to wrap token or wrap token to origin token.
and we can mint or add limit order or trade with such wrap tokens instead of origin token in iZiSwap.
for sdk of box, see :ref:`here<box>` for more infomation.


5. compute sellPoint (price) and sell amount
---------------------------------------------------------

first set decimal price, and transform the decimal price to point on the pool


.. code-block:: typescript
    :linenos:

    const sellPriceDecimalAByB = 0.25
    const sellPoint = priceDecimal2Point(sellToken, earnToken, sellPriceDecimalAByB, PriceRoundingType.PRICE_ROUNDING_UP)
    
secondly, query pool contract to get pointDelta, sell point of limit order must be times of pointDelta.

.. code-block:: typescript
    :linenos:

    const poolAddress = await getPoolAddress(limitOrderManager, testA, testB, fee)
    const pool = getPoolContract(poolAddress, web3)
    const pointDelta = await getPointDelta(pool)

thirdly, compute sellPoint rounding to times of pointDelta.

.. code-block:: typescript
    :linenos:

    const state = await getPoolState(pool)
    let sellPointRoundingPointDelta = sellPoint
    if (getSwapTokenAddress(sellToken).toLowerCase() < getSwapTokenAddress(earnToken).toLowerCase()) {
        sellPointRoundingPointDelta = pointDeltaRoundingDown(sellPointRoundingPointDelta, pointDelta)
    } else {
        sellPointRoundingPointDelta = pointDeltaRoundingUp(sellPointRoundingPointDelta, pointDelta)
    }
    const sellAmountDecimal = 1000
    const sellAmount = decimal2Amount(sellAmountDecimal, testA).toFixed(0)

We should notice that, if sellToken is tokenX (etc. sellToken < earnToken), the sell point should be
greater than or equal to current point. otherwise, sell point should be less than or equal to current point.

6.  get newLimitOrder calling
---------------------------------------------------------

when we send a transaction calling limit order manager to add a new limit order, we should specify an empty slot idx.

which is obtained by calling **getDeactiveSlot** before sending this transaction, like following code.

.. code-block:: typescript
    :linenos:

    const slotIdx = await getDeactiveSlot(limitOrderManager, accountAddress)


then, we can fill **AddLimOrderParam** obj, which will be the parameter to interface of creating limit order

.. code-block:: typescript

    const params : AddLimOrderParam = {
        idx: slotIdx,
        sellToken,
        earnToken,
        fee,
        point: sellPointRoundingPointDelta,
        sellAmount
    }

the field of **AddLimOrderParam** is displayed in following code

.. code-block:: typescript

    export interface AddLimOrderParam {
        // slotIdx, to specify an empty slot on contract to store your limit order
        idx: string,
        // which token to sell
        sellToken: TokenInfoFormatted,
        // which token to earn
        earnToken: TokenInfoFormatted,
        // fee of token pair (swap pool)
        fee: number,
        // sell point computed
        point: number,
        // undecimal amount of sell token you want to sell
        sellAmount: string,
        deadline?: string,
        // only sellToken is WBNB/WETH or other wrapped chain token (erc20 form), this field has meaning
        // if strictERC20Token is true, you will provide sellToken from your existing wrapped chain token (erc20 form)
        // and msg.value can be 0.
        // if this field is false, msg.value should not be smaller than sellAmount, and the LimitOrderManager contract
        // will transform your provided bnb/eth or other chain token to wrapped chain token form (erc20 form). 
        strictERC20Token?: boolean
    }

thirdly, call **getNewLimOrderCall** to get calling and options obj

.. code-block:: typescript
    :linenos:

    const {newLimOrderCalling, options} = getNewLimOrderCall(
        limitOrderManager, 
        accountAddress, 
        chain, 
        params,
        gasPrice
    )

7. approve
---------------

before send transaction or estimate gas, you need to approve contract limitOrderManager to have authority to spend your token,
because you need transfer some sellToken to pool.

.. code-block:: typescript
    :linenos:

    // the approve interface abi of erc20 token
    const erc20ABI = [{
      "inputs": [
        {
          "internalType": "address",
          "name": "spender",
          "type": "address"
        },
        {
          "internalType": "uint256",
          "name": "amount",
          "type": "uint256"
        }
      ],
      "name": "approve",
      "outputs": [
        {
          "internalType": "bool",
          "name": "",
          "type": "bool"
        }
      ],
      "stateMutability": "nonpayable",
      "type": "function"
    }];
    // if sellToken is not chain token (BNB on bsc chain or ETH on eth chain...), we need transfer tokenA to pool
    // otherwise we can skip following codes
    {
        const sellTokenContract = new web3.eth.Contract(erc20ABI, sellToken.address);
        // you could approve a very large amount (much more greater than amount to transfer),
        // and don't worry about that because limitOrderManager only transfer your token to pool with amount you specified and your token is safe
        // then you do not need to approve next time for this user's address
        const approveCalling = sellTokenContract.methods.approve(
            limitOrderAddress, 
            "0xffffffffffffffffffffffffffffffff"
        );
        // estimate gas
        const gasLimit = await mintCalling.estimateGas({from: account})
        // then send transaction to approve
        // you could simply use followiing line if you use metamask in your frontend code
        // otherwise, you should use the function "web3.eth.accounts.signTransaction"
        // notice that, sending transaction for approve may fail if you have approved the token to limitOrderManager before
        // if you want to enlarge approve amount, you should refer to interface of erc20 token
        await approveCalling.send({gas: gasLimit})
    }

8.  estimate gas (optional)
---------------------------
of course you can skip this step if you don't want to limit gas.
before estimate gas and send transaction, make sure you have approve limitOrderAddress of sellToken

.. code-block:: typescript
    :linenos:

    // before estimate gas and send transaction, 
    // make sure you have approve limitOrderAddress of sellToken
    const gasLimit = await newLimOrderCalling.estimateGas(options)

9. finally, send transaction!
------------------------------

for metamask or other explorer's wallet provider, you can easily write 

.. code-block:: typescript
    :linenos:

    await newLimOrderCalling.send({...options, gas: gasLimit})

otherwise, if you are runing codes in console, you could use following code

.. code-block:: typescript
    :linenos:

    const gasPrice = '5000000000'
    const signedTx = await web3.eth.accounts.signTransaction(
        {
            ...options,
            to: limitOrderAddress,
            data: newLimOrderCalling.encodeABI(),
            gas: new BigNumber(gasLimit * 1.1).toFixed(0, 2),
        }, 
        privateKey
    )
    // nonce += 1;
    const tx = await web3.eth.sendSignedTransaction(signedTx.rawTransaction);

after this step, we have successfully minted the liquidity (if no revert occurred).