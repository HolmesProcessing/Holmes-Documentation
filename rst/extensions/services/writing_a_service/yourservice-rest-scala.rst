YourServiceREST.scala
-----------------------

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
    
    package org.novetta.zoo.services.yourservice
    
    import dispatch.Defaults._
    import dispatch.{url, _}
    import org.json4s.JsonAST.{JString, JValue}
    import org.novetta.zoo.types.{TaskedWork, WorkFailure, WorkResult, WorkSuccess}
    import collection.mutable
    
    
    case class YourServiceWork(key: Long, filename: String, TimeoutMillis: Int, WorkType: String, Worker: String, Arguments: List[String]) extends TaskedWork {
      def doWork()(implicit myHttp: dispatch.Http): Future[WorkResult] = {
      
        val uri = YourServiceREST.constructURL(Worker, filename, Arguments)
        val requestResult = myHttp(url(uri) OK as.String)
          .either
          .map({
          case Right(content) =>
            YourServiceSuccess(true, JString(content), Arguments)
            
          case Left(StatusCode(400)) =>
            YourServiceFailure(false, JString("Bad request"), Arguments)
          
          # ... have any amount of status codes and corresponding messages as you like
          
          case Left(StatusCode(code)) =>
            YourServiceFailure(false, JString("Some other code: " + code.toString), Arguments)
            
          case Left(something) =>
            YourServiceFailure(false, JString("wildcard failure: " + something.toString), Arguments)
        })
        requestResult
      }
    }
    
    
    case class YourServiceSuccess(status: Boolean, data: JValue, Arguments: List[String], routingKey: String = "yourservice.result.static.totem", WorkType: String = "YOURSERVICE") extends WorkSuccess
    case class YourServiceFailure(status: Boolean, data: JValue, Arguments: List[String], routingKey: String = "", WorkType: String = "YOURSERVICE") extends WorkFailure
    
    
    object YourServiceREST {
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
is instantiated with the result as parameter and returned.

.. warning::
    
    The 2 generic cases at the end of the map should be there in any case to
    avoid exceptions.

The ``YourServiceSuccess`` and ``YourServiceFailure`` classes should be self
explanatory. They extend the default interfaces for success and failure and are
very convenient for mapping cases as done in ``driver.scala``, for example.

``YourServiceREST`` object should be self explanatory as well, it defines how the
request address for your service gets constructed from the supplied parameters.

