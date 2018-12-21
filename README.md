# signed_url

## Requirements

* AWS CLI already configured with at least PowerUser permission
* [Ruby](https://www.ruby-lang.org/en/documentation/installation/) 2.5 installed
* [Docker installed](https://www.docker.com/community-edition)
* [Ruby Version Manager](http://rvm.io/)

## Setup process

### Installing dependencies

```sam-app``` comes with a Gemfile that defines the requirements and manages installing them.

```bash
gem install bundler
bundle install
bundle install --deployment --path signed_url/vendor/bundle
```

* Step 1 installs ```bundler```which provides a consistent environment for Ruby projects by tracking and installing the exact gems and versions that are needed.
* Step 2 creates a Gemfile.lock that locks down the versions and creates the full dependency closure.
* Step 3 installs the gems to ```signed_url/vendor/bundle```.

**NOTE:** As you change your dependencies during development you'll need to make sure these steps are repeated in order to execute your Lambda and/or API Gateway locally.

### Local development

**Invoking function locally through local API Gateway**

```bash
sam local start-api
```

If the previous command ran successfully you should now be able to hit the following local endpoint to invoke your function `http://localhost:3000/signed_url`

**SAM CLI** is used to emulate both Lambda and API Gateway locally and uses our `template.yaml` to understand how to bootstrap this environment (runtime, where the source code is, etc.) - The following excerpt is what the CLI will read in order to initialize an API and its routes:

## Packaging and deployment

AWS Lambda Ruby runtime requires a flat folder with all dependencies including the application. SAM will use `CodeUri` property to know where to look up for both application and dependencies:

Firstly, we need a `S3 bucket` where we can upload our Lambda functions packaged as ZIP before we deploy anything - If you don't have a S3 bucket to store code artifacts then this is a good time to create one:

```bash
aws s3 mb s3://BUCKET_NAME
```

Next, run the following command to package our Lambda function to S3:

```bash
sam package \
    --template-file template.yaml \
    --output-template-file packaged.yaml \
    --s3-bucket REPLACE_THIS_WITH_YOUR_S3_BUCKET_NAME
```

Next, the following command will create a Cloudformation Stack and deploy your SAM resources.

```bash
sam deploy \
    --template-file packaged.yaml \
    --stack-name signedUrl \
    --capabilities CAPABILITY_IAM
```

> **See [Serverless Application Model (SAM) HOWTO Guide](https://github.com/awslabs/serverless-application-model/blob/master/HOWTO.md) for more details in how to get started.**

After deployment is complete you can run the following command to retrieve the API Gateway Endpoint URL:

```bash
aws cloudformation describe-stacks \
    --stack-name sam-app \
    --query 'Stacks[].Outputs'
``` 
