Example Service: Hello World
************************************

Lets implement a service as simple as possible: Hello World.



totem.conf
::::::::::::::::::::::::::::::::::::

.. code-block:: shell

    zoo {
      version = "1.0.0"
      download_directory = "/tmp/"
      requeueKey = "requeue.static.totem"
      misbehaveKey = "misbehave.static.totem"

      rabbit_settings {
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
      }

      enrichers {
        helloworld {
          uri = ["http://127.0.0.1:8888/helloworld/"]
          resultRoutingKey = "helloworld.result.static.totem"
        }
      }
    }



driver.scala
::::::::::::::::::::::::::::::::::::

.. code-block:: scala

    package org.novetta.zoo.driver

    import java.util.concurrent.{Executors, ExecutorService}

    import akka.actor.{ActorRef, ActorSystem, Props}
    import org.novetta.zoo.actors._
    import org.novetta.zoo.services.helloworld{HelloWorldSuccess, HelloWorldWork}
    import org.novetta.zoo.types._

    import org.json4s._
    import org.json4s.JsonDSL._
    import org.json4s.jackson.JsonMethods._
    import akka.routing.RoundRobinPool
    import org.novetta.zoo.util.Instrumented

    import java.io.File

    import com.typesafe.config.{Config, ConfigFactory}

    import scala.util.Random

    object driver extends App with Instrumented {
      lazy val execServ: ExecutorService = Executors.newFixedThreadPool(4000)
      val conf: Config = if (args.length > 0) {
        println("we have args > 0, using args")
        ConfigFactory.parseFile(new File(args(0)))

      } else {
        ConfigFactory.parseFile(new File("/Users/totem/novetta/totem/config/totem.conf"))
      }
      val system = ActorSystem("totem")

      val hostConfig = HostSettings(
        conf.getString("zoo.rabbit_settings.host.server"),
        conf.getInt("zoo.rabbit_settings.host.port"),
        conf.getString("zoo.rabbit_settings.host.username"),
        conf.getString("zoo.rabbit_settings.host.password"),
        conf.getString("zoo.rabbit_settings.host.vhost")
      )

      val exchangeConfig = ExchangeSettings(
        conf.getString("zoo.rabbit_settings.exchange.name"),
        conf.getString("zoo.rabbit_settings.exchange.type"),
        conf.getBoolean("zoo.rabbit_settings.exchange.durable")
      )
      val workqueueConfig = QueueSettings(
        conf.getString("zoo.rabbit_settings.workqueue.name"),
        conf.getString("zoo.rabbit_settings.workqueue.routing_key"),
        conf.getBoolean("zoo.rabbit_settings.workqueue.durable"),
        conf.getBoolean("zoo.rabbit_settings.workqueue.exclusive"),
        conf.getBoolean("zoo.rabbit_settings.workqueue.autodelete")
      )
      val resultQueueConfig = QueueSettings(
        conf.getString("zoo.rabbit_settings.resultsqueue.name"),
        conf.getString("zoo.rabbit_settings.resultsqueue.routing_key"),
        conf.getBoolean("zoo.rabbit_settings.resultsqueue.durable"),
        conf.getBoolean("zoo.rabbit_settings.resultsqueue.exclusive"),
        conf.getBoolean("zoo.rabbit_settings.resultsqueue.autodelete")
      )

      class TotemicEncoding(conf: Config) extends ConfigTotemEncoding(conf) {
        def GeneratePartial(work: String): String = {
          work match {
            case "HELLOWORLD" => Random.shuffle(services.getOrElse("helloworld", List())).head
          }
        }

        def enumerateWork(key: Long, filename: String, workToDo: Map[String, List[String]]): List[TaskedWork] = {
          val w = workToDo.map({
            case ("HELLOWORLD", li: List[String]) =>
              HelloWorldWork(key, filename, 60, "HELLOWORLD", GeneratePartial("HELLOWORLD"), li)

            case (s: String, li: List[String]) =>
              UnsupportedWork(key, filename, 1, s, GeneratePartial(s), li)

            case _ => Unit
          }).collect({
            case x: TaskedWork => x
          })
          w.toList
        }

        def workRoutingKey(work: WorkResult): String = {
          work match {
            case x: HelloWorldSuccess => "helloworld.result.static.zoo"
          }
        }
      }

      val encoding = new TotemicEncoding(conf)

      val myGetter: ActorRef = system.actorOf(RabbitConsumerActor.props[ZooWork](hostConfig, exchangeConfig, workqueueConfig, encoding, Parsers.parseJ).withDispatcher("akka.actor.my-pinned-dispatcher"), "consumer")
      val mySender: ActorRef = system.actorOf(Props(classOf[RabbitProducerActor], hostConfig, exchangeConfig, resultQueueConfig, conf.getString("zoo.requeueKey"), conf.getString("zoo.misbehaveKey")), "producer")


      // Demo & Debug Zone
      val zoowork = ZooWork("http://localhost/rar.exe", "http://localhost/rar.exe", "winrar.exe", Map[String, List[String]]("YARA" -> List[String]()), 0)

      val json = (
        ("primaryURI" -> zoowork.primaryURI) ~
          ("secondaryURI" -> zoowork.secondaryURI) ~
          ("filename" -> zoowork.filename) ~
          ("tasks" -> zoowork.tasks) ~
          ("attempts" -> zoowork.attempts)
        )

      private[this] val loading = metrics.timer("loading")

      val j = loading.time({
        compact(render(json))
      })

      mySender ! Send(RMQSendMessage(j.getBytes, workqueueConfig.routingKey))

      println("Totem is Running! \nVersion: " + conf.getString("zoo.version"))
    }



