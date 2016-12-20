====================================
Install Holmes' Components
====================================

.. links for main components
   -------------------------

.. _Totem: https://github.com/HolmesProcessing/Holmes-Totem
.. _Totem_Dynamic: https://github.com/HolmesProcessing/Holmes-Totem-Dynamic
.. _Storage: https://github.com/HolmesProcessing/Holmes-Storage
.. _Gateway: https://github.com/HolmesProcessing/Holmes-Gateway
.. _Frontend: https://github.com/HolmesProcessing/Holmes-Frontend
.. _Interrogation: https://github.com/HolmesProcessing/Holmes-Interrogation

.. _Cassandra: http://cassandra.apache.org/
.. _RabbitMQ: http://www.rabbitmq.com/
.. _RiakCS: http://docs.basho.com/riak/cs/2.1.1/

.. _Docker: http://www.docker.com


.. other links
   -----------

.. _webupd8team: http://www.webupd8.org/2012/06/how-to-install-oracle-java-7-in-debian.html
.. _go_website: https://golang.org/
.. _sbt: http://www.scala-sbt.org/download.html
.. _toolbox_initscripts: https://github.com/HolmesProcessing/Holmes-Toolbox/tree/master/start-scripts


.. begin intro
   -----------

The sections below detail the installation of all components. There are sections for
Docker and Oracle Java 8, too. (Totem's services are recommended to be installed
and run inside of Docker containers, further Totem and Cassandra require Java)

There is no section describing how to set up Amazon S3. The AWS documentation is
very extensive and is easily accessed in the AWS management console (Menu:
``support`` > ``Documentation``).

The required core components are:

- `Holmes-Totem <Totem_>`_
- `Holmes-Totem-Dynamic <Totem_Dynamic_>`_
- `Holmes-Storage <Storage_>`_
- `Holmes-Gateway <Gateway_>`_
- `Apache Cassandra <Cassandra_>`_
- `RabbitMQ <RabbitMQ_>`_
- *Amazon S3 API* compatible object storage
- Holmes-Totem's services

Recommended components:

- `Holmes-Frontend <Frontend_>`_ (web interface with lots of useful features)
- `Holmes-Interrogation <Interrogation_>`_ (required for Holmes-Frontend)

Optional Components

- `Riak CS <RiakCS_>`_ (alternative to Amazon S3)

Additional guides:

- Installing Java 8
- Installing `Go <go_website_>`_
- Installing `Docker <Docker_>`_
- Init scripts



.. required components section
   ---------------------------

|

----

`Holmes-Totem <Totem_>`_
*************************

In order to compile Holmes-Totem you have to install ``sbt`` (Scala Build Tool).
See the official `website <sbt_>`_ for more info.

.. code-block:: shell

    echo "deb https://dl.bintray.com/sbt/debian /" | sudo tee /etc/apt/sources.list.d/sbt.list > /dev/null
    sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 2EE0EA64E40A89B84B2DF73499E82A75642AC823
    sudo apt-get update
    sudo apt-get install -y sbt

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

|

----

`Holmes-Totem-Dynamic <Totem_Dynamic_>`_
*****************************************

Totem-Dynamic is the Totem responsible for dealing with long running services.

.. code-block:: shell

    mkdir -p /data/holmes-totem-dynamic
    cd /data/holmes-totem-dynamic
    git clone https://github.com/HolmesProcessing/Holmes-Totem-Dynamic.git .
    go get -v -x -u github.com/streadway/amqp
    go build .



|

----

`Holmes-Storage <Storage_>`_
*****************************

This is the component responsible for storing and retrieving objects and results.
It is a wrapper around the database and object storage.

.. code-block:: shell

    mkdir -p /data/holmes-storage
    cd /data/holmes-storage
    git clone https://github.com/HolmesProcessing/Holmes-Storage.git .
    git build .



|

----

`Holmes-Gateway <Gateway_>`_
*****************************

Holmes-Gateway is the endpoint that users interact with when creating tasks for
Holmes-Totem or Holmes-Totem-Dynamic.

.. code-block:: shell

    mkdir -p /data/holmes-gateway
    cd /data/holmes-gateway
    git clone https://github.com/HolmesProcessing/Holmes-Gateway.git .
    go build .

