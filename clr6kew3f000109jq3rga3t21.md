---
title: "What's all about AWS Default VPC?"
seoTitle: "All About AWS Default VPC"
seoDescription: "Explore AWS Default VPC: characteristics, use cases, customization, creation via Console, CLI, Terraform; learn when to avoid them"
datePublished: Tue Nov 14 2023 18:15:00 GMT+0000 (Coordinated Universal Time)
cuid: clr6kew3f000109jq3rga3t21
slug: whats-all-about-aws-default-vpc
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1704860219280/5a3448f5-8437-42fb-adab-bc442ffa4e9f.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1704860232179/89eb036e-ef82-42c6-bdda-0d4151c950f5.png
tags: aws, aws-vpc, aws-networking, aws-default-vpc

---

### **Introduction**

Did you know that your AWS account, if it was created after **4th December 2013** has a default VPC in each AWS Region? In this blog, we will talk about the various characteristics of the default VPC, discuss its use cases, and see how you can create one from the AWS Console, CLI, and Terraform. We will also run through scenarios when you would not want to use the default VPC. So, let's get started 🚀

### **Why do we need AWS Default VPC?**

First thing first, most if not all AWS resources require you to define or choose a Virtual Private Cloud (VPC) for it to run. VPC is the underlying foundation that creates a logically isolated virtual network for your resources. That's where the AWS Default VPC comes into play. So, instead of throwing a bunch of errors (such as missing subnets, no-internet connection, etc.) when you try to launch an EC2 instance or any other AWS service, it can use the default VPC and let you get that aah-hah deployment success message!

YES! You could create your own VPC and launch your resources there (we will cover this later in the blog)! But, what if you don't want to dive into understanding subnetting, setting up an Internet Gateway, Route Tables, etc? If this sounds scary enough, and you want to quickly get started, the default VPC comes to your rescue. You can immediately start launching your Amazon EC2 instances into a default VPC. You can also use services such as Elastic Load Balancing, Amazon RDS, Amazon ECS, Amazon EMR, and others in your default VPC.

### **Characteristics of AWS Default VPC**

Alright! I got that. But what exactly makes up the Default VPC you may ask? Here's your answer:

![				We create a default VPC in each Region, with a default subnet in each Availability Zone.			](https://docs.aws.amazon.com/images/vpc/latest/userguide/images/default-vpc.png align="center")

1. **IPv4 Address Space**: IPv4 CIDR block (`172.31.0.0/16`). Notice the **size /16** here. This provides a vast pool of **65,536 private IPv4 addresses**, essential for scalability and resource allocation. Notice the IPv4 address space in the image below.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1704814652979/5fe3503d-ac78-4814-b46d-5f18ead72a9b.png align="center")
    
2. **Subnets**: Each Availability Zone (AZ) gets a dedicated /20-size public subnet, allowing for **4,096 IP addresses** per subnet. Note that, all the EC2 Instances that you launch in a default subnet get both a public IPv4 address and a private IPv4 address, as well as both public and private DNS hostnames.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1704813601691/d2ce3dac-44bd-4f09-b784-cf3ce27d8731.png align="center")
    
3. **Gateway to the Internet**: An **Internet Gateway (IGW)** that allows seamless connection between your VPC and the Internet, enabling secure and efficient communication. You also get a **route** in the main route table that points to all external traffic (`0.0.0.0/0`) to the internet gateway and internal traffic redirects to the VPC itself (local).
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1704813511095/35bc824b-8297-4c47-873e-deb7c5952c32.png align="center")
    
4. **Security Measures**: A default [**Security Group(SG)**](https://docs.aws.amazon.com/vpc/latest/userguide/default-security-group.html) and a default [**Network Access Control List (NACL)**](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html#default-network-acl) that ensure a secure perimeter for your resources.
    

### **Customize your default VPC**

If required, you can customize your Default VPC to align with your unique requirements which is often the case:

* Add additional IPv4 CIDR blocks (if you need more IP address space)
    
* Add nondefault subnets
    
* Modify Route Tables and add/remove routes
    
* Update rules in the default security group and/or add additional security groups, providing granular control over inbound and outbound traffic.
    
* Configure for AWS Site-to-Site VPN and Direct Connect Gateway
    
* Associate an IPv6 CIDR block
    

### **Access your default VPC and subnets**

Well, thanks for that! Now, I know what default VPC is, why is it needed, and the various components. But, hey there 👋 how and where can I see it? I got you!

1. **Navigate to the** [**AWS VPC Console**](https://console.aws.amazon.com/vpc/).
    
2. **Access "Your VPCs" in the sidebar**.
    
3. **Check the "Default VPC" column for a Yes value**.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1704814452161/4b9ba743-126b-4e0d-abb0-7b19c6106d32.png align="center")
    
4. Click on YOUR-VPC-ID and open **"Resource Map"** to check for the underlying **default subnets, route tables, and IGW.** Alternatively, you can navigate to Subnets and identify them from the "Default Subnets" columns
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1704814931881/40a378d5-6655-48bb-a3f5-40943a5773be.png align="center")
    

