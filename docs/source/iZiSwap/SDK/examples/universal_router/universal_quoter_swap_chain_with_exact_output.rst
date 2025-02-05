
.. _universal_quoter_swap_chain_with_exact_output:

Exact output mode
===================================================

In this example, we will conduct price query and swap with exact output amount in iZiSwap (Desire Mode)
via UniversalQuoter and UniversalSwapRouter.

Here, **quoter with exact output amount** means pre-query amount of token need to pay given exact amount of desired token. For example, if you want to swap N ETH to 2000 USDC, 
the step is used to determine N.

**swap with exact output amount** means to invoke the real swap with the desired amount.

Quoter and swap are called throw 2 different contracts.

Suppose we want to swap test **USDT** token to test **BNB** token, where the test **USDT** is standard ERC-20 token deployed by us on BSC testnet and 
the test **BNB** is native token on BSC testnet.

The full example codes can be found `in this link <https://github.com/izumiFinance/izumi-iZiSwap-sdk/blob/main/example/universalRouter/quoterSwapExactOutput.ts>`_.

1. Some imports
-----------------------------------------------------------

These are some basic modules used in most examples.

.. code-block:: typescript
    :linenos:

    import {BaseChain, ChainId, initialChainTable, TokenInfoFormatted} from 'iziswap-sdk/lib/base/types'
    import {privateKey} from '../../.secret'
    import Web3 from 'web3';
    import { amount2Decimal } from 'iziswap-sdk/lib/token/token';
    import { BigNumber } from 'bignumber.js'

And these are some imports for UniversalQuoter and UniversalSwapRouter.

.. code-block:: typescript
    :linenos:

    import { getSwapExactOutputCall, getUniversalQuoterContract, getUniversalSwapRouterContract, quoteExactOutput } from 'iziswap-sdk/lib/universalRouter'
    import { SwapExactOutputParams } from 'iziswap-sdk/lib/universalRouter/types';

Here **quoteExactOutput** will return the amount of input token needed to pay (N), and
**getSwapExactOutputCall** will return calling for swapping with exact amount of output (or we say desired) token.


.. _universal_quoter_swap_chain_with_exact_output_initialization:

2. Initialization
------------------------------------------------------------------

Following codes define some basic object like rpc, web3 and account
which we may need. The network in this example is bsc testnet.

.. code-block:: typescript
    :linenos:

    const chain:BaseChain = initialChainTable[ChainId.BSCTestnet]
    // const rpc = 'https://bsc-dataseed2.defibit.io/'
    const rpc = 'https://bsc-testnet-rpc.publicnode.com';
    console.log('rpc: ', rpc)
    const web3 = new Web3(new Web3.providers.HttpProvider(rpc))
    const account =  web3.eth.accounts.privateKeyToAccount(privateKey)
    console.log('address: ', account.address)

Following codes define our **UniversalQuoter** contract object.

.. code-block:: typescript
    :linenos:

    const quoterAddress = '0xA55D57C62A1B998E4bDD956c31096b783b7ce1cF'
    const quoterContract = getUniversalQuoterContract(quoterAddress, web3)

    console.log('quoter address: ', quoterAddress)


Following codes define some test tokens deployed by us on bsc testnet.

.. code-block:: typescript
    :linenos:

    const USDC = {
        chainId: chain.id,
        symbol: 'USDC',
        address: '0x876508837C162aCedcc5dd7721015E83cbb4e339',
        decimal: 6
    } as TokenInfoFormatted;

    const USDT = {
        chainId: chain.id,
        symbol: 'USDT',
        address: '0x6AECfe44225A50895e9EC7ca46377B9397D1Bb5b',
        decimal: 6
    } as TokenInfoFormatted;

    const BNB = {
        chainId: chain.id,
        symbol: 'BNB', 
        address: '0xae13d989daC2f0dEbFf460aC112a837C89BAa7cd',
        decimal: 18,
    } as TokenInfoFormatted;

    const WBNB = {
        chainId: chain.id,
        symbol: 'WBNB', 
        address: '0xae13d989daC2f0dEbFf460aC112a837C89BAa7cd',
        decimal: 18,
    } as TokenInfoFormatted;


And following codes define params for universal quoter.

