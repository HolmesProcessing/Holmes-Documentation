**********************************
Trouble Shooting
**********************************

.. _github_documentation: https://github.com/HolmesProcessing/Holmes-Documentation
.. _github_totem:         https://github.com/HolmesProcessing/Holmes-Totem
.. _github_storage:       https://github.com/HolmesProcessing/Holmes-Storage
.. _github_toolbox:       https://github.com/HolmesProcessing/Holmes-Toolbox

If your error is not listed here and there is no issue on github, yet, please
file a new one on the Toolbox repository and describe your
problem as well as a way to reproduce it.
(Repositories can be found here:
`Documentation <github_documentation>`_, `Totem <github_totem>`_,
`Storage <github_storage>`_, and `Toolbox <github_toolbox>`_)


-  **The Cassandra database setup failed due to a connection timeout**

    Holmes-Storage uses a third party library to connect to Cassandra which sets
    the default connection timeout to 600ms. This is a pretty low value, which
    can easily cause problems when creating new tables (slow machines
    may need several seconds to complete such operation).

    To work around this issue, either modify the Holmes-Storage source code
    yourself, or use the version provided at
    ``https://github.com/ms-xy/Holmes-Storage.git``.
    It sets the timeout to 10s.

    However, make sure that you delete any table that has been created before
    you rerun the installer.
    (Even if the connection times out, Cassandra still creates the respective
    table)

    .. code-block:: shell

        ./universal-installer.sh --resume --storage \
            repo:"https://github.com/ms-xy/Holmes-Storage.git"

    .. note::

        The ``--resume`` flag will order the install script to skip all steps
        up to the one that errored out.



-  **The services for Totem fail to start**

    | Make sure that your services are all configured and that the configuration has no errors.
    | By default all services only have a service.conf.example file or similar, which isn't read upon starting the service.
    | Also the example configuration file does not necessarily work out of the box.

    (Example: Yara service misses its Yara rules)
