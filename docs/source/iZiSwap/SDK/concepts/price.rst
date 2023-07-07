.. _price:

Price of an ERC20 token
=========================

Similar to amount, there are two different representations for a token price: `decimal price` and `undecimal price`.

Decimal price
-------------

**Decimal price** is the price in the sense of natural language, and `decimal price` is the price based on `decimal amount`

we continue taking use of example `USDT` and `BNB`.

Suppose we need to pay `300 usdt` for `1 bnb` or need to pay `0.0033 bnb` for `1 usdt` during the trade, 
we say the price of `bnb by usdt` is `300 usdt/bnb` or the price of `usdt by bnb` is `0.0033 bnb/usdt`.
Here, the price `300 usdt/bnb` or `0.0033 bnb/usdt` is called `decimal price` in our sdk

Generally, for `tokenA` and `tokenB` ERC20 tokens, assume that we need to pay `3.3`  tokenB to acquire `1.0` tokenA,
where `3.0` and `1.0` are decimal amounts of tokenB and tokenA during this trade.

Then the `decimal price` of `A by B` is `3.3 (tokenB / tokenA)`, where `A by B` stands for `value of tokenA counted  by tokenB`.
And the `decimal_price_BNB_by_USDT` is `300 usdt/bnb`.

.. code-block:: typescript
    :caption: decimal price
    :name: decimal price
    :linenos:

    decimal_amount_A * decimal_price_A_by_B = decimal_amount_B
    1.0 (tokenA)     * 3.3 (tokenB/tokenA)  = 3.3 (tokenB),

where A_by_B correspond to (tokenB / tokenA) and B_by_A correspond to (tokenA / tokenB).



Undecimal price
-----------------

**Undecimal Price** is the price used in EVM, and is based on `undecimal amount`.

As mentioned in the `undecimal amount` section, there is no `decimal amount` on block chain, we can only see `undecimal amount`.
We need `undecimal price` which describe price based on `undecimal amount`.

In the bnb/usdt example, replace `decimal amount` to `undecimal amount`
and we can see that we pay `300,000,000 undecimal-amount` of usdt for `1,000,000,000,000,000,000 undecimal-amount` of bnb (`trade1`), or we pay `3,300,000,000,000,000 undecimal-amount` of bnb for `1,000,000 undecimal-amount` of usdt (`trade2`).

The `undecimal price` of `trade1` is `300,000,000 / 1,000,000,000,000,000,000 (usdt/ bnb)`.

The `undecimal price` of `trade2` is `3,300,000,000,000,000 / 1,000,000 (bnb / usdt)`.

Generally, from the `decimal price` formula of `tokenA` and `tokenB` in code block :ref:`decimal price`

We can get

.. code-block:: typescript
    :linenos:

    decimal_amount_A * (10 ** decimalA) * decimal_price_A_by_B * (10 ** decimalB / 10 ** decimalA) = decimal_amount_B * (10 ** decimalB)

which equals to

.. code-block:: typescript
    :linenos:

    undecimal_amount_A * (decimal_price_A_by_B * (10 ** decimalB / 10 ** decimalA)) = undecimal_amount_B


And finally we have:

.. code-block:: typescript
    :caption: undecimal price
    :name: undecimal price
    :linenos:

    undecimal_price_A_by_B = decimal_price_A_by_B * (10 ** decimalB / 10 ** decimalA)

Similar to  :ref:`decimal price`, we use `A by B` stand for `value of tokenA counted  by tokenB`.

In the example of `usdt` and `bnb`,  `decimal_price_BNB_by_USDT` is `300 usdt/bnb` and `undecimal_price_BNB_by_USDT` is ` 3 * 10**(-10)`.


Now a problem has arisen. Since `undecimal price` usually is a float number, 
check the next section `point` to find how to describe them in contract.