.. code-block:: typescript
    :linenos:

    const outputAmountDecimal = 0.2;
    const outputAmount = new BigNumber(outputAmountDecimal).times(10 ** WBNB.decimal).toFixed(0)
    
    const quoterParams = {
        // note: 
        //     if you want to pay via native BNB/WBNB,
        //         just change first token in tokenChain to BNB/WBNB
        //         like [BNB, ... other tokens] or [WBNB, ... other tokens]
        //     and if you want to buy BNB/WBNB,
        //         you can just change the last token in tokenChain 
        //         to BNB/WBNB
        //         like [... other tokens, BNB] or [... other tokens, WBNB]
        //     Both of BNB and WBNB are defined in code above
        // tokenChain: [BNB, USDC, USDT],
        tokenChain: [USDT, USDC, BNB],

        // fee percent of pool(tokenChain[i], tokenChain[i+1]) 
        // 0.3 means fee tier of 0.3%
        //     only need for V3Pool
        //     for V2Pool, you can fill arbitrary value
        feeTier: [0.04, 0],
        // isV2[i] == true, means pool(tokenChain[i], tokenChain[i+1]) is a V2Pool
        // otherwise, the corresponding pool is V3Pool
        //     same length as feeTier
        isV2: [false, true],

        outputAmount,
        // "maxInputAmount" is not used in quoter
        //     and you can fill arbitrary value in quoter
        maxInputAmount: '0',
        // outChargeFeeTier% of trader's acquired token (outToken) 
        // will be additionally charged by universalRouter
        // if outChargeFeeTier is 0.2, 0.2% of outToken will be additionally charged
        // if outChargeFeeTier is 0, no outToken will be additionally charged
        // outChargeFeeTier should not be greater than 5 (etc, 5%)
        outChargeFeeTier: 0.2,
    } as SwapExactOutputParams;

In the above code, we ready to buy **0.2** test BNB.
And then, we can see 3 arrays in **quoterParams**, tokenChain, feeTier, and isV2.
These 3 lists together define the path we want to do price inquiry.
We can see that there are 2 pools in our path.
The first pool on the path is a V3-pool with pair of **(USDT, USDC, 0.04%)**, here 0.04% is the fee rate of this pool.
The second pool on the path is a V2-pool with pair of **(USDC, BNB)**.


The fields of **SwapExactOutputParams** is explained in the following code.

.. code-block:: typescript
    :linenos:
    

    export interface SwapExactOutputParams {
        // input: tokenChain.first()
        // output: tokenChain.last()
        tokenChain: TokenInfoFormatted[];
        // fee percent of pool(tokenChain[i], tokenChain[i+1]) 0.3 means 0.3%
        //     only need for V3Pool
        //     for V2Pool, you can fill arbitrary value
        feeTier: number[];
        // isV2[i] == true, means pool(tokenChain[i], tokenChain[i+1]) is a V2Pool
        // otherwise, the corresponding pool is V3Pool
        //     same length as feeTier
        isV2: boolean[];
        // 10-decimal format number, like 100, 150000, ...
        // or hex format number start with '0x'
        // amount = outputAmount / (10 ** outputToken.decimal)
        outputAmount: string;
        maxInputAmount: string;
        // outChargeFeeTier% of trader's acquired token (outToken) 
        // will be additionally charged by universalRouter
        // if outChargeFeeTier is 0.2, 0.2% of outToken will be additionally charged
        // if outChargeFeeTier is 0, no outToken will be additionally charged
        // outChargeFeeTier should not be greater than 5 (etc, 5%)
        outChargeFeeTier: number;
        recipient?: string;
        deadline?: string;
    }

**iZiSwap**'s UniversalQuoter and UniversalSwap contracts support swap chain with multi V2 and V3 swap pools.
For example, if you have some token0, and wants to get token3 through the path
**token0 -> (token0, token1, 0.05%) -> token1 -> (token1, token2, 0.3%) -> token2 -> (token2, token3) -> token3** 
(here, suppose the last pair **(token2, token3)** is a V2-pool),
you should fill the **tokenChain**, **feeTier** and **isV2** fields with following code


.. code-block:: typescript
    :linenos:

    // here, token0..3 are TokenInfoFormatted
    params.tokenChain = [token0, token1, token2, token3]
    params.feeChain = [0.05, 0.3, 0]
    params.isV2 = [false, false, true]


*In general, the supported fee rates for V3-pools on the mainnet are 500 (0.05%), 3000 (0.3%), and 10000 (1%); and for the testnet are 400 (0.04%), 2000 (0.2%) and 10000 (1%). One needs to check if the choosen pool exists and has enough liquidity.*
*The liquidity condition can be checked on the analytics page* `here <https://analytics.izumi.finance>`_ .

.. _universal_quoter_swap_chain_with_exact_output_wrapped_or_native:

3. Exchange With Wrapped Native or Native token
--------------------------------------------------------------------

In sdk-interfaces of UniversalQuoter and UniversalSwapRouter, 
if you want to pay or buy **wrapped native** or **native** token,
just simply set **tokenChain.first()** or **tokenChain.last()** as **wrapped native** or **native** token.
And we can also found that the only difference between **wrapped native** and **native** token is **symbol** field in **TokenInfoFormatted**.

