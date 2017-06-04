Example Service: Hello World
************************************

Lets implement a service as simple as possible: Hello World.



totem.conf
::::::::::::::::::::::::::::::::::::

.. code-block:: shell

  totem {
    version = "0.5.0"

    download_settings {
      connection_pooling = true
      connection_timeout = 1000
      download_directory = "/tmp/"
      thread_multiplier = 4
      request_timeout = 1000
      validate_ssl_cert = true
    }

    tasking_settings {
      default_service_timeout = 180
      prefetch = 3
      retry_attempts = 3
    }

    rabbit_settings {
      requeueKey = "requeue.static.totem"
      host {
        server = "127.0.0.1"
        port = 5672
        username = "guest"
        password = "guest"
        vhost = "/"
      }
      exchange {
        name = "totem"
        type = "topic"
        durable = true
      }
      workqueue {
        name = "totem_input"
        routing_key = "work.static.totem"
        durable = true
        exclusive = false
        autodelete = false
      }
      resultsqueue {
        name = "totem_output"
        routing_key = "*.result.static.totem"
        durable = true
        exclusive = false
        autodelete = false
      }
      misbehavequeue {
        name = "totem_misbehave"
        routing_key = "misbehave.static.totem"
        durable = true
        exclusive = false
        autodelete = false
      }
    }

    services {
      helloworld {
        uri = ["http://127.0.0.1:8888/analyze/?obj="]
        resultRoutingKey = "helloworld.result.static.totem"
      }
    }
  }




driver.scala
::::::::::::::::::::::::::::::::::::

.. code-block:: scala

    package org.holmesprocessing.totem.driver

    import java.util.concurrent.{Executors, ExecutorService}

    import akka.actor.{ActorRef, ActorSystem, Props}
    import org.holmesprocessing.totem.actors._
    import org.holmesprocessing.totem.services.helloworld.{helloworldSuccess, helloworldWork}

    import org.holmesprocessing.totem.types._
    import org.holmesprocessing.totem.util.DownloadSettings

    import org.holmesprocessing.totem.util.Instrumented

    import java.io.File

    import com.typesafe.config.{Config, ConfigFactory}

    import scala.util.Random

    object driver extends App with Instrumented {
      // Define constants
      val DefaultPathConfigFile = "./config/totem.conf"

      lazy val execServ: ExecutorService = Executors.newFixedThreadPool(4000)
      val conf: Config = if (args.length > 0) {
        println("Using manual config file: " + args(0))
        ConfigFactory.parseFile(new File(args(0)))
      } else {
        println("Using default config file: " + DefaultPathConfigFile)
        ConfigFactory.parseFile(new File(DefaultPathConfigFile))
      }
      val system = ActorSystem("totem")

      println("Configuring details for Totem Tasking")
      val taskingConfig = TaskingSettings(
        conf.getInt("totem.tasking_settings.default_service_timeout"),
        conf.getInt("totem.tasking_settings.prefetch"),
        conf.getInt("totem.tasking_settings.retry_attempts")
      )

      println("Configuring details for downloading objects")
      val downloadConfig = DownloadSettings(
        conf.getBoolean("totem.download_settings.connection_pooling"),
        conf.getInt("totem.download_settings.connection_timeout"),
        conf.getString("totem.download_settings.download_directory"),
        conf.getInt("totem.download_settings.thread_multiplier"),
        conf.getInt("totem.download_settings.request_timeout"),
        conf.getBoolean("totem.download_settings.validate_ssl_cert")
      )

      println("Configuring details for Rabbit queues")
      val hostConfig = HostSettings(
        conf.getString("totem.rabbit_settings.host.server"),
        conf.getInt("totem.rabbit_settings.host.port"),
        conf.getString("totem.rabbit_settings.host.username"),
        conf.getString("totem.rabbit_settings.host.password"),
        conf.getString("totem.rabbit_settings.host.vhost")
      )

      val exchangeConfig = ExchangeSettings(
        conf.getString("totem.rabbit_settings.exchange.name"),
        conf.getString("totem.rabbit_settings.exchange.type"),
        conf.getBoolean("totem.rabbit_settings.exchange.durable")
      )

      val workqueueKeys = List[String](
        conf.getString("totem.rabbit_settings.workqueue.routing_key"),
        conf.getString("totem.rabbit_settings.requeueKey")
      )
      val workqueueConfig = QueueSettings(
        conf.getString("totem.rabbit_settings.workqueue.name"),
        workqueueKeys,
        conf.getBoolean("totem.rabbit_settings.workqueue.durable"),
        conf.getBoolean("totem.rabbit_settings.workqueue.exclusive"),
        conf.getBoolean("totem.rabbit_settings.workqueue.autodelete")
      )

      val resultQueueConfig = QueueSettings(
        conf.getString("totem.rabbit_settings.resultsqueue.name"),
        List[String](conf.getString("totem.rabbit_settings.resultsqueue.routing_key")),
        conf.getBoolean("totem.rabbit_settings.resultsqueue.durable"),
        conf.getBoolean("totem.rabbit_settings.resultsqueue.exclusive"),
        conf.getBoolean("totem.rabbit_settings.resultsqueue.autodelete")
      )

      val misbehaveQueueConfig = QueueSettings(
        conf.getString("totem.rabbit_settings.misbehavequeue.name"),
        List[String](conf.getString("totem.rabbit_settings.misbehavequeue.routing_key")),
        conf.getBoolean("totem.rabbit_settings.misbehavequeue.durable"),
        conf.getBoolean("totem.rabbit_settings.misbehavequeue.exclusive"),
        conf.getBoolean("totem.rabbit_settings.misbehavequeue.autodelete")
      )

      println("Configuring setting for Services")
      class TotemicEncoding(conf: Config) extends ConfigTotemEncoding(conf) { //this is a class, but we can probably make it an object. No big deal, but it helps on mem. pressure.
        def GeneratePartial(work: String): String = {
          work match {
            case "HELLOWORLD" => Random.shuffle(services.getOrElse("helloworld", List())).head
            case _ => ""
          }
        }

        def enumerateWork(key: Long, orig_filename: String, uuid_filename: String, workToDo: Map[String, List[String]]): List[TaskedWork] = {
          val w = workToDo.map({
            
            case ("HELLOWORLD", li: List[String]) =>
              pdfparseWork(key, uuid_filename, taskingConfig.default_service_timeout, "HELLOWORLD", GeneratePartial("HELLOWORLD"), li)
            case (s: String, li: List[String]) =>
              UnsupportedWork(key, orig_filename, 1, s, GeneratePartial(s), li)
            case _ => Unit //need to set this to a non Unit type.
          }).collect({
            case x: TaskedWork => x
          })
          w.toList
        }

        def workRoutingKey(work: WorkResult): String = {
          work match {
            case x: helloworldSuccess => conf.getString("totem.services.helloworld.resultRoutingKey")
            case _ => ""
          }
        }
      }

      println("Completing configuration")
      val encoding = new TotemicEncoding(conf)

      println("Creating Totem Actors")
      val myGetter: ActorRef = system.actorOf(RabbitConsumerActor.props[ZooWork](hostConfig, exchangeConfig, workqueueConfig, encoding, Parsers.parseJ, downloadConfig, taskingConfig).withDispatcher("akka.actor.my-pinned-dispatcher"), "consumer")
      val mySender: ActorRef = system.actorOf(Props(classOf[RabbitProducerActor], hostConfig, exchangeConfig, resultQueueConfig, misbehaveQueueConfig, encoding, conf.getString("totem.rabbit_settings.requeueKey"), taskingConfig), "producer")

      println("Totem version " + conf.getString("totem.version") + " is running and ready to receive tasks")

    }



