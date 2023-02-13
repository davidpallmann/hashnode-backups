# Hello, .NET Lambda Annotations

#### This episode: .NET Lambda Annotations Framework and simplified coding of HTTP endpoints. In this Hello, Cloud blog series, we're covering the basics of AWS cloud services for newcomers who are .NET developers. If you love C# but are new to AWS, or to this particular service, this should give you a jumpstart.

In this post we'll introduce The .NET Lambda Annotations Framework and use it in a "Hello, Cloud" .NET program to create an API for date operations. We'll do this step-by-step, making no assumptions other than familiarity with C# and Visual Studio. We're using Visual Studio 2022 and .NET 6.

# .NET Lambda Annotations Framework: What is it, and why use It?

> "Simple ingredients prepared in a simple way - that's the best way to take your everyday cooking to a higher level." —Jose Andres

[AWS Lambda](https://aws.amazon.com/lambda/?c=ser&sec=srv) is a serverless, event-driven compute service that has propelled the serverless revolution. Lambda handles all the administration of compute resources, and you only pay for requests and compute time. Programming-wise, you just supply the code, which are functions. It's hard to imagine things getting much simpler.

And yet, there is room for improvement. You normally write handler functions for AWS Lambda that take 2 parameters, an event object and an `ILambdaContext` parameter used for logging. If you're writing Lambda functions for an HTTP API, you've got to pull your HTTP endpoint parameters out of a `APIGatewayHttpApiV2ProxyRequest` object. Sometimes, that code is longer than the actual function logic itself.

[.NET Lambda Annotations Framework](https://www.nuget.org/packages/Amazon.Lambda.Annotations) (hereafter "Lambda Annotations"), also known as Amazon.Lambda.Annotations, is a nuget package for writing Lambda functions. AWS describes it as "a programming model for writing .NET Lambda functions that allows idiomatic .NET coding patterns".

### Traditional vs. Annotated Lambda Functions

Let's consider an example from the framework's [announcement blog post](https://aws.amazon.com/blogs/developer/introducing-net-annotations-lambda-framework-preview/) by Norm Johanson. The function below implements an API Gateway endpoint that expects x and y integer parameters from a request and returns their sum. Of the 22 lines in the body, 21 of them are dedicated to parameter extraction and the return object. There's just one statement performing the function logic.

```csharp
public APIGatewayHttpApiV2ProxyResponse LambdaMathAdd(APIGatewayHttpApiV2ProxyRequest request, ILambdaContext context)
{
    if (!request.PathParameters.TryGetValue("x", out var xs))
    {
        return new APIGatewayHttpApiV2ProxyResponse
        {
            StatusCode = (int)HttpStatusCode.BadRequest
        };
    }
    if (!request.PathParameters.TryGetValue("y", out var ys))
    {
        return new APIGatewayHttpApiV2ProxyResponse
        {
            StatusCode = (int)HttpStatusCode.BadRequest
        };
    }
    var x = int.Parse(xs);
    var y = int.Parse(ys);
    return new APIGatewayHttpApiV2ProxyResponse
    {
        StatusCode = (int)HttpStatusCode.OK,
        Body = (x + y).ToString(),
        Headers = new Dictionary<string, string> { { "Content-Type", "text/plain" } }
    };
}
```

Now, let's contrast that to an equivalent function written using .NET Lambda Annotations. This much shorter function has just one statement which performs the logic, and some attributes. The attributes are the "annotations" the framework allows us to write. The `[LambdaFunction]` attribute identifies this as a Lambda function, and the `[HttpApi...]` attribute defines an HTTP API endpoint for a GET method and its path. This function concerns itself only with the parameters and return type it needs to deal with, and doesn't have to worry about the plumbing. There's no need to write a Lambda function handler and work with `APIGatewayHttpApiV2ProxyRequest`, `ILambdaContext`, and `APIGatewayHttpApiV2ProxyResponse` objects.

```csharp
[LambdaFunction]
[HttpApi(LambdaHttpMethod.Get, "/add/{x}/{y}")]
public int Add(int x, int y)
{
    return x + y;
}
```

In summary, this annotated function is a more **natural** way to code an HTTP endpoint method in C#. But, you ask, don't we need all those things that went away? We do indeed, but they are created for you. Under the hood, when you build the program, code is generated that translates between the code you've written and what Lambda functions need. The project's CloudFormation template gets updated as well.

# Our Hello, Lambda Annotations Project

In this tutorial, we'll create an API for date operations, defining several HTTP endpoints in one source file. We'll end up with a `datediff/from/to` function that returns the number of days between 2 dates, and a `dateadd/date/days` function that adds a certain number of days to a date and returns the new date.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1676249520276/4845eb59-fd4d-4a45-92dc-6fdef601da9d.png align="left")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1676249542853/176a6252-0ddd-494a-a1ca-d52a679b4dac.png align="left")

[source code](https://github.com/davidpallmann/HelloLambdaAnnotations)

## One-time Setup

For any of the tutorials in the Hello, Cloud series you need the following:

1. An [AWS account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/), and an understanding of what is included in the [AWS Free Tier](https://aws.amazon.com/premiumsupport/knowledge-center/what-is-free-tier/).
    
2. [Microsoft Visual Studio 2022](https://visualstudio.microsoft.com/). If you're using an older version of Visual Studio you won't be able to use .NET 6. If you use a different IDE, you may have to find alternatives to some tutorial steps.
    
3. [AWS Toolkit for Visual Studio](https://aws.amazon.com/visualstudio/). You'll need to [configure the toolkit](https://docs.aws.amazon.com/toolkit-for-visual-studio/latest/user-guide/keys-profiles-credentials.html) to access your AWS account and create an IAM user. Your default AWS profile will be linked to this user when running programs from the command line.
    

## Step 1: Create a Project

In this step, you'll create a new serverless application in Visual Studio using the Annotations Framework template.

1. Launch Visual Studio and create a new project.
    
2. Search for and select the **AWS Serverless Application (.NET Core - C#)** project template. Click **Next**.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1676244543505/81172478-f9a8-4a8e-97e3-b96a3dea3fde.png align="center")
    
3. Name your project **HelloLambdaAnnotations** and select a development folder. Click **Create**.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1676244585133/7e8f813c-bbfd-491e-a697-35bd2ef43b2d.png align="center")
    
4. On the Select Blueprint dialog, search for and select the **Annotations Framework** blueprint. Click **Finish**.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1676244711599/743eff5b-06fb-4f0d-8a05-b7a1f40a5794.png align="center")
    
5. Review the generated project and note the following. The project contains the Amazon.Lambda.Annotations nuget package. The Function.cs file contains sample functions for a calculator with names such as Add and Multiply.
    

## Step 2: Code Your API Functions

In this step, you'll code your own API functions using Lambda Annotations.

1. Below the Default function, insert the code below at the end of this step to add two functions, DateDiff and DateAdd. DateDiff will accept two date string parameters, `from` and `to`, and return the number of days difference between them. DateAdd will accept a `date` and `days` parameter. It will add the days value to the original date and return a new date string.
    
2. Remove the other date functions below that came with the blueprint.
    
3. Remove the Default function that came with the blueprint.
    
4. Build the program.
    

Function.cs code to add

```csharp
        /// <summary>
        /// Compute the difference in days between two dates, from and to.
        /// </summary>
        /// <param name="from"></param>
        /// <param name="to"></param>
        /// <returns>number of days between from and to.</returns>
        [LambdaFunction()]
        [HttpApi(LambdaHttpMethod.Get, "/datediff/{from}/{to}")]
        public int DateDifferenceInDays(string from, string to, ILambdaContext context)
        {
            int days = Convert.ToInt16((DateTime.Parse(to) - DateTime.Parse(from)).TotalDays);
            context.Logger.LogInformation($"datediff {from} {to} is {days}");
            return days;
        }

        /// <summary>
        /// Add a day count to a date and return the new date.
        /// </summary>
        /// <param name="date">starting date</param>
        /// <param name="days">number of days to add</param>
        /// <returns>resulting date</returns>
        [LambdaFunction()]
        [HttpApi(LambdaHttpMethod.Get, "/dateadd/{date}/{days}")]
        public string DateAdd(string date, int days, ILambdaContext context)
        {
            return (DateTime.Parse(date).AddDays(days)).ToShortDateString();
        }
```

## Step 3. Deploy Function to AWS Lambda

In this step you'll publish your API to AWS Lambda.

1. In Visual Studio Solution Explorer, right-click the `HelloLambdaAnnotations` project and select **Publish to AWS Lambda..**.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1676246291031/c12664ef-5192-40d6-9ef8-343ec2a84e10.png align="center")
    
2. On the Publish to AWS Serverless Application wizard,
    
    1. select a Region to publish to. We're using us-west-2 (Oregon).
        
    2. Enter a Stack Name of **HelloLambdaAnnotations**.
        
    3. Next to S3 Bucket, click **New...** and enter **hello-lambda-annotations** for Bucket name. Click **OK**.
        
    4. Click **Publish**.
        
        ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1676246670754/923c2027-e749-464a-97c3-0a866240e723.png align="center")
        
3. Wait for publishing to complete. When you see a Status of CREATE\_COMPLETE at top, publishing is complete.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1676246735677/540e5125-6c76-4981-993e-90e12eee4d85.png align="center")
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1676246788373/b5cca829-f16d-43e1-8db6-2c1f03b1c49b.png align="center")
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1676247096410/f3da579d-b34c-42e9-b267-a6d9ec06b4d8.png align="center")
    

