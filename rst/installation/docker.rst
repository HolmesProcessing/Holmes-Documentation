Install Docker_
==================

.. _Docker: http://www.docker.com

Linux Kernels before 3.10 can not run Docker. Please verify by issuing
``uname -r`` in a terminal that your Kernel is suitable.

.. code-block:: shell
    
    sudo apt-get update
    sudo apt-get install curl
    
    # install docker
    curl -sSL https://get.docker.com/ | /bin/sh
    
    # install docker-compose
    sudo sh -c "curl -L https://github.com/docker/compose/releases/download/1.7.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose"
    sudo chmod +x /usr/local/bin/docker-compose

Done. Yes that's it, now you have docker compose and docker.

.. warning::
    
    Installing docker-compose like this will overwrite any existing docker-compose
    installation.
