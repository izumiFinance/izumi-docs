quoter/swap with exact input
============================

in this example, we will do quoter and swap with exact input amount in iZiSwap.

here, **quoter with exact input amount** means pre-query amount of token acquired given exact input token amount.

**swap with exact input amount** means doing real swap with given exact input amount.

in this example, quoter and swap are called throw 2 different contracts.

1. some imports
-----------------------------------------------------------

first some base imports

.. code-block:: typescript
    :linenos:

    import {BaseChain, ChainId, initialChainTable} from 'iziswap-sdk/lib/base/types'
    import {privateKey} from '../../.secret'
    import Web3 from 'web3';
    import { amount2Decimal, fetchToken, getErc20TokenContract } from 'iziswap-sdk/lib/base/token/token';
    import { BigNumber } from 'bignumber.js'
    import { getQuoterContract, quoterSwapChainWithExactInput } from 'iziswap-sdk/lib/quoter/funcs';
    import { QuoterSwapChainWithExactInputParams } from 'iziswap-sdk/lib/quoter/types';
    import { getSwapChainWithExactInputCall, getSwapContract } from 'iziswap-sdk/lib/swap/funcs';
    import { SwapChainWithExactInputParams } from 'iziswap-sdk/lib/swap/types';

second, some imports for quoter and swap

.. code-block:: typescript
    :linenos:

    import { getQuoterContract, quoterSwapChainWithExactInput } from '../../src/quoter/funcs';
    import { QuoterSwapChainWithExactInputParams } from '../../src/quoter/types';
    import { getSwapChainWithExactInputCall, getSwapContract } from '../../src/swap/funcs';
    import { SwapChainWithExactInputParams } from '../../src/swap/types';

in the above code, **quoterSwapChainWithExactInput** will return amount of token acquired.
**getSwapChainWithExactInputCall** will return calling for swapping with exact input.

2. some initialization
-----------------------------------------------------------

.. code-block:: typescript
    :linenos:

    const chain:BaseChain = initialChainTable[ChainId.BSCTestnet]
    const rpc = 'https://bsc-dataseed2.defibit.io/'
    console.log('rpc: ', rpc)
    const web3 = new Web3(new Web3.providers.HttpProvider(rpc))
    console.log('aaaaaaaa')
    const account =  web3.eth.accounts.privateKeyToAccount(privateKey)
    console.log('address: ', account.address)

    const testAAddress = '0xCFD8A067e1fa03474e79Be646c5f6b6A27847399'
    const testBAddress = '0xAD1F11FBB288Cd13819cCB9397E59FAAB4Cdc16F'

    // TokenInfoFormatted of token 'testA' and token 'testB'
    const testA = await fetchToken(testAAddress, chain, web3)
    const testB = await fetchToken(testBAddress, chain, web3)
    const fee = 2000 // 2000 means 0.2%

here we take example of paying token **testA** to acquire token **testB**

3. use quoter to pre-query amount of token **testB** acquired
-------------------------------------------------------------

first, we use **getQuoterContract** to get quoter contract

.. code-block:: typescript
    :linenos:

    const quoterAddress = '0x12a76434182c8cAF7856CE1410cD8abfC5e2639F'
    console.log('quoter address: ', quoterAddress)
    const quoterContract = getQuoterContract(quoterAddress, web3)

second, use **quoterSwapChainWithExactInput** to query

.. code-block:: typescript
    :linenos:

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

in the above code, we ready to pay **50** testA (decimal amount).we simply call function **quoterSwapChainWithExactInput** to get acquired amount of token **testB**
the function **quoterSwapChainWithExactInput** need 2 params:
first is **quoterContract** which is obtained through **getQuoterContract** before.
second is an object of **QuoterSwapChainWithExactInputParams**, which describe informations such as **swap chains** and **input amount**

the fields of **QuoterSwapChainWithExactInputParams** is explained in the following code.

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
if you have some token0, and wants to get token3 through the path
**(token0, token1, 0.05%) => (token1, token2, 0.3%) => (token2, token3, 0.3%)**, 

you should fill the **tokenChain** and **feeChain** fields with following code


.. code-block:: typescript
    :linenos:

    // here, token0..3 are TokenInfoFormatted
    params.tokenChain = [token0, token1, token2, token3]
    params.feeChain = [500, 3000, 3000]

4. use swap to do pay token **testA** to get token **testB**
-------------------------------------------------------------

first, we use **getQuoterContract** to get quoter contract

.. code-block:: typescript
    :linenos:

    const swapAddress = '0xBd3bd95529e0784aD973FD14928eEDF3678cfad8'
    const swapContract = getSwapContract(swapAddress, web3)

second, use **getSwapChainWithExactInputCall** to get calling of swap

.. code-block:: typescript
    :linenos:

    const swapParams = {
        ...params,
        // slippery is 1.5%
        minOutputAmount: new BigNumber(amountB).times(0.985).toFixed(0)
    } as SwapChainWithExactInputParams
    
    const gasPrice = '5000000000'

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

in the above code, we ready to pay **50** testA (decimal amount).we simply call function **getSwapChainWithExactInputCall** to get acquired amount of token **testB**
the params needed by function **getSwapChainWithExactInputCall** can be viewed in the following code

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

usually, we can fill **SwapChainWithExactInputParams** through following code

.. code-block:: typescript
    :linenos:

    const swapParams = {
        ...params,
        // slippery is 1.5%, here amountB is value returned from quoter
        minOutputAmount: new BigNumber(amountB).times(0.985).toFixed(0)
    } as SwapChainWithExactInputParams


5. estimate gas (optional)
--------------------------

of course you can skip this step if you donot want to limit gas

.. code-block:: typescript
    :linenos:

    const gasLimit = await swapCalling.estimateGas(options)
    console.log('gas limit: ', gasLimit)

6. send transaction!
--------------------

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

after sending transaction, we will successfully do swapping with exact amount of input token (if no revert occured)