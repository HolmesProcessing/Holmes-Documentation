totem.conf
------------

The modifications to ``totem.conf`` are very simple and consist of inserting
your service into the enrichers dictionary. This tells Totem your service exists,
where to reach it, and where to store its results:

.. code-block:: shell
    
    enrichers {
      ...
      yourservice {
        uri = ["http://address:port/yourservice/"]
        resultRoutingKey = "yourservice.result.static.totem"
      }
    }

In case you have multiple instances for your service, you can provide several
addresses to the ``uri`` parameter. Totem will take care of load balancing
automatically.
