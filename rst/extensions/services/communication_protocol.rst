Communication Protocol
************************

Totem communicates with its services via the http protocol. The request for the
service result is of type GET with an empty body.

.. code-block:: html

    http://address:port/servicename/sampleid

The result returned by the service is an appropriate status code, paired with a
valid JSON reply body on success or a http error message reflecting the error
that occured. (plus optionally a JSON reply body)

.. code-block:: json

    {
      "key": "value",
      "key": {
        "subkey": "value",
        "subkey": "value"
      }
    }

This means that your service must act like a webserver. If the service isn't
written in a language that offers simple webserver capabilities (like golang),
then using a wsgi compatible python wrapper is adviced, like `Tornado`_.

.. _Tornado: http://www.tornadoweb.org/en/stable/