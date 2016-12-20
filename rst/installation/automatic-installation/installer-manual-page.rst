************************************
Holmes-Toolbox Universal Installer
************************************

::::::::::::::::::::::::::::::::::::
Usage
::::::::::::::::::::::::::::::::::::

.. code-block:: shell

    ./universal-installer.sh
        [--rabbitmq]
        [--cassandra]
        [--storage [path:...] [repo:...] [[config-url:...]|[config-create:...]] [no-initscript] [erase] [skip-setup]]
        [--totem [path:...] [repo:...] [branch:...] [no-services] [no-initscript] [erase]]
        [--resume]


::::::::::::::::::::::::::::::::::::
Log-Output
::::::::::::::::::::::::::::::::::::

The Universal Installer logs all output to a file called ``universal-installer.log``
which is located in the same directory as the installer script.


::::::::::::::::::::::::::::::::::::
Option-Details
::::::::::::::::::::::::::::::::::::

--rabbitmq          Install RabbitMQ (Task scheduler for Holmes-Totem)
--cassandra         Install Apache-Cassandra (Database)


--storage           Install Holmes-Storage (Storage backend for Holmes-Totem)

  Available options for Holmes-Storage:

  +------------------------------+------------------------------------------------------------------------------+
  | path:/install/path           | | Path to install Holmes-Storage in.                                         |
  |                              | | *(default: /data/holmes-storage)*                                          |
  +------------------------------+------------------------------------------------------------------------------+
  | repo:http://repo.url         | | Repository that Holmes-Storage is pulled from, without protocoll and       |
  |                              | | without ``.git`` extension.                                                |
  |                              | | *(default: github.com/HolmesProcessing/Holmes-Storage)*                    |
  +------------------------------+------------------------------------------------------------------------------+
  | config-url:http://config.url | | Specify the location of the configuration file, will be loaded with curl.  |
  |                              | | *(incompatible with config-create)*                                        |
  +------------------------------+------------------------------------------------------------------------------+
  | config-create:TYPE           | | Specify the configuration to be created.                                   |
  |                              | | *(semi-automatic, incompatible with config-url option)*                    |
  +------------------------------+------------------------------------------------------------------------------+
  | no-initscript                | Do not install init scripts for upstart/systemd.                             |
  +------------------------------+------------------------------------------------------------------------------+
  | erase                        | | Purge any previous installation in the specified installation location.    |
  |                              | | **This does NOT purge existing database entries.**                         |
  +------------------------------+------------------------------------------------------------------------------+
  | skip-setup                   | Do not run database setup routines.                                          |
  +------------------------------+------------------------------------------------------------------------------+

  Available config-create ``TYPE's`` are:

  +------------------------------+------------------------------------------------------------------------------+
  | local                        | Install Cassandra, configure Holmes-Storage to use Cassandra and LocalFS.    |
  +------------------------------+------------------------------------------------------------------------------+
  | local-objstorage             | Configure Holmes-Storage to use Cassandra and LocalFS.                       |
  +------------------------------+------------------------------------------------------------------------------+
  | local-cassandra              | Install Cassandra, configure Holmes-Storage to use Cassandra and S3.         |
  +------------------------------+------------------------------------------------------------------------------+
  | cluster                      | Configure Holmes-Storage to use Cassandra and S3.                            |
  +------------------------------+------------------------------------------------------------------------------+
  | local-mongodb                | Install MongoDB, configure Holmes-Storage to use MongoDB.                    |
  +------------------------------+------------------------------------------------------------------------------+
  | cluster-mongodb              | Configure Holmes-Storage to use MongoDB.                                     |
  +------------------------------+------------------------------------------------------------------------------+

  Default ``TYPE`` is cluster.


--totem          Install Holmes-Totem

  Available options for Holmes-Totem:

  +------------------------------+------------------------------------------------------------------------------+
  | path:/install/path           | | Path to install Holmes-Totem to.                                           |
  |                              | | *(default: /data/holmes-totem)*                                            |
  +------------------------------+------------------------------------------------------------------------------+
  | repo:http://repo.url         | | Repository that Holmes-Totem is pulled from.                               |
  |                              | | *(default: https://github.com/HolmesProcessing/Holmes-Totem.git)*          |
  +------------------------------+------------------------------------------------------------------------------+
  | branch:repository-branch     | | Branch to checkout after cloning the repository.                           |
  |                              | | (If not supplied, no checkout)                                             |
  +------------------------------+------------------------------------------------------------------------------+
  | no-services                  | | Do not install Totems default services.                                    |
  |                              | | **Implied by ``no-initscript``**                                           |
  +------------------------------+------------------------------------------------------------------------------+
  | no-initscript                | Do not install init scripts for upstart/systemd                              |
  +------------------------------+------------------------------------------------------------------------------+
  | erase                        | Purge any previous installation in the specified installation location.      |
  +------------------------------+------------------------------------------------------------------------------+


--resume         Attempt to resume at the last known checkpoint.

    **Use with utmost care**

    | This option should only be used with the exact same parameters as before.

    Otherwise proper functionality cannot be guaranteed.

    **Further, installing Holmes-Storage may fail repeatedly if it fails at**
    **the database setup. See the trouble shooting section for info on how**
    **to deal with this.**



::::::::::::::::::::::::::::::::::::
Important Notes
::::::::::::::::::::::::::::::::::::


    * **Unique Paths**

        If you install both, Holmes-Totem and Holmes-Storage, the given installation
        paths must be unique for each of them.


    * **Totem requires a custom configuration**

        Please fork the original repository and create/adjust configuration as
        necessary.

        A repository with a working config is available at:

        .. code-block:: text

            https://github.com/ms-xy/Holmes-Totem.git


::::::::::::::::::::::::::::::::::::
Trouble Shooting
::::::::::::::::::::::::::::::::::::


    * **If Storage fails to start / setup due to timeouts connecting to Cassandra**

        In this case, try using the following repository during installation
        (``repo`` option for ``--storage``):

        .. code-block:: text

            https://github.com/ms-xy/Holmes-Storage.git

        This version of Holmes-Storage sets the request timeout to 10
        seconds (default 600ms). If this does not help, your chosen server most
        likely offers too little resources for a reasonable Cassandra
        installation and you should use a different one.

        .. note::

            You can use the ``--resume`` flag to skip all previous steps and
            retry the database setup directly.

        .. warning::

            Delete any tables that got created or the database setup will fail
            with the error that the tables already exist. (Cassandra creates
            the tables, even if the connection timed out)



::::::::::::::::::::::::::::::::::::
Examples
::::::::::::::::::::::::::::::::::::

  Here's a quick rundown of examples. For detailed information, please see the
  corresponding example section.

  * A complete setup (RabbitMQ + Cassandra + Storage + Totem) on one server:

    .. code-block:: shell

        ./universal-installer.sh \
            --rabbitmq \
            --cassandra \
            --storage \
                config-create:local \
            --totem \
                repo:"your-git-repo"

  * Only Storage, in cluster mode (configure Cassandra + S3 external):

    .. code-block:: shell

        ./universal-installer.sh --storage

  * Only Totem, without its services as init job, erase an old installation
    if there is one in the same installation directory:

    .. code-block:: shell

        ./universal-installer.sh \
            --totem \
                repo:"your-git-repo" \
                no-services \
                erase

  * Install Storage and Totem in custom directories:

    .. code-block:: shell

        ./universal-installer.sh \
            --totem \
                repo:"your-git-repo" \
                path:/custom/install/path/totem \
            --storage \
                path:/custom/install/path/storage

