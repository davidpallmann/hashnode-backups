## Hello, .NET Deploy!

#### This episode: AWS Deployment Tool for .NET CLI. In this Hello, Cloud blog series, we're covering the basics of AWS cloud services for newcomers who are .NET developers. If you love C# but are new to AWS, or to this particular tool, this should give you a jumpstart.

In this post we'll introduce the AWS Deployment Tool for .NET and use it to deploy a "Hello, Cloud" .NET program to several different AWS services. We'll do this step-by-step, making no assumptions other than familiarity with C# and Visual Studio. We're using Visual Studio 2022 and .NET 6.

# AWS .NET Deployment Tool : What is it, and why use It?

> “If it hurts, do it more frequently, and bring the pain forward.” —Jez Humble, Continuous Delivery: Reliable Software Releases Through Build, Test, and Deployment Automation 

There are many kinds of .NET applications, and there are many AWS services capable of hosting them. With so many services and choices available, it can seem overwhelming. Where does a developer start? Which compute service should I use? How can I get started easily? What are good practices I and my team should follow? 

The AWS .NET Deployment tool helps you with a guided experience. It's what's known as opinionated tooling. The tool is for modern, cloud-native .NET applications that target running on Linux. You can't use it with legacy .NET framework or Windows applications. At present, the tool supports ASP .NET Core, .NET console, or Blazor WebAssembly applications written in .NET Core 3.1, .NET 5, or .NET 6. The tool is in preview, and is an open source project.

![diagram-targets.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650131463140/GQvrRbwnw.png)

You can run the tool from the command line with the .NET CLI, but it's also integrated into the AWS Toolkit for Visual Studio. As the tool guides you through deployment, it explains the logic behind its recommendations. At present, the AWS .NET deployment tool can guide you to target deployment on AWS App Runner, Amazon Elastic Container Service (ECS) with AWS Fargate, AWS Elastic Beanstalk, or S3 & CloudFront (for Blazor WebAssembly apps). In the future, the plan is for the tooling to merge with the .NET AWS Lambda experience.

Whether you use the .NET CLI or Visual Studio, the underlying .NET tool orchestrator handles the deployment. It integrates with the .NET CLI and Docker to build and containerize applications, and with the AWS Cloud Development Kit (CDK) and AWS CloudFormation to deploy to AWS.

![diagram-tool-components.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650122958563/dckLUc5pt.png)

There are already a number of IDE deployment wizards and command line ways to deploy services to AWS, so why use the AWS .NET Deployment tool? The tool provides a unified experience and a minimal number of steps. You don't need to know a lot about AWS to use it, but as you learn about advanced AWS features, the tool allows you to use them.  The interactive experience the tool provides can be reused in automated deployments. It's extensible and can be customized for your organization's needs.

# Our Hello, .NET Deployment Project

We will create a Blazor WebAssembly project and a Web API project. We'll use the AWS .NET deployment tool to deploy the Blazor project to S3 and CloudFront. Then we'll deploy the Web API project, first to AWS Elastic Beanstalk, and afterward to Amazon ECS.

![02-deploy-blazor-04-deploying.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650117209901/-za8ZHwKa.png)

![02-test-browser.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650117221683/jiOrQgGiR.png)

![03-browse-test-beanstalk.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650119813713/G2q8o-tjK.png)

![03-aws-beanstalk.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650119825831/aEmj9MB7T.png)

![04-aws-view-ecs.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650121239617/N2bcYyS4o.png)

