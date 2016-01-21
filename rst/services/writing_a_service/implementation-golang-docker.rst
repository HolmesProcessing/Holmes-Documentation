.. _implementation-service-go-docker:

Service Implementation: Go with Docker
--------------------------------------------

.. _Docker: http://www.docker.com
.. _Golang: https://golang.org/

This tutorial will show how to utilize the **Go Programming Language**
to write the actual service. (\> Golang_)

The same considerations that apply to the an implementation in Python also apply
to one written in **Go**. (\> :ref:`Service with Python<implementation-service-python-docker>`)



Install Dependencies
^^^^^^^^^^^^^^^^^^^^^^

.. note::
    
    Please refer to the Dockerfile below to see how to use it with Docker.


First of all, **Go** is required:

.. code-block:: shell

    apt-get install golang

Whilst **Go** natively has a very good webserver, it lacks a good
router. More specific, the router lacks the ability to parse request URIs
directly into variables.
In our example we will use httprouter:

.. code-block:: shell

    go get github.com/julienschmidt/httprouter

Further we need a way to parse a config file:

.. code-block:: shell

    go get gopkg.in/ini.v1

If you have any further dependencies, they need to be installed in this step as well.
(For example any additional frameworks)



Dockerfile
^^^^^^^^^^^^

If you put all of that into a Dockerfile, the result might look like this:

.. code-block:: shell
    
    # choose the operating system image to base of, refer to docker.com for available images
    FROM golang

    # install what you need here
    RUN go get github.com/julienschmidt/httprouter
    RUN go get gopkg.in/ini.v1

    # create a folder to contain your service's files
    RUN mkdir -p /service
    
    # add all files relevant for running your service to your container
    ADD service.go /service
    RUN go build -o /service/service.run /service/service.go

    # create a new user with limited access
    RUN useradd -s /bin/bash service
    RUN chown -R service /service
    
    # switch user and switch to work dir    
    USER service
    WORKDIR /service

    # expose our container on some port
    EXPOSE 7715
    
    # define our command to run if we start our container
    ENTRYPOINT /service/service.run

It is important to think about the ordering the commands have in the Dockerfile,
as that can speed up or slow down the container build process heavily.
Stuff that does not need to be done on every build should go to the front of the
Dockerfile, stuff that changes should go towards the end of the file.

(Docker cashes previous build steps and if nothing changes, those build steps
will be reused on the next build, speeding it up by a lot, especially when
installing python like in this Dockerfile)



The service.go
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Lets start with some code right away.

.. code-block:: go

    package main

    import (
        "encoding/json"
        "fmt"
        "io/ioutil"
        "net/http"
        "os"
        
        "github.com/julienschmidt/httprouter"
        "gopkg.in/ini.v1"
    )

These were all the imports required for this tutorial. If you have any
further dependencies, those go in this import section.

-------------------------

Every service requires configuration parameters, these are best stored in a config
file. Since INI style config files are quite common we will introduce you to
parsing those with **Go**.

The parsed config is best stored in a global struct and having some helper
functions will come in handy:

.. code-block:: go

    var (
        Config struct {
            __file__        *ini.File
            
            ServiceName     string
            ServiceVersion  string
            Host            string
            Port            string
            
            Copyright       string
            Description     string
        }
    )

    func config(key string) string{
        if Config.__file__.Section("").HasKey(key) {
            return Config.__file__.Section("").Key(key).String()
        }
        return ""
    }

    func fetch_config(cfg *ini.File) {
        Config.__file__         = cfg
        Config.ServiceName      = config("ServiceName")
        Config.ServiceVersion   = config("ServiceVersion")
        Config.Host             = config("Host")
        Config.Port             = config("Port")
        Config.Copyright        = config("Copyright")
        Config.Description      = config("Description")
    }

If you want to load the copyright or description from a file, the following code
snippet could be used inside of the fetch_config function:

.. code-block:: go

    func fetch_config(cfg *ini.File) {
        ...
        data, err := ioutil.ReadFile(Config.Copyright)
        if (err != nil) {
            fmt.Println("An error occured reading the copyright file:", err)
            Config.Copyright = "Error loading copyright information."
        } else {
            Config.Copyright = string(data)
        }

Obviously we now need a main function that reads the config file and hands it to
the *mapconfig* function:

.. code-block:: go

    func main() {
        // if called with parameters, assume the first is the config path
        // otherwise use the default path
        var configpath string = "./service.conf"
        if (len(os.Args) > 1) {
            configpath = os.Args[1]
        }
        
        cfg, err := ini.Load(configpath)
        if (err != nil) {
            fmt.Println("An error occured reading the config file:", err)
        } else {
            fetch_config(cfg)
        }
    }

An example configuration file might look like this:

.. code-block:: ini

    ServiceName     =   HelloWorldService
    ServiceVersion  =   v2.0
    CopyrightFile   =   ./COPYRIGHT
    DescriptionFile =   ./DESCRIPTION
    Host            =   localhost
    Port            =   7715

The config variable ``Port`` is required in our setup, ``Host`` can be left
blank which results in binding on the localhost. ``ServiceName``,
``ServiceVersion``, ``Description`` and ``Copyright`` are required to provide
sufficient information about the service if queried for it.

----------------------------------

Now that the config is handled, what is missing for a fully functional basic
service is a webserver to accept requests, bound to the defined host and port.
``http`` together with ``httprouter`` makes creating a webserver as easy as:

.. code-block:: go

    func main() {
        ...
        } else {
            fetch_config(cfg)
            
            router := httprouter.New()
            router.GET("/info", info_handler)
            router.GET("/analysis/:file", handler)
            http.ListenAndServe(Config.Host+":"+Config.Port, router)
        }
    }

Note that it is necessary to specify the unique pathname /analysis, otherwise
/info will collide with the /:file wildcard. (httprouter limitation)

.. code-block:: go

    func info_handler(w http.ResponseWriter, r *http.Request, ps httprouter.Params)
    func handler(w http.ResponseWriter, r *http.Request, ps httprouter.Params)

The info_handler delivers a html page containing name, description, possibly
copyright and licence information.
The handler function should write a valid JSON string.
Both utilize ``fmt.Fprint(w, result)`` to write the
result.
It could looke like this:

.. code-block:: go

    func info_handler(w http.ResponseWriter, r *http.Request, ps httprouter.Params) {
        info := "<p>Service: "+Config.ServiceName+"<br>"
        info += "Version: "+Config.ServiceVersion+"</p>"
        info += "<p>"+Config.Description+"</p>"
        info += "<p>"+Config.Copyright+"</p>"
        fmt.Fprint(w, info)
    }

    func handler(w http.ResponseWriter, r *http.Request, params httprouter.Params) {
        // assemble filename and check if exists
        filename := "/tmp/" + params.ByName("file")
        if _, err := os.Stat(filename); os.IsNotExist(err) {
            http.NotFound(w, r)
            return
        }
        // get some result
        result, err := serviceWorkFunction(filename)
        if (err != nil) {
            w.WriteHeader(http.StatusInternalServerError)
            fmt.Fprint(w, "500 - "+err.Error())
            return
        }
        // write result
        fmt.Fprint(w, result)
    }

In this case the ``serviceWorkFunction`` would do all of the hard work.
This could consist of several analysis tasks and maybe even a http request to
gather additional data.
Here it just simply returns the filename and some boolean value:

.. code-block:: go

    type Result struct {
        StringData  string
        BoolData    bool
    }

    func serviceWorkFunction(filename string) (string, error){
        // generate some result
        result_obj := Result{filename, true}
        // convert to json
        result, err := json.Marshal(result_obj)
        if (err != nil) {
            return "", err
        }
        return string(result), nil
    }

Notice how the result is converted to a valid JSON string from a struct.
For the request url ``localhost:7715/analysis/special.exe`` the output could
look like this:

.. code-block:: json
    
    {"StringData":"/tmp/special.exe","BoolData":true}

----------------------------------

.. _below:

