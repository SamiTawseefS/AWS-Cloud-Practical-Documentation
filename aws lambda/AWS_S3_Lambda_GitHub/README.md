# AWS S3 → Lambda Complete Documentation

This README documents two event-driven AWS implementations for processing files uploaded to Amazon S3.

- **Direct S3 → Lambda → CloudWatch Logs**
- **S3 → EventBridge → Lambda → CloudWatch Logs**

## Architecture Comparison

| Architecture | Flow |
|---|---|
| Direct S3 → Lambda | Upload file → S3 ObjectCreated event → Lambda → CloudWatch Logs |
| S3 → EventBridge → Lambda | Upload file → S3 Object Created event → EventBridge rule → Lambda → CloudWatch Logs |

---

AWS S3 → Lambda Complete Documentation

Direct S3 Trigger and S3 → EventBridge → Lambda Architecture

## 1. Introduction

This document combines two AWS event-driven implementations for processing files uploaded to Amazon S3. The first implementation uses a direct S3-to-Lambda trigger. The second implementation uses Amazon EventBridge between S3 and Lambda. Both implementations use Python Lambda functions and Amazon CloudWatch Logs for verification.

## 2. Architecture Comparison

Direct S3 → Lambda:
Upload file → S3 ObjectCreated event → Lambda → CloudWatch Logs

S3 → EventBridge → Lambda:
Upload file → S3 Object Created event → EventBridge rule → Lambda → CloudWatch Logs

PART A — Direct S3 → Lambda

## 3. Direct S3 → Lambda Overview

In this architecture, Amazon S3 directly invokes the Lambda function whenever an object is created in the configured bucket.

Upload test.txt to S3
 ↓
S3 ObjectCreated event
 ↓
S3 directly invokes Lambda
 ↓
Lambda reads the S3 event
 ↓
CloudWatch Logs displays the result

## 4. Create the S3 Bucket

Open the AWS Management Console.

Search for S3.

Click Create bucket.

Enter a unique bucket name.

Keep the required/default settings.

Click Create bucket.

## 5. Create the Lambda Function

Open AWS Lambda.

Select Functions and click Create function.

Choose Author from scratch.

Enter a function name, for example: myapp.

Select a Python 3.x runtime.

Use a basic Lambda execution role.

Click Create function.

## 6. Direct S3 Lambda Code

For the direct S3 trigger, the event structure is read from event['Records'][0]['s3'].

import json
import urllib.parse


def lambda_handler(event, context):
 print("Lambda function started")

 try:
 record = event["Records"][0]

 bucket_name = record["s3"]["bucket"]["name"]

 file_name = urllib.parse.unquote_plus(
 record["s3"]["object"]["key"],
 encoding="utf-8"
 )

 file_size = record["s3"]["object"].get("size", 0)

 print("--------------------------------")
 print("File uploaded successfully")
 print("Bucket name:", bucket_name)
 print("File name:", file_name)
 print("File size:", file_size, "bytes")
 print("--------------------------------")

 return {
 "statusCode": 200,
 "body": json.dumps({
 "message": "S3 event processed successfully",
 "bucket": bucket_name,
 "file_name": file_name,
 "file_size": file_size
 })
 }

 except KeyError as error:
 print("Invalid S3 event")
 print("Missing key:", str(error))

 return {
 "statusCode": 400,
 "body": json.dumps({
 "message": "Invalid S3 event",
 "missing_key": str(error)
 })
 }

 except Exception as error:
 print("Unexpected error:", str(error))

 return {
 "statusCode": 500,
 "body": json.dumps({
 "message": "Lambda execution failed",
 "error": str(error)
 })
 }

## 7. Add S3 as the Lambda Trigger

Open the Lambda function.

Select Configuration → Triggers.

Click Add trigger.

Select S3.

Select the S3 bucket.

Select All object create events.

Optionally configure a .txt suffix/filter.

Complete the acknowledgement if AWS displays one.

Click Add.

## 8. Test Direct S3 → Lambda

Open the S3 bucket.

Click Upload.

Upload test.txt.

Open Lambda → Monitor → View CloudWatch logs.

Open the latest log stream and verify the bucket, file name, and file size.

Direct S3 trigger configuration.

CloudWatch Logs for the Lambda execution.

PART B — S3 → EventBridge → Lambda

## 9. EventBridge Architecture Overview

In this architecture, S3 sends object events to Amazon EventBridge. An EventBridge rule filters the event and invokes the Lambda function when the event matches the configured bucket and .txt suffix.

