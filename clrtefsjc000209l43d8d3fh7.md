---
title: "How to implement CI/CD in AWS with AWS CodePipeline"
seoTitle: "Implement CI-CD in AWS"
seoDescription: "Build CI/CD Pipeline for your Java Tomcat Application with AWS Beanstalk and CodePipeline"
datePublished: Wed Jan 17 2024 18:15:00 GMT+0000 (Coordinated Universal Time)
cuid: clrtefsjc000209l43d8d3fh7
slug: how-to-implement-cicd-in-aws-with-aws-codepipeline
canonical: https://adex.ltd/streamlining-ci-cd-process-with-aws-codepipeline
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1706197341507/23e3bcf9-a375-4e3a-823f-c419847f7453.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1706198278621/118d4ded-a959-4b02-8639-1100bd2d8ef6.png
tags: aws, devops, ci-cd, aws-cicd, awscodepipeline

---

### **Introduction**

Continuous Integration/Continuous Deployment (CI/CD) is a set of software development practices that involves automating the process of integrating, testing, and deploying code changes to production. CI/CD pipelines have become the industry standard practice in modern software development, allowing organizations to deliver software faster and with higher quality.

**Continuous Integration (CI)** refers to the process of integrating code changes into a shared repository such as GitHub, building the application, and testing it automatically to ensure that the code is error-free and ready to be deployed. The aim is to catch any conflicts and errors early in the development phase.  
  
**Continuous Delivery (CD)** takes a step further and deploys these changes to a development or production environment. The goal here is to deliver the application to the end-user in the fastest manner and shorten the feedback loop.

### **The Benefits of CI/CD**

Here’s a closer look into the benefits of CI/CD and why they have become the industry standard process.

1. **Short release time:** Automating the process of integrating and deploying code changes allows for quicker and more frequent releases, enabling faster time-to-market and the ability to respond to user feedback and change requirements more efficiently.
    
2. **Early detection of issues:** By integrating and testing code changes regularly, CI/CD helps catch bugs, errors, and conflicts early in the development process, reducing the risk of issues reaching production.
    
3. **Increased collaboration:** CI/CD ensures collaboration among team members by encouraging regular integration of code changes and resolving conflicts early. This leads to better communication, coordination, and teamwork among developers.
    
4. **Improved software quality:** By automating testing and deployment, CI/CD helps ensure that only high-quality, thoroughly tested code reaches production, resulting in more stable, secure, and reliable software.
    
5. **Greater agility and innovation:** CI/CD enables teams to quickly adapt to changing requirements and market conditions, promoting innovation and the ability to continuously improve the software product.
    

### **Demo - AWS CodePipeline for a Java Web app - Tomcat**

There are several CI/CD tools available today such as Jenkins, GitHub Actions, GitLab CI/CD, CircleCI, Bamboo, Azure DevOps, etc each having their own strengths and weaknesses. In this blog, we will use AWS CodePipeline which integrates well with other AWS services to create the CI/CD pipeline for a sample Java web app served by Tomcat. Feel free to read further as the process is very similar for other web applications such as Node.Js, Django, .NET, and Laravel.

#### **Prerequisites**

Before we begin, make sure you have the following prerequisites:

1. An AWS account with appropriate permissions to create and configure CodePipeline, CodeBuild, and Elastic Beanstalk resources.
    
2. Source code for your application. You can use our sample application for demo purposes.
    
    <div data-node-type="callout">
    <div data-node-type="callout-emoji">💡</div>
    <div data-node-type="callout-text">GitHub Repo: <a target="_blank" rel="noopener noreferrer nofollow" href="https://github.com/adexltd/aws-ci-cd" style="pointer-events: none">GitHub - adexltd/aws-ci-cd: Sample Java Maven Application for AWS CI/CD Demo!</a></div>
    </div>
    

We will use the following AWS services to create the pipeline:

1. **AWS CodePipeline:** A fully managed CI/CD service that automates the building, testing, and deployment of applications.
    
2. **AWS CodeBuild:** A fully managed build service that compiles source code, runs tests, and produces software packages ready to deploy.
    
3. **AWS Elastic Beanstalk:** Elastic Beanstalk is a fully managed service that makes it easy to deploy, manage, and scale applications in the AWS Cloud. It will handle the process of provisioning and maintaining underlying required infrastructure such as EC2, S3, and ASG based on our application environment.
    

### **High-Level Architecture and Process Flow**

