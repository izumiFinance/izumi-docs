.. _quoter_swap_chain_with_exact_output:

Exact output mode
=============================

In this example, we will conduct price query and swap with exact output amount in iZiSwap (Desire Mode).

Here, **quoter with exact output amount** means pre-query amount of token need to pay given exact amount of desired token. For example, if you want to swap N ETH to 2000 USDC, 
the step is used to determine N.

**swap with exact output amount** means to invoke the real swap with the desired amount.

Quoter and swap are called throw 2 different contracts.

Suppose we want to swap **USDT** (testA) token to **BUSD** (testB) token, where these two tokens are standard ERC-20 tokens deployed on BSC.
The full example code of this chapter can be found `here <https://github.com/izumiFinance/izumi-iZiSwap-sdk/blob/main/example/quoterAndSwap/quoterSwapChainWithExactOutput.ts>`_.

1. Some imports
-----------------------------------------------------------

These are some base modules used in most examples.

.. code-block:: typescript
    :linenos:

    import Web3 from 'web3';
    import { BigNumber } from 'bignumber.js'
    import {privateKey} from '../../.secret'
    import {BaseChain, ChainId, initialChainTable} from 'iziswap-sdk/lib/base/types'
    import { amount2Decimal, fetchToken, getErc20TokenContract } from 'iziswap-sdk/lib/base/token/token';


And these are some imports for quoter and swap.

.. code-block:: typescript
    :linenos:

    import { getQuoterContract, quoterSwapChainWithExactOutput } from 'iziswap-sdk/lib/quoter/funcs';
    import { QuoterSwapChainWithExactOutputParams } from 'iziswap-sdk/lib/quoter/types';
    import { getSwapChainWithExactOutputCall, getSwapContract } from 'iziswap-sdk/lib/swap/funcs';
    import { SwapChainWithExactOutputParams } from 'iziswap-sdk/lib/swap/types';

Here **quoterSwapChainWithExactOutput** will return the amount of input token needed to pay (N), and
**getSwapChainWithExactOutputCall** will return calling for swapping with exact amount of output (or we say desired) token.

2. Initialization
-----------------------------------------------------------

.. code-block:: typescript
    :linenos:

    const chain:BaseChain = initialChainTable[ChainId.BSC]
    const rpc = 'https://bsc-dataseed2.defibit.io/' //BSC network rpc node
    const web3 = new Web3(new Web3.providers.HttpProvider(rpc))
    const account =  web3.eth.accounts.privateKeyToAccount(YOUR_PRIVATE_KEY) // replace privateKey to your sk
    console.log('address: ', account.address)

    const testAAddress = '0x55d398326f99059ff775485246999027b3197955' // USDT
    const testBAddress = '0xe9e7CEA3DedcA5984780Bafc599bD69ADd087D56' // BUSD

    // TokenInfoFormatted of token 'USDT' and token 'BUSD'
    const testA = await fetchToken(testAAddress, chain, web3)
    const testB = await fetchToken(testBAddress, chain, web3)
    const fee = 500 // 500 means 0.05%

We take example of paying token **testA** to acquire token **testB** with the pool of fee rate **0.05%**.

*In general, the supported fee rates for the mainnet are 500 (0.05%), 3000 (0.3%), and 10000 (1%); and for the testnet are 400 (0.04%), 2000 (0.2%) and 10000 (1%). One needs to check if the choosen pool exists and has enough liquidity.*
*The liquidity condition can be checked on the analytics page* `here <https://analytics.izumi.finance>`_ .


3. Use quoter to pre-query amount of token **testB** acquired
-----------------------------------------------------------------

First, we use **getQuoterContract** to get the Quoter contract

.. code-block:: typescript
    :linenos:

    const quoterAddress = '0x64b005eD986ed5D6aeD7125F49e61083c46b8e02' // Quoter address on BSC, more can be found in the deployed contracts section.
    const quoterContract = getQuoterContract(quoterAddress, web3)

Then, use **quoterSwapChainWithExactOutput** to query

.. code-block:: typescript
    :linenos:

    const amountB = new BigNumber(10).times(10 ** testA.decimal)

    const params = {
        // pay testA to buy testB
        tokenChain: [testA, testB],
        feeChain: [fee],
        outputAmount: amountB.toFixed(0)
    } as QuoterSwapChainWithExactOutputParams

    const {inputAmount} = await quoterSwapChainWithExactOutput(quoterContract, params)

    const amountA = inputAmount
    const amountADecimal = amount2Decimal(new BigNumber(amountA), testA)

    console.log(' amountB to desired: ', 10)
    console.log(' amountA to pay: ', amountADecimal)

