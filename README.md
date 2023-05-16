# project2
Amazon ECS Demo with fullstack app / DevOps practices / Terraform sample
Table of content
Solution overview
General information
Infrastructure
Infrastructure Architecture
Infrastructure considerations due to demo proposals
CI/CD Architecture
Prerequisites
Usage
Autoscaling test
Application Code
Client app
Client considerations due to demo proposal
Server app
Cleanup
Security
License
Solution overview
This repository contains Terraform code to deploy a solution that is intended to be used to run a demo. It shows how AWS resources can be used to build an architecture that reduces defects while deploying, eases remediation, mitigates deployment risks and improves the flow into production environments while gaining the advantages of a managed underlying infrastructure for containers.

General information
The project has been divided into two parts:

Code: the code for the running application
client: Vue.js code for the frontend application
server: Node.js code for the backend application
Infrastructure: contains the Terraform code to deploy the needed AWS resources for the solution
Infrastructure
The Infrastructure folder contains the terraform code to deploy the AWS resources. The Modules folder has been created to store the Terraform modules used in this project. The Templates folder contains the different configuration files needed within the modules. The Terraform state is stored locally in the machine where you execute the terraform commands, but feel free to set a Terraform backend configuration like an AWS S3 Bucket or Terraform Cloud to store the state remotely. The AWS resources created by the script are detailed bellow:

AWS Networking resources, following best practices for HA
2 ECR Repositories
1 ECS Cluster
2 ECS Services
2 Task definitions
4 Autoscaling Policies + Cloudwatch Alarms
2 Application Load Balancer (Public facing)
IAM Roles and policies for ECS Tasks, CodeBuild, CodeDeploy and CodePipeline
Security Groups for ALBs and ECS tasks
2 CodeBuild Projects
2 CodeDeploy Applications
1 CodePipeline pipeline
2 S3 Buckets (1 used by CodePipeline to store the artifacts and another one used to store assets accessible from within the application)
1 Dynamodb table (used by the application)
1 SNS topic for notifications
Infrastructure Architecture
The following diagram represents the Infrastructure architecture being deployed with this project:



Infrastructure considerations due to demo proposals
The task definition template (Infrastructure/Templates/taskdef.json) that enables the CodePipeline to execute a Blue/Green deployment in ECS has hardcoded values for the memory and CPU values for the server and client application.

Feel free to change it, by adding for example a set of "sed" commands in CodeBuild (following the ones already provided as example) to replace the values dynamically.

Feel fre to create a subscriptor for the SNS topic created by this code, in order to get informed of the status of each finished CodeDeploy deployment.

CI/CD Architecture
The following diagram represents the CI/CD architecture being deployed with this project:



Prerequisites
There are general steps that you must follow in order to launch the infrastructure resources.

Before launching the solution please follow the next steps:

Install Terraform, use Terraform v0.13 or above. You can visit this Terraform official webpage to download it.
Configure the AWS credentials into your machine (~/.aws/credentials). You need to use the following format:
    [AWS_PROFILE_NAME]
    aws_access_key_id = Replace_with_the_correct_access_Key
    aws_secret_access_key = Replace_with_the_correct_secret_Key
Generate a GitHub token. You can follow this steps to generate it.
Usage
1. Fork this repository and create the GitHub token granting access to this new repository in your account.

2. Clone that recently forked repository from your account (not the one from the aws-sample organization) and change the directory to the appropriate one as shown below:

cd Infrastructure/
3. Run Terraform init to download the providers and install the modules

terraform init 
4. Run the terraform plan command, feel free to use a tfvars file to specify the variables. You need to set at least the followign variables:

aws_profile = according to the profiles name in ~/.aws/credentials
aws_region = the AWS region in which you want to create the resources
environment_name = a unique name used for concatenation to give place to the resources names
github_token = your GitHub token, the one generated a few steps above
repository_name = your GitHub repository name
repository_owner = the owner of the GitHub repository used
terraform plan -var aws_profile="your-profile" -var aws_region="your-region" -var environment_name="your-env" -var github_token="your-personal-token" -var repository_name="your-github-repository" -var repository_owner="the-github-repository-owner"
Example of the previous command with replaced dummy values:

terraform plan -var aws_profile="development" -var aws_region="eu-central-1" -var environment_name="developmentenv" -var github_token="your-personal-token" -var repository_name="your-github-repository" -var repository_owner="the-github-repository-owner"
5. Review the terraform plan, take a look at the changes that terraform will execute:

terraform apply -var aws_profile="your-profile" -var aws_region="your-region" -var environment_name="your-env" -var github_token="your-personal-token" -var repository_name="your-github-repository" -var repository_owner="the-github-repository-owner"
6. Once Terraform finishes the deployment open the AWS Management Console and go to the AWS CodePipeline service. You will see that the pipeline, which was created by this Terraform code, is in progress. Add some files and Dynamodb items as mentioned here. Once the pipeline finished successfully and the before assets were added, go back to the console where Terraform was executed, copy the application_url value from the output and open it in a browser.

7. In order to access the also implemented Swagger endpoint copy the swagger_endpoint value from the Terraform output and open it in a browser.

Autoscaling test
To test how your application will perform under a peak of traffic, a stress test configuration file is provided.

For this stress test Artillery is being used. Please be sure to install it following these steps.

Once installed please change the ALB DNS to the desired layer to test (front-/backend) in the target attribute, which you can copy from the generated Terraform output, or you can also search it in the AWS Management Console.

To execute it run the following commands:

Frontend layer:

artillery run Code/client/src/tests/stresstests/stress_client.yml
Backend layer:

artillery run Code/server/src/tests/stresstests/stress_server.yml
To learn more about Amazon ECS Autoscaling please take a look to this documentation.

Application Code
Client app
The Client folder contains the code to run the frontend. This code is written in Vue.js and uses the port 80 in the deployed version, but when run localy it uses port 3000.

The application folder structure is separeted in components, views and services, despite the router and the assets.

Client considerations due to demo proposals
The assets used by the client application are going to be requested from the S3 bucket created with this code. Please add 3 images to the created S3 bucket.

The Dynamodb structure used by the client application is the following one:

  - id: N (HASH)
  - path: S
  - title: S
Feel free to change the structure as needed. But in order to have full demo experience, please add 3 Dynamodb Items with the specified structure from above. Below is an example.

Note: The path attribute correspondes to the S3 Object URL of each added asset from the previous step.

Example of a Dynamodb Item:

{
  "id": {
    "N": "1"
  },
  "path": {
    "S": "https://mybucket.s3.eu-central-1.amazonaws.com/MyImage.jpeg"
  },
  "title": {
    "S": "My title"
  }
}
Server app
The Server folder contains the code to run the backend. This code is written in Node.js and uses the port 80 in the deployed version, but when run localy it uses port 3001.

Swagger was also implemented in order to document the APIs. The Swagger endpoint is provided as part of the Terraform output, you can grab the output link and access it through a browser.

The server exposes 3 endpoints:

/status: serves as a dummy endpoint to know if the server is up and running. This one is used as the health check endpoint by the AWS ECS resources
/api/getAllProducts: main endpoint, which returns all the Items from an AWS Dynamodb table
/api/docs: the Swagger enpoint for the API documentation
