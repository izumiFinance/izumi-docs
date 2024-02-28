.. _oracle:

oracle
====================

in this example, we will display the way to
query recent time weighted average (TWA) point on a swap pool.

query in solidity
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
