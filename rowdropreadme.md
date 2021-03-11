# row_drop_automation
**Row Drop Container**- Based on the input parsed arguments, the row drop container downloads to local the file from gcs input path, drops the top and/or the bottom rows of the file, and uploads the modified file to the gcs output path.

**row drop container path** = `"./airflow-containers/row_drop_automation"`


| s. no | parsed argument            | type       | required| help                                                           | default                                   |
| ----- | ---------------------------| -----------| --------| ---------------------------------------------------------------|-------------------------------------------
| 1     | `dag_id`                   | str        | True    | "dag id to query data from datastore"                          |                                           
| 2     | `run_id`                   | str        | True    | "run id of the dag"                                            |                     
| 3     | `output_bucket`            | str        | True    | "output bucket name to store the modified file"                |
| 4     | `drop_top`                 | str        | True    | "number of rows to drop from the top of the file"              |                  
| 5     | `drop_bot`                 | str        | True    | "number of rows to drop from the bottom of the file"           |
| 6     | `airflow_task_id`          | str        | True    | "airflow task id"                                              |
| 7     | `input_path`               | str        | False   | "absolute path to csv report"                                  |                  
| 8     | `input_bucket`             | str        | False   | "input bucket name"                                            |
| 9     | `output_path`              | str        | False   | "absolute path where the modified file will be stored"         |
| 10    | `input_from_datastore`     | json.loads | False   | "keys required to retrieve corresponding values from data store"|                  
| 11    | `additional_dag_params`    | json.loads | False   | "additional dag params"                                        |
| 12    | `output_path_params`       | json.loads | False   | "parameters to build gcs output path where file will be stored"|
| 13    | `file_name_params`         | json.loads | False   | "parameters to build output file name where file will be stored"|                  
| 14    | `input_file_format_details`| json.loads | False   | "parameters related to input file format"                      | json.dumps(constant.input_file_format_details)**
| 15    | `skip_datastore`           | str        | False   | "Whether to use data store or not"                             | False

** Default value of `args.input_file_format_details`
```bash
input_file_format_details = {
        'delimiter': ',',
        'encoding': 'utf-8',
        'header': '0',
        'file_format': 'csv'
    }
}
```
### Sample Json objects to be passed as dag parameters

```bash
retailer = 'Chewy'
country = "USA"
company = "Hill's"
workbook_name = "Sales and Geographical Metrics"
view_range = "month"
view_name = "Sales Analysis Download"

output_path_params = [
    'retailer',
    'country',
    'company',
    'workbook',
    'granularity',
    'view_name']

file_name_params = [
    'dag_start_date',
    'run_id',
    'report_start_date',
    'table_name'
]


input_file_format_details = {
    'delimiter': '\t',
    'encoding': 'utf-16',
    'header': '0',
    'file_format': 'tsv'
}

filter_map_1 = {"dag_id": dag.dag_id, "run_id": '{{run_id}}',
                'airflow_task_id': dw_task_id}

key_in_datastore_for_input_bucket = 'output_bucket_name'
key_in_datastore_for_input_path = 'gcs_output_file_path'
key_in_datastore_for_report_start_date = 'report_start_date'
key_in_datestore_for_report_end_date = 'report_end_date'


input_from_datastore = {
    'kind': 'ReportDownloadTask',
    'filter_map': filter_map_1,
    'fetch_datastore_params': {'input_bucket': key_in_datastore_for_input_bucket,
                               'input_path': key_in_datastore_for_input_path,
                               'report_start_date': key_in_datastore_for_report_start_date,
                               'report_end_date': key_in_datestore_for_report_end_date,
                               'schema': key_in_datestore_for_schema
		       ‘dag_start_date’: key_in_datestore_for_dag_start_date
                               }
    ‘order_task_entries_params’ = {
    			'order_by_key_list': ['dag_start_date'],
   			 'descending_order': True
}

}


additional_dag_params = {
    'retailer': retailer,
    'country': country,
    'company': company,
    'workbook': workbook_name,
    'granularity': view_range,
    'view_name': view_name,
    'dag_start_date': dag_start_date,
    'run_id': run_id,
    'report_start_date': report_start_date,
    'table_name': table_name
}

or

If not fetching from datastore(args.input_from_datastore is None) can pass directly as additional dag params

additional_dag_params = {
    'retailer': retailer,
    'country': country,
    'company': company,
    'workbook': workbook_name,
    'granularity': view_range,
    'view_name': view_name,
    'dag_start_date': dag_start_date,
    'run_id': run_id,
    'report_start_date': report_start_date,
    'table_name': table_name,
    'report_end_date': report_end_date,
    'kind': 'ReportDownloadTask',
    'filter_map': filter_map_1,
    'schema': ["Month of","Brand","Part Number","Vendor Part Number","Units Sold","COGS","CPU"]
}

```


