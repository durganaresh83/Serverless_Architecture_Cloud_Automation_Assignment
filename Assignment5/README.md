# Assignment 5: Auto-Tagging EC2 Instances on Launch Using AWS Lambda and Boto3

To automatically apply tags to EC2 instances immediately upon launch, using a Lambda function triggered by CloudWatch Events (EventBridge).<br>

# EC2 Setup:
. Ensure your AWS account has permissions to launch EC2 instances.<br>
. No special setup is needed here yet — we’ll test this after configuring Lambda and EventBridge. <br>

# Create IAM Role for Lambda
. Go to IAM → Roles → Create Role.<br>
Choose:<br>
. Trusted entity: AWS Service<br>
. Use case: Lambda<br>
. Click Next → Permissions.<br>
. Attach the following policies:<br>
  . AmazonEC2FullAccess<br>
. Click Next<br>
. Click Create Role.<br>

<img width="1262" height="611" alt="image" src="https://github.com/user-attachments/assets/31043792-a6b4-41c5-8cc7-bad0348a6cad" />

# Create the Lambda Function
. Go to AWS Lambda → Create function. <br>
 Configure: <br>
. Function name: durga-AutoTag_EC2_On_Launch <br>
. Runtime: Python 3.12 (or latest) <br>
. Permissions: Use existing role → select Lambda_AutoTag_EC2_Role <br>
. Click Create Function. <br>

<img width="1238" height="727" alt="image" src="https://github.com/user-attachments/assets/6fd1e698-d785-4a75-b04b-5c56ea6d7856" />

# Boto3 Python Script
In the Lambda code editor, delete the default code and paste the following:

import boto3
from datetime import datetime

def lambda_handler(event, context):
    ec2 = boto3.client('ec2')

    # ---- Extract instance ID from the CloudWatch event ----
    try:
        instance_id = event['detail']['instance-id']
    except KeyError:
        print("No instance ID found in event data.")
        return {"status": "No instance ID in event"}

    # ---- Define tags ----
    current_date = datetime.utcnow().strftime("%Y-%m-%d")
    tags = [
        {'Key': 'LaunchDate', 'Value': current_date},
        {'Key': 'Environment', 'Value': 'Development'}  # You can change this custom tag
    ]

    # ---- Apply tags ----
    ec2.create_tags(Resources=[instance_id], Tags=tags)
    print(f"✅ Tagged instance {instance_id} with {tags}")

    return {
        'statusCode': 200,
        'body': f"Instance {instance_id} tagged successfully."
    }

# Code Explanation
  1.	Connects to the EC2 service using boto3.client('ec2').<br>
  2.	Extracts the instance ID from the CloudWatch event payload.<br>
  3.	Creates two tags — one with the current date, one custom (like “Environment”).<br>
  4.	Calls create_tags() to attach those tags to the launched instance.<br>
  5.	Logs confirmation to CloudWatch.<br>

# Configure Lambda Settings
 . In Configuration → General Configuration → Edit:<br>
 . Increase Timeout to 1 minute<br>
 
<img width="1295" height="747" alt="image" src="https://github.com/user-attachments/assets/d9c0f32a-4990-4fa5-bbca-8e6b8c1efeae" />

# Create CloudWatch Event Rule (Trigger)
. Now will create an EventBridge rule that triggers the Lambda when a new EC2 instance launches.<br>
. Go to EventBridge → Rules → Create Rule.
Choose:
. Rule name: durga-Trigger_Lambda_On_EC2_Launch

  <img width="1342" height="572" alt="image" src="https://github.com/user-attachments/assets/a98386b1-8ecf-4f67-9a18-c382d6900b97" />

. Event Source: AWS events or EventBridge

Event Pattern
 . Under Event Source, select AWS Services → EC2. <br>
 . Under Event Type, choose: "EC2 Instance State-change Notification" <br>
 . Scroll to Specific state(s) and select: "running"

 <img width="1342" height="713" alt="image" src="https://github.com/user-attachments/assets/1bcabf22-9252-4e61-a6ed-f44acbe32cf3" />

Click Next, add a Target:
 . Target type: Lambda function <br>
 . Function: AutoTag_EC2_On_Launch <br>

<img width="1581" height="742" alt="image" src="https://github.com/user-attachments/assets/b539ace8-21a1-4df4-a748-60a1afbfe4bb" />

# Testing
. Go to EC2 → Launch Instances. <br>
. Create a simple instance <br>
. Wait 30–60 seconds after it launches. <br>

<img width="1248" height="658" alt="image" src="https://github.com/user-attachments/assets/ea138148-87e5-4370-abf5-668bd1ffd79b" />

# EC2 instance
. Tags are attached to the EC2 instance after launch.

<img width="1343" height="762" alt="image" src="https://github.com/user-attachments/assets/3f39b7e4-ec35-4d16-8224-e85ffb6909af" />

# CLoudWatch logs

<img width="1481" height="350" alt="image" src="https://github.com/user-attachments/assets/81df3b63-7cde-4e29-85b0-740b8bd37dbd" />


# Summary:
. The Lambda function automatically tags newly launched EC2 instances with the current date and a custom tag.<br>
. A new EC2 instance was launched to trigger the Lambda via CloudWatch Events, and the instance tags were verified in the EC2 console. <br>
. The instance received both tags automatically as expected, confirming successful Lambda execution.

