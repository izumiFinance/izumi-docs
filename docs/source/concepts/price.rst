Price of erc20 token
=====================

here, we describe concepts about price

etc, `decimal price` and `undecimal price` `point`

decimal price
-------------

we continue to take use of example `USDT` and `BNB`

suppose on a centralized dex, we need to pay `300 usdt` for `1 bnb` or need to pay `0.0033 bnb` for `1 usdt`,

then we say the price of `bnb by usdt` is `300 usdt/bnb` or we say the price of `usdt by bnb` is `0.0033 bnb/usdt`

here, the price `300 usdt/bnb` or `0.0033 bnb/usdt` is called `decimal price` in our sdk

generally,  `tokenA` and `tokenB` as 2 erc20-token, and we assume that we need to pay `3.3`  tokenB to acquire `1.0` tokenA.
here, `3.0` and `1.0` are decimal amounts of tokenB and tokenA during this trade. 

then the `decimal price` of `A by B` is `3.3 (tokenB / tokenA)`, 
here we use `A by B` stand for `value of tokenA counted  by tokenB`, 
it means how much tokenB to acquire one tokenA , 
and in the above example `decimal_price_BNB_by_USDT` is `300 usdt/bnb`

.. code-block:: typescript
    :caption: decimal price
    :name: decimal price
    :linenos:

    decimal_amount_A * decimal_price_A_by_B = decimal_amount_B
    1.0 (tokenA)     * 3.3 (tokenB/tokenA)  = 3.3 (tokenB)

notice: in above formula , A_by_B correspond to (tokenB / tokenA), B_by_A correspond to (tokenA / tokenB)

`decimal price` is price based on `decimal amount`


undecimal price
---------------

as mentioned in `1.2 undecimal amount`, there is no `decimal amount` on block chain, we can only see `undecimal amount`
so we need `undecimal price` which describe price based on `undecimal amount`

see the bnb/usdt example in `2.1` and replace `decimal amount` to `undecimal amount`
and we can see that we pay `300,000,000 undecimal-amount` of usdt for `1,000,000,000,000,000,000 undecimal-amount` of bnb (`trade1`), or we pay `3,300,000,000,000,000 undecimal-amount` of bnb for `1,000,000 undecimal-amount` of usdt (`trade2`)

the `undecimal price` of `trade1` is `300,000,000 / 1,000,000,000,000,000,000 (usdt/ bnb)`
the `undecimal price` of `trade2` is `3,300,000,000,000,000 / 1,000,000 (bnb / usdt)`

more generaly, from `decimal price` formular of `tokenA` and `tokenB` in code block :ref:`decimal price`

we can get

.. code-block:: typescript
    :linenos:

    decimal_amount_A * (10 ** decimalA) * decimal_price_A_by_B * (10 ** decimalB / 10 ** decimalA) = decimal_amount_B * (10 ** decimalB)

which is

.. code-block:: typescript
    :linenos:

    undecimal_amount_A * (decimal_price_A_by_B * (10 ** decimalB / 10 ** decimalA)) = undecimal_amount_B

and we can get

.. code-block:: typescript
    :caption: undecimal price
    :name: undecimal price
    :linenos:

    undecimal_price_A_by_B = decimal_price_A_by_B * (10 ** decimalB / 10 ** decimalA)

also, like :ref:`decimal price`, here we use `A by B` stand for `value of tokenA counted  by tokenB`

and in the example of `usdt` and `bnb` `decimal_price_BNB_by_USDT` is `300 usdt/bnb`
 
`undecimal_price_BNB_by_USDT` is ` 3 * 10**(-10)`

there is a problem, `undecimal price` usually is a float number, how to describe them in contract

see next section `point`

point
-----

in swap pool contracts, the **undecimal price** is discreted to points

a **point** is a integer number in (-800000, 800000)

they have following relationship:

.. code-block:: typescript
    :linenos:

    undecimal_price = 1.0001 ** point

current point of swap pool, and point on pool
---------------------------------------------

Each izi-swap pool supports a trade pair, (tokenX, tokenY)

address of tokenX and tokenY can be queried from pool's view function `tokenX()` and `tokenY()`

there is a **restrict** for each pool that dictionary order of **tokenX lower case address** must be smaller than **tokenY lower case address**


**current point** of swap pool is point of **undecimal price X by Y** after last trade in this pool.

**current point** value can be queried from pool's view function `state()`

when we know **current point**, we can get **current undecimal price X by Y**

.. code-block:: typescript
    :linenos:

    current_undecimal_price_X_by_Y = 1.0001 ** current_point

and we can get **current undecimal price Y by X**

.. code-block:: typescript
    :linenos:

    current_undecimal_price_Y_by_X = 1.0001 ** (-current_point)

point on pool
-------------

**point_on_pool** describe **undecimal_price_X_by_Y**, not **undecimal_price_Y_by_X**

.. code-block:: typescript
    :name: point on pool
    :caption: point on pool
    :linenos:

    undecimal_price_X_by_Y = 1.0001 ** point_on_pool

**tokenX** is the token with smaller address in the pool's trade pair

**tokenY** is the token with larger address in the pool's trade pair

transform **undecimal price** to **point** on pool
--------------------------------------------------

suppose we have 2 tokens, **tokenA** and **tokenB**, and we know undecimal price **undecimal_price_A_by_B**, how to transform **undecimal_price_A_by_B** to corresponding **point** on the pool

first, compare dictionary order of tokenA and tokenB, as mentioned in :ref:`point on pool`, tokenX of pool is token with smaller address amoung tokenA and tokenB
then we can get `undecimal_price_X_by_Y`. etc

.. code-block:: typescript
    :linenos:

    if (tokenA.address.toLowerCase() < tokenB.address.toLowerCase())
        undecimal_price_X_by_Y = undecimal_price_A_by_B
    else
        undecimal_price_X_by_Y = 1.0 / undecimal_price_A_by_B

secondly we use fomula in :ref:`point on pool` to compute **point** on the pool

.. code-block:: typescript
    :linenos:

    point_on_pool = Math.round(Math.log(1.0001, undecimal_price_X_by_Y))