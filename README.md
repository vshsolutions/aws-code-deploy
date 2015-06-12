# AWS Code Deploy Executer

This script deploys applications with the [AWS Code Deploy](http://docs.aws.amazon.com/codedeploy/latest/userguide/welcome.html) service. This script has been adapted to be easily portable and configurable via environment variables such that it can be incorporated within CI services that do not natively include support for Code Deploy. Additionally, this script includes additional functionality described below that is typically not included in out-of-box Code Deploy CI systems. For more information, refer to the [AWS Code Deploy](http://docs.aws.amazon.com/codedeploy/latest/userguide/welcome.html) documentation or
the [AWS CLI API](http://docs.aws.amazon.com/cli/latest/reference/deploy/index.html).

Features:
 * Compression of source contents
 * Ability to limit the number of stored revisions by a key prefix to help reduce S3 total file size
 * Server side encryption for revisions
 * Full diagnostic output from failed instances

Operating Systems Supported:
 * Debian
 * Ubuntu 14.04


## How to Include In Your Proect

#### 1. Via Composer (For PHP Projects)

#### 2. Git Submodule

#### 3. Git Subtree


## Variables

```
AWS_CODE_DEPLOY_KEY
AWS_CODE_DEPLOY_SECRET
AWS_CODE_DEPLOY_REGION

AWS_CODE_DEPLOY_APPLICATION_NAME

AWS_CODE_DEPLOY_DEPLOYMENT_GROUP_NAME
AWS_CODE_DEPLOY_DEPLOYMENT_CONFIG_NAME

AWS_CODE_DEPLOY_APP_SOURCE

AWS_CODE_DEPLOY_S3_BUCKET
AWS_CODE_DEPLOY_S3_KEY_PREFIX
AWS_CODE_DEPLOY_S3_FILENAME
AWS_CODE_DEPLOY_S3_LIMIT_BUCKET_FILES
AWS_CODE_DEPLOY_S3_SSE

AWS_CODE_DEPLOY_REVISION_DESCRIPTION
```


## Usage






## AWS Code Deploy Workflow

To deploy an application with AWS Code Deploy, the Wercker step follow this steps :

#### Step 1 : [Configuring AWS](http://docs.aws.amazon.com/cli/latest/reference/configure/index.html)

This initial step consists on configuring AWS.

The following configuration allows to setup this step :

* `key` (required): AWS Access Key ID
* `secret` (required): AWS Secret Access Key
* `region` (optional): Default region name

#### Step 2 : [Defining Application](http://docs.aws.amazon.com/cli/latest/reference/deploy/create-application.html)

This second step consists on defining the application. If the application does not exists this step create the application in Code Deploy.

The following configuration allows to setup this step :

* `application-name` (required): Name of the application to deploy
* `application-version` (optional): Version of the application to deploy. By default: Short commit id _(eg. fec8f4a)_

#### Step 3 : [Defining Deployment Config](http://docs.aws.amazon.com/cli/latest/reference/deploy/create-deployment-config.html) (optional)

This step consists on creating a deployment config. This step is totally *optional* because you can use the deployment strategy already defined in Code Deploy.

The following configuration allows to setup this step :

* `deployment-config-name` (optional): Deployment config name. By default : _CodeDeployDefault.OneAtATime_
* `minimum-healthy-hosts` (optional): The minimum number of healthy instances during deployment. By default : _type=FLEET_PERCENT,value=75_

#### Step 4 : [Defining Deployment Group](http://docs.aws.amazon.com/cli/latest/reference/deploy/create-deployment-group.html)

This step consists on defining a deployment group. If the deployment group provided does not exists this step create a deployment group in Code Deploy.

The following configuration allows to setup this step :

* `deployment-group-name` (required): Deployment group name
* `service-role-arn` (optional): Service role arn giving permissions to use Code Deploy when creating a deployment group
* `ec2-tag-filters` (optional): EC2 tags to filter on when creating a deployment group
* `auto-scaling-groups` (optional): Auto Scaling groups when creating a deployment group

#### Step 5 : [Pushing to S3](http://docs.aws.amazon.com/cli/latest/reference/deploy/push.html)

This step consists to push the application to S3.

The following configuration allows to setup this step :

* `s3-bucket` (required): S3 Bucket
* `s3-source` (optional): S3 Source. By default : _._
* `s3-key` (optional): S3 Key. By default: _{application-name}_

#### Step 6 : [Registering Revision](http://docs.aws.amazon.com/cli/latest/reference/deploy/register-application-revision.html)

This step consists to register the revision in Code Deploy.

The following configuration allows to setup this step :

* `revision` (optional): Revision of the application to deploy. By default: _{application-name}-{application-version}.zip_
* `revision-description` (optional): Description of the revision of the application to deploy

#### Step 7 : [Creating Deployment](http://docs.aws.amazon.com/cli/latest/reference/deploy/create-deployment.html)

This final step consists to create the deployment in Code Deploy.

The following configuration allows to setup this step :

* `deployment-description` (optional): Description of the deployment
* `deployment-overview` (optional): Visualize the deployment. By default : _true_

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