The framework requires one Holmes-Gateway running in Master mode and the Master
Gateway needs a SSL certificate to function. If you don't have a SSL certificate
at hand you can simply create a self signed one by using the provided shell
script:

.. code-block:: shell

    ./mkcert.sh



|

----

`Apache Cassandra <Cassandra_>`_
*********************************

Cassandra is the default database for Holmes-Totem.
A Cassandra deployment should consist of at least 3 nodes (see Cassandra
documentation).

Cassandra requires Java 7 or 8 (see Java 8 installation section).

If any of the below does not work (e.g. due to a key signature error), please
visit `cassandra.apache.org/download/ <http://cassandra.apache.org/download/>`_.
Our documentation might be outdated in that case.

If you want to use version pinning instead of installing the latest release,
make sure that the version you install is 3.5 or newer.

.. code-block:: shell

    sudo apt-get update
    sudo apt-get install python python3 libjna-java curl

    ## Ubuntu 16.04 or greater requires installing python-support:
    # sudo curl -o /tmp/python-support_1.0.15_all.deb http://launchpadlibrarian.net/109052632/python-support_1.0.15_all.deb
    # sudo dpkg -i /tmp/python-support_1.0.15_all.deb
    # sudo rm /tmp/python-support_1.0.15_all.deb

    echo "deb http://www.apache.org/dist/cassandra/debian 39x main" \
        | sudo tee -a /etc/apt/sources.list.d/cassandra.sources.list

    curl https://www.apache.org/dist/cassandra/KEYS \
        | sudo apt-key add -

    sudo apt-key adv --keyserver pool.sks-keyservers.net --recv-key A278B781FE4B2BDA

    sudo apt-get update
    sudo apt-get install -y cassandra



|

----

`RabbitMQ <RabbitMQ_>`_
************************

.. image:: amqp-broker.png
   :align: center

| An AMPQ message broker is required as a task and result transport. The default one is RabbitMQ_.

To install RabbitMQ with base settings:

.. code-block:: text

    sudo apt-get update
    sudo apt-get install apt-transport-https wget

    echo 'deb http://www.rabbitmq.com/debian/ testing main' | sudo tee /etc/apt/sources.list.d/rabbitmq.list
    wget -O- https://www.rabbitmq.com/rabbitmq-signing-key-public.asc | sudo apt-key add -

    sudo apt-get update
    sudo apt-get install rabbitmq-server

    sudo service rabbitmq-server start


In case SELinux is installed, it may prevent port binding. Make sure the
following ports are open for RabbitMQ to work properly:

.. code-block:: none

    4369 (epmd), 25672 (erlang-dist)
    5672, 5671 (AMQP 0-9-1 with and without TLS)
    15672 (management plugin)
    61613, 61614 (if STOMP is enabled)
    1883, 8883 (if MQTT is enabled)



|

----

Totem's Services
*****************

Holmes-Totem's services are designed to be used with Docker, a software
containerization plaftorm. (See the Docker installation guide for help
installing it)

There is no further installation required after installing Docker.
Container deployment and scaling can be managed by the tool ``docker-compose``,
whose installation is also explained in the Docker installation section.

Note that before building and running the service containers, you should consult
the configuration documentation. Copying and modification of service
configuration files as well as the docker-compose config is required.



.. recommended components
   -----------------------

|

----

`Holmes-Frontend <Frontend_>`_ (optional)
******************************************

.. note::

    For Holmes-Frontend to work, a running instance of Holmes-Interrogation
    is required!

Holmes-Frontend is basically just a Javascript driven web application that can
be run on any machine - even the operators - if it is capable of serving the
HTML files and associated assets (Javascript, CSS).

One option is to host it using an Apache2 webserver (or nginx or any webserver
for that matter). It is important to stress that the allow-origin directive has
to be set to ``*`` (the Frontend issues requests to Holmes-Interrogation, which
can run on any other address).

E.g. for the Apache2 this can be done by setting

.. code-block:: shell

    Header set Access-Control-Allow-Origin "*"

inside of a ``<Directory``, ``<Location>``, ``<Files>``, or ``<VirtualHost>``
configuration.