### Working of Row Drop container

### STEP 1: Store into params- `set_params_by_args` method

Sets the params dictionary based on the directly passed dag parameters

Will return params (a dictionary), with key- value pairs, where key is parameter name and the value is the corresponding value passed directly via dag parameter

If a parameter value is not passed as dag parameter, then in the corresponding key value pair- the value will be None, which will be updated in next step if available in datastore

```bash
params ={'dag_id': args.dag_id,
            'run_id': args.run_id,
            'output_bucket': args.output_bucket,
            'drop_top': args.drop_top,
            'drop_bot': args.drop_bot,
            "airflow_task_id": args.airflow_task_id,
            'input_path': args.input_path,
            'input_bucket': args.input_bucket,
            'output_path': args.output_path,
            "encoding": args.input_file_format_details['encoding'],
            "delimiter": args.input_file_format_details['delimiter'],
            "header": args.input_file_format_details['header'],
            "file_format": args.input_file_format_details['file_format'],
            "skip_datastore": args.skip_datastore,
            'input_from_datastore': args.input_from_datastore,
            'additional_dag_params': args.additional_dag_params,
            'output_path_params': args.output_path_params,
            'file_name_params': args.file_name_params,
            'input_file_format_details': args.input_file_format_details,
            'row_count': None,
            'exceptions': None,
            'exception_details': None
        }




```

Returns `params`

### STEP 2: fetch_and_store_datastore_arguments

**Updates the params dictionary**

If a parameter value is not passed as a standard dag parameter, and passed as one of the `args.additional_dag_params`, then params dictionary key-value pairs are updated.

Returns updated `params`

### STEP 3: fetch_and_store_datastore_arguments

If a parameter value is neither passed as a standard dag parameter nor as in `args.additional_dag_params`, and if the datastore key names are passed to fetch from datastore using  `args.input_from_datastore['fetch_datastore_params']`. 

Then the datastore keys are used to fetch the values from the datastore, and the key’s value will be updated in `params` dictionary and also a new key-value pair will be created if not already present.

***3.1. get the task entry***

Get the list of entries based on the `filter_ma`p and `kind`

Sort the obtained list of entries based on the `order_task_entries_params[‘order_by_key_list’]` and `order_task_entries_params[‘descending_order’]`, where `order_task_entries_params` is passed as a nested json object in `args.input_from_datastore` from the dag

    order_task_entries_params = args.input_from_datastore['order_task_entries_params']
    
    report_entries = get_task_entry(client, filter_map, kind, order_task_entries_params)

    def get_task_entry(client, filter_map, kind, order_task_entries_params):
    
        entries = fetch list of task entries based on input filter_map and kind
        
        if order_task_entries_params is not None:

            entries = sorted(entries, key=itemgetter(*order_task_entries_params['order_by_key_list']), reverse=order_task_entries_params['descending_order'])

        return entries

Note:

`order_task_entries_params[‘order_by_key_list’]`: should be a list object of keys on which the entries will be sorted

`order_task_entries_params[‘descending_order’]`: takes value True or False

***3.2. Update the params dictionary***

    if params[‘input_from_datastore’] is not None  and  params['skip_datastore'] = False
    		
        for key in params[‘input_from_datastore’]['fetch_datastore_params'].keys():
        
             params[key] = entry[params['input_from_datastore']['fetch_datastore_params'][key]]


Returns updated `params`

