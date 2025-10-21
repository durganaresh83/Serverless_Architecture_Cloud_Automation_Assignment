# Automated S3 Bucket Cleanup Using AWS Lambda and Boto3

# Automate the deletion of files older than 30 days in a specific S3 bucket.

# S3 Setup:
In the AWS Console, go to S3 → Buckets → Create bucket

Created the bucket in the same region as where my Lambda function is running.
<img width="777" height="292" alt="image" src="https://github.com/user-attachments/assets/f4dfeb54-42d5-4b96-96e7-86df1becda52" />

# Upload Files:
Upload multiple files to the bucket (e.g., text or image files).
<img width="1242" height="663" alt="image" src="https://github.com/user-attachments/assets/2394eca3-9c72-4e2a-8bae-34c21d55e5f2" />

To simulate “old files,” you can:
Temporarily change your system date and upload files -  I have tried this option but this is not working, so i have tried the below option to change the metadata with below commands

aws s3 cp "C:/Users/durga/Downloads/" s3://durga-s3-cleanup-bucket/ --recursive --metadata x-amz-meta-last-modified=2025-09-15T10:30:00Z --metadata-directive REPLACE

Upload some files and then manually edit their metadata (though AWS S3 records the true LastModified timestamp). Example one image is shown here.
<img width="1375" height="257" alt="image" src="https://github.com/user-attachments/assets/23c23918-925a-402f-ba09-90dd9895a046" />

# Create IAM Role for Lambda
Go to IAM → Roles → Create role.
Choose Trusted entity type: AWS Service.
Choose Use case: Lambda.
Click Next → Permissions.
Attach the following policies:
  AmazonS3FullAccess
  CloudWatchLogsFullAccess for viewing logs.

Role name: durga-lab-1
<img width="1152" height="706" alt="Screenshot 2025-10-21 154629" src="https://github.com/user-attachments/assets/b79c45ab-6bfd-401d-ab50-35c724845906" />

# Create the Lambda Function
Go to AWS Lambda → Create function.
Configure:
Name: **durga-S3_Bucket_Cleanup**
Runtime: Python 3.12
Permissions: Use existing role → select the role.
<img width="1611" height="723" alt="image" src="https://github.com/user-attachments/assets/b1ef133f-4b27-4a8c-a251-44c02641c118" />

# Write the Boto3 Python Script
Update the code in the editor and in the Code tab, and click Deploy to save changes.

import boto3
from datetime import datetime, timezone, timedelta

def lambda_handler(event, context):
    bucket_name = 'durga-s3-cleanup-bucket'
    days_threshold = 30

    s3 = boto3.client('s3')
    threshold_date = datetime.now(timezone.utc) - timedelta(days=days_threshold)
    print(f"Deleting files with 'x-amz-meta-last-modified' >= {threshold_date}")

    response = s3.list_objects_v2(Bucket=bucket_name)

    if 'Contents' not in response:
        print("No objects found in the bucket.")
        return {"status": "No files found"}

    deleted_files = []
    for obj in response['Contents']:
        key = obj['Key']
        head_obj = s3.head_object(Bucket=bucket_name, Key=key)
        metadata = head_obj.get('Metadata', {})

        # Read and parse custom metadata date (if present)
        meta_last_modified = metadata.get('x-amz-meta-last-modified')
        if not meta_last_modified:
            print(f"Skipping {key}: 'x-amz-meta-last-modified' metadata not found.")
            continue

        try:
            meta_last_modified_dt = datetime.strptime(meta_last_modified, '%Y-%m-%dT%H:%M:%SZ').replace(tzinfo=timezone.utc)
        except ValueError:
            print(f"Skipping {key}: Invalid date format in metadata.")
            continue

        if meta_last_modified_dt < threshold_date:
            print(f"Deleting {key}: metadata date is {meta_last_modified_dt} (< threshold)")
            s3.delete_object(Bucket=bucket_name, Key=key)
            deleted_files.append(key)
        else:
            print(f"Keeping {key}: metadata date {meta_last_modified_dt} is greater than or equal to threshold.")

    print(f"Deleted files: {deleted_files}")
    return {
        'statusCode': 200,
        'body': f"Deleted files: {deleted_files}"
    }

# Configure Lambda Settings
In Lambda → Configuration → General Configuration, click Edit.
Increase the Timeout to at least 15 minutes (to handle large buckets).
<img width="1476" height="655" alt="image" src="https://github.com/user-attachments/assets/02199681-55a7-4e75-893f-1af4b7f4773c" />

# Manual Invocation & Testing
In the Test tab, create a new event "test_delete"
<img width="1742" height="616" alt="image" src="https://github.com/user-attachments/assets/cd8cb166-6748-4707-a887-ee5940cbf8ae" />

Open CloudWatch Logs → find your Lambda log group.
<img width="1460" height="537" alt="image" src="https://github.com/user-attachments/assets/38312783-714f-4bf0-a06e-3feacc906804" />

Go to your S3 bucket in the console.
Refresh → confirm that only recent files remain.
<img width="1485" height="692" alt="image" src="https://github.com/user-attachments/assets/8eb418a3-6108-4fca-994c-dfe6413c4a60" />


# Summary:
&#8226; The Lambda function deletes S3 bucket objects older than 30 days automatically.<br>
&#8226; The function was manually invoked from the AWS Lambda console to verify file deletion.<br>
&#8226; Files older than 30 days were deleted, and recent files remained as expected.<br>










