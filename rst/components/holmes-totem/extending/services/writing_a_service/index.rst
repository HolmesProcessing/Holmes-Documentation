************************************
Writing a Service
************************************

.. role:: red
    :class: font-color-red


Overview
::::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::

Files To Add
==========================

| The following table declares variables used in the sections below.

These need to be replaced by their respective values.

Compare with other service implementations for suitable values.


- All additional files required by your service also belong into the folder
  ``src/main/scala/org/holmesprocessing/totem/services/SERVICE_NAME``.

+---------------------------+---------------------------------------------------+
| SERVICE_NAME              | The name of the service, e.g. My_Totem_Service    |
+---------------------------+---------------------------------------------------+
| SERVICE_NAME_CAMELCASE    | CamelCase version of your service's name,         |
|                           | e.g. MyTotemService                               |
+---------------------------+---------------------------------------------------+
| SERVICE_NAME_UPPERCASE    | Upper case version of the service's name,         |
|                           | e.g. MY_TOTEM_SERVICE                             |
+---------------------------+---------------------------------------------------+
| SERVICE_NAME_LOWERCASE    | Lower case version of your service's name,        |
|                           | e.g. my_totem_service                             |
+---------------------------+---------------------------------------------------+
| CONFSTORAGE_SERVICE_NAME  | String CONFSTORAGE\_ followed by an upper case    |
|                           | version of your service's name,                   |
|                           | e.g. CONFSTORAGE_MY_TOTEM_SERVICE                 |
+---------------------------+---------------------------------------------------+
| SERVICE_IP                | The IP at which the service is reachable.         |
|                           | Usually this should be 127.0.0.1.                 |
+---------------------------+---------------------------------------------------+
| SERVICE_PORT              | The convention for ports is simple, file          |
|                           | processing services are in the ``77xx`` range,    |
|                           | no-file services are in the ``79xx`` range.       |
|                           | Additionally by default there is a gap of 10 free |
|                           | ports per service, so if the first service starts |
|                           | at ``7700`` the second starts at ``7710``.        |
+---------------------------+---------------------------------------------------+
| SERVICE_CLASS_SUCCESS     | The name of the service's success class,          |
|                           | e.g. MyTotemServiceSuccess                        |
+---------------------------+---------------------------------------------------+
| SERVICE_CLASS_WORK        | The name of the service's work class,             |
|                           | e.g. MyTotemServiceWork                           |
+---------------------------+---------------------------------------------------+

Service Files
----------------------------

All files in this section belong into the folder
``src/main/scala/org/holmesprocessing/totem/services/SERVICE_NAME``.

Dockerfile
^^^^^^^^^^^^^^^^^^^

In order to make optimal use of Docker's caching ability, you must use the given
Dockerfiles and extend them according to your services needs.

