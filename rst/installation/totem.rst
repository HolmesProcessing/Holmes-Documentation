Install Totem_
===============

.. _Totem: https://github.com/HolmesProcessing/Holmes-Totem
.. _RabbitMQ: http://www.rabbitmq.com/

.. note::
    
    For installing and running Totem in Docker, skip to the Dockerfile below!



RabbitMQ_
-----------

Totem requires a AMPQ message broker, with the default one being RabbitMQ_.
The message brokers duty is to equally distribute tasks between Totem instances
and to retrieve results.

To install RabbitMQ with base settings:

.. code-block:: shell
    
    deb http://www.rabbitmq.com/debian/ testing main
    wget https://www.rabbitmq.com/rabbitmq-signing-key-public.asc
    sudo apt-key add rabbitmq-signing-key-public.asc
    
    sudo apt-get update
    sudo apt-get install rabbitmq-server
    
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

Totem currently only works with Java 8 due to limitations of the used frameworks.

The easiest way to get it is to use the webupd8team_ repository:

.. code-block:: shell
    
    sudo add-apt-repository ppa:webupd8team/java
    sudo apt-get update
    sudo apt-get install oracle-java8-installer



Scala Build Tool
------------------

To compile Totem ``sbt`` (Scala Build Tool) is required. See the official
website_ for more info.

.. _website: http://www.scala-sbt.org/download.html

.. code-block:: shell
    
    echo "deb https://dl.bintray.com/sbt/debian /" | sudo tee -a /etc/apt/sources.list.d/sbt.list
    sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 642AC823
    sudo apt-get update
    sudo apt-get install sbt



Build
---------

After satisfying all pre-requisites, the only thing left to do is building Totem
with sbt.

.. code-block:: shell
    
    git clone https://github.com/HolmesProcessing/Holmes-Totem.git holmes-totem
    cd holmes-totem
    sbt assembly

Which will produce a jar-file, which in turn you can start by issuing:

.. code-block:: shell
    
    java -jar target/scala-2.11/totem-assembly-1.0.jar config/totem.conf



Docker (Totem)
----------------

Here is a Dockerfile for running your Totem in Docker.

.. note::
    
    You will want to fork your own copy of the repository in order to change the
    settings.

.. code-block:: shell
    
    FROM java:oracle-java8

    # enable https for apt
    RUN apt-get update && apt-get install -y apt-transport-https

    # install scala sbt
    RUN echo "deb https://dl.bintray.com/sbt/debian /" | tee -a /etc/apt/sources.list.d/sbt.list
    RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 642AC823
    RUN apt-get update
    RUN apt-get install -y sbt

    # setup Holmes-Totem
    RUN apt-get install -y git
    RUN mkdir /data
    WORKDIR /data
    RUN git clone https://github.com/HolmesProcessing/Holmes-Totem
    WORKDIR /data/Holmes-Totem
    RUN sbt assembly

    # start totem
    CMD java -jar /data/Holmes-Totem/target/scala-2.11/totem-assembly-1.0.jar /data/Holmes-Totem/config/totem.conf



Docker (RabbitMQ)
-------------------

Here is a Dockerfile for running a RabbitMQ server.


.. code-block:: shell

    FROM java:oracle-java8

    # enable https for apt
    RUN apt-get update && apt-get install -y apt-transport-https

    # install rabbitmq
    RUN echo "deb http://www.rabbitmq.com/debian/ testing main" | tee -a /etc/apt/sources.list.d/rabbitmq.list
    RUN wget https://www.rabbitmq.com/rabbitmq-signing-key-public.asc
    RUN apt-key add rabbitmq-signing-key-public.asc
    RUN apt-get update && apt-get install -y rabbitmq-server

    # It's advides to run your stuff using supervisor
    # else you have to chain it to CMD below

    # start totem
    CMD service rabbitmq-server start && java -jar /data/Holmes-Totem/target/scala-2.11/totem-assembly-1.0.jar /data/Holmes-Totem/config/totem.conf



Docker (Totem + RabbitMQ)
--------------------------

Here is a Dockerfile for running your Totem in Docker together with RabbitMQ.

.. note::
    
    You will want to fork your own copy of the Totem repository in order to
    change the settings.

.. code-block:: shell

    FROM java:oracle-java8

    # enable https for apt
    RUN apt-get update && apt-get install -y apt-transport-https

    # install rabbitmq
    RUN echo "deb http://www.rabbitmq.com/debian/ testing main" | tee -a /etc/apt/sources.list.d/rabbitmq.list
    RUN wget https://www.rabbitmq.com/rabbitmq-signing-key-public.asc
    RUN apt-key add rabbitmq-signing-key-public.asc
    RUN apt-get update && apt-get install -y rabbitmq-server

    # install scala sbt
    RUN echo "deb https://dl.bintray.com/sbt/debian /" | tee -a /etc/apt/sources.list.d/sbt.list
    RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 642AC823
    RUN apt-get update
    RUN apt-get install -y sbt

    # setup Holmes-Totem
    RUN apt-get install -y git
    RUN mkdir /data
    WORKDIR /data

    # you might want to clone your fork here
    # (which should be configured to run your service)
    RUN git clone https://github.com/HolmesProcessing/Holmes-Totem
    WORKDIR /data/Holmes-Totem
    RUN sbt assembly

    # add your service related stuff here
    # you can also go the ADD . /data/yourservice route
    WORKDIR /data
    RUN git clone https://github.com/you/yourservice

    # build and run yourservice
    WORKDIR /data/yourservice
    RUN make
    # It's advides to run your stuff using supervisor
    # else you have to chain it to CMD below

    # start totem
    CMD service rabbitmq-server start && java -jar /data/Holmes-Totem/target/scala-2.11/totem-assembly-1.0.jar /data/Holmes-Totem/config/totem.conf


bootstrap.sh
------------

.. _here: https://github.com/webstergd/Holmes-Totem/blob/bootstrap/bootstrap.sh

Can also be found here_.

.. code-block:: shell

    # General Requirements
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get install build-essential python-dev python-pip git


    # Install Docker
    # if 14.04
    sudo apt-get install linux-image-extra-$(uname -r)
    sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
    echo "deb https://apt.dockerproject.org/repo ubuntu-trusty main" | sudo tee -a /etc/apt/sources.list.d/docker.list
    sudo apt-get update
    sudo apt-get purge lxc-docker
    sudo apt-cache policy docker-engine
    sudo apt-get install docker-engine

    # Update Docker User
    sudo usermod -aG docker totem

    # Install Docker Compose
    sudo pip install docker-compose


    # Install Java 
    sudo apt-get install python-software-properties
    sudo add-apt-repository ppa:webupd8team/java
    sudo apt-get update
    sudo apt-get install oracle-java8-installer


    # SBT
    echo "deb https://dl.bintray.com/sbt/debian /" | sudo tee -a /etc/apt/sources.list.d/sbt.list
    sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 642AC823
    sudo apt-get update
    sudo apt-get install sbt

    # Install Totem
    sudo mkdir /data
    cd /data
    sudo git clone https://github.com/HolmesProcessing/Holmes-Totem.git
    sudo chown -R totem:totem Holmes-Totem

    # Built Totem
    sbt assembly

    # Start Docker containers (e.g. services)
    cd /data/Holmes-Totem/config
    docker-compose up -d

    # Start Totem
    cd /data/Holmes-Totem
    java -jar 
