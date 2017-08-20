Services-Output
*****************

HTTP Error Codes
##################
Holmes Services attempts to return appropriate HTTP error code when a requested for analysis. This codes will be used by the watchdog to manage results.

The Error codes table follow this pattern:

- 200 (OK) - Service returns analysis results in JSON format.
- 4XX (client errors)
- 5XX (server errors) from Service logic section
- 6XX - Custom Errors


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


These error codes will notify watchdog about the behavior of the service and based on these error codes, watchdog can manage results more intelligently

Based on the error codes, the behavior of TOTEM changes which will be discussed in Holmes-TOTEM sections.


JSON
=======

The mime type of the data returned by Totem Service is JSON. The output should be meaningful so that it it becomes easier for the interrogation planner to produce knowledge from the results. 

The output is a typical json (* part of output for returned by pemeta*)

.. code-block:: json

	{
		"Exports": {},
		"Resources": "",
		"Imports": [
			{
			"DllName": "SHELL32.dll",
				"Functions": [
					"ShellExecuteA",
					"FindExecutableA"
				]
			}
		]
	}

.. note::

	All the first level keys of the output JSON should be property of the file that the Service is supposed to analysed. If the a particular property is absent in the file, then the value for the key should empty.

In the above snippet, **Imports** is a property of PE file. Imports is a list of DLLs. Each item of the list contains Dll Name and functions present in that DLL

Also, **Exports** value is an empty struct. This means that the service has analysed Exports, but could not find any results.
