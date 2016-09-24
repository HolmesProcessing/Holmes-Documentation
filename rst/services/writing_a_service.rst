Writing a Service
=================

.. role:: red
    :class: font-color-red


Overview
--------

**For a service to be accepted into the main repository, the proper additions to
the following files need to be made:**

- Config Files
    - ``totem.conf``
    - ``docker-compose.yml.example``
    - ``compose_download_conf.sh``
- Scala Files
    - ``driver.scala``

**Additionally the following files need to be added:**

- Service Files
    - ``Dockerfile``
    - ``LICENSE``
    - ``README.md``
    - ``acl.conf``
    - ``service.conf.example``
    - ``watchdog.scala``
    - ``<SERVICE_NAME_CAMELCASE>REST.scala``
- Any additional files your service needs
    - Python e.g.: ``service.py``
    - Go e.g.: ``service.go``

.. warning::

    Try to stick to the naming convention (uppercase/lowercase/camelcase were
    appropriate) to avoid confusion!

    If you did not write a service yet, please consult the subcategories.


Details
--------

.. toctree::

    writing_a_service/files-to-edit
    writing_a_service/files-to-add