- Dockerfile for Python2:

  .. code-block:: dockerfile

    FROM python:2-alpine

    # add tornado
    RUN pip install tornado

    # create folder
    RUN mkdir -p /service
    WORKDIR /service

    # add holmeslibrary
    RUN apk add --no-cache \
            wget \
        && wget https://github.com/HolmesProcessing/Holmes-Totem-Service-Library/archive/v0.1.tar.gz \
        && tar xf v0.1.tar.gz \
        && mv Holmes-Totem-Service* holmeslibrary \
        && rm -rf /var/cache/apk/* v0.1.tar.gz

    ###
    # service specific options
    ###

    # INSTALL SERVICE DEPENDENCIES
    #
    #   RUN pip install <stuff>
    #   RUN apk add --no-cache <stuff>
    #   RUN wget <url> && tar xf <stuff> ...
    #   ...
    #

    # add the files to the container
    COPY LICENSE /service
    COPY README.md /service
    COPY service.py /service

    # ADD FURTHER SERVICE FILES
    #
    #   COPY specialLibrary/ /service/specialLibrary
    #   COPY extraFile.py /service
    #   ...
    #

    # add the configuration file (possibly from a storage uri)
    ARG conf=service.conf
    ADD $conf /service/service.conf

    CMD ["python2", "service.py"]


- Dockerfile for Python3

  .. code-block:: dockerfile

    FROM python:alpine

    # add tornado
    RUN pip3 install tornado

    # create folder
    RUN mkdir -p /service
    WORKDIR /service

    # add holmeslibrary
    RUN apk add --no-cache \
            wget \
        && wget https://github.com/HolmesProcessing/Holmes-Totem-Service-Library/archive/v0.1.tar.gz \
        && tar xf v0.1.tar.gz \
        && mv Holmes-Totem-Service* holmeslibrary \
        && rm -rf /var/cache/apk/* v0.1.tar.gz

    ###
    # service specific options
    ###

    # INSTALL SERVICE DEPENDENCIES
    #
    #   RUN pip install <stuff>
    #   RUN apk add --no-cache <stuff>
    #   RUN wget <url> && tar xf <stuff> ...
    #   ...
    #

    # add the files to the container
    COPY LICENSE /service
    COPY README.md /service
    COPY service.py /service

    # ADD FURTHER SERVICE FILES
    #
    #   COPY specialLibrary/ /service/specialLibrary
    #   COPY extraFile.py /service
    #   ...
    #

    # add the configuration file (possibly from a storage uri)
    ARG conf=service.conf
    ADD $conf /service/service.conf

    CMD ["python3", "service.py"]


- Dockerfile for Go 1.7:

  .. code-block:: dockerfile

    FROM golang:alpine

    # create folder
    RUN mkdir -p /service
    WORKDIR /service

    # get go dependencies
    RUN apk add --no-cache \
            git \
        && go get github.com/julienschmidt/httprouter \
        && rm -rf /var/cache/apk/*

    ###
    # passivetotal specific options
    ###

    # INSTALL SERVICE DEPENDENCIES
    #
    #   RUN go get <stuff>
    #   RUN apk add --no-cache <stuff>
    #   RUN wget <url> && tar xf <stuff> ...
    #   ...
    #

    # create directory to hold sources for compilation
    RUN mkdir -p src/SERVICE_NAME_LOWERCASE

    # add files to the container
    # sources files to GOPATH instead of /service for compilation
    COPY LICENSE /service
    COPY README.md /service
    COPY service.go /service

    # ADD FURTHER SERVICE FILES
    #
    #   COPY specialLibrary/ /service/specialLibrary
    #   COPY extraFile.go /service
    #   ...
    #

    # add the configuration file (possibly from a storage uri)
    ARG conf=service.conf
    ADD $conf /service/service.conf

    # build service
    RUN go build -o service.run *.go

    # clean up git
    # clean up behind the service build
    # clean up golang it is not necessary anymore
    RUN apk del --purge \
            git \
        && rm -rf /var/cache/apk/* \
        && rm -rf $GOPATH \
        && rm -rf /usr/local/go

    CMD ["./service.run"]


  .. warning::

    If you require a more complex namespacing in your service's code, check out
    the Passivetotal service's
    `Dockerfile <https://github.com/HolmesProcessing/Holmes-Totem/blob/master/src/main/scala/org/holmesprocessing/totem/services/passivetotal/Dockerfile>`_


LICENSE
^^^^^^^^^^^^^^^^^

- The license under which the service is distributed.

README.md
^^^^^^^^^^^^^^^^^

- An appropriate readme file for your service (also displayed if the service's
  info url is looked up)

  Example::

    # Passivetotal service for Holmes-Totem

    ## Description

    A simple service to check PassiveTotal for additional enrichment data.
    If you do not have an API key, visit http://www.passivetotal.org to get one.

    ## Usage

    Build and start the docker container using the included Dockerfile.

    Upon building the Dockerfile downloads a list of TLDs from iana.org.
    To update this list of TLDs, the image needs to be built again.

    The service accepts domain names, IP addresses and emails as request objects.
    These have to be supplied as a parameter after the request URL.
    (If the analysisURL parameter is set to /passivetotal, then a request for the
    domain www.passivetotal.org would look like this: /passivetotal/www.passivetotal.org)

    The service performs some checks to determine the type of the input object.
    If a passed domain name contains an invalid TLD, it is invalid and rejected.
    If a passed email address contains an invalid domain, it is rejected.
    If a passed IP is in a reserved range, it is rejected. (IETF RFC6890, RFC4291)

    Only if a request object is determined valid, it is sent to selected passivetotal
    api endpoints. The maximum of simultaneous requests is 9.
    If an error is encountered in any of the api queries, the request fails and returns
    an appropriate error code. Check the local logs for detailed information.
    If the query succeeds, a JSON struct containing all 9 api endpoints is returned.
    Those endpoints that were not queried are set to null.


acl.conf
^^^^^^^^^^^^^^^^^^

- Currently empty


service.conf.example
^^^^^^^^^^^^^^^^^^^^^^^^

- JSON file containing service settings like internal port (default must be
  8080), but also service specific settings like maybe output limits or
  parsing limits.

  Example:

  .. code-block:: json

    {
        "port": 8080,
        "max-output-lines": 10000
    }


watchdog.scala
^^^^^^^^^^^^^^^^^^^^

- Currently empty


YourServiceREST.scala
^^^^^^^^^^^^^^^^^^^^^^^^

.. note::
    
    For most services ``YourServiceREST.scala`` can be copy & pasted and the
    names adjusted.

Now that Totem knows about your service and where to find it, we need to tell it
how to communicate with the service. This is done in a separate file, that
defines 3 (or more) classes and one object: 

.. code-block:: scala
    
    case class YourServiceWork
    case class YourServiceSuccess
    case class YourServiceFailure
    object YourServiceREST

Full working example:

.. code-block:: scala
    
    package org.holmesprocessing.totem.services.yourservice

    import dispatch.Defaults._
    import dispatch.{url, _}
    import org.json4s.JsonAST.{JString, JValue}
    import org.holmesprocessing.totem.types.{TaskedWork, WorkFailure, WorkResult, WorkSuccess}
    import collection.mutable


    case class yourserviceWork(key: Long, filename: String, TimeoutMillis: Int, WorkType: String, Worker: String, Arguments: List[String]) extends TaskedWork {
      def doWork()(implicit myHttp: dispatch.Http): Future[WorkResult] = {

        val uri = yourserviceREST.constructURL(Worker, filename, Arguments)
        val requestResult = myHttp(url(uri) OK as.String)
          .either
          .map({
          case Right(content) =>
            yourserviceSuccess(true, JString(content), Arguments)

          case Left(StatusCode(404)) =>
            yourserviceFailure(false, JString("Not found (File already deleted?)"), Arguments)

          case Left(StatusCode(500)) =>
            yourserviceFailure(false, JString("Objdump service failed, check local logs"), Arguments) //would be ideal to print response body here

          case Left(StatusCode(code)) =>
            yourserviceFailure(false, JString("Some other code: " + code.toString), Arguments)

          case Left(something) =>
            yourserviceFailure(false, JString("wildcard failure: " + something.toString), Arguments)
        })
        requestResult
      }
    }


    case class yourserviceSuccess(status: Boolean, data: JValue, Arguments: List[String], routingKey: String = "yourservice.result.static.totem", WorkType: String = "YOURSERVICE") extends WorkSuccess
    case class yourserviceFailure(status: Boolean, data: JValue, Arguments: List[String], routingKey: String = "", WorkType: String = "YOURSERVICE") extends WorkFailure


    object yourserviceREST {
      def constructURL(root: String, filename: String, arguments: List[String]): String = {
        arguments.foldLeft(new mutable.StringBuilder(root+filename))({
          (acc, e) => acc.append(e)}).toString()
      }
    }




**Explanation**

The ``YourServiceWork`` class initiates the request with your service, creating
the final ``uri`` via the ``YourServiceREST`` object.

The request result is gathered and depending on what the returned HTTP status
code was, a specific class (``YourServiceSuccess`` or ``YourServiceFailure``)
is instantiated with the result as a parameter and returned.

.. warning::
    
    The 2 generic cases at the end of the map should be there in any case to
    avoid exceptions.

The ``YourServiceSuccess`` and ``YourServiceFailure`` classes should be self-explanatory. They extend the default interfaces for success and failure and are
very convenient for mapping cases as done in ``driver.scala``, for example.

``YourServiceREST`` object should be self-explanatory as well, it defines how the
request address for your service gets constructed from the supplied parameters.




Service-logic file
^^^^^^^^^^^^^^^^^^^^^^^^

This is the file that makes the Service act like a web server. The service can be accessible from 2 endpoints.

+----------------+-----------------------------------+
| Endpoint       | Operation                         |
+================+===================================+
| /              | Provide information about the     |
|                | service                           |
+----------------+-----------------------------------+
| /analyze?obj=? | Perform tasking and return results|
+----------------+-----------------------------------+

Endpoint - '/'
""""""""""""""""""""""
You can use this page to view the information about the Service. Basically this page should state every aspect of Service the you are creating.

The INFO page should contain

1. Author name.
2. Service name and version. ( or any metadata about the service. )
3. Brief Description about the Service.
4. Licence
5. General information about how to use the Service and expected JSON output.

**Info-output Generation in Go**
  .. code-block:: Go

    func info_output(f_response http.ResponseWriter, r *http.Request, ps httprouter.Params) {
        fmt.Fprintf(f_response, `<p>%s - %s</p>
            <hr>
            <p>%s</p>
            <hr>
            <p>%s</p>
            `,
            metadata.Name,
            metadata.Version,
            metadata.Description,
            metadata.License)
    }

**Info-Output Generation in Python**
  .. code-block:: python

    class InfoHandler(tornado.web.RequestHandler):
            # Emits a string which describes the purpose of the analytics
            def get(self):
                info = """
      <p>{name:s} - {version:s}</p>
      <hr>
      <p>{description:s}</p>
      <hr>
      <p>{license:s}</p>
      <hr>
      <p>{copyright:s}</p>
                """.strip().format(
                    name        = name,
                    version     = version,
                    description = description,
                    license     = license,
                    copyright   = copyright
                )
                self.write(info)
        return InfoHandler


Endpoint '/analyze?obj='
"""""""""""""""""""""""""""
In this Endpoint you write the logic for interacting with **analyer library** and producing the JSON output and appropriate error codes.

Reading configuration file
............................

Before you start about writing the service logic, you first need to parse settings from service.conf ( renamed from service.conf.example ). Since the format Service's configuration file JSON, you can use any JSON parsing library.

**Reading configuration in Golang**

With the ``json`` package it's a snap to read JSON data into your Go programs. The json package provides Decoder and Encoder types to support the common operation of reading and writing streams of JSON data. We read the configuration file and then we fit the output to ``Config`` struct

.. code-block:: Go

    package main

    import (
        "encoding/json" 
        "flag" 
        "os"
    )

    // ....

    var (
        config *Config
        configPath string
    )

    // ....

    type Config struct {
    HTTPBinding string MaxNumberOfObjects int
    }

    // ....

    flag.StringVar(&configPath, "config", "", "Path to the configuration file") flag.Parse()

    config := &Config{}

    cfile, _ := os.Open(configPath) dec := json.NewDecoder(cfile) // reading from json data

    if err := dec.Decode(&config); err != nil {
    // handle error

    }

**Reading configuration for Python**

Try opening the path, reading it all in and parsing it as a JSON. If an error occurs, throw a tornado.web.HTTPError (well define behaviour by tornado for these) If parsing succeeds, update provided config dictionary.

.. code-block:: python

    # reading configuration file
      configPath = "./service.conf"
      config = json.loads(open(configPath).read())


HTTP Error Codes
.......................

Holmes totem service is RESTful service which communicates with HTTP protocol.  The first line of the HTTP response is called the status line and includes a numeric status code (such as "404") and a textual reason phrase (such as "Not Found"). Also when something went wrong in the service, we should return a HTTP error code so that user agent can debug accordingly.


+----------------+---------------------------+--------------------------------------------------------------+ 
|HTTP Error Code | Summary                   | Operation                                                    | 
+================+===========================+==============================================================+ 
| 200            |  OK                       |  Services returns result                                     |
+----------------+---------------------------+--------------------------------------------------------------+
| 400            |  Bad Operation            |  Missing argument `Obj`.                                     |
+----------------+---------------------------+--------------------------------------------------------------+
| 401            |  Authorization required   |  *currently empty*                                           |
+----------------+---------------------------+--------------------------------------------------------------+
| 404            |  Not Found                | Invalid URL format.  A user should follow URL API scheme     |
|                |                           | to submit objects.                                           |
+----------------+---------------------------+--------------------------------------------------------------+
| 500            |  Internal Server Error    | Generating JSON failed                                       |
+----------------+---------------------------+--------------------------------------------------------------+

- Raising error codes in Go
.. code-block:: Go

   func returnCode500(w http.ResponseWriter, r *http.Request) {
    http.Error(f_response, "Generating JSON failed'", 500)
   }

   func returnCode400(w http.ResponseWriter, r *http.Request) {
    http.Error(f_response, " Missing argument Obj", 400)
   }

   func returnCode404(w http.ResponseWriter, r *http.Request) {
    http.Error(f_response, " Invalid URL format. ", 404)
   }

- Raising error codes in Python
.. code-block:: python

    # For python tornado.  
    raise tornado.web.HTTPError(status_code=code, log_message=custom_msg)


Files to Edit
==========================

Config Files
--------------------
The following files can be found in the ``config/`` folder within the
Holmes-Totem repository.

totem.conf.example
^^^^^^^^^^^^^^^^^^^^^^^^

- | The entry in the totem.conf tells Totem your service exists, where to reach it, and where to store its results.

  The ``uri`` field supports multiple address entries for automatic load balancing.

  .. code-block:: shell

    totem {
        services {
            SERVICE_NAME_LOWERCASE {
                uri = ["http://SERVICE_IP:SERVICE_PORT/analyze/?obj="]
                resultRoutingKey = "SERVICE_NAME_LOWERCASE.result.static.totem"
            }
        }
    }


docker-compose.yml.example
^^^^^^^^^^^^^^^^^^^^^^^^^^^

- Holmes-Totem relies on Docker to provide the services. As such all services need
  to provide an entry in the docker-compose file.

  .. code-block:: yaml

    services:
      SERVICE_NAME_LOWERCASE:
        build:
          context: ../src/main/scala/org/holmesprocessing/totem/services/SERVICE_NAME_LOWERCASE
          args:
            conf: ${CONFSTORAGE_SERVICE_NAME}service.conf
        ports:
          - "SERVICE_PORT:8080"
        restart: unless-stopped

  If the service processes files (i.e. it needs access to ``/tmp/`` on the host),
  the following option needs to be added additionally to build, ports, and restart:

  .. code-block:: yaml

    volumes:
      - /tmp:/tmp:ro


compose_download_conf.sh
^^^^^^^^^^^^^^^^^^^^^^^^^

- This file runs docker-compose with certain environmental variables set, that allow fetching service configuration files from a server.
  Add an ``export`` statement like this:

  .. code-block:: shell

    export CONFSTORAGE_SERVICE_NAME=${CONFSTORAGE}zipmeta/

  .. warning::

    In the above example, ``${CONFSTORAGE}`` is the actual term and nothing
    needs to be replaced there.


Scala Files
-------------------

The following files can be found in the
``src/main/scala/org/holmesprocessing/totem/`` folder within the
Holmes-Totem repository

driver.scala
^^^^^^^^^^^^^^^^^^^^^^^^

- Import your services scala classes (see the respective section for information
  on these classes).

  .. code-block:: scala

      import org.holmesprocessing.totem.services.SERVICE_NAME_LOWERCASE.{
          SERVICE_CLASS_SUCCESS,
          SERVICE_CLASS_WORK
      }

- Add a case to the method ``GeneratePartial``

  .. code-block:: scala

      def GeneratePartial(work: String): String = {
        work match {
          case "SERVICE_NAME_UPPERCASE" => Random.shuffle(services.getOrElse("SERVICE_NAME_LOWERCASE", List())).head
        }
      }

- Add a case to the method ``enumerateWork``.

  .. warning::
      If your service does not process
      files but rather the input string, use ``uuid_filename`` instead of
      ``orig_filename`` below.

  .. code-block:: scala

      def enumerateWork(key: Long, orig_filename: String, uuid_filename: String, workToDo: Map[String, List[String]]): List[TaskedWork] = {
        val w = workToDo.map({
          case ("SERVICE_NAME_UPPERCASE", li: List[String]) => SERVICE_CLASS_WORK(key, orig_filename, taskingConfig.default_service_timeout, "SERVICE_NAME_UPPERCASE", GeneratePartial("SERVICE_NAME_UPPERCASE"), li)
        }).collect({
          case x: TaskedWork => x
        })
        w.toList
      }

- Add a case to the method ``workRoutingKey``

  .. code-block:: scala

    def workRoutingKey(work: WorkResult): String = {
      work match {
        case x: SERVICE_CLASS_SUCCESS => conf.getString("totem.services.SERVICE_NAME_LOWERCASE.resultRoutingKey")
      }
    }
