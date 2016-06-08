
.. _Holmes-Toolbox: https://github.com/HolmesProcessing/Holmes-Toolbox

******************************************
Installation Using The Installer Script
******************************************

    A simple mostly unconfigured bare installation of Holmes-Totem and/or
    Holmes-Storage can be achieved by executing the `universal-installer.sh`.
    
    To get access to the installer script, clone the repository found at
    `Holmes-Toolbox`_.
    
    .. code-block:: shell
        
        git clone https://github.com/HolmesProcessing/Holmes-Toolbox

    
    **A fully functional system needs several components:**
    
    * Cassandra / MongoDB
    * RiakCS / Amazon S3 / LocalFS (Component of Holmes-Storage)
    * RabbitMQ
    * Holmes-Storage
    * Holmes-Totem
    
    Of these components the following can be installed with the installer
    script:
    
    * Cassandra
    * RabbitMQ
    * Storage (optionally configured for LocalFS)
    * Holmes-Totem
    

.. toctree::
    
    Installer Documentation <installer-manual-page.rst>
    Example: Local <example-local-installation.rst>
    Example: Cluster <example-cluster-installation.rst>
    Trouble Shooting <trouble-shooting.rst>
    
