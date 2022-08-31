Amount of ERC20 Token
=====================

here, we describe concepts about amount.

etc, `decimal amount` and `undecimal amount`

decimal amount
--------------

for example, we use `USDT` to buy `BNB` or use `BNB` to buy `USDT`

suppose on a centralized dex, we need to pay `600 usdt` for `2 bnb` or need to pay `0.0099 bnb` for `3 usdt`

here,  we call these numbers `600` `2` `0.0099` `3`, as `decimal amount`

`decimal amount`, is the amount of token we usually see on centralized dex

`decimal amount` may not be an integer like `0.0099 bnb` or `1.25 eth`


undecimal amount
----------------

in evm, we donot has data type for float, all data type for number are integer, like `uint8` `uint256` `int128` ...

but `decimal amount` may be float number.

how to describe some float number amount of token like `1.25 bnb`?

when we want to transfer `1.25 bnb` to an address, the raw data we see in the transaction is `1,250,000,000,000,000,000` which is `1.25 * (10 ** 18)`

here, `18` is decimals value of bnb, `1.25` is `decimal amount` mentioned above, and the raw data in the transaction, `1,250,000,000,000,000,000`, is called `undecimal amount`

Each erc-20 token has a view function named `decimals()`, which will return decimal value of that token

we can get following fomula for `undecimal amount` and `decimal amount`

.. code-block:: typescript
    :linenos:

    undecimal_amount = decimal_amount * (10 ** decimal_of_token)

in our example, decimals value of bnb on `bsc chain` is 18, `decimal amount` of `1.25 bnb` means `undecimal amount` `1,250,000,000,000,000,000`

usually, most erc-20 token's decimal value on most chains is `18`, because `10**18` is nearly `2**64`

but for some erc-20 token,  they may have other decimal value. decimal value of usdt or usdc is 6 on most chains
