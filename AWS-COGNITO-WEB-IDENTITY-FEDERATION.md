# Web Identity Federation
We will be implementing a simple serverless application which uses Web Identity Federation.
The application runs using the following technologies

*	S3 for front-end application hosting
*	Google API Project as an ID Provider
*	Cognito and IAM Roles to swap Google Token for AWS credentials

The application runs from a browser, gets the user to login using a Google ID and then loads all images from a private S3 bucket into a browser using presignedURLs.

This project consists of the below stages :-
*	Provision the environment and review tasks <= THIS STAGE
*	Create Google API Project & Client ID
*	Create Cognito Identity Pool
*	Update App Bucket & Test Application

![image](https://user-images.githubusercontent.com/116161693/210277689-2efeb3b1-77ca-4d3f-84e7-7b1eba30af8e.png)

**Login to an AWS Account**

Login to an AWS account using a user with admin privileges and ensure your region is set to us-east-1 N. Virginia
Click HERE to auto configure the infrastructure the app requires Check the The following resource(s) require capabilities: [AWS::IAM::ManagedPolicy, AWS::IAM::Role] box
Click Create Stack
Wait for the STACK to move into the CREATE_COMPLETE state before continuing.

**Verify S3 bucket**

Open the S3 console https://s3.console.aws.amazon.com/s3/home?region=us-east-1
Open the bucket starting webidf-appbucket
It should have objects within it, including index.html and scripts.js

![image](https://user-images.githubusercontent.com/116161693/210277720-0187444b-6109-413b-9a9d-664c74ca3941.png)

**Click the Permissions Tab**

Verify Block all public access is set to Off
Click Bucket Policy
Verify there is a bucket policy

Verify privatebucket
Open the bucket starting webidf-patchesprivatebucket-
Load the objects in the bucket so you are aware of the contents
Verify there is no bucket policy and the bucket is entirely private

![image](https://user-images.githubusercontent.com/116161693/210277759-e22c95e9-0fb6-47e5-8199-ef7760de8a6b.png)

Verify CloudFormation Distribution
Move to the CloudFront consle https://us-east-1.console.aws.amazon.com/cloudfront/v3/home?region=us-east-1#/distributions
Locate the distribution pointing at origin webidf-appbucket-.... and click
Locate the distribution domain name
Note down as the WebApp URL this name prefixed with https i.e if yours is d1o4f0w1ew0exo.cloudfront.net then your WebApp URL is https://https://d1o4f0w1ew0exo.cloudfront.net (note, the copy icon may copy the https:// for you)	

![image](https://user-images.githubusercontent.com/116161693/210277805-ccd664c5-3b2b-41c5-8173-b25808e2fc4b.png)

Any application that uses OAuth 2.0 to access Google APIs must have authorization credentials that identify the application to Google's OAuth 2.0 server
In this stage we need to create those authorization credentials.
You will need a valid google login, GMAIL will do.
If you don't have one, you will need to create one as part of this process.
Move to the Google Credentials page https://console.developers.google.com/apis/credentials
Either sign in, or create a google account

![image](https://user-images.githubusercontent.com/116161693/210277816-482d0180-52af-4e77-8cee-1e39f90bd642.png)

You will be moved to the Google API Console

![image](https://user-images.githubusercontent.com/116161693/210277850-4157ff71-fa10-4926-8dba-55236cf04b67.png)

You may have to set your country and agree to some terms and conditions, thats fine go ahead and do that.
Click the Select a project dropdown, and then click NEW PROJECT
For project name enter PetIDF
Click Create

![image](https://user-images.githubusercontent.com/116161693/210277855-f6807eb6-677a-4193-9c97-f30ce7ebf86a.png) 

**Click on Configure Consent screen**

Click Credentials
Click CONFIGURE CONSENT SCREEN

![image](https://user-images.githubusercontent.com/116161693/210277864-b8d0d324-afb2-4120-8da9-c28d6ba02503.png)

because our application will be usable by any google user, we have to select external users
Check the box next to External and click CREATE

![image](https://user-images.githubusercontent.com/116161693/210277874-3fcab783-1c9d-4b6f-81f2-9a69fc2e5705.png)

Next you need to give the application a name ... enter PetIDF in the App Name box.
enter your own email in user support email

![image](https://user-images.githubusercontent.com/116161693/210277894-1241fd92-ce96-441c-9c89-363541cdd5d4.png)

enter your own email in Developer contact information
Click SAVE AND CONTINUE
Click SAVE AND CONTINUE
Click SAVE AND CONTINUE
Click BACK TO DASHBOARD

![image](https://user-images.githubusercontent.com/116161693/210277908-87a59b98-f925-4ed9-aab3-d58c0e8eb5d9.png)
 
Click Credentials on the menu on the left
Click CREATE CREDENTIALS and then OAuth client ID
In the Application type download select Web Application
Under Name enter PetIDFServerlessApp
We need to add the WebApp URL, this is the distribution domain name of the cloudfront distribution (making sure it has https:// before it)

 ![image](https://user-images.githubusercontent.com/116161693/210277919-a4d0671d-d7f5-4058-af79-a726e40fbe6d.png)

Click ADD URI under Authorized JavaScript origins
Enter the endpoint URL, you need to enter the Distribution DNS Name of your CloudFront distribution (created by the 1-click deployment), you should add https:// at the start, it should look something like this https://d292zn5b0cjcz8.cloudfront.net but you NEED to use your own distributions DNS name DONT USE THIS ONE
Click CREATE
You will be presented with two pieces of information
•	Client ID
•	Client Secret
Note down the Client ID you will need it later.
You wont need the Client Secret again.
Once noted down safely, click OK

![image](https://user-images.githubusercontent.com/116161693/210277936-e6aea3a8-7071-4111-86d5-3eedaa07df4d.png)

Move to the Cognito Console https://console.aws.amazon.com/cognito/home?region=us-east-1# On the menu on the left, select Federated Identities
We're going to be creating a new identity pool
If this is your first, the creation process will begin immediatly, if you already have any identity pools you'll have to click federated identities then click on Create new identity pool
In Identity pool name enter PetIDFIDPool

 
![image](https://user-images.githubusercontent.com/116161693/210277942-0ba309c7-a72f-48ff-9211-8b0891f6f656.png)

Expand Authentication Providers and click on Google+
In the Google Client ID box, enter the Google Client ID you noted down in the previous step.
Click Create Pool

 ![image](https://user-images.githubusercontent.com/116161693/210277953-9c284887-3e31-469b-8342-8add95654a20.png)

Expand View Details
This is going to create two IAM roles
One for Your authenticated identities and another for your Your unauthenticated identities
For now, we're just going to click on Allow we can review the roles later.

![image](https://user-images.githubusercontent.com/116161693/210277972-bb4cd283-2a86-4849-8af9-f73a4487bdc5.png)


You will be presented with your Identity Pool ID, note this down, you will need it later. Click to move back to the dashboard.

![image](https://user-images.githubusercontent.com/116161693/210277980-35381d49-f2bb-4f38-afbc-d2eb313215f8.png)

Click on go to dashboard

![image](https://user-images.githubusercontent.com/116161693/210277989-57b720e1-9455-48bc-a2d9-15367648346b.png)


The serverless application is going to read images out of a private bucket created by the initial cloudformation template.
The bucket is called patchesprivatebucket
Move to the IAM Console https://console.aws.amazon.com/iam/home?region=us-east-1#/home
Click Roles

![image](https://user-images.githubusercontent.com/116161693/210278008-41ff598f-b6fc-42cf-8568-28822532c1cb.png)

Locate and click on Cognito_PetIDFIDPoolAuth_Role
Click on Trust Relationships
See how this is assumable by cognito-identity.amazonaws.com
With two conditions
*	StringEquals cognito-identity.amazonaws.com:aud your congnito ID pool
*	ForAnyValue:StringLike cognito-identity.amazonaws.com:amr authenticated
This means to assume this role - you have to be authenticated by one of the ID providers defined in the cognito ID pool.

![image](https://user-images.githubusercontent.com/116161693/210278034-930bcd28-bee4-4896-8656-1846ef616224.png)

When you use PETIDF with cognito, this role is assumed on your behalf by cognito, and its what generates temporary AWS credentials which are used to access AWS resources.
Click permissions .. this defines what these credentials can do.
The cloudformation template created a managed policy which can access the privatepatches bucket Click Add permissions and then Attach policies

![image](https://user-images.githubusercontent.com/116161693/210278044-8383c6bb-8382-4ef3-9bc9-c8514c551afb.png)

Type PrivatePatches in the search box and press enter
Check the box next to PrivatePatchesPermissions and click Attach Policies

![image](https://user-images.githubusercontent.com/116161693/210278057-67369995-18bc-41c9-8d6e-f791fef79cb0.png)
 
Download index.html and scripts.js from the S3
Move to the S3 Console https://s3.console.aws.amazon.com/s3/home?region=us-east-1
Open the webidf-appbucket- bucket
select index.html and click Download & save the file locally
select scripts.js and click Download & save the file locally

![image](https://user-images.githubusercontent.com/116161693/210278070-dfb1b186-2a68-462d-956f-b6f168e98e35.png)

Open the index.html in a code editor.
Locate the REPLACE_ME_GOOGLE_APP_CLIENT_ID placeholder
Replace this with YOUR CLIENT ID
Save index.html
Open the scripts.js in a code editor.
Locate the IdentityPoolId: REPLACE_ME_COGNITO_IDENTITY_POOL_ID placeholder
Replace the REPLACE_ME_COGNITO_IDENTITY_POOL_ID part with your IDENTITY POOL ID you noted down in the previous step

![image](https://user-images.githubusercontent.com/116161693/210278082-9a5645bd-80e8-4f6c-81ba-c56fb5c05219.png)

Locate the Bucket: "REPLACE_ME_NAME_OF_PATCHES_PRIVATE_BUCKET" placeholder.
Replace REPLACE_ME_NAME_OF_PATCHES_PRIVATE_BUCKET with with bucket name of the webidf-patchesprivatebucket- bucket Save scripts.js

![image](https://user-images.githubusercontent.com/116161693/210278099-6d87ca6d-9366-4981-8b71-1dbf068072a7.png)

![image](https://user-images.githubusercontent.com/116161693/210278130-6a465d6b-6b36-4699-85a3-917e5d52d79e.png)


Back on the S3 console, inside the webidf-appbucket- bucket.
Click delete the previous files and Upload the updated index.html and scripts.js files and click Upload

![image](https://user-images.githubusercontent.com/116161693/210278140-468d4cd9-167c-4d4f-a3e9-fe3fba2c2cd3.png)

Open the WebApp URL you noted down earlier, the distribution domain name of the cloudfront distribution
With the browser console open, Click Sign In
Sign in with your google account

![image](https://user-images.githubusercontent.com/116161693/210278183-20b0213f-d6c4-4702-b6d8-973e70d2f879.png)

Once signed in you should see 3 cat pictures loaded from a private S3 bucket
Click on each of them, notice the URL which is used? it's a presignedURL generated by the JS running in browser, using the API's which you can access using the cognito credentials.

![image](https://user-images.githubusercontent.com/116161693/210278165-f4c5f91d-2d7b-4826-8790-8ac8e179346b.png)


When you click the Sign In button a few things happen:- (watch the console)
*	You authenticate with the Google IDP
*	a Google Access token is returned
*	This token is provided as part of the API Call to Cognito
*	If successful this exchanges this for Temporary AWS credentials
*	These are used to list objects in the private bucket
*	for all objects, presignedURLs are generated and used to load the images in the browser.
Once signed in you should see 3 cat pictures loaded from a private S3 bucket
Click on each of them, notice the URL which is used? it's a presignedURL generated by the JS running in browser, using the API's which you can access using the cognito credentials.
All of this is done with no self-managed compute.

 
