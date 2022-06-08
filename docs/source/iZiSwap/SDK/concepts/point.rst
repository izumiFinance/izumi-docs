.. _point:

Point
=====================

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