Get pool and create pool
================================

In this section, we provide a simple example for getting pool address of a pair or creating a pool. 

The full example code of this chapter can be spotted `here <https://github.com/izumiFinance/iZiSwap-sdk/blob/main/example/pool/createPoolAndGetPool.ts>`_.

You should notice that, example and interfaces in this chapter require the sdk with minimum version of **1.5.0**.

1. Some imports
---------------

.. code-block:: typescript
    :linenos:

    import {BaseChain, ChainId, initialChainTable} from 'iziswap-sdk/lib/base/types'
    import {privateKey} from '../../.secret'
    import Web3 from 'web3';
    import { getCreatePoolCall, getFactoryContract, getPoolAddress } from 'iziswap-sdk/lib/pool/funcs';
    import { BigNumber } from 'bignumber.js'

Detail of these imports can be viewed in the following content.

.. _base_obj_create_get_pool:

2. Specify chain, rpc, web3, and account
--------------------------------------------------

.. code-block:: typescript
    :linenos:

    const chain:BaseChain = initialChainTable[ChainId.BSCTestnet]
    const rpc = 'https://data-seed-prebsc-1-s3.binance.org:8545/'
    console.log('rpc: ', rpc)
    const web3 = new Web3(new Web3.providers.HttpProvider(rpc))
    const account =  web3.eth.accounts.privateKeyToAccount(privateKey)
    console.log('address: ', account.address)

where

* - **BaseChain** is a data structure to describe a chain, in this example we use **bsc test** chain.
* - **ChainId** is an enum to describe **chain id**, value of the enum is equal to value of **chain id**.
* - **initialChainTable** is a mapping from some most used **ChainId** to **BaseChain**. You can fill fields of BaseChain by yourself.
* - **privateKey** is a string, which is your private key, and should be configured by your self.
* - **web3** is a public package to interact with block chain.
* - **rpc** is the rpc url on the chain you specified.

.. _FactoryContract_forCreate:

3. Get web3.eth.Contract object of factory
---------------------------------------------------

.. code-block:: typescript
    :linenos:

    const factoryAddress = '0x7fc0574eAe768B109EF38BC32665e6421c52Ee9d'
    const factoryContract = getFactoryContract(factoryAddress, web3)

Here, **getFactoryContract** is an api provided by our sdk, which returns a **web3.eth.Contract** object of **FactoryContract**.

4. Choose token pair of the pool
---------------------------------------------------------

.. code-block:: typescript
    :linenos:

    
    const testAAddress = '0xCFD8A067e1fa03474e79Be646c5f6b6A27847399'
    const testBAddress = '0xAD1F11FBB288Cd13819cCB9397E59FAAB4Cdc16F'

    // feeRate = feeContractNumber / 1e6
    // etc, 3000 means 0.3%
    // you should choose a proper feeRate of new pool
    // which should be supported by factory on that chain
    const feeContractNumber = 400;

Here, we will create a pool of pair **(testAAddress, testBAddress, feeContractNumber)**,
Where

 * - **testAAddress**: address of an erc20 token.
 * - **testBAddress**: address of another erc20 token.
 * - **feeContractNumber**: an int number, fee/1e6 is fee rate of pool, etc, 2000 means 0.2% fee rate.
  
You can fill **testAAddress** and **testBAddress** with your expected erc20 token address,
and fill **feeContractNumber** with your expected fee number. 

But notice that, when you fill value of **feeContractNumber**, you should garrantee
that the corresponding fee rate is supported by **iZiSwap** on that chain.

5. Check whether the pool has been created
---------------------------------------------------------

.. code-block:: typescript
    :linenos:

    const poolAddress = await getPoolAddress(factoryContract, testAAddress, testBAddress, feeContractNumber);
    if (!(new BigNumber(poolAddress).eq('0'))) {
        // if poolAddress is not zero address,
        //     this means some one has created the pool with same token pair (testAAddress, testBAddress, feeContractNumber) before.
        //     in this situation,  we can not create pool with same token pair (testAAddress, testBAddress, feeContractNumber) in the following code.
        console.log('pool has been created! Address: ', poolAddress);
        return;
    }

The function **getPoolAddress(...)** imported from **'iziswap-sdk/lib/pool/funcs'** queries **factoryContract** to get iZiSwap pool address of 
token pair **(testAAddress, testBAddress, feeContractNumber)**, where

 * - **factoryContract**: iZiSwap Factory contract, acquired in step 3.
 * - **testAAddress**: address of an erc20 token, filled in step 4.
 * - **testBAddress**: address of another erc20 token, filled in step 4.
 * - **feeContractNumber**: an int number, fee/1e6 is fee rate of pool, etc, 2000 means 0.2% fee rate.

