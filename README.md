<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# Fetch Data with AWS Lambda

**Project Link:** [View Project](https://nextwork.ai/projects/465096ce-f136-53fb-8a99-ebab7c406cd9)

**Author:** Maria Jasmin Ahorro  
**Email:** jasminahorro23@gmail.com

---

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/465096ce-f136-53fb-8a99-ebab7c406cd9_p9thryj2)

## Introducing Today's Project!

In this project, I will demonstrate how to retrieve DynamoDB table data using an AWS Lambda function. I'm doing this project to learn how to build serverless backend architectures and work with AWS cloud services.

### Tools and concepts

Services I used were Amazon DynamoDB and AWS Lambda. Key concepts I learnt include Lambda functions, execution roles, IAM permission policies, the AWS SDK, and serverless architecture.

### Project reflection

This project took me approximately 20 minutes to complete. The most challenging part was troubleshooting the AccessDeniedException error during Lambda testing and understanding how IAM execution roles restrict access to DynamoDB tables. It was most rewarding to successfully attach the AmazonDynamoDBReadOnlyAccess policy, resolve the permission issue, and watch the Lambda function seamlessly query and retrieve user data from DynamoDB without managing any servers!

I chose to do this project today because I wanted to gain hands-on experience building serverless cloud architectures and understand how to connect AWS Lambda functions securely to a DynamoDB table. Something that would make learning with NextWork even better is having more interactive troubleshooting sandboxes or step-by-step architecture diagrams that illustrate how different AWS services communicate under the hood during complex workflows!

## Project Setup

To set up my project, I created a database using Amazon DynamoDB named UserData. The partition key is userId, which means it acts as a unique label and the primary identifier used to organize and distribute data across servers for fast and efficient querying.

In my DynamoDB table, I added a sample user item with a userId of 1, a name of Test User, and an email address. DynamoDB is schemaless, which means you do not need to pre-define attributes or columns before adding data, and every item in your table can have a unique set of attributes.

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/465096ce-f136-53fb-8a99-ebab7c406cd9_a112c3d5)

### AWS Lambda

AWS Lambda is a serverless compute service that lets you run code without managing servers, and I'm using Lambda in this project to create a function that handles incoming requests and retrieves data from our DynamoDB table on demand.

## AWS Lambda Function

My Lambda function has an execution role, which is an IAM role automatically created by AWS for the function. By default, the role grants basic permissions to write logs to Amazon CloudWatch Logs.

My Lambda function will query the DynamoDB table to retrieve user data based on the incoming event's user ID and return the matching record.

The code uses AWS SDK, which is a software development kit that provides pre-written libraries and tools to interact with AWS services easily, and my code uses the SDK to communicate with DynamoDB so we can retrieve table data without writing complex custom API requests from scratch.

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/465096ce-f136-53fb-8a99-ebab7c406cd9_a1b2c3d5)

## Function Testing

To test whether my Lambda function works, I created a test event in the Lambda console. The test is written in JSON format with a userId of 1. If the test is successful, I'd see a successful execution response displaying the user data retrieved from our DynamoDB table.

The test displayed a 'success' because the Lambda function executed successfully without syntax or runtime code errors, but the function's response was actually an AccessDeniedException because the execution role lacked the necessary IAM permissions to perform the dynamodb:GetItem action on the UserData table.

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/465096ce-f136-53fb-8a99-ebab7c406cd9_u1v2w3x4)

## Function Permissions

To resolve the AccessDenied error, we attached the AmazonDynamoDBReadOnlyAccess policy to our execution role because our Lambda function specifically requires read and query access to retrieve items from the DynamoDB table.

There were four DynamoDB permission policies I could choose from, but I didn't pick AmazonDynamoDBFullAccess because it grants unnecessary permissions to create, update, and delete tables and data, which violates the principle of least privilege since our Lambda function only needs to read records.

I also didn't pick AWSLambdaDynamoDBExecutionRole or AWSLambdaInvocation-DynamoDB, because they're designed for working with DynamoDB Streams (reading a feed of table changes and triggering functions from it), not for querying items. AmazonDynamoDBReadOnlyAccess was the right choice because our Lambda function only needs read and query permissions to fetch user records.

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/465096ce-f136-53fb-8a99-ebab7c406cd9_3ethryj2)

## Final Testing and Reflection

To validate my new permission settings, I re-tested the Lambda function. The results were successful because attaching the AmazonDynamoDBReadOnlyAccess policy resolved the AccessDeniedException error, allowing the function to successfully retrieve the user data from our DynamoDB table.

Web apps are a popular use case of using Lambda and DynamoDB. For example, I could build a backend service for an e-commerce platform where Lambda fetches product details and order history from DynamoDB on demand, or create a profile-fetching service for a social media app that retrieves user data seamlessly.

![Image](https://nextwork.ai/overjoyed_magenta_proud_yak/uploads/465096ce-f136-53fb-8a99-ebab7c406cd9_p9thryj2)

---

*Built with [NextWork](https://nextwork.ai) - [View this project](https://nextwork.ai/projects/465096ce-f136-53fb-8a99-ebab7c406cd9)*
