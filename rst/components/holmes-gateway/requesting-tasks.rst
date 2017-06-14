Requesting Tasks
*********************

In order to request a task, a user sends an HTTPS-request (GET/POST) to Gateway containing the following form fields:

username: The user's login name
password: The user's password
task: The task which should be executed in JSON-form, as described below.
A task consists of the following attributes:

primaryURI: The user enters only the sha256-sum here, as Gateway will prepend this with the URI to its version of Holmes-Storage
secondaryURI (optional): A secondary URI to the sample, if the primaryURI isn't found
filename: The name of the sample
tasks: All the tasks that should be executed, as a dictionary
tags: A list of tags associated with this task
attempts: The number of attempts. Should be zero
source: The source this sample belongs to. The executing organization is chosen mainly based on this value
download: A boolean specifying, whether totem has to download the file given as PrimaryURI
For this purpose, any web browser or command line utility can be used. The following demonstrates an exemplary evocation using CURL. The --insecure parameter is used, to disable certificate checking.

.. code-block:: shell


	curl --data 'username=test&password=test&task=[{"primaryURI":"3a12f43eeb0c45d241a8f447d4661d9746d6ea35990953334f5ec675f60e36c5","secondaryURI":"","filename":"myfile","tasks" :{"PEID":[],"YARA":[]},"tags":["test1"],"attempts":0,"source":"src1","download":true}]' --insecure https://localhost:8090/task/


Alternatively, it is possible to use Holmes-Toolbox for this task, as well. First a file must be prepared containing a line with the sha256-sum, the filename, and the source (separated by single spaces) for each sample.

.. code-block:: shell


	./Holmes-Toolbox --gateway https://127.0.0.1:8090 --tasking --file sampleFile --user test --pw test --tasks '{"PEID":[], "YARA":[]}' --tags '["mytag"]' --comment 'mycomment' --insecure

If no error occurred, nothing or an empty list will be returned. Otherwise, a list containing the faulty tasks, as well as a description of the errors will be returned.

You can also use the Web-Interface by opening the file submit_task.html in your browser. However, you will need to create an exception for the certificate by visiting the website of the Gateway manually, before you can use the web interface.