HelloWorldREST.scala
::::::::::::::::::::::::::::::::::::

.. code-block:: scala

    package org.novetta.zoo.services.zipmeta

    import dispatch.Defaults._
    import dispatch.{url, _}
    import org.json4s.JsonAST.{JString, JValue}
    import org.novetta.zoo.types.{TaskedWork, WorkFailure, WorkResult, WorkSuccess}
    import collection.mutable


    case class HelloWorldWork(key: Long, filename: String, TimeoutMillis: Int, WorkType: String, Worker: String, Arguments: List[String]) extends TaskedWork {
      def doWork()(implicit myHttp: dispatch.Http): Future[WorkResult] = {

        val uri = HelloWorldREST.constructURL(Worker, filename, Arguments)
        val requestResult = myHttp(url(uri) OK as.String)
          .either
          .map({

          case Right(content) =>
            HelloWorldSuccess(true, JString(content), Arguments)

          case Left(something) =>
            HelloWorldFailure(false, JString("Hello World failed ... :/ (maybe the service isn't up and running?)"), Arguments)

        })
        requestResult
      }
    }


    case class HelloWorldSuccess(status: Boolean, data: JValue, Arguments: List[String], routingKey: String = "helloworld.result.static.totem", WorkType: String = "HELLOWORLD") extends WorkSuccess
    case class HelloWorldFailure(status: Boolean, data: JValue, Arguments: List[String], routingKey: String = "", WorkType: String = "HELLOWORLD") extends WorkFailure


    object HelloWorldREST {
      def constructURL(root: String, filename: String, arguments: List[String]): String = {
        arguments.foldLeft(new mutable.StringBuilder(root+filename))({
          (acc, e) => acc.append(e)}).toString()
      }
    }



Dockerfile
::::::::::::::::::::::::::::::::::::

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
    ADD helloworld.py /service

    # create a new user with limited access
    RUN useradd -s /bin/bash service
    RUN chown -R service /service

    # switch user and work directory
    USER service
    WORKDIR /service

    # expose our container on some port
    EXPOSE 8888

    # define our command to run if we start our container
    CMD python2 /service/helloworld.py



helloworld.py
::::::::::::::::::::::::::::::::::::

.. code-block:: python

    import tornado
    import tornado.web
    import tornado.httpserver
    import tornado.ioloop

    import os
    from os import path


    class Service (tornado.web.RequestHandler):
        def get(self, filename):
            data = {
                "message": "Hello World!"
            }
            self.write(data)


    class Info(tornado.web.RequestHandler):
        def get(self):
            description = """
                <p>Copyright 2015 Holmes Processing
                <p>Description: This is the HelloWorld Service for Totem.
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
        server.listen(8888)
        tornado.ioloop.IOLoop.instance().start()


    if __name__ == '__main__':
        main()



