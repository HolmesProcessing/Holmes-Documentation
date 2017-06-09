Installation
***************

.. _S3: https://aws.amazon.com/documentation/s3/
.. _RIAKCS: http://docs.basho.com/riak/cs/2.1.1/
.. _Cassandra: http://cassandra.apache.org
.. _FakeS3: https://github.com/jubos/fake-s3
.. _Minio: https://github.com/minio/minio
.. _Pithos: http://pithos.io
.. _RIAKInstall: http://docs.basho.com/riak/cs/2.1.1/tutorials/fast-track/local-testing-environment/
.. _SASI: https://github.com/apache/cassandra/blob/trunk/doc/SASI.md
.. _Views: https://www.datastax.com/dev/blog/new-in-cassandra-3-0-materialized-views
.. _CassandraINS: https://www.datastax.com/dev/blog/new-in-cassandra-3-0-materialized-views
.. _CassandraOPS: http://cassandra.apache.org/doc/latest/operating/index.html
.. _AP: https://wiki.apache.org/cassandra/ArchitectureOverview
.. _ONE: https://cassandra.apache.org/doc/latest/tools/nodetool/nodetool.html
.. _TWO: https://docs.datastax.com/en/cassandra/3.0/cassandra/tools/toolsNodetool.html
.. _CassandraREAPER: https://github.com/thelastpickle/cassandra-reaper
.. _NodetoolRepair: https://docs.datastax.com/en/cassandra/3.0/cassandra/tools/toolsRepair.html
.. _NodetoolCompact: https://docs.datastax.com/en/cassandra/3.0/cassandra/tools/toolsCompact.html
.. _Compaction: https://docs.datastax.com/en/cassandra/3.0/cassandra/dml/dmlHowDataMaintain.html#dmlHowDataMaintain__dmlHowDataMaintain
.. _DOAN: http://www.doanduyhai.com/blog/?p=2058
.. _Elastic: https://www.elastic.co/products/elasticsearch
.. _Solr: http://lucene.apache.org/solr/
.. _Spark: https://spark.apache.org

Dependencies
##############


Supported Databases
""""""""""""""""""""""""

Holmes-Storage supports multiple databases and splits them into two categories: Object Stores and Document Stores. Object Stores are responsible for storing the file-based malicious objects collected by the analyst: binary files such as PE32 and ELF, PDFs, HTML code, Zips files etc. Document Stores contain the output of Holmes-Totem and Holmes-Totem-Dynamic Services. This was done to enable users to more easily select their preferred solutions while also allowing the mixing of databases for optimization purposes. In production environments, we strongly recommend using an `S3 <S3_>`_ compatible Object Store, such as `RIAK-CS <RIAKCS_>`_, and a clustered deployment of `Cassandra <Cassandra_>`_ for the Document Stores.

