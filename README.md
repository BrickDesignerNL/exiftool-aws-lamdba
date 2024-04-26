# About this fork
Work in progress.
Don't expect working code.
## Done
* Upgrade to ExifTool version 12.84
* Upgrade to Perl Maint version 5.38.2
* Upgrade to Python version 3.12
* Automatically take the latest 3.12 release of the Python Lambda image
* Use ARM architecture instead of Intel architecture for Lambda
## TODO
Things I like to add / be added
* Add a SQS queue + dead letter queue
* Make code automatically take latest stable version of ExifTool
* Make code automatically take latest stable Perl Maint
* Make code automatically pick latest stable Lambda Python image and update the template accordingly - for now now it only takes the latest 3.12 version.
* Add CodePipeline
* Add CodeBuild
* Trigger new build when ExifTool updates
* Trigger new build when a new Perl Maint version is released
* Trigger new build when AWS Lambda runtime image updates
* Automatically test new build
* Automatiocally deploy new build when test is succesful
# From source repo
## Exiftool Lambda

The code provides a fully working example of AWS Lambda using [exiftool](https://exiftool.org/).
It is written in python and intended to work under limited disk and RAM resources. The core functionality simply downloads chunks of a file and fills stdin for `exiftool`. As soon as exiftool get all information, it exits. That way, even large files can be analyzed.

The AWS diagram is presented below:


    +---------------+                            +-------------------------+
    |               |  s3:ObjectCreated:* event  |                         |
    |   S3 bucket   +--------------------------->|   Exiftool AWS Lambda   |
    |               |                            |                         |
    +---------------+                            +-------------------------+

More information can be found on [blog post](https://codegyver.com/2022/08/22/exiftool-aws-lambda/).

## Usage

Exiftool lambda can be useful for obtaining metadata of uploaded file. I.e. user uploads a file to S3, it is then passed to our Lambda and extracted metadata is returned in JSON format (which can be later passed to another Lambda for further processing).

Since AWS Lambda is limited environment, additional layer needs to be created and attached. In order to generate layer (with `exiftool` executable and its dependencies) a command needs to be run:

    docker compose --env-file .docker.env -f docker-compose.yml up --build --force-recreate layer

Please note that all environment variables are stored in `.docker.env`. In order to adjust `perl` or `exiftool` version it needs to be changed there. After docker image is build it copies `exiftool.zip` to layer directory.

Once we have layer generated, we can run tests. This also happens via docker to mimic the AWS Lambda environment.

    docker compose --env-file .docker.env -f docker-compose.yml up --build --force-recreate test

## Deploy

Deploy to AWS can be done with provided SAM (Cloudformation) template. In order to run a `sam` command, we need to have the following variables prepared:

    # SAM_BUCKET - bucket for storing sam temporary files
    # STACK_NAME - name of the stack
    # AWS_REGION - aws region where to deploy to

Then we can package and deploy as with these `sam` commands:

    sam package --template-file aws.template.yaml --output-template-file packaged.yaml --s3-bucket SAM_BUCKET --region AWS_REGION

    sam deploy  --template-file packaged.yaml --s3-bucket SAM_BUCKET --stack-name STACK_NAME --region AWS_REGION --capabilities CAPABILITY_IAM
