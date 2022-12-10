# How to create S3 Static Website using S3 on AWS
![image](https://user-images.githubusercontent.com/116161693/206851953-9d453e30-07b4-4cbf-9b6b-547f1896bfc0.png)
### steps
Create a bucket on S3 named **mywebsitebucket-nov-2022**

![image](https://user-images.githubusercontent.com/116161693/206852843-aa351664-738f-4c30-8ccd-9f9bdfbcf111.png)

**Note:** S3 bucket must have a unique bucket name and written in initial caps

Go to <b>S3 </b> > select your bucket > Edit Public Acess Settings > unselect "Block all public access" > Save > Confirm<br>

![image](https://user-images.githubusercontent.com/116161693/206855021-1a60a65f-ebd2-42e1-b3b4-47ada0647166.png)

Create three files, <b>index.html</b>, <b>error.html</b>, <b>about.html</b><br>

<a href="https://github.com/isaacarnault/aws-s3-static-website" target="_blank">Link</a>. <br>

Upload the files in the S3 bucket

![image](https://user-images.githubusercontent.com/116161693/206856114-b295eb24-6300-4bea-9a89-006099e175d5.png)

Files uploaded on S3 bucket
![image](https://user-images.githubusercontent.com/116161693/206854348-c2f25a5f-cf31-4574-8ebb-dc3fd82aa3c4.png)

Click on your bucket > select your 3 files > click on Actions > Make public<br>

![image](https://user-images.githubusercontent.com/116161693/206856244-772a34a4-26d6-4a54-92ca-aed2b365c1be.png)

Click on Properties > Static Website Properties > select "Use this bucket to host a website" > Index document: type index.html > Error document: type error.html

![image](https://user-images.githubusercontent.com/116161693/206855824-6dd4a989-f62d-4e56-b337-3e5efefe0d64.png)

Click on Static Website Hosting to get the endpoint url.

![image](https://user-images.githubusercontent.com/116161693/206855917-4c0d6496-2f50-4de0-a7cc-c187ed5cded8.png)

**Congratulations!** Our website is online, you can visit it using the provided endpoint url.
![image](https://user-images.githubusercontent.com/116161693/206854840-ca8dd045-2a74-475c-8eff-4908862d917e.png)


