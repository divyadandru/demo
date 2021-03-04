# containers/row_drop_automation
#### Row Drop Container- Based on the input parsed arguments, the row drop container downloads to local the file from gcs input path, drops the top and/or the bottom rows of the file, and uploads the modified file to the gcs output path.

**row drop container path** = "./cp-saa-external-sales/containers/row_drop_automation"


| s. no | parsed argument            | type       | required| help                                                           | default                                   |
| ----- | ---------------------------| -----------| --------| ---------------------------------------------------------------|-------------------------------------------
| 1     | dag_id                     | str        | True    | "dag id to query data from datastore"                          |                                           
| 2     | run_id                     | str        | True    | "run id of the dag"                                            |                     
| 3     | output_bucket_name         | str        | True    | "output bucket name to store the modified file"                |
| 4     | drop_top                   | str        | True    | "number of rows to drop from the top of the file"              |                  
| 5     | drop_bot                   | str        | True    | "number of rows to drop from the bottom of the file"           |
| 6     | airflow_task_id            | str        | False   | "airflow task id"                                              |
| 7     | input_file_path            | str        | False   | "absolute path to csv report"                                  |                  
| 8     | input_bucket_name          | str        | False   | "input bucket name"                                            |
| 9     | output_path                | str        | False   | "absolute path where the modified file will be stored"         |
| 10    | input_datastore_parameters | json.loads | False   | "parameters required to retrieve from data store"              |                  
| 11    | additional_dag_params      | json.loads | False   | "additional dag params"                                        |
| 12    | output_path_params         | json.loads | False   | "parameters to build output path where file will be stored"    |
| 13    | file_name_params           | json.loads | False   | parameters to build output file name where file will be stored"|                  
| 14    | input_file_format_details  | json.loads | False   | "parameters related to input file format"                      | json.dumps(constant.input_file_format_details)**
| 15    | report_start_date          | str        | False   | "report start date"                                            | 
| 16    | report_end_date            | str        | False   | "report end date"                                              |
| 15    | dag_start_date             | str        | False   | "date on which the dag run"                                    | datetime.datetime.now().strftime("%Y-%m-%d")
| 16    | skip_datastore             | str        | False   | "Whether to use data store or not"                             | False

** Default value of args.input_file_format_details
```bash
input_file_format_details = {
        'delimiter': ',',
        'encoding': 'utf-8',
        'header': '0',
        'file_format': 'csv'
    }
}
```
### Json objects Samples

```bash

retailer = 'Chewy'
country = "USA"
company = "Hill's"
workbook_name = "Sales and Geographical Metrics"
view_range = "month"
view_name = "Sales Analysis Download"

additional_dag_params = {
    'retailer': retailer,
    'country': country,
    'company': company,
    'workbook': workbook_name,
    'granularity': view_range,
    'view_name': view_name
}

output_path_params = [
    'retailer',
    'country',
    'company',
    'workbook',
    'granularity',
    'view_name']

file_name_params = [
    dag_start_date,
    '{{run_id}}',
    report_start_date
]

input_file_format_details = {
    'delimiter': '\t',
    'encoding': 'utf-16',
    'header': '0',
    'file_format': 'tsv'
}

filter_map_1 = {"dag_id": dag.dag_id, "run_id": '{{run_id}}',
                'airflow_task_id': dw_task_id, 'dag_start_date': dag_start_date}

input_bucket_field_name = 'output_bucket_name'
input_path_field_name = 'gcs_output_file_path'
report_start_date_field_name = 'report_start_date'
report_end_date_field_name = 'report_end_date'

input_datastore_parameters = {
    'kind': 'ReportDownloadTask',
    'filter_map': filter_map_1,
    'datastore_field_names': {
                              'input_bucket': input_bucket_field_name,
                              'input_path': input_path_field_name,
                              'report_start_date': report_start_date_field_name,
                              'report_end_date': report_end_date_field_name,
                              }
}


```


### Working of Row Drop container

### STEP 1: Store into params- set_params_by_args method

Sets the params dictionary based on the directly passed dag parameters

Will return params (a dictionary), with key- value pairs, where key is parameter name and the value is the corresponding value passed directly via dag parameter

If a parameter value is not passed as dag parameter, then in the corresponding key value pair- the value will be None, which will be updated in next step if available in datastore

