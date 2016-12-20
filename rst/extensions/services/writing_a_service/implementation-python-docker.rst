.. _implementation-service-python-docker:

Service Implementation: Python with Docker
--------------------------------------------

.. _CRITs: https://crits.github.io
.. _Docker: http://www.docker.com
.. _Tornado: http://www.tornadoweb.org/en/stable/

Any programming language could be used to implement your service.
This tutorial uses Python because it's simple and the services
for the popular analysis platform CRITs_ are written in it.

As well any kind of virtualization method can be used to restrict the environment of
your Service. This tutorial shows how to use Docker_.

All services made available to the public should provide a
Dockerfile (uniformity and ease of use across Totem).



Install Dependencies
^^^^^^^^^^^^^^^^^^^^^^

.. note::
    
    Please refer to the Dockerfile below to see how to use it with Docker.


First of all, Python is required, as well as pip:

.. code-block:: shell
    
    apt-get install python python-pip

As already explained in the section **Communication Protocol** the Service needs
to act as a webserver, accepting requests from Totem.
For details please read up in the corresponding section.

One easy way of providing such a webserver is to use Tornado_:

.. code-block:: shell
    
    pip install tornado

That's all dependencies we'll need for a simple service that does basically nothing.
If you have any further dependencies, they need to be installed in this step as well.
(Like additional python frameworks)



Dockerfile
^^^^^^^^^^^^

If you put all of that into a Dockerfile, the result might look like this:

.. code-block:: shell
    
    # choose the operating system image to base of, refer to docker.com for available images
    FROM ubuntu:14.04

    # install what you need here
    RUN apt-get update && apt-get -y upgrade
    RUN apt-get install -y python python-pip
    RUN pip install tornado

    # create a folder to contain your service's files
    RUN mkdir -p /service
    
    # add all files relevant for running your service to your container
    ADD service.py /service
    # in case you want to access the holmeslibrary:
    # ADD ../library/holmeslibrary.py /service

    # create a new user with limited access
    RUN useradd -s /bin/bash service
    RUN chown -R service /service
    
    # switch user and work directory
    USER service
    WORKDIR /service

    # expose our container on some port
    EXPOSE 7715
    
    # define our command to run if we start our container
    CMD python2 /service/service.py

It is important to think about the ordering the commands have in the Dockerfile,
as that can speed up or slow down the container build process heavily.
Stuff that does not need to be done on every build should go to the front of the
Dockerfile, stuff that changes should go towards the end of the file.

(Docker cashes previous build steps and if nothing changes, those build steps
will be reused on the next build, speeding it up by a lot, especially when
installing python like in this Dockerfile)



The service.py
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This requires several imports for Tornado. (And the holmeslibary in case you
want to use it, which is encouraged as it provides useful helper classes)

.. code-block:: python
    
    import tornado
    import tornado.web
    import tornado.httpserver
    import tornado.ioloop
    
    import os
    from os import path
    
    import holmeslibrary

For a really basic webserver you 4 ingredients, a main function, an application
function, a info request handler, and a request handler that actually does the
service's work.

.. code-block:: python
    
    class Service (tornado.web.RequestHandler):
        def get(self, filename):
            data = {
                "message": """
                    Hello World I'm a service and I'm doing important analysis work!
                    The file I'm requested to process has the sample-id: %s
                """ % filename
            }
            self.write(data)
    
    class Info(tornado.web.RequestHandler):
        def get(self):
            description = """
                <p>Copyright 2015 Holmes Processing
                <p>Description: Service Description goes here.
            """
            self.write(description)

    class Application(tornado.web.Application):
        def __init__(self):
            handlers = [
                (r'/', Info),
                (r'/yourservice/([a-zA-Z0-9\-]*)', Service),
            ]
            settings = dict(
                template_path=path.join(path.dirname(__file__), 'templates'),
                static_path=path.join(path.dirname(__file__), 'static'),
            )
            tornado.web.Application.__init__(self, handlers, **settings)
            self.engine = None


    def main():
        server = tornado.httpserver.HTTPServer(Application())
        server.listen(7715)
        tornado.ioloop.IOLoop.instance().start()


    if __name__ == '__main__':
        main()

The port in the main function needs to be adjusted as necessary and all the
services work should go either into the Service class, or should be called from
there.

.. warning::
    
    Please note, that while the Info class writes a string, the Service class must
    write a dictionary. (Totem communicates via JSON!)
