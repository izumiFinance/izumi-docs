.. _price:

Price of ERC20 token
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

see next section `point`.
