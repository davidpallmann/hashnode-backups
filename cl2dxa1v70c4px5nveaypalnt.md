## Hello, Cost Explorer!

#### This episode: AWS Cost Explorer and cost analysis. In this Hello, Cloud blog series, we're covering the basics of AWS cloud services for newcomers who are .NET developers. If you love C# but are new to AWS, or to this particular service, this should give you a jumpstart.

In this post we'll introduce AWS Cost Explorer and use it in a "Hello, Cloud" .NET program to query AWS charges. We'll do this step-by-step, making no assumptions other than familiarity with C# and Visual Studio. We're using Visual Studio 2022 and .NET 6.

# AWS Cost Explorer : What is it, and why use It?

> "From great power comes a great electricity bill.” —Unknown

Cloud computing comes with a very different cost model from the traditional "capital expenditures" purchasing of servers and other IT equipment. The utility computing model is like your home utility bills for electricity or water: you pay for what you use. And, like your home utilities, you can sometimes be surprised by your monthly bill. Maybe you (or a member of your family) have been using more electricity than you expected. Maybe someone left the lights on or the water faucet running. 

Cloud computing charges are a bit more complicated and nuanced than your home utility bills: exactly what you get charged for varies across cloud services. Sometimes, allocating cloud service resources results in the creation of other associated resources such as EC2 instances, a load balancer, auto-scaling groups, IAM roles, S3 buckets, or CloudFormation stacks. If you're not fully aware of your allocations or their pricing models, you could be surprised by your next bill. You might even believe you've deleted a cloud application, only to later discover that some associated resource is still allocated and continues to accrue charges. 

