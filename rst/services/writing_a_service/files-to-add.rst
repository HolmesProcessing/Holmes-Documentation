
Files to Add
--------------
| The following table declares variables used in the sections below.

These need to be replaced by their respective values.

Compare with other service implementations for suitable values.

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
^^^^^^^^^^^^^^

All files in this section belong into the folder
``src/main/scala/org/holmesprocessing/totem/services/SERVICE_NAME``.

Dockerfile
"""""""""""
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
    # sources files to to GOPATH instead of /service for compilation
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
""""""""
- The license under which the service is distributed.

README.md
""""""""""
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

    The service accepts domain names, ip addresses and emails as request objects.
    These have to be supplied as a parameter after the request URL.
    (If the analysisURL parameter is set to /passivetotal, then a request for the
    domain www.passivetotal.org would look like this: /passivetotal/www.passivetotal.org)

    The service performs some checks to determine the type of the input object.
    If a passed domain name contains an invalid TLD, it is invalid and rejected.
    If a passed email address contains an invalid domain, it is rejected.
    If a passed IP is in a reserved range, it is rejected. (ietf rfcs 6890, 4291)

    Only if a request object is determined valid, it is sent to selected passivetotal
    api endpoints. The maximum of simultaneous requests is 9.
    If an error is encountered in any of the api queries, the request fails and returns
    an appropriate error code. Check the local logs for detailed information.
    If the query succeeds, a json struct containing all 9 api end points is returned.
    Those endpoints that were not queried are set to null.


acl.conf
"""""""""
- Currently empty


service.conf.example
"""""""""""""""""""""
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
"""""""""""""""
- Currently empty


Service Logic
""""""""""""""
- | At least one executable file or script is required to run your service.

  This could be for example a Python script or a Go executable.

  .. note::

    See the respective section for the contents of this one.

    TODO: create this documentation section

- All additional files required by your service also belong into the folder
  ``src/main/scala/org/holmesprocessing/totem/services/SERVICE_NAME``.

