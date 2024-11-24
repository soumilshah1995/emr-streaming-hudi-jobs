# emr-streaming-hudi-jobs
emr-streaming-hudi-jobs
![image](https://github.com/user-attachments/assets/d0538cdd-5792-49be-955c-904e169d2270)


# PySpark Job
```
try:
    import os
    import sys
    from pyspark.sql import SparkSession
    from pyspark.sql.functions import col, to_date, date_format
    from pyspark.sql.types import *
    from pyspark.sql.functions import *
except Exception as e:
    print(e)

# Initialize SparkSession
spark = SparkSession.builder \
    .appName("Hudi Streaming Example") \
    .getOrCreate()

hudi_path = sys.argv[1]
check_point_path = sys.argv[2]

print(f"""
========================================
hudi_path {hudi_path}
check_point_path {check_point_path}
========================================
""")

# Read the Hudi stream
stream_df = spark.readStream \
    .format("hudi") \
    .load(hudi_path)

# Define batch processing logic
def process_batch(spark_df, batch_num):
    try:
        print("batch_num", batch_num)
        spark_df.show()  # Print the dataframe for debugging purposes
    except Exception as e:
        print(f"Error in processing batch {batch_num}: {e}")


# Write the stream to console using foreachBatch
query = stream_df.writeStream \
    .format("console") \
    .option("checkpointLocation", check_point_path) \
    .foreachBatch(process_batch) \
    .trigger(processingTime="1 minute").start()

# Wait for the stream to finish
query.awaitTermination()

```


# Submit jobs
```
export APPLICATION_ID="your-application-id"
export BUCKET_HUDI="your-bucket-name"
export IAM_ROLE="your-iam-role-arn"

aws emr-serverless start-job-run \
    --application-id $APPLICATION_ID \
    --name "HudiJobStreamingRun" \
    --mode 'STREAMING' \
    --execution-role-arn $IAM_ROLE \
    --job-driver '{
        "sparkSubmit": {
            "entryPoint": "s3://'$BUCKET_HUDI'/jobs/job.py",
            "entryPointArguments": ["s3://'$BUCKET_HUDI'/hudi/", "s3://'$BUCKET_HUDI'/hudi_checkpoint/"],
            "sparkSubmitParameters": "--conf spark.jars=/usr/lib/hudi/hudi-spark-bundle.jar --conf spark.serializer=org.apache.spark.serializer.KryoSerializer"
        }
    }' \
    --configuration-overrides '{
        "monitoringConfiguration": {
            "s3MonitoringConfiguration": {
                "logUri": "s3://'$BUCKET_HUDI'/logs/"
            }
        }
    }'

```

![image](https://github.com/user-attachments/assets/157d6b46-764c-4b1f-819c-6933184e8836)
![image](https://github.com/user-attachments/assets/70543a5d-7ace-4825-be44-26c71c84fea4)

