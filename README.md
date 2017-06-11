# AWS-S3-IAM-Elixir-Phoenix
[DRAFT] Step by Step Guide

Road Map
    Step 1 AWS (IAM) — Create an IAM user with permissions to upload and read from S3 (Full Access)..
    Step 2 AWS (S3) — Create an S3 bucket with the correct permissions and policies so that it can be accessed by our application.
    Step 3 Application Code — Write code to facilitate the image upload within your Elixir/Phoenix application.

AWS (IAM)

Hello & Welcome! We will have created a user in AWS IAM that has permissions to upload and read from an S3 bucket.


Let’s begin!

Before we write any application code we need to set up an AWS account. Once you’ve created an account, navigate to the IAM (Identity and Access Management) service. It can be found by searching in the AWS Services search bar. Click on the link to navigate to the IAM dashboard:


AWS Console

The IAM service allows you to create users with specific permissions. Permissions can be limited so that access is restricted to certain AWS services. We want to create a new user that has full access to S3.


IAM Dashboard

Click on the “Add User” button.


IAM Users Dashboard

Next give your user a name, this tutorial use s3-upload-tutorial, and then select the “Programmatic Access” checkbox. By selecting this we are saying that we want to give access to our application for the AWS SDK. Then click the “Next: Permissions” button.


IAM Add user — Step 1 Details

We then want to add permissions to the user we are creating. This can be done in a number of ways but for this tutorial we attache the permissions directly to the user. We want to give full access to S3 so search for S3 in the "search bar" and then choose the AmazonS3FullAccess policy, then click “Next: Review”.


IAM Add user — Step 2 Permissions

The review page shows you the details of the user that you are about to create. You should see a user with programmatic access and an attached policy for AmazonS3FullAccess. Click “Create User”.


IAM Add user — Step 3 Review

You should then see a "SUCCESS" message and your newly created user below. Make a note of your Access Key ID and Secret aAccess Key.


IAM Add user — Step 4 Complete

Nice! We have now created a user with permissions to upload to, and read from, AWS S3. 
