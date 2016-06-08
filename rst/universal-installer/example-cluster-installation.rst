**********************************
Totem + Storage + Database Cluster
**********************************

Any decent installation should consist of Holmes-Totem and Holmes-Storage
backed up by all other required components installed externally.
In other words, you want to install RabbitMQ, Cassandra/MongoDB, and RiakCS
on dedicated servers.

This example assumes that you have already set up these boxes. For RabbitMQ
you can use the installer. However, you need at least 3 nodes for Cassandra and
4 nodes for RiakCS.
The configuration necessary to get such a cluster up and running cannot be done
by the install script.

In order to install Totem and Storage and then configure Storage for cluster mode use:

.. code-block:: shell
    
    universal-installer.sh \
        --storage \
        --totem \
            repo:"some://custom.repo.url"

The ``--storage`` flag tells the installer to run the Holmes-Storage installer.
Since ``create-config:cluster`` is the default mode, you can omit this option.

Please note that Holmes-Totem **requires** custom configuration files.
Fork the Holmes-Totem GitHub repository (can be found
`here <holmes_totem_repository_>`_) and create the necessary configuration files
/ edit existing configuration files.
(A very basic working configuration can be found
`here <holmes_totem_example_config_>`_)

Details about the configuration can be found in the
Holmes-Totem Configuration section of this documentation.

For trouble shooting, please refer to the trouble shooting section.

.. _holmes_totem_repository: https://github.com/HolmesProcessing/Holmes-Totem.git
.. _holmes_totem_example_config: https://github.com/ms-xy/Holmes-Totem.git
