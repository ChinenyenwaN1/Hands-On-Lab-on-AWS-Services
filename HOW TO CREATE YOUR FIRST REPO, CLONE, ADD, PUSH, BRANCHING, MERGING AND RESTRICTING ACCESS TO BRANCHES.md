# HOW TO CREATE YOUR FIRST REPO, CLONE, ADD, PUSH, BRANCHING, MERGING AND RESTRICTING ACCESS TO BRANCHES

Step 1: Step 1: Go to your [AWS account](https://aws.amazon.com/) and log into your AWS account. If you do not have an account, proceed by creating a free account. Once you log-in, you should see a page as shown below:

![image](https://user-images.githubusercontent.com/116161693/203466803-2c433c6d-19a3-470c-9a7a-a45f688f6a1f.png)

Search for CodeCommit and click on that service. Further, click on Create Repository to create a repository.

![image](https://user-images.githubusercontent.com/116161693/208099989-04402a35-7cc5-4a90-928f-8c812b8e0e3a.png)

You’ll be prompted to add your Repository Name and Description. Add those and click on Create.

![image](https://user-images.githubusercontent.com/116161693/208100035-01f8d4b9-0847-4411-88c8-165bba727b03.png)

You should get a success message as I got.
There are two ways of connecting your repository – SSH and HTTPS. In this case, I’ll be using HTTPS. Now that a repository has been created, go ahead and create files in the repository. When you create a repository, it’s always empty. You’ll have to create and add files. We will be using Cloud9 environment for this tutorial

![image](https://user-images.githubusercontent.com/116161693/208100108-0c7f94a8-8530-44a5-9978-08e4d55e94c9.png)

Once you’ve created the file. Go ahead and clone your repository. 
This shows that our repository is empty

![image](https://user-images.githubusercontent.com/116161693/208100417-a2d7f06a-9c60-4b6c-a0ed-a5a1942a517c.png)
 
Now let’s change our directory to add our files.

![image](https://user-images.githubusercontent.com/116161693/208100522-d1e43e8d-30f8-468f-95ba-92012bb03ef0.png)

This is the link the code , copy it to my-webpage directory and run ls command to confirm if they were added successfully.

![image](https://user-images.githubusercontent.com/116161693/208100558-b7e7b763-b677-4a09-825e-e9c6a00868d7.png)

run git status to confirm status of the repository

![image](https://user-images.githubusercontent.com/116161693/208101620-4909abe4-1097-4996-ae89-26daafc992a1.png)

Run “git add .” to add files to the master branch
 
![image](https://user-images.githubusercontent.com/116161693/208101671-83b40019-fc4a-4327-b87b-0dbac3227ac5.png)

next, run git commit -m "first commit" and hit enter key

![image](https://user-images.githubusercontent.com/116161693/208101842-9dc42cf8-6d58-4ad5-a4d6-3bd1275620bb.png)

its time to push our files to my-webpage branch, run
```
git push
```

![image](https://user-images.githubusercontent.com/116161693/208102233-c4ead294-8593-4287-99bb-d02411d924ac.png)

Refresh the codecommit page and the files have been added as shown below

![image](https://user-images.githubusercontent.com/116161693/208102944-b0f6627d-a5d7-4d89-9ee7-f29be10e3c02.png)

click on commit and you will see a similar page like this

![image](https://user-images.githubusercontent.com/116161693/208103122-9c2c82ec-d453-47b8-9b30-dda1c7c9218a.png)

let's check our master branch 

![image](https://user-images.githubusercontent.com/116161693/208103189-a515ee2a-7660-4551-993c-7fc4aef7dae8.png)

Open index.html and edit the file

![image](https://user-images.githubusercontent.com/116161693/208103243-788b63a6-99cb-4d84-8168-8a18e0885341.png)

let's see what happens when we edit our index.html by running the below command

```
git status
```

```
run git add . index.html
```

```
git commit -m "modified index to Chinenye"
```

```
git push
```

the above commands will add the edited index.html to the master's branch

![image](https://user-images.githubusercontent.com/116161693/208103391-c0146ab3-d26f-402a-a0a4-0f35541ed479.png)

![image](https://user-images.githubusercontent.com/116161693/208104559-8e780f71-153a-4fd2-a7b8-e764f6467e86.png)

Refresh the codecommit page and the files have been added

![image](https://user-images.githubusercontent.com/116161693/208104688-e8cd6c72-cd48-4e9e-a759-77e26ecb0d1e.png)
 
Click on Commit, this highlights the changes made in the master branch

![image](https://user-images.githubusercontent.com/116161693/208104730-ae06c5fc-27ce-4b2b-90d8-b6eb79fc8e9b.png)

Now let’s see how we can create a branch, we have only one default branch called master branch. run

```
git checkout -b "my-feature"
```

![image](https://user-images.githubusercontent.com/116161693/208105044-2c978113-9409-4f77-969b-b3c6d54bc433.png)

let us edit the index.html file and push code to the new branch created by running the below command.

```
git status
```

```
git add .
```

```
git commit -m "modified index to Chinenye Nwogeh"
```

```
git push
```


![image](https://user-images.githubusercontent.com/116161693/208106130-5db8b025-e0ed-470d-b76a-6ac98afafce4.png)

### Branches are basically copies of the original file which can be allocated to different people. They can make changes, commit them and push it to CodeCommit. After certain tests when these codes are verified, they can be merged with the master branch. Branches we have and what we modified

this is the information in our master branch

![image](https://user-images.githubusercontent.com/116161693/208106931-19264783-e021-4e0f-8506-6e0fd3ba65fd.png)

and this is the edited information on my-feature branch

![image](https://user-images.githubusercontent.com/116161693/208107004-fdaa605c-614a-44b5-b98a-999559206985.png)

Once you click on the branch, you’ll see that it contains all the files that exist on your master branch.

![image](https://user-images.githubusercontent.com/116161693/208108706-0613c1ad-9581-4287-a2a0-3e3a7192e3a1.png)

Select the master branch as you’re comparing the current branch with the master branch. Click on Compare.

![image](https://user-images.githubusercontent.com/116161693/208108955-10eeaccb-5fa5-4150-a6d7-f33a86810db7.png)

Suppose you agree with the changes made in this branch and you’d like to reflect these changes to your master branch, you can merge the two branches. Add Title 

![image](https://user-images.githubusercontent.com/116161693/208109003-c22d5240-78f5-420d-b2e1-8258fafd8e6f.png)

and click on Create. 

![image](https://user-images.githubusercontent.com/116161693/208109015-205bd561-2c62-4315-9baf-5d99569bc760.png)
 
Click on Merge to finally merge the two branches.

![image](https://user-images.githubusercontent.com/116161693/208109048-65124392-cc1e-413b-b4b1-fa026bb64460.png)

Master’s branch has been update as shown below after merging the two branches
 
![image](https://user-images.githubusercontent.com/116161693/208109095-810016d4-4c85-4cb7-93cf-ba233674e141.png)

## How to restrict users from pushing code to branches
 
Create group for junior developers that will not be able to push to master branch

![image](https://user-images.githubusercontent.com/116161693/208109430-3514c166-fafd-4c40-8d0f-55c1e1b93dcb.png)

Create the below policy

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Deny",
            "Action": [
                "codecommit:GitPush",
                "codecommit:DeleteBranch",
                "codecommit:PutFile",
                "codecommit:MergeBranchesByFastForward",
                "codecommit:MergeBranchesBySquash",
                "codecommit:MergeBranchesByThreeWay",
                "codecommit:MergePullRequestByFastForward",
                "codecommit:MergePullRequestBySquash",
                "codecommit:MergePullRequestByThreeWay"
            ],
            "Resource": "arn:aws:codecommit:*:*:*",
            "Condition": {
                "StringEqualsIfExists": {
                    "codecommit:References": [
                        "refs/heads/main"
                     ]
                },
                "Null": {
                    "codecommit:References": "false"
                }
            }
        }
    ]
}
```

![image](https://user-images.githubusercontent.com/116161693/208109613-04b5aa30-08ac-4a17-bd04-363e2f7fd70a.png)

 
Add policy name and click on create policy
 
![image](https://user-images.githubusercontent.com/116161693/208109631-4d754859-666b-4241-9a0e-f7a3187ae982.png)

You should see something similar to the below,

![image](https://user-images.githubusercontent.com/116161693/208109657-4c430f72-4ffa-4276-aa70-cc9ef3b0f752.png)

Now add a user to the junior developer group created.

![image](https://user-images.githubusercontent.com/116161693/208109747-8eae0fc4-57c4-41bd-a2eb-4736f07b7df1.png)

Click on add users

![image](https://user-images.githubusercontent.com/116161693/208109777-c75be6a2-2a88-43bf-971b-ff97b4688e15.png)

User added successfully, now lets go to our terminal and test if this user can push code to the master
branch
![image](https://user-images.githubusercontent.com/116161693/208109827-3eae36de-315a-4378-8127-da2202ff5dcb.png)
 
**Note:** I have administrators access but the created explicit deny policy will make it impossible for me to 
Push into codecommit. 
![image](https://user-images.githubusercontent.com/116161693/208109895-9a79cd2d-078e-4592-9807-249c968cec95.png)


![image](https://user-images.githubusercontent.com/116161693/208110013-ac0d9592-58e7-40b8-9d3a-2585ac680274.png)

Now let’s edit the index.html again and run git status command:

![image](https://user-images.githubusercontent.com/116161693/208109932-8353f1aa-f138-447b-ab75-35f2faf4d64c.png)

This has changed.

![image](https://user-images.githubusercontent.com/116161693/208110076-828bb792-71e1-4d6d-be94-5aa8696a3499.png)

Lets commit it locally,

![image](https://user-images.githubusercontent.com/116161693/208110103-946403eb-bd52-4581-9db9-9d2b39a82b4b.png)

Now let push it to our branch
 
![image](https://user-images.githubusercontent.com/116161693/208110186-25f79249-48cc-41c8-aa03-211a6493fb49.png)

the junior developer is restricted from pushing code and the only option is to create a new branch and edit the file and make a pull request for the senior manage to verify changes made and merge changes. 
 
 ![image](https://user-images.githubusercontent.com/116161693/208110584-31984dff-d8ed-4569-b451-9930a5a0c505.png)

![image](https://user-images.githubusercontent.com/116161693/208110642-f6d24458-09ed-4773-a0af-cb37cbf50a5a.png)

![image](https://user-images.githubusercontent.com/116161693/208110664-15a8136d-1b9d-41fc-b01e-bdc0b7191534.png)

Now let’s refresh the page and check if the new branch has appeared.
 
![image](https://user-images.githubusercontent.com/116161693/208110747-4e64649e-afc3-4a21-b0cf-2f0a868e73c9.png)

Next is to create a pull request for the senior developers to check what has been edited 

![image](https://user-images.githubusercontent.com/116161693/208110772-d9d2b526-49f3-492a-94d2-f822a81b7e5f.png)

these are the changes made;

![image](https://user-images.githubusercontent.com/116161693/208110877-520ebe27-3a29-47e0-9c5e-06a3bd1cd15e.png)

You get a success pull request notification. This is a way to tell Senior developer to confirm changes made before merging.

![image](https://user-images.githubusercontent.com/116161693/208110920-e763498f-17a2-441b-9232-f1dca4bf8b8c.png)
 

Congratulations on getting to this stage.
Now let’s create a notification rule  trigger
Click on settings > notification> create notification rule

![image](https://user-images.githubusercontent.com/116161693/208111006-c7b257ff-c9ce-4310-87a6-3e981e5b28ea.png)
 
![image](https://user-images.githubusercontent.com/116161693/208111148-e5b81589-28c5-44f5-97b7-59ffb26e420e.png)

click on create target

![image](https://user-images.githubusercontent.com/116161693/208111249-1e08cf00-e7ad-460f-abf1-48023f377705.png)

![image](https://user-images.githubusercontent.com/116161693/208111269-80fabf41-71cf-4ced-97db-39629663a72e.png)

successful

![image](https://user-images.githubusercontent.com/116161693/208111368-00c4f5d4-3e60-454f-8a0c-579da6a352d7.png)

Next let us create trigger,

![image](https://user-images.githubusercontent.com/116161693/208111482-40225382-9ab3-4759-b543-56ae37a9ad7b.png)

![image](https://user-images.githubusercontent.com/116161693/208111531-54e96ddb-e564-45b9-9089-a129c4fb865c.png)

successful

![image](https://user-images.githubusercontent.com/116161693/208111639-9202cfac-bb41-43d1-a3fd-5b33b4ba8511.png)

 When we go to cloudwatch, you will notice that an event has been created.
 
![image](https://user-images.githubusercontent.com/116161693/208111696-1b425c6c-4c90-4ab5-953d-2af91d17bcac.png)


**Congratulations for following along to the end.**
