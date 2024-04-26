# About this fork
Work in progress.
Not working code yet.
Intention os to upgrade to latest Lambda runtime & latest version of ExifTool. Ising ARM instead of Intel architecture and adding an SQS queue to ensure event delivery.

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
