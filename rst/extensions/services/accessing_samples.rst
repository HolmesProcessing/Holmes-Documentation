Accessing Samples
************************

If Totem and the service run on the same host, it is sufficient for the service
to be able to access Totems sample storage location. (by default this is /tmp)

When running the service in Docker, the ``-v`` command line option does the
trick to achieve that:

.. code-block:: shell

    docker run -v /tmp:/tmp:ro imagename

This maps the hosts /tmp folder read-only to the Docker container providing the
service.

**However** if the service can't access the sample directory, there needs to be
a fileserver installed with Totem allowing the service access to the
samples stored by Totem. Any fileserver should suffice.