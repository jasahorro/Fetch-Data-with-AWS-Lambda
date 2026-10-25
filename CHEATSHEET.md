# Fetch Data with AWS Lambda — Cheat Sheet

Lesson notes for reviewing later. The project write-up is in [README.md](README.md).

## Contents

1. [What I built](#what-i-built)
2. [Key terms](#key-terms)
3. [DynamoDB and the partition key](#dynamodb-and-the-partition-key)
4. [Adding an item: JSON view and schemaless data](#adding-an-item-json-view-and-schemaless-data)
5. [Creating the Lambda function](#creating-the-lambda-function)
6. [Execution roles](#execution-roles)
7. [The code and the AWS SDK](#the-code-and-the-aws-sdk)
8. [Testing and the AccessDeniedException](#testing-and-the-accessdeniedexception)
9. [Choosing a permission policy](#choosing-a-permission-policy)
10. [Use cases](#use-cases)
11. [Build order and cleanup](#build-order-and-cleanup)
12. [Self-quiz](#self-quiz)

---

## What I built

- 🗄️ **A DynamoDB table** called `UserData`, with partition key `userId` and one sample item (`userId` 1, `Test User`, `test@example.com`).
- ⚡ **A Lambda function** (Node.js) that uses the **AWS SDK** to look up a user by `userId` and return their data.
- 🧪 **A test event** in the Lambda console. The first run returned an **AccessDeniedException**.
- 🔑 **The fix:** attaching **AmazonDynamoDBReadOnlyAccess** to the function's **execution role**, following least privilege.

This is the **data tier** of a three-tier architecture. The CloudFront project built the presentation tier, and the API Gateway project built the logic tier's front door.

```
  Test event { "userId": "1" }
        │
        ▼
  ┌──────────────────────┐    GetItem (AWS SDK)    ┌──────────────────────┐
  │ Lambda function      │ ──────────────────────► │ DynamoDB: UserData   │
  │ (Node.js + AWS SDK)  │                         │ partition key userId │
  │                      │ ◄────────────────────── │ { 1, Test User, ...} │
  └──────────┬───────────┘      the user item      └──────────────────────┘
             │ assumes
             ▼
  ┌──────────────────────────────────────────┐
  │ Execution role (IAM)                     │
  │ • basic CloudWatch Logs (default)        │
  │ • AmazonDynamoDBReadOnlyAccess  ← added  │
  └──────────────────────────────────────────┘
```

## Key terms

| Term | Plain meaning | Analogy |
|---|---|---|
| **DynamoDB** | AWS's fast, flexible NoSQL database | A huge, well-labelled filing cabinet |
| **Table** | A collection of items in DynamoDB | One drawer of the cabinet |
| **Item** | One record in a table | One folder in the drawer |
| **Attribute** | One field on an item (`name`, `email`) | A line on the folder's form |
| **Partition key** | The attribute DynamoDB uses to organize and spread data across servers | The label on each folder's tab |
| **Schemaless** | Items don't need a fixed set of attributes defined in advance | Every folder can hold different forms |
| **NoSQL** | A database that doesn't use fixed tables of rows and columns | Folders instead of one giant spreadsheet |
| **DynamoDB JSON** | JSON that also labels each value's data type | A form that says "this box is a number" |
| **Lambda function** | Code you run in AWS Lambda without managing servers | A worker who shows up only when called |
| **Runtime** | The environment the code runs in (Node.js here) | The kitchen a recipe is cooked in |
| **Execution role** | The IAM role that sets what the function is allowed to do | The worker's access badge |
| **AWS SDK** | Pre-written code libraries for using AWS services | A toolbox instead of building every tool yourself |
| **Test event** | A fake JSON event you send to the function to see how it reacts | A fire drill |
| **Least privilege** | Give only the permissions a job needs | A key to one room, not the master key |
| **DynamoDB Stream** | A feed of recent changes to a table (last 24 hours) | A live news feed for the table |

## DynamoDB and the partition key

- **DynamoDB** is fast and flexible, so it's great for apps that need quick access to large amounts of data, such as games.
- The **partition key** is the heart of how DynamoDB organizes data. It's a label for grouping items, and under the hood DynamoDB uses it to **spread data across servers** for quick access and efficient queries.
- In this table the partition key is **`userId`**, so each user is found by their ID.

> **Note:** the lesson says every item must have a **unique partition key**. That's true here, because the table has a partition key only. If a table also has a **sort key**, items can share a partition key as long as the partition key and sort key *together* are unique.

## Adding an item: JSON view and schemaless data

- The console's **JSON view** lets you write the item as code instead of filling in a form. That's much faster when an item has many attributes.
- Turn **off DynamoDB JSON** to write plain JSON. DynamoDB JSON also labels each value's type:

| Plain JSON | DynamoDB JSON |
|---|---|
| `{ "userId": "1", "name": "Test User" }` | `{ "userId": { "S": "1" }, "name": { "S": "Test User" } }` |

(`S` = string, `N` = number.)

- The item had `userId` 1, `name` Test User and `email` test@example.com. You **didn't have to create** `name` or `email` beforehand.
- That's because DynamoDB is **schemaless**: you can add attributes as you go, and each item can have a different set. This flexibility is a key benefit of **NoSQL** databases.

> **Note:** the lesson says DynamoDB "stores your data in JSON". More precisely, JSON is how the console and SDK *show and send* the data. DynamoDB's internal storage format isn't JSON.

## Creating the Lambda function

- **Lambda** runs your code without you managing servers, only when it's needed (no paying for idle time), and scales automatically.
- **Ways to create a function:** from scratch (writing the code yourself, ✅ this project), from a **blueprint**, **cloned** from an existing function, **imported from a ZIP**, or **deployed from a repository**.
- **Runtime:** **Node.js**, which runs JavaScript on the server side so it can act as a web app's backend.

## Execution roles

- An **execution role** is an IAM role for the function. It defines what the function can do, such as accessing DynamoDB.
- **By default**, AWS creates a role that can only **write logs to CloudWatch**. It can't read DynamoDB, which is why the first test failed.
- **Why have one?** Security. The function shouldn't have unlimited access to your account. It gets only the permissions its job needs, which limits the damage if anything goes wrong.
- The function's **Permissions tab** shows its execution role and that role's permissions. This is where you control what the function can access.

## The code and the AWS SDK

The function code does two things:

| Part | What it does |
|---|---|
| **First half** | Uses the **AWS SDK for JavaScript** to take a `userId` and get the matching item from the `UserData` table |
| **Second half** (from `try {`) | Handles errors from the database call and returns a tailored error message saying what went wrong |

- The **AWS SDK** is a library of pre-written code for using AWS services, such as creating an S3 bucket or launching an EC2 instance, without writing it all yourself.
- There's a different SDK for each language. This code uses the **JavaScript** one to talk to DynamoDB.
- **Without the SDK**, you'd have to write the raw, signed requests to AWS yourself, which is much more complex and error-prone.

## Testing and the AccessDeniedException

- The **Test tab** is where you run the function with **fake events** to see how it reacts.
- This test asked the function to find the item with **`userId` 1**. The event is written in **JSON** so Lambda can easily read it.

**What happened:**

| Run | Console said | The response | Why |
|---|---|---|---|
| 1st | ✅ "Succeeded" | ❌ `AccessDeniedException` | The execution role had no permission to read DynamoDB, so DynamoDB blocked the function |
| 2nd (after the fix) | ✅ "Succeeded" | ✅ The user item | The role now had **AmazonDynamoDBReadOnlyAccess** |

**Why "success" when access was denied?** "Succeeded" only means the **code ran without crashing**. The code's `try`/`catch` caught the DynamoDB error and returned it as a normal response. It doesn't mean the function did what you wanted, so always **read the response**, not just the status.

## Choosing a permission policy

The four DynamoDB policies to choose from:

| Policy | What it allows | Right for this function? |
|---|---|---|
| **AmazonDynamoDBReadOnlyAccess** | Reading and querying data | ✅ **Yes**: the function only reads |
| **AmazonDynamoDBFullAccess** | Everything: creating, deleting and managing tables, plus reading and writing data | ❌ Far more than needed |
| **AWSLambdaDynamoDBExecutionRole** | Reading a **DynamoDB Stream**, a feed of the table's changes in the last 24 hours | ❌ For Streams, not reading items |
| **AWSLambdaInvocation-DynamoDB** | Triggering Lambda functions from events in a DynamoDB Stream (e.g. notifying a user when they add a product to their cart) | ❌ For Streams, not reading items |

**Why not FullAccess?** It breaks **least privilege**. If someone got unauthorized access to the function, they could edit it to change or delete important data, which could take the service down or cause data loss.

> **Tip:** ReadOnlyAccess is still broad, because it can read **every** table in the account. The tightest option is a custom inline policy that allows only `dynamodb:GetItem` on the `UserData` table's ARN. That's what the "Enhancing Security" extension covers.

## Use cases

Lambda + DynamoDB make a serverless backend for web apps, fetching data when a user logs in or uses the app:

| App | Lambda fetches |
|---|---|
| 🛒 E-commerce | Product details, search results, order history |
| 📱 Social media | User profiles and their linked videos or images |
| 📰 News sites and blogs | Articles that match a user's query |

## Build order and cleanup

1. Create a DynamoDB table **`UserData`** with partition key **`userId`**.
2. **Create item** → switch to **JSON view**, turn **off** DynamoDB JSON, paste the item (`userId` 1, `Test User`, `test@example.com`).
3. Create a Lambda function **from scratch**, runtime **Node.js**. It gets a default execution role.
4. Paste the code and **Deploy**.
5. **Test tab:** create a test event `{ "userId": "1" }` and run it. You get "Succeeded" but an **AccessDeniedException** ⚠️
6. **Configuration → Permissions:** open the execution role in IAM → **Add permissions → Attach policies** → **AmazonDynamoDBReadOnlyAccess**.
7. Run the test again. The user item comes back ✅

**Cleanup:** delete the DynamoDB table (keep **Delete all CloudWatch alarms** ✅ ticked and leave **Create an on-demand backup** ❌ unticked, since backups cost extra) → delete the Lambda function → delete its execution role in IAM → delete the CloudWatch log group `/aws/lambda/<function name>`.

> **Check first:** the three-tier series continues. If the next project connects these pieces together, keep them until it's done.

---

## Self-quiz

<details><summary>1. What is DynamoDB good for?</summary>

It's a fast, flexible NoSQL database, great for apps that need quick access to large volumes of data, such as games.
</details>

<details><summary>2. What does the partition key do?</summary>

It's the label DynamoDB uses to organize items and spread them across servers, for quick access and efficient queries. Here it's `userId`.
</details>

<details><summary>3. Must every item have a unique partition key?</summary>

In a table with only a partition key (like this one), yes. If the table also has a sort key, items can share a partition key as long as the partition key and sort key together are unique.
</details>

<details><summary>4. Why use the JSON view when adding an item?</summary>

You can write the item as code instead of filling in a form field by field, which saves time when there are many attributes.
</details>

<details><summary>5. What's the difference between plain JSON and DynamoDB JSON?</summary>

DynamoDB JSON also labels each value's data type, e.g. `{ "S": "Test User" }` for a string. Plain JSON just has the values.
</details>

<details><summary>6. What does "schemaless" mean?</summary>

You don't have to define attributes in advance. You can add them as you go, and each item can have a different set. That's why `name` and `email` didn't need setting up first.
</details>

<details><summary>7. Name three ways to create a Lambda function besides writing it from scratch.</summary>

Any three of: from a blueprint, cloned from an existing function, imported from a ZIP file, or deployed from a repository.
</details>

<details><summary>8. What is an execution role, and what can the default one do?</summary>

The IAM role that defines what a Lambda function is allowed to do. The default one can only write logs to CloudWatch.
</details>

<details><summary>9. Why does Lambda use an execution role at all?</summary>

Security. The function gets only the permissions its job needs instead of unlimited access to the account.
</details>

<details><summary>10. What is the AWS SDK, and why use it?</summary>

A library of pre-written code for using AWS services. It saves you writing the raw requests yourself, which would be more complex and error-prone.
</details>

<details><summary>11. What do the two halves of the function code do?</summary>

The first half uses the SDK to get the item for a `userId` from `UserData`. The second half (`try {` onwards) handles errors and returns a tailored message.
</details>

<details><summary>12. What is the Test tab for?</summary>

Running the function with fake events to see how it reacts, before connecting it to anything real.
</details>

<details><summary>13. Why did the first test give an AccessDeniedException?</summary>

The execution role had no permission to read DynamoDB, so DynamoDB blocked the function from reading the table.
</details>

<details><summary>14. Why did the console say "Succeeded" when access was denied?</summary>

"Succeeded" only means the code ran without crashing. The error was caught and returned as a normal response, so you have to read the response to see whether it worked.
</details>

<details><summary>15. Where do you see and change a function's permissions?</summary>

The function's **Permissions** tab (under Configuration), which links to its execution role in IAM.
</details>

<details><summary>16. Which policy did you attach, and why that one?</summary>

**AmazonDynamoDBReadOnlyAccess**, because the function only needs to read data. That follows least privilege.
</details>

<details><summary>17. What's the risk of attaching AmazonDynamoDBFullAccess instead?</summary>

It allows creating, deleting and changing tables and data. If someone got unauthorized access to the function, they could edit it to change or delete important data.
</details>

<details><summary>18. What do AWSLambdaDynamoDBExecutionRole and AWSLambdaInvocation-DynamoDB do?</summary>

Both are for **DynamoDB Streams**. The first lets a function read the stream of table changes, and the second triggers functions from stream events. Neither is for reading items.
</details>

<details><summary>19. How could you make the permissions even tighter than ReadOnlyAccess?</summary>

Use a custom inline policy that allows only `dynamodb:GetItem` on the `UserData` table's ARN, instead of read access to every table.
</details>

<details><summary>20. When deleting the table, which two options do you see, and what do you pick?</summary>

**Delete all CloudWatch alarms**: keep it ticked, to avoid charges. **Create an on-demand backup**: leave it unticked, because backups cost extra.
</details>
