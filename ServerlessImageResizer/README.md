# Table of Contents

1. [Info about the tools used](#info-about-the-tools-used)
   - [S3](#s3)
   - [SSM](#ssm)
     - [Parameters Used](#parameters-used)
     - [Parameter Types in SSM](#parameter-types-in-ssm)
     - [Benefits](#benefits)
     - [Local Development](#local-development)
     - [Working with Parameters via AWS CLI](#working-with-parameters-via-aws-cli)
       - [Putting Parameters](#putting-parameters)
       - [Getting Parameters](#getting-parameters)
       - [Finding Parameters by Path](#finding-parameters-by-path)
   - [Lambda](#lambda)

# Info about the tools used:

## S3

## SSM

AWS Systems Manager Parameter Store (SSM) is used in this project to manage configuration values securely and efficiently. 

### Parameters Used:
- `/thumbnail-app/images-bucket`: Name of the S3 bucket storing original images
  - Type: String
  - Tier: Standard
- `/thumbnail-app/resized-bucket`: Name of the S3 bucket storing resized images
  - Type: String
  - Tier: Standard

### Parameter Types in SSM
SSM Parameter Store supports three parameter types:
1. **String**
   - Basic string values
   - Used for plain text configuration
   - Example: bucket names, region codes, API endpoints

2. **StringList**
   - Comma-separated list of values
   - Used for arrays or multiple related values
   - Example: list of allowed IP addresses, supported regions

3. **SecureString**
   - Encrypted string values using AWS KMS
   - Used for sensitive data
   - Example: passwords, API keys, credentials
   - Automatically encrypted/decrypted when accessed

### Benefits:
1. **Configuration Management**
   - Stores bucket names and other config values outside of code
   - Follows infrastructure-as-code best practices
   - Easy to update values without code changes

2. **Security**
   - Parameters can be encrypted if needed
   - Access controlled via IAM policies
   - Prevents hardcoding sensitive values in code

3. **Environment Management**
   - Simplifies management of different environments (dev, staging, prod)
   - Parameters can be environment-specific

### Local Development
Parameters can be added using the Makefile command:

### Working with Parameters via AWS CLI

#### Putting Parameters

- **String Parameter**
  ```bash
  aws ssm put-parameter \
      --name "/thumbnail-app/images-bucket" \
      --value "my-bucket-name" \
      --type String \
      --region us-east-1
  ```

- **StringList Parameter**
  ```bash
  aws ssm put-parameter \
      --name "/thumbnail-app/allowed-sizes" \
      --value "100x100,200x200,300x300" \
      --type StringList \
      --region us-east-1
  ```

- **SecureString Parameter**
  ```bash
  aws ssm put-parameter \
      --name "/thumbnail-app/api-key" \
      --value "secret-api-key" \
      --type SecureString \
      --region us-east-1
  ```

#### Getting Parameters

- **Get a Single Parameter**
  ```bash
  aws ssm get-parameter \
      --name "/thumbnail-app/images-bucket" \
      --region us-east-1
  ```

- **Get a SecureString Parameter with Decryption**
  ```bash
  aws ssm get-parameter \
      --name "/thumbnail-app/api-key" \
      --with-decryption \
      --region us-east-1
  ```

- **Get Multiple Parameters**
  ```bash
  aws ssm get-parameters \
      --names "/thumbnail-app/images-bucket" "/thumbnail-app/resized-bucket" \
      --region us-east-1
  ```

- **Get Multiple Parameters with Decryption**
  ```bash
  aws ssm get-parameters \
      --names "/thumbnail-app/images-bucket" "/thumbnail-app/api-key" \
      --with-decryption \
      --region us-east-1
  ```

#### Finding Parameters by Path

- **Get Parameters by Path**
  ```bash
  aws ssm get-parameters-by-path \
      --path "/thumbnail-app/" \
      --region us-east-1 \
      --recursive
  ```

- **Get Parameters by Path with Decryption**
  ```bash
  aws ssm get-parameters-by-path \
      --path "/thumbnail-app/" \
      --with-decryption \
      --region us-east-1 \
      --recursive
  ```

## Lambda

## SNS (Simple Notification Service)
- Message publishing and processing service (PubSub)
- Used to send messages to subscribers (email, SQS, Lambda)
- DLQ (Dead Letter Queue)
  - Queue for messages that cannot be processed successfully
  - Messages can be republished or discarded after a certain number of retries
- Allows fanout to multiple subscribers(consumers like Email, HTTP Endpoint, SQS, Texting, Lamda)
- Fully managed and durable with automatic scaling
  - Fully managed by AWS
  - Durable and reliable message delivery
  - Automatic scaling of subscribers
- Consists of:
  - Topic: Central entity to publish messages
  - Subscriptions: Connects SNS to endpoints (email, http, https, sqs, application)
