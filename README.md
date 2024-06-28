# catch-me-if-you-can game
A fun little game using React, Flask, Redis, and OpenStreetMap

# Installation and Setup

## On localhost:

### Pre-requisite
python, npm, redis_server

Download and open https://github.com/aashishrc/catch-me-if-you-can in your machine.

In your terminal/command prompt 'cd' into the game folder.

#### Backend
Start your local redis server (make sure redis is installed). Run the below command in your terminal

```redis-server```

In a new terminal windown, go inside 'be' folder and run

```python wsgi.py```

This will start your Flask backend server and should show like this:

![alt text](image.png)

Note down the IP address the Flask app is runing on. This will be the value of REACT_APP_API_SERVICE_URL in your .env file (create .env file inside 'fe' folder) and add the a similar entry 
REACT_APP_API_SERVICE_URL=http://10.0.0.209:5000  //Replace with your Flask URL

#### Frontend

In a new terminal windown, go inside 'fe' folder and run
```npm start```
This should start your React app in your browser at http:localhost:3000

## Using Docker:

### Pre-requisite
docker

Download and open https://github.com/aashishrc/catch-me-if-you-can in your machine.

In your terminal/command prompt 'cd' into the game folder.

In a new terminal windown run the following commands

```docker-compose build -t catch-me-if-you-can .```

```docker-compose up```


## Using AWS

### Steps:

#### ECR

Login in to AWS Console and head to ECR. Create 3 new repositories.
Name them as catch-me-if-you-can-db, catch-me-if-you-can-be, catch-me-if-you-can-fe. 

#### CodeBuild

Navigate to AWS CodeBuild and create a new project named catch-me-if-you-can. 

Under Source, select Github and choose 'Public Repository' and provide the Repository URL as https://github.com/aashishrc/catch-me-if-you-can


After selecting your Source, Primary source webhook events should be visible. Check the box for Rebuild every time a code change is pushed to this repository and add Webhook Filter Event type as PUSH.

Under Environment, select Operating System as Ubuntu, select the correct VPC, Subnets and Security Groups.

Add 2 Environment Variables: AWS_ACCOUNT_ID and AWS_REGION and given them correct values.

Finally, choose 'Use a buildspec file' option and Create the build project.

A new role 'codebuild-catch-me-if-you-can-service-role' should be created in your IAM roles. Add 'AmazonEC2ContainerRegistryFullAccess', 'AmazonEC2ContainerRegistryReadOnly', and 'EC2InstanceProfileForImageBuilderECRContainerBuilds' permissions.

Now that you have your CodeBuild set up, everytime we make a new push to the repository, the PUSH webhook will trigger. However, for testing purposes we will start the build manually by clicking the Start Build button inside your project.

This will fetch the buildspec.yml file from our GitHub repo and start the build. This will take some time.

#### Task Definitions

Now navigate to ECS Task Definitions and create a task definition for your frontend, backend and database. Name them as catch-me-if-you-can-fe-td, catch-me-if-you-can-be-td, and catch-me-if-you-can-db-td. 

We will be using Lauch type as AWS Fargate for ease of deployment 

Select ecsTaskExecutionRole under Task Roles or None (if the role doesn't exist), and ecsTaskExecutionRole or New Role under Task execution role.

Under Container section, we have to be careful for each Task Definition.
First let's create the 'db' container. The image URI will be the same as the URI available for the respective Repository in ECR. 

'db' container will have URI as <AWS_ACCOUNT_ID>.dkr.ecr.<AWS_REGION>.amazonaws.com/catch-me-if-you-can-db:prod 
Container port as 6397 for Redis 

'be' container will have URI as <AWS_ACCOUNT_ID>.dkr.ecr.<AWS_REGION>.amazonaws.com/catch-me-if-you-can-be:prod 
Container port as 5000 for FLASK backend

'fe' container will have URI as <AWS_ACCOUNT_ID>.dkr.ecr.<AWS_REGION>.amazonaws.com/catch-me-if-you-can-fe:prod 
Container port as 80 for React frontend

Select log collection checkbox. This will enable CloudWatch logging.
Submit.

#### Target Groups

Create a new Target Groups 

1 'catch-me-if-you-can-db-tg' target group for db, target type TCP

1 'catch-me-if-you-can-fe-tg' target group for fe, target type HTTP & 1 'catch-me-if-you-can-be-tg'target group for be, target type HTTP

#### Load Balancers

Create Network Load balancer 'catch-me-if-you-can-nlb' for your database. Select the same VPC, subnets and security groups as you selected before. 

Create Application Load balancer 'catch-me-if-you-can-alb'. Select the same VPC, subnets and security groups as you selected before. 

#### EKS

Create a EKS cluster (with fargate serverless infrastructure)

Create 3 services (similar to docker compose). Services are created using your Task Definitions and Load Balancers. Services run your tasks.