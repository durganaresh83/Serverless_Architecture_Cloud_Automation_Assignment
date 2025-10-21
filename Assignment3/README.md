# Assignment 3: Monitor Unencrypted S3 Buckets Using AWS Lambda and Boto3

Create an AWS Lambda function that automatically checks all S3 buckets in your account and identifies which ones do not have server-side encryption (SSE) enabled.

# S3 Setup:
. Go to the AWS Management Console → S3. <br>
. Click Create bucket and make a few buckets:<br>
Example names:<br>
 . durga-secure-bucket-demo (enable default encryption)<br>
 . durga-public-bucket-demo (do not enable encryption)<br>

For some buckets, under “Default encryption”, choose: "Enable encryption" <br>
For others, disable encryption using the AWS command below (to simulate the insecure case).<br>
. aws s3api delete-bucket-encryption --bucket bucket-name <br>

# Create IAM Role for Lambda
. Go to IAM → Roles → Create role.<br>
Choose:
Trusted entity type: AWS Service.<br>
Use case: Lambda.
Click Next → Permissions.<br>
Attach the policy:
. AmazonS3ReadOnlyAccess <br>
. CloudWatchLogsFullAccess (for log viewing) <br>
Click Next, name your role: <br>
Click Create Role. <br>

<img width="1272" height="602" alt="image" src="https://github.com/user-attachments/assets/79c8597f-8c86-4d05-b0da-c0b021bb8af0" />

# Step 3: Create the Lambda Function
Go to AWS Lambda → Create function.<br>
Choose:<br>
Function name: durga_Monitor_Unencrypted_S3_Buckets<br>
Runtime: Python 3.12 (or latest)<br>
Permissions: Use existing role → select Lambda_S3_Encryption_Monitor_Role<br>
Click Create Function.<br>

<img width="1527" height="682" alt="image" src="https://github.com/user-attachments/assets/6d5640b3-140e-4b19-95c7-c82d2bfcc17c" />

# Boto3 Python Script
. Write the Boto3 code in the Code tab and click Deploy to save your changes.<br>

import boto3

def lambda_handler(event, context):
    s3 = boto3.client('s3')

    # ---- List all S3 buckets ----
    response = s3.list_buckets()
    buckets = [bucket['Name'] for bucket in response['Buckets']]
    print(f"Found {len(buckets)} buckets: {buckets}")

    unencrypted_buckets = []

    # ---- Check encryption status for each bucket ----
    for bucket in buckets:
        try:
            enc = s3.get_bucket_encryption(Bucket=bucket)
            rules = enc['ServerSideEncryptionConfiguration']['Rules']
            print(f"✅ {bucket} has encryption enabled: {rules}")
        except s3.exceptions.ClientError as e:
            error_code = e.response['Error']['Code']
            # If there is no encryption configuration, the error code is usually 'ServerSideEncryptionConfigurationNotFoundError'
            if error_code == 'ServerSideEncryptionConfigurationNotFoundError':
                print(f"❌ {bucket} does NOT have server-side encryption enabled.")
                unencrypted_buckets.append(bucket)
            else:
                print(f"⚠️ Could not check {bucket}: {error_code}")

    if unencrypted_buckets:
        print("Unencrypted buckets found:", unencrypted_buckets)
    else:
        print("✅ All buckets have server-side encryption enabled.")

    return {
        'statusCode': 200,
        'unencrypted_buckets': unencrypted_buckets
    }

# Code Explanation:
1.	Initializes S3 client using boto3.client('s3').<br>
2.	Lists all buckets using list_buckets().<br>
3.	For each bucket, calls get_bucket_encryption().<br>
4.	If the call fails with ServerSideEncryptionConfigurationNotFoundError, marks it as unencrypted.<br>
5.	Prints all unencrypted bucket names for logging in CloudWatch.<br>

# Configure Lambda Settings
In Configuration → General configuration → Edit, increase:
. Timeout: 1 minute.

<img width="1428" height="670" alt="image" src="https://github.com/user-attachments/assets/dbadd6e6-405e-4c64-b024-a4980c79ba58" />

Manual Invocation & Testing
. Go to the Test tab in Lambda. Provide the event name "encryp_1"
<img width="1128" height="712" alt="image" src="https://github.com/user-attachments/assets/8e6cadf7-af2a-444b-a189-a92ca76a8adc" />

. Open Monitor → Logs → View logs in CloudWatch.
<img width="1452" height="757" alt="image" src="https://github.com/user-attachments/assets/6af0efc8-6f4c-414b-b20b-e31d6112b5c5" />

# Summary:
. The Lambda function scans all S3 buckets and identifies those without server-side encryption enabled. <br>
. The function was manually invoked from the AWS Lambda console while reviewing CloudWatch logs for unencrypted bucket detection.<br>

**Note: Due to limited permissions, not able to delete the encryption settings for the S3 buckets.**






