Install Holmes-Processing
==========================

.. _Totem: https://github.com/HolmesProcessing/Holmes-Totem
.. _Storage: https://github.com/HolmesProcessing/Holmes-Storage
.. _Cassandra: http://cassandra.apache.org/
.. _RiakCS: http://docs.basho.com/riak/cs/2.1.1/
.. _RabbitMQ: http://www.rabbitmq.com/

The sections below details the setup of all the possible components.
Whilst Java 8 is not really a component, it is a requirement for 2 other
component and as such has its own section.

(Cassandra requires Java 7 or 8, Holmes-Totem requires Java 8)

A complete system consists of:

- `Holmes-Totem <Totem_>`_
- `Holmes-Storage <Storage_>`_
- `Apache Cassandra <Cassandra_>`_
- `Riak CS <RiakCS_>`_
- `RabbitMQ <RabbitMQ_>`_



`RabbitMQ <RabbitMQ_>`_
-------------------------

Totem requires a AMPQ message broker, with the default one being RabbitMQ_.
Each Holmes-Totem instance queries this message broker for tasks and pushes results
back to it.
Holmes-Storage queries the queue and stores the results in the database.

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



Java 8
-------

.. _webupd8team: http://www.webupd8.org/2012/06/how-to-install-oracle-java-7-in-debian.html

Java is a requirement for Cassandra as well as for Holmes-Totem.
Holmes-Totem currently only works with Java 8 due to limitations of the
frameworks that are used.

The easiest way to install it is to use the webupd8team_ repository:

.. code-block:: shell
    
    sudo apt-get update
    sudo apt-get install apt-transport-https
    
    ## Ubuntu before 12.10:
    # sudo apt-get install python-software-properties
    #
    ## All others:
    sudo apt-get install software-properties-common
    
    sudo add-apt-repository -y ppa:webupd8team/java
    sudo apt-get update
    
    sudo apt-get install -y oracle-java8-installer
    sudo update-alternatives --config javac



`Apache Cassandra <Cassandra_>`_
---------------------------------

Cassandra is the default database for Holmes-Totem.
It should be set up with 3 nodes at least for redundancy and load balancing.

Cassandra requires Java 7 or 8 (how to install Java 8 see above).

.. code-block:: shell
    
    sudo apt-get update
    sudo apt-get install python python3 libjna-java curl
    
    ## Ubuntu 16.04 or greater requires installing python-support:
    # sudo curl -o /tmp/python-support_1.0.15_all.deb http://launchpadlibrarian.net/109052632/python-support_1.0.15_all.deb
    # sudo dpkg -i /tmp/python-support_1.0.15_all.deb
    # sudo rm /tmp/python-support_1.0.15_all.deb
    
    echo "deb http://debian.datastax.com/community stable main" | sudo tee /etc/apt/sources.list.d/cassandra.sources.list
    curl -L http://debian.datastax.com/debian/repo_key | sudo apt-key add -
    
    sudo apt-get update
    sudo apt-get install -y cassandra=3.0.5



`Riak CS <RiakCS_>`_ / S3 Object Storage
-----------------------------------------

RiakCS is the default object storage database for Holmes-Totem.
Its installation and configuration is complex. Please refer to the RiakCS
website for help with installing it.

Alternatively you can use Amazon S3 for object storage.



`Holmes-Storage <Storage_>`_
-----------------------------

This is the component responsible for storing and retrieving objects and results.
It is a wrapper around the database and object storage.

Before installing Holmes-Storage, you need to install
`The Go Programming Language <go_website_>`_.

.. _go_website: https://golang.org/

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

The next step is to download and build Holmes-Storage, which ``go`` happily does
for you (note that there is no ``https://`` or ``git://`` in front of your URL):

.. code-block:: shell
    
    go get -v -x -u "github.com/HolmesProcessing/Holmes-Storage"

After go finishes downloading and compiling Holmes-Storage, the executable can
be found in ``$HOME/go/bin/Holmes-Storage``.

| Create a dedicated installation folder in ``/data/holmes-storage``.
| Copy the executable there.
Download the configuration example from the root of the Holmes-Storage
repository (`here <https://github.com/HolmesProcessing/Holmes-Storage>`_) and
adjust it to your needs.



`Holmes-Totem <Totem_>`_
-------------------------

Docker
^^^^^^^

If you want to run Holmes-Totem's services, you need to install Docker. Please
refer to the Docker section.

Then after the installation of Holmes-Totem, you can simply run
``docker-compose up -d .`` in the ``config`` directory of your Holmes-Totem
installation.

Scala Build Tool
^^^^^^^^^^^^^^^^^

In order to compile Holmes-Totem you have to install ``sbt`` (Scala Build Tool).
See the official website_ for more info.

.. _website: http://www.scala-sbt.org/download.html

.. code-block:: shell
    
    echo "deb https://dl.bintray.com/sbt/debian /" | sudo tee /etc/apt/sources.list.d/sbt.list > /dev/null
    sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 2EE0EA64E40A89B84B2DF73499E82A75642AC823
    sudo apt-get update
    sudo apt-get install -y sbt

Clone and build Holmes-Totem
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

After installing ``sbt`` clone Holmes-Totem from the GitHub repository and
build it from source.

.. code-block:: shell
    
    git clone https://github.com/HolmesProcessing/Holmes-Totem.git
    cd Holmes-Totem
    sbt assembly

Which will produce a jar-file, which in turn you can start by issuing:

.. code-block:: shell
    
    java -jar target/scala-2.11/totem-assembly-1.0.jar config/totem.conf

However, you'll need to configure it first. (See the Holmes-Totem section of
this documentation)



Init Scripts
-------------

.. _toolbox_initscripts: https://github.com/HolmesProcessing/Holmes-Toolbox/tree/master/start-scripts

If you want your Holmes-Totem / Holmes-Storage to start on system start, you
need to install service or unit files (depending on which init system you have).

Example files can be found in the
`Holmes-Toolbox <toolbox_initscripts_>`_ .

| Systemd unit files go into ``/etc/systemd/system/``.
Upstart configuration files go into ``/etc/init/``.
