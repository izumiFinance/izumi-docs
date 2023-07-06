Amount of ERC20 Token
=====================

In terms of amount (quantity), there are two different representations for the token: `decimal amount` and `undecimal amount`.

Decimal amount
--------------

**Decimal amount** is the quantity in the sense of natural language.

For example, we use `USDT` to buy `BNB` or use `BNB` to buy `USDT`,  we need to pay `600 usdt` for `2 bnb` or need to pay `0.0099 bnb` for `3 usdt`.
Here,  we call these numbers `600`, `2`, `0.0099`, `3` as `decimal amount`.

`decimal amount`, is the amount of token we usually see in CEX or the frontend of DEX.

`decimal amount` may not be an integer, e.g., `0.0099 bnb` or `1.25 eth`.


Undecimal amount
----------------

**Undecimal Amount** is the quantity used in EVM.

EVM has no data type for float,  and all data type for number are integer, e.g., `uint8` `uint256` `int128` ...

Since `decimal amount` may be float number, a problem is how to represent some float number amounts of token like `1.25 bnb`.

When we want to transfer `1.25 bnb` to an address, the raw data we see in the transaction is `1,250,000,000,000,000,000`, which is `1.25 * (10 ** 18)`.
Here, `18` is decimals value of bnb, `1.25` is `decimal amount` mentioned above, and the raw data in the transaction, `1,250,000,000,000,000,000`, is called `undecimal amount`.

Each erc-20 token has a view function named `decimals()`, which will return decimal value of that token.
And we can get following relationship between `undecimal amount` and `decimal amount`:

.. code-block:: typescript
    :linenos:

    undecimal_amount = decimal_amount * (10 ** decimal_of_token)

In our example, decimals value of bnb on `bsc chain` is 18. Thus, `decimal amount` of `1.25 bnb` means `undecimal amount` `1,250,000,000,000,000,000`.

Usually, most erc-20 token's decimal value on most chains is `18`, because `10**18` is nearly `2**64`. However, for some erc-20 token, they may have other decimal value. decimal value of usdt or usdc is 6 on most chains.
