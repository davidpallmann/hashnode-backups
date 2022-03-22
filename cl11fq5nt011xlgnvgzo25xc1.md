## Hello, API Gateway!

#### This episode: Amazon API Gateway. In this Hello, Cloud blog series, we're covering the basics of AWS cloud services for newcomers who are .NET developers. If you love C# but are new to AWS, or to this particular service, this should give you a jumpstart.

In this post we'll introduce API Gateway and use it to create a "Hello, Cloud" .NET API to store and retrieve restaurant menus. We'll do this step-by-step, making no assumptions other than familiarity with C# and Visual Studio. We're using Visual Studio 2022 and .NET 6.

# Amazon API Gateway : What is it, and why use It?

> "Anyone can dream up great ideas, but an idea is nothing until it's realized, be it as a website, a physical product, an app, or a user interface." —Jens Martin Skibsted

The importance of APIs continues to rise, in concert with worldwide demand for more integration and automation. APIs are becoming a widespread expectation. Some companies' entire product lines are APIs, such as Stripe or Twilio. APIs put your products and services in reach of others, and enable you to use the products and services of others.

The "[API Gateway](https://microservices.io/patterns/apigateway.html)" design pattern provides a single entry point that clients can use to talk to your services, and bears some similarities to the Facade pattern. Think of it as the front door to your business logic, functionality, and data. This pattern is particularly useful for microservices, where the facade of an API Gateway can cater to client needs. Different clients might require varying protocols, API versions, authentication, orchestration, or transient fault retries. An API Gateway can provide these features, and it often makes sense to handle them there rather than implement them in each individual microservice.

