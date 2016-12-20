*******************
Local Installation
*******************

| A full local installation needs to serve all 5 components.

However, it is not really useful to install a RiakCS object storage on a single
node, thus it is sufficient to only install the following 4 components:

    * Cassandra
    * RabbitMQ
    * Holmes-Storage (configured for LocalFS)
    * Holmes-Totem

The following command automatically installs all the mentioned components and
guides the user through a basic configuration for Holmes-Storage:

.. code-block:: shell

    universal-installer.sh \
        --cassandra \
        --rabbitqm \
        --storage \
            config-create:local \
        --totem \
            repo:"some://custom.repo.url"


The ``--cassandra`` and ``--rabbitmq`` flags should be self-explanatory.

The ``--storage`` flag tells the installer to run the Holmes-Storage installer
and orders it to create a config file for a local installation.
Whilst this should work without a problem on any decent server, it may fail during
the database setup routine if Cassandra is a bit slow (default timeout is 600ms
per request).
If you run into this issue, you have two options:

- Try using a version of Holmes-Storage modified specifically to address this issue.
- Tune the Cassandra installation to serve the requests faster.

For the first option there exists a repository that addresses users that face
this issue.
Instead of ``--storage``, use:
``--storage repo:"https://github.com/ms-xy/Holmes-Storage.git"``.

If you opt to do the manual tuning, the bottleneck during the setup might be the
slow table creation commands.

Please note that Holmes-Totem **requires** custom configuration files.
Fork the Holmes-Totem GitHub repository (can be found `here <here_>`_) and create the
necessary configuration files / edit existing configuration files.
(A very basic working configuration can be found `here <example_config_>`_)

Details about the configuration can be found in the
Holmes-Totem Configuration section of this documentation.

For trouble shooting, please refer to the trouble shooting section.

.. _here: https://github.com/HolmesProcessing/Holmes-Totem.git
.. _example_config: https://github.com/ms-xy/Holmes-Totem.git
