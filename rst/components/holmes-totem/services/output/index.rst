Services-Output
*****************

HTTP Error Codes
##################
Holmes Services attempts to return appropriate HTTP Error Codes when a requested for analysis. This codes will be used by the watchdog to manage results.

The Error codes table follow this pattern:

200 (OK) - Service returns analysis results in JSON format.
4XX (client errors) 
5XX (server errors) from Service logic section
6XX - Custom Errors

If the service returns an error, it returns an empty JSON format and corresponding error code.

List of Errors codes:

These are general error codes:

+--------+----------------------------+---------------------------------------------------------+
| Code   |         Message            |             Description                                 |
+--------+----------------------------+---------------------------------------------------------+
| 400    |      Bad request           |Trying use new endpoint other than `/`and `/analyse`     |
+--------+----------------------------+---------------------------------------------------------+
| 401    |      Unauthorised          |  Invalid authentication                                 |
+--------+----------------------------+---------------------------------------------------------+
| 404    |      Not Found             |  The resource cannot be found.                          |
+--------+----------------------------+---------------------------------------------------------+
| 500    |      Internal Server Error |When there is problem in the service logic               |
+--------+----------------------------+---------------------------------------------------------+
| 503    |      Cannot Create JSON    | JSON encoding error                                     |
+--------+----------------------------+---------------------------------------------------------+




Apart from these, you can define your own custom error codes. We would suggest doing so starting with 600. (refer to PEMETA service which uses error codes to report error to TOTEM)

+--------+----------------------+---------------------------------------------------+
| Code   |     Message          |         Description                               |
+--------+----------------------+---------------------------------------------------+
| 601    |  ALLOCATION FAILURE  |   Cannot allocate memory to LIBPE function        |
+--------+----------------------+---------------------------------------------------+

All the services by default should run on port 8080 so that docker compose can port forward it to a specified port.

These error codes will notify watchdog about the behavior of the service and based on these error codes, watchdog can manage results more intelligently

Based on the error codes, the behavior of TOTEM changes which will be discussed in Holmes-TOTEM sections.


JSON
=======

The mime type of the data returned by Totem Service is JSON. The output should be meaningful so that it it becomes easier for the interrogation planner to produce knowledge from the results. 

The output is a typical json (*an example for json.org*)

.. code-block:: json

	{"menu": {
	  "id": "file",
	  "value": "File",
	  "popup": {
	    "menuitem": [
	      {"value": "New", "onclick": "CreateNewDoc()"},
	      {"value": "Open", "onclick": "OpenDoc()"},
	      {"value": "Close", "onclick": "CloseDoc()"}
	    ]
	  }
	}}


.. warning::

	All the first level keys of the output JSON should be property of the file that the Service is supposed to analysed. If the a particular property is absent in the file, then the value for the key should empty.



