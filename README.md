# API Gateway to DynamoDB

Use AWS API Gateway as a proxy for DynamoDB.

## 🗂 Project structure

```yaml
.
├── .devcontainer      # used by VsCode to launch a devcontainer with SAM
├── docs               # documentation
├── scripts            # utility scripts to deploy and delete the AWS Cloudformation Stack
├── .gitignore         # .gitignore file
├── LICENCE            # license of the project
├── README.md          # this file
├── samconfig.toml     # configuration used by SAM when deploying the Cloudformation Stack
├── swagger.yaml       # extended OpenAPI description of the API Gateway configuration
├── template.yaml      # template that describes the serverless architecture and its resources
└── test-api.http      # basic CRUD operations to test the API
```

## 🧾 Requirements

The Serverless Application Model Command Line Interface (SAM CLI) is an
extension of the AWS CLI with added functionalities. It can also emulate your
application's build environment and API.

To install SAM, you can follow there
[instructions](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html).

You will also need an [AWS](https://aws.amazon.com/) account and a
[IAM user](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users.html) with
the right access policy you can use to connect from the console.\
The file storing the credentials should be in the default location
`~/.aws/credentials`, unless otherwise specified.

Optionally, you can install also the [AWS CLI](https://aws.amazon.com/cli/) for
more fine-tuned control over your operations.

### 🐳 Use the DevContainer

If you are using VsCode, you could take advantage of its DevContainer
functionalities. In that case, you would just need to have
[Docker](https://docs.docker.com/get-docker/) installed.

You still need the credentials of an account with enough permissions to perform
the deployment.\
The credentials file will be fetched from `~/.aws/credentials`, unless otherwise
specified.

## ⚙️ SAM configuration

The configuration is specified in the _samconfig.toml_ file. You can change it
freely, or add some more environments. Some of the settings include:

- **Profile**: name of the AWS profile you saved the console credentials for.
- **Confirm changeset**: whether to ask for manual confirmation after showing
  the changeset sam will produce
- **Stack Name**: The name of the stack to deploy to CloudFormation. This should
  be unique to your account and region, and a good starting point would be
  something matching your project name.
- **AWS Region**: The AWS region you want to deploy your app to.
- **Capabilities**: Many AWS SAM templates, including this example, create AWS
  IAM roles required for the AWS Lambda function(s) include access to AWS
  services. By default, these are scoped down to minimum required permissions.
  To deploy an AWS CloudFormation stack that creates or modifies IAM roles, the
  `CAPABILITY_NAMED_IAM` value for `capabilities` must be provided. If
  permission isn't provided through the config file, to deploy this example you
  must explicitly pass `--capabilities CAPABILITY_NAMED_IAM` to the `sam deploy`
  command.
- **s3 bucket**: name of the bucket used to upload the code. Must be globally
  unique.
- **s3 prefix**: folder inside the bucket that contains the uploaded code.
- **Parameter overrides**: key="value" pairs used to override the values of the
  parameters specified in the _template.yaml_ file

Here's an example of a _samconfig.toml_:

```toml
version = 0.1                                 # required

[dev.deploy.parameters]                       # used by sam deploy --config-env dev
profile = "default"                           # aws profile
confirm_changeset = true
capabilities = "CAPABILITY_NAMED_IAM"         # needed to manage IAM roles
stack_name = "library-stack"                  # name of the stack that will be deployed
s3_prefix = "library-stack-folder"            # folder in the S3 bucket
s3_bucket = "library-stack-bucket"            # S3 bucket unique name
region = "eu-west-1"
# should not include sensitive tokens. Overrides the parameters "apiStage" and "token"
parameter_overrides = "apiStage=\"dev\" tableName=\"library-table\""

[dev.local_invoke.parameters]                 # used by sam local invoke --config-env dev
env_vars = "env.json"                         # file that stores the environment variables
```

## ▶️ Deploy

To deploy the Cloudformation Stack, you can run

```bash
sam deploy --config-file samconfig.toml --config-env dev
# or
./scripts/deploy.sh
```

If a Stack happens to be in a ROLLBACK state, preventing you from doing any more
deployments or you simply want to delete it, you can do so from the AWS console
or, if you have the AWS CLI installed, you can run

```bash
aws cloudformation delete-stack --stack-name library-stack
# or
./scripts/delete.sh
```

Here's an example of what this template would produce:

![aws-schema](http://www.plantuml.com/plantuml/proxy?cache=no&src=https://raw.githubusercontent.com/TendTo/APIGateway-DynamoDB/master/docs/architecture.puml)

## Customization

This project is meant to be an example, so it is pretty simplistic. You may want
to change the model or add more routes.\
These are a few places in the template you may want to customize.

_template.yaml_

```yaml
#############################################
# Alther the key schema
#############################################
KeySchema:
  - AttributeName: pk
    KeyType: HASH
  - AttributeName: sk
    KeyType: RANGE
#############################################
# Alter the list of global secondary indexes
#############################################
GlobalSecondaryIndexes:
  - IndexName: GSI_1
    ...
```

_swagger.yaml_

```yaml
#############################################
# Change the model or add new ones
#############################################
definitions:
  Author: ...
#############################################
# Change the request mapping templates
#############################################
requestTemplates:
  application/json:
    Fn::Sub: |
      ...
```

## 🎨 Features

- **Parsed request-response**: use of
  [mapping templates](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-override-request-response-parameters.html)
  to make sure the request is well-formatted and the response is as concise as
  possible and with proper status code.\
  The response mapping is mostly generic so that it is possible to add or modify
  models with minimal changes
- **Ready to use**: once properly configured and deployed, the whole API is
  ready to use. There is no need to tweak any settings manually from the AWS
  console. Every resource needed is already defined in the template

## ⚠️ Limitations

- **Response mapping template**: the template used is generic, but it is not suited for parsing through _lists_ or _mapping_ of any length.  
  To do it properly, AWS mapping templates should support [Velocimacros](https://velocity.apache.org/engine/1.7/user-guide.html#velocimacros),
  but right now they [don't](https://forums.aws.amazon.com/thread.jspa?threadID=284790).  
  If your model has a known depth, you can duplicate the list or map macro that many times, giving it each time a different name.

## 📚 Resources

- [AWS Compute Blog: Using Amazon API Gateway as a proxy for DynamoDB](https://aws.amazon.com/it/blogs/compute/using-amazon-api-gateway-as-a-proxy-for-dynamodb/)
- [Medium: Using API Gateway To Get Data From Dynamo DB Without Using AWS Lambda](https://medium.com/@likhita507/using-api-gateway-to-get-data-from-dynamo-db-using-without-using-aws-lambda-e51434a4f5a0)