4\. In the top section of the publish page, copy the AWS Serverless URL. This is the base path to your API. Ours was `https://7mcinuk7u6.execute-api.us-west-2.amazonaws.com/`.

## Step 4: Test the API

Now it's time to test our API endpoints.

1. In a browser, visit the API URL you just copied. You see an instructional page.
    
2. Test the datediff function. Add `/datediff/2022-01-01/2023-01-01` to the end of the path and hit ENTER. You should get a response of 365. Try some different date values. In our second example, we used to `/datediff/1969-07-20/2023-02-12` to see how many days had elapsed since the Apollo 11 moon landing.

    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1676247446738/50e382cd-bc61-430b-a215-f8a7c9a1871c.png align="left")

    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1676247546259/73b9d4d6-b9b3-42be-85c0-8f989f1763a5.png align="left")

1. Test the dateadd function. Change the path to `/dateadd/2022-12-25/7` and press ENTER. The function should compute that one week after Christmas 2022 is New Year's Day 2023. Try another example.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1676248225838/85dce557-c340-4352-b8ce-a842427438db.png align="center")
    

Congratulations, you've just created AWS Lambda function HTTP endpoints with the .NET Lambda Annotations Framework.

## Step 5: Shut it Down

When you're done with the tutorial, delete the AWS resources.

