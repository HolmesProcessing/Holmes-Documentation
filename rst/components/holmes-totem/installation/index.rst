Installation
*******************

Dependencies
################

In order to compile Holmes-Totem you have to install ``sbt`` (Scala Build Tool).
See the official `website <sbt_>`_ for more info.

.. code-block:: shell

    echo "deb https://dl.bintray.com/sbt/debian /" | sudo tee /etc/apt/sources.list.d/sbt.list > /dev/null
    sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 2EE0EA64E40A89B84B2DF73499E82A75642AC823
    sudo apt-get update
    sudo apt-get install -y sbt


Configuration
################


There's two files of interest here:

- ``config/docker-compose.yml``
- ``config/totem.conf``

The docker-compose config file is responsible for service launch configuration.
Each entry in the ``services:`` section is of the form:

.. code-block:: yaml

  <name>:
    build:
      context: ../src/main/scala/org/holmesprocessing/totem/services/<name>
      args:
        conf: ${CONFSTORAGE_<name_uppercase>}service.conf
    ports:
      - "<port>:8080"
    restart: unless-stopped
    volumes:
      - /tmp:/tmp:ro

+-------------------+---------------------------------------------------------------------------------+
| <name>            | The name of the service, all lowercase usually and the same as the folder name. |
+-------------------+---------------------------------------------------------------------------------+
| <name_uppercase>  | The same as the <name>, but uppercase.                                          |
+-------------------+---------------------------------------------------------------------------------+
| <port>            | The service port. Convention is 7700,7710,7720, ... for the services processing |
|                   | files and 9700,9710,9720, ... for the services that do not.                     |
|                   |                                                                                 |
|                   | The simple reason for this is, that scaling the service deployment using        |
|                   | a solution like docker-compose requires some free ports.                        |
|                   |                                                                                 |
+-------------------+---------------------------------------------------------------------------------+

The totem config file mainly tells Totem what services are available and where
to find them. But you can also configure its request timeouts here.

.. code-block:: json

  download_settings {
    connection_pooling = true
    connection_timeout = 1000
    download_directory = "/tmp/"
    thread_multiplier = 4
    request_timeout = 1000
  }

  tasking_settings {
    default_service_timeout = 180
    prefetch = 3
    retry_attempts = 3
  }

  rabbit_settings {
    requeueKey = "requeue.static.totem"
    host {
      server = "127.0.0.1"
      port = 5672
      username = "guest"
      password = "guest"
      vhost = "/"
    }

    ...
  }

  services {
    asnmeta {
      uri = ["http://127.0.0.1:9700/analyze/?obj="]
      resultRoutingKey = "asnmeta.result.static.totem"
    }

    ...
  }

The above is an excerpt of the default configuration file.
It is important to note that the ``connection_timeout`` and ``request_timeout``
are (counter-intuitively) not just associated with downloading samples. They
apply to "downloading" results from services as well. If you
experience a lot of service failures due to timeouts consider increasing these
values. Additionally the ``tasking_settings.default_service_timeout`` may need
changing, too. (The former two are given as milliseconds, the
latter as seconds)

Most settings regarding RabbitMQ are of no interest to a regular user. The only
things that need to be adjusted are the credentials and the address.

More interesting are the service entries. They offer the ability to configure
multiple URIs for each service for automatic load balancing.
The schema for the URIs and the routing key is always the same though.

.. code-block:: json

  "http://<address>/analyze/?obj="

.. code-block:: json

  "<servicename>.result.stastic.totem"

The suffix ``result.static.totem`` always corresponds to the suffix defined in
the RabbitMQ settings.


Running
##########

After installing ``sbt`` clone Holmes-Totem from the GitHub repository and
build it from source.

.. code-block:: shell

    mkdir -p /data/holmes-totem
    cd /data/holmes-totem
    git clone https://github.com/HolmesProcessing/Holmes-Totem.git .
    sbt assembly

This will produce a jar-file. To start Totem, issue:

.. code-block:: shell

    java -jar target/scala-2.11/totem-assembly-1.0.jar config/totem.conf

However, you'll need to configure it first. (See the configuration documentation)