You should notice that the function **getPoolAddress** imported from **'iziswap-sdk/lib/pool/funcs'** requires the sdk with minimum version of **1.5.0**.

If **poolAddress** is zero address, we can use following steps to create a pool of this pair,
otherwise, we cannot create pool with this pair again.

And due to the fact this example has been tested, which means the pool of this token pair **(testAAddress, testBAddress, feeContractNumber)** has been created successfully.
If you donot modify any element (**testAAddress** or **testBAddress** or **feeContractNumber**) of this token pair and run this example directly, your code will exit in this step.

6. Get calling for creating the pool.
---------------------------------------------------------

.. code-block:: typescript
    :linenos:

    // you can choose a proper initial point, which
    // specify init price of tokenX (by tokenY)
    const initPointXByY = 100;
    const gasPrice = '5000000000';

    // get calling
    // before calling getCreatePoolCall(...)
    // we should garrentee that
    // tokenXAddress.toLowerCase() < tokenYAddress.toLowerCase()
    let tokenXAddress = testAAddress;
    let tokenYAddress = testBAddress;
    if (tokenXAddress.toLowerCase() > tokenYAddress.toLowerCase()) {
        tokenXAddress = testBAddress;
        tokenYAddress = testAAddress;
    }
    const {createPoolCalling, options} = getCreatePoolCall(
        factoryContract,
        tokenXAddress,
        tokenYAddress,
        feeContractNumber,
        initPointXByY,
        account.address,
        chain,
        gasPrice,
    )

The function **getCreatePoolCall(...)** imported from **'iziswap-sdk/lib/pool/funcs'** will return corresponding txn data for creating a new pool,
where

 * - **factoryContract**: iZiSwap Factory contract, acquired in step 3.
 * - **tokenXAddress**: address of tokenX, here tokenXAddress is `min{tokenAAddress, tokenBAddress}`.
 * - **tokenYAddress**: address of tokenY, here tokenYAddress is `max{tokenAAddress, tokenBAddress}`.
 * - **feeContractNumber**: an int number, fee/1e6 is fee rate of pool, etc, 2000 means 0.2% fee rate.
 * - **initPointXByY**: an int number specifying initial point (see :ref:`here<point>`) of the pool.
 * - **account.address**: address of your private key, acquired in step 2.
 * - **chain**: chain object, acquired in step 2.
 * - **gasPrice**: gas price, string or int number.

The function **getCreatePoolCall** returns 2 object, **createPoolCalling** and **options**.
When **createPoolCalling** and **options** are ready, we can estimate gas.

When calling this function, you should garrentee that **tokenXAddress.toLowerCase() < tokenYAddress.toLowerCase()**, otherwise the returned
**createPoolCalling** and **options** will be undefined.

Another thing you should notice is that the function **getCreatePoolCall** imported from **'iziswap-sdk/lib/pool/funcs'** requires the sdk with minimum version of **1.5.0**.

7.   Estimate gas (optional)
-----------------------------
You can skip this step if you do not want to limit gas.

.. code-block:: typescript
    :linenos:

    // esitmate gas
    // if error occurs when estimating gas,
    //     this means some one might create the pool before with
    //     same token pair (testAAddress, testBAddress, feeContractNumber) before
    //     you can call getPoolAddress(...) to get the pool address
    //     as mentioned above
    const gasLimit = await createPoolCalling.estimateGas(options)
    console.log('gas limit: ', gasLimit)

8.  Finally, send transaction!
------------------------------

Now, we can send transaction to mint a new liquidity position.

For metamask or other injected wallet provider, you can easily write 

.. code-block:: typescript
    :linenos:

    await createPoolCalling.send({...options, gas: new BigNumber(gasLimit * 1.1).toFixed(0, 2)})

Otherwise, if you are running codes in console, you could use the following code

.. code-block:: typescript
    :linenos:

    // sign transaction
    const signedTx = await web3.eth.accounts.signTransaction(
        {
            ...options,
            to: factoryAddress,
            data: createPoolCalling.encodeABI(),
            gas: new BigNumber(gasLimit * 1.1).toFixed(0, 2),
        }, 
        privateKey
    )
    // send transaction
    const tx = await web3.eth.sendSignedTransaction(signedTx.rawTransaction);
    console.log('tx: ', tx);

Finally, we have successfully created a pool (if no revert occurred).