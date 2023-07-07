.. _point:

Point
=====================

In iZiSwap pool contracts, the **undecimal price** is discreted into **points**.

A **point** is a integer number in range (-800000, 800000).

The relationship between the i-th **undecimal price** and **point** is:

.. code-block:: typescript
    :linenos:

    undecimal_price = 1.0001 ** point

Point on pool
----------------

Each iZiSwap pool supports a trade pair: (tokenX, tokenY), where addresses of tokenX and tokenY can be queried from the pool's view function `tokenX()` and `tokenY()`.

There is a **restrict** for each pool that dictionary order of **tokenX lower case address** must be smaller than **tokenY lower case address**, that is, 

**tokenX** is the token with smaller address in the pool's trade pair.

**tokenY** is the token with larger address in the pool's trade pair.

**point_on_pool** describe **undecimal_price_X_by_Y**, not **undecimal_price_Y_by_X**

.. code-block:: typescript
    :name: point on pool
    :caption: point on pool
    :linenos:

    undecimal_price_X_by_Y = 1.0001 ** point_on_pool



Current point of an iZiSwap pool
--------------------------------------------------

**current point** of a iZiSwap pool is the point of **undecimal price X by Y** after last trade in this pool, and the **current point** value can be queried from pool's view function `state()`.

If we know the  **current point**, we can get the **current undecimal price X by Y** by

.. code-block:: typescript
    :linenos:

    current_undecimal_price_X_by_Y = 1.0001 ** current_point

and the  **current undecimal price Y by X** by

.. code-block:: typescript
    :linenos:

    current_undecimal_price_Y_by_X = 1.0001 ** (-current_point)




Transform **undecimal price** to **point** on pool
--------------------------------------------------

Suppose that we have 2 tokens, **tokenA** and **tokenB**, and we know the undecimal price **undecimal_price_A_by_B**. 

To transform **undecimal_price_A_by_B** to the corresponding **point** on the pool,
first, compare dictionary order of tokenA and tokenB, as mentioned in :ref:`point on pool`, tokenX of pool is token with smaller address among tokenA and tokenB.

Now we can get `undecimal_price_X_by_Y` by

.. code-block:: typescript
    :linenos:

    if (tokenA.address.toLowerCase() < tokenB.address.toLowerCase())
        undecimal_price_X_by_Y = undecimal_price_A_by_B
    else
        undecimal_price_X_by_Y = 1.0 / undecimal_price_A_by_B

Then we use the formula in :ref:`point on pool` to compute **point** on the pool

.. code-block:: typescript
    :linenos:

    point_on_pool = Math.round(Math.log(1.0001, undecimal_price_X_by_Y))

*Strictly speaking, the transformation from `undecimal price` to `point` is an approximation. However, if the `undecimal price` comes from the iZiSwap system, 
the transformation is exact, since all prices from the system is discrete. Otherwise, the approximation is accurate enough for the most cases in reality.*