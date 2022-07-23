## Hello, CDK!

#### This episode: AWS Cloud Development Kit and Infrastructure-as-Code. In this Hello, Cloud blog series, we're covering the basics of AWS cloud services for newcomers who are .NET developers. If you love C# but are new to AWS, or to this particular service, this should give you a jumpstart.

In this post we'll introduce Amazon CDK and write AWS infrastructure-as-code in C#. We'll do this step-by-step, making no assumptions other than familiarity with C# and Visual Studio. We're using Visual Studio 2022 and .NET 6.

# Amazon CDK : What is it, and why use It?

> “CDK has been a gamechanger for us. It has drastically improved our feedback cycle and reduced the time it takes to go from brand-new to fully deployed infrastructure."
—Tyler van Hensbergen, Software Engineer, Stedi

Cloud applications require infrastructure, and configuring it manually can be cumbersome, time-consuming, or inconsistent. Infrastructure as code (IaC) provides a refreshing alternative: automated infrastructure management through code. What kind of code? AWS's infrastructure automation platform is [AWS CloudFormation](https://aws.amazon.com/cloudformation/), in which you model your entire cloud environment in JSON or YAML files. That's terrific, but as a developer you might yearn for a way to model your environment in your preferred programming language, such as C#.

[AWS Cloud Development Kit](https://aws.amazon.com/cdk/) (hereafter "CDK") is a software development framework that allows you to write Infrastructure-as-Code (IaC) in a programming language. AWS describes it as "an open-source software development framework to define your cloud application resources using familiar programming languages". C# is a supported programming language, along with JavaScript, TypeScript, Python, Java, and Go.

With CDK, you work in your familiar programming language to define your cloud infrastructure, and CloudFormation is generated for you. For a .NET developer, this means your IaC and application can all be written in the same programming language and reside in the same solution. That's highly advantageous for productivity, and does away with the need to switch to a different language or development environment for IaC. 

![diagram.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1656881716719/TNgsndCo2.png align="left")

In a CDK project, you compose a **stack** from high-level components called **constructs**.  For C# CDK development, you use a NuGet package. You use the CDK CLI to manage CDK deployments, which generates and deploys CloudFormation. CDK is built on Node.js and npm, which you must install to use CDK. 

A large collection of open source CDK libraries, or "constructs", are available on [Construct Hub](https://constructs.dev/). There are [over 600 .NET constructs](https://constructs.dev/search?langs=dotnet&sort=downloadsDesc&offset=0) available.

The AWS CDK CLI is how you invoke CDK operations, such as deploying or redeploying your stack, or destroying an existing stack. Other AWS CLI tools also make use of CDK, including the AWS Deploy tool for .NET CLI and AWS Copilot.

# Our Hello, CDK Project

We will develop, in a single C# solution, two AWS Lambda functions and a CDK project. The CDK project will define infrastructure for the Lambda functions and Amazon API Gateway endpoints. We'll deploy and later destroy a stack using the CDK CLI.

![cdk-redeploy-3.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1656883058376/y31_u1ckw.png align="left")

![test-browser-date.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1656883081574/rR_NkH6SD.png align="left")

![test-browser-time.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1656883090448/QBG7VaJxr.png align="left")

[source code](https://github.com/davidpallmann/hello-cdk)

## One-time Setup

For any of the tutorials in the Hello, Cloud series you need the following:

1. An  [AWS account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/), and an understanding of what is included in the  [AWS Free Tier](https://aws.amazon.com/premiumsupport/knowledge-center/what-is-free-tier/). 
2. [Microsoft Visual Studio 2022](https://visualstudio.microsoft.com/). If you're using an older version of Visual Studio you won't be able to use .NET 6. If you use a different IDE, you may have to find alternatives to some tutorial steps.
3. [AWS Toolkit for Visual Studio](https://aws.amazon.com/visualstudio/). You'll need to [configure the toolkit ](https://docs.aws.amazon.com/toolkit-for-visual-studio/latest/user-guide/keys-profiles-credentials.html) to access your AWS account and create an IAM user. Your default AWS profile will be linked to this user when running programs from the command line.

## Step 1: Install CDK

In this step, you'll install CDK and its prerequisites.

1. Open a command/terminal or PowerShell window.

2. Install [node.js](https://nodejs.org/en/) for your operating system, which will also install Node Package Manager (npm). You can check whether you already have npm with the command `npm --version`.

    ![npm-version.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1656883268995/-tUFucseM.png align="left")

3. Install the AWS CLI. You can check whether you already have the AWS CLI with the command `aws --version`. 

    ![aws-version.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1656883343449/gfHhezbxh.png align="left")

    If you don't have it, install it and configure it:

    A. install the AWS CLI from [https://aws.amazon.com/cli/](https://aws.amazon.com/cli/). 

    B. Follow these instructions to [configure the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html).

4. Set your your default region by running the command `aws configure`. We're using **us-east-1 (N. Virginia)** in this tutorial.

    ```dos
aws configure
```

5. Use the npm command below to install the AWS CDK CLI.

    ```dos
npm install -g aws-cdk
```

    ![npm-cdk.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1656769370607/iemgDjnqd.png align="left")

6. Run the `cdk` with no parameters to see help, and take note of the actions available to you. 

    ![cdk-help.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1656769494048/QO-TkM5ff.png align="left")

7. Now that prerequisites and the CDK CLI are installed, close your command/terminal window and open a new one.

## Step 2: Create a CDK Project

In this step, you'll create a CDK project for the AWS Lambda function.

1. In a command/terminal window, create a new development folder named `hello-cdk` and CD to it.

    ![md-hello-cdk.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1656769787001/lO0gzbk5u.png align="left")

2. Run the `cdk init` command below to create the scaffolding for a C# CDK project.

    ```dos
cdk init app --language csharp
```

    ![cdk-init.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1656770254446/_AADUrWz1.png align="left")

    Your `hello-cdk` folder now contains a generated C# solution, in the `src` subfolder.

    ![folder-generated-project.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1656770395781/DeoE_h1L9.png align="left")

3. Open the solution in Visual Studio or your preferred IDE.

4. In Solution Explorer, right-click the **HelloCdk** project and select **Properties**. Set the Target framework to **.NET 6**.

5. An important file used by the CDK, but not formally part of the project, is cdk.json in the top-most hello-cdk folder. Let's add it to the project so we can easily work with it in Visual Studio:

    A. In Solution Explorer, right-click the `HelloCdk` project and select **Add > Existing Item**. 

    B. In the Add dialog, select **All Files**.

    C. Browse to and select the cdk.json file in your top-most `hello-cdk` folder. 

    D. The `cdk.json` file is added to your project. Open it in the code editor to get a look at it. From this file, the CDK command will know how to run the CDK project, the files to include/exclude, and which features to enable.

    ```json
{
  "app": "dotnet run --project src/HelloCdk/HelloCdk.csproj",
  "watch": {
    "include": [
      "**"
    ],
    "exclude": [
      "README.md",
      "cdk*.json",
      "src/*/obj",
      "src/*/bin",
      "src/*.sln",
      "src/*/GlobalSuppressions.cs",
      "src/*/*.csproj"
    ]
  },
  "context": {
    "@aws-cdk/aws-apigateway:usagePlanKeyOrderInsensitiveId": true,
    "@aws-cdk/core:stackRelativeExports": true,
    "@aws-cdk/aws-rds:lowercaseDbIdentifier": true,
    "@aws-cdk/aws-lambda:recognizeVersionProps": true,
    "@aws-cdk/aws-lambda:recognizeLayerVersion": true,
    "@aws-cdk/aws-cloudfront:defaultSecurityPolicyTLSv1.2_2021": true,
    "@aws-cdk-containers/ecs-service-extensions:enableDefaultLogDriver": true,
    "@aws-cdk/aws-ec2:uniqueImdsv2TemplateName": true,
    "@aws-cdk/core:checkSecretUsage": true,
    "@aws-cdk/aws-iam:minimizePolicies": true,
    "@aws-cdk/core:validateSnapshotRemovalPolicy": true,
    "@aws-cdk/aws-codepipeline:crossAccountKeyAliasStackSafeResourceName": true,
    "@aws-cdk/core:target-partitions": [
      "aws",
      "aws-cn"
    ]
  }
}
```

## Step 3: Add a Date AWS Lambda Function

In this step, you'll add a serverless project with a Lambda function to the solution that returns the date.

1. In Solution Explorer, right-click the `HelloCdk` solution and select **Add > New Project**.

2. In the Add a new project wizard, choose project type **Serverless**, then select the **AWS Serverless Application (.NET Core - C#)** project template and click **Next**.

    ![vs-add-project-serverless.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1656790772628/63Ba7gBRy.png align="left")

3. On the Configure your project page, set the Project name to **DateFunction**.

4. On the Select Blueprint wizard, select **Empty Serverless Application** and click **Finish**.

    ![vs-add-project-blueprint.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1656790911781/zx_IMyEy7.png align="left")

5. Open the `DateFunction` project's `Function.cs` file in the code editor, and replace the code with the code below. We are changing two things from the generated code: the namespace is now `DateFunction`, and the function handler sets the response body to a date string reflecting the current date.

DateFunction - Function.cs

```csharp
using System.Net;
using Amazon.Lambda.Core;
using Amazon.Lambda.APIGatewayEvents;

// Assembly attribute to enable the Lambda function's JSON input to be converted into a .NET class.
[assembly: LambdaSerializer(typeof(Amazon.Lambda.Serialization.SystemTextJson.DefaultLambdaJsonSerializer))]

namespace DateFunction;

public class Functions
{
    /// <summary>
    /// Default constructor that Lambda will invoke.
    /// </summary>
    public Functions()
    {
    }


    /// <summary>
    /// A Lambda function to respond to HTTP Get methods from API Gateway
    /// </summary>
    /// <param name="request"></param>
    /// <returns>The API Gateway response.</returns>
    public APIGatewayProxyResponse Get(APIGatewayProxyRequest request, ILambdaContext context)
    {
        context.Logger.LogInformation("Get Request\n");

        var response = new APIGatewayProxyResponse
        {
            StatusCode = (int)HttpStatusCode.OK,
            Body = $"{DateTime.Today.ToLongDateString()}",
            Headers = new Dictionary<string, string> { { "Content-Type", "text/plain" } }
        };

        return response;
    }
}
```

## Step 4: Add a Time AWS Lambda Function

In this step, you'll add another serverless project to the solution, with a Lambda function that returns the time. You'll follow the same steps you did for Step 3.

1. In Solution Explorer, right-click the `HelloCdk` solution and select **Add > New Project**.

2. In the Add a new project wizard, choose project type **Serverless**, then select the **AWS Serverless Application (.NET Core - C#)** project template and click **Next**.

3. On the Configure your project page, set the Project name to **TimeFunction** and click **Create**.

4. On the Select Blueprint wizard, select **Empty Serverless Application** and click **Finish**.

5. Open the `TimeFunction` project's `Function.cs` file in the code editor, and replace the code with the code below. This function handler sets the response body to a time string reflecting the current time.

6. Save your changes and build the solution.

TimeFunction - Function.cs

```csharp
using System.Net;
using Amazon.Lambda.Core;
using Amazon.Lambda.APIGatewayEvents;

// Assembly attribute to enable the Lambda function's JSON input to be converted into a .NET class.
[assembly: LambdaSerializer(typeof(Amazon.Lambda.Serialization.SystemTextJson.DefaultLambdaJsonSerializer))]

namespace TimeFunction;

public class Functions
{
    /// <summary>
    /// Default constructor that Lambda will invoke.
    /// </summary>
    public Functions()
    {
    }


    /// <summary>
    /// A Lambda function to respond to HTTP Get methods from API Gateway
    /// </summary>
    /// <param name="request"></param>
    /// <returns>The API Gateway response.</returns>
    public APIGatewayProxyResponse Get(APIGatewayProxyRequest request, ILambdaContext context)
    {
        context.Logger.LogInformation("Get Request\n");

        var response = new APIGatewayProxyResponse
        {
            StatusCode = (int)HttpStatusCode.OK,
            Body = $"{DateTime.Now.ToLongTimeString()}",
            Headers = new Dictionary<string, string> { { "Content-Type", "text/plain" } }
        };

        return response;
    }
}
```

## Step 5: Add the Lambda functions to the CDK project

In this step, you'll add the two Lambda functions to the CDK project.

1. From Solution Explorer, open `HelloCdkStack.cs` in the code editor.

2. Add a using statement at top for the Amazon.CDK.AWS.Lambda namespace:

    ```csharp
using Amazon.CDK.AWS.Lambda;
```

3. In the `HelloCdkStack` function body, add the code below to create a CDK Lambda function. The parameters to the `Amazon.CDK.AWS.Lambda.Function` specify this class instance, the CDK name `dateLambda`, and function properties. The function properties specify the .NET 6.0 runtime, the location where published code can be found, and the Lambda function handler name, which must be of the form project::type::method.

    ```csharp
            var dateLambda = new Amazon.CDK.AWS.Lambda.Function(this, "dateLambda", new FunctionProps
            {
                Runtime = Runtime.DOTNET_6,
                Code = Code.FromAsset("src/DateFunction/bin/Release/net6.0/linux-x64/publish"),
                Handler = "DateFunction::DateFunction.Functions::Get"
            });
```

4. Below that, add a similar statement with the code below for the TimeFunction Lambda function. 

    ```csharp
            var timeLambda = new Amazon.CDK.AWS.Lambda.Function(this, "timeLambda", new FunctionProps
            {
                Runtime = Runtime.DOTNET_6,
                Code = Code.FromAsset("src/TimeFunction/bin/Release/net6.0/linux-x64/publish"),
                Handler = "TimeFunction::TimeFunction.Functions::Get"
            });
```

Your HelloCdkStack.cs code should now look like this:

```csharp
using Amazon.CDK;
using Amazon.CDK.AWS.Lambda;
using Constructs;

namespace HelloCdk
{
    public class HelloCdkStack : Stack
    {
        internal HelloCdkStack(Construct scope, string id, IStackProps props = null) : base(scope, id, props)
        {
            var dateLambda = new Amazon.CDK.AWS.Lambda.Function(this, "dateLambda", new FunctionProps
            {
                Runtime = Runtime.DOTNET_6,
                Code = Code.FromAsset("src/DateFunction/bin/Release/net6.0/linux-x64/publish"),
                Handler = "DateFunction::DateFunction.Functions::Get"
            });

            var timeLambda = new Amazon.CDK.AWS.Lambda.Function(this, "timeLambda", new FunctionProps
            {
                Runtime = Runtime.DOTNET_6,
                Code = Code.FromAsset("src/TimeFunction/bin/Release/net6.0/linux-x64/publish"),
                Handler = "TimeFunction::TimeFunction.Functions::Get"
            });

        }
    }
}
```

## Step 6: Publish the Solution and Bootstrap CDK

In this step, you'll use the `dotnet publish` command to build the solution and create the publish files.

1. Open a command/terminal windows and CD to the to `hello-cdk` project folder, where `cdk.json` resides.

2. Enter the command below to publish the .NET solution. This will create runtime files for the Lambda functions where the CDK project will be looking for them.

    ```dos
dotnet publish src -c Release -r linux-x64 -p:PublishReadyToRun=false --self-contained
```

    ![dotnet-publish-src.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1656799601655/tU8qv5I2j.png align="left")

3. Next we'll bootstrap the CDK with the `cdk bootstrap` command. This is something you only need to run once, for a region. Bootstrapping the CDK will install resources in your AWS environment necessary for CDK operations. Run the command below.

   ```dos
cdk bootstrap
```

    ![cdk-bootstrap.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1656798753039/43SXHie_c.png align="left")

## Step 7: Deploy with CDK

Now we're ready for the CDK command to do its magic. In this step, you'll use the CDK CLI to execute your CDK project, creating CloudFormation and deploying your two Lambda functions.

1. In a command/terminal window, CD to the project folder.

2. Run the command below to deploy:

    ```dos
cdk deploy
```

3. The CDK command shows you security changes it plans to make, new IAM roles needed for the two Lambda functions. Respond with **y** to proceed.

    ![cdk-deploy-1.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1656798878673/IgAUZijh2.png align="left")

4. Now wait for the CDK to deploy, which may take a few minutes. The output is very detailed, and that's so you know what's being done and have the information you need to correct any errors that may occur. 

    If you do get an error, review the error message(s) to understand the nature of the problem. Double check you did all of the prior steps properly, and that your path names are correct and match what's on your machine.

    If deployment completes without errors, the end of the output will look like this:

    ![cdk-deploy-2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1656850909440/nGgT0m9XD.png align="left")

5. Let's verify the deployment really took place. In a browser, open the AWS management console, and set the region at top right to the region you deployed to. Then navigate to **AWS Lambda** and select **Functions** on the left pane. You should see your two Lambda functions.

    ![aws-lambdas-view.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1656800177052/9pOG2ltLn.png align="left")

Congratulations, you've successfully deployed two Lambdas to AWS using a C# CDK project and the CDK CLI! However, we can't access them yet. Let's add API Gateway to the mix so we can get to our Lambda functions from a browser.

## Step 8: Add an API Gateway

In this step, you'll add an API Gateway to your CDK project and again deploy it. Then you'll test accessing your API and your Lambda functions from a browser.

1. In Visual Studio, open `HelloCdkStack.cs` in the code editor.

2. Add a using statement at top for the Amazon.CDK.AWS.APIGateway namespace:

    ```csharp
using Amazon.CDK.AWS.APIGateway;
```

3. In the constructor, insert two statements to define API gateway endpoints for the Lambda functions.

    A. Insert this statement below the `var dateLambda =...` statement. 

    ```csharp
            new LambdaRestApi(this, "dateApiEndpoint", new LambdaRestApiProps
            {
                Handler = dateLambda
            });
```

    B. Insert this statement below the `var timeLambda =...` statement. 

    ```csharp
            new LambdaRestApi(this, "timeApiEndpoint", new LambdaRestApiProps
            {
                Handler = timeLambda
            });
```

4. Save your changes.

5. In the command/terminal window, again publish the .NET solution to generate runtime files:

    ```dos
dotnet publish src -c Release -r linux-x64 -p:PublishReadyToRun=false --self-contained
```

6. Redeploy by running `cdk deploy`. Review the changes and confirm with **y**.

    ```dos
cdk deploy
```

    ![cdk-redeploy-1.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1656801433188/EXt6XxxrS.png align="left")

    ![cdk-redeploy-2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1656801474322/wScyxgt4D.png align="left")

7. Once again, wait for the deployment to complete, and troubleshoot any errors. Dozens of more steps were added for the API Gateway, but in code you merely added two statements to make all that happen.

    On successful completion, you will see API Gateway endpoints for your two Lambda functions. Record the endpoint URLs.

    ![cdk-redeploy-3.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1656850835752/huGAgZacw.png align="left")

8. In the AWS management console, navigate to **Amazon API Gateway** and you should see that your API Gateway endpoints have been deployed.

    ![aws-gateway-endpoints.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1656850523865/EDRNxN50L.png align="left")

9. Visit your Date endpoint in a browser, and you should see the date displayed. Visit your Time endpoint in a browser, and you should see the time displayed (local to the cloud instance).

    ![test-browser-date.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1656850693338/o0UQOfDAZ.png align="left")

    ![test-browser-time.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1656850746611/AglIwVgWG.png align="left")

    You can also use tools like curl, iwr (in PowerShell), Postman, or Fiddler to test and inspect responses to your API Gateway endpoints.

    ![test-curl-date.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1656851228507/5CVVaOtGD.png align="left")

Nice! You've successfully added API Gateway endpoints for your two Lambda functions and re-deployed with CDK.

10. In case you're curious about the CloudFormation that CDK is generating, look at your development folder structure and you'll see a folder named `cdk.out`. This contains the generated CloudFormation files. Take a peek, and I think you'll agree that working in C# in a CDK project with constructs is far simpler than writing lengthy JSON yourself.

![folder-generated.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1656884445668/Ti6J-iJzt.png align="left")

## Step 9: Delete Your Deployment

When you're done working with your Hello, CDK project, follow the steps below to delete the deployment. You don't want to accrue charges for something you're not using.

1. Open a command/terminal windows and CD to your `hello-cdk` project folder.

2. Run the `cdk destroy` command and confirm the *Are you sure* prompt with **y**.

    ![cdk-destory-1.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1656807716525/EZ4t2IYG-.png align="left")

3. Wait for the CDK destroy operation to complete.

    ![cdk-destroy-2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1656808071361/wdArkWp3d.png align="left")

4. In the AWS management console, navigate to **AWS Gateway** and confirm the API Gateway has been removed.

5. Navigate to **AWS Lambda** and confirm the two Lambda functions have been removed.

# Where to Go From Here

Infrastructure-as-Code is powerful and necessary for streamlined, reliable cloud development. It's well worth making a part of your cloud development. With AWS Cloud Development Kit, you can work on IaC right alongside your application code in the same solution, the same programming language, and the same IDE.

In this tutorial, you installed dependencies and the CDK CLI, generated a CDK C# project, and created two Lambda functions. You added C# code to the CDK project for your two Lambda functions and API Gateway endpoints. The code was short and simple. You used the CDK CLI to deploy and redeploy your software. You saw how simple the CDK project was, just a handful of statements.

The AWS CDK allowed you to work in familiar C#, and had the building blocks to make defining Lambda functions and API Gateway endpoints very simple. The CDK CLI gave you plenty of detailed information, helpful both for learning and troubleshooting. The CDK CLI generated and executed CloudFormation, but you didn't have to work with it directly.

IaC can seem intimidating at times, and it can get complex. If you have a good understanding of the AWS services you use and their configuration options, you'll write better IaC. If you get familiar with the CDK .NET constructs and object model, the CDK code will come more readily.

This tutorial only covered a small amount of surface area, Amazon API Gateway and AWS Lambda. You'll want to learn how to use CDK in your CI/CD pipelines, leverage environment variables, and deploy across multiple environments such as test, staging, and production. To go further, read the documentation, take tutorials, and practice using CDK for different things. I got my start with CDK watching the PJ Pittle video linked below. Remember, before writing your own CDK code, check Construct Hub for existing constructs that may do what you need.

# Further Reading

AWS Documentation

[AWS Cloud Development Kit](https://aws.amazon.com/cdk/)

[AWS Cloud Development Kit](https://github.com/aws/aws-cdk) on GitHub

[AWS CDK Toolkit Developer Guide](https://docs.aws.amazon.com/cdk/v2/guide/index.html)

[AWS CDK .NET Documentation](https://docs.aws.amazon.com/cdk/api/v2/dotnet/api/index.html)

[Working with the AWS CDK in C#](https://docs.aws.amazon.com/cdk/v2/guide/work-with-cdk-csharp.html)

[AWS CDK Examples](https://github.com/aws-samples/aws-cdk-examples/) on GitHub

[.NET Workshop AWS Cloud Development Kit (AWS CDK)](https://cdkworkshop.com/40-dotnet.html)

Dependencies and Installations

[Node.js and Node Package Manager (npm)](https://nodejs.org/en/download/) download

Install AWS CDK CLI: [npm install -g aws-cdk](npm install -g aws-cdk)

Community Resources

[cdk.dev](https://cdk.dev/) CDK Community

[ConstructHub](https://constructs.dev/) open source CDK libraries

[AWS SDK for .NET Documentation](https://docs.aws.amazon.com/sdk-for-net/index.html)

Videos

*Note: AWS CDK is at v2 at the time of this writing. Some of these videos and blogs may reference CDK v1*.

[How to use C# with the AWS CDK to manage your Infrastructure as Code](https://www.youtube.com/watch?v=5aBf0W0_FDY) by PJ Pittle

[Build Your Infrastructure in .NET with AWS Cloud Development Kit (CDK) - AWS Online Tech Talks](https://www.youtube.com/watch?v=UGvkVdjk6vY) by Martin Beeby

[AWS Cloud Development Kit (CDK) with the .NET](https://www.youtube.com/watch?v=i6xJuAiWQLw) by Taz Hussein

[AWS re:Invent 2019: Infrastructure as .NET with AWS CDK](https://www.youtube.com/watch?v=cxaFQMWMs7g) by Nikki Stone and Steve Roberts

Blogs

[The AWS CDK using C#](https://graham-beer.github.io/2020/aws-cdk-csharp-8/) by Graham Beer

[AWS CDK for .NET](https://aws.amazon.com/blogs/developer/aws-cdk-for-net/) by Aaron Costley

[Deployment Projects with the new AWS .NET Deployment Experience](https://aws.amazon.com/blogs/developer/dotnet-deployment-projects/)

[4 ways to deploy a .NET Core Lambda using AWS CDK](https://www.mytechramblings.com/posts/deploy-dotnet-lambdas-with-aws-cdk/)

[Deploying an ASP.NET Core API to AWS Fargate using CDK](https://awstip.com/deploying-an-asp-net-core-api-to-aws-fargate-using-cdk-dab10bef51a1) by Christian Eder

[Hello, Cloud blog series](https://davidpallmann.hashnode.dev/series/hello-cloud)