![CI-CD Process Overview](https://ds0xrsm6llh5h.cloudfront.net/blogs/image_65b8fb69-49f5-4e7d-87fb-53c583aa2ade_20240118065758.png align="left")

### **Flow Description**

As seen from the diagram above, the pipeline will be triggered automatically whenever there is a code change in the source repository. AWS CodeBuild will then take this source code and build the application with applied configurations. This generates an application package (artifact) and stores it in S3 Bucket. Finally, in the deployment phase, Elastic Beanstalk will pull the artifact from S3 and launch the application from the bundle in the pre-defined environment. Additionally, CloudWatch produces the logs from CodeBuild that we can use for monitoring the build process. That concludes our flow.

### **Implementation Guide**

**Step 1: Create an Elastic Beanstalk Environment**

1. Open the AWS Management Console, navigate to the Elastic Beanstalk service, and click the *"Create application"* button. This will launch a configuration wizard.
    
2. In the first step, choose the *"Web Server environment"* as the environment tier.
    
3. Now, you can select the appropriate options for our application, such as platform, platform version application code source, and presets. We will go will the *“Single Instance”* preset as it is free tier eligible.
    

![Elastic Beanstalk configuration](https://ds0xrsm6llh5h.cloudfront.net/blogs/image_4358fff2-ce65-4462-a0e0-92034129d035_20240118065817.png align="center")

![Elastic Beanstalk Configuration](https://ds0xrsm6llh5h.cloudfront.net/blogs/image_3f85d189-20ef-4776-a779-d6e30fdcabe6_20240118065934.png align="center")

```bash
For our sample application: we have the following settings:

Application name : ci-cd-demo
Environment name : Cicddemo-env
Platform: Tomcat 8.5 
Platform Version : 4.3.7
```

4\. Next, you can configure service access. Here, we will create a new service role. Optionally, you can add EC2 key pair to access EC2 servers deployed by Elastic Beanstalk.

![Elastic Beanstalk configuration](https://ds0xrsm6llh5h.cloudfront.net/blogs/image_5c495045-eeba-4d75-81de-a9eefc98841d_20240118070101.png align="center")

5\. Next, we will use the default options and select *“Skip to review”*. If you wish, you can set up networking (selecting custom VPCs, assigning Public IP to our EC2 instances), and create databases and tags. You can also choose the capacity of Auto Scaling Group, and configure monitoring and logging through CloudWatch metrics.

6\. Review the configurations and click the *"Submit"* button to create the Elastic Beanstalk environment.

#### **Step 2: Create a buildspec.yml file for CodeBuild**

The *buildspec.yml* file is a configuration file that defines the build steps for your application and is used by CodeBuild to execute the build process (we will see the use of CodeBuild later). In your source code repository, create a buildspec.yml file at the root level of your project directory. This file contains the build steps for your application, such as building and testing your code.

Here is a **buildspec.yml** file for our sample application:

```bash
version: 0.2
phases:
  pre_build:
    commands:
    - echo "Pre-Build Phase"
  build:
    commands:
      - echo "Build Phase Started"
      - mvn clean package
  post_build:
    commands:
      - echo "Build Succeded"
artifacts:
  files:
    - target/aws-ci-cd*/*
  discard-paths: yes
```

This **buildspec.yml** file specifies three build phases: *build, test,* and *post\_build*. If you are using another web framework such as Node.js, you can install your dependencies in the *pre\_build* stage.

Here, we build our maven package in the build stage which creates the build artifact in the target/aws-ci-cd directory. The `artifacts` section of the build specification file provides details about where to store the artifacts and the format in which they should be stored.

By default, CodeBuild stores the build artifacts in an S3 bucket created and managed by CodeBuild. The S3 bucket is named with a prefix `codepipeline-*` followed by a unique identifier for the CodeBuild project.

You can also configure the build project to use a custom S3 bucket for storing the build artifacts. In our case, `target/aws-ci-cd*/*` pattern will be used to include files from the `target` directory that match the `aws-ci-cd*/*` pattern. Once the build process is complete, the resulting artifact files will be uploaded to the S3 bucket created by CodeBuild.

![Build Artifact stored in S3](https://ds0xrsm6llh5h.cloudfront.net/blogs/image_29d2f1de-62f5-4b45-8e90-e9e088da5d34_20240118070125.png align="center")

We can customize the buildspec.yml file based on the requirements of the application, such as adding additional build steps, tests, or deployment instructions. Make sure to commit and push the buildspec.yml file to your source code repository if you have not already.

**Step 3: Set up a CodePipeline**

Now, let’s set up a code pipeline that glues together everything we have done so far and deploy our application to ElasticBeanStalk

1. Open the AWS Management Console, navigate to the AWS CodePipeline service, and click the *"Create pipeline"* button.
    
2. Enter a pipeline name, and create a new service role.
    
3. Next, in the Source Stage, select your source provider (GitHub v2), and choose the repository and branch that you want to use for your application source code.
    
4. You might need to create a connection to GitHub if you are doing this for the first time. Give a connection name and click on “Install a new app” that will install AWS Connector for GitHub. Once connected, make sure the change detection option is checked to trigger the pipeline from the source code change.
    

![](https://ds0xrsm6llh5h.cloudfront.net/blogs/image_b9bf7582-d059-4ce9-8836-3428ef1db68b_20240118070145.png align="left")

![GitHub Connection](https://ds0xrsm6llh5h.cloudfront.net/blogs/image_10847827-7ab0-4113-998e-82e75753bc44_20240118070158.png align="center")

![Code Pipeline- Source Configuration](https://ds0xrsm6llh5h.cloudfront.net/blogs/image_aa442aa8-f8d0-4580-b59c-f2c05ffe1375_20240118070209.png align="center")

5\. Next, in the **Build Stage** select AWS CodeBuild as your build provider, and select *“Create Project”* option. Here, we will configure a CodeBuild project that will compile the source code, run tests (if present), and produce a software package that is ready to deploy.

![Code Pipeline- Build Stage Configuration](https://ds0xrsm6llh5h.cloudfront.net/blogs/image_7b01a4d1-7323-4b80-a932-75b3946a80c8_20240118070226.png align="center")

* **(Build Stage Continued)**: Enter the project name and choose a runtime environment for your build, such as Node.js, Java, or Python, and specify the build configurations. We will choose the following configurations.
    

![Code Pipeline- Build Stage Configuration](https://ds0xrsm6llh5h.cloudfront.net/blogs/image_0116aa0a-e278-46cd-8280-0ce9a1f030be_20240118070237.png align="center")

* **(Build Stage Continued)** : Remember, we created buildspec.yaml earlier? It comes into use here. We will choose the *“Use a buildspec file”* option which in turn will look for buildspec.yaml in our repository root by default. Make sure to rename it if you have any other name for the build spec configs.
    
* Optionally, you can configure additional build options such as webhook triggers, CloudWatch Logs, and S3 Logs. Review your project configuration, add environment variables (if any), select “Buid type” as Single build, and click the "Continue to CodePipeline".
    

5\. In the **Deploy Stage**, select *AWS Elastic Beanstalk* as your deployment provider, and choose the environment which we created in **Step 1.**

![Deploy Configuration](https://ds0xrsm6llh5h.cloudfront.net/blogs/image_865cc7a1-7003-45e9-bd29-7b8fa027e9ae_20240118070317.png align="left")

6\. Finally, review your pipeline configuration and click the *"Create pipeline"* button to create your pipeline.

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text"><strong>Optional:</strong> We can create multiple environments in Beanstalk, for instance, separate production and dev environments. To quickly simulate these, Go to Elastic Beanstalk&gt;Environment&gt;Cicddemo-env&gt;Actions&gt;Clone environment. We will name it as Cicddemo-prod. After this go to your pipeline, click on ‘Edit’, and follow the instructions to Add Stage. (This may differ as per your project requirement and it’s fine to just use one deployment environment)</div>
</div>

**Step 4: Test the CI/CD pipeline**

Now that we have set up the entire CI/CD pipeline, it's time to test it by making changes to your application source code and triggering a pipeline run.

1. Make changes to your application source code, such as fixing a bug, adding a new feature, or updating a configuration file. Here, I will just change the welcome message in */src/main/webapp/index.jsp* to **“Welcome to CI/CD!”**
    
2. Commit and push the changes to GitHub or your source repository. This will trigger the pipeline to run automatically.
    
3. ![CodePipeline - Process](https://ds0xrsm6llh5h.cloudfront.net/blogs/image_d32c34f2-34c3-40d1-b711-7ee0676f4797_20240118070359.png align="center")
    
    Open the AWS CodePipeline service, and navigate to your pipeline to see the status of different stages. Here, we have used two environments for deployment. You might only see one if you have not created a new Beanstalk environment and added it to the pipeline.
    
4. CodePipeline will automatically start the build process in CodeBuild, which will compile our source code, run tests, and produce a deployment package. Once the build is successful, CodePipeline will automatically deploy the application to Elastic Beanstalk according to the deployment settings.
    
5. Monitor the pipeline run in the CodePipeline console, and check the build details from CodeBuild or Elastic Beanstalk environment events and logs for any errors or issues.
    
6. ![Elastic Beanstalk Environment](https://ds0xrsm6llh5h.cloudfront.net/blogs/image_975b4af5-6882-4e40-ae0e-a67370ef4cca_20240118070429.png align="left")
    
    Once the deployment is complete, we can access the application on the Elastic Beanstalk environment URL to verify that the changes have been successfully deployed.
    

![Sample App Message](https://ds0xrsm6llh5h.cloudfront.net/blogs/image_036642c1-0c82-44a1-8058-32abe07f7814_20240118070556.png align="center")

### **Conclusion**

Implementing a CI/CD pipeline is a crucial step in modern software development practices to ensure efficient and reliable application delivery. We explored AWS’s powerful tools like CodePipeline, CodeBuild, and Elastic Beanstalk which can be easily integrated to set up a robust CI/CD pipeline on the cloud. By following these steps, you can easily automate the process of building, testing, and deploying your applications, saving time and ensuring consistent quality in your software releases.

### **Resources**

AWS CodePipeline: [https://aws.amazon.com/codepipeline/](https://aws.amazon.com/codepipeline/)

CodeBuild buildspec specification: [Build specification reference for CodeBuild - AWS CodeBuild](https://docs.aws.amazon.com/codebuild/latest/userguide/build-spec-ref.html)

Official Hands-on: [https://aws.amazon.com/getting-started/hands-on/continuous-deployment-pipeline/](https://aws.amazon.com/getting-started/hands-on/continuous-deployment-pipeline/)