1. In Visual Studio, go to AWS Explore and refresh the view.
    
2. Expand CloudFormation.
    
3. Right-click the HelloLambdaAnnotations node and select **Delete**.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1676248818479/af56702b-26d1-42d5-b98c-c740bd2fb603.png align="center")
    

## Where To Go From Here

In this tutorial, you saw how naturally you can express HTTP endpoints in C# with the.NET Lambda Annotations Framework. The framework uses Source Code Generators, and is not available for other .NET languages such as F#.

To go further, start using the framework for your own HTTP endpoints. Review the resources linked below for more information and tutorials. You can get a deeper explanation of the mechanics from the announcement blog by Norm Johanson.

## Further Reading

AWS Documentation

[AWS SDK for .NET Developer Guide: Using annotations to write AWS Lambda functions](https://docs.aws.amazon.com/sdk-for-net/v3/developer-guide/aws-lambda-annotations.html)

[nuget: Amazon.Lambda.Notations](https://www.nuget.org/packages/Amazon.Lambda.Annotations)

[aws/aws-lambda-dotnet GitHub](https://github.com/aws/aws-lambda-dotnet/tree/master/Libraries/src/Amazon.Lambda.Annotations)

Videos

[Lambda Annotations Framework for .NET](https://www.youtube.com/watch?v=ZtXWIKrZSMQ) by James Eastham

[AWS re:Invent 2022: AMster & The Brit's Code Corner: Using .NET Lambda Annotations, Pt 1](https://www.youtube.com/watch?v=AVjppxTF1YI)

Blogs

[Introducing .NET Annotations Lambda Framework](https://aws.amazon.com/blogs/developer/introducing-net-annotations-lambda-framework-preview/) by Norm Johanson

[Dependency Injection with the Lambda Annotations Library for .NET - Part 1, Lambda Applications](https://nodogmablog.bryanhogan.net/2022/10/dependency-injection-with-the-lambda-annotations-library-for-net-part-1-lambda-applications/) by Bryan Hogan

[Dependency Injection with the Lambda Annotations Library for .NET - Part 2, Lambda Functions](https://nodogmablog.bryanhogan.net/2022/10/dependency-injection-with-the-lambda-annotations-library-for-net-part-2-lambda-functions/) by Bryan Hogan

[Hello, Cloud series home](https://davidpallmann.hashnode.dev/series/hello-cloud)

Courseware

[AWS Lambda for the .NET Developer](https://www.udemy.com/course/aws-lambda-dotnet/) - Udemy course by Rahul Nath