`params` dictionary now contains all the parameters(standard_dag_parameters, additional_dag_parameters, parameters_fetched_from_datastore) in a single object. 

`params` will be used from here on.

### STEP 4: download the file from gcs to local


* **Case 1**

      if params['input_path'] is not None and params['input_bucket'] is not None:
    
          download the file from gcs to local

* **Case 2**

      Else:

	  Raise exception ("input_file_path and/or input_bucket_name are neither provided as dag  parameters nor fetched from datastore")


### STEP 5: Construct local output file path (using constant local file name)

```bash
constant.local_output_file_name = 'dropped_row_report.csv'

local_output_file_path = os.path.join(os.path.dirname(os.path.abspath(__file__)), constant.local_output_file_name)
```


### STEP 6: Dropping rows- read the locally downloaded file, drop the rows based on the `params[‘encoding’]` and `params[‘delimiter’]`

Use `csv.writer` to write the required rows based on `params['drop_top']` and `params['drop_bot']` number

Returns `row_count`


### STEP 7: Build gcs output file path - get_output_file_path

Calls **get_output_file_name**

	output_file_name = None

	If params[‘file_name_params’] is not None i.e args.file_name_params 
	    then build the output_file_name 
	    assign it to the above variable output_file_name

Build output_file_name based on params[‘file_name_params’] and params[‘additional_dag_params’]
 ( fail if any of the element in file_name_params does not exist in keys of  params[‘additional_dag_params’])     
    

Return output_file_name to **get_output_file_path** method

Return to get_output_file_path method

In **get_output_file_path** method

 * **Case 1** if params['output_path'] is is not None i.e args.output_path is provided:
 
   * **Case 1.1** 
  
         if output_file_name is not None:
      
			       It means the provided output_path is not absolute path 
             
			       Hence append the output_file_name also to the path
	
   * **Case 1.2**

         Else:
      
             It means the provided output_path is absolute
 

* **Case 2** 

 if params[‘output_path’] is not None i.e. args.output_path is not provided, check if params[‘output_path_params’] is not None i.e. args.output_path_params
is provided

Build output path based on params[‘output_path_params’] and params[‘additional_dag_params’]

( fail if any of the element in params[‘output_path_params’] does not exist in keys of  params[‘additional_dag_params’]) 

          output_path = os.path.join(output_path, 'row_drop')

          if output_file_name is still None i.e args.file_name_params is not provided:

                Use default file name
                
                if params['report_start_date'] is not None:

                    output_file_name = dagstartdate_runid_reportstartdate.csv
                    
                else:
                    
                    output_file_name = dagstartdate_runid.csv
                
           output_path = os.path.join(output_path, output_file_name)
        
* **Case 3**
 
  If params[‘output_path’] is None and params[output_path_params] is None: 
  
  i.e. neither args.output_path nor args.output_path_params is provided. Then consider a default output_path


          output_path = '{}/{}/row_drop.csv'.format(params['dag_id'], params['run_id'])


### STEP 8: upload the modified file from local output file path to gcs output file path

* **Case 1** 

      if params['output_bucket_name'] is not None:
			
          Upload to gcs

* **Case 2**

      Else:
			
          Raise exception("output_bucket_name is None")


### STEP 9: Datastore: if args.skip_datastore is false

**handle_row_drop_task**

handle_row_drop_task method checks if the datastore entry for the given filter value is available or not. 

If the entry is already present then it will update the existing entry, else create a new entry and store it to a given Entity. 

Finally Calls put_snapshot_task_entry

**put_snapshot_task_entry**

Stores all the key- value pairs present in updated params dictionary

1. dag_id          
1. run_id
1. output_bucket
1. drop_top
1. drop_bot
1. airflow_task_id
1. input_path
1. input_bucket
1. output_path
1. encoding
1. delimiter
1. header           
1. file_format
1. skip_datastore
1. input_from_datastore
1. additional_dag_params 
1. output_path_params
1. file_name_params
1. input_file_format_details
1. row_count
1. exceptions
1. exception_details
1. modified_at
1. created_at

Additionally stores all the values fetched from datastore and all the key-value pairs passed in additional_dag_params seperately
