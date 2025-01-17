# Use cases<a name="r_create_model_use_cases"></a>

The following use cases demonstrate how to use CREATE MODEL to suit your needs\.

## Simple CREATE MODEL<a name="r_simple_create_model"></a>

The following summarizes the basic options of the CREATE MODEL syntax\.

### Simple CREATE MODEL syntax<a name="r_simple-create-model-synposis"></a>

```
CREATE MODEL model_name 
    FROM { table_name | ( select_query ) }
    TARGET column_name
    FUNCTION prediction_function_name
    IAM_ROLE { default }
    SETTINGS (
      S3_BUCKET 'bucket',
      [ MAX_CELLS integer ]
    )
```

### Simple CREATE MODEL parameters<a name="r_simple-create-model-parameters"></a>

 *model\_name*   
The name of the model\. The model name in a schema must be unique\.

FROM \{ *table\_name* \| \( *select\_query* \) \}  
The table\_name or the query that specifies the training data\. They can either be an existing table in the system, or an Amazon Redshift\-compatible SELECT query enclosed with parentheses, that is \(\)\. There must be at least two columns in the query result\. 

TARGET *column\_name*  
The name of the column that becomes the prediction target\. The column must exist in the FROM clause\. 

FUNCTION *prediction\_function\_name*   
A value that specifies the name of the Amazon Redshift machine learning function to be generated by the CREATE MODEL and used to make predictions using this model\. The function is created in the same schema as the model object and can be overloaded\.  
Amazon Redshift machine learning supports models, such as Xtreme Gradient Boosted tree \(XGBoost\) models for regression and classification\.

IAM\_ROLE \{ default \}  
 Use the default keyword to have Amazon Redshift use the IAM role that is set as default and associated with the cluster when the CREAT MODEL command runs\.

 *S3\_BUCKET *'bucket'**   
The name of the Amazon S3 bucket that you previously created used to share training data and artifacts between Amazon Redshift and SageMaker\. Amazon Redshift creates a subfolder in this bucket prior to unload of the training data\. When training is complete, Amazon Redshift deletes the created subfolder and its contents\. 