If you want to pay test **BNB** to buy test **USDT**, you can use following code.
And in the following code, params is an instance of **SwapExactInputParams**

.. code-block:: typescript
    :linenos:

    // objects in tokenChain are all TokenInfoFormatted
    // BNB and USDT are defined above in section 2.
    params.tokenChain = [BNB, ... /* other mid tokens*/, USDT]

"BNB" and "USDT" in above code can be defined by following code (or in example code of section 2).
And the **options** in :ref:`this section<get_universal_exact_in_calling>` will contain corresponding **msg.value**.

.. code-block:: typescript
    :linenos:

    const USDT = {
        chainId: chain.id,
        symbol: 'USDT',
        address: '0x6AECfe44225A50895e9EC7ca46377B9397D1Bb5b',
        decimal: 6
    } as TokenInfoFormatted;

    const BNB = {
        chainId: chain.id,
        symbol: 'BNB', 
        address: '0xae13d989daC2f0dEbFf460aC112a837C89BAa7cd',
        decimal: 18,
    } as TokenInfoFormatted;


If you want to pay test **USDT** to buy test **WBNB**, you can use following code.

.. code-block:: typescript
    :linenos:

    // objects in tokenChain are all TokenInfoFormatted
    // WBNB and USDT are defined above in section 2.
    params.tokenChain = [USDT, ... /* other mid tokens*/, WBNB]

"WBNB" in above code can be defined by following code (or in example code of section 2)

.. code-block:: typescript
    :linenos:

    const WBNB = {
        chainId: chain.id,
        symbol: 'WBNB',  // here the only difference with "BNB"
        address: '0xae13d989daC2f0dEbFf460aC112a837C89BAa7cd',
        decimal: 18,
    } as TokenInfoFormatted;

more detail can be viewed in the code comment in :ref:`section 2<universal_quoter_swap_chain_with_exact_output_initialization>`.


4. Out Token FeeTier
-----------------------------------------------------------

In the code in :ref:`section 2<universal_quoter_swap_chain_with_exact_output_initialization>`,
we can notice that the object **quoterParams** has a field named **outChargeFeeTier**.

This field specify the fee tier to charge from **out token**, in this example, the **out token** is test **BNB**.

If you specify **outChargeFeeTier** as 0.2, 0.2% of **out token** will be charged before sending to trader.

If you want 0.3% of out token is charged, you can use following code.

.. code-block:: typescript
    :linenos:

    params.outChargeFeeTier = 0.3


.. _universal_quoter_swap_chain_with_exact_output_query:

5. Use UniversalQuoter to pre-query amount of test **USDT** need to pay
--------------------------------------------------------------------------

.. code-block:: typescript
    :linenos:

    // whether limit maximum point range for each V3Pool in quoter
    const limit = true; 
    const {inputAmount} = await quoteExactOutput(quoterContract, quoterParams, limit);

    const inputAmountDecimal = amount2Decimal(new BigNumber(inputAmount), USDT)

    console.log(' input amount decimal: ', inputAmountDecimal)
    console.log(' output amount decimal: ', outputAmountDecimal)


In the above code, we are ready to buy **0.2** test BNB (decimal amount, and value of **outputAmountDecimal** has been defined in :ref:`section 2<universal_quoter_swap_chain_with_exact_output_initialization>`). 
We simply call function **quoteExactOutput** to get the amount of test **USDT** need to pay.
The function **quoteExactOutput** need 3 params:

* - **quoterContract**: obtained through **getUniversalQuoterContract** in :ref:`section 2<universal_quoter_swap_chain_with_exact_output_initialization>`
* - a **quoterParams** instance: obtained in :ref:`section 2<universal_quoter_swap_chain_with_exact_output_initialization>`
* - **limit**: a boolean, true if we want to limit point range (no more than 10000) walked through in V3 pools during quoting, and false if we donot limit it.

Now we have finished the Quoter part. 

6. Use UniversalSwap to actually pay test token USDT to get test BNB
-----------------------------------------------------------------------------

First, we use **getSwapContract** to get the Swap contract

.. code-block:: typescript
    :linenos:

    const swapAddress = '0x8684E397A84D718dD65da5938B6985BA60C957c5' // Swap contract on BSC testnet
    const swapContract = getUniversalSwapRouterContract(swapAddress, web3)

Second, use **getSwapExactInputCall** to get calling (transaction handler) of swap:

.. code-block:: typescript
    :linenos:

    const swapParams = {
        ...quoterParams,
        // slippery is 1.5%
        maxInputAmount: new BigNumber(inputAmount).times(1.015).toFixed(0)
    } as SwapExactOutputParams
    
    const gasPrice = '5000000000'

    const {calling: swapCalling, options} = getSwapExactOutputCall(
        swapContract, 
        account.address, 
        chain, 
        swapParams, 
        gasPrice
    )

