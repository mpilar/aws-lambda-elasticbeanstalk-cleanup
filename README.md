aws-lambda-elasticbeanstalk-cleanup
============

A simple lambda function that cleans up your Elastic Beanstalk applications versions for an specific aws region, so you keep every Elastic Beanstalk application under the same limit and you stop manually deleting versions when you reach your region limit.

For each Elastic Beanstalk application in your region, this script will delete all the application versions that are above the limit.

# Using aws-lambda-elasticbeanstalk-cleanup

## Using AWS Serverless Application Model

You can install the Lambda function using the SAM definition included by running the following commands:

    aws cloudformation package --template-file CleanUpSAM.yaml --s3-bucket <package bucket> --output-template CleanUpSAM-transform.yaml --s3-prefix <prefix>
    aws cloudformation deploy --template-file CleanUpSAM-transform.yaml --stack-name <stack name> --capabilities CAPABILITY_IAM

This will create a Lambda function, IAM role and schedule. See below for the relevant info on environment variables for the lambda you can modify.

Note the argument `--capabilities CAPABILITY_IAM` is required to create the IAM role for the SAM package.

For more information on AWS SAM see [here](http://docs.aws.amazon.com/lambda/latest/dg/deploying-lambda-apps.html).

## Manual installation

### Create the Lambda function
Create a new Python Lambda Function in your aws console, and paste the code.

Set the Memory and Timeout values for your function, for example:

    Memory (MB): 128
    Timeout    : 1 min

And that's it, you can then configure how your function will be triggered, using a Scheduled Event or so :).

### Create the IAM Role

In order to let your lambda function clean up your application versions, you need a proper execution role, here is the one i use:

    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "logs:CreateLogGroup",
                    "logs:CreateLogStream",
                    "logs:PutLogEvents"
                ],
                "Resource": "arn:aws:logs:*:*:*"
            },
            {
                "Effect": "Allow",
                "Action": [
                    "elasticbeanstalk:DescribeApplications",
                    "elasticbeanstalk:DescribeApplicationVersions",
                    "elasticbeanstalk:DescribeEnvironments",
                    "elasticbeanstalk:DeleteApplicationVersion",
                    "s3:GetObject",
                    "s3:ListBucket",
                    "s3:DeleteObject"
                ],
                "Resource": "*"
            }
        ]
    }

## Environment Variables

The lambda function uses the following environment variables to control the region and how many versions per application to retain.

    VERSION_LIMIT = 25 # (Per Elastic Beanstalk application)
    REGION = us-west-2

## Considerations

There can be scenarios when a deployed application version is outside of your versions limit, this scripts takes that in consideration and it excludes deployed application versions from the clean up.

This script also deletes the version from S3.
