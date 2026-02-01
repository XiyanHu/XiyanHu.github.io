---
title: Deploy Memos on AWS for fun
date: 2023-05-01 20:28:06
categories: 
- 瞎折腾
---
Memos [Github Link](https://github.com/usememos/memos) is a lightweight, self-hosted memo hub. Open Source. Currently there are docs supporting manage hosting [link](https://usememos.com/docs/install/managed) on Render.com, Fly.io, PikaPods.com.

I tried to deploy Memos app on AWS with ECR, ECS Fargate and Application Load balancer(ALB). JUST FOR FUN. Because it is too **EXPENSIVE**. Memo is a long-lived app and almost certainly needs a load balancer. ALB by itself will cost $16.2/month.
My setup has no HTTPS, no CDK. It is against a lot of best practices :).

I will recommend using Fly.io
- You can manage the whole thing at $0! 
- Automatic db backup to S3 bucket with easy setup
- Easy to upgrade

I quickly moved to Fly.io after I finished this blog :)

Assume you already have a AWS account.

### Local code change
* Pull the git repo to local environment https://github.com/usememos/memos
* If you want to automatically backup db file to a S3 bucket
  *  Go to AWS Console S3 page, create a bucket. Give it a name, for example, mymemos
  *  Modify your local repo following this [commit example](https://github.com/usememos/memos/compare/main...XiyanHu:memos:blog), replace litestream.yml file with your information
     *  This is basically copy paste from https://github.com/hu3rror/memos-on-fly

### Deploy to ECR
*  Search ECR in AWS console. Click Create repository, give it a name, for example, mymemos
*  You will see your docker on the dashboard. Click on your docker repository
*  Click View Push Commands, and follow the instruction there to deploy your app
{% asset_img push_command_screenshot.png This is an example image %}

### Create ALB
*  Search load balancers in AWS console. Find EC2 and on left panel click Load Balancers
*  Click Create load balancer, choose Application Load Balancer
   *  Give it a name, choose VPC and Security groups
   *  Under Listeners and routing tab, click Create target group 
      *  Choose IP addresses as target type
      *  Protocol HTTP and Port 5230
   * Select the target group just created
   * Click Create Load Balancer

### Config Security group
*  I only have a default security group, select it. Under Inbound rules tab, click Edit Inbound rules
*  Add Type Custom TCP, Port Range 5230, Source 0.0.0.0/0
*  Add Type HTTP, Port Range 80, Source 0.0.0.0/0
*  Save it and go to outbound rules, Add Type Custom TCP, Port Range 5230, Source 0.0.0.0/0

### Config ECS
* Search ECS in AWS console, on left panel, click Task definitions
* Click Create new task definition
 * Set Image Url to XXXXXX.dkr.ecr.us-east-1.amazonaws.com/mymemos, you can get it from ECR page
 * Set port tp 5230 with is the default in memos app
 * Click Create if you don't want to do other fancy settings
{% asset_img task_definition.png This is an example image %}
* Click Cluster on left panel, then Create cluster
  * Set Cluster name, add VPC and subnet
  * Click Create
* Select the cluster you created
  * If you want to save money, consider [Fargate Spot](https://aws.amazon.com/blogs/aws/aws-fargate-spot-now-generally-available/). 70% discount!!!
  * Under Services Tab, click Create
    * Under Deployment configuration, choose the family created in the previous step
{% asset_img service_deployment_config.png This is an example image %}
  * Under Load Balancer, choose the ALB and target group created in the previous step 
{% asset_img service_config_lb.png This is an example image %}
  * After service is created you should see a task running. 
  * Go to your broswer with your ALB DNS name you should see the memo homepage let you to create an user
{% asset_img alb.png This is an example image %}
{% asset_img signin.png This is an example image %}
  * In the s3 bucket you should see a folder name with memos_prod.db/. 

### What's Next
- Consider replace ALB with API Gateway + CloudMap to cut cost : https://awsteele.com/blog/2022/10/15/cheap-serverless-containers-using-api-gateway.html
- Move to Fly.io :) 