In the above code, we ready to buy **10** testB (decimal amount). We simply call function **quoterSwapChainWithExactOutput** to get acquired amount of token **testB**.
The function **quoterSwapChainWithExactOutput** need 2 params:

* - **quoterContract**: obtained through **getQuoterContract** before
* - a **QuoterSwapChainWithExactOuptutParams** instance: describes information such as **swap chains** and **output amount**

The fields of **QuoterSwapChainWithExactOutputParams** is explained in the following code.

.. code-block:: typescript
    :linenos:

    export interface QuoterSwapChainWithExactOutputParams {

        // input: tokenChain.first()
        // output: tokenChain.last()
        tokenChain: TokenInfoFormatted[];

        // feeChain[i] / 1e6 is feeTier
        // 3000 means 0.3%
        // (tokenChain[i], feeChain[i], tokenChain[i+1]) means i-th iZi-swap-pool in the swap chain
        // in that pool, tokenChain[i] is the token payed to the pool, tokenChain[i+1] is the token acquired from the pool
        // ofcourse, feeChain.length + 1 === tokenChain.length
        feeChain: number[];

        // 10-decimal format number, like 100, 150000, ...
        // or hex format number start with '0x'
        // amount = outputAmount / (10 ** outputToken.decimal)
        outputAmount: string;
    }

**iZiSwap**'s quoter and swap contracts support swap chain with multi swap pools.
For example, if you have some token0, and wants to get token3 through the path
**token0 -> (token0, token1, 0.05%) -> token1 -> (token1, token2, 0.3%) -> token2 -> (token2, token3, 0.3%) -> token3**, 
you should fill the **tokenChain** and **feeChain** fields with following code


.. code-block:: typescript
    :linenos:

    // here, token0..3 are TokenInfoFormatted
    params.tokenChain = [token0, token1, token2, token3]
    params.feeChain = [500, 3000, 3000]

Now we have finished the Quoter part. 

4. Use Swap to actually pay token **testA** to get token **testB**
--------------------------------------------------------------------

First, we use **getSwapContract** to get the swap contract

.. code-block:: typescript
    :linenos:

    const swapAddress = '0xBd3bd95529e0784aD973FD14928eEDF3678cfad8' // Swap contract on BSC
    const swapContract = getSwapContract(swapAddress, web3)

Second, use **getSwapChainWithExactOutputCall** to get calling of swap

.. code-block:: typescript
    :linenos:

    const swapParams = {
        ...params,
        // slippery is 1.5%
        maxInputAmount: new BigNumber(amountA).times(1.015).toFixed(0)
    } as SwapChainWithExactOutputParams
    
    const gasPrice = '3000000000' // default BSC gas price

    const tokenA = testA
    const tokenB = testB
    const tokenAContract = getErc20TokenContract(tokenA.address, web3)
    const tokenBContract = getErc20TokenContract(tokenB.address, web3)

    const tokenABalanceBeforeSwap = await tokenAContract.methods.balanceOf(account.address).call()
    const tokenBBalanceBeforeSwap = await tokenBContract.methods.balanceOf(account.address).call()

    console.log('tokenABalanceBeforeSwap: ', tokenABalanceBeforeSwap)
    console.log('tokenBBalanceBeforeSwap: ', tokenBBalanceBeforeSwap)

    const {swapCalling, options} = getSwapChainWithExactOutputCall(
        swapContract, 
        account.address, 
        chain, 
        swapParams, 
        gasPrice
    )

In the above code, we ready to buy **10** testB (decimal amount). We simply call function **getSwapChainWithExactOutputCall** to get acquired amount of token **testA**.
The params needed by function **getSwapChainWithExactOutputCall** can be viewed in the following code

.. code-block:: typescript
    :linenos:

    /**
     * @param swapContract, swap contract, can be obtained through getSwapContract(...)
     * @param account, address of user
     * @param chain, object of BaseChain, describe which chain we are using
     * @param params, some settings of this swap, including swapchain, output amount, max input amount
     * @param gasPrice, gas price of this swap transaction
     * @return swapCalling, calling of this swap transaction
     * @return options, options of this swap transaction, used in sending transaction
     */
    export const getSwapChainWithExactOutputCall = (
        swapContract: Contract, 
        account: string,
        chain: BaseChain,
        params: SwapChainWithExactOutputParams, 
        gasPrice: number | string
    ) : {swapCalling: any, options: any}

**SwapChainWithExactOutputParams** has following fields

