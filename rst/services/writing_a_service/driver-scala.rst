driver.scala
--------------

In this file, there's two things to do.
The one thing is importing your services work and success classes.
These classes don't exist yet and will be created later.

.. code-block:: scala
    
    import org.novetta.zoo.services.yourservice{YourServiceSuccess, YourServiceWork}

The other thing to do is adjust the ``TotemicEncoding`` class. There the appropriate
cases need to be inserted into all 3 functions.

.. code-block:: scala
    
    def GeneratePartial(work: String): String = {
      work match {
        
        ...
        
        case "YOURSERVICE" => Random.shuffle(services.getOrElse("yourservice", List())).head
        
        ...
        
      }
    }

.. code-block:: scala
    
    def enumerateWork(key: Long, filename: String, workToDo: Map[String, List[String]]): List[TaskedWork] = {
      val w = workToDo.map({
        case ("YOURSERVICE", li: List[String]) =>
          
          ...
          
          YourServiceWork(key, filename, 60, "YOURSERVICE", GeneratePartial("YOURSERVICE"), li)
          
          ...
          
      }).collect({
        case x: TaskedWork => x
      })
      w.toList
    }

.. code-block:: scala
    
    def workRoutingKey(work: WorkResult): String = {
      work match {
        
        ...
        
        case x: YourServiceSuccess => "yourservice.result.static.zoo"
        
        ...
        
      }
    }