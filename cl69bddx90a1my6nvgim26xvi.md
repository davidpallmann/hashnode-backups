## Hello, Deployment Projects!

#### This episode: AWS .NET deployment projects. In this Hello, Cloud blog series, we're covering the basics of AWS cloud services for newcomers who are .NET developers. If you love C# but are new to AWS, or to this particular service, this should give you a jumpstart.

In this post we'll introduce the deployment projects feature of AWS deployment tools for .NET CLI and use it to customize IaC for a "Hello, Cloud" .NET program. We'll do this step-by-step, making no assumptions other than familiarity with C# and Visual Studio. We're using Visual Studio 2022 and .NET 6.

# AWS .NET Deployment Projects : What are they, and why use them?

> "As you have more resources in life, it's your obligation to deploy those for the benefit of others." —Stephen Schwarzman

The AWS deployment tool for .NET CLI gives you an easy way to build and deploy .NET applications and services to AWS with the `dotnet aws deploy` command. We've previously looked at it in [Hello, .NET Deploy!](https://davidpallmann.hashnode.dev/hello-net-deploy), and the equivalent feature in the AWS Toolkit for Visual Studio in [Hello, Publish to AWS!](https://davidpallmann.hashnode.dev/hello-publish-to-aws). It's a guided, automatic experience. Although Cloud Development Kit (CDK) code is automatically generated, you don't get to see it or change it by default: a recipe supplied by AWS governs the infrastructure-as-code for your deployment target. 

