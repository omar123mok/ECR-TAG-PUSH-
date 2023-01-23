
# ECR-Tag-Push-

This is a GitHub Actions workflow that is triggered on a "push" event. The workflow consists of several steps that perform the following actions:

1.Check out the repository using the actions/checkout@v2 action.

2.Configure AWS credentials using the aws-actions/configure-aws-credentials@v1 action, using secrets stored in the GitHub repository for the AWS access key ID, secret access key, and region.

3.Login to Amazon Elastic Container Registry (ECR) using the aws-actions/amazon-ecr-login@v1 action.

4.Build a Docker image, tag it with the GitHub commit SHA, and push it to the ECR repository specified in the workflow.

5.The environment variable REGISTRY, REPOSITORY, IMAGE_TAG are defined in the workflow.

## Event

Triggered by push

## Code

The Workflow script.


```yaml
name: ecr

on  : push

jobs:
    ecr-login:
      runs-on: ubuntu-latest

      steps:
        - name: Checkout repo
          uses: actions/checkout@v2
        
        - name: Configure AWS credentials
          uses: aws-actions/configure-aws-credentials@v1
          with:
            aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
            aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
            aws-region: ${{ secrets.AWS_REGION }}

        - name: Login to Amazon ECR
          id: login-ecr
          uses: aws-actions/amazon-ecr-login@v1

        - name: Build, tag, and push docker image to Amazon ECR
          env:
            REGISTRY: ${{ steps.login-ecr.outputs.registry }}
            REPOSITORY: repo
            IMAGE_TAG: ${{ github.sha }}
          run: |
            docker build -t $REGISTRY/$REPOSITORY:$IMAGE_TAG .
            docker push $REGISTRY/$REPOSITORY:$IMAGE_TAG

```
# Dockerfile
The Dockerfile defines the steps to create an image of a Python 3.6 environment with the necessary dependencies for a Flask application.

# Code

```Dockerfile
# use a python image
FROM python:3.6

# set the working directory in the container to /app
WORKDIR /app

# add the current directory to the container as /app
COPY . /app

# pip install flask
# RUN pip install --upgrade pip && \
#     pip install \
#         Flask \
#         awscli \
#         flake8 \
#         pylint \
#         pytest \
#         pytest-flask

# expose the default flask port
EXPOSE 8080

# execute the Flask app
ENTRYPOINT ["python"]
HEALTHCHECK CMD curl --fail http://localhost:8080/ || exit 1
CMD ["/app/app.py"]
```

It does the following:

1.Uses the python:3.6 image as the base image for the container

2.Sets the working directory inside the container to /app

3.Copies the contents of the current directory on the host machine to /app in the container

4.Installs the flask, awscli, flake8, pylint, pytest and pytest-flask package via pip.(commented out)

5.Exposes port 8080 on the container

6.Sets the command to start the Flask app using python, and the healthcheck command using curl.

7.Finally, the entrypoint of the container is set to python and command to run is set to /app/app.py

This Dockerfile will be used to build the image with all the dependencies required to run a Flask application.




