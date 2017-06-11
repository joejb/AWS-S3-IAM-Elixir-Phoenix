Road Map

  Step 1 AWS (IAM) — Create an IAM user with permissions to upload and read from S3 (Full Access).

  Step 2 AWS (S3) — Create an S3 bucket with the correct permissions and policies so that it can be accessed by our application.

  Step 3 Application Code — Write code to facilitate the image upload within your Elixir/Phoenix application.


AWS (S3)

Hello again & Welcome. Step 2 - Image Uploads with S3, Elixir + Phoenix.

In Step 1 we created a new IAM user with permissions to upload to, and read from, AWS S3. By the end of this step you will have created an S3 bucket that used to store all of your images.


Let’s get started!

Firstly, sign in to your AWS account and navigate to the S3 service. The service can be found just as we did the IAM service in Step 1.


S3 Dashboard

Click on the "Create Bucket" button just below the search bar. You will then be asked to create some details about the bucket you want to create. Enter a bucket name and select a region that you want the bucket to apply to and then click "Create".


S3 Create bucket

You should see the newly created bucket in your dashboard. Click on it.


Bucket created

Click on the "Permissions" tab at the top and then click on "Everyone". Select the tick boxes for "read only" and then click "Save". You should see a user that already has read and write access at the top:

Next click on the "Bucket Policy" button beneath the tabs. It will then prompt you to enter a bucket policy. Enter the following, replacing "your_bucket_name" with the name you chose for your bucket, and then press "Save":

{
  "Version": "2008-10-17",
  "Statement": [
  {
  "Sid": "AddPerm",
  "Effect": "Allow",
  "Principal": "*",
  "Action": [
  "s3:GetObject",
  "s3:PutObject"
  ],
  "Resources": "arn:aws:s3:::your_bucket_name"
  }
 ]
}

Cool! We have now successfully created an S3 bucket with the correct permissions and policies that will allow us to access it within our application. Check out Step 3 where we will write the application code that will interact with our newly created bucket!
