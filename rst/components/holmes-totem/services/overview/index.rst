Overview
**************
Holmes Totem Static Services perform analysis on a given object and provide a result.

The Services in Holmes Processing are designed as microservices. The Planners in Holmes are the central orchestration engine for ensuring that the Services performing the given task at an optimal level. Under this umbrella, Holmes Processing uses Holmes Totem Services which is built on the ideas of micro services. These are pluggable services that we can use or remove easily according to the needs.

Holmes Totem Services are kind of web servers that communicate with Totem via HTTP protocol. They do analysis using a particular malware analysis library (we call it analyzer library) and returns result. Totem sends a request for analysis and Services responds with final result of analysis along with HTTP error code. This final result will eventually gets stored in Holmes Storage in JSON format.

Each Service runs in an isolated environment with the illusion that they are the only process running in the system. We use Docker containers to isolate a Totem Service. Each Totem-Services are loosely coupled and does analysis individually. They don't depend on other Totem-Service to create the final output. Also, there will not be any interprocess communication between Totem Services. The reason for this design choice is that this improves fault-tolerance and provides great flexibility.



Analyser Library
***********************

Analyser library is a module to parse and work with a particular file type.

These modules parse the file and get all the information/properties of the file type. Later Interrogation planner will infer and score the sample. 

The Totem Service takes a sample through GET request, and gives it to the analyser library. The library will parse the file and provides an output. The Service then uses this output and properly serialize the data and returns the result to TOTEM.

The analyzer library can also be any malware analysis tool. We can either directly import the library or directly parse the command line output.

If the analyzer library is in Python, we would suggest creating a Totem Service in Python. If the analyser library is in C or if the tool is command line, then we would suggest using Go
