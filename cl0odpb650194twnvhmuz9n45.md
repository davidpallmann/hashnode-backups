## Hello, Graviton!

#### This episode: AWS Graviton and .NET on ARM64. In this Hello, Cloud blog series, we're covering the basics of AWS cloud services for newcomers who are .NET developers. If you love C# but are new to AWS, or to this particular feature, this should give you a jumpstart.

In this post we'll introduce AWS Graviton and deploy a "Hello, Cloud" .NET 6 function to AWS Lambda that runs on ARM64 processors. We'll do this step-by-step, making no assumptions other than familiarity with C# and Visual Studio. We're using Visual Studio 2022 and .NET 6.

# AWS Graviton: What is it, and why use It?

> "Gravitons are the avatars of general covariance." —Frank Wilczek 

ARM processors are everywhere. These powerful and efficient reduced-instruction set computer (RISC) chips power our phones and tablets, and are finding adoption in other areas, including laptops and servers. In 2018, Amazon Web Services introduced [AWS Graviton](https://aws.amazon.com/pm/ec2-graviton) (hereafter "Graviton"), its own ARM64-based processors.

Why should ARM processors matter to you? Price and performance. AWS describes Graviton processors as "designed by AWS to deliver the best price performance for your cloud workloads running in Amazon EC2". The original Graviton generation was followed in 2019 by Graviton2, which substantially increased performance, with up to 40% better price performance over comparable x86-based instances. AWS Graviton3, now in preview, brings another significant performance boost, with up to 25% better compute performance, up to 2x higher floating-point performance, and up to 2x faster cryptographic workload performance compared to Graviton2 processors.

Can you run your .NET workloads on Graviton processors? Yes you can, if you modernize to .NET running on Linux. You can run ARM64 .NET code on Graviton processors via Amazon EC2, AWS Lambda, Amazon ECS, or Amazon EKS. Today, AWS offers 12 different types of Graviton instances across 23 regions:

| EC2 Instance Family | Use cases |
| ------------------- | --------- |
| M6g, M6gd | General-purpose workloads |
| T4g | Burstable general-purpose workloads |
| C6g, C6gd, C6gn | Computer-intensive workloads |
| R6g, R6gd, X2gd | Memory-intensive workloads |
| Im4gn, Is4gen | Storage-intensive workloads |
| G5g | GPU-based graphics and machine learning workloads |

For some insight into .NET performance, read the .NET 5 benchmark on Graviton2 processors at the end of this post. "The Graviton2 instance handled 64% more requests per dollar for the MvcJsonOutput2M test, and provides much better performance per dollar across all the tests."

# Our Hello, Graviton Project

We will create a .NET 6 function that performs temperature conversion, and deploy it to AWS Lambda to run on ARM64 Graviton processors. We've previously worked with Lambda functions in [Hello, Lambda](https://davidpallmann.hashnode.dev/hello-lambda), but now we're doing so with .NET 6 and ARM64 architecture.

![05-vs-publish-2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647119475162/w-1j1EuRZ.png)

[source code](https://github.com/davidpallmann/hello-graviton)

## One-time Setup

For any of the tutorials in the Hello, Cloud series you need the following:

1. An  [AWS account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/), and an understanding of what is included in the  [AWS Free Tier](https://aws.amazon.com/premiumsupport/knowledge-center/what-is-free-tier/). 

2. [Microsoft Visual Studio 2022](https://visualstudio.microsoft.com/). If you're using an older version of Visual Studio you won't be able to use .NET 6. If you use a different IDE, you may have to find alternatives to some tutorial steps.

3. [AWS Toolkit for Visual Studio](https://aws.amazon.com/visualstudio/). You'll need to [configure the toolkit ](https://docs.aws.amazon.com/toolkit-for-visual-studio/latest/user-guide/keys-profiles-credentials.html) to access your AWS account and create an IAM user. Your default AWS profile will be linked to this user when running programs from the command line.

    In addition, install the following for this tutorial:

4. The latest [AWS Lambda templates and tools](https://docs.aws.amazon.com/lambda/latest/dg/csharp-package-cli.html): 

```none
dotnet new -i Amazon.Lambda.Templates
dotnet tool install -g Amazon.Lambda.Tools
``` 

## Step 1: Create a New AWS Lambda Function

In this step, we'll create a new AWS Lambda function in the AWS console.

1. Navigate to the [AWS console](https://aws.amazon.com/console/) and sign in. 

2. At top-right, select an [AWS Region that supports AWS Lambda with Graviton2 processors](https://aws.amazon.com/about-aws/whats-new/2021/09/better-price-performance-aws-lambda-functions-aws-graviton2-processor/). We're going to use **us-west-2 (Oregon)**.

3. Navigate to the AWS Lambda section. You can enter **lambda** in the search bar to find it. 

4. Click **Functions** in the left panel.

5. Click **Create a Function** and enter/select the following:

    A. Select **Author from Scratch**, since this will be a simple, self-contained function.

    B. Function name: **hello-graviton**

    C. Runtime: **.NET 6 (C#/PowerShell)**

    D. Permissions - change default execution role: **Create a new role with basic permissions**

    E. Click **Create Function**.

    ![02-create-lambda.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647111422951/oJSDU1mkp.png)

    The page that follows will show your configured-but-not-yet-uploaded hello-graviton function.

6. Take note of the role that was automatically generated for the function. You can find this on the Configuration tab, Permissions section in the Execution role panel. 

    ![02-create-lambda-02.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647111722944/a3-DHHW3r.png)

    Click on the role to see its detail and record the Amazon Resource Name (ARN). If in the future you want to deploy your function using the dotnet command, you'll need the Amazon Resource Name (ARN) for this role.

    ![02-create-lambda-03.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647111915368/QYxUzBqky.png)

## Step 2: Create a New Lambda Project with Visual Studio

Now we're ready to write our C# function in Visual Studio.
1. Launch Visual Studio and select **Create New Project**. 

2. Browse/search/filter for **Lambda Empty Function (AWS)** and select it. Then click **Next**.

    ![03-vs-create-project.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647120171926/25ieWVIwT.png)

3. On the next page, enter the Project name **hello-graviton**.

4. Enter your preferred folder location.

5. Click **Next**.

    ![03-vs-create-project-2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647108176632/ltbuFiRnO.png)

6. On the next page, enter **default** for profile and enter your region name (ours is **us-west-2**).

    ![03-vs-create-project-3.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647120245424/ekzL64jwL.png)

7. Click **Create**.

    Moments later, your project has been created.

## Step 3: Code Your Lambda Function

Now we'll replace the generated Lambda function with our own.
1. In Visual Studio, open Function.cs file in the code editor.
2. Replace the code in Function.cs with the version below, which converts temperatures in degrees Celsius to Fahrenheit.
3. Build your project.

```csharp
using Amazon.Lambda.Core;

// Assembly attribute to enable the Lambda function's JSON input to be converted into a .NET class.
[assembly: LambdaSerializer(typeof(Amazon.Lambda.Serialization.SystemTextJson.DefaultLambdaJsonSerializer))]

namespace helloGraviton
{
    public class Function
    {

        /// <summary>
        /// A simple function that takes a temperature in degrees Celsius and returns a conversion to degrees Fahrehneit.
        /// </summary>
        /// <param name="input">degrees Celsius</param>
        /// <param name="context"></param>
        /// <returns>degrees Fahrenheit</returns>
        public double FunctionHandler(double degreesC, ILambdaContext context)
        {
            return (degreesC * 9/5) + 32;
        }
    }
}
``` 

## Step 4: Deploy Lambda Function to AWS

Now we'll publish our function to AWS. We can do that right from Visual Studio. There are several other ways to publish, including the dotnet command or uploading from the AWS console. 

1. In Visual Studio, open the AWS Explorer and set the region to the same region you set in Step 1. For us, that's **us-west-2 (Oregon)**. If you expand the AWS Lambda node of the explorer, you should see the **hello-graviton** Lambda function you created in Step 1.

    ![05-vs-aws-explorer.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647112160188/BRrCcDJq9.png)

2. In Solution Explorer, right-click the **hello-graviton** project and select **Publish to AWS Lambda**.

    ![05-vs-publish.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647112306680/eaZg8dOaN.png)

3. In the Upload Lambda Function dialog, enter/select the following:

    A. Package type: **Zip**.

    B. Lambda Runtime: **.NET 6**.

    C. Architecture: **ARM**.

    D. Function Name: **hello-graviton**.

    E. Handler: **hello-graviton::helloGraviton.Function::FunctionHandler**. This value must be of the form assembly::class::method and exactly match the assembly, class, and method names of your function, or invocation will fail. In our case, the assembly name is hello-graviton, the class name (including namespace) is helloGraviton.Function, and the method name is FunctionHandler.

    F. Click **Upload**.

    ![05-vs-publish-2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647113197721/TgE2s4VX_.png)

4. Click **Upload **and wait for the function to upload. If the Next and Upload buttons are disabled, check that you filled out everything on the dialog, and that the Handler value is correct.

    ![05-vs-publish-3.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647113269055/NqMyjgEuc.png)

5. After publishing completes, if there are no errors, you should be at a test page for the Lambda function in Visual Studio. When you see **Last update status: Successful** at top, your function has been deployed!

    ![05-vs-publish-4.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647112811247/Hitfc4eHQ.png)

6. Back in the AWS console, navigate back to AWS Lambda > Functions > hello-graviton, or refresh the page if already there. At the bottom of the page in the Runtime settings panel, you see evidence that your code has been published. The handler name is set, and the architecture is Arm64.

    ![05-vs-publish-5-aws.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647120477668/YAPUPECcE.png)

### Publishing from the Command Line

By the way, an alternative to the publish operation we just performed from Visual Studio is to use the command line and the **dotnet lambda** command. If you use this method, you'll need to specify the ARN of the role that AWS generated for you in Step 1

```none
dotnet lambda deploy-function hello-cloud --function-role <role>
``` 

## Step 5: Test the Lambda Function

With our AWS Lambda function deployed, we can test it. Well do that from Visual Studio in this post, but be aware that you can also test from the AWS console or with the `dotnet lambda` command.

1. Since we deployed with Visual Studio in Step 4, you should already be on the Lambda test page in Visual Studio. 

2. In the Sample Input large text box, enter "100", or any Celsius temperature you wish to test.

3. Click the **Invoke **button.

4. Your function executes, and the output is shown at right in the Response area. We see that the input value of 100 degrees Celsius has been converted to 212 degrees Fahrenheit. Give yourself a hand, you just executed .NET code on an ARM64 Graviton processor!

5. Note the RequestID and duration in MS for the function execution are available in the log output pane.

    ![06-vs-test.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647113472750/bqr9uERQ1.png)

6. Try out some different values.

    ![06-vs-test-2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647113930091/GQIGRUkDC.png)

## Step 6: Delete the Function

When you're done with hello-graviton, remember to delete it. 

1. Navigate to Lambda > Functions.

2. Check the check box next to **hello-graviton**.

3. From the Actions menu, select **Delete** and confirm.

Congratulations! You're on your way to serverless greatness with .NET 6 and ARM64 on AWS! 

# Where to Go From Here

If you've never run .NET code on ARM64 processors before, you've now seen how simple it is. In this tutorial, you created an AWS Lambda function, implemented the function in .NET 6, and deployed to AWS Lambda from Visual Studio. All you had to do to run on ARM architecture was specify an option in the Publish to AWS Lambda dialog.

If you're used to working with .NET and AWS Lambda, you'll want to read Norm Johanson's blog post on AWS Lambda .NET 6 runtime support (below under Further Reading) to be aware of changes and new options, such as using minimal code top-level statements. Be aware that some of the AWS Lambda project templates require updates to work with ARM64. See Bryan Hogan's blog posts (also linked below) for instructions. 

If your ARM64 interest lies with other AWS services, such as Amazon ECS, read the service documentation and recent AWS blog posts to understand how to configure running on Graviton processors and which regions support them.

If you're attracted to ARM64 processors by the price-performance benefits, read more about AWS Graviton and review benchmarks at the links below. Consider what kind of benchmarks and tests will help you measure the price-performance improvement for your own applications. You can evaluate AWS Graviton2 now, and Graviton3 once it is generally available.

# Further Reading

AWS Documentation

[AWS Graviton processor](https://aws.amazon.com/ec2/graviton/?refid=16d7371a-b59f-470f-890c-6d3cbdd99fe3)

[Workshop: Amazon EKS: Running ASP.NET Core Application on Graviton2](https://graviton2-workshop.workshop.aws/en/amazoncontainers/amazoneks/aspnet.html)

[Workshop: .NET 5 on AWS Graviton2](https://catalog.us-east-1.prod.workshops.aws/workshops/c36ccd6e-9145-4e97-b1b5-1069d6d68ed0/en-US/lab8-net5-graviton2)

[aws-graviton-getting-started: .NET on Graviton](https://github.com/aws/aws-graviton-getting-started/blob/main/dotnet.md)

[.NET 6 Support on AWS Guide](https://github.com/aws-samples/aws-net-guides/tree/master/RuntimeSupport/dotnet6)

Videos

[Deep dive into AWS Graviton3 and Amazon EC2 C7g instances](https://www.youtube.com/watch?v=WDKwwFQKfSI)

Blogs

[Powering .NET 5 with AWS Graviton2: Benchmarks](https://aws.amazon.com/blogs/compute/powering-net-5-with-aws-graviton2-benchmark-results/)

[Build and deploy .NET web applications to ARM-powered AWS Graviton2 Amazon ECS Clusters using AWS CDK](https://aws.amazon.com/blogs/devops/build-and-deploy-net-web-applications-to-arm-powered-aws-graviton-2-amazon-ecs-clusters-using-aws-cdk/)

[Join the Preview - Amazon EC2 C7g Instances Powered by New AWS Graviton3 Processors](https://aws.amazon.com/blogs/aws/join-the-preview-amazon-ec2-c7g-instances-powered-by-new-aws-graviton3-processors/)

[Introducing the .NET 6 runtime for AWS Lambda](https://aws.amazon.com/blogs/compute/introducing-the-net-6-runtime-for-aws-lambda/) by Norm Johanson

[.NET 6 Lambdas on ARM64 - Part 1, Functions](https://nodogmablog.bryanhogan.net/2022/03/net-6-lambdas-on-arm64-part-1-functions) by Bryan Hogan

[.NET 6 Lambdas on ARM64 - Part 2, Serverless](https://nodogmablog.bryanhogan.net/2022/03/net-6-lambdas-on-arm64-part-2-serverless/) by Bryan Hogan

[Hello, Cloud blog series](https://davidpallmann.hashnode.dev/series/hello-cloud)   