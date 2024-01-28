---
title: "How to exec into your containers running on Amazon ECS"
seoTitle: "How to exec into your containers running on Amazon ECS"
datePublished: Wed Nov 29 2023 18:15:00 GMT+0000 (Coordinated Universal Time)
cuid: clrxb7j1i000109jv8ycg9u5l
slug: how-to-exec-into-your-containers-running-on-amazon-ecs
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1706434586758/c74c302d-6094-435d-83f6-663a0ff43864.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1706434732958/a842254d-ce8d-4f3d-a3aa-354c8cc8f645.png
tags: docker, aws, containers, ecs

---

## **Introduction**

**Amazon** Elastic Container Service (ECS) is a powerful container orchestration service that enables you to run containers in a highly scalable and cost-effective manner.

**ECS Exec** is a feature provided by AWS Systems Manager that leverages the AWS Identity and Access Management (IAM) role associated with your ECS task to grant you secure access to your containers

In this blog post, we will explore how to leverage ECS Exec to access containers running on both Fargate and EC2-backed ECS tasks.

### The Need

**Picture this:** You are a cloud engineer overseeing a fleet of containers running smoothly on Amazon's Elastic Container Service (ECS). Everything seems under control until one of your containerized applications starts throwing errors during peak traffic hours. You checked everything on your monitoring solution such as CloudWatch and it does provide information of substance. Or, perhaps, you need to run some commands and interact with various processes on your containers.

Either way, you need a way to quickly execute into the container. One solution is to SSH into the EC2 host running the containers and then perform `docker exec` against the container. But wait, Fargate does not even allow direct SSH. So, what now? There comes ECS Exec into play, a relatively new functionality, released by AWS in March 2021, that allows users to either run an interactive shell or a single command directly against a container.

For ECS running on EC2, this also removes the need for SSH or direct access to the host. This capability simplifies container management and makes it easier to diagnose issues, gather logs, and perform other necessary tasks.

### **Pre-requisites**

Before you can start using ECS Exec, you need to ensure you have the following in place:

1. **An AWS ECS Cluster**: You should already have an ECS cluster set up with one or more tasks running, either on Fargate or EC2 instances.
    
2. **IAM Permissions**: Ensure that the IAM roles associated with your ECS tasks have the necessary permissions to use Systems Manager. Specifically, you need the `ssm:StartSession` permission.
    
3. **Systems Manager Agent (SSM Agent)**: SSM Agent should be installed and running on your container instances. It comes pre-installed on most Amazon Machine Images (AMIs) provided by AWS.
    
4. **AWS Systems Manager Session Manager Plugin**: You should have the AWS Systems Manager Session Manager plugin installed on your local machine. This plugin enables you to initiate ECS Exec sessions from your terminal.
    
5. Note: ECS Exec is not currently supported using the AWS Management Console.
    

## **Steps**

Now, let's walk through the steps to access your containers using ECS Exec:

### **Step 1: Set Up AWS Systems Manager Session Manager Plugin**

If you haven't already installed the AWS Systems Manager Session Manager plugin on your local machine, you can follow the official AWS documentation to do so. This plugin is available for various operating systems, and installation is straightforward.

### **Step 2: Access Your Container**

Once the Session Manager plugin is installed, you can access your container using the `aws ssm start-session` command. Here's a basic example:

```bash
aws ecs execute-command  \
    --region $AWS_REGION \
    --cluster <cluster-name> \
    --task <task-id> \
    --container <container-name> \
    --command "/bin/bash" \
    --interactive
```

### **Step 3: Execute Commands**

After initiating the session, you'll be presented with a shell prompt that allows you to execute commands inside your container. You can run diagnostics, check logs, or make configuration changes as needed.

### **Step 4: Exit the Session**

When you're done, simply type `exit` to exit the session.

### **Step 5: Clean Up (Optional)**

It's a good practice to clean up unused sessions and resources. You can do this through AWS Systems Manager.

## **Conclusion**

ECS Exec, powered by AWS Systems Manager, is a valuable tool for container management and troubleshooting on Amazon ECS. It simplifies the process of accessing your containers running on Fargate and EC2, eliminating the need for SSH or direct host access. By following the prerequisites and steps outlined in this blog post, you can take advantage of ECS Exec to efficiently diagnose issues, collect logs, and perform maintenance tasks within your containers. This capability not only streamlines your container orchestration but also enhances your overall operational efficiency when working with ECS.

As you continue to leverage ECS Exec, you'll find it to be an essential part of your container management toolkit, improving your ability to maintain and troubleshoot containerized applications with ease.