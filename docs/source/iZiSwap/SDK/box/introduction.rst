Introduction
=============================

Transfer Fee
--------------

for some erc-20 token, fee may be charged during token transfer.

we cannot mint or add limit order or exchange them directly on iziswap.

to deal with this problem, we can deploy **Wrap Token** contract (similar to wETH and wBNB) for each such **"transfer fee"** token.

Wrap Token
--------------

we deploy **Wrap Token** contract for each token with **transfer fee** ready to put on iZiSwap.

user can call **deposit(...)** interface of **Wrap Token** to deposit corresponding origin token and get corresponding amount of **Wrap Token**.
the amount of **Wrap Token** is actual received amount of origin token.

also, user can call **withdraw(...)** interface of **Wrap Token** to burn some amount of **Wrap Token** and 
withdraw corresponding amount of origin token. 

**deposit** and **withdraw** is similar to wETH and wBNB.

There is no transfer fee when we transfer **Wrap Token** to each other. And in iZiSwap, user can mint or exchange
the **Wrap Token** instead of origin token.


mint or swap with Wrap Token
-----------------------------

before user mint or collect or exchange, if user wants to pay some token with transfer fee, he can first deposit them into **Wrap Token** and
acquire some amount of **Wrap Token**.

then, he/she can call mint or swap interfaces paying acquired **Wrap Token** instead of origin token.

after he/she mint or collect or exchange, the user can withdraw remained token or earned token.


Box
--------

for easier mining/exchanging with wrap token, we provide a contract named box packaging some mint or swap interfaces.

user can call these interfaces to provide or get origin token. transformation from origin token to wrap token is occured

inside **Box** interfaces.

and also, we provide corresponding frontend sdk to call these interfaces.

currently, **Box** donot support **adding/decreasing/collecting** limit order, if you want to do these with "transfer fee" token.
you should transform the token to corresponding **Wrap Token** by yourself.

**Box** only support **operations of liquidity** and **swap with exact input amount**

the intefaces packaged in box are following.

**Box.mint(...)** is to mint a liquidity to iZiSwap, the token can be **Wrap Token**, **Wraped Chain Token** or normal erc-20 token.

**Box.addLiquidity(...)** is to add some liquidity to exising liquidity on iZiSwap, the token can be **Wrap Token**, **Wraped Chain Token** or normal erc-20 token.

**Box.collect(...)** is to collect tokens from user's liquidity, the token can be **Wrap Token**, **Wraped Chain Token** or normal erc-20 token.

**Box.decreaseLiquidity(...)** is to decrease some liquidity and withdraw corresponding token, and collect all fee gained by this liquidity.
the token can be **Wrap Token**, **Wraped Chain Token** or normal erc-20 token.