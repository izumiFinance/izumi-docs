.. _oracle:

Oracle
====================


iZiSwap, as a DEX, has the ability to provide price feeds externally. While the pools have real-time prices, real-time prices are susceptible to manipulation. Therefore, iZiSwap provides TWA (Time-Weighted Average) prices to external sources.

In this example, we will show how to query a recent TWA price point of a swap pool, in Solidity code.


Expand Observation Queue
-------------------------------

If we want to query a TWA price point of a swap pool, we should guarantee
that the observation queue has enough capacity.

One can call the `pool.state(...)` interface of a swap pool to get the current capacity (slots) of the queue.
Each slot records the latest trading price, and it is updated only once per block.


.. code-block:: solidity
    :linenos:

    interface IiZiSwapPool {
        function state()
            external view
            returns(
                uint160 sqrtPrice_96,
                int24 currentPoint,
                uint16 observationCurrentIndex,
                uint16 observationQueueLen,
                uint16 observationNextQueueLen,
                bool locked,
                uint128 liquidity,
                uint128 liquidityX
            );
    }

The returned value **observationNextQueueLen** is the current capacity of of the pool.

To expand the capacity of observation queue on a pool, just call following interface of swap pool.

.. code-block:: solidity
    :linenos:

    interface IiZiSwapPool {
        function expandObservationQueue(uint16 newNextQueueLen) external;
    }

Here **newNextQueueLen** is the new capacity you want to expand to.


Query TWA Price Point
-------------------------------

One can easily query recent TWA point from deployed
oracle contract by calling  the following interface.

.. code-block:: solidity
    :linenos:

    interface IOracle {
        function getTWAPoint(address pool, uint256 delta)
            external
            view
            returns (bool enough, int24 avgPoint, uint256 oldestTime);
    }

where the interface `getTWAPoint` has following params

.. code-block:: solidity
    :linenos:

    pool: address, swap pool address you want to query

    delta: uint256, seconds of time period. This interface calculates TWA point from delta ago to now, i.e., the time period is [block.Timestamp - delta, block.Timestamp]


The interface will return following values.

.. code-block:: solidity
    :linenos:

    enough: bool, whether oldest point in the observation queue is older than (currentTime - delta)

    avgPoint: int24, TWA point within [block.Timestamp - delta, block.Timestamp]

    oldestTime: the oldest time of point in the observation queue.


We also provide a simple example to call oracle's interface in solidity.

.. code-block:: solidity
    :linenos:

    pragma solidity ^0.8.4;

    interface IOracle {
        function getTWAPoint(address pool, uint256 delta)
            external
            view
            returns (bool enough, int24 avgPoint, uint256 oldestTime);
    }

    contract TestOracle {

        address public oracleAddress;

        constructor(address _oracleAddress) {
            oracleAddress = _oracleAddress;
        }

        function testOracle(address pool, uint256 delta)
            external
            view
            returns (bool enough, int24 avgPoint, uint256 oldestTime)
        {
            // call getTWAPoint interface
            (enough, avgPoint, oldestTime) = IOracle(oracleAddress).getTWAPoint(pool, delta);
        }
    }

The code above can also be spotted `here <https://github.com/izumiFinance/iZiSwap-periphery/blob/main/contracts/test/TestOracle.sol>`_.