```bash
params ={
            'dag_id': args.dag_id,
            'run_id': args.run_id,
            'output_bucket_name': args.output_bucket_name,
            "airflow_task_id": args.airflow_task_id,
            'drop_top': args.drop_top,
            'drop_bot': args.drop_bot,
            'input_path': args.input_file_path,
            'input_bucket': args.input_bucket_name,
            'output_path': args.output_path,
            "encoding": args.input_file_format_details['encoding'],
            "delimiter": args.input_file_format_details['delimiter'],
            "header": args.input_file_format_details['header'],
            "file_format": args.input_file_format_details['file_format'],
            "filter_map": args.input_datastore_parameters['filter_map'],
            "kind": args.input_datastore_parameters['kind'],
            "report_start_date": args.report_start_date,
            "report_end_date": args.report_end_date,
            "skip_datastore": args.skip_datastore,
            "dag_start_date": args.dag_start_date
        }

```

Returns params


### STEP 2: fetch_and_store_datastore_arguments

**Updates the params dictionary**

If a parameter value is not passed as a dag parameter, and the field names are passed to fetch from datastore using **args.input_datastore_parameters['datastore_field_names']** then the key’s value will be updated and also a new key-value pair will be created if not already present.

      if args.input_datastore_parameters is provided  and  params['skip_datastore'] = False

          get the task entry

          for key in args.input_datastore_parameters['datastore_field_names'].keys():

              params[key] = entry[args.input_datastore_parameters['datastore_field_names'][key]]

Returns updated params


### STEP 3: download the file from gcs to local


* **Case 1**

      if params['input_path'] is not None and params['input_bucket'] is not None:
    
          download the file from gcs to local

* **Case 2**

      Else:

	        Raise exception ("input_file_path and/or input_bucket_name are neither provided as dag  parameters nor fetched from datastore")


### STEP 4: Construct local output file path (using constant local file name- not dependent on user input)


### STEP 5: Dropping rows- read the locally downloaded file, drop the rows based on the params[‘encoding’] and params[‘delimiter’]

Use csv.writer to write the required rows based on drop_top and drop_bot number

Returns row count


### STEP 6: Build gcs output file path - get_output_file_path

output_file_name = None

If args.file_name_params is provided then build the output_file_name and assign it to the above variable

 * **Case 1** if params['output_path'] is is not None i.e args.output_path is provided:
 
   * **Case 1.1** 
  
         if output_file_name is not None:
      
			       It means the provided output_path is not absolute path 
             
			       Hence append the output_file_name also to the path
	
   * **Case 1.2**

         Else:
      
             It means the provided output_path is absolute
 

* **Case 2** 

      if args.output_path is not provided, check if args.output_path_params is not None

          Build output path based on path params

          ( fail if any of the element in output_path_params does not exist in keys of  additional_dag_params) 

          output_path = os.path.join(output_path, 'row_drop')

          if output_file_name is still None i.e args.file_name_params is not provided:

                Use default file name

                output_file_name = dagstartdate_runid_reportstartdate.csv

           output_path = os.path.join(output_path, output_file_name)
        
* **Case 3**
 
      If neither args.output_path nor args.output_path_params is provided. 

          Then consider a default output_path

          output_path = '{}/{}/row_drop.csv'.format(params['dag_id'], params['run_id'])


### STEP 7: upload the modified file from local output file path to gcs output file path

* **Case 1** 

      if params['output_bucket_name'] is not None:
			
          Upload to gcs

* **Case 2**

      Else:
			
          Raise exception("output_bucket_name is None")


### STEP 8: Datastore: if args.skip_datastore is false

**handle_row_drop_task**

handle_row_drop_task method checks if the datastore entry for the given filter value is available or not. 

If the entry is already present then it will update the existing entry, else create a new entry and store it to a given Entity. 

Finally Calls put_snapshot_task_entry

**put_snapshot_task_entry**

Stores

1. dag_id 

1. run_id

1. report_start_date

1. report_end_date

1. input_file_path

1. input_bucket_name

1. output_bucket

1. path_params

1. output_file_path

1. ‘modified_at' = datetime.datetime.utcnow()

1. exceptions

1. exception_details

1. status

1. row_count

1. filter_kind

1. filter_map

1. airflow_task_id

1. dag_start_date

1. schema
