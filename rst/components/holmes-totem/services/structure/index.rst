Structure
**************
A Holmes Totem Service requires four files. These files are

* ``Scala File`` : Logic for communication with totem.
* ``Configuration File`` : Configuration settings which can be changed as needed.
* ``Service Logic`` : Web Server and Logic of the entire service.
* ``Dockerfile`` : To containerize and isolate the entire Service



Configuration file
======================
Let's discuss how Holmes being a distributed system, uses configuration files. Configuration files configure initial settings for the Services. The settings of each of these Services will be stored in a configuration file.

Service Configuration for Holmes Totem
-----------------------------------------
The essential settings needed for starting Totem Service are located in configuration file. The file format of Holmes’ configuration files is `.conf`. The format of the text is JSON. A user can change these values to change the behavior of the Service.

For each Service, The Service author should create a file called `service.conf` where the essential configuration settings for the service like HTTPBinding, Maximum number of objects etc. A type Totem Service configuration file will look like this:

.. code-block:: json

	{
		"settings": {
			"httpbinding": 8080
		},
		"<service_name": {
			"<Key": "Value",
			"<key>": "Value"
		}
	}

.. warning::
	All the services by default should run on port 8080 so that docker compose can port forward it to a specified port


Centralised Service configuration of Holmes Totem
----------------------------------------------------
Holmes system allows an admin to store the Totem Service configurations in a central location and make the system automatically load it from there upon upstart. This is useful when you start up multiple Totem at different locations all working with the same service configuration because copying of all the Services everywhere is quite tedious. Instead, you only have to modify the config file on one machine and upload it and then rebuilt the containers on all the machines. It makes distributed service configuration changes easier.


Holmes Storage is extended to allow for storing configuration-files (uploaded over HTTP) into its database and query them (also over HTTP). The `upload_config.sh <https://github.com/HolmesProcessing/Holmes-Totem/blob/master/config/upload_configs.sh/>`_. script can be used for uploading configuration to Storage. This script creates environment variables CONFSTORAGE which storage the URI where the configuration files are stored in Storage and configuration files are uploaded to storage. The `compose_download_conf.sh <https://github.com/HolmesProcessing/Holmes-Totem/blob/master/config/compose_download_conf.sh/>`_. script creates the environment variable for each Service like for example CONFSTORAGE_ASNMETA which contains the URI of the configuration file for the Service

The docker-compose.yml.example therefore takes a look at environment variables pointing to the running instance of Holmes Storage and sets these arguments correctly for each Service. By doing this, whenever the containers are built, the configuration-files are pulled from the server. As you can see in the docker-compose.yml.example, the Services always have a line like this:

.. code-block:: shell
	
	conf: ${CONFSTORAGE_ASNMETA}service.conf

Usually the environment variable `CONFSTORAGE_ASNMETA` (and all the others as well) are empty, so the Dockerfiles just get the local config version

Also The following are the modifications done for Dockerfile to accept an argument specifying the location of the file service.conf

.. code-block:: shell

	# add the configuration file (possibly from a storage uri)
	ARG conf=service.conf
	ADD $conf /service/service.conf

If docker-compose did not set the conf-argument, it defaults to service.conf, otherwise it is left as it was. Docker's ADD can also download the file via HTTP.

Reading Configuration files for Services.
---------------------------------------------
Each Service runs independently in an isolated docker container. The configuration settings for the Service has to be provided in `service.conf`. When The service runs, it first looks for the configuration file, in order to read its settings, and applies them to the Service. The configuration file has to be in JSON format.

Service Logic
=========================

Communication
-------------------
Currently Holmes Totem Services uses REST approach which leverages the HTTP protocol for communication. Totem requests a task to the Service by GET request. The format of url is 

.. code-block:: shell

	GET http://address:port/analyze/obj?=sample-id

When a request to /analyze route is made, totem looks for the sample in Holmes Storage, if the sample if found, that will be submitted for analysis to the Service and the Service responds with result. If TOTEM could not find the sample in Storage, It simply returns 404 HTTP error. The various endpoints through which a Service can be interacted with is written in API Endpoints

API Endpoints
---------------------

**GET  /**

Returns general documentation information about the Service.
* Resource URL: http://address:port/
* Parameters: None

**GET /analyse**

Returns the analysis result for a given sample

* Resource URL:   ``http://address:port/analyze/?obj=sample-id``
* Parameters: The name of the object to be analysed.
* Example Request : ``http://address:port/analyze/?obj=sample-id``

Example Response : For example response, please refer to README.md of any Service

Scala File
-----------------
Holmes-Totem schedules the execution of its Services. Holmes Totem Services are web servers that receive tasks via HTTP request. This file tells Service How to interact with Totem. Totem imports this file in `driver.scala` and schedules the task

Containerization
----------------------

Since we are trying to analyse malware sample, there could be a risk that analysis could could damage a environment in which Service is running on. To minimize this risk, we should we use sandbox environment. 

For our purpose, we generally need a Virtual Machine. But only need virtualization for the sake of isolation. And we want them to be lightweight. So Docker is ideal for our requirements. 
To pack and isolate the above discussed parts (except Scala file), we need to do containerization. A typical Dockerfile of Holmes Totem Service will look like:

.. code-block:: shell

	FROM <base-image>

	# Create folder
	RUN mkdir -p /service
	WORKDIR /service

	# Get Language dependencies
	RUN apk add --no-cache \
	        git \
	        && rm -rf /var/cache/apk/*

	# Get Analyzer Library dependencies
	RUN apk add --no-cache \
		   && rm -rf /var/cache/apk/*

	# Clean Up
	RUN apk del --purge \
	        git 

	# Set environment variables.

	# Installing Analyzer library

	# add Service files to the container.
	COPY LICENSE /service
	COPY README.md /service
	COPY service.{go, py} /service

	# build the service file

	# add the configuration file (possibly from a storage uri)
	ARG conf=service.conf
	ADD $conf /service/service.conf

	# Run the Service
	CMD ["./service", "--config=service.conf"]


**BASE IMAGE:**


The FROM directive in dockerfile is used to mention the base image. To make the container light weight, we use Alpine Linux.

You need to choose the docker image suitable for the task you are trying to achieve. That is choosing the right container for the language in which in you are writing Service.

For Go: 
``FROM golang:alpine``

For Python:
``FROM python:alpine``