[Amazon API Gateway](https://aws.amazon.com/api-gateway/) (hereafter "API Gateway") is the API Gateway design pattern as a service. AWS describes it as "a fully managed service that makes it easy for developers to create, publish, maintain, monitor, and secure APIs at any scale". API Gateway allows you to build HTTP APIs, RESTful APIs, and Websocket APIs (useful for real-time chat and streaming). With API Gateway providing your public API endpoints, your microservices and their supporting resources can stay private.

![diagram-general.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647688455953/ypqqg0pbL.png)

[Features of API Gateway](https://docs.aws.amazon.com/apigateway/latest/developerguide/welcome.html#api-gateway-overview-features) include multiple options for authentication, canary release deployments, logging to AWS CloudTrail and Amazon CloudWatch, CloudFormation template support, custom domain names, and integration with AWS Web Application Firewall and AWS X-Ray. You can access API Gateway via the AWS management console, the AWS SDK, the AWS CLI, and AWS Tools for Windows PowerShell. API Gateway integrates particularly well with AWS Lambda functions, which will be the focus of our tutorial.

Amazon API Gateway is priced per millions of requests per month and rates vary by region. The [AWS Free Tier](https://aws.amazon.com/api-gateway/pricing/?loc=ft#Free_Tier) provides 1 million free API calls per month for 12 months.

# Our Hello, API Gateway Project

We will create a simple API for storing and retrieving restaurant menu data. We will create 2 AWS Lambda functions, one for storing menu data and one for retrieving it. Menu data will be stored in and retrieved from S3. Then, we'll create a REST API for storing and retrieving restaurant menus that integrates to the AWS Lambda functions. 

![07-test-GET-name.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647825620892/B9ejfyXVv.png)

The API we'll create has these methods. and will be backed by 2 Lambda functions, one to store menu JSON data in S3 and one to retrieve menu data from S3.

| Path | Method | Description | Example |
| ---- | ---- | ----- | ---- |
| / | GET | retrieve default menu | GET / |
| / | PUT | store default menu | PUT / |
| /name | GET | retrieve named menu | GET /breakfast |
| /name | PUT | store named menu | PUT /breakfast |

[source code](https://github.com/davidpallmann/hello-api-gateway)

## One-time Setup

For any of the tutorials in the Hello, Cloud series you need the following:

1. An  [AWS account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/), and an understanding of what is included in the  [AWS Free Tier](https://aws.amazon.com/premiumsupport/knowledge-center/what-is-free-tier/). 
2. [Microsoft Visual Studio 2022](https://visualstudio.microsoft.com/). If you're using an older version of Visual Studio you won't be able to use .NET 6. If you use a different IDE, you may have to find alternatives to some tutorial steps.
3. [AWS Toolkit for Visual Studio](https://aws.amazon.com/visualstudio/). You'll need to [configure the toolkit ](https://docs.aws.amazon.com/toolkit-for-visual-studio/latest/user-guide/keys-profiles-credentials.html) to access your AWS account and create an IAM user. Your default AWS profile will be linked to this user when running programs from the command line.

## Step 1: Create AWS Artifacts Create an S3 Bucket

In this step, you'll create some artifacts in the AWS console, including an S3 bucket for storage, two AWS Lambda functions, and an IAM role allowing the Lambda functions to access the S3 bucket. 

1. Sign in to the [AWS management console](https://aws.amazon.com/console/). At the top right, select a region you want to work in that supports Amazon API Gateway and AWS Lambda. We're using **us-west-1 (N. California)**. Set the same region consistently as you go through these steps.

2. Create an S3 bucket with a unique name for menu storage. We named ours **hello-api-gateway-menu**.

    A. Navigate to Amazon S3. You can enter **s3** in the search bar.

    B. Click **Buckets** on the left panel.

    C. Click the **Create bucket** button at right.

    D. Enter a unique bucket name to store your menu data.

    E. Set the desired region.

    F. At the bottom of the page, click **Create bucket**. After a moment, you should see your new bucket listed under Buckets.

    ![01-aws-create-bucket.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647806502723/5thBJXBRD.png)

    ![01-aws-create-bucket-02.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647806516283/lrIDm1MbY.png)

    G. Click on the bucket name to view its detail. On the Properties tab, find and record the Amazon Resource Name (ARN) of the bucket. Ours is `arn:aws:s3:::hello-api-gateway-menu`.

    ![01-aws-create-bucket-03.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647806598637/uhaMvA3hI.png)

3. Create the MenuPut Lambda function. This function will store menu data.

    A. Navigate to AWS Lambda. You can enter **lambda** in the search box.

    B. On the left panel, click **Functions**.

    C. Click **Create Function**.

    D. Select Author from Scratch, since this will be a simple, self-contained function.

    E. For Function name, enter **MenuPut**.

    F. Under Runtime, select **.NET 6 (C#/PowerShell)**.

    G. Expand the Change default execution role section and select **Create a new role with basic Lambda permissions**. Look below for "Lambda will create an execution role named [role]", and _record the execution role name_. Ours is `MenuPut-role-vrb0zu5u`.

    ![01-aws-create-MenuPut.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647806996779/1wJjbf83-.png)

    H. Click **Create Function** and wait for the function to be created. 
    
    ![01-aws-create-MenuPut-02.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647807102061/q6AudLKhp.png)

4. Create the MenuGet Lambda function. This function will retrieve the menu.

    A. Navigate back to AWS Lambda > Functions.

    B. Click **Create Function**.

    C. Select Author from Scratch.

    D. For Function name, enter **MenuGet**.

    E. Under Runtime, select **.NET 6 (C#/PowerShell)**.

    F. Expand the Change default execution role section and select **Use an existing role**. For *Existing role, enter the role name you recorded in #3-G above.

    ![01-aws-create-MenuGet.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647807244959/cVH6fch7g.png)

    G. Click **Create Function** and wait for the function to be created. 

5. Create an IAM Role giving the Lambda execution role permission to access the S3 bucket.

    A. Navigate to the Identity and Access Management (IAM) area. You can enter **iam** in the search box.

    B. In the left panel, click **Roles**.

    C. Search for the Lambda execution role name you recorded in #3-G above and click on it to views its details.

    D. On the permissions tab, select **Create inline policy** under the Add permissions dropdown.

    ![01-aws-update-role.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647807351056/8CR3h3TSC.png)

    E. On the JSON tab, enter the JSON below at the end of this step. Replace [BUCKET-ARN] with the ARM for your S3 bucket in #2-G above.

    ![01-aws-update-role-02.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647807470470/WjRRzKEeS.png)

    F. Click **Review policy**. Name the policy **s3-bucket-menu** and click **Create policy**. You should now see your s3-bucket-menu inline policy as well as the auto-generated Lambda basic execution role policy attached to the role.

    ![01-aws-update-role-03.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647807576736/vIZhQi9XI.png)

    ![01-aws-update-role-04.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647807667927/6d-AMQc69.png)

Inline Policy JSON

```JSON
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ExampleStmt",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Effect": "Allow",
      "Resource": [
        "arn:aws:s3:::[BUCKET-ARN]/*"
      ]
    }
  ]
}
```

## Step 2: Create the MenuPut Lambda Function

In this step, you'll create a Lambda function named MenuPut that can store a menu in an S3 bucket. 

1. Launch Visual Studio and select Create Project.

    A. Find and select the **Lambda Simple S3 Function** template and click **Next**.

    ![01-vs-create-lambda-s3.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647711291005/G8WeUxdc2.png)

    B. On the Configure your new project page, name the project **MenuPut** and choose a development folder to work in. Then click **Next**.

    ![01-vs-create-lambda-s3-02.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647711438856/W053TMBfT.png)

    C. On the Additional information page, set Profile to **default** and Region to your region. We're using **us-west-1**. Then click **Create**.

    ![02-vs-create-menu-put-03.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647808049444/-q8HznRcw.png)

2. In the AWS Explorer pane, set your region.

3. In Solution Explorer, right-click the MenuPut project and select **Manage NuGet Packages...**. Find and install the **Amazon.APIGateway.LambdaEvents** package.

    ![02-nuget.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647808337913/kSCxtbL5g.png)

4. Open Function.cs in the code editor, and replace with the code at the end of this step. Replace [BUCKET-NAME] with the name of your S3 bucket created in Step 1.

5. Save your changes and build the project.

6. In Solution Explorer, right-click the MenuPut project and select **Publish to AWS Lambda**. Enter/select the following in the publishing wizard:

    A. Function Name: select **MenuPut**.

    B. Function Handler: **MenuPut::MenuPut.Function::FunctionHandler**.

    C. Click **Upload**.

    ![02-publish-MenuPut.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647808756453/20zpUzr6H.png)

    ![02-publish-MenuPut-02.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647810308086/6zFlgoTGA.png)

    D. Wait for the function to upload. If successful, you will end up at the function test page with the message **Last Update Status: Successful** at top.

    ![02-publish-MenuPut-03.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647808915149/8UGNhAc7-.png)

Function.cs

```csharp
using Amazon.Lambda.APIGatewayEvents;
using Amazon.Lambda.Core;
using Amazon.S3;

[assembly: LambdaSerializer(typeof(Amazon.Lambda.Serialization.SystemTextJson.DefaultLambdaJsonSerializer))]

namespace MenuPut
{
    public class Function
    {
        const string BUCKET = "[BUCKET-NAME]";

        IAmazonS3 S3Client { get; set; }

        /// <summary>
        /// Constructor. Create S3 client.
        /// </summary>
        public Function()
        {
            S3Client = new AmazonS3Client();
        }

        /// <summary>
        /// Constructs an instance with a preconfigured S3 client. This can be used for testing the outside of the Lambda environment.
        /// </summary>
        /// <param name="s3Client"></param>
        public Function(IAmazonS3 s3Client)
        {
            this.S3Client = s3Client;
        }

        /// <summary>
        /// Store menu JSON in S3 menu object. 
        /// </summary>
        /// <param name="request">API Gateway proxy request containing body (menu JSON).</param>
        /// <param name="context">Lambda context object used for logging</param>
        /// <returns></returns>
        public async Task<APIGatewayProxyResponse> FunctionHandler(APIGatewayProxyRequest request, ILambdaContext context)
        {
            try
            {
                context.Logger.LogLine($"Updating menu");
                await PutMenu(request.Body);

                return new APIGatewayProxyResponse
                {
                    StatusCode = 200
                };
            }
            catch (Exception ex)
            {
                return new APIGatewayProxyResponse
                {
                    StatusCode = 500,
                    Body = ex.ToString()
                };
            }
        }

        private async Task PutMenu(string menuJson)
        {
            var response = await S3Client.PutObjectAsync(new Amazon.S3.Model.PutObjectRequest()
            {
                BucketName = BUCKET,
                Key = "menu.json",
                ContentType = "text/json",
                ContentBody = menuJson
            });
        }
    }
}
```

## Step 3: Create the MenuGet Lambda Function

In this step, you'll create a Lambda function named MenuGet that can retrieve a menu from an S3 bucket. 

1. In Visual Studio, close the previous solution and select **Create Project**.

    A. Find and select the **Lambda Simple S3 Function** template and click **Next**.

    B. On the Configure your new project page, name the project **MenuGet** and choose a development folder to work in. Then click **Next**.

    ![03-vs-create-MenuGet.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647809049335/X9D-XaoKx.png)

    C. On the Additional information page, set Profile to **default** and Region to your region. We're using **us-west-1**. Then click **Create**.

2. In Solution Explorer, right-click the MenuGet project and select **Manage NuGet Packages...**. Find and install the **Amazon.APIGateway.LambdaEvents** package.

    ![02-nuget.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647808337913/kSCxtbL5g.png)

4. Open Function.cs in the code editor, and replace with the code at the end of this step. Replace [BUCKET-NAME] with the name of your S3 bucket created in Step 1.

5. Save your changes and build the project.

6. In Solution Explorer, right-click the MenuGet project and select **Publish to AWS Lambda**. Enter/select the following in the publishing wizard:

    A. Function Name: select **MenuGet**.

    B. Handler: **MenuGet::MenuGet.Function::FunctionHandler**.

    C. Click **Upload**.

    D. Wait for the function to upload. If successful, you will end up at the function test page with the message **Last Update Status: Successful** at top.

Function.cs

```csharp
using Amazon.Lambda.APIGatewayEvents;
using Amazon.Lambda.Core;
using Amazon.S3;

[assembly: LambdaSerializer(typeof(Amazon.Lambda.Serialization.SystemTextJson.DefaultLambdaJsonSerializer))]

namespace MenuGet
{
    public class Function
    {
        const string BUCKET = "[BUCKET-NAME]";

        IAmazonS3 S3Client { get; set; }

        /// <summary>
        /// Constructor. Create S3 client.
        /// </summary>
        public Function()
        {
            S3Client = new AmazonS3Client();
        }

        /// <summary>
        /// Constructs an instance with a preconfigured S3 client. This can be used for testing the outside of the Lambda environment.
        /// </summary>
        /// <param name="s3Client"></param>
        public Function(IAmazonS3 s3Client)
        {
            this.S3Client = s3Client;
        }

        /// <summary>
        /// Retrieve menu JSON from S3 menu object. 
        /// </summary>
        /// <param name="request">API Gateway proxy request.</param>
        /// <param name="context">Lambda context object used for logging</param>
        /// <returns>menu JSON</returns>
        public async Task<APIGatewayProxyResponse> FunctionHandler(APIGatewayProxyRequest request, ILambdaContext context)
        {
            try
            {
                context.Logger.LogLine($"Retrieving menu");
                var menuJson = await GetMenu();

                return new APIGatewayProxyResponse
                {
                    StatusCode = 200,
                    Body = menuJson
                };
            }
            catch (Exception ex)
            {
                return new APIGatewayProxyResponse
                {
                    StatusCode = 500,
                    Body = ex.ToString()
                };
            }
        }

        private async Task<string> GetMenu()
        {
            var response = await S3Client.GetObjectAsync(BUCKET, "menu.json");
            using (var reader = new StreamReader(response.ResponseStream))
            {
                return await reader.ReadToEndAsync();
            }
        }
    }
}
```
## Step 4: Create API Gateway API and the PUT method

In this step, you'll use the AWS Console to configure an API for with API Gateway. We'll define a PUT method for the / path, that invokes the MenuPut Lambda function to store a menu in our S3 menu.json object.

1. In the AWS console, navigate to AWS Lambda and confirm the 2 Lambda functions you created and deployed in Steps 2 and 3 are present.

    ![04-aws-verify-lambdas.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647809708094/7_5i1W-hw.png)

2. Navigate to API Gateway in the console. You can enter **api** in the search box.

3. On the Lambda > APIs page, find the REST API panel and click **Build**.

    ![03-aws-rest-api-01.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647718703367/s5pz6Nimn.png)

4. On the APIs > Create page, enter/select the following:

    A. Choose the protocol: **REST**

    B. Create new API: **New API**

    C. Settings - API name: **menu**

    D. Description: **hello API gateway menu API**.

    E. Endpoint Type: **Regional**.

    F. Click **Create API**.

    ![04-create-api.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647812056421/yXI1yOI20.png)

5. Next, we'll define a PUT method for storing menu JSON. On the / Methods page, select **Create Method** from the Actions dropdown, and enter/select the following:

    A. set the action dropdown to **PUT** and click the checkbox icon button.

    B. Integration type: **Lambda Function**.

    C. Use Lambda Proxy Integration: **check**.

    D. Lambda Region: the region where your Lambda is deployed. We're using **us-west-1**.

    E. Lambda Function: **MenuPut**.

    F. Use Default Timeout: (check).

    G. Click **Save** and click **OK** for the Add Permission to Lambda Function confirmation.

    ![04-create-api-02.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647811174192/Fj0R4GFYS.png)

6. You now see a visualization of how the API method will integrate with the Lambda function. If not, select **PUT** from the left panel.

    ![04-create-api-03.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647719463431/J0c3k46Yb.png)

7. Click the **Test** section of the Client part of the diagram. On the / - PUT - Method Test page:

    A. Enter the JSON below in the Request Body input box.

    ```json
{
    "menu": {
        "items": [
            {
                "item": "dinner-rolls",
                "name": "Dinner Rolls",
                "qty": 3,
                "price": 2.99
            }
        ]
    }
}
```

    ![04-test-api.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647810118456/1SbkCI1UI.png)

    B. Click **Test**. The API method will execute, calling the MenuPut Lambda function. Results will be displayed at right, including the response JSON, response headers, and execution log. If all goes well, you will receive a 200 response. If instead you get an error response, use the exception detail and log to troubleshoot.

    ![04-test-api-02.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647810421297/Rn4qW9CZi.png)

8. In the AWS console, navigate to your S3 bucket. It should now contain an object named `menu.json` which should contain the JSON you entered in #7 above.

    ![04-test-api-03.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647811310005/1IzQmk4ZJ.png)

## Step 5: Create API Gateway GET method

In this step, well extend the API by adding a GET method for the / path. This method will call the MenuGet Lambda function to retrieve the stored menu from our S3 menu.json object.

1. Navigate back to API Gateway and select your **menu** API.

2. Add a GET method and configure it to call your MenuGet Lambda function:

     A. With / selected in the left panel, select **Create Method** from the Actions dropdown.

    B. Select **GET** and click the checkmark icon button.

    C. On the / - GET - Setup page, check **Use Lambda Proxy Integration**.

    D. For Lambda Function, select **MenuGet**.

    E. Click **Save** and click **OK** to confirm the notification about Lambda function permissions.

    ![04-create-api-MenuGet.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647811649340/OkqyIM_Xn.png)

3. With **GET** selected in the left panel, click **TEST** and then **Test**.

    ![04-test-get.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647811759337/NVXd3-mv7.png)

    ![04-test-get-02.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647811794911/4prugMuS4.png)

4. Wait for the API call to complete. If all goes well, the response body should be the menu JSON you stored when you tested the PUT operation earlier.

    ![04-test-get-03.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647811918552/Lh5fPiTky.png)

5. Optional: if you want to see how your Lambda function configuration has changed as a result of this step, navigate to AWS Lambda and view either of your functions. On the Configuration tab, select Triggers. You'll see that an API Gateway trigger has been configured for your menu API's GET and PUT method.

    ![05-lambda-trigger.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647911037767/MZLbj0xWT.png)

Congratulations! You've used Amazon API Gateway to invoke Lambda functions!

## Step 6: Extend the PUT API method to support a REST path parameter

Currently, we can store a menu with the PUT / method, but there is only a single menu name menu.json. In this step, you'll add API Gateway support for PUT /{name}, which will allow multiple menus, such as breakfast, lunch, and dinner menus.

1. In the AWS console, navigate to API Gateway.

2. Select the **menu** API.

3. In the left panel, select **/**.

4. In the Actions dropdown, select **Create resource**. On the New Child Resource page enter/select the following:

    A. Resource Name: **name**.

    B. Resource Path: **{name}**.

    ![06-create-resource.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647822585952/cZ4hn-KRn.png)

   C. Click **Create resource**. You should now see the /{name} parameter listed in the left panel.

    ![06-create-resource-02.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647822677593/oOvf873wi.png)

5. From the Action drop-down, select **Create Method** and enter/select the following:

    A. Select **PUT** and click the checkbox icon button.

    B. Integration type: **Lambda Function**.

    C. Use Lambda Proxy Integration: **check**.

    D. Lambda Region: your region.

    E. Lambda Function: select **MenuPut**.

    F. Click **Save**.

6. In Visual Studio, open the MenuPut solution and open Function.cs in the code editor. Replace FunctionHandler with the updated code at the end of this step. Replace [BUCKET-NAME] with your bucket name. Save your change and build the code.

The new code checks whether API Gateway has provided a path parameter `name`. If it is, that name is used for the menu name. If not, the default of menu is used as in the original implementation.

7. In Solution Explorer, right-click the MenuPut project and select **Publish to AWS Lambda**. Complete the wizard and deploy the updated MenuPut function to AWS Lambda. Wait for deployment to complete.

8. In the AWS console, navigate to API Gateway and view the **menu** API.

9. On the left panel, select the bottom item (the second **PUT** method).

10. To the right click **Test** and enter/select the following:

    A. {name}: **breakfast**.

    B. Enter this JSON:

    ```JSON
{
    "menu": {
        "items": [
            {
                "item": "apple-pancakes",
                "name": "Apple Pancakes",
                "qty": 5,
                "price": 11.99
            },
            {
                "item": "strawberry-waffle",
                "name": "Strawberry Waffle",
                "qty": 1,
                "price": 13.99
            }
        ]
    }
}
```

    ![06-test-PUT-name.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647824237902/M_uvPv2Jq.png)

    C. Click **Test** to call the API method. The response should confirm that the menu name was taken from the `name` parameter (**breakfast**). 

    ![06-test-PUT-name-02.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647824362017/TKOmo1bYd.png)

6. Verify the new object was created by navigating to S3 and your bucket, where you should now see a `breakfast.json` object as well as the original `menu.json` object.

    ![06-test-PUT-name-03.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647824412867/MhgyokSqm.png)

Give yourself a pat on the back. You've now passed a REST path parameter to a Lambda function via Amazon API Gateway.

Function.cs

```csharp
using Amazon.Lambda.APIGatewayEvents;
using Amazon.Lambda.Core;
using Amazon.S3;

[assembly: LambdaSerializer(typeof(Amazon.Lambda.Serialization.SystemTextJson.DefaultLambdaJsonSerializer))]

namespace MenuPut
{
    public class Function
    {
        const string BUCKET = "[BUCKET-NAME]";

        IAmazonS3 S3Client { get; set; }

        /// <summary>
        /// Constructor. Create S3 client.
        /// </summary>
        public Function()
        {
            S3Client = new AmazonS3Client();
        }

        /// <summary>
        /// Constructs an instance with a preconfigured S3 client. This can be used for testing the outside of the Lambda environment.
        /// </summary>
        /// <param name="s3Client"></param>
        public Function(IAmazonS3 s3Client)
        {
            this.S3Client = s3Client;
        }

        /// <summary>
        /// Store menu JSON in S3 menu object. 
        /// </summary>
        /// <param name="request">API Gateway proxy request containing body (menu JSON) and optional path parameter 'name'.</param>
        /// <param name="context">Lambda context object used for logging</param>
        /// <returns></returns>
        public async Task<APIGatewayProxyResponse> FunctionHandler(APIGatewayProxyRequest request, ILambdaContext context)
        {
            try
            {
                var menuJson = request.Body;
                context.Logger.LogLine($"Menu JSON: {menuJson}");

                var name = "menu";
                if (request.PathParameters != null && request.PathParameters.ContainsKey("name"))
                {
                    name = request.PathParameters["name"];
                }

                context.Logger.LogLine($"Updating menu {name}");
                await PutMenu(name, menuJson);

                return new APIGatewayProxyResponse
                {
                    StatusCode = 200,
                    Body = $"Updated menu {name}.json"
                };
            }
            catch (Exception ex)
            {
                return new APIGatewayProxyResponse
                {
                    StatusCode = 500,
                    Body = ex.ToString()
                };
            }
        }

        private async Task PutMenu(string name, string menuJson)
        {
            var response = await S3Client.PutObjectAsync(new Amazon.S3.Model.PutObjectRequest()
            {
                BucketName = BUCKET,
                Key = name + ".json",
                ContentType = "text/json",
                ContentBody = menuJson
            });
        }
    }
}
```

## Step 7: Extend the GET API method to support a REST path parameter

In this step, you'll make a similar change so the GET method can also use a name path parameter. This will allow retrieval of menus by name.

1. In the AWS console, navigate to API Gateway.

2. Select the **menu** API.

3. In the left panel, select **/{name}**.

4. From the Action drop-down, select **Create Method** and enter/select the following:

    A. Select **GET** and click the checkbox icon button.

    B. Integration type: **Lambda Function**.

    C. Use Lambda Proxy Integration: **check**.

    D. Lambda Region: your region.

    E. Lambda Function: select **MenuGet**.

    F. Click **Save** and click **OK** to confirm the permission notification.

    ![07-create-PUT-name.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647824714606/sjGKFOkDP.png)

6. In Visual Studio, open the MenuGet solution and open Function.cs in the code editor. Replace FunctionHandler with the updated code at the end of this step. Replace [BUCKET-NAME] with your bucket name. Save your change and build the code.

The new code checks whether API Gateway has provided a path parameter `name`. If it is, that name is used for the menu name. If not, the default of menu is used as in the original implementation.

7. In Solution Explorer, right-click the MenuGet project and select **Publish to AWS Lambda**. Complete the wizard and deploy the updated MenuGet function to AWS Lambda. Wait for deployment to complete.

8. In the AWS console, navigate to API Gateway and view the **menu** API.

9. On the left panel, select the bottom item (the second **GET** method).

    ![07-test-GET-name.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647825031964/nb2TILfma.png)

10. To the right click **Test** and enter/select the following:

    A. {name}: **breakfast**.

    ![07-test-GET-name-02.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647825075691/8oV_UtdCj.png)

    B. Click **Test** to call the API method. The response should contain the breakfast menu JSON you stored in Step 6.

    ![07-test-GET-name-03.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647825143238/f3RLXzXx8.png)

You've now fully implemented our API.

Function.cs

```csharp
using Amazon.Lambda.APIGatewayEvents;
using Amazon.Lambda.Core;
using Amazon.S3;

[assembly: LambdaSerializer(typeof(Amazon.Lambda.Serialization.SystemTextJson.DefaultLambdaJsonSerializer))]

namespace MenuGet
{
    public class Function
    {
        const string BUCKET = "[BUCKET-NAME]";

        IAmazonS3 S3Client { get; set; }

        /// <summary>
        /// Constructor. Create S3 client.
        /// </summary>
        public Function()
        {
            S3Client = new AmazonS3Client();
        }

        /// <summary>
        /// Constructs an instance with a preconfigured S3 client. This can be used for testing the outside of the Lambda environment.
        /// </summary>
        /// <param name="s3Client"></param>
        public Function(IAmazonS3 s3Client)
        {
            this.S3Client = s3Client;
        }

        /// <summary>
        /// Retrieve menu JSON from S3 menu object. 
        /// </summary>
        /// <param name="request">API Gateway proxy request.</param>
        /// <param name="context">Lambda context object used for logging</param>
        /// <returns>menu JSON</returns>
        public async Task<APIGatewayProxyResponse> FunctionHandler(APIGatewayProxyRequest request, ILambdaContext context)
        {
            try
            {
                context.Logger.LogLine($"Retrieving menu");

                var name = "menu";
                if (request.PathParameters != null && request.PathParameters.ContainsKey("name"))
                {
                    name = request.PathParameters["name"];
                }

                var menuJson = await GetMenu(name);

                return new APIGatewayProxyResponse
                {
                    StatusCode = 200,
                    Body = menuJson
                };
            }
            catch (Exception ex)
            {
                return new APIGatewayProxyResponse
                {
                    StatusCode = 500,
                    Body = ex.ToString()
                };
            }
        }

        private async Task<string> GetMenu(string name)
        {
            var response = await S3Client.GetObjectAsync(BUCKET, name + ".json");
            using (var reader = new StreamReader(response.ResponseStream))
            {
                return await reader.ReadToEndAsync();
            }
        }
    }
}
```

## Step 8: Deploy the API

Although we've been testing our API in the console, it is not yet deployed. In this step, you'll deploy the API so that it can be accessed over the Internet. 

**Be security-conscious:** since we are not securing our API in this tutorial, do not leave the API published for very long. The final step in this tutorial deletes the AWS artifacts.

1. In the AWS console, navigate to API Gateway > APIs. Click on the **menu** API to view its detail.

2. Select **/** in the left panel.

3. From the Actions menu, select **Deploy API**.

    A. Deployment stage: select **[New Stage]**.

    B. Stage name: **test**.

    ![08-deploy.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647825926816/ZpDETWHKZ.png)

    C. Click **Deploy**. Your API is deployed, and you see a page full of details about it.

    ![08-deploy-02.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647826287203/kcZNqkpZ7.png)

    D. An **Invoke URL** is displayed at top. Copy the URL to the clipboard.

    E. In a browser, browse to the URL, adding "breakfast" to the end of the path. Your breakfast.json should appear. 

    ![08-test-browser.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1647826168027/P06nNLL2h.png)

Congratulations! You've now accessed your API publicly and seen it retrieve data from S3. If you like, you can exercise your API using Curl, Postman, or whatever tool you prefer for testing HTTP requests.

## Step 9: Delete AWS Artifacts

When you're finished with the Hello, API Gateway project, follow these steps to shut it down. You don't want to risk unintentional charges for something you're not using. Since we did not secure our API in this tutorial, it's important to not leave it active.

1. In the AWS console, navigate to API Gateway > APIs. Select the **menu** API and choose **Delete** from the Actions menu. Confirm the prompts to delete the API.

2. Navigate to AWS Lambda > Functions. Check the boxes next to the **MenuGet** and **MenuPut** functions, then select **Delete** from the Actions dropdown. Confirm the prompts to delete the functions.

3. Navigate to Amazon S3. Select your menu bucket. Delete each object in the bucket, and then the bucket itself. Confirm the prompts.

# Where to Go From Here

Amazon API Gateway makes it easy to define APIs and connect them to your back-end systems. In this tutorial, you defined two Lambda functions to store and retrieve a menu, then defined an API that connected them to PUT and GET methods. After that, you added a REST path parameter for menu name and updated those Lambda functions to use the parameter when present. As we did so, you received some exposure to the API Gateway UI for defining APIs.

We did not cover API security in this tutorial, and you'll want to get that in place before doing anything more than a Hello, Cloud. You'll want to understand the differences between HTTP, REST, and Websocket APIs, explore API Gateway's features, and understand how to observe and monitor your APIs.

# Further Reading

AWS Documentation

[Amazon API Gateway](https://aws.amazon.com/api-gateway/)

[Amazon API Gateway Resources](https://aws.amazon.com/api-gateway/resources/)

[Announcing HTTP APIs for Amazon API Gateway](https://aws.amazon.com/blogs/compute/announcing-http-apis-for-amazon-api-gateway/)

[Announcing WebSocket APIs in Amazon API Gateway](https://aws.amazon.com/blogs/compute/announcing-websocket-apis-in-amazon-api-gateway/)

[Controlling and managing access to A REST API in API Gateway](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-control-access-to-api.html)

[Integrate a REST API with an Amazon Cognito user pool](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-enable-cognito-user-pool.html)

[Control access to a REST API using Amazon Cognito user pools as authorizer](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-integrate-with-cognito.html)

Reference

[Forbes: The History and Rise of APIs](https://www.forbes.com/sites/forbestechcouncil/2020/06/23/the-history-and-rise-of-apis/?sh=25e5267145c2)

[Microservice Architecture: Pattern: API Gateway / Backends for Frontends](https://microservices.io/patterns/apigateway.html)

Videos

[Amazon API Gateway - How to Build HTTP APIs](https://www.youtube.com/watch?v=43DQm2ObWSU&t=1469s) by Rahul Nath

[An Introduction to AWS API Gateway with AWS Lambda (.NET Core 3.1 Application](https://www.youtube.com/watch?v=ad2md33t1U0) by DotNetCore Central

[Secure Amazon API Gateway REST API using Lambda Authorizer - C# and .NET Core 3.1](https://www.youtube.com/watch?v=W0t0MeFSNDk)

Blogs

[Amazon API Gateway for the .NET Developer - How to Build HTTP APIs](https://www.rahulpnath.com/blog/amazon-api-gateway-http-apis/) by Rahul Nath

[Hello, Cloud blog series](https://davidpallmann.hashnode.dev/series/hello-cloud)
