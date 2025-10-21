# Automated Instance Management Using AWS Lambda and Boto3

# Ec2 Setup:
Created 2 EC2 instances with tagging.
Auto-Start Instance
<img width="1221" height="451" alt="image" src="https://github.com/user-attachments/assets/7bcd863a-57b8-4ac4-a3ca-bc9604156bb6" />

Auto-Stop Instance
<img width="1225" height="505" alt="image" src="https://github.com/user-attachments/assets/a6727543-d0ef-48dd-a098-e25515c195f6" />

# Lambda Function Creation:

Creating the new lambda function.
<img width="1456" height="685" alt="image" src="https://github.com/user-attachments/assets/0e5829ec-4ea9-4dc9-85e6-e8b87bf13747" />

A list of permissions is attached to the role, which is attached to the Lambda function.
<img width="1500" height="703" alt="image" src="https://github.com/user-attachments/assets/16e638fc-e301-4691-b15d-daa71fa87e81" />

# Coding:
   - Using Boto3 in the Lambda function:
   - Detect all EC2 instances with the `Auto-Stop` tag and stop them.
   - Detect all EC2 instances with the `Auto-Start` tag and start them.

# Boto3 function:
import boto3

def lambda_handler(event, context):
    ec2 = boto3.client('ec2')

    # ---- Handle Auto-Stop instances ----
    stop_filters = [{'Name': 'tag:Action', 'Values': ['Auto-Stop']}]
    stop_response = ec2.describe_instances(Filters=stop_filters)
    
    stop_instances = []
    for reservation in stop_response['Reservations']:
        for instance in reservation['Instances']:
            if instance['State']['Name'] != 'stopped':
                stop_instances.append(instance['InstanceId'])

    if stop_instances:
        print("Stopping instances:", stop_instances)
        ec2.stop_instances(InstanceIds=stop_instances)
    else:
        print("No instances to stop.")

    # ---- Handle Auto-Start instances ----
    start_filters = [{'Name': 'tag:Action', 'Values': ['Auto-Start']}]
    start_response = ec2.describe_instances(Filters=start_filters)
    
    start_instances = []
    for reservation in start_response['Reservations']:
        for instance in reservation['Instances']:
            if instance['State']['Name'] == 'stopped':
                start_instances.append(instance['InstanceId'])

    if start_instances:
        print("Starting instances:", start_instances)
        ec2.start_instances(InstanceIds=start_instances)
    else:
        print("No instances to start.")

    return {
        'statusCode': 200,
        'body': f"Started: {start_instances}, Stopped: {stop_instances}"
    }

# Configure Lambda Settings
Timeout:
Click Configuration → General Configuration → Edit.
Set Timeout to 1 minute.
<img width="1390" height="642" alt="image" src="https://github.com/user-attachments/assets/49df415d-a6fa-4786-8193-598a2c769431" />

# Manual Invocation & Testing
Make sure to follow the steps below:
Your Auto-Stop instance is running. - Instance named as stop, which is in running state
Your Auto-Start instance is stopped. - Instance named as start, which is in a stopped state.
<img width="1646" height="80" alt="image" src="https://github.com/user-attachments/assets/dd92716f-b66f-4b08-b216-799179476113" />

# Go to your Lambda function and click Test.
<img width="1336" height="676" alt="image" src="https://github.com/user-attachments/assets/10e91802-19be-4145-b78e-a93237ed8717" />

# CloudWatch Logs
<img width="1487" height="437" alt="image" src="https://github.com/user-attachments/assets/27de1ce5-d714-4784-bb75-dc6d6cfd5c44" />

After the manual invocation, the Stop instance is stopped and the Start instance is started.
<img width="1650" height="87" alt="image" src="https://github.com/user-attachments/assets/c5499311-a87e-42da-867d-844551f77fa0" />

# Summary:
. This assignment demonstrates automated EC2 instance management using AWS Lambda and Boto3. <br>
. Based on the AWS tags, we can start or stop the instances for cost savings. <br>