Another option is to use the simplistic file server provided with Holmes-Frontend.
You can find it in the ``server`` directory within the repository.
To build it, you need to install ``Go`` (see the respective section) and run
``go build`` in the directory.

The ``web`` directory in the repository holds all the files that need to be hosted.





|

----

`Holmes-Interrogation <Frontend_>`_ (optional)
***********************************************

Installation of Holmes-Interrogation is again analogue to the installation of
Holmes-Storage and Holmes-Gateway.

.. code-block:: shell

    go get -v -x -u "github.com/HolmesProcessing/Holmes-Interrogation"

Copy the executable to ``/data/holmes-interrogation``, as well as the configuration.
It is important to note that Holmes-Interrogation requires a SSL certificate as
well. You can re-use the ``mkcert.sh`` found in the Holmes-Storage repository
for this purpose.



.. optional components
   --------------------

|

----

`Riak CS <RiakCS_>`_ (optional)
********************************

RiakCS is the default object storage database for Holmes-Totem.
Its installation and configuration is complex. Please refer to the RiakCS
website for help with installing it.

Alternatively you can use Amazon S3 for object storage.



.. additional installation guides
   -------------------------------

|

----

Java 8
*******

Java is a requirement for Cassandra as well as for Holmes-Totem.
Whilst Cassandra works with either Java 7 or Java 8, Holmes Totem is limited to
Java 8 due to dependencies.

The easiest way to install it is to use the webupd8team_ repository:

.. code-block:: shell

    sudo apt-get update
    sudo apt-get install apt-transport-https

    ## Ubuntu before 12.10:
    # sudo apt-get install python-software-properties
    ## Instead of:
    sudo apt-get install software-properties-common

    sudo add-apt-repository -y ppa:webupd8team/java
    sudo apt-get update

    sudo apt-get install -y oracle-java8-installer
    sudo update-alternatives --config javac



|

----

`Go <go_website_>`_
********************

Right now Go is a requirement to use Holmes-Storage, as it needs to be built
from sources. In the future we may opt to instead publish pre-built binaries
for the most popular systems.

Before installing Holmes-Storage, you need to install
`The Go Programming Language <go_website_>`_.

.. code-block:: shell

    sudo apt-get update
    sudo apt-get install curl

    curl -o /tmp/go.tar.gz -L "https://storage.googleapis.com/golang/go1.6.1.linux-amd64.tar.gz"
    sudo tar -C /usr/local -xzf /tmp/go.tar.gz
    rm /tmp/go.tar.gz

    # The following environmental variables need to be set in order to run go:
    export GOPATH="$HOME/go"
    export PATH=$PATH:/usr/local/go/bin
    export GOROOT=/usr/local/go



|

----

Docker_
********

It is recommended to install Docker in addition to Holmes-Totem. Paired with
``docker-compose`` it allows for easy and flexible scaling and deployment of
Totem's services. (Dockerfiles for all standard services are provided, as well
as an example ``docker-compose.yaml``)

Note that Linux Kernels before 3.10 do not support Docker. Please verify that
your Kernel is suitable by running ``uname -r`` in a terminal and checking the
version number.

There are two ways of installing Docker. The first is to download the official
installation script from ``docker.com`` and execute it.
The other is to use the Docker version provided by your Linux Distribution's
repositories. Make sure that the provided version is not too old. (If that is
the case you can try to find an alternative repository)

To install Docker run:

.. code-block:: shell

    curl -sSL https://get.docker.com/ | /bin/sh

Installing docker-compose should be done using pip. However, you can also do the
following:

.. code-block:: shell

    sudo sh -c "curl -L https://github.com/docker/compose/releases/download/1.7.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose"
    sudo chmod +x /usr/local/bin/docker-compose

Done. Yes that's it, now you have docker compose and docker.

.. warning::

    Installing docker-compose like this will overwrite any existing
    docker-compose installation.



|

----

Init Scripts
*************

If you want your Holmes-Totem, Holmes-Storage, or Holmes-Gateway to start on
system start, you need to install service or unit files (depending on which
init system you have).

Example files can be found in the
`Holmes-Toolbox <toolbox_initscripts_>`_ .

| Systemd unit files go into ``/etc/systemd/system/``.

Upstart configuration files go into ``/etc/init/``.