The guided deployment experience is very nice, but what if you need to customize it? What if you need to add a database, add permissions to a role, or override a property in the generated CDK? [Deployment projects](https://aws.github.io/aws-dotnet-deploy/docs/project/) is the feature that allows you to modify the CDK. The deployment tool can generate a C# deployment project, which you can change to customize the resources deployed to AWS. 

![diagram-dotnet-commands.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659209402624/6JE4Guekl.png align="left")

You generate a deployment project by running the command `dotnet aws deployment-project generate` in your application project folder. The tool prompts you to select your deployment target, just as it does during deployment. It then generates a C# CDK deployment project.

![dotnet-deployment-project-generate.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659209599042/Yin9NtVET.png align="left")

By default, the tool generates a deployment project in a sibling folder, using the same name as your application project with `.Deployment` added to the end. You can override those behaviors with the  `--project-display-name name` and `--output` options.

# Our "Hello, Deployment Projects" Project

We will create a simple Razor website that looks up significant events for this day in history from a DynamoDB table and displays them. We'll be hosting the site in AWS App Runner. After we develop the web project, we'll generate a deployment project and modify it so the App Runner service can access the DynamoDB table. We'll be using the same configuration used in [Hello, VPC Connector!](https://davidpallmann.hashnode.dev/hello-app-runner-vpc-connector#heading-step-5-create-iam-role), but this time we'll do it in the CDK deployment project. Our updates to the CDK will create a DynamoDB table, a VPC endpoint for DynamoDB, a VPC connector for App Runner, a policy granting access to the DynamoDB table, and modifications to the App Runner instance role. The only thing we'll do manually in the AWS console is add data records to the table.

![dotnet-aws-deploy-5.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659270946847/Cqnqv6QfS.png align="left")

![after-aws-dynamodb-table.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659207800481/qVQKtf9JU.png align="left")

![after-aws-apprunner-service.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659207821456/IEv6Z2au8.png align="left")

![test-browser-12-07.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659207754701/T-J3VPT7p.png align="left")

[source code](https://github.com/davidpallmann/hello-deployment-projects)

## One-time Setup

For any of the tutorials in the Hello, Cloud series you need the following:

1. An [AWS account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/), and an understanding of what is included in the [AWS Free Tier](https://aws.amazon.com/premiumsupport/knowledge-center/what-is-free-tier/). 
2. [Microsoft Visual Studio 2022](https://visualstudio.microsoft.com/). If you're using an older version of Visual Studio you won't be able to use .NET 6. If you use a different IDE, you may have to find alternatives to some tutorial steps.
3. [AWS Toolkit for Visual Studio](https://aws.amazon.com/visualstudio/). You'll need to [configure the toolkit ](https://docs.aws.amazon.com/toolkit-for-visual-studio/latest/user-guide/keys-profiles-credentials.html) to access your AWS account and create an IAM user. Your default AWS profile will be linked to this user when running programs from the command line.

## Step 1: Create a Web Application

In this step, you'll create a .NET web application

1. Open a command/terminal window and CD to a development folder.

2. Run the `dotnet new` command below to create a web application named **hello-apprunner**.

    ```dos
dotnet new webapp –n hello-apprunner
```

3. Open the hello-apprunner project in Visual Studio.

4. In Solution Explorer, right-click the `hello-apprunner` project and select **Manage NuGet Packages...**. Search for and install the **AWSSDK.DynamoDBv2** package.

5. Open `Index.html.cs`, the main page code-behind file, in the code editor and replace with the code below at the end of this step. This code has an `OnGet` method that expects a `day=mm/dd` parameter indicating which day to show information for. If omitted, it defaults to today. The code searches a DynamoDB table named `OnThisDate` for all records with a partition key of mm/dd and stores their Title property, which will be text of the form "1941 Attack on Pearl Harbor". We store a heading and the retrieved titles in the page's model as properties `Heading` and `Items`.

6. Open `Index.html` in the code editor and replace with the code at the end of this step. The code displays the model properties `Heading` and `Items`.

7. Save your changes and ensure the program can build. Since the DynamoDB table does not yet exist, we won't be able to test this locally.

Index.html.cs

```csharp
using Amazon;
using Amazon.DynamoDBv2;
using Amazon.DynamoDBv2.DocumentModel;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;

namespace hello_apprunner.Pages;

public class IndexModel : PageModel
{
    static readonly RegionEndpoint region = RegionEndpoint.USWest2;

    private readonly ILogger<IndexModel> _logger;

    public string Heading { get; internal set; }
    public List<string> Items { get; } = new List<string>();

    public IndexModel(ILogger<IndexModel> logger)
    {
        _logger = logger;
    }

    public async Task<IActionResult> OnGet(string day = null)
    {
        try
        {
            _logger.LogInformation($"00 enter GET, day = {day}");

            if (day == null)
            {
                day = DateTime.Today.ToString("MM/dd");
            }

            Heading = $"What happened on {day}?";

            var client = new AmazonDynamoDBClient(region);
            Table table = Table.LoadTable(client, "OnThisDate");

            var filter = new ScanFilter();
            filter.AddCondition("Day", ScanOperator.Equal, day);

            var scanConfig = new ScanOperationConfig()
            {
                Filter = filter,
                Select = SelectValues.SpecificAttributes,
                AttributesToGet = new List<string> { "Title" }
            };

            _logger.LogInformation($"10 table.Scan");

            Search search = table.Scan(scanConfig);

            do
            {
                _logger.LogInformation($"20 table.GetNextSetAsync");
                var matches = await search.GetNextSetAsync();
                foreach (var match in matches)
                {
                    Items.Add(Convert.ToString(match["Title"]));
                }
            } while (!search.IsDone);

            _logger.LogInformation($"30 exited results loop");
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "90 Exception");
            Heading = $"Error: {ex.Message}";
        }
        return Page();
    }
}
```
Index.html

```
@page
@model IndexModel
@{
    ViewData["Title"] = "On This Date";
}

<div class="text-center">
    <h1 class="display-4">On This Date</h1>
    <p>@Model.Heading</p>
    @if (@Model.Items.Count == 0)
    {
        <p>Sorry, we have no data about anything interesting happening on this day. Check back tomorrow!</p>
    }
    else
    {
        foreach(var item in @Model.Items)
        {
            <p>@item</p>
        }
    }
</div>
```

## Step 2: Generate a Deployment Project and Update the CDK.

In this step, you'll generate a deployment project and get familiar with it.

1. In the command/terminal window, cd to the `hello-apprunner` folder.

2. Run the command below to generate a deployment project.

    ```dos
dotnet aws deployment-project generate
```

3. Select the option number corresponding to AWS App Runner.

4. Confirm the prompt to continue by entering **y**.

5. A hello-apprunner.Deployment project should now exist in a neighboring folder.

    ![folders-projects.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659211065423/qdysqk6Un.png align="left")

6. Open the deployment project in Visual Studio and examine the project structure. 

    ![deployment-project-structure.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659211651438/Q8fATnduu.png align="left")

    A. This is a CDK project, and has the `Amazon.CDK.Lib` and `AWS.Deploy.Recipes.CDK.Common` packages installed.

    B. Examine the project files in Solution Explorer. The `Generated` folder contains generated code. Open Recipe.cs in the code editor. This is CDK generated from a deployment recipe, the default IaC code. You won't ever modify this directly, but will work in another file where you can extend or override this code.

    ![deployment-project-recipe.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659212245831/BZ2V2tNeo.png align="left")

    C. Open AppStack.cs in the code editor. This is where you can modify the CDK. We'll be working here in the next step.

    ![deployment-project-appstack-generated.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659212194029/kiI0g5qMD.png align="left")

One other thing to note: In addition to generating this new deployment project, the tool also modified the`hello-apprunner` web project. An `aws-deployments.json` file was added listing that references this deployment project. As a result, future deployments from the application project with `dotnet aws deploy` will be aware of this deployment project and offer it as a destination.

## Step 3. Extend the CDK to Create DynamoDB, VPC, and IAM Resources

In this step, you'll add new CDK statements to the deployment project to create a DynamoDB table and resources enabling App Runner to access the table. That will include a VPC endpoint for DynamoDB, a VPC connector for App Runner, and IAM artifacts.

1. In Visual Studio, where the deployment project should already be open, open AppStack.cs in the code editor.

2. In the code, you'll be inserting a lengthy chunk of code below the `// Create custom CDK constructs here...` comment block (about line 35), and will also be adding some code after the `Create additional CDK constructs here` block (about line 41). You can replace all of the code with the code at the end of this step, or add it block by block by following the code walkthrough below.

3. Save changes and confirm the project builds.

AppStack.cs

```csharp
// Copyright Amazon.com, Inc. or its affiliates. All Rights Reserved.
// SPDX-License-Identifier: Apache-2.0

using System;
using System.Collections.Generic;
using Amazon.CDK;
using Amazon.CDK.AWS.AppRunner;
using Amazon.CDK.AWS.IAM;
using AWS.Deploy.Recipes.CDK.Common;
using hello_apprunner.Deployment.Configurations;
using Amazon.CDK.AWS.ECR;
using Amazon.CDK.AWS.ECS;
using Amazon.CDK.AWS.DynamoDB;
using Amazon.CDK.AWS.EC2;

using CfnService = Amazon.CDK.AWS.AppRunner.CfnService;
using CfnServiceProps = Amazon.CDK.AWS.AppRunner.CfnServiceProps;
using Constructs;

namespace hello_apprunner.Deployment
{
    public class AppStack : Stack
    {
        private readonly Configuration _configuration;

        internal AppStack(Construct scope, IDeployToolStackProps<Configuration> props)
            : base(scope, props.StackName, props)
        {
            _configuration = props.RecipeProps.Settings;

            // Setup callback for generated construct to provide access to customize CDK properties before creating constructs.
            CDKRecipeCustomizer<Recipe>.CustomizeCDKProps += CustomizeCDKProps;

            // Create custom CDK constructs here that might need to be referenced in the CustomizeCDKProps. For example if
            // creating a DynamoDB table construct and then later using the CDK construct reference in CustomizeCDKProps to
            // pass the table name as an environment variable to the container image.

            // Get default VPC

            var defaultVpc = Vpc.FromLookup(this, "default-vpc", new VpcLookupOptions { IsDefault = true });

            // Create VPC Connector

            var securityGroup = new SecurityGroup(this, "sg-vpc-connector", new SecurityGroupProps
                {
                    AllowAllOutbound = true,
                    Vpc = defaultVpc
                });

            securityGroup.AddIngressRule(
                Peer.AnyIpv4(),
                Port.AllTraffic()
            );

            securityGroup.AddEgressRule(
                Peer.AnyIpv4(),
                Port.AllTraffic()
            );

            var vpcConnector = new CfnVpcConnector(this, "apprunner-vpc-conn",
               new CfnVpcConnectorProps
               {
                   VpcConnectorName = "apprunner-vpc-connector",
                   Subnets = defaultVpc.SelectSubnets(null).SubnetIds,
                   SecurityGroups = new string[] { securityGroup.SecurityGroupId }
               });

            new CfnOutput(this, "apprunner-vpc-connector", new CfnOutputProps { Value = vpcConnector.LogicalId });

            // Create DynamoDB table

            var tableProps = new TableProps
            {
                RemovalPolicy = RemovalPolicy.DESTROY,
                TableName = "OnThisDate",
                PartitionKey = new Amazon.CDK.AWS.DynamoDB.Attribute
                {
                    Name = "Day",       // 12/07
                    Type = AttributeType.STRING
                },
                SortKey = new Amazon.CDK.AWS.DynamoDB.Attribute
                {
                    Name = "Title",     // 1941 Pearl Harbor Attack
                    Type = AttributeType.STRING
                },
                BillingMode = BillingMode.PAY_PER_REQUEST,
            };
            var table = new Table(this, "OnThisDate", tableProps);

            new CfnOutput(this, "DynamoDB-OnThisDate", new CfnOutputProps{ Value = table.TableName });

            // Create VPC Endpoint for DynamoDB

            var dynamoGatewayEndpoint = defaultVpc.AddGatewayEndpoint("dynamo-gateway-endpoint", new GatewayVpcEndpointOptions 
            {
                Service = GatewayVpcEndpointAwsService.DYNAMODB
            });

            new CfnOutput(this, "dynamo-gateway-endpoint", new CfnOutputProps{ Value = dynamoGatewayEndpoint.VpcEndpointId });

            // Create the recipe defined CDK construct with all of its sub constructs.
            var generatedRecipe = new Recipe(this, props.RecipeProps);

            // Create additional CDK constructs here. The recipe's constructs can be accessed as properties on
            // the generatedRecipe variable.

            // Add IAM policy granting access to DynamoDB table

            var policyDoc = new PolicyDocument(new PolicyDocumentProps()
            {
                Statements = new PolicyStatement[] {
                    new PolicyStatement(new PolicyStatementProps()
                    {
                        Effect = Effect.ALLOW,
                        Actions = new string[] { "dynamodb:*" },
                        Resources = new string[] { table.TableArn }
                    })
                }
            });

            var policyDynamoTable = new ManagedPolicy(this, "dynamodb-on-this-date", new ManagedPolicyProps()
            {
                ManagedPolicyName = "dynamodb-policy-on-this-date",
                Document = policyDoc
            });

            // Add policies to instance role.

            generatedRecipe.TaskRole.AddManagedPolicy(ManagedPolicy.FromAwsManagedPolicyName("AmazonDynamoDBFullAccess"));
            generatedRecipe.TaskRole.AddManagedPolicy(ManagedPolicy.FromAwsManagedPolicyName("AWSAppRunnerFullAccess"));
            generatedRecipe.TaskRole.AddManagedPolicy(policyDynamoTable);

            // Associate VPC Connector with App Runner service

            var appRunner = generatedRecipe.AppRunnerService;
            if (generatedRecipe != null && generatedRecipe.AppRunnerService != null)
            {
                var egressConfig = new CfnService.EgressConfigurationProperty
                {
                    EgressType = "VPC",
                    VpcConnectorArn = vpcConnector.AttrVpcConnectorArn
                };
                var network = new CfnService.NetworkConfigurationProperty
                {
                    EgressConfiguration = egressConfig
                };
                generatedRecipe.AppRunnerService.NetworkConfiguration = network;
            }

        }

        /// <summary>
        /// This method can be used to customize the properties for CDK constructs before creating the constructs.
        ///
        /// The pattern used in this method is to check to evnt.ResourceLogicalName to see if the CDK construct about to be created is one
        /// you want to customize. If so cast the evnt.Props object to the CDK properties object and make the appropriate settings.
        /// </summary>
        /// <param name="evnt"></param>
        private void CustomizeCDKProps(CustomizePropsEventArgs<Recipe> evnt)
        {
            // Example of how to customize the container image definition to include environment variables to the running applications.
            // 
            //if (string.Equals(evnt.ResourceLogicalName, nameof(evnt.Construct.AppRunnerService)))
            //{
            //    if (evnt.Props is CfnServiceProps props)
            //    {
            //        Console.WriteLine("Customizing AppRunner Service");
            //    }
            //}
        }
    }
}
```

### Understand the Code

Let's understand what all this code does:

**Get default VPC** (lines 38-40). When we later create a VPC connector for App Runner, we'll need to reference a VPC. This code looks up the default VPC.

```csharp
// Get default VPC

var defaultVpc = Vpc.FromLookup(this, "default-vpc", new VpcLookupOptions { IsDefault = true });
```

**Create VPC Connector** (42-68). App Runner can't get to other AWS services like DynamoDB unless we connect it to a VPC. The actual VPC connector is created at line 60, but it will want a security group, so we first create one, allowing unrestricted inbound and outbound traffic. When the VPC connector is created, we give it a name, all the subnets from the default VPC (so we have multiple availability zones covered), and the security group we just created. The `new CfnOutput` statement on line 68 adds the VPC connector Id to the CloudFormation output.

```csharp
// Create VPC Connector

var securityGroup = new SecurityGroup(this, "sg-vpc-connector", new SecurityGroupProps
    {
        AllowAllOutbound = true,
        Vpc = defaultVpc
    });

securityGroup.AddIngressRule(
    Peer.AnyIpv4(),
    Port.AllTraffic()
);

securityGroup.AddEgressRule(
    Peer.AnyIpv4(),
    Port.AllTraffic()
);

var vpcConnector = new CfnVpcConnector(this, "apprunner-vpc-conn",
    new CfnVpcConnectorProps
    {
        VpcConnectorName = "apprunner-vpc-connector",
        Subnets = defaultVpc.SelectSubnets(null).SubnetIds,
        SecurityGroups = new string[] { securityGroup.SecurityGroupId }
    });

new CfnOutput(this, "apprunner-vpc-connector", new CfnOutputProps { Value = vpcConnector.LogicalId });
```

**Create DynamoDB table** (70-90). We create a TableProps object that specifies the CloudFormation removal policy, the table name, a partition key ("Day"), a sort key ("Title"), and a billing mode. On line 88 the table is created with these properties.

```csharp
// Create DynamoDB table

var tableProps = new TableProps
{
    RemovalPolicy = RemovalPolicy.DESTROY,
    TableName = "OnThisDate",
    PartitionKey = new Amazon.CDK.AWS.DynamoDB.Attribute
    {
        Name = "Day",       // 12/07
        Type = AttributeType.STRING
    },
    SortKey = new Amazon.CDK.AWS.DynamoDB.Attribute
    {
        Name = "Title",     // 1941 Pearl Harbor Attack
        Type = AttributeType.STRING
    },
    BillingMode = BillingMode.PAY_PER_REQUEST,
};
var table = new Table(this, "OnThisDate", tableProps);

new CfnOutput(this, "DynamoDB-OnThisDate", new CfnOutputProps{ Value = table.TableName });
```

**Create VPC endpoint for DynamoDB** (92-99). The VPC connector is only half the connection from App Runner to Dynamo. This code creates a VPC endpoint for DynamoDB. Now the default VPC serves as a bridge for App Runner and DynamoDB to communicate.

```csharp
// Create VPC Endpoint for DynamoDB

var dynamoGatewayEndpoint = defaultVpc.AddGatewayEndpoint("dynamo-gateway-endpoint", new GatewayVpcEndpointOptions 
{
    Service = GatewayVpcEndpointAwsService.DYNAMODB
});

new CfnOutput(this, "dynamo-gateway-endpoint", new CfnOutputProps{ Value = dynamoGatewayEndpoint.VpcEndpointId });
```

**Create recipe defined CDK with all of its sub constructs** (101-105). This code was generated. Our next additions are below, under the `Create additional CDK constructs here` comment block.

```csharp
// Create the recipe defined CDK construct with all of its sub constructs.
var generatedRecipe = new Recipe(this, props.RecipeProps);

// Create additional CDK constructs here. The recipe's constructs can be accessed as properties on
// the generatedRecipe variable.
```

**Add IAM policy** (107-125). Next, we create an IAM policy granting access to DynamoDB table. The first step is creating a policy document, then creating a managed policy with the policy document.

```csharp
// Add IAM policy granting access to DynamoDB table

var policyDoc = new PolicyDocument(new PolicyDocumentProps()
{
    Statements = new PolicyStatement[] {
        new PolicyStatement(new PolicyStatementProps()
        {
            Effect = Effect.ALLOW,
            Actions = new string[] { "dynamodb:*" },
            Resources = new string[] { table.TableArn }
        })
    }
});

var policyDynamoTable = new ManagedPolicy(this, "dynamodb-on-this-date", new ManagedPolicyProps()
{
    ManagedPolicyName = "dynamodb-policy-on-this-date",
    Document = policyDoc
});
```

**Add policies to instance role** (127-131). An EC2 instance role (called a task role in the CDK project) already exists in the generated CDK, surfaced to us in a property called `TaskRole`. We add the policy we just created, and a few others.

```csharp
// Add policies to instance role.

generatedRecipe.TaskRole.AddManagedPolicy(ManagedPolicy.FromAwsManagedPolicyName("AmazonDynamoDBFullAccess"));
generatedRecipe.TaskRole.AddManagedPolicy(ManagedPolicy.FromAwsManagedPolicyName("AWSAppRunnerFullAccess"));
generatedRecipe.TaskRole.AddManagedPolicy(policyDynamoTable);
```

**Associate VPC Connector with App Runner service** (133-148). We created our VPC connector earlier, but it needs to be associated with our App Runner service. The CDK gives us a `generatedRecipe` property we use to set the necessary properties on the App Runner service, which include the VPC Connector's Amazon Resource Name (ARN).

```csharp
// Associate VPC Connector with App Runner service

var appRunner = generatedRecipe.AppRunnerService;
if (generatedRecipe != null && generatedRecipe.AppRunnerService != null)
{
    var egressConfig = new CfnService.EgressConfigurationProperty
    {
        EgressType = "VPC",
        VpcConnectorArn = vpcConnector.AttrVpcConnectorArn
    };
    var network = new CfnService.NetworkConfigurationProperty
    {
        EgressConfiguration = egressConfig
    };
    generatedRecipe.AppRunnerService.NetworkConfiguration = network;
}
```

## Step 4: Deploy the Web Application

In this step, you'll deploy the web application, which will run your deployment project.

1. In a command/terminal window, CD to the web project folder (not the deployment project).

2. Run `aws configure` and confirm or set the region you want to work in. We're using us-west-2 (Oregon).

3. Run `dotnet aws deploy` to deploy to AWS.

    ```dos
dotnet aws deploy
```

4. The choices offered by the tool now include your deployment project. Enter the number for **Deployment project for hello-apprunner to AWS App Runner.**. 

    ![dotnet-aws-deploy.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659214990662/MdWH-T0li.png align="left")

5. Review the settings displayed, and press Enter to proceed with deployment.

    ![dotnet-aws-deploy-2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659215087952/r7p540RAG.png align="left")

6. Wait for deployment, which will take a few minutes. While it is proceeding, you'll see the web application containerized.

    ![dotnet-aws-deploy-3.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659215191785/mug3EcmE5.png align="left")

    Next, the CloudFormation generated from the CDK deployment project will execute. You'll see the artifacts you added to the CDK listed, such as the DynamoDB table, VPC connector, and IAM artifacts.

    ![dotnet-aws-deploy-4.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659215338197/TfawpM3GR.png align="left")

7. When deployment completes, confirm there were no errors. If errors are reported, note the error details, which are specific and should identify where in the CDK code the problem occurred. Go back and double check your work, then retry deployment.

    ![dotnet-aws-deploy-3-error.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659215430201/xx-Gc1vG7.png align="left")

     A successful deployment will end with a summary of created resources and the App Runner service endpoint.

    ![dotnet-aws-deploy-5.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659270963770/Nzh_Fzqm2.png align="left")

8. Sign in to the AWS console, set the region, and verify the AWS artifacts the deployment created:

    A. Navigate to **AWS App Runner** and select **Services**. You now see a `hello-apprunner-service` listed.

    ![after-aws-apprunner-service.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659215768035/zGPCn2MMm.png align="left")

    B. Click on the App Runner service to view its detail, including the VPC connector details on the Configuration tab. Record the endpoint URL. Now, browse to the endpoint URL. A web page comes up, but no data is listed, because we haven't yet populated the DynamoDB table.

    ![test-browser-empty.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659215953955/5ioDHw7j-.png align="left")

    C. Navigate to **DynamoDB** and select **Tables**. You see a table named **OnThisDate**. 

    ![after-aws-dynamodb-table.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659216029936/7eUaPQBWo.png align="left")

    D. Navigate to **VPC**, then select **Endpoints** from the left pane. You see a VPC endpoint listed for the DynamoDB service of Endpoint type **Gateway**.

    ![after-aws-vpc-endpoint-dynamodb.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659216112061/w6-99Dujr.png align="left")

   Congratulations, your deployment project has created and cross-connected a collection of AWS resources.

## Step 5: Populate DynamoDB Table

In this step, you'll add records to the DynamoDB table so you can see the application work.

1. In the AWS console, navigate to **DynamoDB** and select **Tables** from the left pane.

2. Click the **OnThisDate** table to view its detail, then click **Explore table items**.

3. Use the **Create item** button to add multiple records with the sample data below, or any alternative data you prefer. There are only two fields: Day, which holds mm/dd value such as "12/07", and Title, which holds text of the form "1941 Attack on Pearl Harbor".

| Day | Title |
|-----|----|
| 01/01 | 1804 Haiti declares independence from France |
| 01/01 | 1863 Abraham Lincoln issues the Emancipation Proclamation |
| 01/01 | 1995 World Trade Organization (WTO) established |
| 01/01 | 2002 European Union adopts the Euro monetary unit |
| 01/02 | 1905 Russia surrenders Port Arthur to the Japanese in the Russo-Japanese War |
| 01/02 | 1935 Trial of Bruno Haptmann begins for kidnapping and murder of Charles Lindbergh's infant son |
| 01/03 | 1521 Martin Luther excommunicated by Pope Leo X |
| 01/03 | 1959 Alaska becomes the 49th US state |
| 01/03 | 1962 Pope John XXIII excommunicates Fidel Castro |
| 01/03 | 1977 Apple incorporated by Steve Jobs and Steve Wozniak |
| 01/04 | 1948 Burma granted independence |
| 01/05 | 1933 Construction begins on the Golden Gate Bridge in San Francisco |
| 01/05 | 2005 Dwarf planet Eris is discovered at Palomar Observatory |
| 12/07 | 1941 Attack on Pearl Harbor |
| 12/07 | 1988 Armenian earthquake (6.8) devastates Spitak |

![dynamodb-data.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659216806118/eDTHlIgV-.png align="left")

## Step 6: Test

And now, we get to see it all work together. The deployment project has deployed your App Runner service, DynamoDB table, and the VPC and IAM resources to allow them to connect. Let's see the web application perform.

1. In a browser, visit the App Runner endpoint URL you recorded in Step 4. It defaults to today's date, and unless that happens to be one of the dates you entered data for, it will not show any data.

    ![test-browser-empty.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659297023900/EHTtoSCWP.png align="left")

2. Add **?day=12/07** to the end of the path. Now you see two records returned. It worked!

    ![test-12-07.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659217006163/oPSdNHkWQ.png align="left")

3. Try **day=01/01**, **day=01/02** and the other dates you entered.

    ![test-01-01.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659217196041/rZec7xx5c.png align="left")

    ![test-01-02.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659217291386/2vVPFgOAe.png align="left")

4. Optional: add more date entries to the DynamoDB table if you wish.

Congratulations! You've used AWS deployment tools for .NET and a deployment project to customize and deploy an application and database to AWS. Well done!

## Step 7: Delete the Deployment

We went to all the trouble of a CDK deployment project for a reason: to end up with an easy, repeatable, consistent deployment experience. We also get that for deleting a deployment. All of that applies to deleting a deployment, which we'll now do. Once you're done with the project,delete the deployment as follows:

1. In a command/terminal window, CD to the web project folder.

2. Run the command `dotnet aws delete-deployment`command below and confirm with **y**.

     ```dos
dotnet aws delete-deployment hello-apprunner
```

    ![dotnet-aws-delete-deployment.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659217654476/L0FLr0hvG.png align="left")

3. Wait for the delete deployment to complete, and confirm the resources are no longer listed in the AWS console.

    ![dotnet-aws-delete-deployment-2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659217905679/s9AIMLf-k.png align="left")

    ![before-aws-apprunner-no-services.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659217684531/sMg6B1qqI.png align="left")

    ![before-aws-dynamodb-no-tables.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659217692312/Wsc-dmUnx.png align="left")

    ![before-aws-vpc-no-endpoints.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659217702519/pUbxkXvN5.png align="left")

# Where to Go From Here

Infrastructure-as-code is critical to professional software development, especially at scale. It can also be intimidating due to the large surface area of cloud services. The AWS deployment tool for .NET CLI gives you a guided, recipe-driven experience. Guided experiences are pleasant, but can turn frustrating if the inner workings are hidden from you, or you can't customize the result. Fortunately, the deployment projects feature lets you look under the hood and gives you control, allowing you to modify or extend the CDK. You have full control over the resources being created.

In this tutorial, you created a simple web application that retrieves records from a DynamoDB table. You used the deployment tool to generate a CDK deployment project for App Runner. You modified the CDK to create the table plus the VPC and IAM resources needed to allow the App Runner application to access the DynamoDB table. You deployed your web project with the deployment project, and saw the AWS artifacts created. You entered data in the DynamoDB table and saw the website work. Lastly, you used the deployment tool to delete the deployment. 

Getting confident with CDK and deployment projects is a matter of practice and reviewing reference documentation and examples. In particular, read 
[Deployment Projects with the new AWS .NET Deployment Experience](https://aws.amazon.com/blogs/developer/dotnet-deployment-projects/) by Norm Johanson. Look at the generated code in deployment projects to learn how to structure CDK. CDK writing is easiest when you have a clear idea of what needs to happen in terms of AWS resources and how they are configured to work together. In the tutorial, we had already identified the resources and configuration in an earlier blog post on VPC Connectors and were confident the configuration would function. That made the CDK writing straightforward.

# Further Reading

AWS Documentation

[AWS .NET deployment tool - Deployment projects](https://aws.github.io/aws-dotnet-deploy/docs/project/)

[AWS Cloud Development Kit Documentation](https://docs.aws.amazon.com/cdk/index.html)

Blogs

[Deployment Projects with the new AWS .NET Deployment Experience](https://aws.amazon.com/blogs/developer/dotnet-deployment-projects/) by Norm Johanson

[Hello, .NET Deploy!](https://davidpallmann.hashnode.dev/hello-net-deploy)

[Hello, App Runner VPC Connector!](https://davidpallmann.hashnode.dev/hello-app-runner-vpc-connector)

[Hello, Cloud blog series](https://davidpallmann.hashnode.dev/series/hello-cloud)