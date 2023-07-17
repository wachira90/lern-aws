# Using a container image to add the AWS AppConfig Lambda extension

1. Dockerfile

```Dockerfile
FROM public.ecr.aws/lambda/python:3.8 AS builder
COPY extension.zip extension.zip
RUN yum install -y unzip \
  && unzip extension.zip -d /opt \
  && rm -f extension.zip

FROM public.ecr.aws/lambda/python:3.8
COPY --from=builder /opt /opt
COPY index.py ${LAMBDA_TASK_ROOT}
CMD [ "index.handler" ]
```


2. Download the extension layer to the same directory as the Dockerfile.

```sh
aws lambda get-layer-version-by-arn \
  --arn arn:aws:lambda::layer:AWS-AppConfig-Extension:00 \
  | jq -r '.Content.Location' \
  | xargs curl -o extension.zip
```

3. Create a Python file named index.py in the same directory as the Dockerfile.

```py
import urllib.request

def handler(event, context):
    return {
       # replace parameters here with your application, environment, and configuration names
       'configuration': get_configuration('myApp', 'myEnv', 'myConfig'),
    }

def get_configuration(app, env, config):
    url = f'http://localhost:2772/applications/{app}/environments/{env}/configurations/{config}'
    return urllib.request.urlopen(url).read()
```

4. Run the following steps to build the docker image and upload it to Amazon ECR.

```sh
// set environment variables
export ACCOUNT_ID = <YOUR_ACCOUNT_ID>
export REGION = <AWS_REGION>

// create an ECR repository
aws ecr create-repository --repository-name test-repository

// build the docker image
docker build -t test-image .

// sign in to AWS
aws ecr get-login-password \
  | docker login \
  --username AWS \
  --password-stdin "$ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com"

// tag the image 
docker tag test-image:latest "$ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com/test-repository:latest"

// push the image to ECR 
docker push "$ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com/test-repository:latest"            
```

5. Use the Amazon ECR image that you created above to create the Lambda function. For more information about a Lambda function as a container, see Create a Lambda function defined as a container image.

https://docs.aws.amazon.com/lambda/latest/dg/getting-started-create-function.html#gettingstarted-images-function

6. Ensure that the Lambda function execution role has the appconfig:GetConfiguration permission set.

https://docs.aws.amazon.com/appconfig/2019-10-09/APIReference/API_GetConfiguration.html