HelloWorldREST.scala
::::::::::::::::::::::::::::::::::::

.. code-block:: scala

    package org.holmesprocessing.totem.services.pdfparse

    import dispatch.Defaults._
    import dispatch.{url, _}
    import org.json4s.JsonAST.{JString, JValue}
    import org.holmesprocessing.totem.types.{TaskedWork, WorkFailure, WorkResult, WorkSuccess}
    import collection.mutable


    case class helloworldWork(key: Long, filename: String, TimeoutMillis: Int, WorkType: String, Worker: String, Arguments: List[String]) extends TaskedWork {
      def doWork()(implicit myHttp: dispatch.Http): Future[WorkResult] = {

        val uri = helloworldREST.constructURL(Worker, filename, Arguments)
        val requestResult = myHttp(url(uri) OK as.String)
          .either
          .map({
          case Right(content) =>
            helloworldSuccess(true, JString(content), Arguments)

          case Left(StatusCode(404)) =>
            helloworldFailure(false, JString("Not found (File already deleted?)"), Arguments)

          case Left(StatusCode(500)) =>
            helloworldFailure(false, JString("Helloworld service failed, check local logs"), Arguments) //would be ideal to print response body here

          case Left(StatusCode(code)) =>
            helloworldFailure(false, JString("Some other code: " + code.toString), Arguments)

          case Left(something) =>
            helloworldFailure(false, JString("wildcard failure: " + something.toString), Arguments)
        })
        requestResult
      }
    }


    case class helloworldSuccess(status: Boolean, data: JValue, Arguments: List[String], routingKey: String = "helloworld.result.static.totem", WorkType: String = "HELLOWORLD") extends WorkSuccess
    case class pdfparseFailure(status: Boolean, data: JValue, Arguments: List[String], routingKey: String = "", WorkType: String = "HELLOWORLD") extends WorkFailure


    object helloworldREST {
      def constructURL(root: String, filename: String, arguments: List[String]): String = {
        arguments.foldLeft(new mutable.StringBuilder(root+filename))({
          (acc, e) => acc.append(e)}).toString()
      }
    }




Dockerfile for Python
::::::::::::::::::::::::::::::::::::

