Search Swap Path
============================

In this example, we will do path searching with **exact input amount** (amount of payed token).
The complete example codes can be found `here <https://github.com/izumiFinance/iZiSwap-sdk/blob/main/example/search/searchWithExactInput.ts>`__.

The search result can be used in **Exact Input** mode of **swap** or **quoter** .
Also we can use same search result in **Exact Output** mode of **swap** or **quoter**.

If you want to do path searching with **exact output amount** (amount of acquired token),
You can refer the codes in the `here <https://github.com/izumiFinance/iZiSwap-sdk/blob/main/example/search/searchWithExactOutput.ts>`_.

The only difference of these two modes while doing path searching can be viewed in :ref:`this subsection <diff_for_exact_output>`


1. Some imports
-----------------------------------------------------------

.. code-block:: typescript
    :linenos:

    import {BaseChain, ChainId, initialChainTable, TokenInfoFormatted} from '../../src/base/types'
    import Web3 from 'web3';
    import { BigNumber } from 'bignumber.js'
    import { getMulticallContracts } from '../../src/base';
    import { PoolPair, SearchPathQueryParams, SwapDirection } from '../../src/search/types';
    import { searchPathQuery } from '../../src/search/func'

Here **searchPathQuery** will help us do path searching.

2. Initialization
-----------------------------------------------------------

Here we take example of bsc test net. 
And we need address of **Quoter**, **LiquidityManager** and **Multicall**, **Tokens**

**Multicall** is contracts we deployed on each chain to speed up **view** or **pure** calling.

You can refer to our docs for addresses of those contracts on different chain.

In this example, payed token from user is **BNB** and acquired token is **USDT**
And also we may need some middle token for searching, etc, **iZi**, **USDC**, **iUSD** ...

And in this example, we init max amount of **BNB** is **10.0**

.. code-block:: typescript
    :linenos:

    const rpc = 'https://data-seed-prebsc-1-s3.binance.org:8545/'
    console.log('rpc: ', rpc)
    const web3 = new Web3(new Web3.providers.HttpProvider(rpc))
    
    const quoterAddress = '0x4bCACcF9A0FC3246449AC8A42A8918F2349Ed543'

    const BNB = {
        chainId: ChainId.BSCTestnet,
        symbol: "BNB",
        address: "0xae13d989daC2f0dEbFf460aC112a837C89BAa7cd",
        decimal: 18,
    } as TokenInfoFormatted
    
    const USDT = {
        chainId: ChainId.BSCTestnet,
        symbol: "USDT",
        address: "0x6AECfe44225A50895e9EC7ca46377B9397D1Bb5b",
        decimal: 6
    } as TokenInfoFormatted

    const USDC = {
        chainId: ChainId.BSCTestnet,
        symbol: "USDC",
        address: "0x876508837C162aCedcc5dd7721015E83cbb4e339",
        decimal: 6
    }

    const iZi = {
        chainId: ChainId.BSCTestnet,
        symbol: "iZi",
        address: "0x60D01EC2D5E98Ac51C8B4cF84DfCCE98D527c747",
        decimal: 18
    }

    const iUSD = {
        chainId: ChainId.BSCTestnet,
        symbol: "iUSD",
        address: "0x60FE1bE62fa2082b0897eA87DF8D2CfD45185D30",
        decimal: 18,
    }

    const support001Pools = [
        {
            tokenA: iUSD,
            tokenB: USDT,
            feeContractNumber: 100
        } as PoolPair,
        {
            tokenA: USDC,
            tokenB: USDT,
            feeContractNumber: 100
        } as PoolPair,
        {
            tokenA: USDC,
            tokenB: iUSD,
            feeContractNumber: 100
        } as PoolPair,
    ]

    // example of exact input amount
    const amountInputBNB = new BigNumber(10).times(10 ** BNB.decimal).toFixed(0)
    
    const multicallAddress = '0x5712A9aeB4538104471dD85659Bd621Cdd7e07D8'
    const multicallContract = getMulticallContracts(multicallAddress, web3)
    const liquidityManagerAddress = '0xDE02C26c46AC441951951C97c8462cD85b3A124c'


3. construct params
---------------------------------------------------------------

.. code-block:: typescript
    :linenos:

    // params
    const searchParams = {
        chainId: Number(ChainId.BSCTestnet),
        web3: web3,
        multicall: multicallContract,
        tokenIn: BNB,
        tokenOut: USDT,
        liquidityManagerAddress,
        quoterAddress,
        poolBlackList: [],
        midTokenList: [BNB, USDT, USDC, iZi],
        supportFeeContractNumbers: [3000, 500, 100],
        support001Pools,
        direction: SwapDirection.ExactIn,
        amount: amountInputBNB
    } as SearchPathQueryParams

**SearchPathQueryParams** defined fields of params need by our path-searching function.