Upload test.txt to S3
 ↓
S3 creates Object Created event
 ↓
EventBridge receives event
 ↓
EventBridge rule matches bucket + .txt
 ↓
Lambda function is invoked
 ↓
CloudWatch Logs displays result

## 10. Enable EventBridge Notifications on S3

Open S3 and select the bucket.

Open the Properties tab.

Scroll to Amazon EventBridge.

Enable Send notifications to Amazon EventBridge for all events in this bucket.

Make sure the setting shows On.

Amazon EventBridge notifications enabled for the S3 bucket.

## 11. Create the Lambda Function

Open AWS Lambda.

Select Functions → Create function.

Choose Author from scratch.

Enter the function name, for example: myapp.

Select a Python 3.x runtime.

Use a basic Lambda execution role.

Click Create function.

## 12. EventBridge Lambda Code

For an EventBridge S3 event, the event information is available under the event detail object. If the rule is configured for the standard EventBridge S3 Object Created event, use the corresponding EventBridge event structure.

import json
import urllib.parse


def lambda_handler(event, context):
 print("Lambda started from EventBridge")
 print("Received event:")
 print(json.dumps(event))

 try:
 bucket_name = event["detail"]["bucket"]["name"]

 file_name = urllib.parse.unquote_plus(
 event["detail"]["object"]["key"],
 encoding="utf-8"
 )

 file_size = event["detail"]["object"].get("size", 0)

 print("--------------------------------")
 print("TXT file uploaded successfully")
 print("Bucket name:", bucket_name)
 print("File name:", file_name)
 print("File size:", file_size, "bytes")
 print("--------------------------------")

 return {
 "statusCode": 200,
 "body": json.dumps({
 "message": "S3 event processed through EventBridge",
 "bucket": bucket_name,
 "file_name": file_name,
 "file_size": file_size
 })
 }

 except KeyError as error:
 print("Invalid EventBridge S3 event")
 print("Missing key:", str(error))

 return {
 "statusCode": 400,
 "body": json.dumps({
 "message": "Invalid EventBridge event",
 "missing_key": str(error)
 })
 }

 except Exception as error:
 print("Unexpected error:", str(error))

 return {
 "statusCode": 500,
 "body": json.dumps({
 "message": "Lambda execution failed",
 "error": str(error)
 })
 }

Click Deploy after adding the code.

Lambda function code configured in the AWS Console.

## 13. Create the EventBridge Rule

Open Amazon EventBridge.

Select Rules.

Click Create rule.

Enter a rule name, for example: myrule.

Select the default event bus.

Continue to the event pattern configuration.

EventBridge rule configured on the default event bus.

## 14. Configure the EventBridge Event Pattern

Use the following rule pattern and replace the bucket name if necessary:

{
 "source": ["aws.s3"],
 "detail-type": ["Object Created"],
 "detail": {
 "bucket": {
 "name": ["student-s3-eventbridge-demo-2026-unique"]
 },
 "object": {
 "key": [
 {
 "suffix": ".txt"
 }
 ]
 }
 }
}

The rule matches:

Amazon S3 events.

Object Created events.

Objects in the specified bucket.

Object keys ending with .txt.

## 15. Add Lambda as the EventBridge Target

In the EventBridge rule, configure the target.

Select AWS service.

Select Lambda function.

Choose the myapp Lambda function.

Review the configuration.

Click Create rule.

## 16. Test the EventBridge Workflow

Open the S3 bucket.

Click Upload.

Upload a .txt file such as i am billa.txt.

Wait for the EventBridge rule to process the event.

Open Lambda → Monitor → View CloudWatch logs.

Open the latest log stream.

CloudWatch Logs showing the EventBridge S3 event and TXT file details.

## 17. Direct S3 vs EventBridge

Direct S3 → Lambda uses the S3 notification event structure, typically accessed through event['Records'][0]['s3']. EventBridge S3 events use the EventBridge event structure, with bucket and object information under event['detail'].

Do not mix the two event formats. The Lambda code must match the trigger architecture.

## 18. Final Result

Direct architecture:
S3 → Lambda → CloudWatch Logs

EventBridge architecture:
S3 → EventBridge → Lambda → CloudWatch Logs

## 19. Conclusion

Both architectures provide event-driven file processing. The direct S3-to-Lambda method is simpler, while EventBridge provides an event-routing layer where rules can filter and route events to targets. The EventBridge implementation in this project filters S3 Object Created events for the selected bucket and .txt files.
