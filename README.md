# batch-etl-vs-elt-gcp

## ETL (Extract, Transform, Load): Dataproc

Dataproc is the Google Cloud solution offering based on Apache Spark framework. It creates instances in a gke cluster with pods based on official [Apache Spark releases](https://cloud.google.com/dataproc/docs/concepts/versioning/dataproc-version-clusters#supported_dataproc_versions).

Dataproc jobs can be created from gcloud cli or from Google Cloud console

```bash
export CLOUDSDK_CORE_PROJECT=my-project
export GCP_PROJECT=my-project
export GCP_REGION=my-region
export CLUSTER_NAME=my-cluster

```

```bash
gcloud services enable dataproc.googleapis.com
gsutil mb -l $GCP_REGION gs://$GCP_PROJECT-wordcount
gcloud dataproc clusters create $CLUSTER_NAME --region=$GCP_REGION --zone=$GCP_REGION-b --single-node --master-machine-type=n1-standard-2
gsutil cp -r "gs://acg-gcp-labs-resources/data-engineer/dataproc/*" .
gcloud dataproc jobs submit pyspark wordcount.py --cluster=$CLUSTER_NAME --region=$GCP_REGION -- gs://acg-gcp-labs-resources/data-engineer/dataproc/romeoandjuliet.txt gs://$GCP_PROJECT-wordcount/output/
gsutil cp -r "gs://$GCP_PROJECT-wordcount/output/*" .
gcloud dataproc clusters delete $CLUSTER_NAME --region=$GCP_REGION 
```

## Customize Dataproc Cluster

You can customize you cluster dependencies by:

- Modifying cluster environment
    - Creating a cluster with [a custom conda environment](https://cloud.google.com/dataproc/docs/tutorials/python-configuration#conda-related_cluster_properties) with all the dependencies
    - Or by [adding properties](https://cloud.google.com/dataproc/docs/tutorials/python-configuration#image_version_20) to cluster creation to install the left depenendencies.

- Creating a dataproc [custom image](https://cloud.google.com/dataproc/docs/guides/dataproc-images#use_a_custom_image) from [Dataproc official base images](https://cloud.google.com/dataproc/docs/concepts/versioning/dataproc-version-clusters#supported_dataproc_versions)

*[See docs](https://cloud.google.com/dataproc/docs/guides/dataproc-images#use_a_custom_image) for Scala jobs that deps (eg. using BigQuery jdbc connector)* 

# Dataproc Serverless

Dataproc Serverless for Spark runs workloads within [docker images](https://cloud.google.com/dataproc-serverless/docs/guides/custom-containers), where at runtime, Dataproc Spark will be mounted on. Additional dependencies can be added if they associate with the official [Dataproc Apache Spark runtime versions](https://cloud.google.com/dataproc-serverless/docs/concepts/versions/spark-runtime-versions).


# Dataproc Workflow Templates

Workflow templates can also be used to [submit spark/hadoop job workflows in two flavours](https://cloud.google.com/dataproc/docs/concepts/workflows/overview#kinds_of_workflow_templates). Either in a managed cluster where the workflow runs on an ephemeral cluster until it ends (https://cloud.google.com/dataproc/docs/concepts/workflows/workflow-parameters#parameterizing_a_workflow_template), or on an already [existing cluster](https://cloud.google.com/dataproc/docs/concepts/workflows/workflow-parameters#parameterizing_a_workflow_template)

# Workflow Scheduling

Workflow templates can be schedule/triggered in [several ways](https://cloud.google.com/dataproc/docs/concepts/workflows/workflow-schedule-solutions), ie. via [Cloud Functions](https://cloud.google.com/dataproc/docs/tutorials/workflow-function) based on events, via [Cloud Scheduler](https://cloud.google.com/dataproc/docs/tutorials/workflow-scheduler) or in Cloud Composer ([from an Airflow DAG](https://cloud.google.com/dataproc/docs/tutorials/workflow-composer))


## ELT (Extract, Load, Transform): BigQuery

BigQuery is the DataWarehouse product of choice for Google Cloud Platform.

BigQuery can handle remote SQL queries to multple Google Data products, such as Google Drive, Google Analytics, Cloud Storage, Cloud SQL, etc.

It also offers integration with BigLake. BigLake offers access to direct access to different storage Data Lakes (across multiple projects and regions) via Dataplex. 
In Dataplex, you can govern data lake access, structure and joining different lakes, automate export to BigQuery projects, and without the need for manual refreshing or creating external tables anymore. Currently it offers BigQuery and Cloud Storage mode access to look up.

There's also native integration with SQL Transformation tools such as DBT and Dataform. Where you can create different SQL pipelines.

DBT is the standard SQL transformation tool. It is written in python which is available on pypi and conda repositories. It allows templating SQL code via jinja injection. In DBT every SQL creates a table, you organize SQL in folders which represents a model (in BigQuery it stores table in a Dataset). It also provides validation testing and documentation.

DBT can be run via CLI or via request to a FastAPI DBT server [which is also opensource](https://github.com/dbt-labs/dbt-server)

DBT CLI also offers dbt docs command which spins up a gunicorn server for lookup & documentation.

DBT is opensource but it also offers an enterprise solution (SaaS product). 

* DBT community version has a list of official dialects it can connect to, but also has a big community which provides non official libraries and plugins.
* DBT cloud version provides an IDE, job scheduler, IDE, git integration, alert & monitoring, and customer support.

Dataform is the Google official alternative to DBT. It is a managed service integrated now in Google Cloud console, it is written in javascript which is available on npm repositories. It integrates with different version control systems ie. GitHub, Gitlab, provides alerting & monitoring. And provides schedules via [Cloud Scheduler](https://cloud.google.com/dataform/docs/schedule-executions-workflows) and [Cloud Composer](https://cloud.google.com/dataform/docs/schedule-executions-composer) making builds from git commits.

