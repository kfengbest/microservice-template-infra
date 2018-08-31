A Docker Container Microservice Template on AWS Cloud
=====================================================

Features
--------

1. One-Click Deployment

Components
----------

1. Application CI/CD Pipeline
2. Elastic Beanstalk Based Microservice Framework


Quick Start
-----------

1. Prerequisite:

    * A AWS account/IAM user with Administrative Permission.
    
    * Install awscli
        ```bash
        pip install awscli
        ```
    * Configure awscli with your access key and token.
        ```bash
        aws configure
        ```
    * Choose a AWS region that supports Code* Services and ElasticBeanstalk.
    
2. Clone code
    ```bash
        git clone https://github.com/kfengbest/microservice-template-infra.git
    ```
3. Create TWO CodeCommit Repository in your AWS account, One for infra code and the other for app code
    ```bash
       aws codecommit create-repository --repository-name MyMicroserviceInfra
       aws codecommit create-repository --repository-name MyMicroserviceApp
    ```
4. Edit and Push the code to the MyMicroserviceInfra Repository.

   Edit config.prod.json and config.stag.json, modify application repo ARN to your  
   new created MyMicroserviceApp ARN. And then commit and push the code to 
   MyMicroserviceInfra Repo. (You can also edit other parameters if needed.)

   Since we can not access code repository in the company directly, This is 
   a temporary solution for now. 
   The following pipeline can be used to bypass the above restriction. But they
   are not included here now. 
   
   Git(Company) -> Jenkins -> S3 -> CodeDeploy -> EB
    
5. The template can be deployed in two way. (Assuming in the Region us-east-1)

    a. Without Infrastructure CI/CD Pipeline:
    ```bash
    cd solutions/
    aws s3api create-bucket --bucket [BUCKET_NAME] --region us-east-1
    aws cloudformation package \
       --template-file Main.template.yaml \
       --s3-bucket [BUCKET_NAME] \
       --output-template-file packaged.template.yaml
    aws cloudformation deploy \
       --template-file packaged.template.yaml \
       --stack-name [YOUR_STACK_NAME] \
       --capabilities CAPABILITY_IAM \
       --parameter-overrides RepositoryArn=[YOUR_APP_REPO_ARN] \
                           RepositoryBranch=master \
                           InstanceType=t2.micro \
                           AvailabilityZones=us-east-1a,us-east-1b \
                           NumberOfAZs=2 \
                           AutoScalingMinSize=2 \
                           AutoScalingMaxSize=4 \
                  
    ```
    Please modify above parameters according to your needs.
    
    b. With Infrastructure CI/CD Pipeline:
    
    ```bash
    cd solutions/CfnCD/
    aws cloudformation deploy \
       --template-file SimpleCfnCd.template.yaml \
       --stack-name AppInfraCiCdStack \
       --capabilities CAPABILITY_IAM \
       --parameter-overrides CfnRepoArn=[YOUR_INFRA_REPO_ARN] \
                           CfnStackName=AppStack \
                           ApprovalEmails=[YOUR_EMAIL] \
                           NumOfEmails=1 \
                           UseStaging=no
    ```
    
 6. Now you can commit code in your application repository.
