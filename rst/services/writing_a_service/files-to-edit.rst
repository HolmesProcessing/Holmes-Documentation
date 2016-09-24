
Files to Edit
--------------
| The following table declares variables used in the sections below.
| These need to be replaced by their respective values.

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

Config Files
^^^^^^^^^^^^^
The following files can be found in the ``config/`` folder within the
Holmes-Totem repository.

totem.conf.example
"""""""""""""""""""
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
"""""""""""""""""""""""""""
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
"""""""""""""""""""""""""""
- This file runs docker-compose with certain environmental variables set, that allow fetching service configuration files from a server.
  Add an ``export`` statement like this:

  .. code-block:: shell

    export CONFSTORAGE_SERVICE_NAME=${CONFSTORAGE}zipmeta/

  .. warning::

    In the above example, ``${CONFSTORAGE}`` is the actual term and nothing
    needs to be replaced there.


Scala Files
^^^^^^^^^^^^
The following files can be found in the
``src/main/scala/org/holmesprocessing/totem/`` folder within the
Holmes-Totem repository

driver/driver.scala
""""""""""""""""""""
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
