# CodePipeline

I will be implementing a full code pipeline incorportating commit, build and deploy steps.

![image](https://user-images.githubusercontent.com/116161693/212893236-77d3cce0-4e26-45ce-8342-8ebb75446108.png)

This project consists of 5 stages :-
- STAGE 1 : Configure Security & Create a CodeCommit Repo
- STAGE 2 : Configure CodeBuild to clone the repo, create a container image and store on ECR
- STAGE 3 : Configure a CodePipeline with commit and build steps to automate build on commit.
- STAGE 4 : Create an ECS Cluster, TG's , ALB and configure the code pipeline for deployment to ECS Fargate

# STAGE 2 - CODE BUILD

we will configure the Elastic Container Registry and use the codebuild service to build a docker container and push it into the registry.

## CREATE A PRIVATE REPOSITORY

Move to the Container Services console, the repositories (https://us-east-1.console.aws.amazon.com/ecr/repositories?region=us-east-1)  
Create a Repository.  
It should be a private repository.  
..and for the alias/name pick 'catpipeline'.  
Note down the URL and name (it should match the above name).

![image](https://user-images.githubusercontent.com/116161693/212898343-0b863f3a-e7ba-4414-b035-e18a2a08afab.png)

This is the repository that codebuild will store the docker image in, created from the codecommit repo.   


## SETUP A CODEBUILD PROJECT

Next, we will configure a codebuild project to take what's in the codecommit repo, build a docker image & store it within ECR in the above repository.

Move to the codebuild console (https://us-east-1.console.aws.amazon.com/codesuite/codebuild/projects?region=us-east-1)  

Create code build project
  
### PROJECT CONFIGURATION
For `Project name` put `catpipeline-build`.  
Leave all other options in this section as default.  

### SOURCE
For `Source Provider` choose `AWS CodeCommit`  
For `Repository` choose the repo you created in stage 1  
Check the `Branch` checkbox and pick the branch from the `Branch` dropdown (there should only be one).  

### ENVIRONMENT
for `Environment image` pick `Managed Image`  
Under `Operating system` pick `Amazon Linux 2`  
under `Runtime(s)` pick `Standard`
under `Image` pick `aws/codebuild/amazonlinux2-x86_64-standard:X.0` where X is the highest number.  
Under `Image version` `Always use the latest image for this runtime version`  
Under `Envrironment Type` pick `Linux`  
Check the `Privileged` box (Because we're creating a docker image)  
For `Service role` pick `New Service Role` and leave the default suggested name which should be something like `codebuild-catpipeline-service-role`  
Expand `Additional Configuration`  
We're going to be adding some environment variables

Add the following:-

```
AWS_DEFAULT_REGION with a value of us-east-1
AWS_ACCOUNT_ID with a value of your AWS_ACCOUNT_ID_REPLACEME
IMAGE_TAG with a value of latest
IMAGE_REPO_NAME with a value of your ECR_REPO_NAME_REPLACEME
```

### BUILDSPEC
The buildspec.yml file is what tells codebuild how to build your code.. the steps involved, what things the build needs, any testing and what to do with the output (artifacts).

A build project can have build commands included... or, you can point it at a buildspec.yml file, i.e one which is hosted on the same repository as the code.. and that's what you're going to do.  

Check `Use a buildspec file`  
you don't need to enter a name as it will use by default buildspec.yml in the root of the repo. If you want to use a different name, or have the file located elsewhere (i.e in a folder) you need to specify this here.  

### ARTIFACTS
No changes to this section, as we're building a docker image and have no testing yet, this part isn't needed.

### LOGS

This is where the logging is configured, to Cloudwatch logs or S3 (optional).  

For `Groupname` enter `a4l-codebuild`  
and for `Stream Name` enter `catpipeline`  

Create the build Project

![image](https://user-images.githubusercontent.com/116161693/212898700-991e99c6-49d0-4bb0-88ae-29694b7bf0e5.png)

## BUILD SECURITY AND PERMISSIONS

Our build project will be accessing ECR to store the resultant docker image, and we need to ensure it has the permissons to do that. The build process will use an IAM role created by codebuild, so we need to update that roles permissions with ALLOWS for ECR.  

Go to the IAM Console (https://us-east-1.console.aws.amazon.com/iamv2/home#/home)  
Then Roles  
Locate and click the codebuild cat pipeline role i.e. `codebuild-catpipeline-build-service-role`  
Click the `Permissions` tab and we need to Add a permission and it will be an `inline policy`

![image](https://user-images.githubusercontent.com/116161693/212899120-117d90ed-a501-47f0-9fc4-f8547a864ce0.png)

Select to edit the raw `JSON` and delete the skeleton JSON, replacing it with

![image](https://user-images.githubusercontent.com/116161693/212899209-feed44a6-bbc1-447a-a58b-138533315c79.png)


```
{
  "Statement": [
	{
	  "Action": [
		"ecr:BatchCheckLayerAvailability",
		"ecr:CompleteLayerUpload",
		"ecr:GetAuthorizationToken",
		"ecr:InitiateLayerUpload",
		"ecr:PutImage",
		"ecr:UploadLayerPart"
	  ],
	  "Resource": "*",
	  "Effect": "Allow"
	}
  ],
  "Version": "2012-10-17"
}
```

Move on and review the policy.  
Name it `Codebuild-ECR` and create the policy
This means codebuild can now access ECR.  

![image](https://user-images.githubusercontent.com/116161693/212899278-2b28c5cf-2a11-48db-9d2d-94049eafd2f8.png)


## BUILDSPEC.YML

Create a file in the local copy of the `catpipeline-codecommit-XXX` repo called `buildspec.yml`  
Into this file add the following contents :-  

```
version: 0.2

phases:
  pre_build:
	commands:
	  - echo Logging in to Amazon ECR...
	  - aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com
  build:
	commands:
	  - echo Build started on `date`
	  - echo Building the Docker image...          
	  - docker build -t $IMAGE_REPO_NAME:$IMAGE_TAG .
	  - docker tag $IMAGE_REPO_NAME:$IMAGE_TAG $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG      
  post_build:
	commands:
	  - echo Build completed on `date`
	  - echo Pushing the Docker image...
	  - docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG
```

Then add this locally, commit and stage

``` 
git add -A .
git commit -m “add buildspec.yml”
git push 
```
![image](https://user-images.githubusercontent.com/116161693/212899470-ec162c8b-ab7e-4de4-b8c9-800a8b2e62cf.png)

buildspec.yml has been added to our catpipeline-codecommit-1

![image](https://user-images.githubusercontent.com/116161693/212899569-2e9ec16f-e528-42b2-9b01-65ddf8cdc4c9.png)

## TEST THE CODEBUILD PROJECT

Open the CodeBuild console (https://us-east-1.console.aws.amazon.com/codesuite/codebuild/projects?region=us-east-1)  
Open `catpipeline-build`  
Start Build  
Check progress under phase details tab and build logs tab  

![image](https://user-images.githubusercontent.com/116161693/212899716-11cb2312-d9b7-492a-8f7b-cf87123e9cfa.png)

## TEST THE DOCKER IMAGE

Use this link to deploy an EC2 instance with docker installed (https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/quickcreate?templateURL=https://learn-cantrill-labs.s3.amazonaws.com/aws-codepipeline-catpipeline/ec2docker.yaml&stackName=DOCKER) accept all details, check the checkbox and create the stack.
Wait for this to move into the `CREATE_COMPLETE` state before continuing.  

![image](https://user-images.githubusercontent.com/116161693/212899784-d14870ab-ab1d-4495-8013-ce2e68b6007a.png)

Move to the EC2 Console ( https://us-east-1.console.aws.amazon.com/ec2/home?region=us-east-1#Home: )  
Instances  
Select `A4L-PublicEC2`, right click, connect  
Choose EC2 Instance Connect, leave everything with defaults and connect.  
![image](https://user-images.githubusercontent.com/116161693/212899840-0c7a37e0-6bb4-485b-9e2d-c225c1adfabc.png)

Docker should already be preinstalled and the EC2 instance has a role which gives ECR permissions which you will need for the next step.

test docker via  `docker ps` command
it should output an empty list  


run a `aws ecr get-login-password --region us-east-1`, this command gives us login information for ECR which can be used with the docker command. To use it use this command.

you will need to replace the placeholder with your AWS Account ID (with no dashes)

`aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ACCOUNTID_REPLACEME.dkr.ecr.us-east-1.amazonaws.com`

Go to the ECR console (https://us-east-1.console.aws.amazon.com/ecr/repositories?region=us-east-1)  
Repositories  
Click the `catpipeline` repository  
For `latest` copy the URL into your clipboard  

![image](https://user-images.githubusercontent.com/116161693/212900028-54d4a9f4-47fa-41da-a330-e5c0340211e5.png)

run the command below pasting in your clipboard after docker p
`docker pull ` but paste in your clipboard after the space, i.e 
`docker pull ACCOUNTID_REPLACEME.dkr.ecr.us-east-1.amazonaws.com/catpipeline:latest` (this is an example, you will need your image URI)  

run `docker images` and copy the image ID into your clipboard for the `catpipeline` docker image

run the following command replacing the placeholder with the image ID you copied above.  

`docker run -p 80:80 IMAGEID_REPLACEME`

![image](https://user-images.githubusercontent.com/116161693/212899982-bd3a0a65-a6b4-4b9d-a4e9-92783dc86921.png)

Move back to the EC2 console tab  
Click `Instances` and get the public IPv4 address for the A4L-PublicEC2 instance.  
open that IP in a new tab, ensuring it's http://IP not https://IP  
You should see the docker container running, with cats in containers as shown below. if so, this means your automated build process is working.  

![image](https://user-images.githubusercontent.com/116161693/212900239-22910ca4-4e92-4c80-a99f-d74203e28c6e.png)

# STAGE 3 - CODE PIPELINE

In this stage we will create a code pipeline which will utilize CODECOMMIT and CODEBUILD to create a continous build process. The aim is that every time a new commit is made to the comecommit repo, a new docker image is created and pushed to ECR. Initially you will be skipping the codedeploy step which will be added next.

## PIPELINE CREATION

Move to the codepipeline console (https://us-east-1.console.aws.amazon.com/codesuite/codepipeline/pipelines)  
Create a pipeline  
For `Pipeline name` put `catpipeline`.  
Chose to create a `new service role` and keep the default name, it should be called `AWSCodePipelineServiceRole-us-east-1-catpipeline` 
Expand `Advanced Settings` and make sure that `default location` is set for the artifact store and `default AWS managed key` is set for the `Encryption key`  
next

![image](https://user-images.githubusercontent.com/116161693/212900886-bbb7920f-c558-4db0-8f20-be5a8c5b59b6.png)

### Source Stage

Pick `AWS CodeCommit` for the `Source Provider`  
Pick `catpipeline-codecommit` for the report name
Select the branch from the `Branch name` dropdown
From `Detection options` pick `Amazon CloudWatch Events` and for `Output artifact format` pick `CodePipeline default`  
next

![image](https://user-images.githubusercontent.com/116161693/212900913-cc7f1aaf-b8e5-437e-ba99-af1928370991.png)

### Build Stage

Pick `AWS CodeBuild` for the `Build provider`  
Pick `US East (N.Virginia)` for the `Region`  
Choose the project you created earlier in the `Project name` search box  
For `Build type` choose `Single Build`
next

![image](https://user-images.githubusercontent.com/116161693/212900958-0d2edabe-6b9c-48e6-95b3-e27f4e5cd7e7.png)

### Deploy Stage

Skip deploy Stage, and confirm  
Create the pipeline

## PIPELINE TESTS
The pipeline will do an initial execution and it should complete without any issues.  
You can click details in the build stage to see the progress...or wait for it to complete  

![image](https://user-images.githubusercontent.com/116161693/212901198-0204172b-6ab5-4065-8f3f-898984d7c026.png)

Open the S3 console in a new tab (https://s3.console.aws.amazon.com/s3/) and open the `codepipeline-us-east-1-XXXX` bucket  
Click `catpipeline/`  
This is the artifacts area for this pipeline, leave this tab open you will be using in later in the demo.  

## UPDATE THE BUILD STAGE

Find your local copy of the catpipeline-codecommit repo and update the buildspec.yml file as below

```
version: 0.2

phases:
  pre_build:
	commands:
	  - echo Logging in to Amazon ECR...
	  - aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com
	  - REPOSITORY_URI=$AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME
	  - COMMIT_HASH=$(echo $CODEBUILD_RESOLVED_SOURCE_VERSION | cut -c 1-7)
	  - IMAGE_TAG=${COMMIT_HASH:=latest}
  build:
	commands:
	  - echo Build started on `date`
	  - echo Building the Docker image...          
	  - docker build -t $REPOSITORY_URI:latest .
	  - docker tag $REPOSITORY_URI:latest $REPOSITORY_URI:$IMAGE_TAG    
  post_build:
	commands:
	  - echo Build completed on `date`
	  - echo Pushing the Docker image...
	  - docker push $REPOSITORY_URI:latest
	  - docker push $REPOSITORY_URI:$IMAGE_TAG
	  - echo Writing image definitions file...
	  - printf '[{"name":"%s","imageUri":"%s"}]' "$IMAGE_REPO_NAME" "$REPOSITORY_URI:$IMAGE_TAG" > imagedefinitions.json
artifacts:
  files: imagedefinitions.json
```
![image](https://user-images.githubusercontent.com/116161693/212901373-dab47569-be61-4302-81d0-63916107bf3d.png)

## TEST A COMMIT

run

```
git add -A .
git commit -m "updated buildspec.yml"
git push
```

Watch the pipeline run, and generate a new version of the image in ECR automatically when the commit happens.
![image](https://user-images.githubusercontent.com/116161693/212901446-7f9c7aa7-a507-4e9b-a24a-fd187dba2de4.png)

![image](https://user-images.githubusercontent.com/116161693/212901485-91a9d83a-e71b-4803-9200-467f9d32dd83.png)

# STAGE 4 - CODE DEPLOY

In this stage, you will configure automated deployment of the cat pipeline application to ECS Fargate

## Configure a load balancer

First, you're going to create a load balancer which will be the entry point for the containerised application

Go to the EC2 Console (https://us-east-1.console.aws.amazon.com/ec2/v2/home?region=us-east-1#Home:) then Load Balancing -> Load Balancers -> Create Load Balancer.  
Create an application load balancer.  
Call it `catpipeline`  
Internet Facing
IPv4
For network, select your default VPC and pick ALL subnets in the VPC.  
Create a new security group (this will open a new tab)
Call it `catpipeline-SG` and put the same for description
Delect the default VPC in the list
Add an inbound rule, select HTTP and for the source IP address choose 0.0.0.0/0
Create the security group.  

Return to the original tab, click the refresh icon next to the security group dropdown, and select `catpinepine-SG from the list` and remove the default security group.  

Under `listners and routing` make sure HTTP:80 is configured for the listner.  
Create a target group, this will open a new tab
call it `catpipelineA-TG`, ensure that `IP`, HTTP:80, HTTP1 and the default VPC are selected.  
Click next and then create the target group, for now we wont register any targets.  
Return to the original tab, hit the refresh icon next to target group and pick `catpipelineA-TG` from the list.  
Then create the load balancer. 
This will take a few minutes to create, but you can continue on with the next part while this is creatign.

![image](https://user-images.githubusercontent.com/116161693/212902060-3c5ddb94-2dee-436f-b6b0-bd30420dbbcf.png)

## Configure a Fargate cluster

Move to the ECS console (https://us-east-1.console.aws.amazon.com/ecs/home?region=us-east-1#/getStarted)
Clusters, Create a Cluster and it needs to be `Networking Only` (this will be using fargate)  
Move on, and name the cluster `allthecatapps`
We will be using the default VPC so there is no need to create one (don't check the box)
Create the cluster. **if you get a error at this point, it may be the first time you are using ECS and thats ok** exit the creation process and then rerun it. There is service activation happening behind the scenes when you first use it, and it can prevent you creating a cluster. exiting and retrying solves 99% of these issues, for the 1% wait 10 minutes and then repeat.


## Create Task and Container Definitions

Go to the ECS Cluster (https://us-east-1.console.aws.amazon.com/ecs/home?region=us-east-1#/clusters)  
Move to `Task Definitions` and create a task definition.  
Select Fargate and move on.  
Call it `catpipelinedemo` and for `operating system family` put `Linux`  
Select `ecsTaskExecutionRole` under task role.  
Pick `1GB` for task member and `0.5vCPU` for task CPU.  
Add Container
Container name `catpipeline`
For image put the URL to your image in ECR
	to get this, move to ECR console, repositories, click your repo, then click `Copy URI`
Add a port mapping `80` `tcp`
Add the container.

![image](https://user-images.githubusercontent.com/116161693/212902189-18a35f0c-591d-4267-9c35-0e7cb1cf54f1.png)

Create (_if you get an error here, click back and then create again_)  
View Task Definition & Click `JSON`, copy the json down somewhere as the `task definition json`  


## DEPLOY TO ECS - CREATE A SERVICE
Click Actions, Create Service.

![image](https://user-images.githubusercontent.com/116161693/212902327-4a804058-f575-46c1-a89b-42d3c76fe156.png)

for `Launch type` pick `FARGATE`
for `Service Name` pick `catpipelineservice`  
for `Number of tasks` pick 2
for `Deployment type` pick `rolling update`
Next Step
for `Cluster VPC*` pick the default VPC
for `Subnets` select all subnets in the default VPC
for `Auto-Assign public IP` choose `ENABLED`  
for `Load Balancer Type` pick `Application Load Balancer`
for `Load balancer name` pick `catpipeline`  
for `container to load balance` select 'catpipeline:80:80' and click `Add to load balancer`
for `Production listener port` select `80:HTTP` from the dropdown
for `Target group name` pick `catpipelineA-TG`  
Next
Next
Create Service
View Service

![image](https://user-images.githubusercontent.com/116161693/212902422-2e92ea1b-13bc-49a9-a22d-98c11be10e43.png)

The service is now running with the :latest version of the container on ECR, this was done using a manual deployment

![image](https://user-images.githubusercontent.com/116161693/212902460-bf429d84-f8b5-4df5-934c-a205726acfdb.png)

## TEST

Move to the load balancer console (https://us-east-1.console.aws.amazon.com/ec2/v2/home?region=us-east-1#LoadBalancers)  
Pick the `catpipeline` load balancer  
Copy the `DNS name` into your clipboard  
Open it in a browser, ensuring it is using http:// not https://  
You should see the container of cats website - if it fits, i sits

![image](https://user-images.githubusercontent.com/116161693/212902539-302e9da1-e34a-4989-8cb0-c39f292df5e4.png)

## ADD A DEPLOY STAGE TO THE PIPELINE

Move to the code pineline console (https://us-east-1.console.aws.amazon.com/codesuite/codepipeline/pipelines?region=us-east-1)
Click `catpipeline` then `edit`
Click `+ Add Stage`  
Call it `Deploy` then Add stage  
Click `+ Add Action Group`  
for `Action name` put `Deploy`  
for `Action Provider` put `Amazon ECS`  
for `Region` pick `US East (N.Virginia)`  
for `Input artifacts` select `Build Artifact`  (this will be the `imagedefinitions.json` info about the container)  
for `Cluster Name` pick `allthecatapps`  
for `Service Name` pick `catpipelineservice`  
for `Image Definitions file` put `imagedefinitions.json`  
Click Done
Click Save & Confirm

## TEST

in the local repo `catpipeline-codecommit` run `touch test.txt`  
then run

```
git add -A .
git commit -m "test pipeline"
git push
```
 
watch the code pipeline console (https://us-east-1.console.aws.amazon.com/codesuite/codepipeline/pipelines/catpipeline/view?region=us-east-1)

make sure each pipeline step completes


