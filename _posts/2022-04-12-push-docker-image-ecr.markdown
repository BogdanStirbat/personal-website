---
layout: post
title:  "Push a locally created Docker image to Amazon ECR"
date:   2022-04-12 12:16:18 +0300
categories: jekyll update
---

[Elastic Container Registry (ECR)](https://aws.amazon.com/ecr/) is a fully managed container registry offered by AWS. Once a container is pushed 
on ECR, you can run it in AWS using [Elastic Container Service (ECS)](https://aws.amazon.com/ecs/) or 
[Elastic Kubernetes Service (EKS)](https://aws.amazon.com/eks/) .

Suppose you already created a Docker image locally, and you want to push it to ECR. There are several steps to run:
 - Create an admin IAM user
 - install the AWS CLI tool
 - configure AWS CLI to use the configured admin IAM user's credentials 
 - create the repository in ECR

For creating the admin IAM user, you can follow the instructions from [here](https://docs.aws.amazon.com/mediapackage/latest/ug/setting-up-create-iam-user.html).
You will need an Access Key ID and a Secret access key.

For installing the AWS CLI tool, [here](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) you can
find instructions.

After the tool is installed, you need to configure it. Just execute `aws configure` and you will be prompted to enter the 
credentials for the admin IAM user.

The last step is to create the repository in ECR. Enter the AWS ECR console, and create the repository from there. 
Then, just push the local image to ECR.


### Links

https://docs.aws.amazon.com/mediapackage/latest/ug/setting-up-create-iam-user.html

https://mydeveloperplanet.com/2021/09/07/how-to-deploy-a-spring-boot-app-on-aws-ecs-cluster/

https://mydeveloperplanet.com/2021/10/12/how-to-deploy-a-spring-boot-app-on-aws-fargate/
