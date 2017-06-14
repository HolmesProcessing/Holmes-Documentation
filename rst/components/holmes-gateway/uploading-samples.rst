Uploading Samples
**********************


In order to upload samples to storage, the user sends an HTTPS-encrypted POST request to /samples/ of the Holmes-Gateway. Gateway will forward every request for this URI directly to storage. If storage signals a success, Gateway will immediately issue a tasking-request for the new samples, if the configuration-option AutoTasks is not empty.

You can use Holmes-Toolbox for this purpose. Just replace the storage-URI with the URI of Gateway. Also, make sure, your SSL-Certificate is accepted. You can do so either by adding it to your system's certificate store or by using the command-line option --insecure. The following command uploads all files from the directory $dir to the Gateway instance residing at 127.0.0.1:8090 using 5 threads.


.. code-block:: shell

	./Holmes-Toolbox --gateway https://127.0.0.1:8090 --user test --pw test --dir $dir --src foo --comment something --workers 5 --insecure
