Overview
***************

Holmes-Storage is responsible for managing the interaction of Holmes Processing with the database backends. At its core, Holmes-Storage organizes the information contained in Holmes Processing and provides a RESTful and AMQP interface for accessing the data. Additionally, Holmes-Storage provides an abstraction layer between the specific database types. This allows a Holmes Processing system to change database types and combine different databases together for optimization.

When running, Holmes-Storage will:

- Automatically fetch the analysis results from Holmes-Totem and Holmes-Totem-Dynamic over AMQP for storage

- Support the submission of objects via a RESTful API
- Support the retrieval of results, raw objects, object meta-data, and object submission information via a RESTful API


We have designed Holmes-Storage to operate as a reference implementation. In doing so, we have optimized the system to seamlessly plug into other parts of Holmes Processing and optimized the storage of information for generic machine learning algorithms and queries through a web frontend. Furthermore, we have separated the storage of binary blobs and textual data in order to better handle how data is stored and compressed. As such, Holmes-Storage will store file based objects (i.e. ELF, PE32, PDF, etc) in an S3 compatible backend and the meta information of the objects and results from the analysis in Cassandra. With respect to non-file type objects, these are purely stored in Cassandra. In our internal production systems, this scheme has supported 10s of million of objects along with the results from associated Totem and Totem-Dynamic Services with minimal effort. However, as with any enterprise system, customization will be required to improve the performance for custom use cases. Anyway, we hope this serves you well or at least helps guide you in developing custom Holmes-Storage Planners.
