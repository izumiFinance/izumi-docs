swap with exact input amount
================================

here, we provide a simple example for swap with exact input amount through box's interface

The full example code of this chapter can be spotted `here <https://github.com/izumiFinance/izumi-iZiSwap-sdk/blob/main/example/box/swapWithExactInput.ts>`_.


1. some imports
---------------

.. code-block:: typescript
    :linenos:

    import {BaseChain, ChainId, initialChainTable, TokenInfoFormatted} from 'iziswap-sdk/lib/base/types'
    import {privateKey} from '../../.secret'
    import Web3 from 'web3';
    import { BigNumber } from 'bignumber.js'
    import { getBoxContract, getSwapChainWithExactInputCall, SwapChainWithExactInputParams } from 'iziswap-sdk/lib/box';

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

.. _BoxContract_forSwapAmount:

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


5. construct params and get swap calling
------------------------------------------------------------------

.. code-block:: typescript
    :linenos:

    const swapChainWithExactInputParams = {
        tokenChain: [feeB, wBNB],
        feeChain: [2000],
        inputAmount: '100000000000000000',
        minOutputAmount: '180000000000000000'
    } as SwapChainWithExactInputParams

    const gasPrice = '15000000000'

    const { swapCalling, options } = getSwapChainWithExactInputCall(
        boxContract,
        account.address,
        chain,
        swapChainWithExactInputParams,
        gasPrice
    )

in the above code, we ready to pay some **FeeB** to earn some **BNB**.
**tokenChain** and **feeChain** in **SwapChainWithExactInputParams** describe the swap path of this trading.
the **SwapChainWithExactInputParams.inputAmount** is undecimal amount of token we want to pay.
here, we wants to pay "100000000000000000" of token **FeeB**.

the field **minOutputAmount** of **SwapChainWithExactInputParams** is minimum undecimal amount of **BNB** we want to get.
we can determine this value in a finer way by querying quoter (see :ref:`here<quoter_swap_chain_with_exact_input_query>`)

we should notice that, if the input token (etc, tokenChain[0]) has transfer fee, we should take a relatively lower **minOutputAmount**,
because when **Box** contract calls **Swap** contract,
user will pay **WrapToken** and the amount of it is less than **inputAmount**.
this is due to the transfer fee cost in **WrapToken.deposit(...)**.
and the **Swap** contract will see that user get less output token.

but if the input token has no transfer fee, whether output token (etc, tokenChain[tokenChain.length - 1]) has transfer fee or not,
we could take a higher **minOutputAmount** as usual and not necessary to use a lower value,
because all transfers after box calling **Swap**'s interface, including user paying, transfering between pools, and user acquiring,
donot has transfer fee (token with transfer fee donot exist in iZiSwap, but their wrap token).
and the **Swap** contract will **not** see user get less output token. And **Box** will not check amount of user's acquire.

6. approve
---------------------------
notice that you should do following steps before estimate gas or send transaction in this "swap" case.

first, if **FeeB** is input token of this trading, you should approve box to deposit your **FeeB** token to corresponding **WrapToken**, 
because box will call **deposit** interface of **WrapToken** to help you deposit your **FeeB**, the box needs your approve.
you can view **depositApprove** interface of **WrapToken** contract for more information.

second, if **FeeB** is input token of this trading, you should approve **WrapToken** to transfer your **FeeB** token, because in **deposit** interface of **WrapToken**,
the **WrapToken** contract call transfer interface of **FeeB** to transfer your **FeeB** token, and **WrapToken** needs your approve.

thirdly, if input token is **USDT** or **iZi** or other normal erc20 token instead of wbnb/weth,
you should approve **Box** to transfer your corresponding erc20 token

in this case, token **FeeB** is token with transfer fee, and we should do following 2 steps to approve.

first, calling **depositApprove** to give boxContract authority to call **depositFrom** of feeB's **wrapToken**

.. code-block:: typescript
    :linenos:

    const wrapTokenABI = [
        {
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
            "name": "depositApprove",
            "outputs": [],
            "stateMutability": "nonpayable",
            "type": "function"
        },
        {
            "inputs": [
                {
                "internalType": "address",
                "name": "from",
                "type": "address"
                },
                {
                "internalType": "address",
                "name": "to",
                "type": "address"
                },
                {
                "internalType": "uint256",
                "name": "amount",
                "type": "uint256"
                }
            ],
            "name": "depositFrom",
            "outputs": [
                {
                "internalType": "uint256",
                "name": "actualAmount",
                "type": "uint256"
                }
            ],
            "stateMutability": "nonpayable",
            "type": "function"
        },
    ]
    const wrapTokenContract = web3.eth.Contract(wrapTokenABI, feeB.wrapTokenAddress)
    const depositApproveCalling = wrapTokenContract.methods.depositApprove(boxAddress, '0xffffffffffffffffffffffffffffffff')
    const depositApproveGasLimit = depositApproveCalling.estimateGas({from: account})
    await depositApproveCalling.send({gas: depositApproveGasLimit})

second, calling **approve** to give feeB's **wrapToken** authority to operate your feeB token

.. code-block:: typescript
    :linenos:

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
    const feeBContract = new web3.eth.Contract(erc20ABI, feeB.address);
    // you could approve a very large amount (much more greater than amount to transfer),
    // and don't worry about that because feeB's wrapTokenContract only transfer your token to it with amount you specified and your token is safe
    // then you do not need to approve next time for this user's address
    const approveCalling = feeBContract.methods.approve(
        feeB.wrapTokenAddress, 
        "0xffffffffffffffffffffffffffffffff"
    );
    // estimate gas
    const gasLimit = await approveCalling.estimateGas({})
    // then send transaction to approve
    // you could simply use followiing line if you use metamask in your frontend code
    // otherwise, you should use the function "web3.eth.accounts.signTransaction"
    // notice that, sending transaction for approve may fail if you have approved the token to swapContract before
    // if you want to enlarge approve amount, you should refer to interface of erc20 token
    await approveCalling.send({gas: Number(gasLimit)})


if your input token is a normal erc20 token which has no transfer fee (like iZi or USDT),
you just need to write following code instead of 2 steps above (we suppose the input token is testA)

.. code-block:: typescript
    :linenos:

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
    // suppose the input token is "testA"
    const testAAddress = '0xCFD8A067e1fa03474e79Be646c5f6b6A27847399'
    const testAContract = new web3.eth.Contract(erc20ABI, testAAddress);
    // you could approve a very large amount (much more greater than amount to transfer),
    // and don't worry about that because boxContract only transfer your token to it with amount you specified and your token is safe
    // then you do not need to approve next time for this user's address
    const approveCalling = testAContract.methods.approve(
        boxAddress, 
        "0xffffffffffffffffffffffffffffffff"
    );
    // estimate gas
    const gasLimit = await approveCalling.estimateGas({})
    // then send transaction to approve
    // you could simply use followiing line if you use metamask in your frontend code
    // otherwise, you should use the function "web3.eth.accounts.signTransaction"
    // notice that, sending transaction for approve may fail if you have approved the token to swapContract before
    // if you want to enlarge approve amount, you should refer to interface of erc20 token
    await approveCalling.send({gas: Number(gasLimit)})


7.  estimate gas (optional)
---------------------------
of course you can skip this step if you donot want to limit gas.

if you want, you can estimate or send the transaction

.. code-block:: typescript
    :linenos:

    const gasLimit = await swapCalling.estimateGas(options)

8.  finally, send transaction!
------------------------------

notice that you should do following steps before estimate gas or send transaction in this "mint" case.

first, if **FeeB** is input token of this trading, you should approve box to deposit your **FeeB** token to corresponding **WrapToken**, 
because box will call **deposit** interface of **WrapToken** to help you deposit your **FeeB**, the box needs your approve.
you can view **depositApprove** interface of **WrapToken** contract for more information.

second, if **FeeB** is input token of this trading, you should approve **WrapToken** to transfer your **FeeB** token, because in **deposit** interface of **WrapToken**,
the **WrapToken** contract call transfer interface of **FeeB** to transfer your **FeeB** token, and **WrapToken** needs your approve.

thirdly, if input token is **USDT** or **iZi** or other normal erc20 token instead of wbnb/weth,
you should approve **Box** to transfer your corresponding erc20 token,
you can view interfaces corresponding to approve or approval in erc20's interfaces for more information.

after above steps, you can estimate or send the transaction

for metamask or other explorer's wallet provider, you can easily write 

.. code-block:: typescript
    :linenos:

    await swapCalling.send({...options, gas: Number(gasLimit)})

otherwise, if you are runing codes in console, you could use following code

.. code-block:: typescript
    :linenos:

    // sign transaction
    const signedTx = await web3.eth.accounts.signTransaction(
        {
            ...options,
            to: boxAddress,
            data: swapCalling.encodeABI(),
            gas: new BigNumber(Number(gasLimit) * 1.1).toFixed(0, 2),
        }, 
        privateKey
    )
    // send transaction
    const tx = await web3.eth.sendSignedTransaction(signedTx.rawTransaction);

after this step, we have successfully add liquidity on existing liqudity through **Box** (if no revert occured)