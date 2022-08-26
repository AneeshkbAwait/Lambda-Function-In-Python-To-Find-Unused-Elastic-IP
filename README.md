# Lambda-Function-In-Python-To-Find-Unused-Elastic-IP

AWS Lambda is a serverless compute service that runs your code in response to events and automatically manages the underlying compute resources for you.

Here, I have written a code in Python which will find any unused elastic IPs in your AWS infrastructure and it will send a notification based on the SNS topic which I created earlier. 

Later, I updated the code to work with AWS Lambda, and the Lambda function will trigger once in a week, which can be scheduled using AWS Event Bridge or CloudWatch service.

Steps involved in the complete setup:

##### 1. Create an SNS topic for Email alert and add a subscription for the topic.
![alt text](https://s3.ap-south-1.amazonaws.com/githubpjts.aneeshponnu.tech/SNS+Topic.PNG "SNS topic for Email ALert")
##### 2. Create a Lambda role with required EC2 and SNS privileges. 
Lambda function requires certain privileges at Ec2 and SNS topic services, so that it can fetch the required details and can execute the task coded in it. For that we have to create a Lambda role.
![alt text](https://s3.ap-south-1.amazonaws.com/githubpjts.aneeshponnu.tech/Lambda+Role.PNG "Lambda Role for Access")
##### 3. Create the Lambda function with the code written in Python.
Create a Lambda Function in AWS which should use Python as the runtime environment with the below Python code.
The function "lambda_unusedEip" will identify any elastic IPs not associated with any Ec2 instance and later it will send an email Notification for unused elastic IPs via SNS Notification Service.
```
import boto3

REGION = "ap-south-1"
TOPIC_ARN = "arn:aws:sns:ap-south-1:254356662539:Notify-Unused-Eip" # Note: Topic ARN
unused_eips = []

sns_client = boto3.client('sns',region_name=REGION )
ec2 = boto3.client('ec2',region_name=REGION )
response = ec2.describe_addresses()

def lambda_unusedEip(event, context):
# check if there is any existing elastic IP
  check = list(response['Addresses'])

  if not check:
    print("Elastic IP does not exist | Exiting program.....")
    exit()

# If eip is there, we will check if it is associated with instance or not
  for address in response['Addresses']:
    if 'InstanceId' in address:
        print('Elastic IP {} is associated with instance {}'.format(address['PublicIp'], address['InstanceId']))
    else:
        print('Elastic IP {} is unused and can be released'.format(address['PublicIp']))
        unused_eips.append("Unused elastic ip: {}".format(address['PublicIp']))

# Sending email Notification for unused Elastic Ips via SNS Notification Service   
  for unused in unused_eips:
    sns_client.publish(
        TopicArn=TOPIC_ARN, 
        Subject='Alert - Unused Elastic Ip To be dissociated',
        Message=str(unused)
      )
    return "success"
```
##### 4. Create a cron to execute the Lambda function once in a week.
Create a weekly schedule (cron) for the Lambda Function. Cron should be scheduled in either AWS Event Bridge or CloudWatch services, depending on your preference. I have used AWS Event Bridge for setting up cron.
![alt text](https://s3.ap-south-1.amazonaws.com/githubpjts.aneeshponnu.tech/Cron+Event.PNG "AWS Event Bridge")

I hope this topic is clear to everyone who read this. If you have anything to share with me on this topic, please get in touch with me.
