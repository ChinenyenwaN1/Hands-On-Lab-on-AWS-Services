# DynamoDB Lambda Triggers

# Overview

We will be setting up a DynamoDB table full of cats that are waiting to be adopted. When the `adopted` field is updated to `true`, a Lambda will be triggered that will send an email to the adoption center telling them that “[cat abc] has been adopted!”

# Instructions

## Stage 1 - Creating the DynamoDB table

Head to the DynamoDB dashboard: [https://ap-southeast-2.console.aws.amazon.com/dynamodbv2/home?#tables](https://ap-southeast-2.console.aws.amazon.com/dynamodbv2/home?#tables)

Click on Create table

For **Table Name**, we will use `cat-adoption`

The **Partition Key** will be `catID` (string)

Leave the **Sort Key** blank

Change the **Table Settings** as to “Customize settings”
![image](https://user-images.githubusercontent.com/116161693/218500561-734bfcc2-51fb-48c2-aed2-02a653f46109.png)

Leave the **Table Class** as “DynamoDB Standard”

Set the ********************Read/write capacity settings******************** to “On-demand”

![image](https://user-images.githubusercontent.com/116161693/218500632-85b7cd93-7921-4080-95e1-2a2600df3979.png)


Under ************************************Secondary Indexes w************************************e’ll create a **Global Index**, with the **Partition Key** as `adopted` (string), and no **Sort Key** (leave it blank). The **Index Name** will be “adopted-index”, and the **Attribute Projections** will be “All”

![image](https://user-images.githubusercontent.com/116161693/218500713-f51d3e5c-7fc7-4fab-8e45-4a186c8b2f06.png)

Leave **Encryption at rest** as “Owned by Amazon DynamoDB”

Click Create table

![image](https://user-images.githubusercontent.com/116161693/218500748-70adabf7-8692-49ad-a880-4de77d5f6b33.png)

## Stage 2 - Populating the table

Now you get to add as many cats as you like to the `cat-adoption` table. In the real world this would be populated by an external service, or a frontend website the cat adoption agency uses, but in this case we will enter the data manually.

Click on your newly created `cat-adoption` table, click Actions → Create item

![image](https://user-images.githubusercontent.com/116161693/218500871-f59c43b6-c207-4dba-9df6-c67fe46e5e2c.png)

Set the `catID` **Value** to “cat0001”

Click Add new attribute → String, set the **Attribute name** to “catName”, set the **Value** to your chosen cats name

Click Add new attribute → String, set the **Attribute name** to “adopted”, set the **Value** to “false”

Click Create item

![image](https://user-images.githubusercontent.com/116161693/218500949-f9a4c9ce-735a-4e01-bceb-bd8ab701cd9b.png)

Repeat this process as many times as you like, don’t forget to update the `catID` each time with an incrementing number.

## Stage 2a - Querying our data

Head to Explore table items

To show why we created a global secondary index on the table, click on **Scan or query items**, select **********Query**********, change the ************Index************ to “adopted-index” and search for “true” in the partition key. 

Searching for “true” will return no objects (assuming you set all of your cat records to adopted: false):
![image](https://user-images.githubusercontent.com/116161693/218501130-427b6077-0cd6-40b5-b2b3-9363e92a5bd7.png)

Searching for “false” will return all cats that haven’t been adopted:

![image](https://user-images.githubusercontent.com/116161693/218501183-f9d4cf64-9f4e-46ad-842f-49d64bc9f9c2.png)

**Querying** uses indexes, which is far more efficient and cost effective, without having this secondary index we would need to **Scan** the table to look for cats which are/aren’t adopted, which means DynamoDB iterates over every record and checks that key. With tables that have thousands or millions of records, this can be very expensive (and slow).

## Stage 3 - Setting up SNS

Head to the SNS console: [https://ap-southeast-2.console.aws.amazon.com/sns/v3/home?region=ap-southeast-2#/topics](https://ap-southeast-2.console.aws.amazon.com/sns/v3/home?region=ap-southeast-2#/topics)

Click on Create topic

Set the ********Type******** to “Standard”

Set the ********Name******** to be “Adoption-Alerts”

![image](https://user-images.githubusercontent.com/116161693/218501313-3ae10a58-a12f-4981-ade6-1f8fa5097a18.png)

Under **************************Access policy**************************, leave the ************Method************ as “Basic”

![image](https://user-images.githubusercontent.com/116161693/218501452-641fe8fb-9cad-4ec2-8c38-c60a0d6dd3b4.png)

Change **Define who can publish messages to the topic** to “Only the specified AWS accounts” and enter your account ID (found in the top right of the screen)

Change **Define who can subscribe to this topic** to “Only the specified AWS accounts” and enter your account ID again

***********In the real world, this should be locked down further to only the resources you want publishing to the topic, but in this temporary example set up, locking down to just the account is fine and safe enough***********

Leave all other options as default

Click on Create topic

On the next page, click on Create subscription

![image](https://user-images.githubusercontent.com/116161693/218501555-06ce169a-e6cf-4a4c-88d9-860dbef8c053.png)

Change the ****************Protocol**************** to “Email”

In the ****************Endpoint**************** field, enter your personal email

Click Create subscription

You will receive a confirmation email shortly after, with a link you need to click on. This tells SNS that you’re happy to receive emails from the topic, and prevents spam from being sent via SNS.

******************Side note: While writing this, my confirmation went to Spam in Gmail, so don’t forget to check there.******************

Your subscription should now be in the Confirmed state:

![image](https://user-images.githubusercontent.com/116161693/218501711-5e669195-9442-472b-aebc-4f61868d78da.png)

## Stage 4 - Create the Lambda

Head to the Lambda console: [https://ap-southeast-2.console.aws.amazon.com/lambda/home?region=ap-southeast-2#/functions](https://ap-southeast-2.console.aws.amazon.com/lambda/home?region=ap-southeast-2#/functions)

Click Create function

Leave **Author from scratch** selected

Set the **************************Function name************************** to `cat-adoption-function`

Set the **************Runtime************** to “Python 3.9”

Leave the ************************Architecture************************ as “x86_64”

Open the ****************************************************Change default execution role**************************************************** and set the ****************************Execution role**************************** to “Create a new role from AWS policy templates”

Set the ******************Role name****************** to `cat-adoption-function-role`

Add the `Amazon SNS publish policy` template to the ********************************Policy templates********************************

Click Create function

![image](https://user-images.githubusercontent.com/116161693/218501803-8126636a-4950-421b-be4c-93edbb3b07b3.png)

In the **Code** tab, enter the following code:

```python
import boto3

def lambda_handler(event, context):
    sns = boto3.client('sns')
    accountid = context.invoked_function_arn.split(":")[4]
    region = context.invoked_function_arn.split(":")[3]
    for record in event['Records']:
        message = record['dynamodb']['NewImage']
        catName = message['catName']['S']

        response = sns.publish(
            TopicArn=f'arn:aws:sns:{region}:{accountid}:Adoption-Alerts',
            Message=f"{catName} has been adopted!",
            Subject='Cat adopted!',
        )
```

This code basically iterates through all the records that DynamoDB passes to the Lambda function, and then publishes a message to SNS.

Don’t forget to click Deploy to save the function.

![image](https://user-images.githubusercontent.com/116161693/218501866-580c757e-d918-408f-87e7-ae17c893b59e.png)

## Stage 4a - Add DynamoDB permissions to the Lambda role

Head to the Lambda console: [https://ap-southeast-2.console.aws.amazon.com/lambda/home?region=ap-southeast-2#/functions](https://ap-southeast-2.console.aws.amazon.com/lambda/home?region=ap-southeast-2#/functions)

Click on your newly created function, then the **Configuration** tab, then **Permissions.**

Click on the role 

![image](https://user-images.githubusercontent.com/116161693/218502385-9998d174-b6c9-4280-8ad8-a4c6c1bf37bc.png)

Click on Add Permissions and then Attach Policies

Search for “AmazonDynamoDBFullAccess” and select it

Click Attach Policies

Your role policies should look something like this (the blurred policy ID will be different for you, so I’ve blurred it out to make it simpler):

![image](https://user-images.githubusercontent.com/116161693/218501951-b19ea0de-07b6-44ae-ace6-26077b9948bb.png)

This is required so the Lambda function can read from the DynamoDB Stream. In the real world this should be locked down a lot further, but in this case, we’re okay with the Lambda having full DynamoDB permissions to ***all*** tables.

## Stage 5 - Enabling the DynamoDB Stream

Head back to the DynamoDB console: [https://ap-southeast-2.console.aws.amazon.com/dynamodbv2/home?region=ap-southeast-2#table?initialTagKey=&name=cat-adoption&tab=streams](https://ap-southeast-2.console.aws.amazon.com/dynamodbv2/home?region=ap-southeast-2#table?initialTagKey=&name=cat-adoption&tab=streams)

Click on the ************************************Export and streams************************************ tab, then under **DynamoDB stream details** click Enable

On the next page, under **View type**, select ****************“New image”. We only care about the new data, we don’t need the previous record data from before it was changed.

![image](https://user-images.githubusercontent.com/116161693/218503145-c548aad1-a3d8-4695-9349-6bd391b354eb.png)

Click Enable stream

On the next page, under **DynamoDB stream details,** click Create trigger

On the next page, under **Lambda function** select your newly created “cat-adoption-function”, leave the other options as is.

Click Create trigger

![image](https://user-images.githubusercontent.com/116161693/218502979-ba5d163c-1c75-4886-b316-a0875ac5fece.png)

## Stage 6 - Testing it out

Head back to the DynamoDB console: [https://ap-southeast-2.console.aws.amazon.com/dynamodbv2/home?region=ap-southeast-2](https://ap-southeast-2.console.aws.amazon.com/dynamodbv2/home?region=ap-southeast-2#table?initialTagKey=&name=cat-adoption&tab=streams)

Click on Explore table items

You should see the cat items you created earlier, if not, click on Scan → Run

![image](https://user-images.githubusercontent.com/116161693/218503317-12fcdf79-cd00-4e72-9151-e5bce85891d3.png)

Change one of the “adopted” fields to “true”

After a few seconds you should receive an email telling you that cat has been adopted!

