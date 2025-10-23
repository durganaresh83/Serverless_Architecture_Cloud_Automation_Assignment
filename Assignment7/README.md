# DynamoDB Item Change Alert Using AWS Lambda, Boto3, and SNS

# Objective: Automate the process to receive an alert whenever an item in a DynamoDB table gets updated.

# DynamoDB Setup:
. Go to the AWS Management Console → DynamoDB. <br>
. Click Create table.<br>
    . Table name: durga-table <br>
    . Partition key: StudentID (String) <br>

<img width="1498" height="643" alt="image" src="https://github.com/user-attachments/assets/ec96d19e-4b52-44de-9fea-ef5f741511ef" />
   
. Leave other settings default → click Create table.

<img width="1740" height="730" alt="image" src="https://github.com/user-attachments/assets/b082590c-f487-4c5f-9e90-b3466affcc7c" />

Once the table is created: <br>
  . Go to Explore table items → Create item. <br>

<img width="1282" height="551" alt="image" src="https://github.com/user-attachments/assets/0fdf853c-ca1a-4069-9be4-beb0be12fbf1" />

. Add a few sample items, for example:
{
  "StudentID": "S001",
  "Name": "Alice",
  "Score": 85
}
<img width="925" height="237" alt="image" src="https://github.com/user-attachments/assets/9ee4ed79-5a46-4f67-8150-34378b3d013a" />

# SNS Setup:
. Go to Amazon SNS → Topics → Create topic. <br>

. Choose Standard type. <br>

. Topic name: durga-dynamoDBUpdateAlert.

    . Click Create topic.
<img width="1522" height="587" alt="image" src="https://github.com/user-attachments/assets/642186dd-7e03-484e-952e-4b3e3a957532" />

. Under Subscriptions, click Create subscription.

. Protocol: Email

. Endpoint: Your email address

<img width="1701" height="692" alt="image" src="https://github.com/user-attachments/assets/88cab284-ab95-446a-87ed-fc613efddd1c" />

. Check your email inbox and confirm the subscription by clicking the link in the AWS confirmation email

<img width="811" height="357" alt="image" src="https://github.com/user-attachments/assets/891bef1f-d947-41f8-831d-9bd5f0a8f997" />

# IAM Role for Lambda

. Go to IAM → Roles → Create role.

. Trusted entity: AWS Service <br>
    . Use case: Lambda <br>

Attach the following policies: <br>
    . AWSLambdaBasicExecutionRole <br>
    . AmazonDynamoDBFullAccess <br>
    . AmazonSNSFullAccess <br>

. Provide the role name and create it.

<img width="1150" height="652" alt="image" src="https://github.com/user-attachments/assets/c2471815-d581-472e-84e5-8db6863288b8" />

# Lambda Function

. Go to AWS Lambda → Create function. <br>

. Author from scratch <br>

. Name: durga-dynamoDBUpdateNotifier <br>

. Runtime: Python 3.x <br>

. Role: Use existing role → Lambda_DynamoDB_SNS_Role <br>

<img width="1291" height="692" alt="image" src="https://github.com/user-attachments/assets/e17e4f80-f9b6-4c75-b0a0-f41054e9e1a1" />

In the code editor, update the Boto3 code and deploy the changes.

import boto3
import json
import os

sns_client = boto3.client('sns')
SNS_TOPIC_ARN = 'arn:aws:sns:eu-west-2:975050024946:durga-dynamoDBUpdateAlert'

def lambda_handler(event, context):
    for record in event['Records']:
        if record['eventName'] == 'MODIFY':
            old_image = record['dynamodb'].get('OldImage')
            new_image = record['dynamodb'].get('NewImage')

            message = {
                'TableName': record['eventSourceARN'].split('/')[1],
                'EventType': record['eventName'],
                'OldItem': old_image,
                'NewItem': new_image
            }

            # Publish to SNS
            sns_client.publish(
                TopicArn=SNS_TOPIC_ARN,
                Message=json.dumps(message, indent=2),
                Subject='DynamoDB Item Updated!'
            )

            print("Notification sent:", message)

    return {'statusCode': 200, 'body': 'Success'}

# Enable DynamoDB Stream
. Go to your DynamoDB table → Exports and streams → Manage stream.

. Enable stream and select:

. New and old images (so you can see before/after update data)

<img width="1731" height="421" alt="image" src="https://github.com/user-attachments/assets/28e4a9e2-536d-4937-9e99-87fd4786af9e" />

. Now link the stream to your Lambda:

. Go to DynamoDB → Exports and streams → Trigger.

. Click Create trigger → Select existing Lambda function.

. Click Create.

<img width="1663" height="451" alt="image" src="https://github.com/user-attachments/assets/0c2beff6-bbe3-4792-9987-1070cbd87b5b" />

# Testing:
. Go to DynamoDB → Explore table items.

. Edit one of the items (e.g., change "Score": 85 → "Score": 90) and click Save.

<img width="1337" height="412" alt="image" src="https://github.com/user-attachments/assets/a8c71cc5-406a-412e-9d5f-1ab67b13dcc1" />

. Wait a few seconds (5–10 seconds).

We should receive an email from SNS with the details of the modified item. <br>

<img width="628" height="725" alt="image" src="https://github.com/user-attachments/assets/f3dd1e27-9504-49a6-87c2-f99ff13ecd67" />

We can also go to CloudWatch Logs (under Lambda → Monitor → View logs) to verify that the function <br>

<img width="1147" height="721" alt="image" src="https://github.com/user-attachments/assets/8f02fe61-6097-46b2-889e-dbee1fb58d2c" />