In this example, the swap mode is **swap with exact input**, and we need to fill 
**SearchPathQueryParams.direction** with **SwapDirection.ExactIn** and to fill
**SearchPathQueryParams.amount** with **undecimal amount of input token (BNB)**

**supportFeeContractNumbers** is a list containing supported fees of swap-pool. And we only consider
pools with fee within this list in our path searching. And number **3000** means fee tier of **0.3%**

Due to the fact that pools with fee tier of **0.01%** may consume much more gas than others. We need to
limit the usage of such pools. 
And field **support001Pools** is a list containing supported pool with fee tier of **0.01%**.
When we are doing path searching, pools with **0.01%** fee tier and outside that list will not be considered.
Each element in **support001Pools** is a struct of **PoolPair**.

If we want to ignore some pool, we can fill the field **poolBlackList**.
Element in **poolBlackList** is also struct of **PoolPair**.

*In general, the supported fee rates for the mainnet are 500 (0.05%), 3000 (0.3%), and 10000 (1%); and for the testnet are 400 (0.04%), 2000 (0.2%) and 10000 (1%). One needs to check if the choosen pool exists and has enough liquidity.*
*The liquidity condition can be checked on the analytics page* `here <https://analytics.izumi.finance>`__ .

.. _diff_for_exact_output:

4. difference for mode of exact output
---------------------------------------------------------------

If we want to do path searching in the mode of **swap with exact output**, 
we need to fill **SearchPathQueryParams.direction** with **SwapDirection.ExactOut**
and to fill **SearchPathQueryParams.amount** with **undecimal amount of output token (USDT)** in the above code.

You can refer the codes `here <https://github.com/izumiFinance/iZiSwap-sdk/blob/main/example/search/searchWithExactOutput.ts>`_
for more details.

5. Searching!
---------------------------------------------------------------


.. code-block:: typescript
    :linenos:

    // pathQueryResult stores optimized swap-path 
    //     and estimated swap-amount (output amount for exactIn, and input amount for exactOut)
    // preQueryResult caches data of pools and their state (current point) 
    //     which will be used during path-searching
    //     preQueryResult can be used for speed-up for next search
    //     etc, if you want to speed up a little next search, 
    //     just use following code:
    //     await searchPathQuery(searchParams, preQueryResult)
    //     cached data in preQueryResult can be used for different
    //     pair of <inputToken, outputToken> or different direction
    //     but notice that, cached data in preQueryResult can not be
    //     used in different chain
    const {pathQueryResult, preQueryResult} = await searchPathQuery(
        searchParams
    )


5. take result
---------------------------------------------------------------

After calling **searchPathQuery**, we can print the optimized path.

.. code-block:: typescript
    :linenos:

    // print output amount
    console.log('output amount: ', pathQueryResult.amount)
    // print path info
    // which can be filled to swap params
    // see example of "example/quoterAndSwap/"
    console.log('fee chain: ', pathQueryResult.path.feeContractNumber)
    console.log('token chain: ', pathQueryResult.path.tokenChain)

In the above code,
**pathQueryResult.path.feeContractNumber** is type of **number[]**
and **pathQueryResult.path.tokenChain** is type of **TokenInfoFormatted[]**
data of these 2 arrays
can be used to fill the fields named **feeChain** and **tokenChain**
in 2 structs named **SwapChainWithExactInputParams** and **SwapChainWithExactOutputParams** which
is used as input parameter for interfaces of **swap**.

The fields of **SwapChainWithExactInputParams** and **SwapChainWithExactOutputParams** can be viewed in following code.

.. code-block:: typescript
    :linenos:

    export interface SwapChainWithExactInputParams {
        // input: tokenChain[0]
        // output: tokenChain[1]
        tokenChain: TokenInfoFormatted[];
        // fee / 1e6 is feeTier
        // 3000 means 0.3%
        feeChain: number[];
        // 10-decimal format integer number, like 100, 150000, ...
        // or hex format number start with '0x'
        // decimal amount = inputAmount / (10 ** inputToken.decimal)
        inputAmount: string;
        minOutputAmount: string;
        recipient?: string;
        deadline?: string;
        // true if treat wrapped coin(wbnb or weth ...) as erc20 token
        strictERC20Token?: boolean;
    }

    export interface SwapChainWithExactOutputParams {
        // input: tokenChain[0]
        // output: tokenChain[1]
        tokenChain: TokenInfoFormatted[];
        // fee / 1e6 is feeTier
        // 3000 means 0.3%
        feeChain: number[];
        // 10-decimal format number, like 100, 150000, ...
        // or hex format number start with '0x'
        // amount = outputAmount / (10 ** outputToken.decimal)
        outputAmount: string;
        maxInputAmount: string;
        recipient?: string;
        deadline?: string;
        // true if treat wrapped coin(wbnb or weth ...) as erc20 token
        strictERC20Token?: boolean;
    }

And the example codes of calling interfaces of **swap** can be viewed in :ref:`quoter_swap_chain_with_exact_input` or :ref:`quoter_swap_chain_with_exact_output`

