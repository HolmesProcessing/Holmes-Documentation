Install Fileserver
===================

.. _Tornado: http://www.tornadoweb.org/en/stable/

In case you are hosting files on a different machine than just Totem - which is
not unlikely - you'll have to distribute your samples to your services somehow.

The easiest way to do that is by having a Fileserver running on each Totem host,
that allows accessing the samples stored in the /tmp directory of the host.

The simplest Fileserver possible is Tornado_.

.. code-block:: python
    
    import tornado.web
    
    settings = {
        "static_path": "/tmp"),
        "cookie_secret": "YOUR_SECRET_VALUE",
        "login_url": "/login",
        "xsrf_cookies": True,
    }
    
    application = tornado.web.Application([
        (r"/*", tornado.web.StaticFileHandler,
         dict(path=settings['static_path'])),
    ], **settings)

Safe as ``fileserver.py`` and run it with ``python2 fileserver.py``.

In case you prefer encapsulating it into a Docker container:

.. code-block:: none
    
    FROM ubuntu:14.04

    RUN apt-get update && apt-get -y upgrade
    RUN apt-get install -y python python-pip
    RUN pip install tornado

    RUN mkdir -p /fileserver
    ADD fileserver.py /fileserver

    RUN useradd -s /bin/bash fileserver
    RUN chown -R fileserver /fileserver
    
    USER fileserver
    WORKDIR /fileserver

    # serving as a HTTP file server on port 80 of the container
    EXPOSE 80
    
    CMD python2 /fileserver/fileserver.py

Build and run your Dockerfile:

.. code-block:: shell
    
    docker build -t fileserver .
    docker run -p 8080:80 -v /tmp:/tmp:ro fileserver:latest

This would publish the container port 80 to the host port 8080.
