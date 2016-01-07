Install Docker_
==================

.. _Docker: http://www.docker.com

.. note::

    Setting up docker is only necessary if you want to run
    Totem in Docker!

The easiest way to deploy a Totem instance or any of it's services is by using
Docker_.
In case of something not working, this documentation could be out of date. In
that case please refer to the Docker website.

Linux Kernels before 3.10 can not run Docker. Please verify by issuing
``uname -r`` in a terminal that your Kernel is suitable.

.. warning::
    
    Make sure to adjust the ubuntu-trusty/debian-wheezy entries in the code below.

**Ubuntu:**

.. code-block:: shell
    
    # add the gpg key for the apt repository 
    sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
    
    # purge any existing Docker apt entries in /etc/apt/sources.list.d and add new repository
    # adjust ubuntu-trusty to fit your Ubuntu version
    sudo echo “deb https://apt.dockerproject.org/repo ubuntu-trusty main” > /etc/apt/sources.list.d/docker.list
    
    # Update and purge old repository
    sudo apt-get update && sudo apt-get purge lxc-docker
    
    # Install Docker, this requires the linux image extra as well
    sudo apt-get install linux-image-extra-$(uname -r)
    sudo apt-get install docker-engine
    sudo service docker start
    
    # Verify that the installation was complete
    sudo docker run hello-world

**Debian:**

.. code-block:: shell
    
    # add the gpg key for the apt repository 
    sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
    
    # purge any existing Docker apt entries in /etc/apt/sources.list.d and add new repository
    # adjust ubuntu-trusty to fit your Ubuntu version
    sudo echo “deb https://apt.dockerproject.org/repo debian-wheezy main” > /etc/apt/sources.list.d/docker.list
    
    # Update and purge old repository
    sudo apt-get update && sudo apt-get purge lxc-docker* docker.io*
    
    # Install Docker, this requires the linux image extra as well
    sudo apt-get install linux-image-extra-$(uname -r)
    sudo apt-get install docker-engine
    sudo service docker start
    
    # Verify that the installation was complete
    sudo docker run hello-world