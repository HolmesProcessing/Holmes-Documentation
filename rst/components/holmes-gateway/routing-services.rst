Routing Services through Different queues
**********************************************

By modifying gateway's config-file, it is possible to push different services into different AMQP-queues / exchanges. This way, it is possible to route some services to Holmes-Totem-Dynamic. The keys AMQPDefault and AMQPSplitting are used for this purpose. AMQPSplitting consists of a dictionary mapping Service names to Queues, Exchanges, and RoutingKeys. If the service is not found in this dictionary, the values from AMQPDefault are taken. e.g.

.. code-block:: json

	"AMQPDefault":   {"Queue": "totem_input", "Exchange": "totem", "RoutingKey": "work.static.totem"},
	"AMQPSplitting": {"CUCKOO":     {"Queue": "totem_dynamic_input", "Exchange": "totem_dynamic", "RoutingKey": "work.static.totem"},
	                  "DRAKVUF":    {"Queue": "totem_dynamic_input", "Exchange": "totem_dynamic", "RoutingKey": "work.static.totem"},
	                  "VIRUSTOTAL": {"Queue": "totem_dynamic_input", "Exchange": "totem_dynamic", "RoutingKey": "work.static.totem"}}


This configuration will route services CUCKOO and DRAKVUF to the queue "totem_dynamic_input", while every other service is routed to "totem_input".