.. code-block:: shell

    # choose the operating system image to base of, refer to docker.com for available images
    FROM :apline

     # create a folder to contain your service's files
    RUN mkdir -p /service
    WORKDIR /service

    # add Tornado or Flask or any WSGI compliant wrapper  
    RUN pip install tornado

    # add dependencies for helloworld
    RUN pip3 install <....>

    # add all files relevant for running your service to your container
    COPY helloworld.py /service
    COPY LICENSE /service

    # add the configuration file (possibly from a storage URI)
    ARG conf=service.conf
    ADD $conf /service/service.conf

    CMD ["python3", "helloworld.py"]


Dockerfile for Golang
::::::::::::::::::::::::::::::::::::

.. code-block:: shell

    # choose the operating system image to base of, refer to docker.com for available images
    FROM golang:aplpine

    # create a folder to contain your service's files
    RUN mkdir -p /service
    WORKDIR /service

    # add Go dependencies
    RUN apk add --no-cache \
                        git \
                    && go get github.com/julienschmidt/httprouter \
                    && rm -rf /var/cache/apk/*

    # add dependencies for helloworld


    # add all files relevant for running your service to your container
    COPY helloworld.py /service
    COPY README.md /service


    # build the service
    RUN go build helloworld.go

    # add the configuration file (possibly from a storage URI)
    ARG conf=service.conf
    ADD $conf /service/service.conf

    CMD ["./helloworld", "--config=service.conf"]


helloworld.py
::::::::::::::::::::::::::::::::::::

.. code-block:: python

    import tornado
    import tornado.web
    import tornado.httpserver
    import tornado.ioloop
    import json

    import os
    from os import path


    # reading configuration file
    configPath = "./service.conf"
    config = json.loads(open(configPath).read())

    # service logic
    class Service (tornado.web.RequestHandler):
        def get(self, filename):
          # Getting object submitteed through URL
          object = self.get_argument('obj', strip=False)
            data = {
                "message": "Hello World!"
            }
            self.write(data)
        
          # return appropriate error codes
          raise tornado.web.HTTPError(status_code=code, log_message=custom_msg)

    # Generating info-output    
    class Info(tornado.web.RequestHandler):
        def get(self):
            description = """
                <p>Copyright 2017 Holmes Processing
                <p>Description: This is the HelloWorld Service for Totem.
            """
            self.write(description)


    class Application(tornado.web.Application):
        def __init__(self):
            handlers = [
                (r'/', Info),
                (r'/analyze/', Service),
            ]
            settings = dict(
                template_path=path.join(path.dirname(__file__), 'templates'),
                static_path=path.join(path.dirname(__file__), 'static'),
            )
            tornado.web.Application.__init__(self, handlers, **settings)
            self.engine = None


    def main():
        server = tornado.httpserver.HTTPServer(Application())
        server.listen(8888)
        tornado.ioloop.IOLoop.instance().start()


    if __name__ == '__main__':
        main()


helloworld.go
::::::::::::::::::::::::::::::::::::

.. code-block:: go
    
    package main

    import (
      "encoding/json"
      "flag"
      "github.com/julienschmidt/httprouter"
      "os"
      "net/http"
      "fmt"
    )

    var (
      config   *Config
      helloworld string
      metadata Metadata = Metadata{
        Name:        "helloworld",
        Version:     "0.1",
        Description: "./README.md",
        Copyright:   "Copyright 2017 Holmes Group LLC",
        License:     "./LICENSE",
      }
    )

    type Config struct {
      HTTPBinding        string
      MaxNumberOfObjects int
    }

    type Metadata struct {
      Name        string
      Version     string
      Description string
      Copyright   string
      License     string
    }

    type Result struct {
      key string  
    }
    func main() {

      var configPath string

      flag.StringVar(&configPath, "config", "", "Path to the configuration file")
      flag.Parse()

      // reading configuration file.
      config := &Config{}
      cfile, _ := os.Open(configPath)
      json.NewDecoder(cfile).Decode(&config)
      

      router := httprouter.New()
      router.GET("/analyze/", handler_analyze)
      router.GET("/", handler_info)
      http.ListenAndServe(":8080", router)
    }

    func handler_info(f_response http.ResponseWriter, r *http.Request, ps httprouter.Params) {
      // info-output
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

    func handler_analyze(f_response http.ResponseWriter, request *http.Request, params httprouter.Params) {
      obj := request.URL.Query().Get("obj")
      if obj == "" {
        http.Error(f_response, "Missing argument 'obj'", 400)
        return
      }
      sample_path := "/tmp/" + obj
      if _, err := os.Stat(sample_path); os.IsNotExist(err) {
        http.NotFound(f_response, request)
        return
      }
      
      //-----------------------------------------------------------------// 
      //                                                                 //
      //                    Write your service logic.                    //
      //                                                                 //
      //-----------------------------------------------------------------//
      result := &Result{
        key : "value",
      }

      f_response.Header().Set("Content-Type", "text/json; charset=utf-8")
      json2http := json.NewEncoder(f_response)

      if err := json2http.Encode(result); err != nil {
        http.Error(f_response, "Generating JSON failed", 500)
        return
      }
    }


