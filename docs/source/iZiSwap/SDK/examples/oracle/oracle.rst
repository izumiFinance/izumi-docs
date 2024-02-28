.. _oracle:

oracle
====================

in this example, we will display the way to
query recent time weighted average (TWA) point on a swap pool.

expand observation queue
-------------------------------

if we want to query TWA point on a swap pool, we should guarantee
that the observation queue has enough capacity.

we can call `pool.state(...)` interface on swap pool to get current capacity
of the queue.

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

the returned value **observationNextQueueLen** is the current capacity of
of the pool.

to expand the capacity of observation queue on a pool,
just call following interface of swap pool.

.. code-block:: solidity
    :linenos:

    interface IiZiSwapPool {
        function expandObservationQueue(uint16 newNextQueueLen) external;
    }

in the above code, newNextQueueLen is the new
capacity you want to expand to.


Query TWA point in solidity
-------------------------------

we can easily query recent TWA point from deployed
oracle contract by easily calling following interface.

.. code-block:: solidity
    :linenos:

    interface IOracle {
        function getTWAPoint(address pool, uint256 delta)
            external
            view
            returns (bool enough, int24 avgPoint, uint256 oldestTime);
    }

in the above code, the interface `getTWAPoint` has following params

.. code-block:: solidity
    :linenos:

    pool: address, swap pool address you want to query
    delta: uint256, seconds of time period, 
        this interface calculates TWA point from delta ago to now
        etc, the time period is [block.Timestamp - delta, block.Timestamp]


And the interface will return following values.

.. code-block:: solidity
    :linenos:

    enough: bool, whether oldest point in the observation queue is older than (currentTime - delta)
    avgPoint: int24, TWA point within [block.Timestamp - delta, block.Timestamp]
    oldestTime: the oldest time of point in the observation queue.

and we also provide a simple example to call oracle's interface in solidity.

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
