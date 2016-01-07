Writing a Service
=================

.. toctree::
    
    writing_a_service/totem-conf
    writing_a_service/driver-scala
    writing_a_service/yourservice-rest-scala
    writing_a_service/implementation-python-docker


For Totem to recognize your service, references need to be inserted into two files:

1. ``config/totem.conf``
2. ``src/main/scala/org/novetta/zoo/driver/driver.scala``

And one file specific to your service needs to be added:

3. ``src/main/scala/org/novetta/zoo/services/yourservice/YourServiceREST.scala``

Where *"src/main/scala/org/novetta/zoo/services/yourservice/"* is the folder that
all your services files go into.

And of course, your service needs to be written. This is covered in subsection 4.

.. note::
    
    It is necessary to modify all of these files because of the way Totem is
    designed for maximum scalability and distributivity.
    
    Further this section assumes your service's name is ``YourService``

.. warning::
    
    Try to stick to the naming convention (uppercase/lowercase/camelcase were
    appropriate) to avoid confusion!
    
    If you haven't yet, you should make yourself familiar with Scala in order to
    understand what's happening in the Scala files.