.. code-block:: typescript
    :linenos:

    export interface SwapChainWithExactOutputParams {
        
        // input: tokenChain.first()
        // output: tokenChain.last()
        tokenChain: TokenInfoFormatted[];

        // feeChain[i] / 1e6 is feeTier
        // 3000 means 0.3%
        // (tokenChain[i], feeChain[i], tokenChain[i+1]) means i-th iZi-swap-pool in the swap chain
        // in that pool, tokenChain[i] is the token payed to the pool, tokenChain[i+1] is the token acquired from the pool
        // ofcourse, feeChain.length + 1 === tokenChain.length
        feeChain: number[];

        // 10-decimal format number, like 100, 150000, ...
        // or hex format number start with '0x'
        // amount = outputAmount / (10 ** outputToken.decimal)
        outputAmount: string;
        // if actual amount of input token > maxInputAmount, the transaction will be reverted
        maxInputAmount: string;

        // who will get outputToken, default is payer
        recipient?: string;

        // latest timestamp to execute this swap transaction, default is 0xffffffff, 
        // etc max number of uint32, which is larger than latest unix-time
        deadline?: string;

        // default is false
        // when the input or output token is wbnb or weth or other wrapped chain-token
        // user wants to pay bnb/eth directly (send the transaction with value > 0) or acquire bnb/eth directly
        // if this field is undefined or false, user will send the swap calling with value > 0 or acquire bnb/eth directly
        // if this field is true, user will send the swap calling with value===0 and pay eth/bnb through weth/wbnb 
        //    like other erc-20 tokens or acquire weth/wbnb like other erc-20 tokens
        strictERC20Token?: boolean;
    }

Usually, we can fill **SwapChainWithExactOutputParams** through following code

.. code-block:: typescript
    :linenos:

    const swapParams = {
        ...params,
        // slippery is 1.5%, here amountA is value returned from quoter
        maxInputAmount: new BigNumber(amountA).times(1.015).toFixed(0)
    } as SwapChainWithExactOutputParams

Again, if one of the tokens is the chain gas token (e.g., ETH on Ethereum), please refer to the previous section to check how to set 
the `strictERC20Token` param.

..
    we should notice that, if tokenX or tokenY is chain token (like `ETH` on ethereum or `BNB` on bsc),
    we should specify one or some fields in `swapParams` to indicate sdk paying/acquiring in form of `Chain Token`
    or paying/acquiring in form of `Wrapped Chain Token` (like `WETH` on ethereum or `WBNB` on bsc).

    In the sdk version 1.1.* or before, one should specify a field named `strictERC20Token` to indicate that.
    `true` for paying/acquiring token in form of `Wrapped Chain Token`, `false` for paying/acquiring in form of `Chain Token`.
    In the sdk version 1.2.* or later, you have two ways to indicate sdk. 

    The first way is as before, specifing `strictERC20Token` field.
    The second way is specifing `strictERC20Token` as undefined and specifying the corresponding token in this param as 
    `WETH` or `ETH`.


5. Approve (skip if you pay chain token directly)
---------------------------------------------------

before send transaction or estimate gas, you need to approve contract liquidityManager to have authority to spend yuor token,
because you need transfer some tokenA and some tokenB to pool.

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
    // if tokenA is not chain token (BNB on bsc chain or ETH on eth chain...), we need transfer tokenA to pool
    // otherwise we can skip following codes
    {
        const tokenAContract = new web3.eth.Contract(erc20ABI, testAAddress);
        // you could approve a very large amount (much more greater than amount to transfer),
        // and don't worry about that because swapContract only transfer your token to pool with amount you specified and your token is safe
        // then you do not need to approve next time for this user's address
        const approveCalling = tokenAContract.methods.approve(
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
        await approveCalling.send({gas: gasLimit})
    }

6. estimate gas (optional)
--------------------------

of course you can skip this step if you donot want to limit gas

.. code-block:: typescript
    :linenos:

    const gasLimit = await swapCalling.estimateGas(options)
    console.log('gas limit: ', gasLimit)

7. send transaction!
--------------------

now, we can then send transaction to swap

for metamask or other explorer's wallet provider, you can easily write

.. code-block:: typescript
    :linenos:

    await swapCalling.send({...options, gas: gasLimit})

otherwise, you could use following code

.. code-block:: typescript
    :linenos:

    // sign transaction
    // options is returned from getSwapChainWithExactInputCall
    const signedTx = await web3.eth.accounts.signTransaction(
        {
            ...options,
            to: swapAddress,
            data: swapCalling.encodeABI(),
            gas: new BigNumber(gasLimit * 1.1).toFixed(0, 2),
        }, 
        privateKey
    )
    // send transaction
    const tx = await web3.eth.sendSignedTransaction(signedTx.rawTransaction);
    console.log('tx: ', tx);

after sending transaction, we will successfully do swapping with exact amount of desired(or we say output) token (if no revert occured)