[AWS Cost Explorer](https://aws.amazon.com/aws-cost-management/aws-cost-explorer/) (hereafter "Cost Explorer") is an AWS management console interface for viewing and analyzing AWS costs. AWS describes it as "an easy-to-use interface that lets you visualize, understand, and manage your AWS costs and usage over time." You can use Cost Explorer to review costs for the past year, and to forecast spend for the coming year based on your usage.

![diagram-access-methods.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650840146585/eUOT_liE7.png)

You can access AWS Cost Explorer in 3 different ways:
1. AWS management console. Cost Explorer has a rich UI with multiple views, filters, and built-in reports which is excellent for analysis. 
2. Command line. You can query Cost Explorer using the AWS CLI, which returns JSON responses. 
3. Cost Explorer API. Your own applications can query the AWS Cost Explorer API to access cost and usage information programmatically.

Cost Explorer is not free, and you have to enable it in the AWS console. The AWS Cost Explorer API will charge you $0.01 per request. You can optionally enable hourly granularity in Cost Explorer, which will give you access to hourly EC2 resource usage data for the last 14 days. This comes at an extra cost. The AWS Cost Explorer best practices documentation has tips for minimizing your charges. 

# Our Hello, Cost Explorer Project

We will first get familiar with Cost Explorer in the AWS console. Then, we'll query it from the AWS CLI. Finally, we'll write a .NET program to get cost and usage information from the Cost Explorer API.

![01-aws-cost-explorer-sel-region.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650840352046/2gxwqLVm9.png)

![02-cli-service.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650840399698/vLtztzrtX.png)

![03-dotnet-run.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650840410282/ogBbdTW7E.png)

[source code](https://github.com/davidpallmann/hello-cost-explorer)

## One-time Setup

For any of the tutorials in the Hello, Cloud series you need the following:

1. An  [AWS account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/), and an understanding of what is included in the  [AWS Free Tier](https://aws.amazon.com/premiumsupport/knowledge-center/what-is-free-tier/). 
2. [Microsoft Visual Studio 2022](https://visualstudio.microsoft.com/). If you're using an older version of Visual Studio you won't be able to use .NET 6. If you use a different IDE, you may have to find alternatives to some tutorial steps.
3. [AWS Toolkit for Visual Studio](https://aws.amazon.com/visualstudio/). You'll need to [configure the toolkit ](https://docs.aws.amazon.com/toolkit-for-visual-studio/latest/user-guide/keys-profiles-credentials.html) to access your AWS account and create an IAM user. Your default AWS profile will be linked to this user when running programs from the command line.

## Step 1: Use Cost Explorer in the AWS Console

In this step, you'll get familiar with Cost Explorer in the AWS console. Since your AWS usage and cost data will differ from mine, we will provide general steps on exploring your data. I'll point out along the way how I used Cost Explorer to analyze my own usage. 

1. Sign in to the AWS console. Navigate to AWS Cost Management. You can enter **cost** in the search bar.

2. On the left panel, select **Cost Explorer**. If this is the first time you've visited, you'll get a notice: *Since this is your first visit, it will take some time to prepare your cost and usage data. Please check back in 24 hours.* If you get this message, this is a good time to run an errand or do some laundry. Come back periodically and refresh the page, until you no longer get the message.

    ![01-welcome-aws-cost-mgmt.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650749973333/69Db214JM.png)

3. If AWS Cost Management is ready for you, the AWS Cost Management Home page should now show you a cost summary showing you how daily costs are tracking for recent months. This is useful, but it's not actually the Cost Explorer tool.

    ![01-aws-home.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650750832260/65I6w_-hy.png)

4. Click the **Cost Explorer** button at the top right of the cost summary to view this data in Cost Explorer. We can see this view gives us many filters and controls for analysis. In my case, this account usually has minor costs each weekend when I temporarily allocate resources for a Hello, Cloud blog. However, there are some relatively large charges at the end of March.

     ![01-aws-cost-explorer-summary.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650751130807/8mFfhTc-G.png)

5. Find some data you want to study. Zoom in on a date range and go to the Services view. 

    A. Find an interesting section of your data to analyze, and set the date range on the upper left to focus on that. 

    B. On the Group By bar, click **Service** to view which services are responsible for your charges. 

     To drill into my above-average charges, I set the date range to March 26-April 1. The Services view shows me a breakdown by service, both in the bar chart and the table below. It's now evident that these charges came from the Personalize, CloudWatch, and Data Pipeline services. Moreover, it's clear that nearly all of the cost is from Personalize, the purple bars.  Connecting this with known activity, it fits: I blogged "Hello, Personalize" on March 27. Nearly a week went by before I remembered to deallocate my artifacts. Oops.

    ![01-aws-cost-explorer-sel-days-services.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650752734507/yrlkcW8NH.png)

6. View costs by region. On the Group By bar, click **Region**. My view shows that most of my charges are in US-West-2 (Oregon), which I usually default to because many AWS services are supported in that region. I used to use US-West-1 (N. California), but not anymore. The minor charges indicate I may have left something behind, still allocated. The big surprise is that US-East-1 (N. Virginia) is listed, which I never intentionally use. I may have started some work in the console before realizing I had the wrong region selected, and never went back to remove it. Already we can see Cost Explorer is revealing a lot.

    ![01-aws-cost-explorer-sel-region.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650753451368/C631OBRqw.png)

8. How can you combine views? In my case, I want the service breakdown from #5 above, but I want to know which services are being used in US-West-1.  I can do that by returning to the Group By Service view, then setting a filter at right to only show Region US-East-1. This tells me exactly what I need to know. CloudWatch is charging me a minor amount. Navigating to CloudWatch in the console for that region, I see there are CloudWatch alarms I can delete, and I remove them.

    ![01-aws-cost-explorer-sel-services-filter-region-us-west-1.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650754764346/y1It7E6Mq.png)

7. Repeating the above for the other surprise region, US-East-1, I learn CloudWatch, Data Pipeline, and Lambda are the services charging me, and now I know which areas of the console to visit to inspect and possibly delete them.

    ![01-aws-cost-explorer-sel-services-filter-region-us-east-1.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650753695803/7FfUAtv2n.png)

    When I set my console region to US-East-1 and visit the CloudWatch area, I see there are 14 CloudWatch alarms. Reading through them, I see they are DynamoDB alarms for a table named "FactBook". Then I remember: I created and blogged about an [AWS website and Alexa voice skill for the CIA World FactBook](http://davidpallmann.blogspot.com/2019/02/cia-world-factbook-on-aws-part-3-alexa.html) data (a public-domain reference) back in 2019, and had completely forgotten about it. I decided I didn't actually want to delete it after all, so I left it in place.

8. There are built-in reports in Cost Explorer. The views of service usage we've done above are available in the Monthly Usage by Service built-in report.

    A. In the AWS console, select **Reports** on the left pane.

    ![01-aws-reports.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650836689213/gHKSiGdcm.png)

    B. Select the **Monthly cost by service** report.

    C. You see a view of monthly cost broken out by service.

    ![01-reports-service-monthly.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650836667684/4PecC2gJF.png)

    D. If you wish, use the filters and other controls to tailor the view.

    The above illustrates the value of AWS Cost Explorer. You can quickly and easily drill in on costs to understand where they are coming from. Taking a look in the AWS console was also important, because something I thought was a left-over was in fact something I want to keep running.

## Step 2: Use Cost Explorer from the Command Line

In this step, you'll use the `aws ce` command to query Cost Explorer. We'll only look at a few variations of one view, *cost and usage*, but be aware that many cost views and option parameters are available, which are documented [here](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ce/index.html). 

1. If you don't already have it installed, [install the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) and configure it.

2. Open a command/terminal window.

3. Run the command below to get a daily breakdown of costs by month for January-March 2022. Feel free to change the date parameters to your liking.

   ```dos
aws ce get-cost-and-usage --time-period Start=2022-04-01,End=2022-04-23 --granularity=MONTHLY --metrics BlendedCost
```

    The JSON includes a ResultsByTime array, each showing a month and amount billed. Results are a month at a time because the command line contained the parameter `--granularity=MONTHLY`. 

    ![02-cli-monthly.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650833710548/CcMKVvD_R.png)

4. Let's change the granularity to daily. Run the modified version of the command below, this time listing charges for January by day. 

   ```dos
aws ce get-cost-and-usage --time-period Start=2022-01-01,End=2022-01-31 --granularity=DAILY --metrics BlendedCost
```

    Now you get cost separated out for each day in the response.

    ![02-cli-daily.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650833959376/XO1r6HfZF.png)

4. Let's add some parameters to get a breakdown by service. As we saw in Step 1 with the Cost Explorer in the AWS console, one of the dimensions available is by service. The command below adds a `--group-by Type=DIMENSION,Key=SERVICE` parameter. 

   ```dos
aws ce get-cost-and-usage --time-period Start=2022-01-01,End=2022-02-01 --granularity=MONTHLY --metrics BlendedCost --group-by Type=DIMENSION,Key=SERVICE
```

    The results now include a charge breakdown by service.

    ![02-cli-service.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650834786984/15YvKbYlN.png)

## Step 3. Write a .NET Program for Cost Reporting

Now that we've seen what AWS Cost Explorer can do, it's time to work with it programmatically. In this step you'll write a .NET 6 program that queries the API for cost and usage by service.

1. Open a command/terminal window and CD to a development folder.

2. Run the `dotnet new` command below to create a new .NET 6 console application:

   ```dos
dotnet new console -n hello-cost-explorer
```

    ![02-dotnet-new.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650755258468/Gbmo7OMtc.png)

3. Launch Visual Studio and open the hello-cost-explorer project.

4. In Solution Explorer, right-click the hello-cost-explorer project and select **Manage NuGet Packages...**. Search for and install the **AWSSDK.CostExplorer** package.

    ![02-nuget.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650755437003/yXmMInVBS.png)

5. Open Program.cs in the editor and replace with the code at the end of this step. 

6. Save your changes and ensure you can build the program.

7. In a command/terminal window, CD to the project location and run the command with a date range on the command line. Specify dates in the form yyyy-mm-dd, for example 2021-12-31.

   ```dos
dotnet run -- 2022-01-01 2022-04-01
```

    You see a list of date ranges for each month, under which services and charges are listed. 

    ![03-dotnet-run.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1650835330464/zqiPahWtC.png)

8. Run the command again with other date ranges.

Congratulations! You've successfully queried the AWS Cost Explorer API using the AWS SDK from .NET code.

Program.cs

```csharp
using Amazon;
using Amazon.CostExplorer;
using Amazon.CostExplorer.Model;

// Get date range from the command line

if (args.Length < 2)
{
    Console.WriteLine("Usage: dotnet run -- [start-yyyy-mm-dd] [end-yyyy-mm-dd]");
}

var startDate = args[0];
var endDate = args[1];

// Query the Cost Explorer API

var client = new AmazonCostExplorerClient();
var request = new GetCostAndUsageRequest()
{
    Granularity = "MONTHLY",
    GroupBy = { new GroupDefinition() { Key = "SERVICE", Type = GroupDefinitionType.DIMENSION } },
    TimePeriod = new DateInterval()
    {
        Start = startDate,
        End = endDate
    },
    Metrics = { "BlendedCost" }
};

var response = await client.GetCostAndUsageAsync(request);

// Display results

foreach (var result in response.ResultsByTime)
{
    Console.WriteLine($"{result.TimePeriod.Start}-{result.TimePeriod.End}");
    foreach(var group in result.Groups)
    {
        foreach(var key in group.Keys)
        {
            var amount = group.Metrics.FirstOrDefault().Value.Amount;
            if (amount != "0")
            {
                Console.WriteLine($"    {key} {group.Metrics.FirstOrDefault().Value.Amount} {group.Metrics.FirstOrDefault().Value.Unit}  ");
            }
        }
    }
    Console.WriteLine();
}
```

### Understand the Code

The code is very brief and easy to understand. Program.cs instantiates an `AmazonCostExplorerClient`. To get cost and usage data, we create a `GetCostAndUsageRequest`, specifying a monthly granularity, grouping by service, and a time period that was specified on the command line. The request is passed to the `GetCostAndUsageAsync` method.

The code iterates through the response's `ResultsByTime` array. Each element contains groups, and each group contains a key and metric. We iterate through them, displaying the key (service name) and metric (amount and unit of currency).

# Where to Go From Here

The elastic, pay-for-use nature of cloud computing brings many benefits, including trading fixed expenses for variable expenses, lower costs from massive economy of scale, affordable experimentation, fast scale, and freedom to walk away without an ongoing obligation. Cloud computing can also be financially dangerous if you don't understand pricing models, underestimate your usage, aren't disciplined, or accidentally leave the meter running. AWS Cost Explorer is a vital tool for analyzing and understanding your AWS cloud charges. You can use it to optimize your spending. 

In this tutorial, you used Cost Explorer to review your cloud charges in three ways. First, you used it in the AWS management console. Next, you used it from the command line via the AWS CLI. Third, you wrote .NET code to query the Cost Explorer API. In each form of access, the available views and filter parameters followed a consistent model. We learned how to get a breakdown of charges by service, but we barely scratched the service of what you can do with Cost Explorer.

Cost Explorer is only a part of AWS's cost management offerings. To understand what's available, I recommend reading through A Beginner’s Guide to AWS Cost Management, linked below. If you are confident that you'll maintain a consistent level of cloud usage, you can reduce costs with **Savings Plans**. In exchange for a 1- or 3-year usage commitment, you can reduce your costs by as much as two-thirds. The Cost and Management console can make recommendations about savings plans.

To go further with Cost Explorer, use it! Get familiar with the many reports and views available. Look into the Budgets and Savings Plans features, which we did not cover here. Consider whether the hourly granularity option is something you need, which comes at additional cost.

# Further Reading

AWS Documentation

[AWS Cost Explorer](https://aws.amazon.com/aws-cost-management/aws-cost-explorer/)

[AWS CLI - ce command](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ce/index.html)

[Best Practices for the AWS Cost Explorer API](https://docs.aws.amazon.com/cost-management/latest/userguide/ce-api-best-practices.html)

[A Beginner's Guide to AWS Cost Management](https://aws.amazon.com/blogs/aws-cloud-financial-management/beginners-guide-to-aws-cost-management/)

[AWS Billing and Cost Management Documentation](https://docs.aws.amazon.com/account-billing/?id=docs_gateway)

[Savings Plans](https://docs.aws.amazon.com/savingsplans/index.html)

[AWS SDK for .NET Documentation](https://docs.aws.amazon.com/sdk-for-net/index.html)

Videos

[Minutes to Cloud: AWS Cost Explorer with AWS Command Line Interface](https://www.youtube.com/watch?v=L7iuyBn7Krs) by Shazli Mohd Ghazali

Blogs

[AWS Cloud Financial Management blog](https://aws.amazon.com/blogs/aws-cloud-financial-management/)

[Using AWS Cost Explorer to analyze data transfer costs](https://aws.amazon.com/blogs/mt/using-aws-cost-explorer-to-analyze-data-transfer-costs/) by Ashish Mehra

[Hello, Cloud blog series](https://davidpallmann.hashnode.dev/series/hello-cloud)