### **Creating Default VPC**

Whoo! This looks interesting. So, can I create one if I don't have one already? Well, certainly. Let's look at three different ways of achieving this.

1. **AWS Console**: Navigate to "**Your VPCs**," execute "**Actions,**" and select **"Create Default VPC.**" You are all set to go!
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1704815462149/8ef16ba7-a9b5-46da-a2af-68080b06cf19.png align="center")
    
2. **AWS CLI**: If you are more off of a CLI wizard, as you should be as a DevOps Engineer, you can use the following command to create a new default VPC.
    
    ```bash
    aws ec2 create-default-vpc
    ```
    
    <div data-node-type="callout">
    <div data-node-type="callout-emoji">💡</div>
    <div data-node-type="callout-text">Note that you would need to <a target="_blank" rel="noopener noreferrer nofollow" href="https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html" style="pointer-events: none">install AWS CLI</a> and authenticate to your account to run the command. Read the configuration guide <a target="_blank" rel="noopener noreferrer nofollow" href="https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html" style="pointer-events: none">here</a></div>
    </div>
    

**3\. Terraform:** If you do everything as IaC (Infrastructure as Code), you can create the same with the following [Terraform Resource Block](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/default_vpc).

```bash
resource "aws_default_vpc" "default" {
  tags = {
    Name = "Default VPC"
  }
}
```

### **Can I delete a default VPC?**

The short answer is **YES!** But be careful if that's what you want and only do so if you have custom VPC for your use cases. Otherwise, you might not be able to use many AWS services.

### **Can I make an existing VPC the default VPC or restore a deleted default VPC in Amazon VPC?**

Here, the answer is **NO!** You cannot convert your existing non-default VPC to your default VPC. You also cannot restore a previous default VPC that you deleted.

### Should I always use the default VPC?

Now this is the question you should be asking! While it's very easy to get started creating resources and get it up and running in the default VPC. Once you have multiple resources for different projects, especially in production, it is generally advised not to use default VPC.

This is because you can isolate your workloads on multiple dedicated VPC, say for instance your containers and your database in RDS can run on a separate custom VPC. This creates a logical separation and adds an extra layer of security. You also get much control over IP addresses and subnetting in custom VPCs. On top, most security audits will flag default VPCs, SGs, etc. Check out the **tfsec** flag details [here](https://aquasecurity.github.io/tfsec/v1.8.0/checks/aws/vpc/no-default-vpc/)

### **Conclusion**

AWS Default VPC comes with predefined IPv4 addresses, subnets, IGW and other components to help you quickly get started with AWS services. It abstracts the creation of a custom VPC and provides you with a ready-to-use VPC. However, for production use cases, it's advised to create your own custom VPC for specific workloads and additional security.

If you like my blog, do follow and check out my videos on YouTube where I publish videos on DevOps and Cloud. Here's one about Docker Bridge Network that explains how containers communicate with each other and to the internet:

%[https://www.youtube.com/watch?v=jCJAiOSuOkg] 

**Resources**

1. [https://docs.aws.amazon.com/vpc/latest/userguide/default-vpc.html](https://docs.aws.amazon.com/vpc/latest/userguide/default-vpc.html)
    
2. [https://docs.aws.amazon.com/vpc/latest/userguide/default-security-group.html](https://docs.aws.amazon.com/vpc/latest/userguide/default-security-group.html)
    
3. [https://aquasecurity.github.io/tfsec/v1.8.0/checks/aws/vpc/no-default-vpc/](https://aquasecurity.github.io/tfsec/v1.8.0/checks/aws/vpc/no-default-vpc/)
    
4. [https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/default\_vpc](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/default_vpc)