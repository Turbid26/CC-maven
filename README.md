# CC-maven

Lambda-DynamoDB:
1.Create s3 bucket with versioning enabled and dynamodb table.
2.Create lambda function with roles and perms
3. add s3 trigger
import boto3
from uuid import uuid4

def lambda_handler(event, context):

    dynamodb = boto3.resource('dynamodb')
    table = dynamodb.Table('Newtable')

    if 'Records' in event:
        for record in event['Records']:

            bucket_name = record['s3']['bucket']['name']
            object_key = record['s3']['object']['key']
            size = record['s3']['object'].get('size', -1)
            event_name = record.get('eventName', 'Unknown')
            event_time = record.get('eventTime', 'Unknown')

            table.put_item(
                Item={
                    'private': str(uuid4()),
                    'Bucket': bucket_name,
                    'Object': object_key,
                    'Size': size,
                    'Event': event_name,
                    'EventTime': event_time
                }
            )

    return {
        'statusCode': 200,
        'body': 'Data stored in DynamoDB'
    }

SNS:
1. create topic, standard, create subscription type email and confirm in email.
2. create s3 bucket, same region as sns, default settings
3. edit access policy
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowS3Publish",
      "Effect": "Allow",
      "Principal": {
        "Service": "s3.amazonaws.com"
      },
      "Action": "SNS:Publish",
      "Resource": "arn:aws:sns:ap-south-1:829289607642:myemailtopic",
      "Condition": {
        "ArnLike": {
          "aws:SourceArn": "arn:aws:s3:::sns-testbucket-week8"
        }
      }
    }
  ]
}
4. go to s3 bucket, properties, event notifications, create with sns topic

sns sqs lambda:
1.create s3, sqs, sns
2. go to sns, create subscription sqs protocol, endpoint sqs.
3. edit sqs access policy
{
"Statement": [
{
"Effect": "Allow",
"Principal": {
"Service": "sns.amazonaws.com"
},
"Action": "sqs:SendMessage",
"Resource": "SQS ARN",
"Condition": {
"ArnEquals": {
"aws:SourceArn": "SNS ARN"
}
}
}
]
}
4. edit sns access policy
{
"Version": "2012-10-17",
"Id": "example-ID",
"Statement": [
{
"Sid": "Example SNS topic policy",
"Effect": "Allow",
"Principal": {
"Service": "s3.amazonaws.com"
},
"Action": [
"SNS:Publish"
],
"Resource": "SNS topic ARN",
"Condition": {
"ArnLike": {
"aws:SourceArn": "S3 bucket ARN"
},
"StringEquals": {
"aws:SourceAccount": "Account ID"
}
}
}
]
}
5. create event notification in s3 with destination as sns. check if message appears in sqs on upload.
6. create lambda function with sqs trigger
for record in event['Records']:
print("Message received from SQS:")
print(record['body'])

Elastic Load Balancer
1. create 2 ec2 instances with SSH(22) everywhere and http(80) everywhere. connect and run:
sudo yum update -y
sudo yum install httpd -y
sudo systemctl start httpd
sudo systemctl enable httpd
sudo su
echo "This is Server 1/2" > /var/www/html/index.htmlselect e
2.create security group lb-sg with http(80) anywhere
3.create application load balancer. scheme:Internet facing, listeners http(80), default vpc and all subnets. security group lb-sg
4. new target group, instance type, http(80), include as pending.
5. go to dns of alb

Auto Scaling
1.select ec2 instance. actions->image->create image. wait till ami available.
2.ec2->instances->launch templates->create launch template. existing keypair, security group lb-sg.
3. create autoscaling group, default vpc, all subnets, attack to existing load balancer, cap 2 min 1 max 4

Beanstalk:
1.create beanstalk (new version), tomcat, 9 11, upload war file
2.give roles, default vpc, all subnets, enable public ip

Lex:
1.create blank bot with new role and all default. add utterances and slots for age, city, nights, date
2.click ohnh intents, slot types, add blank slot type for roomtype. add values single double, suite and save. create new slot.
3. add response cards card groups
4. add initial and confirmation

IAM user:
1. create new iam user s3-specialist with fulls3access. provide user access to mangeent console, custom password, attack policies directly. download csv file and login to new user.
2.in root, go to security credentials, create access key, cli, download key
aws configure
aws ec2 describe-instances
aws s3 ls
aws s3 mb s3://bucket-name to create

IAM role:
1. create iam role, aws service, ec2, amazons3fullaccess
2. connect to ec2 instance. aws s3 ls.
3. instances->actions->security->modify iam role and attach to instance. then run aws s3 ls
