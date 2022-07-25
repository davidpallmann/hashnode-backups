## Hello, App Runner VPC Connector!

#### This episode: AWS App Runner and VPC Connectors. In this Hello, Cloud blog series, we're covering the basics of AWS cloud services for newcomers who are .NET developers. If you love C# but are new to AWS, or to this particular service, this should give you a jumpstart.

In this post we'll introduce App Runner's VPC Connector feature and use it in a "Hello, Cloud" .NET program to access a DynamoDB table. We'll do this step-by-step, making no assumptions other than familiarity with C# and Visual Studio. We're using Visual Studio 2022 and .NET 6.

# VPC Connector : What is it, and why use It?

> "Eventually everything connects – people, ideas, objects. The quality of the connections is the key to quality per se." —Charles Eams 

AWS App Runner is a fully-managed, easy to use service for hosting web applications. We first took a look at it in [Hello, App Runner](https://davidpallmann.hashnode.dev/hello-app-runner). By default, an App Runner application can only access public endpoints. What if your App Runner site needs to connect to another AWS service, such as RDS, DynamoDB, or S3? App Runner's VPC Connector is the feature that makes this possible, and it can do so without the need to expose those services publicly.

A [VPC Connector](https://docs.aws.amazon.com/apprunner/latest/dg/network-vpc.html) associates your App Runner service with a Virtual Private Cloud (VPC) by creating a VPC endpoint. If you also set up VPC endpoints for other service(s) you want to access, your App Runner service can access them.

![diagram.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1658631012495/UNlAtdyV-.png align="left")

Although hosting a web application in App Runner is extremely easy, and doesn't require an understanding of AWS infrastructure, that changes when you need to connect to other AWS services. You'll have to know something about VPC, subnets, and security groups, plus familiarity with the other service(s) you want to connect to.

# Our Hello, App Runner VPC Connector Project

Our Hello, Cloud project uses App Runner and DynamoDB. We're going to create a .NET web API project, which generates a mock weather forecast API. We'll then upgrade that to read weather data from a DynamoDB table. We'll create our App Runner service with a VPC connector, and a VPC endpoint for our DynamoDB table, allowing them to connect. 

![01-aws-create-table-2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1658671812064/ib6vpeAyb.png align="left")

![05-aws-create-apprunnner-5.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1658672001224/TVkVSmm4d.png align="left")

![08-test-2-dallas.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1658741710762/wJb3arsjh.png align="left")

[source code](https://github.com/davidpallmann/HelloAppRunnerVpc)

## One-time Setup

For any of the tutorials in the Hello, Cloud series you need the following:

1. An  [AWS account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/), and an understanding of what is included in the  [AWS Free Tier](https://aws.amazon.com/premiumsupport/knowledge-center/what-is-free-tier/). 
2. [Microsoft Visual Studio 2022](https://visualstudio.microsoft.com/). If you're using an older version of Visual Studio you won't be able to use .NET 6. If you use a different IDE, you may have to find alternatives to some tutorial steps.
3. [AWS Toolkit for Visual Studio](https://aws.amazon.com/visualstudio/). You'll need to [configure the toolkit ](https://docs.aws.amazon.com/toolkit-for-visual-studio/latest/user-guide/keys-profiles-credentials.html) to access your AWS account and create an IAM user. Your default AWS profile will be linked to this user when running programs from the command line.

## Step 1: Create DynamoDB table

In this step, you'll create a DynamoDB table named Weather and create some data records.

1. In a browser, sign in to the AWS management console.

2. At top right, set the region you want to work in. Choose a region that [supports App Runner](https://docs.aws.amazon.com/general/latest/gr/apprunner.html) and [DynamoDB](https://docs.aws.amazon.com/general/latest/gr/ddb.html). We're using us-east-1 (N. Virginia).

3. Navigate to **Amazon DynamoDB** and click **Create table**:

    A. Table name: **Weather**.

    B. Partition key: **Location**. 

    C. Sort key: **Timestamp**.

    D. Click **Create table**.

    ![01-aws-create-table.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1658637228243/FAWrR0n8e.png align="left")

4. Click on the **Weather** table name to get to its detail page, then click **Explore table items**.

5. **Create item** and add an item with the following attributes: Location **Dallas**, Timestamp **2022-07-23T06:00:00**. Click **Create item**. Click the record to add more attributes. Click **Add new attribute** to add attributes `TempC`(Number) **33**, `TempF` (Number) **92**, and `Summary` (String) **Hot**. Click **Create item**. Finally, click **Save changes**.

    ![01-aws-add-item.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1658638311316/aJAyPSO0P.png align="left")

6. Add more items. Add each item by choosing **Duplicate Item** from the Actions dropdown.

    A. Create an item with attributes `Location` **Dallas**, `Timestamp` **2022-07-23T12:00:00**, `TempC` **43**, `TempF` **109**, and `Summary` **Scorching**. 

    B. Create an item with attributes `Location` **Dallas**, `Timestamp` **2022-07-23T18:00:00**, `TempC` **36**, `TempF` **97**, and `Summary` **Hot**.

    C. Create an item with attributes `Location` **Minneapolis**, `Timestamp` **2022-07-23T06:00:00**, `TempC` **13**, `TempF` **56**, and `Summary` **Cool**.

    D. Create an item with attributes `Location` **Minneapolis**, `Timestamp` **2022-07-23T12:00:00**, `TempC` **22**, `TempF` **72**, and `Summary` **Balmy**.

    E. Create an item with attributes `Location` **Minneapolis**, `Timestamp` **2022-07-23T18:00:00**, `TempC` **19**, `TempF` **67**, and `Summary` **Balmy**.

    ![01-aws-create-table-2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1658638641980/xNRNj7RGs.png align="left")

## Step 2: Create .NET Web API project

In this step, you'll use the `dotnet new` command to create a Web API project, and update its code to retrieve data from the DynamoDB table.

1. Open a command/terminal window and CD to a development folder.

2. Run the `dotnet new` command below to create a new Web API project named `HelloAppRunnerVpc`.

   ```dos
dotnet new webapi -n HellpAppRunnerVpc
```

    ![06-dotnet-new.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1658635797362/Aquj0fLeh.png align="left")

3. Open the HelloAppRunnerVpc project in Visual Studio 2022.

4. The generated project is a WeatherForecast API, very commonly used in .NET samples. To try it, press F5 and test it with Swagger. You'll see the service has a /WeatherForecast action that returns mock weather data JSON. Stop the program from running.

    ![02-test-local-unchanged.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1658636352701/NRCYi3QRd.png align="left")

5. In AWS Explorer, set the same region you chose in Step 1.

6. In Solution Explorer, right-click the `HelloAppRunnerVpc` project and select **Manage NuGet Packages...**. Search for and install the **AWSSDK.DynamoDBv2** package.

7. Open Program.cs in the code editor, and remove or comment out the `app.UseHttpsRedirection();` statement.

6. Open`WeatherForecast.cs` and replace it with the code below at the end of this step.

7. Open `WeatherForecastController.cs` in the `Controllers` folder, and replace with the code below. Replace [region] with the region you are working in. This code implements a health check method at the root of the service, and a WeatherForecast method at /WeatherForecast that takes a location parameter and retrieves data for it from the DynamoDB `Weather` table. It performs a table scan to find records whose partition key matches the location. The results are output as an array of JSON records.

8. Save your changes and ensure the project builds.

WeatherForecast.cs

```csharp
namespace HelloAppRunnerVpc;

public class WeatherForecast
{
    public DateTime Date { get; set; }

    public int TemperatureC { get; set; }

    public int TemperatureF { get; set; }

    public string? Summary { get; set; }
}

```
WeatherForecastController.cs

```csharp
using Amazon;
using Amazon.DynamoDBv2;
using Amazon.DynamoDBv2.DocumentModel;
using Microsoft.AspNetCore.Mvc;

namespace HelloAppRunnerVpc.Controllers;

[ApiController]
[Route("")]
public class WeatherForecastController : ControllerBase
{
    static readonly RegionEndpoint region = RegionEndpoint.[region];

    private readonly ILogger<WeatherForecastController> _logger;

    public WeatherForecastController(ILogger<WeatherForecastController> logger)
    {
        _logger = logger;
    }

    [HttpGet("")]
    public string GetHealthcheck()
    {
        return "Healthcheck: Healthy";
    }


    [HttpGet("WeatherForecast")]
    public async Task<IEnumerable<WeatherForecast>> GetWeatherForecast(string location = "Dallas")
    {
        List<WeatherForecast> forecasts = new List<WeatherForecast>();

        try
        {
            _logger.LogInformation($"00 enter GET, location = {location}");

            var client = new AmazonDynamoDBClient(region);
            Table table = Table.LoadTable(client, "Weather");

            var filter = new ScanFilter();
            filter.AddCondition("Location", ScanOperator.Equal, location);

            var scanConfig = new ScanOperationConfig()
            {
                Filter = filter,
                Select = SelectValues.SpecificAttributes,
                AttributesToGet = new List<string> { "Location", "Timestamp", "TempC", "TempF", "Summary" }
            };

            _logger.LogInformation($"10 table.Scan");

            Search search = table.Scan(scanConfig);

            List<Document> matches;
            do
            {
                _logger.LogInformation($"20 table.GetNextSetAsync");
                matches = await search.GetNextSetAsync();
                foreach (var match in matches)
                {
                    forecasts.Add(new WeatherForecast
                    {
                        Date = Convert.ToDateTime(match["Timestamp"]),
                        TemperatureC = Convert.ToInt32(match["TempC"]),
                        TemperatureF = Convert.ToInt32(match["TempF"]),
                        Summary = Convert.ToString(match["Summary"])
                    });
                }
            } while (!search.IsDone);

            _logger.LogInformation($"30 exited results loop");

        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "90 Exception");
        }

        _logger.LogInformation($"99 returning {forecasts.Count} results");

        return forecasts.ToArray();
    }
}
```


## Step 3: Test locally

In this step, you'll test the Web API locally and confirm data retrieval from DynamoDB.

1. Press F5 and wait for the app to build and launch in a browser. 

2. In the browser, remove the Swagger path from the URL to hit the service root, and you should see a health check message. App Runner will regularly ping the site to check health.

    ![03-test-02.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1658638935434/_GCuwr4hM.png align="left")

3. Add **/WeatherForecast?location=Dallas** to the end of the URL path. You should see weather forecast data JSON appear, with values you created in the DynamoDB table in Step 1.

    ![03-test.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1658638830738/g2oC4BJTt.png align="left")

4. Change the URL path to end in **/WeatherForecast?location=Minneapolis**. Now you see figures for that city.

     ![08-test-2-dallas.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1658741723640/4qvcsItKx.png align="left")

5. Try another location name, and you see an empty response because there is no data for it in the table.

6. Stop the program from running.

Although it was straightforward for our app to access DynamoDB when testing locally, that won't be the case in the cloud, because App Runner is restricted to public endpoints by default. We'll have to take the extra steps of adding a VPC connector for App Runner and a matching VPC endpoint for DynamoDB.

## Step 4: Publish Web API project to ECR

In this step, you'll use Publish to AWS to containerize your project and push the container to Amazon Elastic Container Registry (ECR).

1. In Visual Studio, set the region in AWS Explorer to the region you want to work in. We're using us-east-1 (N. Virginia). Be sure to select a region that supports both App Runner and DynamoDB.

2. In Solution Explorer, right-click the HelloAppRunnerVpc project and select **Add > Docker Support**, Target OS **Linux**. A Dockerfile is added to the project.

3. Again right-click the HelloAppRunnerVpc project, and select **Publish to AWS**.

4. In the Publish to AWS wizard, select the Publish to New Target tab and select **Container Image to Amazon Elastic Container Registry (ECR)**.

    ![04-publish-to-aws-ecr.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1658661757519/hxG6L3asX.png align="left")

5. Click the **Edit settings** button at right, and set the container image tag to **latest**.

    ![04-publish-to-aws-ecr-settings.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1658661874081/T9mS3rpb8.png align="left")

6. Click **Publish**, then review and confirm the target and region. Wait for the container to be created and deployed, which will take a couple of minutes.

    ![04-publish-to-aws-ecr-done.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1658662272166/ZM8TfWBgv.png align="left")

## Step 5: Create IAM Role

In this step, you'll use the AWS console to create an IAM role allowing the App Runner EC2 instances to access the DynamoDB table.

1. Browse to the AWS management console, sign in. At top right, select the same region you used in Step 4.

2. Create a policy that allows access to the `Weather` DynamoDB table:

    A. Navigate to **Identity and Access Management (IAM)**

    B. Select **Policies** from the left pane and click **Create policy**.

    C. Create the policy and enter the JSON below at the end of this step, replacing [account] with your 12-digit AWS account number, and [region] with your region (we used `us-east-1`). 

    D. Click **Next: Tags** and **Next: Review**. Name the policy **ddb-Weather** and click **Create policy**.

    ![05-aws-create-policy.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1658634306892/oC3xZoAav.png align="left")

    ![05-aws-create-policy-2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1658634316893/OQQBtJ41r.png align="left")

3. Create a role for App Runner EC2 instances:

    A. Select **Roles** from the left pane and click **Create role**.

    B. For Trusted entity type, select **AWS service**.

    C. For Use case, select **EC2** and click **Next**.

    ![05-aws-create-role.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1658634715384/qgk_s2QXc.png align="left")

    D. Search for and select these permissons: **ddb-Weather**, **AmazonDynamoDBFullAccess**, and **AWSAppRunnerFullAccess**. Then Click **Next**.

    ![05-aws-create-role-2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1658671572341/TnNj_4A21.png align="left")

    E. For Trust relationships, replace with the JSON below at the end of this step.

    F. Name the role `AppRunnerInstanceDynamoDB` and click **Create role**.

    ![05-aws-create-role-3.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1658635074695/GCP4W0SY9.png align="left")

JSON for policy ddb-weather

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "",
            "Effect": "Allow",
            "Action": "dynamodb:*",
            "Resource": "arn:aws:dynamodb:[region]:[account]:table/Weather"
        }
    ]
}
```

Trust relationships JSON for role AppRunnerInstanceDynamoDB

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": [
                    "ec2.amazonaws.com",
                    "tasks.apprunner.amazonaws.com"
                ]
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
```

## Step 6: Create App Runner service

In this step, you'll create the App Runner service and a VPC connector.

1. In the AWS console, navigate to **AWS App Runner** and click **Create service**.

    A. Repository type: **Container registry**.

    B. Provider: **Amazon ECR**.

    C. Click **Browse** and select the container you deployed to ECR in Step 4.

    ![05-browse-container.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1658663118750/0rueY-3Kj.png align="left")

    D. For Deployment settings trigger, select **Automatic**.

    E. For ECR access role, select **Create new service role**.

    F. Click **Next**.

    ![05-aws-create-apprunnner-1.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1658663223068/FWZSroI-D.png align="left")

2. On the Configure service page,

    A. Service name: **HelloAppRunnerVpc**.

    B. Port: **80**.

    ![05-aws-create-apprunnner-2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1658663416245/UHgjF0AGH.png align="left")

3. Expand the Security section and set Instance role to **AppRunnerInstanceDynamoDB**.

    ![05-aws-create-apprunnner-2a.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1658667351110/I6ofvdbNX.png align="left")

4. Expand the Networking section and create a VPC connector:

    A. Under Networking, select **Custom VPC**.

    B. Under VPC connector, click **Add new**.

    C. VPC connector name: **AppRunnerDynamoDB**.

    D. VPC: select your default VPC.

    E. Subnets: select all subnets.

    F. Security groups: select your default security group.

    ![05-aws-add-vpc-connector.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1658663747240/n9xZC5Qnq.png align="left")

    G. Click **Add**. If you get an error message that one of your subnets doesn't support App Runner services, remove it from the Subnets list and click **Add** again.

    ![05-aws-create-apprunnner-3.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1658671232487/ge_OiW9NV.png align="left")

    H. Click **Next** and **Create & deploy**.

5. Wait for deployment, which will take several minutes. This is a good time for a bio break.

    ![05-aws-create-apprunnner-4.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1658671105150/MMIh_jjeq.png align="left")

6. When service deployment is complete, you'll see a *Create service succeeded* message. Record the default domain URL. Refresh the Event log, and you should see confirmation the service is running.

    ![05-aws-create-apprunnner-5.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1658671021327/LL5FIGWsI.png align="left")

7. Browse to the URL, and you should see your health check. Our service is hosted in App Runner, but it isn't yet able to access DynamoDB. We've created the App Runner VPC connector, but we still need to create a matching VPC endpoint for DynamoDB.

    ![08-test-1-healthcheck.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1658670861707/CErsksrrd.png align="left")

## Step 7: Create a VPC Endpoint for DynamoDB

In this step, you'll create a VPC endpoint for DynamoDB.

1. In the AWS console, navigate to **VPC**.

2. Select **Endpoints** from the left panel and click **Create endpoint**.

3. Name: **vpc-endpoint-dynamodb**.

4. Service category: select **AWS service**.

5. Service: enter **DynamoDB** in the search box and select **com.amazonaws.region.dynamodb**.

    ![07-aws-create-endpoint.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1658664891670/t-V8KsYAB.png align="left")

6. VPC: select your default VPC.

7. Route tables: select the Main route table.

8. Click **Create endpoint**.

    ![07-aws-create-endpoint-2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1658665100251/5Ke_lC3tQ.png align="left")

## Step 8: Test in the Cloud

Now we're ready to put it all together and test our Web API in the cloud.

1. In a browser, visit the service URL you recorded in Step 6. You see the health check response.

    ![08-test-1-healthcheck.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1658668045050/LJv5glOjB.png align="left")

2. Add **/WeatherForecast?location=Dallas** at the end of the path. Now you see records from Dallas that you entered in Step 1. 

    ![08-test-2-dallas.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1658741737297/-D4N4S2NU.png align="left")

3. Change the end of the path to **/WeatherForecast?location=Minneapolis**, and you see records for that location.

    ![08-test-3-minneapolis.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1658741810703/_kV5Jw5r-.png align="left")

    Congratulations! Your App Runner service is talking to DynamoDB!

## Step 9: Shut it Down

When you're all done with the project, shut it down. You don't to accrue charges for something you're not using.

1. In the AWS console, navigate to **App Runner** and delete the `HelloAppRunnerVpc` service.

2. Navigate to **DynamoDB** and delete the `Weather` table.

3. Navigate to **ECR** and delete the `helloapprunnervpc` container image.

# Where to Go From Here

App Runner is really easy to get started with. When you start needing to integrate to other AWS services, it's time to start learning about AWS infrastructure concepts. In this tutorial, you hosted a Web API in App Runner and configured access to DynamoDB. To achieve that, you had to create a policy granting permission to a DynamoDB table, a role for App Runner's EC2 instances, a VPC connector for the App Runner service, and a VPC endpoint for DynamoDB. You encountered subnets and security groups.

This tutorial did not go over CDK infrastructure-as-code for VPC endpoints, something you'll want to explore for a real project. To go further, think about what other services you might want to integrate with your App Runner applications. Perhaps S3 storage, or a different database. Read about the services, understand their networking defaults and options, and build something.

# Further Reading

AWS Documentation

[AWS App Runnner](https://aws.amazon.com/apprunner/)

[New for App Runner - VPC Support](https://aws.amazon.com/blogs/aws/new-for-app-runner-vpc-support/)

[Enabling VPC access for outgoing traffic](https://docs.aws.amazon.com/apprunner/latest/dg/network-vpc.html)

[Using App Runner with VPC endpoints](https://docs.aws.amazon.com/apprunner/latest/dg/network-vpce.html)

[Dive Deep on AWS App Runner VPC Networking](https://aws.amazon.com/blogs/containers/deep-dive-on-aws-app-runner-vpc-networking/)

[AWS SDK for .NET Documentation](https://docs.aws.amazon.com/sdk-for-net/index.html)

Videos

[AWS App Runner VPC Support - Launch Demo](https://www.youtube.com/watch?v=o8tvYG9uIBY)

[App Runner VPC Integration demo using the AWS CDK](https://www.youtube.com/watch?v=EX8mu-WOQpc)

Blogs

[Serverless on AWS with CDK #1: App Runner with VPC Integration](https://dev.to/maxritter/serverless-on-aws-with-cdk-1-app-runner-with-vpc-integration-3k5l)

[Deploy .NET webapp on AWS App Runner with VPC Connector](https://menalb.hashnode.dev/deploy-net-webapp-on-aws-app-runner-with-vpc-connector)

[Hello, Cloud blog series](https://davidpallmann.hashnode.dev/series/hello-cloud)