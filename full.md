# Lesson 13 - Host a Static Website Using

# S3 (Video)

Note: Download the videos archive here.


# Lesson 14 - Managing S3 Access & Policies

S3 offers many options to manage access, encryption, and logging. We are going to see how to
use these features and how to manage your S3 data stores.

## Cross-Origin Resource Sharing (CORS)

CORS defines how a web client loaded in one domain to interact with resources in a different
domain. You can, for instance, allow one domain to interact with a bucket or a static website
hosted on AWS S3.

Say, you have a website [http://www.mywebsite.com](http://www.mywebsite.com) that uses media content (e.g., photos, fonts ..)
stored in an S3 bucket.

You have the choice of allowing everyone to use this content, but it's better to restrict the access
to only your website. This could reduce your usage costs.

In order to configure CORS for S3, navigate to your S3 bucket then click on "Permissions" and
"CORS management":

In the CORS configuration editor, you will find the default configuration:

As you can notice, all origins are allowed by default, but if you want to allow only your domain
name, change the configuration like the following:

```
<!-- Sample policy -->
<CORSConfiguration>
<CORSRule>
<AllowedOrigin>*</AllowedOrigin>
<AllowedMethod>GET</AllowedMethod>
<MaxAgeSeconds>3000</MaxAgeSeconds>
<AllowedHeader>Authorization</AllowedHeader>
</CORSRule>
</CORSConfiguration>
```

What if you are using a CloudFront distribution (with the domain mywebsite.com) that uses
HTTPS?

In this case, you should choose the right protocol because you can configure your CloudFront to
force redirect HTTP to HTTPS, or you have the choice to use both HTTP and HTTPS.

In the first case, make sure to add https://mywebsite.com instead of [http://mywebsite.com;](http://mywebsite.com;)
otherwise, the CloudFront distribution will not be able to access your bucket.

When you download your images and fonts from the S3 bucket, you will normally use the GET
method.

What if you use POST or DELETE methods for other scenarios?

This can be resolved by changing your configuration and add the methods you will use.

Another rule that you can configure is the **MaxAgeSeconds**. It’s the time in seconds during which
the browser will cache the response to a preflight OPTIONS request. When a browser caches this
response, it will not make any new OPTIONS request during the configured time. The default
value is 3000 seconds.

Note: According to the RFC2616, this method (OPTIONS) allows the client to determine the
options and/or requirements associated with a resource, or the capabilities of a server, without
implying a resource action or initiating a resource retrieval.

If the bucket allows HEAD, GET, PUT, DELETE, OPTIONS methods, the client should receive a
response similar to this one:

Note: A preflight request is a CORS request that checks to see if this protocol (CORS) is
understood.

Some other rules could be added like **ExposeHeader** that allows the client to access to a header.
This example allows the client to access the "x-amz-request-id" header.

```
<CORSConfiguration>
<CORSRule>
<AllowedOrigin>http://mywebsite.com</AllowedOrigin>
<AllowedMethod>GET</AllowedMethod>
<MaxAgeSeconds>3000</MaxAgeSeconds>
<AllowedHeader>Authorization</AllowedHeader>
</CORSRule>
</CORSConfiguration>
```
```
<CORSConfiguration>
<CORSRule>
<AllowedOrigin>http://mywebsite.com</AllowedOrigin>
<AllowedMethod>GET</AllowedMethod>
<AllowedMethod>POST</AllowedMethod>
<AllowedMethod>DELETE</AllowedMethod>
<MaxAgeSeconds>3000</MaxAgeSeconds>
<AllowedHeader>Authorization</AllowedHeader>
</CORSRule>
</CORSConfiguration>
```
##### 200 OK

```
Allow: HEAD,GET,PUT,OPTIONS
```

Note: Some of AWS headers are only related to AWS. This type of header starts with "x-amz-".
These are some of them:

```
x-amz-content-sha256
x-amz-date
x-amz-security-token
```
**AllowedHeader** helps you to create a rule in order to specify which headers are allowed in a
preflight request through the **Access-Control-Request-Headers** header

Note: The Access-Control-Request-Headers request header is used when sending a preflight
request to let the distant server know which HTTP headers will be used. This is an example of an
Access-Control-Request-Headers response:

By default, the only authorized header in an S3 bucket is: **Authorization**

Note: We can use the "*" wildcard character in order to let many headers.

e.g:

or

The wildcard could also be used with other elements like the **AllowedOrigin** :

e.g.:

It is possible to create different configurations in the same file. In this example, we will use the
POST method from one origin and disallow it on another one:

```
<ExposeHeader>x-amz-request-id</ExposeHeader>
```
```
Access-Control-Request-Headers: Content-Type
```
```
<!-- Sample policy -->
<CORSConfiguration>
<CORSRule>
...
<AllowedHeader>Authorization</AllowedHeader>
</CORSRule>
</CORSConfiguration>
```
```
<AllowedHeader>x-amz-*</AllowedHeader>
```
```
<AllowedHeader>*</AllowedHeader>
```
```
<AllowedOrigin>*</AllowedOrigin>
```

Remember that if you are hitting a CloudFront distribution instead of your S3 bucket, you should
always make sure to enable the needed methods (GET, POST ..etc.) and whitelist the needed
headers.

To do this, go to your CloudFront distribution Behaviour.

Another way of deciding how your S3 bucket will behave is by using the "Bucket Policy" feature.
We are going to see this later.

## Managing Bucket Policies

```
<CORSConfiguration>
<CORSRule>
<AllowedOrigin>http://www.mywebsite1.com</AllowedOrigin>
<AllowedMethod>GET</AllowedMethod>
<AllowedMethod>POST</AllowedMethod>
<AllowedHeader>*</AllowedHeader>
</CORSRule>
<CORSRule>
<AllowedOrigin>http://www.mywebsite2.com</AllowedOrigin>
<AllowedMethod>GET</AllowedMethod>
<AllowedHeader>*</AllowedHeader>
</CORSRule>
</CORSConfiguration>
```
```
<!-- Sample policy -->
<CORSConfiguration>
<CORSRule>
<AllowedOrigin>http://mywebsite.com</AllowedOrigin>
<AllowedMethod>GET</AllowedMethod>
<AllowedMethod>POST</AllowedMethod>
<AllowedMethod>DELETE</AllowedMethod>
<MaxAgeSeconds>3000</MaxAgeSeconds>
<AllowedHeader>Authorization</AllowedHeader>
</CORSRule>
</CORSConfiguration>
```

We use the bucket policy to grant other AWS accounts, or IAM users access permissions for the
bucket and the objects stored in it. Go to your AWS S3 console and click on "Bucket Policy:

If we are going to change the policy of the previously created bucket "s3static.practicalaws.com",
then this is the ARN (Amazon Resource Name) of this bucket:

### Use Case

Say your work in a startup that develops an Android application internally used by another
company (let’s call it Theta Corp). Your Android developers use S3 to store the new versions of
the Android application, and your client downloads the new Android application (APK file).

At midnight, the Android application will list the S3 bucket and check if there is a newer version of
the application, if it's the case, the mobile application has the right to download and install the
latest version.

Theta Corp wanted to keep the application private, that's why the choice of using a public app
store is not a good one, and that's why S3 is used as an app store for this application. Theta Corp
used the application for its agents and wanted a maximum of privacy and security. Before your
Android application could download the latest version, it should instantiate a new S3 client:

The first thing that may come to your mind is that the Android client should have the right to list
and download the bucket and its objects, but never delete or upload files. This is why we are
going to use a policy for the bucket.

```
arn:aws:s3:::s3static.practicalaws.com
```
```
// Android code
AmazonS3Client s3Client = new AmazonS3Client(new
BasicAWSCredentials(ACCESS_KEY_ID, SECRET_KEY));
```

The first step is creating a user, you can use the current user if you are working on a small
project, but if you need more organization and user separation, it is better to create a new user.

Go to your IAM console and create a new user, then give it the right permissions:

Note: We do not need an AWS Management Console access. Only Programmatic access is
needed. This user needs an S3 read-only access.

Click on next then “Create User”. After creating the user, AWS IAM will generate:

```
Access key ID: "AKIAIBDN4L4S2CV4RBPA"
Secret access key: "sL+FIT4rSaYdq+JfSpq+hz/VuqX0Dnp6sDYSmQEW"
```
Download the .csv file and keep it in a safe place. Your Android developers will use these
generated keys.

Go back to your list of users and click on "android_user".


The blurred text is called AWS Principal. Let’s consider this string as our Principal
"998335703863". The user ARN becomes: arn:aws:iam::998335703863:user/android_user.
Now we have these elements:

```
The Principal: 998335703863
The S3 bucket ARN: arn:aws:s3:::s3static.practicalaws.com
```
We can create a bucket policy:

The ID element specifies an optional identifier for the policy we created.

The version element specifies the language syntax rules that are to be used to process our
bucket policy. Only two values are possible:

```
2012-10-17: This is the current version of the policy language. Recommended.
2008-10-17: This was an earlier version of the policy language. Not recommended.
```
The Statement element is the main element in a bucket policy, and that's why it is required. The
Sid (statement ID) is optional but must be unique within a JSON policy. You can assign a Sid value
to each statement in a statement array. Action should contain the list of actions allowed.

We are going to grant our Android user all of the actions that starts with "Get":

```
s3:GetBucketLocation
```
##### {

```
"Id": "Policy1518491345636",
"Version": "2012-10-17",
"Statement": [
{
"Sid": "Stmt1518491343637",
"Action": [
"s3:Get*"
],
"Effect": "Allow",
"Resource": [
"arn:aws:s3:::s3static.practicalaws.com",
"arn:aws:s3:::s3static.practicalaws.com/*"
],
"Principal": {
"AWS": [
"998335703863"
]
}
}
]
}
```

```
s3:GetBucketCORS
s3:GetIpConfiguration
s3:GetAnalyticsConfiguration
s3:GetObjectVersionForReplication
s3:GetBucketAcl
s3:GetBucketPolicy
s3:GetObject
s3:GetLifecycleConfiguration
s3:GetObjectTorrent
s3:GetBucketVersioning
s3:GetReplicationConfiguration
s3:GetBucketNotification
s3:GetBucketLogging
s3:GetInventoryConfiguration
s3:GetBucketTagging
s3:GetObjectVersionTorrent
s3:GetObjectVersion
s3:GetMetricsConfiguration
s3:GetBucketRequestPayment
s3:GetObjectAcl
s3:GetBucketWebsite
s3:GetObjectTagging
s3:GetAccelerateConfiguration
s3:GetObjectVersionTagging
s3:GetObjectVersionAcl
```
The effect can allow or disallow the setup actions to a resource.

In our case, we are allowing the actions "s3:Get*" to:

```
The bucket: arn:aws:s3:::s3static.practicalaws.com
All of the objects inside this bucket: arn:aws:s3:::s3static.practicalaws.com/*
```
Principal is used to specifying the user (IAM user, federated user, or assumed-role user).

We are using an IAM user in our case.

In most cases, you don't need to write all of the JSON policy by yourself; you can generate it using
the policy generator.

These are the steps to generate the policy automatically:

```
Choose "S3 Bcuket Policy" as the "Type of Policy"
Click on Allow
Add your Principal
Select "Amazon S3" in "AWS Service"
Select all of the actions that starts with "Get"
Add
arn:aws:s3:::s3static.practicalaws.com,arn:aws:s3:::s3static.practicalaws.com/
* to the list of ARNs.
Click on "Add Statement"
Click on "Generate"
```
You will have this generated JSON:

##### {


The generated policy is the same as the one I used:

```
"Id": "Policy1518492495045",
"Version": "2012-10-17",
"Statement": [
{
"Sid": "Stmt1518491343637",
"Action": [
"s3:GetBucketLocation",
"s3:GetBucketCORS",
"s3:GetIpConfiguration",
"s3:GetAnalyticsConfiguration",
"s3:GetObjectVersionForReplication",
"s3:GetBucketAcl",
"s3:GetBucketPolicy",
"s3:GetObject",
"s3:GetLifecycleConfiguration",
"s3:GetObjectTorrent",
"s3:GetBucketVersioning",
"s3:GetReplicationConfiguration",
"s3:GetBucketNotification",
"s3:GetBucketLogging",
"s3:GetInventoryConfiguration",
"s3:GetBucketTagging",
"s3:GetObjectVersionTorrent",
"s3:GetObjectVersion",
"s3:GetMetricsConfiguration",
"s3:GetBucketRequestPayment",
"s3:GetObjectAcl",
"s3:GetBucketWebsite",
"s3:GetObjectTagging",
"s3:GetAccelerateConfiguration",
"s3:GetObjectVersionTagging",
"s3:GetObjectVersionAcl"
],
"Effect": "Allow",
"Resource": [
"arn:aws:s3:::s3static.practicalaws.com",
"arn:aws:s3:::s3static.practicalaws.com/*"
],
"Principal": {
"AWS": [
"998335703863"
] } } ] } {
```
```
"Id": "Policy1518492495045",
"Version": "2012-10-17",
"Statement": [
{
"Sid": "Stmt1518491343637",
"Action": [
"s3:Get*"
```

I replaced:

by:

Now copy the generated policy and paste it in the "Bucket policy editor".

##### ],

```
"Effect": "Allow",
"Resource": [
"arn:aws:s3:::s3static.practicalaws.com",
"arn:aws:s3:::s3static.practicalaws.com/*"
],
"Principal": {
"AWS": [
"998335703863"
]
}
}
]
}
```
```
"s3:GetBucketLocation",
"s3:GetBucketCORS",
"s3:GetIpConfiguration",
"s3:GetAnalyticsConfiguration",
"s3:GetObjectVersionForReplication",
"s3:GetBucketAcl",
"s3:GetBucketPolicy",
"s3:GetObject",
"s3:GetLifecycleConfiguration",
"s3:GetObjectTorrent",
"s3:GetBucketVersioning",
"s3:GetReplicationConfiguration",
"s3:GetBucketNotification",
"s3:GetBucketLogging",
"s3:GetInventoryConfiguration",
"s3:GetBucketTagging",
"s3:GetObjectVersionTorrent",
"s3:GetObjectVersion",
"s3:GetMetricsConfiguration",
"s3:GetBucketRequestPayment",
"s3:GetObjectAcl",
"s3:GetBucketWebsite",
"s3:GetObjectTagging",
"s3:GetAccelerateConfiguration",
"s3:GetObjectVersionTagging",
"s3:GetObjectVersionAcl"
```
```
"s3:Get*"
```

Make sure to change the above values by your values (Principal, name of the bucket ..etc.). Now,
when your Android developer wants to access the latest version of the APK file in the S3 bucket,
she/he will be able to perform all of the actions that starts with "Get". Of course, the developer
should use the good keys:

## S3 Data Encryption

Amazon S3 uses TLS/SSL by default, but for some reason, Theta Corp also wanted to encrypt data
at rest; that's why we are going to use data encryption.

AWS S3 supports 2 types of encryption:

```
Server-side: Amazon S3 will encrypt a bucket object before saving it to the disk. An
encrypted object is decrypted when downloaded.
Client-side: You will manage the encryption process by yourself and use encryption keys to
encrypt your objects before uploading them to S3.
```
The server-side encryption can be done using the AWS S3 dashboard.

```
AmazonS3Client s3Client = new AmazonS3Client(new
BasicAWSCredentials("AKIAIBDN4L4S2CV4RBPA",
"sL+FIT4rSaYdq+JfSpq+hz/VuqX0Dnp6sDYSmQEW"));
```

There are two types of server-side encryption that S3 provides:

```
Using AWS KMS-Managed Keys ( SSE-KMS ): Customer-provided encryption keys
Using S3-Managed Encryption Keys ( SSE-S3 ): Amazon provides and manage the keys
```
SSE-S3 is simpler to use than SSE-KMS since it is managed by AWS. SSE-S3 uses 256-bit Advanced
Encryption Standard (AES-256).

Server-side encryption encrypts only the object data.

Let’s create a new bucket:


Upload a file:

Let’s copy this file "app.apk" to the bucket:

You can also upload your file using the AWS S3 dashboard.

We are now going to encrypt the uploaded file. On your AWS S3 dashboard, click on the file, click
on "properties" then "Encryptio". Choose the AES-256 encryption (SSE-S3):

Click on "Save". Our APK file is now encrypted.

Your Android developer will upload the APK files to the bucket, but she/he may forget that any
uploaded APK should be encrypted, she/he may upload a file without encryption! In this case, we
are going to use the bucket policy to deny unencrypted files upload.

This will deny any request that does not include the "x-amz-server-side-encryption" header.

The Android developer may use another type of encryption like the AWS-KMS encryption, but we
only want to use the AES-256 encryption:

```
aws s3 mb s3://enc-practicalaws
```
```
aws s3 cp my_app.apk s3://enc-practicalaws
```
```
aws s3 cp app.apk s3://enc-practicalaws
```
##### {

```
"Sid": "DenyUnEncryptedObjectUploads",
"Effect": "Deny",
"Principal": "*",
"Action": "s3:PutObject",
"Resource": "arn:aws:s3:::enc-practicalaws/*",
"Condition": {
"Null": {
"s3:x-amz-server-side-encryption": "true"
}
}
}
```

The bucket policy will include both statements, and it will look like this:

Now, in order to test, upload a file with the wrong encryption or without encryption at all. You
should have an error at this step.

##### {

```
"Sid": "DenyUnEncryptedObjectUploads",
"Effect": "Deny",
"Principal": "*",
"Action": "s3:PutObject",
"Resource": "arn:aws:s3:::enc-practicalaws/*",
"Condition": {
"Null": {
"s3:x-amz-server-side-encryption": "true"
}
}
}
```
##### {

```
"Version": "2012-10-17",
"Id": "PutObjPolicy",
"Statement": [
{
"Sid": "DenyIncorrectEncryptionHeader",
"Effect": "Deny",
"Principal": "*",
"Action": "s3:PutObject",
"Resource": "arn:aws:s3:::enc-practicalaws/*",
"Condition": {
"StringNotEquals": {
"s3:x-amz-server-side-encryption": "AES256"
}
}
},
{
"Sid": "DenyUnEncryptedObjectUploads",
"Effect": "Deny",
"Principal": "*",
"Action": "s3:PutObject",
"Resource": "arn:aws:s3:::enc-practicalaws/*",
"Condition": {
"Null": {
"s3:x-amz-server-side-encryption": "true"
}
}
}
]
}
```

Note: Amazon S3 encrypts each object with a unique key. It encrypts the key itself with a master
key that it regularly rotates.

If you would like to get more details about uploading or downloading an encrypted file, you can
get more logs by adding--debug to the AWS S3 CLI command:

Example:

```
aws s3 cp s3://enc-practicalaws/app.apk /tmp/ --debug
```
##### 2018-02-13 15:44:48,

```
760 - MainThread - botocore.endpoint - DEBUG - Sending http request:
<PreparedRequest[
HEAD
]>
2018-02-13 15:44:49,
522 - MainThread - botocore.parsers - DEBUG - Response headers:{
'ETag':'"f24198ca0f9c4d75a9786b4308ea8470"',
'Server':'AmazonS3',
'x-amz-server-side-encryption':'AES256',
'Date':'Tue, 13 Feb 2018 14:44:50 GMT',
'Accept-Ranges':'bytes',
'x-amz-request-id':'DECC475B893E025E',
```

## S3 Logging

Theta Corp applications stored in S3 are secure, and everything seems to work fine, but in 3
months after starting the project, you realized that the costs associated with S3 are increasing.
While the number of users is the same, and the frequency of deployment did not change, the
bills of S3 services were increasing.

One of the things your team was thinking about was that some Android devices that are probably
stuck in a loop of infinite downloads .. this scenario could happen, and it will consume your
bandwidth and, of course, increase bills.

One of the first things to do in order to troubleshoot your system is activating logging.

S3 logs are helpful in security and access audits. To activate S3 logs, you should create a new
bucket where you will store the logs of the "enc-practicalaws" bucket.

Make sure that the source bucket (the bucket to log) and the target bucket (the bucket that stores
logs) are in the same region. Let's name the target bucket "enc-practicalaws-logs":

Go to the source bucket "enc-practicalaws" and click on it, then click on properties and choose
"Server access logging". Activate the option "Enable logging", then choose the target bucket "enc-
practicalaws-logs".

If you would like to add a prefix to your logs to make them easily identified, you can do it by filling
the prefix in the "Target prefix" text input. You can keep it empty if you don't need this.

```
'x-amz-id-
2':'bySlyKxcghVN+rLG7yHMnNjOjZF/ypCOao7DJaZuc2o74le+8O2CT7d+CVq1BaygM1U3n94QwPM=
',
'Content-Type':'application/octet-stream',
'Content-Length':'207663',
'Last-Modified':'Tue, 13 Feb 2018 14:43:05 GMT'
}
```

Click on save.


# Lesson 15 - Host a Static Website Using S3 and CloudFront (video)

Note: Download the videos archive here.

## Notes about the Video

I want to draw your attention to the fact that Amazon Certificate Manager (ACM) that we use to
generate an SSL certificate is not only available in N. Virignia region, but the service was made
available by AWS in many other regions.

# Lesson 17 - Serverless Computing - AWS Lambda (Video)

Note: Download the videos archive here.

## Additional Notes

## Lambda Limits

AWS Lambda has limits, and in this section, we will list all of them (These limits may be subject to
change in the future by AWS)

#### Concurrent executions:

##### 1,000

#### Function and layer storage:

##### 75 GB

#### Elastic network interfaces per VPC:

##### 250

#### Function memory allocation:

128 MB to 3,008 MB, in 64 MB increments.

#### Function timeout:

900 seconds (15 minutes)

#### Function environment variables:

##### 4 KB

#### Function resource-based policy:

##### 20 KB

#### Function layers:

5 layers

#### Function burst concurrency:

500 - 3000 (varies per region)

#### Invocation frequency (requests per second):

```
10 x concurrent executions limit (synchronous – all sources)
10 x concurrent executions limit (asynchronous – non-AWS sources)
```

```
Unlimited (asynchronous – AWS service sources)
```
#### Invocation payload (request and response)

```
6 MB (synchronous)
256 KB (asynchronous)
```
#### Deployment package size:

```
50 MB (zipped, for direct upload)
250 MB (unzipped, including layers)
3 MB (console editor)
```
#### Test events (console editor):

##### 10

#### /tmp directory storage:

##### 512 MB

#### File descriptors:

##### 1,024

#### Execution processes/threads:

##### 1,024


# Lesson 18 - AWS Lambda - Hello World

## Creating Our First AWS Lambda Function

First, log in to your AWS console and open the Lambda console. Click on "Create Function", give it
a name and choose "Create a custom role".

Keep the role with the default policy document since our "Hello World" function will not need any
special permission.

Click on "Allow" and make sure to choose "lambda_basic_execution" as a role. Now let's add this
Node.js "Hello World" function:

Click on "Test":

```
exports.handler = (event, context, callback) => {
// Succeed with the string "Hello world!"
callback(null, 'Hello world!');
};
```

Our test event, once executed, will call the Lambda "Hello World" function. Click on "Create"
followed by "Test", and you will notice the execution results:

This is a basic function.

Using Lambda, you can use another language like Python. In order to do this, let's first remove
the first function, so go back to your dashboard and click on "Action" then "Delete":


Now, create a new function and choose Python 3.6 or Python 3.8 as a runtime.

The Python Hello World function is the following:

This function will return the string "Hello from Lambda" and exits.

Note: For Python, the function handler should be configured correctly in this form
file_name.function_name. The file name should not also contain the extension of the file.

In my example, my file is called lambda_function.py and my function is defined as
lambda_handler. The handler will be:

## AWS Lambda & CloudWatch

Amazon CloudWatch is the cloud monitoring service for AWS resources. It collects, tracks, and
reacts to metrics as well as logs.

Our function logs could be viewed directly from the AWS Lambda dashboard.

By default, this service collects a Lambda function logs automatically and stores it in a folder that
you can find in the dashboard of AWS CloudWatch.

```
def lambda_handler(event, context):
return 'Hello from Lambda'
```
```
lambda_function.lambda_handler
```

## Triggering The Execution Of A Lambda Function

Let's now see how we can trigger our function using one of the AWS services. For this example, I'll
use S3 as a trigger, and I'd need to execute my Hello World function when a file is uploaded to a
bucket. In the function, we are going to read the content of the S3 file (we will upload text file).

Click on S3 from "Designer" and add S3 as a trigger.

Let's now create a bucket:

We also need to add some new rights to the Lambda function. Click on the execution role:

Then choose "Edit Policy" then click on "Add additional permissions". Add the permissions you
need to **list** , **read** , and **write** from/in the bucket, then add the bucket ARN.

```
aws s3 mb s3://test-6507bed907 --region eu-west-3
```
##### ---

```
make_bucket: test-6507bed907
```

Click on "Any" on "object" to guarantee access to all objects in the bucket.

Now create a file and upload it to the S3 bucket:

The log should now be visible in CloudWatch. Visit CloudWatch dashboard and search for the
function name, then choose the right Log Group and you will be able to tail log and see that the
function prints the content of the file to the log file:

Create an SNS topic:

```
echo "This is a test content" > my_file.txt
aws s3 cp my_file.txt s3://test-6507bed907/
```

Configure this SNS with a new Topic and add "Email" as "Protocol", then your email in the email
field.


Go back to the Topic list, copy the ARN of this SNS ressource.

Use the ARN in the Lambda dashboard to configure a target.

Now to test if you receive an email on success, you should re-upload a file and check your email.
You should receive something very similar to:

##### {

```
"version":"1.0",
"requestContext":{
"requestId":"xxxxx",
"functionArn":"arn:aws:lambda:eu-west-
3:xxxxx:function:HelloWorld:$LATEST",
"condition":"Success",
"approximateInvokeCount":1
},
"requestPayload":{
"Records":[
{
[...]
"awsRegion":"eu-west-3",
"eventName":"ObjectCreated:Put",
"userIdentity":{
```

It is also possible to use DynamoDB or AWS Kinesis to stream data to the SNS topic, or use an
SQS queue as a target.

## AWS Lambda & Environment Variables

When developing an application or an AWS Lambda function, one of the best practices in keeping
your environment variables outside of your code. An application configuration may vary between
deploys:

```
Staging
Production
Testing
Developer's machine
..etc
```
e.g.: You database credentials may vary from one environment to another

To avoid the problems that these configurations may cause, You'll need to store your
configurations in the environment.

In order to test this feature, add some environment variables:

```
"principalId":"AWS:AIDAJCLK5KQQMZKMCCQ7G"
},
"requestParameters":{
"sourceIPAddress":"176.185.143.227"
},
"responseElements":{
"x-amz-request-id":"xxxxx",
"x-amz-id-2":"xxxxx"
},
"s3":{
"s3SchemaVersion":"1.0",
"configurationId":"xxxxx",
"bucket":{
"name":"test-6507bed907",
"ownerIdentity":{
"principalId":"xxxxx"
},
"arn":"arn:aws:s3:::test-6507bed907"
},
"object":{
"key":"my_file.txt",
"size":23,
"eTag":"xxxxx",
"sequencer":"xxxxx"
}
}
}
]
},
"responseContext":{
"statusCode":200,
"executedVersion":"$LATEST"
},
"responsePayload":null
}
```

Go back to the code editor and add this function:

As you can see in the code above, I used my code to get the environment variables:

The execution results will be similar to this one:

## AWS Lambda & VPC

```
import os
```
```
def lambda_handler(event, context):
print ( "my 1st variable is: " + os.environ["my_var_a"])
print ( "my 2nd variable is: " + os.environ["my_var_b"])
print ( "my 3rd variable is: " + os.environ["my_var_c"])
return ""
```
```
print ( "my 1st variable is: " + os.environ["my_var_a"])
print ( "my 2nd variable is: " + os.environ["my_var_b"])
print ( "my 3rd variable is: " + os.environ["my_var_c"])
```
```
Response:
""
```
```
Request ID:
"52e46e12-fa27-11e7-8d74-d76017424db1"
```
```
Function Logs:
START RequestId: 52e46e12-fa27-11e7-8d74-d76017424db1 Version: $LATEST
my 1st variable is: my_value_a
my 2nd variable is: my_value_b
my 3rd variable is: my_value_c
END RequestId: 52e46e12-fa27-11e7-8d74-d76017424db1
REPORT RequestId: 52e46e12-fa27-11e7-8d74-d76017424db1 Duration: 0.35 ms
Billed Duration: 100 ms Memory Size: 128 MB Max Memory Used: 21 MB
```

By default, a Lambda function runs inside the default VPC. It's optional to configure the function
to run inside a custom VPC, and the goal is accessing the same VPC resources, e.g., RDS database
instances, Amazon ElastiCache clusters ..etc

Note: If your function requires external access to the Internet, you must ensure that the security
group allows outbound connections and that your VPC has a NAT gateway.

Ensure your role has appropriate permissions to configure VPC.

To enable our Lambda function to access resources inside our private VPC, we must provide
additional VPC-specific configuration information that includes VPC subnet IDs and security
group IDs.

Lambda uses this information to set up ENIs (elastic network interfaces) that enable our function
to connect securely to other resources within this VPC. The next paragraph explains how to
create a role.

## AWS Lambda and Credentials

Sometimes, you need to call other AWS services using your Lambda code, so you will use the AWS
SDK; whether you are using Python, Node.js, or another programming language, you will need to
instantiate a new AWS connection to a given service.

Example, with Boto3 (Python):

If this code is used in a Lambda function (or any other AWS service), we do not need to add the
credentials.

AWS services do not need your credentials; the only thing you need to take care of is the different
rights that your AWS Lambda function has (Execution Roles).

## AWS Lambda & IAM Roles (Execution Roles)

```
import boto3
client = boto3.client(
's3',
aws_access_key_id=ACCESS_KEY,
aws_secret_access_key=SECRET_KEY,
aws_session_token=SESSION_TOKEN,
)
```
```
# Or via the Session
session = boto3.Session(
aws_access_key_id=ACCESS_KEY,
aws_secret_access_key=SECRET_KEY,
aws_session_token=SESSION_TOKEN,
)
```
```
import boto3
client = boto3.client('s3')
```
```
# Or via the Session
session = boto3.Session()
```

To goal of creating a role is having a separate profile to which we can add 1 or more permissions.
In this example, we are going to create a role and add the permission
AWSLambdaVPCAccessExecutionRole that will allow a Lambda function to perform EC2 actions in
order to create ENIs and access VPC resources as well as CloudWatch Logs actions to write logs.

It's described by AWS in the following Policy Document:

To create an IAM role (execution role), sign in to the AWS Management Console and open the IAM
console.

##### {

```
"Version": "2012-10-17",
"Statement": [
{
"Effect": "Allow",
"Action": [
"logs:CreateLogGroup",
"logs:CreateLogStream",
"logs:PutLogEvents",
"ec2:CreateNetworkInterface",
"ec2:DescribeNetworkInterfaces",
"ec2:DeleteNetworkInterface"
],
"Resource": "*"
}
]
}
```

Click "Next", search for "AWSLambdaVPCAccessExecutionRole", activate it then click on "Next:
Review". Give it a name like "lambda_basic_execution_with_access_to_vpc", a description and
create the role.

Now you can go back to your function dashboard and select the new role.

## AWS Lambda & The Code Entry Type

To add our function code to AWS Lambda, we have 3 possibilities:

```
Using the cloud IDE Cloud9
using a ZIP file
Using S3
```
### Using AWS Cloud IDE - Cloud9

This is what we have been doing in the first steps of this lesson.

### Using a .ZIP Archive

We created our first AWS Lambda function online using AWS Cloud9, but it's possible to upload
the code using a ZIP file.

When you code using your machine, creating a deployment package is useful. If you had included
third party dependencies or any static or media file (photos, videos ..etc.), make sure to include
them in your.ZIP archive.

e.g.: If you are using Node.js and you are going to need the node module "x11", you need to
install first:

After installing this module, you will have these files in your machine:

These files should be included in the .ZIP package.

Let's take the example of a Python application that uses the library "requests".

```
npm install x11
```
```
node_modules/x11/lib/x11.js
node_modules/x11/package.json
```

Create a folder:

Then a virtual environment:

and activate it:

We will create a function that returns the status of GET requests on our website
s3static.practicalaws.com.

We should then install "requests":

The dependencies that we want to include in our archive file are located here:

Note that you should add the content of this directory and not the directory itself.

This is the structure of our application folder:

```
mkdir hello_world
```
```
virtualenv -p python3 hello_world
```
. bin/activate

```
import requests
```
```
def lambda_handler(event, context):
return requests.get('https://s3static.practicalaws.com').status_code
```
```
pip install requests
```
```
$VIRTUAL_ENV/lib/python3.6/site-packages
```
```
├── **app.py**
├── bin
│   ├── activate
│   ├── activate.csh
│   ├── activate.fish
│   ├── activate_this.py
│   ├── chardetect
│   ├── easy_install
│   ├── easy_install-3.5
│   ├── pip
│   ├── pip3
│   ├── pip3.5
│   ├── python -> python3
│   ├── python3
│   ├── python3.5 -> python3
│   ├── python-config
│   └── wheel
├── include
│   └── python3.5m -> /usr/include/python3.5m
├── **lib**
│   └── **python3.5**
```

│   ├── abc.py -> /usr/lib/python3.5/abc.py
│   ├── base64.py -> /usr/lib/python3.5/base64.py
│   ├── bisect.py -> /usr/lib/python3.5/bisect.py
│   ├── _bootlocale.py -> /usr/lib/python3.5/_bootlocale.py
│   ├── codecs.py -> /usr/lib/python3.5/codecs.py
│   ├── collections -> /usr/lib/python3.5/collections
│   ├── _collections_abc.py -> /usr/lib/python3.5/_collections_abc.py
│   ├── config-3.5m-i386-linux-gnu -> /usr/lib/python3.5/config-3.5m-i386-
linux-gnu
│   ├── copy.py -> /usr/lib/python3.5/copy.py
│   ├── copyreg.py -> /usr/lib/python3.5/copyreg.py
│   ├── distutils
│   ├── _dummy_thread.py -> /usr/lib/python3.5/_dummy_thread.py
│   ├── encodings -> /usr/lib/python3.5/encodings
│   ├── fnmatch.py -> /usr/lib/python3.5/fnmatch.py
│   ├── functools.py -> /usr/lib/python3.5/functools.py
│   ├── __future__.py -> /usr/lib/python3.5/__future__.py
│   ├── genericpath.py -> /usr/lib/python3.5/genericpath.py
│   ├── hashlib.py -> /usr/lib/python3.5/hashlib.py
│   ├── heapq.py -> /usr/lib/python3.5/heapq.py
│   ├── hmac.py -> /usr/lib/python3.5/hmac.py
│   ├── importlib -> /usr/lib/python3.5/importlib
│   ├── imp.py -> /usr/lib/python3.5/imp.py
│   ├── io.py -> /usr/lib/python3.5/io.py
│   ├── keyword.py -> /usr/lib/python3.5/keyword.py
│   ├── lib-dynload -> /usr/lib/python3.5/lib-dynload
│   ├── linecache.py -> /usr/lib/python3.5/linecache.py
│   ├── locale.py -> /usr/lib/python3.5/locale.py
│   ├── no-global-site-packages.txt
│   ├── ntpath.py -> /usr/lib/python3.5/ntpath.py
│   ├── operator.py -> /usr/lib/python3.5/operator.py
│   ├── orig-prefix.txt
│   ├── os.py -> /usr/lib/python3.5/os.py
│   ├── plat-i386-linux-gnu -> /usr/lib/python3.5/plat-i386-linux-gnu
│   ├── posixpath.py -> /usr/lib/python3.5/posixpath.py
│   ├── __pycache__
│   ├── random.py -> /usr/lib/python3.5/random.py
│   ├── reprlib.py -> /usr/lib/python3.5/reprlib.py
│   ├── re.py -> /usr/lib/python3.5/re.py
│   ├── rlcompleter.py -> /usr/lib/python3.5/rlcompleter.py
│   ├── shutil.py -> /usr/lib/python3.5/shutil.py
│   ├── **site-packages**
│   ├── site.py
│   ├── sre_compile.py -> /usr/lib/python3.5/sre_compile.py
│   ├── sre_constants.py -> /usr/lib/python3.5/sre_constants.py
│   ├── sre_parse.py -> /usr/lib/python3.5/sre_parse.py
│   ├── stat.py -> /usr/lib/python3.5/stat.py
│   ├── struct.py -> /usr/lib/python3.5/struct.py
│   ├── tarfile.py -> /usr/lib/python3.5/tarfile.py
│   ├── tempfile.py -> /usr/lib/python3.5/tempfile.py
│   ├── tokenize.py -> /usr/lib/python3.5/tokenize.py
│   ├── token.py -> /usr/lib/python3.5/token.py
│   ├── types.py -> /usr/lib/python3.5/types.py
│   ├── warnings.py -> /usr/lib/python3.5/warnings.py
│   ├── weakref.py -> /usr/lib/python3.5/weakref.py
│   └── _weakrefset.py -> /usr/lib/python3.5/_weakrefset.py
├── pip-selfcheck.json
└── share


In the .ZIP file, we only need the app.py file as well as the content of the folder
lib/python3.5/site-packages/.

This is the content of my archive:

```
└── python-wheels
├── CacheControl-0.11.5-py2.py3-none-any.whl
├── chardet-2.3.0-py2.py3-none-any.whl
├── colorama-0.3.7-py2.py3-none-any.whl
├── distlib-0.2.2-py2.py3-none-any.whl
├── html5lib-0.999-py2.py3-none-any.whl
├── ipaddress-0.0.0-py2.py3-none-any.whl
├── lockfile-0.12.2-py2.py3-none-any.whl
├── packaging-16.6-py2.py3-none-any.whl
├── pip-8.1.1-py2.py3-none-any.whl
├── pkg_resources-0.0.0-py2.py3-none-any.whl
├── progress-1.2-py2.py3-none-any.whl
├── pyparsing-2.0.3-py2.py3-none-any.whl
├── requests-2.9.1-py2.py3-none-any.whl
├── retrying-1.3.3-py2.py3-none-any.whl
├── setuptools-20.7.0-py2.py3-none-any.whl
├── six-1.10.0-py2.py3-none-any.whl
├── urllib3-1.13.1-py2.py3-none-any.whl
└── wheel-0.29.0-py2.py3-none-any.whl
```

Note: The file app.py as well as the content of the folder lib/python3.5/site-packages/ are
included and have the same file-hierarchy level.

Now, on your Lambda dashboard, change "Code entry type" and set it to "Upload a .ZIP file".

### Using S3

If you already have the .ZIP file, you can simply use S3 by uploading the same package to an S3
bucket and configure your AWS Lambda to use S3 as a deployment type.

Don't forget to change eu-west- 1 by your region.

Open your terminal and update the Lambda function using AWS CLI:

Change these values by your own values:

```
--function-name
--region
--s3-bucket
--s3-key
```
## Lambda Versions

Lambda versions could be defined as the different snapshots that you can make of your project.
A version has a number (id).

You can publish a version from the dashboard by clicking on "Publish new version":

```
Publishing a new version will save a "snapshot" of the code and configuration of the
$LATEST version. You will be unable to edit the new version's code after publishing it.
```
After publishing a version, click on "Qualifiers" and you'll notice that your new version was added.

```
aws s3 mb s3://practical-aws-lambda-code
aws s3 cp code.zip s3://practical-aws-lambda-code --region eu-west-1
```
```
aws lambda update-function-code --function-name myFunction --region eu-west-1 --
s3-bucket practical-aws-lambda-code --s3-key code.zip
```

You may also configure a test before publishing a version.

Say, you made some modifications to your function, and you want to publish the second version
from the CLI.

First of all, if you keep your code without any modifications, Lambda will not increment the actual
version. Let's make any modification on our function, like making the GET request use HTTP
instead of https.

Archive your code and dependencies inside a ZIP package, and upload it to S3:

Update your code:

Publish a new version (2):

## Lambda Aliases

An alias is a pointer to a specific version. Each alias has a unique ARN (Amazon Resource Name).

This could be useful for many scenarios, like pointing a PROD alias to the latest version of your
code.

The alias called "Unqualified" is the default alias, and it always points to the default version
$LATEST.

```
import requests
```
```
def lambda_handler(event, context):
return requests.get('http://s3static.practicalaws.com').status_code
```
```
aws s3 cp code.zip s3://practical-aws-lambda-code --region eu-west-1
```
```
aws lambda update-function-code --function-name myFunction --region eu-west-1 --
s3-bucket practical-aws-lambda-code --s3-key code.zip
```
```
aws lambda publish-version --function-name myFunction --description "this is my
v2" --region eu-west-1 --output=text --query 'Version'
```

Aliases could also be used to attach triggers. In our case, any modification on S3 is a trigger that
invokes the function. If you go to our bucket s3static.practicalaws.com and click on
"Properties" then "Events", you will notice that the event source mapping information is stored in
the bucket notification configuration. The Lambda function ARN is stored in this configuration:

In this case, when you publish a new version of your Lambda function, you need to update the
notification configuration so that Amazon S3 invokes the correct version. The time between
having the new version running in production and the update you will make to the S3 trigger
could result in downtime, even if it could be for a small period of time, but it is still downtime.


This is when aliases are useful, you can attach the trigger to an alias instead of a version. You
need to update the PROD alias to point to the latest stable version.

You can create an alias an attach it to a version using a similar command to this one:

You should, of course, adapt your region, function name, description, function version, and alias
name.

```
aws lambda create-alias --region eu-west-1 --function-name myFunction --
description "My alias" --function-version "\$LATEST" --name PROD
```