Object Stores
"""""""""""""""""

We support two primary object storage databases.

- S3 compatible
- (Soon) MongoDB Gridfs

There are several tools you can use for implementing Object Stores. Depending on the intended scale of your work with Holmes-Storage, we would recommend:

+----------+-------------+--------------+---------------+
| Framework| Workstation | Mid-scale	| Large-scale   |
+==========+=============+==============+===============+
| AWS	   |     []      |      []      |      [x]      |
+----------+-------------+--------------+---------------+
| RIAK-CS  |     []      |      []      |      [x]      |
+----------+-------------+--------------+---------------+
| LeoFS    |     []      |      []      |      [x]      |
+----------+-------------+--------------+---------------+
| Pithos   |     []      |     [x]      |      []       |
+----------+-------------+--------------+---------------+
| Minio    |     []      |     [x]      |      []       |
+----------+-------------+--------------+---------------+
| Fake-S3  |    [x]      |     []       |      []       |
+----------+-------------+--------------+---------------+

If you want to run Holmes-Storage on your local machine for testing or development purposes, we recommend you use lightweight servers compatible with the Amazon S3 API. This will make the installation and usage of Holmes-Storage faster and more efficient. There are several great options to fulfill this role: `Fake-S3 <FakeS3_>`_, `Minio <Minio_>`_, `Pithos <Pithos_>`_ etc. The above-mentioned frameworks are only suggestions, any S3 compatible storage will do. Check out their documentation to find out which option is more suitable for the work you intend to do.

Document Stores
"""""""""""""""""""

We support two primary object storage databases.

- Cassandra
- MongoDB

We recommend a Cassandra cluster for large deployments.

Configuration
################

RiakCS
""""""""

It is recommended to use `RIAK-CS <RIAKCS_>`_ only for large scale or industry deployments. Follow `this <RIAKInstall_>`_ tutorial for installation of RiakCS.

After successful installation, the user’s access key and secret key are returned in the key_id and key_secret fields respectively. Use these keys to update key and secret your config file ( ``storage.conf.example`` )

Holmes-Storage uses Amazon S3 signature version 4 for authentication. To enable authV4 on riak-cs, add ``{auth_v4_enabled, true}`` to advanced.config file ( should be in ``/riak-cs/etc/``)

Fake-S3
""""""""""""
We recommend `Fake-S3 <FakeS3_>`_ as a simple starting point for exploring the system functionality. Most of the current developers of Holmes-Storage are using `Fake-S3 <FakeS3_>`_ for quick testing purposes. `Minio <Minio_>`_ is also encouraged if you want to engage more with development and do more testing.


Check out the documentation of `Fake-S3 <FakeS3_>`_ on how to install and run it.
Afterwards, go to ``/config/storage.conf`` of Holmes-Storage and set the IP and Port your ObjectStorage server is running on. You can decide whether you want your Holmes client to send HTTP or HTTPS requests to the server through the Secure parameter.


Cassandra
""""""""""""""

Holmes-Storage supports single node or cluster installation of Cassandra version 3.10 and higher. The version requirement is because of the significant improvement in system performance when leveraging the newly introduced `SASIIndex <SASI_>`_ for secondary indexing and `Materialized Views <Views_>`_. We highly recommend deploying Cassandra as a cluster with a minimum of three Cassandra nodes in production environments.

New Cassandra clusters will need to be configured before Cassandra is started for the first time. 
We have highlighted a few of the configuration options that are critical or will improve performance. For additional options, please see the `Cassandra installation guide <CassandraINS_>`_.

To edit these values, please open the Cassandra configuration file in your favorite editor. The Cassandra configuration file is typically located in ``/etc/cassandra/cassandra.yaml``.

The Cassandra "cluster_name" must be set and the same on all nodes. The name you select does not much matter but again it should be identical on all nodes. ``cluster_name: 'Holmes Processing'``

Cassandra 3.x has an improved token allocation algorithm. As such, 256 is not necessary and should be decreased to 64 or 128 tokens. ``num_tokens: 128``

You should populate the "seeds" value with the IP addresses for at least two additional Cassandra nodes. ``seeds: <ip node1>,<ip node2>``

The "listen_address" should be set to the external IP address for the current Cassandra node. ``listen_address: <external ip address>``

Installation
################


Copy the default configuration file located in config/storage.conf.example and change it according to your needs.

.. code-block:: shell

	$ cp storage.conf.example storage.conf

Update the ``storage.conf`` file in config folder and adjust the ports and IPs to point to your cluster nodes. To build the Holmes-Storage, just run

.. code-block:: shell

	$ go build

Setup the database by calling

.. code-block:: shell

	$ ./Holmes-Storage --config <path_to_config> --setup

This will create the configured keyspace if it does not exist yet. For Cassandra, the default keyspace will use the following replication options:

.. code-block:: shell

	{'class': 'NetworkTopologyStrategy', 'dc': '2'}

If you want to change this, you can do so after the setup by connecting with cqlsh and changing it manually. For more information about that we refer to the official documentation of Cassandra (Cassandra Replication Altering Keyspace)  You can also create the keyspace with different replication options before executing the setup and the setup won't overwrite that. The setup will also create the necessary tables and indices.

Setup the object storer by calling:

.. code-block:: shell

	$ ./Holmes-Storage --config <path_to_config> --objSetup

Execute storage by calling:

.. code-block:: shell

	$ ./Holmes-Storage --config <path_to_config>


Best Practices
####################
On a new cluster, Holmes-Storage will setup the database in an optimal way for the average user. However, we recommend Cassandra users to please read the `Cassandra's Operations website <CassandraOPS_>`_ for more information Cassandra best practices. We want to expand on three particular practices that in our experience have been proven to be very meaningful in keeping the database healthy.

Nodetool
""""""""""

Based on the CAP theorem, Cassandra can be classified as an `AP <AP_>`_ database. The cost for strong consistency is higher latency, so the database has its own mechanisms to ensure eventual consistency in the system. However, human intervention is often necessary. It is critical that the Cassandra cluster is regularly repaired using the ``nodetool repair`` and nodetool compact command. The following documentations `1 <ONE_>`_, `2 <TWO_>`_ give an overview of the nodetool functionality. We suggest exploring the `Cassandra-Reaper tool <CassandraREAPER_>`_ as a potential way to automate the repair process.

Nodetool Repair
"""""""""""""""""

The purpose of this `command <NodetoolRepair_>`_ is to enforce consistency in the tables across the cluster. We recommend that this is executed on every node, one at a time, at least once a week.

Nodetool Compact
""""""""""""""""""

This is another important maintenance `command <NodetoolCompact_>`_. Cassandra has its own methodology for storing and deleting data, which requires `compaction <Compaction_>`_ to take place in regular intervals in order to save space and maintain efficiency. For more details follow the links above to learn more about Cassandra.

Indexing
""""""""""""""""""

Holmes-Storage uses `SASIIndex <SASI_>`_ for indexing the Cassandra database. This indexing allows for querying of large datasets with minimal overhead. When leveraging Cassandra, most of the Holmes Processing tools will automatically use SASI indexes for speed improvements. Power users wishing to learn more about how to utilize these indexes should please visit the excellent blog post by `Doan DyuHai <DOAN_>`_.

However, while SASI is powerful, it is not meant to be a replacement for advanced search and aggregation engines like `Solr <Solr_>`_, `Elasticsearch <Elastic_>`_, or leveraging `Spark <Spark_>`_. Additionally, Holmes Storage by default does not implement SASI on the table for storing the results of TOTEM Services (results.results). This is because indexing this field can increase storage costs by approximately 40% on standard deployments. If you still wish to leverage SASI on results.results, the following Cassandra command will provide a sane level of indexing.

SASI indexing of TOTEM Service results. WARNING: this will greatly increase storage requirement:

.. code-block:: SQL

	CREATE CUSTOM INDEX results_results_idx ON holmes_testing.results (results) 
	USING 'org.apache.cassandra.index.sasi.SASIIndex' 
	WITH OPTIONS = {
		'analyzed' : 'true', 
		'analyzer_class' : 'org.apache.cassandra.index.sasi.analyzer.StandardAnalyzer', 
		'tokenization_enable_stemming' : 'false', 
		'tokenization_locale' : 'en', 
		'tokenization_normalize_lowercase' : 'true', 
		'tokenization_skip_stop_words' : 'true',
		'max_compaction_flush_memory_in_mb': '512'
	};
