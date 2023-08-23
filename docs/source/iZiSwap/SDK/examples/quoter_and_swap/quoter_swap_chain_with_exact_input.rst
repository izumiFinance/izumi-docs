.. _quoter_swap_chain_with_exact_input:

Exact input mode
============================

In this example, we will conduct price query and swap with exact input amount in iZiSwap (Amount Mode).

Here, **quoter with exact input amount** means pre-query amount of token acquired given exact input token amount. For example, if you want to swap 1 ETH to N USDC, 
the step is used to determine N.

**swap with exact input amount** means to invoke the real swap with the amount.

Quoter and swap are called throw 2 different contracts.

Suppose we want to swap **USDT** (testA) token to **BUSD** (testB) token, where these two tokens are standard ERC-20 tokens deployed on BSC.
The full example codes can be found `here <https://github.com/izumiFinance/izumi-iZiSwap-sdk/blob/main/example/quoterAndSwap/quoterSwapChainWithExactInput.ts>`_.

1. Some imports
-----------------------------------------------------------

These are some base modules used in most examples.

.. code-block:: typescript
    :linenos:

    import Web3 from 'web3';
    import {privateKey} from '../../.secret'
    import { BigNumber } from 'bignumber.js'
    import {BaseChain, ChainId, initialChainTable} from 'iziswap-sdk/lib/base/types'
    import { amount2Decimal, fetchToken, getErc20TokenContract } from 'iziswap-sdk/lib/base/token/token';

And these are some imports for quoter and swap.

.. code-block:: typescript
    :linenos:

    import { SwapChainWithExactInputParams } from 'iziswap-sdk/lib/swap/types';
    import { QuoterSwapChainWithExactInputParams } from 'iziswap-sdk/lib/quoter/types';
    import { getSwapChainWithExactInputCall, getSwapContract } from 'iziswap-sdk/lib/swap/funcs';
    import { getQuoterContract, quoterSwapChainWithExactInput } from 'iziswap-sdk/lib/quoter/funcs';

Here **quoterSwapChainWithExactInput** will return amount of token acquired (N).
**getSwapChainWithExactInputCall** will return calling for swapping with exact input.

2. Initialization
-----------------------------------------------------------

.. code-block:: typescript
    :linenos:

    const chain:BaseChain = initialChainTable[ChainId.BSC]
    const rpc = 'https://bsc-dataseed2.defibit.io/' //BSC network rpc node
    const web3 = new Web3(new Web3.providers.HttpProvider(rpc))
    const account =  web3.eth.accounts.privateKeyToAccount(YOUR_PRIVATE_KEY)  //Fill with your sk, dangerous, never to share 
    console.log('address: ', account.address)

    const testAAddress = '0x55d398326f99059ff775485246999027b3197955' // USDT
    const testBAddress = '0xe9e7CEA3DedcA5984780Bafc599bD69ADd087D56' // BUSD

    // TokenInfoFormatted of token 'USDT' and token 'BUSD'
    const testA = await fetchToken(testAAddress, chain, web3)
    const testB = await fetchToken(testBAddress, chain, web3)
    const fee = 400 // 400 means 0.04%

We are going to pay token **testA** to acquire token **testB**.

.. _quoter_swap_chain_with_exact_input_query:

3. Use Quoter to pre-query amount of token **testB** acquired
---------------------------------------------------------------

First, we use **getQuoterContract** to instantiate a quoter contract.

.. code-block:: typescript
    :linenos:

    const quoterAddress = '0x64b005eD986ed5D6aeD7125F49e61083c46b8e02' // Quoter address on BSC, more can be found in the deployed contracts section.
    const quoterContract = getQuoterContract(quoterAddress, web3)

Second, use **quoterSwapChainWithExactInput** to query.