[source code](https://github.com/davidpallmann/[link])

## One-time Setup

For any of the tutorials in the Hello, Cloud series you need the following:

1. An  [AWS account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/), and an understanding of what is included in the  [AWS Free Tier](https://aws.amazon.com/premiumsupport/knowledge-center/what-is-free-tier/). 
2. [Microsoft Visual Studio 2022](https://visualstudio.microsoft.com/). If you're using an older version of Visual Studio you won't be able to use .NET 6. If you use a different IDE, you may have to find alternatives to some tutorial steps.
3. [AWS Toolkit for Visual Studio](https://aws.amazon.com/visualstudio/). You'll need to [configure the toolkit ](https://docs.aws.amazon.com/toolkit-for-visual-studio/latest/user-guide/keys-profiles-credentials.html) to access your AWS account and create an IAM user. Your default AWS profile will be linked to this user when running programs from the command line.

## Step 1: Install the Tool

In this step, you'll install the AWS .NET Deployment Tool.

1. Open a command/terminal window.

2. Run the command below to install the tool.

```DOS
dotnet tool install -g aws.deploy.cli
```

If you receive a message that the tool is already installed, run the update command below to ensure you are on the latest version.

```DOS
dotnet tool update -g aws.deploy.cli
```

![01-install.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650110936847/Gcn91gLSU.png)

## Step 2: Deploy a Blazor WebAssembly Front-end Application to S3

In this step, you'll create a sample Blazor project and use the AWS .NET deployment tool to deploy it to Amazon S3 as a static web site.

1. Open a command/terminal window and CD to a development folder.

2. Run the dotnet new command below to create a new Blazor WebAssembly project named hello-net-deploy-blazor.

```DOS
dotnet new blazorwasm -n hello-net-deploy-blazor
```

![02-dotnet-new-blazorwasm.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650114020160/7G_i_PZ4J.png)

3. Open the project in Visual Studio and get familiar with the structure and code of the generated Hello, World project.

4. Press F5 to run the project. 

    A. The app opens in a browser tab. You see "Hello, World."

    ![02-run-hello-world.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650114242820/l9BhSZ-cE.png)

    B. On the left panel of the app, click **Counter**.

    C. Click the **Click me** button repeatedly, and observe the Current count increase.

    ![02-run-hello-world-counter.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650114320875/jpyK2r6NL.png)

    D. In Visual Studio, stop the program.

5. In Visual Studio, open the AWS Explorer and select a region you want to work in. We're using US-West-2 (Oregon).

6. In the command/terminal window, CD to the `hello-net-deploy-blazor` project location and set the AWS_REGION environment variable to the region you want to work in.

    On a Windows PC:

    ```DOS
set AWS_REGION=us-west-2
```

   On a Mac:

   ```DOS
export AWS_REGION=us-west-2
```

7. Run the command `dotnet aws deploy` to run the AWS .NET deployment tool. 

    ```DOS
dotnet aws deploy
```

    A. The deployment tool tells you the region it is using, detects the .NET project, and provides option(s) for deployment. In this case, there is one option offered for a Blazor WASM project, deployment to an S3 bucket. Read the description of what will be done, then enter **1**.

    ![02-deploy-blazor-01-choose-deployment.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650114857515/w97G4qBg6.png)

    B. Next you are prompted to name the project. Press **Enter** to accept the default of **hello-net-deploy-blazor**.

    ![02-deploy-blazor-02-set-project-name.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650114968341/_rauyzwXL.png)

    C. The tool now shows you the settings it proposes to use, which you may review or change. We see the index document is `index.html`, the simple project has no error page, the option to redirect on 403/404 errors is enabled, there is no back-end REST API, and we won't be logging access in CloudFront. All of these settings are fine for our purposes, so press **Enter**.

    ![02-deploy-blazor-03-settings.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650115188299/mzFrbtTOt.png)

    D. Now deployment begins, which takes a few minutes. The tool tells you it is doing a build, packaging up the app, and creating a CDK project. 

    ![02-deploy-blazor-04-deploying.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650115292409/kmAYe0R7M.png)

    Soon, deployment with CloudFormation begins. You see various CloudFormation "recipes" run, with details about S3 bucket creation and IAM policies and a CloudFront distribution.

    ![02-deploy-blazor-05-deploying-2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650115484568/OuUon5-4F.png)

    ![02-deploy-blazor-05-deploying-3.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650115753527/unJa5KmgS.png)

    E. Finally, the deployment completes. At the end of the display, you see the endpoint URL of your CloudFront distribution (`https://d8tpiw163opll.cloudfront.net/` for us), and the name of the S3 bucket that holds the website (`hello-net-deploy-blazor-recipecontents3buckete74b-anutmrgtivyf` for us).

    ![02-deploy-blazor-05-done.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650122241227/f00hN27xw.png)

8. Now it's time to test the the deployed app. 

     A. In a browser, visit the CloudFront URL. It works! Our app is deployed and available online, hosted in S3 and CloudFront.

     ![02-test-browser.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650116011726/_e1aFz5JZ.png)

    B. Click the Counter link and test that page as well.

    ![02-test-browser-2-counter.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650116124412/kJpoCwziV.png)

9. Visit the [AWS management console](https://aws.amazon.com/console) to review what was allocated.

    A. Set the region to the region you've been working in.

    B. Navigate to Amazon S3 (you can enter **s3** in the search bar). You should see an S3 bucket, with the same name displayed in Step 5E, along with another bucket for use by the CDK.

    ![02-aws-assets-s3.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650116780989/1utRP7MBs.png)

    C. Navigate to AWS CloudFront. You should see a CloudFront distribution, with the name that was displayed in Step 5E.

    ![02-aws-assets-cloudfront.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650116854167/Yaagw0vjv.png)

10. When you're done admiring your deployed project, let's delete it. We want to see how to delete a deployment with the AWS .NET deployment tool, and we don't want to leave things around that you might get charged for.

    A. In the command/terminal window, run the command below.

    ```DOS
dotnet aws delete-deployment hello-net-deploy-blazor
```
    B. When prompted for confirmation, enter **Y**.

    ![02-delete-deployment-1.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650117331955/K9t7b88oC.png)

     C. Wait for the delete deployment operation to complete.

    ![02-delete-deployment-2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650117764586/gadga3Z87.png)

     D. If you wish, revisit CloudFormation and S3 in the AWS console and confirm that the artifacts have been removed.

Congratulations! You've published a Blazor WebAssembly project to AWS S3 and CloudFront. The AWS NET deployment tool did **all** the work. Deleting the deployment was just as easy. 

## Step 3: Deploy a WebAPI Project to AWS Elastic Beanstalk

In this step, you'll create a WebAPI project and use the AWS .NET deployment tool to deploy it to Amazon Elastic Container Service (ECS) and AWS Fargate.

1. Open a command/terminal window and CD to a development folder.

2. Run the dotnet new command below to create a new Web API project named hello-net-deploy-webapi.

    ```DOS
dotnet new webapi -n hello-net-deploy-webapi
```

    ![03-dotnet-new-webapi.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650118079310/d-xx5dGRg.png)

3. CD to the `hello-net-deploy-webapi` folder, and build and test the project with the dotnet run command.

    ```DOS
dotnet run
```

    ![03-dotnet-run.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650118215265/rW6HnW9F7.png)

4. Note the displayed HTTPS URL and visit that URL in a browser, with **/WeatherForecast** at the end of the path. Our URL is `https://localhost:7272/WeatherForecast`. You see a JSON response with weather data.

    ![03-browser-test.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650118474525/HBs_NhEXC.png)

5. In the command/terminal window, stop the program.

6. Now we'll deploy our Web API project with the AWS .NET deployment tool. In the command/terminal window, run the command below.

    ```DOS
dotnet aws deploy
```

    A. This time, we have choices. The deployment tool offers us the option of deploying to 1) AWS Elastic Beanstalk on Linux, 2) Amazon ECS using AWS Fargate, or 3) AWS App Runner. Choose **1** to deploy to AWS Elastic Beanstalk.

    B. When prompted for project name, press Enter to accept the default of **hello-net-deploy-webapi**.

    ![03-dotnet-aws-deploy-1-select-beanstalk.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650119014028/l3Vpd5lK8.png)

    C. Review and accept the proposed settings by pressing Enter.

    D. Wait for the deployment to complete, and watch the console output to understand what is being done.

    ![03-dotnet-aws-deploy-2-settings.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650119122090/kKXT06Pn9.png)

7. When deployment completes, note the URL. Our is `http://hello-net-deploy-webapi-dev.eba-pxpss6an.us-west-2.elasticbeanstalk.com/`.

    ![03-dotnet-aws-deploy-3-done.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650122163996/QEqPDRIAA.png)

8. Visit the URL in a browser, with /WeatherForecast at the end of the path. Your Web API project is now hosted in AWS Elastic Beanstalk!

    ![03-browse-test-beanstalk.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650119599144/Zj9P9eqD-.png)

9. In the AWS console, visit the AWS Elastic Beanstalk area to view the Elastic Beanstalk allocation the deployment tool created.

    ![03-aws-beanstalk.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650119684497/uoNrHbAzd.png)

10. When you're done with the project, enter the command below to delete the deployment, and wait for the delete to complete.

    ```DOS
dotnet aws delete-deployment hello-net-deploy-webapi
```

    ![03-delete-deployment-beanstalk.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650120024839/SWvsRjR8s.png)

## Step 4: Deploy the Web API Project to Amazon Elastic Container Service (ECS)

In this step, you'll deploy the same Web API project you worked with in Step 3, this time to Amazon Elastic Container Service (ECS).

1. In a command/terminal window, CD to your Web API project folder.

2. Run the command below to again deploy the project.

   ```DOS
dotnet aws deploy
```

3. You should again be presented with 3 options just as in Step 3. This time, enter **2** to select deployment to Amazon ECS using Fargate.

4. When prompted for project name, accept the default of **hello-net-deploy-webapi**.

    ![04-dotnet-deploy-1.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650120452215/3NURtjdjP.png)

5. Review and accept the proposed settings.

     ![04-dotnet-deploy-2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650120492462/7bdlfXLlX.png)

6. Wait for deployment to complete, and note the URL. Ours is `http://hello-Recip-1EY9JEC5GXW7M-2106390135.us-west-2.elb.amazonaws.com/`.

    ![04-dotnet-deploy-3.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650121047940/oeMqmypCS.png)

7.  To test the app, visit the URL in a browser with **/WeatherForecast** at the end of the path. Your app is now hosted in Amazon ECS!

    ![04-test-browser-ecs.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650121130090/uwysnBUJw.png)

6. In the AWS console, navigate to Amazon ECS and view the ECS cluster that has been allocated.

    ![04-aws-view-ecs.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650121205885/Gfq21mcXn.png)

7. When you're done with the project, run the command below in the command/terminal window to delete the deployment. Wait for the delete operation to complete.

    ```DOS
dotnet aws delete-deployment hello-net-deploy-webapi
```

    ![04-delete.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650121489948/22hAenLEu.png)

You've now successful deployed your Web API project to a different compute service, Amazon ECS. 

# Where to Go From Here

The AWS .NET deployment tool gives you a new easy way to deploy. If you've struggled in the past with the setup, configuration, security roles, and options for deployment to AWS services, this guided tool should be a welcome addition. Even though it's doing the work for you, the tool is tremendously useful for learning how AWS works.

In this tutorial, you started with a Blazor WebAssembly front-end project. You used the AWS .NET deployment tool to deploy that to S3 and a CloudFront distribution with a simple command. You didn't have to supply complicated configuration or create security roles; it just worked. You deleted the deployment with another simple command. Next, you created a Web API project. You initially deployed to AWS Elastic Beanstalk. Then, we changed our minds and deployed to Amazon ECS instead. In all of this, we used the same commands and had a common experience.

If you were interested in what the deployment was doing, all you had to do was view the output which contained all the details. We did our deployments and deletions from the command line with the `dotnet aws` command, but we also could have done the same from Visual Studio. To go further, watch Norm Johanson's video demo and blog posts linked below and read the documentation.

# Further Reading

AWS Documentation

[AWS .NET deployment tool for the .NET CLI - Developer Guide](https://docs.aws.amazon.com/sdk-for-net/v3/developer-guide/deployment-tool.html)

[GitHub: AWS .NET deployment tool](https://github.com/aws/aws-dotnet-deploy#readme)

Videos

[AWS re:Invent 2021 - What’s new with .NET development and deployment on AWS](https://www.youtube.com/watch?v=UvTJ_Inb634)

Blogs

[Update on our new AWS .NET Deployment Experience](https://aws.amazon.com/blogs/developer/update-new-net-deployment-experience/)

[Deployment Projects with the new AWS .NET Deployment Experience](https://aws.amazon.com/blogs/developer/dotnet-deployment-projects/)

[Hello, Cloud blog series](https://davidpallmann.hashnode.dev/series/hello-cloud)