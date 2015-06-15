# AWS Code Deploy Executer

This script deploys applications with the [AWS Code Deploy](http://docs.aws.amazon.com/codedeploy/latest/userguide/welcome.html) service. This script has been adapted to be easily portable and configurable via environment variables such that it can be incorporated within CI services that do not natively include support for Code Deploy. Additionally, this script includes additional functionality described below that is typically not included in out-of-box Code Deploy CI systems. For more information, refer to the [AWS Code Deploy](http://docs.aws.amazon.com/codedeploy/latest/userguide/welcome.html) documentation or
the [AWS CLI API](http://docs.aws.amazon.com/cli/latest/reference/deploy/index.html).

Features:
 * Compression of source contents
 * Ability to limit the number of stored revisions by a key prefix to help reduce S3 total file size
 * Server side encryption for revisions
 * Full diagnostic output from failed instances

Operating Systems Supported:
 * Debian 7
 * Ubuntu 12.04
 * Ubuntu 14.04


## How to Include In Your Project

#### 1. Via Composer (For PHP Projects)

#### 2. Git Submodule

#### 3. Git Subtree


## Variables

```
AWS_CODE_DEPLOY_KEY
AWS_CODE_DEPLOY_SECRET
AWS_CODE_DEPLOY_REGION
AWS_CODE_DEPLOY_APPLICATION_NAME
AWS_CODE_DEPLOY_DEPLOYMENT_CONFIG_NAME
AWS_CODE_DEPLOY_MINIMUM_HEALTHY_HOSTS
AWS_CODE_DEPLOY_DEPLOYMENT_GROUP_NAME
AWS_CODE_DEPLOY_SERVICE_ROLE_ARN
AWS_CODE_DEPLOY_EC2_TAG_FILTERS
AWS_CODE_DEPLOY_AUTO_SCALING_GROUPS
AWS_CODE_DEPLOY_APP_SOURCE
AWS_CODE_DEPLOY_S3_BUCKET
AWS_CODE_DEPLOY_S3_KEY_PREFIX
AWS_CODE_DEPLOY_S3_FILENAME
AWS_CODE_DEPLOY_S3_LIMIT_BUCKET_FILES
AWS_CODE_DEPLOY_S3_SSE
AWS_CODE_DEPLOY_REVISION_DESCRIPTION
AWS_CODE_DEPLOY_DEPLOYMENT_DESCRIPTION
```


## Usage






## AWS Code Deploy Workflow with Detailed Variable Information

#### Step 1: Checking Dependencies

The following executables are installed:
    * python-pip
    * aws

#### Step 2: [Configuring AWS](http://docs.aws.amazon.com/cli/latest/reference/configure/index.html)

This step ensures that configuration parameters for AWS cli are properly set.

Environment Variables:

* `AWS_CODE_DEPLOY_KEY` (optional): AWS Access Key ID. If not already configured in **aws** cli, this is required.
* `AWS_CODE_DEPLOY_SECRET` (optional): AWS Secret Access Key. If not already configured in **aws** cli, this is required.
* `AWS_CODE_DEPLOY_REGION` (optional): Default region name

#### Step 3: [Checking Application](http://docs.aws.amazon.com/cli/latest/reference/deploy/create-application.html)

This step ensures the application exists within Code Deploy. If it does not exist, it will attempt to create the application with the specified name.

Environment Variables:

* `AWS_CODE_DEPLOY_APPLICATION_NAME` (required): Name of the application to deploy

#### Step 4: [Deployment Config](http://docs.aws.amazon.com/cli/latest/reference/deploy/create-deployment-config.html) (optional)

This step ensures the specified deployment configuration exists for the application. Defining a custom configuration is *optional* because you can use the deployment strategy already defined in Code Deploy.

Environment Variables:

* `AWS_CODE_DEPLOY_DEPLOYMENT_CONFIG_NAME` (optional): Deployment config name. By default: _CodeDeployDefault.OneAtATime_. Built-in options:
    * CodeDeployDefault.OneAtATime
    * CodeDeployDefault.AllAtOnce
* `AWS_CODE_DEPLOY_MINIMUM_HEALTHY_HOSTS` (optional): The minimum number of healthy instances during deployment. By default: _type=FLEET_PERCENT,value=75_

#### Step 5: [Deployment Group](http://docs.aws.amazon.com/cli/latest/reference/deploy/create-deployment-group.html)

This step ensures the deployment group exists within the specified application. If it does not exist, the script will attempt to create the group using the name and defined service role ARN.

Environment Variables:

* `AWS_CODE_DEPLOY_DEPLOYMENT_GROUP_NAME` (required): Deployment group name
* `AWS_CODE_DEPLOY_SERVICE_ROLE_ARN` (required): Service role arn giving permissions to use Code Deploy when creating a deployment group
* `AWS_CODE_DEPLOY_EC2_TAG_FILTERS` (optional): EC2 tags to filter on when creating a deployment group
* `AWS_CODE_DEPLOY_AUTO_SCALING_GROUPS` (optional): Auto Scaling groups when creating a deployment group

#### Step 6: Compressing Source

This step compresses the specified source directory as a zip file in preparation for uploading to S3. This is useful for application deployments that use unique file names per revision. This helps limit the transfer to and from S3 during deployment.

Environment Variables:

* `AWS_CODE_DEPLOY_APP_SOURCE` (required): Specifies the root source contents of the application. The `appspec.yml` should exist within this directory. If not specified, the script will use the current working directory.
* `AWS_CODE_DEPLOY_S3_FILENAME` (required): The destination name within S3. Note that this should **not** include any prefix keys as these are defined elsewhere. A recommended good practice would be to use a combination of the CI build number with the git short revision. (e.g. "100#c3a5fea.zip")

#### Step 7: [Pushing to S3](http://docs.aws.amazon.com/cli/latest/reference/deploy/push.html)

This step consists to push the application to S3.

Environment Variables:

* `AWS_CODE_DEPLOY_S3_BUCKET` (required): The name of the S3 bucket to deploy the revision
* `AWS_CODE_DEPLOY_S3_KEY_PREFIX` (optional): A prefix to use for the file key. It's highly recommended to structure a bucket with a prefix per deployment group. This allows to limit stored revisions per deployment group. Note: A leading or trailing slash is not required.  For example:
```
AWS_CODE_DEPLOY_S3_BUCKET="my-bucket-test"
AWS_CODE_DEPLOY_S3_KEY_PREFIX="production-www"
AWS_CODE_DEPLOY_S3_FILENAME="100#c3a5fea.zip"

# The resulting stored file would exist at s3://my-bucket-test/production-www/100#c3a5fea.zip
```

#### Step 8: Limiting Deploy Revisions per Bucket/Key

This step ensures that applications with high revision/commit volume with unique filenames can remove old revisions to help limit the size of the container. Large teams can quickly fill S3 with multiple TBs/day depending on the projects. Since deployments typically don't need to store that many versions backwards, this step will ensure that only N revisions exist, removing oldest revisions upon deploy.

Environment Variables:

* `AWS_CODE_DEPLOY_S3_LIMIT_BUCKET_FILES` (optional): Number of revisions to limit. If 0, unlimited. By default: 0

#### Step 9: [Registering Revision](http://docs.aws.amazon.com/cli/latest/reference/deploy/register-application-revision.html)

This step registers a code deploy revision for the uploaded file to the specified application/deployment group.

Environment Variables:

* `AWS_CODE_DEPLOY_REVISION_DESCRIPTION` (optional): A description that is stored within AWS Code Deploy that stores information about the specific revision. Typically, the revision details would store information specific to the commit/CI/build details.


#### Step 10: [Creating Deployment](http://docs.aws.amazon.com/cli/latest/reference/deploy/create-deployment.html)

This step deploys the application revision using the defined deployment settings across all hosts that match the deployment group.

Environment Variables:

* `AWS_CODE_DEPLOY_DEPLOYMENT_DESCRIPTION` (optional): A description that is stored within AWS Code Deploy that stores information about the specific revision.

#### Monitor Deployment

This step monitors the deployment and logs information about the overall status as well as any failed instance statuses.

Environment Variables:

* `AWS_CODE_DEPLOY_DEPLOYMENT_OVERVIEW` (optional): Boolean that specifies whether to log detailed information about the status of the deployment. By Default: _true_



## Example

The following example deploy an `hello` application on the deployment group `development` after pushed the application on the `apps.mycompany.com` S3 bucket :

```
deploy:
  steps:
    - nhuray/aws-code-deploy:
       key: aws_access_key_id
       secret: aws_access_secret_id
       application-name: hello
       application-version: 1.1.0
       deployment-group-name: development
       service-role-arn: arn:aws:iam::89862646$091:role/CodeDeploy
       ec2-tag-filters: Key=app,Value=hello,Type=KEY_AND_VALUE Key=environment,Value=development,Type=KEY_AND_VALUE
       s3-bucket: apps.mycompany.com
```