.. code-block:: typescript
    :linenos:

    // swap 50 USDT -> BUSD
    const amountA = new BigNumber(50).times(10 ** testA.decimal)

    const params = {
        // pay testA to buy testB
        tokenChain: [testA, testB],
        feeChain: [fee],
        inputAmount: amountA.toFixed(0)
    } as QuoterSwapChainWithExactInputParams

    const {outputAmount} = await quoterSwapChainWithExactInput(quoterContract, params)

    const amountB = outputAmount
    const amountBDecimal = amount2Decimal(new BigNumber(amountB), testB)

    console.log(' amountA to pay: ', 50)
    console.log(' amountB to acquire: ', amountBDecimal)

In the above code, we are ready to pay **50** testA (USDT, decimal amount). 
We simply call function **quoterSwapChainWithExactInput** to get the acquired amount of token **testB** (BUSD).
The function **quoterSwapChainWithExactInput** need 2 params:

* - **quoterContract**: obtained through **getQuoterContract** before
* - a **QuoterSwapChainWithExactInputParams** instance: describes information such as **swap chains** and **input amount**

The fields of **QuoterSwapChainWithExactInputParams** is explained in the following code.

.. code-block:: typescript
    :linenos:

    export interface QuoterSwapChainWithExactInputParams {

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
        // amount = inputAmount / (10 ** inputToken.decimal)
        inputAmount: string;
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
----------------------------------------------------------------------

First, we use **getSwapContract** to get the Swap contract

.. code-block:: typescript
    :linenos:

    const swapAddress = '0xBd3bd95529e0784aD973FD14928eEDF3678cfad8' // Swap contract on BSC
    const swapContract = getSwapContract(swapAddress, web3)

Second, use **getSwapChainWithExactInputCall** to get calling (transaction handler) of swap:

.. code-block:: typescript
    :linenos:

    const swapParams = {
        ...params,
        // slippery is 1.5%
        // amountB is the pre-query result from Quoter
        minOutputAmount: new BigNumber(amountB).times(0.985).toFixed(0)
    } as SwapChainWithExactInputParams
    
    const gasPrice = '3000000000' //BSC default gas price

    const tokenA = testA
    const tokenB = testB
    const tokenAContract = getErc20TokenContract(tokenA.address, web3)
    const tokenBContract = getErc20TokenContract(tokenB.address, web3)

    const tokenABalanceBeforeSwap = await tokenAContract.methods.balanceOf(account.address).call()
    const tokenBBalanceBeforeSwap = await tokenBContract.methods.balanceOf(account.address).call()

    console.log('tokenABalanceBeforeSwap: ', tokenABalanceBeforeSwap)
    console.log('tokenBBalanceBeforeSwap: ', tokenBBalanceBeforeSwap)

    const {swapCalling, options} = getSwapChainWithExactInputCall(
        swapContract, 
        account.address, 
        chain, 
        swapParams, 
        gasPrice
    )

In the above code, we ready to pay **50** testA (decimal amount). We simply call function **getSwapChainWithExactInputCall** to get acquired amount of token **testB**.
The params needed by function **getSwapChainWithExactInputCall** can be viewed in the following code:

.. code-block:: typescript
    :linenos:

    /**
     * @param swapContract, swap contract, can be obtained through getSwapContract(...)
     * @param account, address of user
     * @param chain, object of BaseChain, describe which chain we are using
     * @param params, some settings of this swap, including swapchain, input amount, min required output amount
     * @param gasPrice, gas price of this swap transaction
     * @return swapCalling, calling of this swap transaction
     * @return options, options of this swap transaction, used in sending transaction
     */
    export const getSwapChainWithExactInputCall = (
        swapContract: Contract, 
        account: string,
        chain: BaseChain,
        params: SwapChainWithExactInputParams, 
        gasPrice: number | string
    ) : { swapCalling: any, options: any }

**SwapChainWithExactInputParams** has following fields

.. code-block:: typescript
    :linenos:

    export interface SwapChainWithExactInputParams {
        
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
        // amount = inputAmount / (10 ** inputToken.decimal)
        inputAmount: string;

        // if actual acquired amount < minOutputAmount, the transaction will be revert
        minOutputAmount: string;

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

Usually, we can fill **SwapChainWithExactInputParams** through following code

.. code-block:: typescript
    :linenos:

    const swapParams = {
        ...params,
        // slippery is 1.5%, here amountB is value returned from quoter
        minOutputAmount: new BigNumber(amountB).times(0.985).toFixed(0)
    } as SwapChainWithExactInputParams


Notice that in this example, both tokens are ERC-20 compatible tokens and is the general case. However,
if tokenX or tokenY is chain gas token (such as `ETH` on Ethereum or `BNB` on BSD),
we should specify one or some fields in `swapParams` to indicate sdk paying/acquiring in form of `Chain Token`
or paying/acquiring in form of `Wrapped Chain Token` (such as `WETH` on Ethereum or `WBNB` on BSC).

In that case, take **testA** to be BNB as example. 

If you want to use BNB directly, just set testAAddress to be WBNB and `strictERC20Token` is `false` by default. 

.. code-block:: typescript
    :linenos:

    ...

    const testAAddress = '0xbb4CdB9CBd36B01bD1cBaEBF2De08d9173bc095c' // WBNB

    ...

And the BNB need to pay (the value field in the transaction data) is set in the `options` return.


If you want to use WBNB, first to set testAAddress to be WBNB and then to set `strictERC20Token` as `true`.


.. code-block:: typescript
    :linenos:

    ...

    const testAAddress = '0xbb4CdB9CBd36B01bD1cBaEBF2De08d9173bc095c' // WBNB

    ...

    const swapParams = {
        ...
        strictERC20Token: true
        ...
    } as SwapChainWithExactInputParams

    ...

Now the swap will use WBNB instead of BNB.

..
    In the sdk version 1.1.* or before, one should specify a field named `strictERC20Token` to indicate that.
    `true` for paying/acquiring token in form of `Wrapped Chain Token`, `false` for paying/acquiring in form of `Chain Token`.
    In the sdk version 1.2.* or later, you have two ways to indicate sdk. 

    The first way is as before, specifing `strictERC20Token` field.
    The second way is specifing `strictERC20Token` as undefined and specifying the corresponding token in this param as 
    `WETH` or `ETH`.


5. Approve (skip if you pay chain token directly)
---------------------------------------------------

Before sending transaction or estimating gas, you need to approve contract Swap to have authority to spend your token.
Since the contract need to transfer some tokenA or tokenB to the pool.


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
    // if tokenA is not chain token (BNB on BSC or ETH on Ethereum...), we need transfer tokenA to pool
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
        const gasLimit = await mintCalling.estimateGas({from: account})
        // then send transaction to approve
        // you could simply use followiing line if you use metamask in your frontend code
        // otherwise, you should use the function "web3.eth.accounts.signTransaction"
        // notice that, sending transaction for approve may fail if you have approved the token to swapContract before
        // if you want to enlarge approve amount, you should refer to interface of erc20 token
        await approveCalling.send({gas: gasLimit})
    }

6. Estimate gas (optional)
--------------------------

Before actually send the transaction, this is double check (or user experience enhancement measures) to check whether the gas spending is normal.


.. code-block:: typescript
    :linenos:

    const gasLimit = await swapCalling.estimateGas(options)
    console.log('gas limit: ', gasLimit)

7. Send transaction!
--------------------

Now, we can then send the transaction.

For metamask or other explorer's wallet provider, you can easily write

.. code-block:: typescript
    :linenos:

    await swapCalling.send({...options, gas: gasLimit})

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
            gas: new BigNumber(gasLimit * 1.1).toFixed(0, 2),
        }, 
        privateKey
    )
    // send transaction
    const tx = await web3.eth.sendSignedTransaction(signedTx.rawTransaction);
    console.log('tx: ', tx);

After sending transaction, we will successfully finish swapping with exact amount of input token (if no revert occurred).