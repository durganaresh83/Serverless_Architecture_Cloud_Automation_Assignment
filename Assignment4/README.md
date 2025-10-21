# Automatic EBS Snapshot and Cleanup Using AWS Lambda and Boto3

To automate EBS volume backup creation and deletion of old snapshots (older than 30 days) using AWS Lambda and Boto3.

# EBS Setup
. Navigate to EC2 → Volumes.<br>
Either:
  . Use an existing EBS volume, or <br>
  . Click Create Volume:<br>
  . Type: gp2 or gp3<br>
  . Size: e.g., 8 GiB<br>
  . Availability Zone: same as your EC2 instance<br>
  . Click Create Volume<br>
  Once created, note the Volume ID (vol-040de799ecffe5ea8, vol-003e2b47e95230b3c, vol-06d5b27beeecb6d3d)<br>
  
  <img width="1648" height="238" alt="image" src="https://github.com/user-attachments/assets/07230598-d426-4875-b743-0e451b7f831b" />

# Create IAM Role for Lambda
. Go to IAM → Roles → Create Role. <br>
Choose: <br>
  .  Trusted entity: AWS Service <br>
  . Use case: Lambda <br>
  . Click Next → Permissions. <br>
  . Attach the following policies: <br>
    . AmazonEC2FullAccess <br>
. Click Next <br>
. Click Create Role. <br>

<img width="1181" height="627" alt="image" src="https://github.com/user-attachments/assets/dd61571e-b2e1-4b88-97c9-4ca54fbfe93e" />

# Create the Lambda Function
. Go to AWS Lambda → Create function.<br>
. Configure:<br>
    . Function name: durga-EBS_Snapshot_Auto_Backup<br>
    . Runtime: Python 3.12 (or latest)<br>
    . Permissions: Use existing role <br>

<img width="1342" height="660" alt="image" src="https://github.com/user-attachments/assets/644cc3e3-b587-448a-ab91-3c6b0631f8e6" />

# Boto3 Python Script
Write the Boto3 code in CodeTab and click on Deploy. <br>

import boto3
from datetime import datetime, timezone, timedelta

def lambda_handler(event, context):
    ec2 = boto3.client('ec2')

    # ---- Configuration ----
    volume_id = 'vol-06d5b27beeecb6d3d'  # Replace with your actual EBS volume ID
    retention_days = 30

    # ---- Step 1: Create Snapshot ----
    print(f"Creating snapshot for volume: {volume_id}")
    description = f"Automated snapshot for {volume_id} created on {datetime.now(timezone.utc)}"
    
    snapshot = ec2.create_snapshot(VolumeId=volume_id, Description=description)
    snapshot_id = snapshot['SnapshotId']
    print(f"✅ Created snapshot: {snapshot_id}")

    # ---- Step 2: Delete Snapshots older than retention period ----
    snapshots = ec2.describe_snapshots(Filters=[{'Name': 'volume-id', 'Values': [volume_id]}])['Snapshots']
    now = datetime.now(timezone.utc)
    delete_before_date = now - timedelta(days=retention_days)
    
    deleted_snapshots = []
    
    for snap in snapshots:
        start_time = snap['StartTime']
        if start_time < delete_before_date:
            old_snap_id = snap['SnapshotId']
            print(f"🗑️ Deleting old snapshot: {old_snap_id} (Created on {start_time})")
            ec2.delete_snapshot(SnapshotId=old_snap_id)
            deleted_snapshots.append(old_snap_id)

    print(f"Deleted snapshots: {deleted_snapshots}")
    return {
        'statusCode': 200,
        'body': f"Created snapshot: {snapshot_id}, Deleted snapshots: {deleted_snapshots}"
    }

# Code Explanation
  1. 	Connects to EC2 using Boto3.
  2.	Creates a new EBS snapshot using create_snapshot().
  3.	Fetches all snapshots for that volume using describe_snapshots().
  4.	Compares each snapshot’s StartTime with now - 30 days.
  5.	Deletes snapshots older than the retention threshold.
  6. 	Logs all created and deleted snapshot IDs for traceability.

# Configure Lambda Settings
 . Go to Configuration → General Configuration → Edit: <br>
 . Timeout: set to 2 minutes <br>

 <img width="1270" height="650" alt="image" src="https://github.com/user-attachments/assets/0c322abd-3b15-4308-a4f6-18190411ead5" />

 # Manual Invocation and Testing
  . Go to the Test tab in the Lambda console.
  . Click Test to manually invoke the function.
  
  <img width="1731" height="552" alt="image" src="https://github.com/user-attachments/assets/cee867ce-68b1-40f4-8301-6f6224768f26" />
  
  # Invocation result: 
  <img width="1460" height="702" alt="image" src="https://github.com/user-attachments/assets/c5b0419e-1eb9-4f36-8a4f-18322f07f656" />


  . Wait for a few seconds and check CloudWatch Logs: <br>
  <img width="1490" height="757" alt="image" src="https://github.com/user-attachments/assets/d14748c8-5a22-4b58-b2aa-16e6e01270a2" />

# Verify in EC2 → Snapshots:
A new snapshot should appear. Snapshot created from the provided volume ID - "vol-040de799ecffe5ea8"
<img width="1481" height="697" alt="image" src="https://github.com/user-attachments/assets/0b5f916c-c698-43e6-a783-9f025f3d436b" />

# Summary:
. The Lambda function creates EBS volume snapshots automatically and deletes snapshots older than 30 days to optimize storage costs.<br>
. The function was manually invoked from the AWS Lambda console, and snapshot creation/deletion was verified in the EC2 dashboard.<br>
. A new snapshot was successfully created, and older snapshots beyond the 30-day retention period were deleted as expected.<br>
