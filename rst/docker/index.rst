Docker
=======

Some of the key components can be easily installed within a Docker container.

For a guide of how to install Docker, please see the Manual Installation section!

Totem
------

Here is a Dockerfile for running your Totem in Docker.

We do not provide any images for Totem, as you would most likely configure it
different than we do.

.. note::
    
    You will want to fork your own copy of the repository in order to change the
    settings.
    
    Also note that Totems services are Docker containers on their own and
    should be ran outside of the Totem container.

.. code-block:: shell
    
    FROM nimmis/java:oracle-8-jdk

    # enable https for apt
    RUN apt-get update
    RUN apt-get install -y apt-transport-https

    # install scala sbt
    RUN echo "deb https://dl.bintray.com/sbt/debian /" | sudo tee /etc/apt/sources.list.d/sbt.list > /dev/null
    RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 2EE0EA64E40A89B84B2DF73499E82A75642AC823
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



Storage
--------

| We provide a Docker image for Holmes-Storage, that you can find at ``https://hub.docker.com/r/msxy/holmes-storage/``.

This image is an **unconfigured** Holmes-Storage.

Before using the image, you have to first create a new Holmes-Storage
configuration file. See the Holmes-Storage section within the manual
installation section.

After you created a configuration file, move it to an empty folder and create
a Dockerfile there:

.. code-block:: shell
    
    FROM msxy/holmes-storage:latest
    ADD config.json /data/holmes-storage/
    EXPOSE 8016

The Dockerfile needs to expose the port configured in the configuration.
Further the configuration should bind to the IP address of ``0.0.0.0`` not
localhost.

Now build the image:

.. code-block:: shell
    
    docker build -t holmes-storage .

After you have done this, you can start a Holmes-Storage instance by issuing:

.. code-block:: shell
    
    docker run holmes-storage

If you want to use LocalFS as the object storage, it is highly recommend to
mount part of your local filesystem into the docker container to avoid data
inconsistency between your database and your object storage:

.. code-block:: shell

    docker run -v "/host/path/:/data/holmes-storage/objstorage-local-fs:rw" holmes-storage

If any of the other components is running on your host machine bound to localhost
or 127.0.0.1 (for example Cassandra) it is impossible to access them from inside
the container whilst it is in bridge mode.
In this case you need to share the hosts network stack with your container using
the ``--net=host`` option:

.. code-block:: shell
    
    docker run --net=host holmes-storage


RabbitMQ
---------

.. _hub_docker_com_rabbitmq: https://hub.docker.com/_/rabbitmq/

| For details see the `RabbitMQ image <hub_docker_com_rabbitmq_>`_ on *hub.docker.com*.

To start and use it, issue the following command on your Docker host:

.. code-block:: shell

    docker run -d --hostname my-rabbit --name some-rabbit rabbitmq:latest



Apache Cassandra
-----------------

.. _hub_docker_com_cassandr: https://hub.docker.com/_/cassandra/

For details see the `Apache Cassandra image <hub_docker_com_rabbitmq_>`_ on *hub.docker.com*.

- Running Cassandra on a single host:
    
    .. code-block:: shell
        
        # start a new node
        docker run --name cassandra1 -d cassandra:3.5
        
        # connect another node to the newly created cluster
        docker run --name cassandra2 -d --link cassandra1:cassandra cassandra:3.5

- Running Cassandra on multiple hosts:
    
    .. code-block:: shell
        
        # start a new node (substitute 10.42.42.42 by your servers IP)
        docker run --name cassandra1 -d -e CASSANDRA_BROADCAST_ADDRESS=10.42.42.42 -p 7000:7000 cassandra:3.5
        
        # connect another node to the newly created cluster (substitute 10.43.43.43 by your servers IP)
        docker run --name cassandra2 -d -e CASSANDRA_BROADCAST_ADDRESS=10.43.43.43 -p 7000:7000 -e CASSANDRA_SEEDS=10.42.42.42 cassandra:3.5
