## Zero Shot Cardinality Training

This repository contains the infrastructure to produce training datasets for
cardinality estimation. This process is a multi-step process:

* Create down sampled versions of databases, if needed
* Calculate table (collect table/column information, calculate statistics table/column statistics)
* Generate training sql queries
* Run queries to collect actual cardinalities
* Create training datasets (querygraps)

![CardBench](training_datasets/figures/cardbench.png)


As the cost of running this pipeline is significant we plan to release the final
product (the training dataset) in addition to the code that performs all the steps.

The training datasets and their documentation can be found in the training_datasets directory.

<span style="color:red">
We are releasing the code to create the CardBench training datasets incrementally.
</span>


## General Information & Setup

Any statistic and information that is collected or calculated is stored in a set of database tables. The ``statistics_sql_tables_definition.sql`` script generates all the necessary tables. After the tables are created 
please update the ``configuration.py`` file with the ids of the tables. 

The code queries the tables of the "data" database (these are the tables for which we collect cardinalities) and the "metadata" (or statistics) database (these tables store the statistics we calculate). These two databases can be stored in separate systems.

The code was initially designed to work with Big Query as the database backend for both the data and metadata databases. In ``database_connector.py`` we provide an extensible database connector that can be extended to work with 
any database. Changes in the rest of the code will be needed to support the functionality needed to calculate some statistics (for example percentiles requires a percientile SQL function or discovering the schema of a table requires calling the database specific API that returns the column names and types of a table.)

### Calculate Statistics

The first step is to calculate statistics and collect information about the databases. 
The ``calculcate_and_write_statistics.py`` file runs this step. The ``calculate_statistics_library.py`` contains the relevant code.


<span style="color:red">
As this step requires some effort to replicate, we will also release the collected statistics in this repository.
</span>

### Generate Queries
To generate queries we use [this previously released generator](https://github.com/DataManagementLab/zero-shot-cost-estimation/tree/main). The code is slightly modified to accommodate our needs (see ``generate_queries_library/query_generator.py``)

The ``geenerate_and_write_queries.py`` generate queries and writes the queries in a file one at each line. The parameters of the generator are defined in ``geenerate_and_write_queries.py``. We call a file of queries a workload, each workload is identified by an integer workload id. The worload id and the parameters used to generate a workload are stored in the configuration.WORKLOAD_DEFINITION_TABLE.

The query generator takes as input a set of json files that contain the schema, column and string column statistics of the database for which queries will be generated. The jsons can be created from the collected statistics calculated by ``calculate_statistics.py``. To generate the jsons please use ``write_dataset_statistics_to_json_files.py``. To ease experimentation we include the generated jsons in ``generate_queries_library/dataset_statistics_jsons``.

The json files are stored in a directory specified by configuration.DIRECTORY_PATH_JSON_FILES and the workload query files are stored in a directory specified by configuration.DIRECTORY_PATH_QUERY_FILES.

### Execute Queries
``run_queries.py`` takes in as argument a workload id, the integer identifier of workload (a file of queries) that was generated by the Generate Queries code. The queries of the workload are read and executed. The result of the query execution is stored in the configuration.QUERY_RUN_INFORMATION_TABLE database table. The result stores for each query the sql string and the cardinality. Further to identify each query the workload id as well as the query run id are stored in the table. The query run id is a integer identifier of the run. A new run is created every time a workload is run. Therefore a workload that is run multiple times will have create multiple query runs.

### Generate Annotated Query Graphs
TODO

