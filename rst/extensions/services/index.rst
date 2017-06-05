===================
Services for Totem
===================
.. _CRITs: https://crits.github.io
.. _Docker: http://www.docker.com

.. toctree::
    writing_a_service/index
    example_service

Totem's Services are pretty simple to understand. They build upon JSON as a
messaging format, use a RESTful API and are completely independent of Totem itself.
The Scala you need to add one is, in most cases, as simple as it can be:
copy and paste.

Any programming language could be used to implement your service. This tutorial uses Python because it's simple and the services for the popular analysis platform CRITs_ are written in it.

Also any kind of virtualization method can be used to restrict the environment of
your Service. This tutorial shows how to use Docker_.

This chapter contains detailed information about how to write a new Service and a Service example.
