

Swap Simulation (for integration)
----------------------------------


When we are making price inquiries, sometimes we prefer not to invoke remote RPC operations every time. 
This is often due to limitations on RPC access frequency and slower computational speed. 
A typical scenario is a DEX aggregator that requires numerous calls to quoters in order to split orders and find the optimal trading path.
And we provide a TS (almost) equivalent implementation of the swap procedure in iZiSwap's core contracts for these integration usage.

In this tutorial, we mainly talk about **swapX2Y** mode, other modes (**swapX2YDesireY**,
**swapY2X**, **swapY2XDesireX**) are similar to this.

If you are unfamiliar with those 4 **swap...** modes, you can quickly refer to 
:ref:`this page <swap_mode>`

.. toctree::
   :maxdepth: 1

   swap_query_x2y_with_offline_data
   swap_query_x2y_with_online_data


*We also provide a GoLang simulation version. One can refer to this Github* `repo <https://github.com/izumiFinance/iZiSwap-SDK-go>`_.