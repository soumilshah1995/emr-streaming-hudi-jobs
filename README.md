# emr-streaming-hudi-jobs
emr-streaming-hudi-jobs
![image](https://github.com/user-attachments/assets/d0538cdd-5792-49be-955c-904e169d2270)

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