MAX\_CELLS integer   
The maximum number of cells to export from the FROM clause\. The default is 1,000,000\.   
The number of cells is the product of the number of rows in the training data \(produced by the FROM clause table or query\) times the number of columns\. If the number of cells in the training data are more than that specified by the max\_cells parameter, CREATE MODEL downsamples the FROM clause training data to reduce the size of the training set below MAX\_CELLS\. Allowing larger training datasets can produce higher accuracy but also can mean the model takes longer to train and costs more\.  
For information about costs of using Amazon Redshift, see [Costs for using Amazon Redshift ML](cost.md)\.  
For more information about costs associated with various cell numbers and free trial details, see [Amazon Redshift pricing](https://aws.amazon.com/redshift/pricing)\.

## CREATE MODEL with user guidance<a name="r_user_guidance_create_model"></a>

Following, you can find a description of options for CREATE MODEL in addition to the options described in [Simple CREATE MODEL](#r_simple_create_model)\.

By default, CREATE MODEL searches for the best combination of preprocessing and model for your specific dataset\. You might want additional control or introduce additional domain knowledge \(such as problem type or objective\) over your model\. In a customer churn scenario, if the outcome “customer is not active” is rare, then the F1 objective is often preferred to the accuracy objective\. Because high accuracy models might predict “customer is active” all the time, this results in high accuracy but little business value\. For information about F1 objective, see [AutoMLJobObjective](https://docs.aws.amazon.com/sagemaker/latest/APIReference/API_AutoMLJobObjective.html) in the *Amazon SageMaker API Reference*\.

Then the CREATE MODEL follows your suggestions on the specified aspects, such as the objective\. At the same time, the CREATE MODEL automatically discovers the best preprocessors and the best hyperparameters\. 

### CREATE MODEL with user guidance syntax<a name="r_user_guidance-create-model-synposis"></a>

CREATE MODEL offers more flexibility on the aspects that you can specify and the aspects that Amazon Redshift automatically discovers\.

```
CREATE MODEL model_name 
    FROM { table_name | ( select_statement ) }
    TARGET column_name
    FUNCTION function_name
    IAM_ROLE { default }
    [ MODEL_TYPE { XGBOOST | MLP | LINEAR_LEARNER} ]              
    [ PROBLEM_TYPE ( REGRESSION | BINARY_CLASSIFICATION | MULTICLASS_CLASSIFICATION ) ]
    [ OBJECTIVE ( 'MSE' | 'Accuracy' | 'F1' | 'F1Macro' | 'AUC') ]
    SETTINGS (
      S3_BUCKET 'bucket', |
      S3_GARBAGE_COLLECT { ON | OFF }, |
      KMS_KEY_ID 'kms_key_id', |
      MAX_CELLS integer, |
      MAX_RUNTIME integer (, ...)
    )
```

### CREATE MODEL with user guidance parameters<a name="r_user_guidance-create-model-parameters"></a>

 *MODEL\_TYPE \{ XGBOOST \| MLP \| LINEAR\_LEARNER \}*   
\(Optional\) Specifies the model type\. You can specify if you want to train a model of a specific model type, such as XGBoost, multilayer perceptron \(MLP\), or Linear Learner, which are all algorithms that Amazon SageMaker Autopilot supports\. If you don't specify the parameter, then all supported model types are searched during training for the best model\.

 *PROBLEM\_TYPE \( REGRESSION \| BINARY\_CLASSIFICATION \| MULTICLASS\_CLASSIFICATION \)*   
\(Optional\) Specifies the problem type\. If you know the problem type, you can restrict Amazon Redshift to only search of the best model of that specific model type\. If you don't specify this parameter, a problem type is discovered during the training, based on your data\.

OBJECTIVE \( 'MSE' \| 'Accuracy' \| 'F1' \| 'F1Macro' \| 'AUC'\)  
\(Optional\) Specifies the name of the objective metric used to measure the predictive quality of a machine learning system\. This metric is optimized during training to provide the best estimate for model parameter values from data\. If you don't specify a metric explicitly, the default behavior is to automatically use MSE: for regression, F1: for binary classification, Accuracy: for multiclass classification\. For more information about objectives, see [AutoMLJobObjective](https://docs.aws.amazon.com/sagemaker/latest/APIReference/API_AutoMLJobObjective.html) in the *Amazon SageMaker API Reference*\.

MAX\_CELLS integer   
\(Optional\) Specifies the number of cells in the training data\. This value is the product of the number of records \(in the training query or table\) times the number of columns\. The default is 1,000,000\.

MAX\_RUNTIME integer   
\(Optional\) Specifies the maximum amount of time to train\. Training jobs often complete sooner depending on dataset size\. This specifies the maximum amount of time the training should take\. The default is 5,400 \(90 minutes\)\.

S3\_GARBAGE\_COLLECT \{ ON \| OFF \}  
\(Optional\) Specifies whether Amazon Redshift performs garbage collection on the resulting datasets used to train models and the models\. If set to OFF, the resulting datasets used to train models and the models remains in Amazon S3 and can be used for other purposes\. If set to ON, Amazon Redshift deletes the artifacts in Amazon S3 after the training completes\. The default is ON\.

KMS\_KEY\_ID 'kms\_key\_id'  
\(Optional\) Specifies if Amazon Redshift uses server\-side encryption with an AWS KMS key to protect data at rest\. Data in transit is protected with Secure Sockets Layer \(SSL\)\. 

 *PREPROCESSORS 'string' *   
\(Optional\) Specifies certain combinations of preprocessors to certain sets of columns\. The format is a list of columnSets, and the appropriate transforms to be applied to each set of columns\. Amazon Redshift applies all the transformers in a specific transformers list to all columns in the corresponding ColumnSet\. For example, to apply OneHotEncoder with Imputer to columns t1 and t2, use the sample command following\.  

```
CREATE MODEL customer_churn
    FROM customer_data 
    TARGET 'Churn'
    FUNCTION predict_churn
    IAM_ROLE { default }
    PROBLEM_TYPE BINARY_CLASSIFICATION
    OBJECTIVE 'F1'
    PREPROCESSORS '[
    ...
      {"ColumnSet": [
          "t1",
          "t2"
        ],
        "Transformers": [
          "OneHotEncoder",
          "Imputer"
        ]
      },
      {"ColumnSet": [
          "t3"
        ],
        "Transformers": [
          "OneHotEncoder"
        ]
      },
      {"ColumnSet": [
          "temp"
        ],
        "Transformers": [
          "Imputer",
          "NumericPassthrough"
        ]
      }
    ]'
    SETTINGS (
      S3_BUCKET 'bucket'
    )
```

Amazon Redshift supports the following transformers:
+ OneHotEncoder – Typically used to encode a discrete value into a binary vector with one nonzero value\. This transformer is suitable for many machine learning models\.  
+ OrdinalEncoder – Encodes discrete values into a single integer\. This transformer is suitable for certain machine learning models, such as MLP and Linear Learner\. 
+ NumericPassthrough – Passes input as is into the model\.
+ Imputer – Fills in missing values and not a number \(NaN\) values\.
+ ImputerWithIndicator – Fills in missing values and NaN values\. This transformer also creates an indicator of whether any values were missing and filled in\.
+ Normalizer – Normalizes values, which can improve the performance of many machine learning algorithms\.
+ DateTimeVectorizer – Creates a vector embedding, representing a column of datetime data type that can be used in machine learning models\.
+ PCA – Projects the data into a lower dimensional space to reduce the number of features while keeping as much information as possible\.
+ StandardScaler – Standardizes features by removing the mean and scaling to unit variance\. 
+ MinMax – Transforms features by scaling each feature to a given range\.

Amazon Redshift ML stores the trained transformers, and automatically applies them as part of the prediction query\. You don't need to specify them when generating predictions from your model\. 

## CREATE XGBoost models with AUTO OFF<a name="r_auto_off_create_model"></a>

The AUTO OFF CREATE MODEL has generally different objectives from the default CREATE MODEL\.

As an advanced user who already knows the model type that you want and hyperparameters to use when training these models, you can use CREATE MODEL with AUTO OFF to turn off the CREATE MODEL automatic discovery of preprocessors and hyperparameters\. To do so, you explicitly specify the model type\. XGBoost is currently the only model type supported when AUTO is set to OFF\. You can specify hyperparameters\. Amazon Redshift uses default values for any hyperparameters that you specified\.  

### CREATE XGBoost models with AUTO OFF syntax<a name="r_auto_off-create-model-synposis"></a>

```
CREATE MODEL model_name
    FROM { table_name | (select_statement ) }
    TARGET column_name    
    FUNCTION function_name
    IAM_ROLE { default }
    AUTO OFF
    MODEL_TYPE XGBOOST
    OBJECTIVE { 'reg:squarederror' | 'reg:squaredlogerror' | 'reg:logistic' | 
                'reg:pseudohubererror' | 'reg:tweedie' | 'binary:logistic' | 'binary:hinge' | 
                'multi:softmax' | 'rank:pairwise' | 'rank:ndcg' }
    HYPERPARAMETERS DEFAULT EXCEPT (
        NUM_ROUND '10',
        ETA '0.2',
        NUM_CLASS '10',
        (, ...)
    )
    PREPROCESSORS 'none'
    SETTINGS (
      S3_BUCKET 'bucket', |
      S3_GARBAGE_COLLECT { ON | OFF }, |
      KMS_KEY_ID 'kms_key_id', |
      MAX_CELLS integer, |
      MAX_RUNTIME integer (, ...)
    )
```

### CREATE XGBoost models with AUTO OFF parameters<a name="r_auto_off-create-model-parameters"></a>

 *AUTO OFF*   
Turns off CREATE MODEL automatic discovery of preprocessor, algorithm, and hyper\-parameters selection\.

MODEL\_TYPE XGBOOST  
Specifies to use XGBOOST to train the model\. 

OBJECTIVE str  
Specifies an objective recognized by the algorithm\. Amazon Redshift supports reg:squarederror, reg:squaredlogerror, reg:logistic, reg:pseudohubererror, reg:tweedie, binary:logistic, binary:hinge, multi:softmax\. For more information about these objectives, see [Learning task parameters](https://xgboost.readthedocs.io/en/latest/parameter.html#learning-task-parameters) in the XGBoost documentation\.

HYPERPARAMETERS \{ DEFAULT \| DEFAULT EXCEPT \( key ‘value’ \(,\.\.\) \) \}  
Specifies whether the default XGBoost parameters are used or overridden by user\-specified values\. The values must be enclosed with single quotes\. Following are examples of parameters for XGBoost and their defaults\.      
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/redshift/latest/dg/r_create_model_use_cases.html)

The following example prepares data for XGBoost\.

```
DROP TABLE IF EXISTS abalone_xgb;
    
    CREATE TABLE abalone_xgb (
    length_val float, 
    diameter float, 
    height float,
    whole_weight float, 
    shucked_weight float, 
    viscera_weight float,
    shell_weight float, 
    rings int,
    record_number int);
    
    COPY abalone_xgb
    FROM 's3://redshift-downloads/redshift-ml/abalone_xg/'
    REGION 'us-east-1'
    IAM_ROLE default
    IGNOREHEADER 1 CSV;
```

The following example creates an XGBoost model with specified advanced options, such as MODEL\_TYPE, OBJECTIVE, and PREPROCESSORS\.

```
DROP MODEL abalone_xgboost_multi_predict_age;
    
    CREATE MODEL abalone_xgboost_multi_predict_age
    FROM ( SELECT length_val,
                  diameter,
                  height,
                  whole_weight,
                  shucked_weight,
                  viscera_weight,
                  shell_weight,
                  rings 
            FROM abalone_xgb WHERE record_number < 2500 )
    TARGET rings FUNCTION ml_fn_abalone_xgboost_multi_predict_age
    IAM_ROLE default
    AUTO OFF
    MODEL_TYPE XGBOOST
    OBJECTIVE 'multi:softmax'
    PREPROCESSORS 'none'
    HYPERPARAMETERS DEFAULT EXCEPT (NUM_ROUND '100', NUM_CLASS '30')
    SETTINGS (S3_BUCKET 'your-bucket');
```

The following example uses an inference query to predict the age of the fish with a record number greater than 2500\. It uses the function ml\_fn\_abalone\_xgboost\_multi\_predict\_age created from the above command\. 

```
select ml_fn_abalone_xgboost_multi_predict_age(length_val, 
                                                   diameter, 
                                                   height, 
                                                   whole_weight, 
                                                   shucked_weight, 
                                                   viscera_weight, 
                                                   shell_weight)+1.5 as age 
    from abalone_xgb where record_number > 2500;
```

## Bring your own model \(BYOM\) \- local inference<a name="r_byom_create_model"></a>

Amazon Redshift ML supports using bring your own model \(BYOM\) for local inference\.

The following summarizes the options for the CREATE MODEL syntax for BYOM\. You can use a model trained outside of Amazon Redshift with Amazon SageMaker for in\-database inference locally in Amazon Redshift\.

### CREATE MODEL syntax for local inference<a name="r_local-create-model"></a>

The following describes the CREATE MODEL syntax for local inference\.

```
CREATE MODEL model_name
    FROM ('job_name' | 's3_path' )
    FUNCTION function_name ( data_type [, ...] )
    RETURNS data_type
    IAM_ROLE { default }
    [ SETTINGS (
      S3_BUCKET 'bucket', | --required
      KMS_KEY_ID 'kms_string') --optional
    ];
```

Amazon Redshift currently only supports pretrained XGBoost, MLP, and Linear Learner models for BYOM\. You can import SageMaker Autopilot and models directly trained in Amazon SageMaker for local inference using this path\.  

#### CREATE MODEL parameters for local inference<a name="r_local-create-model-parameters"></a>

 *model\_name*   
The name of the model\. The model name in a schema must be unique\.

FROM \(*'job\_name'* \| *'s3\_path'* \)  
The *job\_name* uses an Amazon SageMaker job name as the input\. The job name can either be an Amazon SageMaker training job name or an Amazon SageMaker Autopilot job name\. The job must be created in the same AWS account that owns the Amazon Redshift cluster\. To find the job name, launch Amazon SageMaker\. In the **Training** dropdown menu, choose **Training jobs**\.  
The *'s3\_path'* specifies the S3 location of the \.tar\.gz model artifacts file that is to be used when creating the model\.

FUNCTION *function\_name* \( *data\_type* \[, \.\.\.\] \)  
The name of the function to be created and the data types of the input arguments\. You can provide a schema name\.

RETURNS *data\_type*  
The data type of the value returned by the function\.

IAM\_ROLE \{ default \}  
 Use the default keyword to have Amazon Redshift use the IAM role that is set as default and associated with the cluster when the CREATE MODEL command runs\.  
Use the Amazon Resource Name \(ARN\) for an IAM role that your cluster uses for authentication and authorization\. 

SETTINGS \( S3\_BUCKET *'bucket'*, \| KMS\_KEY\_ID *'kms\_string'*\)  
The S3\_BUCKET clause specifies the Amazon S3 location that is used to store intermediate results\.  
\(Optional\) The KMS\_KEY\_ID clause specifies if Amazon Redshift uses server\-side encryption with an AWS KMS key to protect data at rest\. Data in transit is protected with Secure Sockets Layer \(SSL\)\.  
For more information, see [CREATE MODEL with user guidance](#r_user_guidance_create_model)\.

#### CREATE MODEL for local inference example<a name="r_local-create-model-example"></a>

The following example creates a model that has been previously trained in Amazon SageMaker, outside of Amazon Redshift\. Because the model type is supported by Amazon Redshift ML for local inference, the following CREATE MODEL creates a function that can be used locally in Amazon Redshift\. You can provide a SageMaker training job name\.

```
CREATE MODEL customer_churn
    FROM 'training-job-customer-churn-v4'
    FUNCTION customer_churn_predict (varchar, int, float, float)
    RETURNS int
    IAM_ROLE default
    SETTINGS (S3_BUCKET 'your-bucket');
```

After the model is created, you can use the function *customer\_churn\_predict* with the specified argument types to make predictions\.

## Bring your own model \(BYOM\) \- remote inference<a name="r_byom_create_model_remote"></a>

Amazon Redshift ML also supports using bring your own model \(BYOM\) for remote inference\.

The following summarizes the options for the CREATE MODEL syntax for BYOM\.

### CREATE MODEL syntax for remote inference<a name="r_remote-create-model"></a>

The following describes the CREATE MODEL syntax for remote inference\.

```
CREATE MODEL model_name 
    FUNCTION function_name ( data_type [, ...] ) 
    RETURNS data_type
    SAGEMAKER 'endpoint_name'[:'model_name']
    IAM_ROLE { default };
```

#### CREATE MODEL parameters for remote inference<a name="r_remote-create-model-parameters"></a>

 *model\_name*   
The name of the model\. The model name in a schema must be unique\.

FUNCTION *fn\_name* \( \[*data\_type*\] \[, \.\.\.\] \)  
The name of the function and the data types of the input arguments\. You can provide a schema name\.

RETURNS *data\_type*  
The data type of the value returned by the function\.

SAGEMAKER *'endpoint\_name'*\[:*'model\_name'*\]   
The name of the Amazon SageMaker endpoint\. If the endpoint name points to a multimodel endpoint, add the name of the model to use\. The endpoint must be hosted in the same AWS Region as the Amazon Redshift cluster\. To find your endpoint, launch Amazon SageMaker\. In the **Inference** dropdown menu, choose **Endpoints**\.

IAM\_ROLE \{ default \}  
 Use the default keyword to have Amazon Redshift use the IAM role that is set as default and associated with the cluster when the CREATE MODEL command runs\.

When the model is deployed to a SageMaker endpoint, SageMaker creates the information of the model in Amazon Redshift\. It then performs inference through the external function\. You can use the SHOW MODEL command to view the model information on your Amazon Redshift cluster\.

#### CREATE MODEL for remote inference usage notes<a name="r_remote-create-model-usage-notes"></a>

Before using CREATE MODEL for remote inference, consider the following:
+ The model must accept inputs in the format of comma\-separated values \(CSV\) through a content type of text or CSV in SageMaker\.
+ The endpoint must be hosted by the same AWS account that owns the Amazon Redshift cluster\.
+ The outputs of models must be a single value of the type specified on creating the function, in the format of comma\-separated values \(CSV\) through a content type of text or CSV in SageMaker\. 
+ Models accept nulls as empty strings\.
+ Make sure either that the Amazon SageMaker endpoint has enough resources to accommodate inference calls from Amazon Redshift or that the Amazon SageMaker endpoint can be automatically scaled\. 

##### CREATE MODEL for remote inference example<a name="r_remote-create-model-example"></a>

The following example creates a model that uses a SageMaker endpoint to make predictions\. Make sure that the endpoint is running to make predictions and specify its name in the CREATE MODEL command\.

```
CREATE MODEL remote_customer_churn
    FUNCTION remote_fn_customer_churn_predict (varchar, int, float, float)
    RETURNS int
    SAGEMAKER 'customer-churn-endpoint'
    IAM_ROLE default;
```

## CREATE MODEL with K\-MEANS<a name="r_k-means_create_model"></a>

Amazon Redshift supports the K\-Means algorithm that groups data that isn't labeled\. This algorithm solves clustering problems where you want to discover groupings in the data\. Unclassified data is grouped and partitioned based on its similarities and differences\. 

### CREATE MODEL with K\-MEANS syntax<a name="r_k-means-create-model-synposis"></a>

```
CREATE MODEL model_name
    FROM { table_name | ( select_statement ) }
    FUNCTION function_name
    IAM_ROLE { default }
    AUTO OFF
    MODEL_TYPE KMEANS
    PREPROCESSORS 'string'
    HYPERPARAMETERS DEFAULT EXCEPT ( K 'val' [, ...] )
    SETTINGS (
      S3_BUCKET 'bucket',
      KMS_KEY_ID 'kms_string', |
        -- optional
      S3_GARBAGE_COLLECT on / off, |
        -- optional
      MAX_CELLS integer, |
        -- optional
      MAX_RUNTIME integer
        -- optional);
```

### CREATE MODEL with K\-MEANS parameters<a name="r_k-means-create-model-parameters"></a>

 *AUTO OFF*   
Turns off CREATE MODEL automatic discovery of preprocessor, algorithm, and hyper\-parameters selection\.

MODEL\_TYPE KMEANS  
Specifies to use KMEANS to train the model\. 

PREPROCESSORS 'string'  
Specifies certain combinations of preprocessors to certain sets of columns\. The format is a list of columnSets, and the appropriate transforms to be applied to each set of columns\. Amazon Redshift supports 3 K\-Means preprocessors, namely StandardScaler, MinMax, and NumericPassthrough\. If you don't want to apply any preprocessing for K\-Means, choose NumericPassthrough explicitly as a transformer\. For more information about supported transformers, see [CREATE MODEL with user guidance parameters](#r_user_guidance-create-model-parameters)\.  
The K\-Means algorithm uses Euclidean distance to calculate similarity\. Preprocessing the data ensures that the features of the model stay on the same scale and produce reliable results\.

HYPERPARAMETERS DEFAULT EXCEPT \( K 'val' \[, \.\.\.\] \)  
Specifies whether the K\-Means parameters are used\. You must specify the `K` parameter when using the K\-Means algorithm\. For more information, see [K\-Means Hyperparameters](https://docs.aws.amazon.com/sagemaker/latest/dg/k-means-api-config.html) in the *Amazon SageMaker Developer Guide*

The following example prepares data for K\-Means\.

```
CREATE MODEL customers_clusters
    FROM customers
    FUNCTION customers_cluster
    IAM_ROLE default
    AUTO OFF
    MODEL_TYPE KMEANS
    PREPROCESSORS '[
      {
        "ColumnSet": [ "*" ],
        "Transformers": [ "NumericPassthrough" ]
      }
    ]'
    HYPERPARAMETERS DEFAULT EXCEPT ( K '5' )
    SETTINGS (S3_BUCKET 'bucket');
    
    select customer_id, customers_cluster(...) from customers;
    customer_id | customers_cluster
    --------------------
    12345            1
    12346            2
    12347            4
    12348            0
```