# sales_report

# Step1:
Deploy the required AWS components to complete this assessment using CloudFormation template/CDK.

First 1: To create 2 AWS Buckets using cloudformation one is used for source to connect to github and another for consumption ( to write the file )

```
AWSTemplateFormatVersion: 2010-09-09
Metadata:
  'AWS::CloudFormation::Designer':
    47c7d852-1012-45f8-9d02-bf11ef75cc9d:
      size:
        width: 60
        height: 60
      position:
        x: 57
        'y': 88
      z: 0
    b7a813da-c021-415b-8b08-35b72f8e09fa:
      size:
        width: 60
        height: 60
      position:
        x: 260
        'y': 90
      z: 0
Resources:
  SourceBucket:
    Type: 'AWS::S3::Bucket'
    Properties: {}
    Metadata:
      'AWS::CloudFormation::Designer':
        id: 47c7d852-1012-45f8-9d02-bf11ef75cc9d
  ConsumptionBucket:
    Type: 'AWS::S3::Bucket'
    Properties: {}
    Metadata:
      'AWS::CloudFormation::Designer':
        id: b7a813da-c021-415b-8b08-35b72f8e09fa
        
```

# Step2: 
To Build a CI/CD pipeline (Using codepipeline and codebuild) in such a way that it should be triggered automatically when there is a new change in git.

# Solution

Created the code pipeline which takes input from github(version1) "gopal23688/sales_report" and keep the content in s3 bucket "s3://salesreportinghema"

This code pipeline will checks if there is any change in data and updates the content in s3 bucket and the pipeline created is "Sales_Reporting"

# Step3:
The code should be able to take S3 location as user input,to pick the raw data from.
Data cleansing and transformation techiques to be implemented precisely.
Generate a daily report which shows HEMA sales per day, per product ,per store having the store with highest sales on the top with a roll out of past 7 days.

# Solution

1) Created the AWS Crawlers "HemaReporting" for searching the S3 Bucket "s3://salesreportinghema" and its underlying source tables sales ,products, stores which will identify the tables and create the tables which can be further used in the source while creating the mapping.
2) Created two Jobs for doing the ETL Operation and do the required analysis
   
   --> HemaReport is the job for creating the ETL and which takes the input from "s3://salesreportinghema" and do the necessary ETL tasks and load the data into " "s3://salesreportconsumption/"
   
## Below is the code snippet
```  
   import sys
from awsglue.transforms import *
from awsglue.utils import getResolvedOptions
from pyspark.context import SparkContext
from pyspark.conf import SparkConf
from pyspark.sql import SparkSession
from pyspark.sql.functions import *
from awsglue.context import GlueContext
from awsglue.job import Job
import pandas as pd
from awsglue.dynamicframe import DynamicFrame


args = getResolvedOptions(sys.argv, ['JOB_NAME'])
sc = SparkContext()
glueContext = GlueContext(sc)
spark = glueContext.spark_session
job = Job(glueContext)
job.init(args['JOB_NAME'], args)

sc2 = SparkSession.builder.appName("HemaSalesReport").config ("spark.sql.shuffle.partitions", "50").config("spark.driver.maxResultSize","5g") .config ("spark.sql.execution.arrow.enabled", "true").getOrCreate()

##@params: [JOB_NAME]
args = getResolvedOptions(sys.argv, ['JOB_NAME'])


##@type: DataSource
##@args: [database = "hemareporting", table_name = "products", transformation_ctx = "products"]
##@return: datasource0
##@inputs: []
products = glueContext.create_dynamic_frame.from_catalog(database = "hemareporting", table_name = "products", transformation_ctx = "products")


##@type: DataSource
##@args: [database = "hemareporting", table_name = "sales", redshift_tmp_dir = args["TempDir"], transformation_ctx = "<transformation_ctx>"]
##@return: <output>
##@inputs: []
sales = glueContext.create_dynamic_frame.from_catalog(database = "hemareporting", table_name = "sales", transformation_ctx = "sales")

##@type: DataSource
##@args: [database = "hemareporting", table_name = "stores", redshift_tmp_dir = args["TempDir"], transformation_ctx = "<transformation_ctx>"]
##@return: <output>
##@inputs: []
stores= glueContext.create_dynamic_frame.from_catalog(database = "hemareporting", table_name = "stores", transformation_ctx = "stores")


##Stores table dataframe creation

stores.toDF().select("store_id","store_name").createOrReplaceTempView("stores")



##Products table dataframe creation

products.toDF().select ("product_id","prod_desc").createOrReplaceTempView("products")

##Sales table data frame creation

##conversion of comma to dot format in amount column in the sales table


sales1=sales.toDF().withColumn('amount', regexp_replace('amount', ',', '.'))


sales1.show()


##conversion of string column "amount" to float

sales_format=sales1.withColumn('amount',sales1['amount'].cast("float"))

sales_format.show()




sales_format.select("invoice_date","product_id","store_id","amount").createOrReplaceTempView("sales_new")



##Using SQL finding out the required output for reporting

l_history=spark.sql("SELECT cast(a.invoice_date as date) invoice_date,a.product_id,b.prod_desc, a.store_id,c.store_name,sum(a.amount) as total_sales from sales_new a left join products b on a.product_id=b.product_id left join stores c on a.store_id=c.store_id group by 1,2,3,4,5 order by total_sales desc").na.fill('null')

##Repartitioning the data frame to give output as single file

partitioned_dataframe = l_history.repartition(1)

##Converting data frame to dyanamic frame using fromDF function

output=DynamicFrame.fromDF(partitioned_dataframe, glueContext, "partitioned_dataframe")

##Writing output to S3 Bucket


glueContext.write_dynamic_frame.from_options(frame = output, connection_type = "s3", connection_options = {"path": "s3://salesreportconsumption/"},format = "csv", format_options = {"writeHeader": True}, transformation_ctx = "datasink2")



job.commit()


```

--> report_rename : This is workflow which will check for the output file which is written to "s3://salesreportconsumption/" is in the spark format and this will be changed to desired format as "hemasalesdaily.csv" which has the desired output which needs to present to business lead.


# Code Snippet
```

import boto3
client = boto3.client('s3')

from datetime import datetime

BUCKET_NAME = 'salesreportconsumption'

response = client.list_objects(
Bucket=BUCKET_NAME,
)
name = response["Contents"][0]["Key"]

client.delete_object(Bucket=BUCKET_NAME, Key='hemasalesdaily.csv')


copy_source = {'Bucket': BUCKET_NAME, 'Key': name}
copy_key =  'hemasalesdaily.csv'
client.copy(CopySource=copy_source, Bucket=BUCKET_NAME, Key=copy_key)
client.delete_object(Bucket=BUCKET_NAME, Key=name)

```

# Step 4 : Created the workflow to run the above workflows at 7 AM daily and workflow name is "reportingdaily" which triggers the workflows one after another.


# General

1) I have given necessary roles for making this usecase working 
2) Data is not having the latest data and hence in code sql i have not added date-7 as it will not populate the table output.
  
   Note: I can use the below filter to the data frames to get the output for last 7 days .
```   
   from pyspark.sql.functions import current_date, datediff, unix_timestamp

   sales.toDF().where(invoice_date(current_date(), col("dt")) < 7))

```