In the above code, we ready to buy **0.2** test BNB (decimal amount). We simply call function **getSwapExactOutputCall** to get acquired amount of token test **BNB**.
The params needed by function **getSwapExactOutputCall** can be viewed in the following code:


.. code-block:: typescript
    :linenos:

    
    /**
    * @param universalSwapRouter, universal swap router contract, can be obtained through getUniversalSwapRouterContract(...)
    * @param account, address of user
    * @param chain, object of BaseChain, describe which chain we are using
    * @param params, some settings of this swap, including swapchain, input amount, min required output amount
    * @param gasPrice, gas price of this swap transaction
    * @return calling, calling of this swap transaction
    * @return options, options of this swap transaction, used in sending transaction
    */
    export const getSwapExactOutputCall = (
        universalSwapRouter: Contract<ContractAbi>, 
        account: string,
        chain: BaseChain,
        params: SwapExactOutputParams, 
        gasPrice: number | string
    ) : {calling: any, options: any}


**SwapExactOutputParams** has been explained in :ref:`section 2<universal_quoter_swap_chain_with_exact_output_initialization>`

We usually keep **outChargeFeeTier**, **tokenChain**, **feeTier**, **isV2** unchanged
from "quoterParams", expect **maxInputAmount**. 
And Usually we can fill **SwapExactOutputParams** through following code,

.. code-block:: typescript
    :linenos:

    const swapParams = {
        ...quoterParams,
        // slippery is 1.5%
        maxInputAmount: new BigNumber(inputAmount).times(1.015).toFixed(0)
    } as SwapExactOutputParams


Notice that in this example, input token (test USDT) is ERC20 token and output token (test BNB) is a native token.
However, if you want to pay or receive **wrapped native** (ERC20) or **native** token,
you can refer to :ref:`section 3<universal_quoter_swap_chain_with_exact_output_wrapped_or_native>` 


7. Approve (skip if you pay native token directly)
---------------------------------------------------

Before sending transaction or estimating gas, you need to approve contract Swap to have authority to spend your token.
Since the contract need to transfer some input token to the pool.


If the allowance is enough or the input token is chain gas token, just skip this step.

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
    // if input token is not chain token (BNB on BSC or ETH on Ethereum...), we need transfer input token to pool
    // otherwise we can skip following codes
    {
        const usdtContract = new web3.eth.Contract(erc20ABI, USDT.address);
        // you could approve a very large amount (much more greater than amount to transfer),
        // and don't worry about that because swapContract only transfer your token to pool with amount you specified and your token is safe
        // then you do not need to approve next time for this user's address
        const approveCalling = usdtContract.methods.approve(
            swapAddress, 
            "0xffffffffffffffffffffffffffffffff"
        );
        // estimate gas
        const gasLimit = await approveCalling.estimateGas({from: account})
        // then send transaction to approve
        // you could simply use followiing line if you use metamask in your frontend code
        // otherwise, you should use the function "web3.eth.accounts.signTransaction"
        // notice that, sending transaction for approve may fail if you have approved the token to swapContract before
        // if you want to enlarge approve amount, you should refer to interface of erc20 token
        await approveCalling.send({gas: Number(gasLimit)})
    }

8. Estimate gas (optional)
--------------------------

Before actually send the transaction, this is double check (or user experience enhancement measures) to check whether the gas spending is normal.


.. code-block:: typescript
    :linenos:

    const gasLimit = await swapCalling.estimateGas(options)
    console.log('gas limit: ', gasLimit)

9. Send transaction!
--------------------

Now, we can then send the transaction.

For metamask or other explorer's wallet provider, you can easily write

.. code-block:: typescript
    :linenos:

    // it is suggested to fill the gas with a number a little greater than estimated "gasLimit".
    await swapCalling.send({...options, gas: Number(gasLimit)})

Otherwise, you could use following code

.. code-block:: typescript
    :linenos:

    // sign transaction
    // options is returned from getSwapChainWithExactInputCall
    const signedTx = await web3.eth.accounts.signTransaction(
        {
            ...options,
            to: swapAddress,
            data: swapCalling.encodeABI(),
            gas: new BigNumber(Number(gasLimit) * 1.1).toFixed(0, 2),
        }, 
        privateKey
    )
    // send transaction
    const tx = await web3.eth.sendSignedTransaction(signedTx.rawTransaction);
    console.log('tx: ', tx);

After sending transaction, we will successfully finish swapping with exact amount of input token (if no revert occurred).