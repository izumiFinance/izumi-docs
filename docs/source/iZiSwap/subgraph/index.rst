Subgraph
==================

iZiSwap provides GraphQL-based indexing on most supported blockchain networks. The relevant repository can be found `here <https://github.com/izumiFinance/iZUMi-iZiSwap-theGraph.git>`_.



* `zkSync Era <https://api.studio.thegraph.com/query/24334/izumi-zksync-subgraph/version/latest>`_
* `BNB Chain <https://api.thegraph.com/subgraphs/name/lpcaries/izumi-subgraph-bsc>`_
* `Base <https://api.thegraph.com/subgraphs/name/lpcaries/izumi-subgraph-base>`_
* `Mantle <https://graph.fusionx.finance/subgraphs/name/izumi-subgraph-mantle>`_
* `Arbitrum <https://api.studio.thegraph.com/query/24334/izumi-subgraph-arbitrum/version/latest>`_
* `Manta Pacific <https://api.goldsky.com/api/public/project_clo2asxoz0tlq2ntvfwz7gpay/subgraphs/izumi-manta-subgraph/1.0.1/gn>`_



example:

.. code-block:: 

   query swapsQuery($address: Bytes!, $timestamp: BigInt) {
      swaps(
         where: { account: $address, timestamp_gte: $timestamp }
         orderBy: timestamp
         orderDirection: asc
         first: 1000
      ) {
         amountX
         amountY
         amountUSD
         timestamp
      }
   }

   query positionsQuery($owner: Bytes!, $timestamp: BigInt) {
      liquidities(
         orderBy: transaction__timestamp
         first: 1000
         where: { owner: $owner, transaction_: { timestamp_gte: $timestamp } }
         orderDirection: asc
      ) {
         liquidity
         depositedTokenX
         depositedTokenY
         transaction {
            timestamp
         }
      }
   }
