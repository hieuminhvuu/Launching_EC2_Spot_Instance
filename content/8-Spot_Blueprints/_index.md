---
title : "Spot Blueprint"
date : "`r Sys.Date()`"
weight : 8
chapter : false
pre : " <b> 8. </b> "
---

Spot Blueprints is a functionality provided within the AWS Web Console, in the Spot Request section, that helps to create a few architectures that are most common for Spot, using Infrastructure as code and adhering to Spot Best practices. There are Spot Blueprints for the most popular services including Amazon EC2 Auto Scaling, Amazon EMR, AWS Batch, and Amazon Elastic Kubernetes Service (Amazon EKS).

Spot Blueprints gives you a jump start in using Spot architectures by providing a simple-to-follow infrastructure code template generator that is designed to gather your workload requirements while explaining and configuring Spot best practices along the way. The output of the process is a Cloudformation or Terraform IaaC (Infrastructure as Code) file that you can use and adapt for your projects.

##### Getting started with Spot Blueprints

Open the Amazon EC2 console at [https://console.aws.amazon.com/ec2/](https://console.aws.amazon.com/ec2/).
In the navigation pane, choose Spot Requests.
Choose Spot Blueprints in the top right corner.

![Spot Blueprints](/images/s1/108.png) 

You will see different categories to get you started. From there, you can either download a pre-configured blueprint in AWS CloudFormation or Terraform format, or choose to configure it. You can learn more about Spot Blueprints by reading the [launch blog post](https://aws.amazon.com/blogs/compute/introducing-spot-blueprints-a-template-generator-for-frameworks-like-kubernetes-and-apache-spark/). If you don’t find a blueprint that you need, feel free to provide us feedback using the [Spot Blueprints Feedback link](https://us-east-1.console.aws.amazon.com/ec2/home?redirectFrom=ec2sp&region=us-east-1#SpotBlueprints:).