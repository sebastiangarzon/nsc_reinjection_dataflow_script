# iCarrier Data Dataflow scripts

## Google Dataflow params

**job_name**: Dataflow job name \
**region**: Dataflow execution region \
**zone**: Dataflow execution zone \
**runner**: Runner type (DataflowRunner or DirectRunner) \
**project**: Dataflow exection project \
**staging_location**: Google Storage path to job staging directory \
**temp_location**: Google Storage path to job temproal directory \
**requirements**: Python external libraries installation requiriments file. \
**save_main_session**: Dataflow flag to accept external imports \
**network**: Dataflow job execution network \
**subnetwork**: Dataflow job execution subnetwork

## Join Big Query key-value columns tables & save table in Google Storage


### Script params

**bq_dataset_in**: Input tables dataset \
**bq_dataset_out**: Output table dataset \
**bq_input_query_value**: Input metadata tables BigQuery SELECT to obtain list of tables to be considered as input. \
**bq_input_query_location**: Input metadata SELECT table location (EU, US). \
**bq_input_query_key_col**: Input metadata SELECT table attribute key value column name (SELECT response column name). \
**bq_input_query_name_col**: Input metadata SELECT table name column name (SELECT response column name). \
**bq_tabl_out**: Output table name \
**key_name**: Column name of column to be considered as data key \
**value_name**: Column name of column to be considered as data value \
**output_path**: Output Google Storage path to save output table data

### Example

```

python -m df_bigquery_table_join \
	--job_name bq_master_table_join  \
	--region europe-west1  \
	--zone europe-west1-b  \
	--runner DataflowRunner \    
	--project sirius-143909 \
	--staging_location gs://sirius-dataflow-icarrier-data-tmp/staging \    
	--temp_location gs://sirius-dataflow-icarrier-data-tmp/temp \
	--requirements_file requirements.txt \
	--bq_dataset_in sirius_odatos_profiling \
	--bq_dataset_out sirius_odatos_profiling \
	--bq_input_query_value "SELECT t2.table_name, SUBSTR(t2.table_name, 22, length(t2.table_name)) as table_key FROM sirius_odatos_profiling.INFORMATION_SCHEMA.TABLES AS t2 WHERE t2.table_name LIKE 'icarrier_data_msisdn_%'" \
	--bq_input_query_location EU \
	--bq_input_query_key_col table_key \
	--bq_input_query_name_col table_name \
	--bq_table_out icarrier_data_master_msisdn_profile \
	--key_name msisdn \
	--value_name value \
	--output_path gs://sirius-dataflow-icarrier-data-tmp/data/master

```

## Google Storage to redis

### Script params

**input_file**: Input file Google Storage path \
**redis_conn**: Output redis connection list with format: ip1:port1,ip2:port2,ip3:port3 ... \
**key_name**: Name of the key that will be considered as the Redis key \
**key_prefix**: Redis key prefix to add to key_name value \
**key_encrypted**: Flag that indicates whether key value is encrypted or not \
**balancing**: Flag that indicates whether data insertion is balanced by key to redis_conn hosts or full data is inserted in every host.

### Example

```
python -m df_gs_to_redis  \
	--job_name gs_master_to_redis \
	--runner DataflowRunner \
	--region us-west1  \  
	--project sirius-143909 \  
	--staging_location gs://sirius-dataflow-icarrier-data-tmp-us/staging \    
	--temp_location gs://sirius-dataflow-icarrier-data-tmp-us/temp \
	--requirements_file requirements.txt \
	--input_file gs://sirius-dataflow-icarrier-data-tmp-us/data/master* \
	--redis_conn 10.27.73.10:6379,10.27.73.11:6379 \
	--key_name msisdn \
	--key_prefix msisdn:profile: \
	--key_encrypted true \
	--balancing true \
	--save_main_session true \
	--network  https://www.googleapis.com/compute/v1/projects/sirius-143909/global/networks/odatos \
	--subnetwork https://www.googleapis.com/compute/v1/projects/sirius-143909/regions/us-west1/subnetworks/redis

```
