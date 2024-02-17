Subgraph
==================

iZiSwap provides GraphQL-based indexing on most supported blockchain networks. The relevant repository can be found `here <https://github.com/izumiFinance/iZUMi-iZiSwap-theGraph.git>`_.


* `zkSync Era <https://api.studio.thegraph.com/query/24334/izumi-zksync-subgraph/version/latest>`_
* `BNB Chain <https://api.thegraph.com/subgraphs/name/lpcaries/izumi-subgraph-bsc>`_
* `Base <https://api.thegraph.com/subgraphs/name/lpcaries/izumi-subgraph-base>`_
* `Arbitrum <https://api.studio.thegraph.com/query/24334/izumi-subgraph-arbitrum/version/latest>`_
* `Manta Pacific <https://api.goldsky.com/api/public/project_clo2asxoz0tlq2ntvfwz7gpay/subgraphs/izumi-manta-subgraph/1.0.1/gn>`_
* `Linea <https://graph-node-api.izumi.finance/query/subgraphs/name/izi-swap-linea/graphql?query=query+%7B%0A%09swaps%28first%3A+10%2C+orderBy%3A+timestamp%2C+orderDirection%3A+desc%29+%7B%0A%09%09pool+%7B%0A++++++id%0A++++++tvlUSD%0A++++%7D%0A++++transaction+%7B%0A++++++blockNumber%0A++++%7D%0A++++timestamp%0A++%7D%0A%7D>`_
* `Mantle <https://graph-node-api.izumi.finance/query/subgraphs/name/izi-swap-mantle/graphql?query=query+%7B%0A%09swaps%28first%3A+10%2C+orderBy%3A+timestamp%2C+orderDirection%3A+desc%29+%7B%0A%09%09pool+%7B%0A++++++id%0A++++++tvlUSD%0A++++%7D%0A++++transaction+%7B%0A++++++blockNumber%0A++++%7D%0A++++timestamp%0A++%7D%0A%7D>`_
* `Kroma <https://graph-node-api.izumi.finance/query/subgraphs/name/izi-swap-kroma/graphql>`_


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
