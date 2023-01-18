# MIGRATING A DATABASE WITH THE DATABASE MIGRATION SERVICE

I will be migrating a simple web application (wordpress) from an on-premises environment into AWS.  
The on-premises environment is a virtual web server (simulated using EC2) and a self-managed mariaDB database server (also simulated via EC2)  
We will be migrating this into AWS and running the architecture on an EC2 webserver and RDS managed SQL database.  

This project involves 4 stages :-

- STAGE 1 : Provision the environment and review tasks
- STAGE 2 : Establish Private Connectivity Between the environments (VPC Peer)
- STAGE 3 : Create & Configure the AWS Side infrastructure (App and DB)
- STAGE 4 : Migrate Database & Cutover

![image](https://user-images.githubusercontent.com/116161693/213124145-040e4b7f-4387-4527-b690-53ff34f44723.png)

# STAGE 1 - Provision the environment and review tasks

Click https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/quickcreate?templateURL=https://learn-cantrill-labs.s3.amazonaws.com/aws-dms-database-migration/DMS.yaml&stackName=DMS to apply the base lab infrastructure  

You should take note of the `parameter` values 

- DBName
- DBPassword
- DBRootPassword
- DBUser

![image](https://user-images.githubusercontent.com/116161693/213126272-72bd1e30-52a9-41db-bfe1-ed787e2d803d.png)

You will need all of these in later stages.  
All defaults should be pre-populated, you just need to scroll to the bottom, check the capabilities box and click `Create Stack`  

Once the stack is in the `CREATE_COMPLETE` status you will have a simulated `on-premises` environment and an AWS environment.
Move to the EC2 console https://console.aws.amazon.com/ec2/v2/home?region=us-east-1  
Click `Running Instances`  
Select the `CatWEB` instance  
Copy down its `Public IPv4 DNS` into your clipboard and open it in a new tab.  
You should see the `Animals4life Hall of Fame` load as shown below; this is running from the simulated onpremises environment using the CatDB mariaDB instance.  

![image](https://user-images.githubusercontent.com/116161693/213126705-dbd816b7-25ca-441c-919d-4de84e642c49.png)

# STAGE 2 - Establish Private Connectivity Between the environments (VPC Peer)

Move to the VPC Console https://console.aws.amazon.com/vpc/home?region=us-east-1#  
Click on `Peering Connections` under `Virtual Private Cloud`  
Click `Create Peering Connection`  
for `Peering connection name tag` choose `A4L-ON-PREMISES-TO-AWS`  
for `VPC (Requester)` choose `onpremVPC`  
for `VPC (Accepter)` choose `awsVPC`  
Scroll down and click `Create Peering Connection`  

![image](https://user-images.githubusercontent.com/116161693/213128789-e61ea325-b74e-4720-b63c-f458368d55ea.png)

click `Actions` and then `Accept Request`  
Click `Accept Request`  
![image](https://user-images.githubusercontent.com/116161693/213127431-f71de7ba-ca75-461f-b6c5-343bdb1c3a4d.png)

**Create Routes on the On-premises side**

Move to the route tabes console https://console.aws.amazon.com/vpc/home?region=us-east-1#RouteTables:sort=routeTableId  
Locate the `onpremPublicRT` route table and select it using the checkbox.  
Click on the `Routes` Tab.  
You're going to add a route pointing at the AWS side networking, using the VPC Peer.  
Click `Edit Routes`  
Click `Add Route`  
For Destination enter `10.16.0.0/16`  
Click the `Target` dropdown & click `Peering Connection` and select the `A4L-ON-PREMISES-TO-AWS` then click `Save Changes`  
The Onpremises network can now route to the AWS Network, but as data transfer requires bi-directional traffic flow, you need to do the same at the other side.

![image](https://user-images.githubusercontent.com/116161693/213128733-3b0c271d-1b34-4a34-8718-425ca2659a30.png)

**Create Routes on the AWS side**

Move to the route tabes console https://console.aws.amazon.com/vpc/home?region=us-east-1#RouteTables:sort=routeTableId  
Locate the `awsPublicRT` route table and select it using the checkbox.  
Click on the `Routes` Tab.  
You're going to add a route pointing at the AWS side networking, using the VPC Peer.  
Click `Edit Routes`  
Click `Add Route`  
For Destination enter `192.168.10.0/24`  
Click the `Target` dropdown & click `Peering Connection` and select the `A4L-ON-PREMISES-TO-AWS` then click `Save Changes`  

![image](https://user-images.githubusercontent.com/116161693/213128661-1ce01c37-7daa-494f-ae42-6b8adb4ce805.png)

Move to the route tabes console https://console.aws.amazon.com/vpc/home?region=us-east-1#RouteTables:sort=routeTableId  
Locate the `awsPrivateRT` route table and select it using the checkbox.  
Click on the `Routes` Tab.  
You're going to add a route pointing at the AWS side networking, using the VPC Peer.  
Click `Edit Routes`  
Click `Add Route`  
For Destination enter `192.168.10.0/24`  
Click the `Target` dropdown & click `Peering Connection` and select the `A4L-ON-PREMISES-TO-AWS` then click `Save Changes` 

![image](https://user-images.githubusercontent.com/116161693/213128533-d0c205b2-28c7-4a97-9e4f-73e69ac9b432.png)

At this point we have created the peering connection between the VPCs and the gateway objects within each VPC.  
we have also configured routing from ONPremises -> AWS and vice-versa.  

# STAGE 3 - Create & Configure the AWS Side infrastructure (App and DB)

**CREATE THE RDS INSTANCE**

Move to the RDS Console https://console.aws.amazon.com/rds/home?region=us-east-1  
Click `Subnet Groups`  
Click `Create DB Subnet Group`  
For `Name` call it `A4LDBSNGROUP`
enter the same for description
in the `VPC` dropdown, choose `awsVPC`  
Under availability zones, choose `us-east-1a` and `us-east-1b`  
for subnets check the box next to `10.16.32.0/20` which is privateA and `10.16.96.0/20` which is privateB  
Scroll down and click `Create`  

![image](https://user-images.githubusercontent.com/116161693/213132697-fc3e4d6b-4ad3-4f79-af57-ee82c932d940.png)

Click on `Databases`  
Click `create Database`  
Choose `Standard Create`  
Choose `MariaDB`
Choose `Free Tier` for `Templates`  
![image](https://user-images.githubusercontent.com/116161693/213132768-b2befd0e-c816-4a3a-9ed7-8f263fbd4576.png)

we will be using the same database names and credentials to keep things simple, but note that in production this could be different.

for `DB instance identifier` enter `a4lwordpress`  
for `Master username` choose `a4lwordpress`  
for `Masterpassword` enter the DBPassword parameter for cloudformation which you noted down in stage of this demo
enter that same password in the `Confirm password` box  

![image](https://user-images.githubusercontent.com/116161693/213133256-7e7785c3-a030-4083-a91d-2840bd0071f2.png)

Scroll down to `Connectivity`  
for `Virtual private cloud (VPC)` choose `awsVPC`  
make sure `Subnet Groups` is set for `a4ldbsngroup`  
for `public access` choose `No`  
for `VPC security groups` select `Choose Existing` and choose  `***-awsSecurityGroupDB-***` (*** aren't important)  
remove the `Default` security group by clicking the `X`  

![image](https://user-images.githubusercontent.com/116161693/213133481-2377cee0-f045-43c7-ad3a-25592ce209f8.png)

Scroll down and expand `Additional configuration`  
Under `Initial database name` enter `a4lwordpress`  
Scroll down and click `Create Database`  
![image](https://user-images.githubusercontent.com/116161693/213133551-cc52c0e4-a10e-4786-b989-b120d08ce94e.png)

This will take some time.. and we cant continue to next stage until the database is in a ready state.
![image](https://user-images.githubusercontent.com/116161693/213133654-70538f56-4e5b-4bd6-a160-a7f1659ccb8e.png)

**CREATE THE EC2 INSTANCE**

Move to the EC2 Console https://console.aws.amazon.com/ec2/v2/home?region=us-east-1#Home:  
Click `Instances`  
Click `Launch Instances`  
Enter `awsCatWeb` for the Name of the instance.  
Clck `Amazon Linux` and ensure `Amazon Linux 2 AMI (HVM) ...... SSD Volume Type` is selected.  
Ensuring the architecture is set to `64-bit (x86)` below.   
![image](https://user-images.githubusercontent.com/116161693/213133787-bc5718fa-158d-4888-bdd2-0884de5f2498.png)

Choose the `Free Tier eligable` instance (should be t2.micro or t3.micro)  
Scroll down and choose `Proceed without key pair (not recommended)` in the dropdown  
Next to `Network Settings` click `Edit`  
For `VPC` pick `awsVPC`  
For `Subnet` pick `aws-PublicA`  
Select `Select an existing security group`  
Choose `***-awsSecurityGroupWeb-***` (*** aren't important) 
![image](https://user-images.githubusercontent.com/116161693/213133901-1d38963b-ec8f-4b09-8648-453539245d04.png)
![image](https://user-images.githubusercontent.com/116161693/213134016-5b6a4e20-4b9a-4abc-98b3-a503220b0eef.png)

Scroll down past Storage and expand `Advanced Details` (don't confuse this with `Advanced Network Configuration` in the current area)  
for `IAM Instance Profile` pick `***-awsInstanceProfile-****` (*** aren't important)  
Click `Launch Instance` 

![image](https://user-images.githubusercontent.com/116161693/213134067-e3bc387d-ca58-45c4-8f1c-6d3728db1f4a.png)

Click `View All Instances` 

![image](https://user-images.githubusercontent.com/116161693/213134089-604a85c7-5854-465b-b3c4-8e329754e764.png)

Wait for the `awsCatWeb` instance to be in a `Running` state with `2/2 checks` before continuing.

**Install Wordpress Requirements**

Select the `awsCatWeb` instance, right click, `Connect`  
Select `Session Manager` and click `Connect`  
When connected run the below command to run a privileged bash shell 

```
sudo bash
``` 
then update the instance with the below commnad 

```
yum -y update
```  
![image](https://user-images.githubusercontent.com/116161693/213134710-21b539e2-9be5-4029-90eb-4bf468be2a74.png)

Then install the apache web server with the below command - (the mariadb part is for the mysql tools)

```
yum -y install httpd mariadb
```

Then install php with the below command

```
amazon-linux-extras install -y lamp-mariadb10.2-php7.2 php7.2 
```  
![image](https://user-images.githubusercontent.com/116161693/213134933-efbc81dc-4269-480e-935d-e2a84dd16352.png)


![image](https://user-images.githubusercontent.com/116161693/213134834-c8e2a163-b536-49a3-ab68-6ace824cd8ee.png)

then make sure apache is running and set to run at startup with 

```
systemctl enable httpd
systemctl start httpd
```
![image](https://user-images.githubusercontent.com/116161693/213135017-25bee095-5579-4314-9087-668bfb18cb2a.png)

You now have a running apache web server with the ability to connect to the wordpress database (currently running onpremises)


**MIGRATE WORDPRESS Content over**

we're going to edit the SSH config on this machine to allow password authentication on a temporary basis.  
You will use this to copy the wordpress data across to the awsCatWeb machine from the on-premises CatWeb Machine  

run the below command

```
nano /etc/ssh/sshd_config
```
locate `PasswordAuthentication no` and change to `PasswordAuthentication yes` , then `ctrl+o` to save and `ctrl+x` to exit.  

![image](https://user-images.githubusercontent.com/116161693/213136161-788a05fe-c59c-4f2a-90be-22f663e72be0.png)

then set a password on the ec2-user user  
run the below command

```
passwd ec2-user
```
and enter the `DBPassword` you noted down at the start of the project.  

![image](https://user-images.githubusercontent.com/116161693/213135486-09bc287c-e345-41a2-a499-ec9ef74c0553.png)

**this is only temporary.. we're using the same password throughout the demo to make things easier and less prone to mistakes**

restart SSHD to make those changes with 
```
service sshd restart
```  
or 
```
systemctl restart sshd
```

Return back to the EC2 console https://console.aws.amazon.com/ec2/v2/home?region=us-east-1#Instances:  
with the `awsCatWeb` instance selected, note down its `Private IPV4 Address` you will need this in a moment.  

Select the `CatWeb` instance, right click, `Connect`  
Select `Session Manager` and click `Connect`  
When connected type `sudo bash` to run a privileged bash shell  

move to the webroot folder by typing `cd /var/www/`  

run a `scp -rp html ec2-user@privateIPofawsCatWeb:/home/ec2-user` and answer `yes` to the authenticity warning.  
this will copy the wordpress local files from `CatWeb` (on-premises) to `awsCatWeb` (aws)

![image](https://user-images.githubusercontent.com/116161693/213137000-a14d5cc7-5abd-4de3-81d1-500712e55d70.png)

**now move back to the `awsCatWeb` server, if you dont have it open still, reconnect as per below**

Select the `awsCatWeb` instance, right click, `Connect`  
Select `Session Manager` and click `Connect`  
When connected type `sudo bash` to run a privileged bash shell

move to the `ec2-user` home folder by doing a `cd /home/ec2-user`  
then do an `ls -la` and you should see the html folder you just copied.  
`cd html`  

![image](https://user-images.githubusercontent.com/116161693/213136549-0140ddeb-53e7-4c1a-92af-6f19722157f5.png)

next copy all of these files into the webroot of `awsCatWeb` by doing a `cp * -R /var/www/html/`

**Fix Up Permissions & verify `awsCatWeb` works**

run the following commands to enforce the correct permissions on the files you've just copied across

```
usermod -a -G apache ec2-user   
chown -R ec2-user:apache /var/www
chmod 2775 /var/www
find /var/www -type d -exec chmod 2775 {} \;
find /var/www -type f -exec chmod 0664 {} \;
sudo systemctl restart httpd
```
![image](https://user-images.githubusercontent.com/116161693/213136605-75edb965-846f-4774-a2f0-460afc82da1b.png)

Move to the EC2 running instances console https://console.aws.amazon.com/ec2/v2/home?region=us-east-1#Instances:  
Select the `awsCatWeb` instance  
copy down its `public IPv4 DNS` into your clipboard and open it in a new tab  
if working, this Web Instance (aws) is now loading using the on-premises database.

![image](https://user-images.githubusercontent.com/116161693/213136748-690f6118-f85a-4e03-a83d-1bf450b1783f.png)

At this point you have a functional AWS based wordpress application instance.  
You have migrated content from the on-premises virtual machine (simulated) using SCP.  
And you have tested that its connects to the on-premises DB.
In the next stage you will migrate the on-premises DB to AWS using DMS.
**before you continue to the next stage, make sure the a4lwordpress RDS DB we created is in an Available state** 

# STAGE 4 - Migrate Database & Cutover

**CREATE THE DMS SUBNET GROUP**

https://console.aws.amazon.com/dms/v2/home?region=us-east-1#subnetGroup  
Click `Create Subnet Group`  
For Name and Description use `A4LDMSSNGROUP`
for `VPC` choose `awsVPC`  
for `Add Subnets` choose `aws-privateA` and `aws-privateB`  
Click `Create subnet group`

![image](https://user-images.githubusercontent.com/116161693/213138182-29b00c1d-558d-4e7b-938e-ee274fd386e3.png)

**CREATE THE DMS REPLICATION INSTANCE**

Move to the DMS Console https://console.aws.amazon.com/dms/v2/home?region=us-east-1#replicationInstances  
Click `create Replication Instance`  
for name enter `A4LONPREMTOAWS`
use the same for `Description`  

![image](https://user-images.githubusercontent.com/116161693/213138525-e4c56a82-090d-4a04-94d8-2de1fddc07b2.png)

for `Instance Class` choose `dms.t3.micro`  
for `VPC` choose `awsVPC`  
For `MultiAZ` make sure it's set to `dev or test workload (single AZ)`  
and for `Pubicly Accessible` uncheck the box  
Expand `Advanced security and network configuration`  
Ensure `a4ldmssngroup` is selected in `Replication subnet group`  
For `VPC security group(s)` choose `***-awsSecurityGroupDB-***`  
Click `Create`  

![image](https://user-images.githubusercontent.com/116161693/213138775-e30b3691-fce8-48a4-8ab8-a31a7d915921.png)

**CREATE THE DMS SOURCE ENDPOINT**

Move to https://console.aws.amazon.com/dms/v2/home?region=us-east-1#endpointList  
Click `Create Endpoint`  
For `Endpoint type` choose `Source Endpoint` and make sure that `Select RDS DB Instance` is UNCHECKED  
Under `Endpoint configuration` set `Endpoint identifier` to be `CatDBOnpremises`  
Under `Source Engine` set `mariadb`  
Under `Access to endpoint database` choose `Provide access information manually`  
Under `Server name` use the privateIPv4 address of `CatDB` (get it from EC2 console)  
For port `3306`  
For username `a4lwordpress`  
for password user the DBPassword you noted down in stage 1  
click `create endpoint`  

![image](https://user-images.githubusercontent.com/116161693/213138961-b9a80706-8f1e-4ec8-9321-ecd543fcca81.png)

**CREATE THE DMS DESTINATION ENDPOINT (RDS)**

Move to https://console.aws.amazon.com/dms/v2/home?region=us-east-1#endpointList  
Click `Create Endpoint`  
For `Endpoint type` choose `Target Endpoint`  
Check `Select RDS DB Instance`  
Select `a4lwordpress` in the dropdown  
It will prepopulate the boxes  
Under `Access to endpoint database` choose `Provide access information manually`  
For `Password` enter the DBPassword you noted down in stage1  
Scroll down and click `Create Endpoint`  

![image](https://user-images.githubusercontent.com/116161693/213139156-c48d5026-679c-4027-9464-ad178e9e70a3.png)

**TEST THE ENDPOINTS**

**make sure the replication instance is ready**
Verify by going to `Replication Instances` and make sure the status is `Available`  
Go back to `Endspoints`  

Select the `a4lwordpress` endpoint, click `Actions` and then `Test Connections`

![image](https://user-images.githubusercontent.com/116161693/213139521-0a57f829-708b-4bf0-aae4-a4b14e420a08.png)

Click `Run Test` and make sure after a few minutes the status moves to `successful` 

![image](https://user-images.githubusercontent.com/116161693/213139644-9a63f060-e1a8-479c-811c-28f87ebcdf90.png)

Go back to `Endpoints`  
Select the `catdbonpremises` endpoint, click `Actions` and then `Test Connections`  
Click `Run Test` and make sure after a few minutes the status moves to `successful`

![image](https://user-images.githubusercontent.com/116161693/213139705-45f8c217-3297-4342-9c1b-f570274b847e.png)

If both of these are successful you can continue to the next step.  

**Migrate**
Move to migration tasks  https://console.aws.amazon.com/dms/v2/home?region=us-east-1#tasks  
Click `Create task`  
for `Task identifier` enter `A4LONPREMTOAWSWORDPRESS`
for `Replication instance` pick the replication instance you just created  
for `Source database endpoint` pick `catdbonpremises`  
for `Target database endpoint` pick `a4lwordpress`  
for `Migration type` pick `migrate existing data` **you could pick and replicate changes here if this were a high volume production DB**  
for `Table mappings` pick `Wizard`  
Click `Add new selection rule`  
in `Schema` box select `Enter a Schema`  
in `Schema Name` type `a4lwordpress`  
Scroll down and click `Create Task` 

![image](https://user-images.githubusercontent.com/116161693/213139962-ae26d477-4400-47ef-a418-e520a71185b4.png)
![image](https://user-images.githubusercontent.com/116161693/213139975-eb67251d-833a-44de-af80-85e12135d3bf.png)

This starts the replication task and does a full load from `catdbonpremises` to the RDS Instance.  
It will create the task  
then start the task  
then it will be in the `Running` State until it moves into `Load complete`  

![image](https://user-images.githubusercontent.com/116161693/213140029-13af26f6-d61b-4f31-a010-1b9037070639.png)

At this point the data has been migrated into the RDS instance  

**Cutover the application instance**

Move to the RDS Console https://console.aws.amazon.com/rds/home?region=us-east-1#  
Click `Databases`  
Click `a4lwordpress`  
under `Endpoint & Port` copy the `endpoint` dns name into your clipboard  

Move to the EC2 console https://console.aws.amazon.com/ec2/v2/home?region=us-east-1#Instances:  
Select the `awsCatWeb` instance, right click, `Connect`  
Select `Session Manager` and click `Connect`  
When connected type `sudo bash` to run a privileged bash shell
run `cd /var/www/html`
run `nano wp-config.php`  
locate the define DB_HOST line, and reolace the IP address with the RDS Host you just copied down into you clipboard  
run `ctrl+o` to save and `ctrl+x` to exit.

![image](https://user-images.githubusercontent.com/116161693/213140266-da420cc6-37f4-447a-b9a2-f2bb0eeee9f4.png)

Run the script below, to update the wordpress database with the new instance DNS name

```
#!/bin/bash
source <(php -r 'require("/var/www/html/wp-config.php"); echo("DB_NAME=".DB_NAME."; DB_USER=".DB_USER."; DB_PASSWORD=".DB_PASSWORD."; DB_HOST=".DB_HOST); ')
SQL_COMMAND="mysql -u $DB_USER -h $DB_HOST -p$DB_PASSWORD $DB_NAME -e"
OLD_URL=$(mysql -u $DB_USER -h $DB_HOST -p$DB_PASSWORD $DB_NAME -e 'select option_value from wp_options where option_id = 1;' | grep http)
HOST=$(curl http://169.254.169.254/latest/meta-data/public-hostname)
$SQL_COMMAND "UPDATE wp_options SET option_value = replace(option_value, '$OLD_URL', 'http://$HOST') WHERE option_name = 'home' OR option_name = 'siteurl';"
$SQL_COMMAND "UPDATE wp_posts SET guid = replace(guid, '$OLD_URL','http://$HOST');"
$SQL_COMMAND "UPDATE wp_posts SET post_content = replace(post_content, '$OLD_URL', 'http://$HOST');"
$SQL_COMMAND "UPDATE wp_postmeta SET meta_value = replace(meta_value,'$OLD_URL','http://$HOST');"
```

![image](https://user-images.githubusercontent.com/116161693/213140314-b05a315c-1d4e-4bd2-9809-5a4e0b54c1b7.png)

Move to the EC2 console 
Select `CatDB`, right click `stop instance`   
Move to the EC2 console 
Select `CatWeb`, right click `stop instance`   

Move to the EC2 console 
Select `awsCatWeb` and get its `Public IPv4 DNS` 
Open this in a new tab
It should still show the application ... this is now pointed at the RDS instance after a full migration

![image](https://user-images.githubusercontent.com/116161693/213140508-ab1c5c98-ce19-4758-bf9d-49f519006d0e.png)

At this point you have created a VPC peer between the simulated On-premises environment and AWS
we have fully migrated the wordpress application files from on-premises (simulated) into AWS  
we have provisioned an RDS DB Instance  
And we have used DMS to perform a simple migration of the database from on-premises (simulated) to AWS  
