# AWS SecOnion Lab Setup - Installing and Running Windows 11 Agent in EC2 Instance

##  NOTE: ALL FILES NEEDED TO CONDUCT THE IMPORT PROCESS ARE INCLUDED IN THIS DIRECTORY! 
### YOU ONLY NEED TO MAKE MINOR EDITS TO ONLY TWO!

### Creating a CLI User

Navigate to the AWS IAM tab and select Users:
For cli access, first you must create a cli-user
NOTE: For security purposes, this should only be the user that's allowed to access the cli-interface and not the management gui


![Pasted image 20241013213704](https://github.com/user-attachments/assets/f7d0b444-ef3a-4d7a-998d-66009028cff6)



Attached the policies directly and provide admin access, for now:

![Pasted image 20241013213845](https://github.com/user-attachments/assets/d8d07c97-b6a1-4cd0-a4f1-1ff83178db94)

Click "Create user" 

Next, click on the user you just create and click the tab in Security Credentials



![Pasted image 20241013214100](https://github.com/user-attachments/assets/8d994958-007e-4475-9b64-079940c66395)



Scroll down to Create access key:


![Pasted image 20241013214146](https://github.com/user-attachments/assets/e47eec70-14c3-43dc-8120-15eee09a074e)


Create an access key
Select CLI



![Pasted image 20241013214254](https://github.com/user-attachments/assets/71eb986b-37c5-4ed2-8cc1-55d10997a6f8)



Download the csv and click done:

![Pasted image 20241013214432](https://github.com/user-attachments/assets/c35db91b-327b-422d-bdf0-7bdfca16b1d2)



### Install AWS CLI 

Navigate to web link to download the appropriate version
https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html


![Pasted image 20241014104518](https://github.com/user-attachments/assets/993573ca-807a-4d31-90fd-a90048112c64)




Select your choice, download the installer and install. 
To ensure successful installation, check your aws cli version 

```
aws --verion
```

### Configure AWS CLI

Next head to your terminal and connect to the AWS console:

```
aws configure
```
Add your Access Key ID & Secret Access Key

On initial setup choose your region.

The default output format is JSON.

<img width="270" alt="Pasted image 20241013214732" src="https://github.com/user-attachments/assets/0ae2b29c-c8cb-433d-8516-e5a20bff2d45">


### Creating an s3bucket

Log into AWS and create a bucket


<img width="896" alt="Pasted image 20241013163415" src="https://github.com/user-attachments/assets/67ad01ef-5272-4434-96c7-8b272108fba1">

-



<img width="1279" alt="Pasted image 20241013163518" src="https://github.com/user-attachments/assets/3c2beb3d-2bcb-4a31-a844-beaede3b60dc">



### Create VM Import Role 

Navigate to page to create the vmimport polyabdrole
https://docs.aws.amazon.com/vm-import/latest/userguide/required-permissions.html

Scroll down to the **"Required service role" section and complete instruction 1 & 2**

## NOTE: This document has been completed and is in this directory under the file named "trust-policy.json" - 

## (Do not make any edits to this document)

![Pasted image 20241014105926](https://github.com/user-attachments/assets/36ebf16c-eac7-4546-9ec1-55c437fa01ce)


Navigate to the folder where the file is stored and run he following command in the cli:

```
aws iam create-role --role-name vmimport --assume-role-policy-document "file://trust-policy.json"
```


Next, in the same section, complete step 3 & 6 (4 and 5 are optional and not needed for basic uploads):

### NOTE: The role-policy.json document has been provided:

### You need to edit 4 lines: 12,13,26 & 27

### replace <windows11-docs> with your bucket name:



![Pasted image 20241014111043](https://github.com/user-attachments/assets/3bf31f16-420a-451d-96d3-e5472c2ac116)
-

<img width="537" alt="Pasted image 20241014111338" src="https://github.com/user-attachments/assets/1ad122c5-bcdd-47d9-ab55-fca04396a5bb">


After adding your bucket, run the following command in the directory where the role-policy.json file is located:

```
aws iam put-role-policy --role-name vmimport --policy-name vmimport --policy-document "file://role-policy.json"
```


### Creating an OVA from a VM Image:

On you Virtual application, select the virtual machine and export to ova. 
If you're having an issue exporting to ova ensure you read the instruction from your virtual application documents. 





### Import The .OVA image to the s3bucket
## NOTE: UPLOADING AN OVA CAN TAKE ANYWHERE FROM 1 - 10 OR MORE HOURS TO COMPLETE

GUI takes long
CLI is quicker
An option to set a faster acceleration for large downloads is optional at a cost. 

In terminal, copy the .ova file to s3bucket you set up earlier

```
aws s3 cp win-aws.ova s3://windows11-docs/awswin.ova
```




### Run the AWSCommand to Create an Image
Navigate to page on import instructions:
https://docs.aws.amazon.com/vm-import/latest/userguide/import-vm-image.html

NOTE: The containers.json file has been added to this file: Only edit the following sections:

**Required: S3Bucket**

**Required: S3Key**

Optional: Description



<img width="585" alt="Pasted image 20241014112411" src="https://github.com/user-attachments/assets/06bfcf2f-5193-4c7b-9503-5ad8832c4aa6">



Run the following command in your terminal to create an AMI on your AWS account from the .ova you copied to the s3 bucket:


```
aws ec2 import-image --description "My server VM" --disk-containers "file://containers.json"
```



<img width="651" alt="Pasted image 20241014013934" src="https://github.com/user-attachments/assets/aec5e117-0c09-4e9a-a721-807f954b0134">



Record the **ImportTaskId** to check the status of the upload:

Grab your AMI import id:

<img width="518" alt="Pasted image 20241014014053" src="https://github.com/user-attachments/assets/52a6c298-49c8-4939-a073-c70f01ae5f0f">


#### Run the command to check the status: 
#### NOTE: Check the status occasionally to ensure no issues persist.

```
aws ec2 describe-import-image-tasks --import-task-ids import-ami-04d16fac2b6f61fa0
```


<img width="510" alt="Pasted image 20241014014212" src="https://github.com/user-attachments/assets/9a5b2b35-0293-4057-adb8-c122dcb24936">


After a long time you will eventually see the status is complete:

<img width="568" alt="Pasted image 20241014024431" src="https://github.com/user-attachments/assets/ef1d7922-47a7-4410-a9f9-36e3465761c7">


Proceed to EC2 > Launch instances to see if your AMI is properly uploaded:

<img width="1399" alt="Pasted image 20241014024541" src="https://github.com/user-attachments/assets/09a47976-cc7d-4088-a2f6-add98a07a4ca">


### Launch The Uploaded AMI Instance

Head to launch instance and Browse more AMIs. 

![Pasted image 20241014115648](https://github.com/user-attachments/assets/cee5745a-4e4a-45f0-ad51-2600c96c16f2)


Click on My AMIs tab and select the AMI:

![Pasted image 20241014115831](https://github.com/user-attachments/assets/d750f517-a82e-46a9-9f7b-2b4b8bb95edd)


Name your new image:
Select the appropriate storage:
WIndows requires a minimum (2vCPU 4GiB Memory)
Storage Selection
t2.medium or a t3.medium


Select or create a key pair:

![Pasted image 20241014120212](https://github.com/user-attachments/assets/9edb97b5-2dfd-41c3-980b-89cf29eeb22a)


Networking:

Ensure you adjust the following:

VPC: Same as the SOC Lab

Subnet: Same as the SOC Lab (Public1)

Auto-assign public IP: Enabled
Security Groups: Set Appropriate or Create New: Ensure rule added to allow traffic across VPC:


![Pasted image 20241014120839](https://github.com/user-attachments/assets/1c9ce837-c940-4e8b-8c66-cd0f87d07c33)


Launch the instance and wait:


### Logging into the Windows AMI:

#### NOTE: You will need a New Download file after EACH instance boot up

After a mandatory 4 minute wait post instance launch Connect to the new windows host via RDP:


Download the "Remote desktop file"

<img width="897" alt="Pasted image 20241014161139" src="https://github.com/user-attachments/assets/cc7d4559-9080-45b8-bc21-a38286ac191f">


The file will be sent to your download folder:


<img width="587" alt="Pasted image 20241014161231" src="https://github.com/user-attachments/assets/12d6d67f-882d-4fc5-842b-0f8d6344220f">


Click the "Get Password"

<img width="988" alt="Pasted image 20241014160549" src="https://github.com/user-attachments/assets/90d3283d-5fd2-4746-a0c1-ba0aa8c3d343">



Next, upload the .pem key you used to create the instance and click "Decrypt":


<img width="855" alt="Pasted image 20241014160949" src="https://github.com/user-attachments/assets/403e6bdf-cfad-47e3-a619-4446a1abd144">

Copy the password just below the arrow:


<img width="749" alt="Pasted image 20241014161335" src="https://github.com/user-attachments/assets/216f157b-c0d8-48a1-8f9c-4e0d799f954c">

Enter the password in the RDP prompt:

<img width="432" alt="Pasted image 20241014161445" src="https://github.com/user-attachments/assets/cec32cd8-6374-45ec-9c67-23d221c4c4d8">


You are now logged in:

<img width="883" alt="Pasted image 20241014161522" src="https://github.com/user-attachments/assets/f323488e-c7fd-4698-b908-be60448fc4f7">


