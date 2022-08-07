## Hello, CloudWatch Alarms!

#### This episode: Amazon CloudWatch Alarms. In this Hello, Cloud blog series, we're covering the basics of AWS cloud services for newcomers who are .NET developers. If you love C# but are new to AWS, or to this particular service, this should give you a jumpstart.

In this post we'll introduce CloudWatch Alarms and use them with a "Hello, Cloud" .NET program to monitor AWS resources and send alert notifications. We'll do this step-by-step, making no assumptions other than familiarity with C# and Visual Studio. We're using Visual Studio 2022 and .NET 6.

# Amazon CloudWatch Alarms : What are they, and why use them?

> "There are more things to alarm us than to harm us, and we suffer more often in apprehension than reality.." —Lucius Annaeus Seneca 

Any application or service that is mission critical needs to be monitored and operated proactively. To uphold a service level agreement (SLA) for your customers or employees, you must be aware of system health and respond rapidly to problems. 

Amazon CloudWatch Alarms are metric watchdogs that sound the alarm when a metric falls outside the levels (high or low thresholds) you configure. You can attach multiple alarms to a metric, and each one can have multiple actions. We previously introduced CloudWatch in [Hello, CloudWatch!](https://davidpallmann.hashnode.dev/hello-cloudwatch) and focused on CloudWatch Logs. 

CloudWatch Alarms are a powerful mechanism. They can be simple, or complex and nuanced. You create alarms for CloudWatch metrics, such as an EC2 instance's CPU utilization, the number of messages in an SQS queue, or a Lambda function's error rate. You define a condition that determines whether a metric is in an acceptable range or has crossed a threshold into an alarm condition.

An alarm can be in one of 3 states: 
* OK: the metric is in an acceptable range.
* In Alarm: the metric has breached a threshold.
* Insufficient Data: not enough metric data has yet been collected to determine whether there is an alarm condition.

![diagram-alarms.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659876769488/nfkwr5RLu.png align="left")

An alarm can send notifications to an SNS topic when an alarm condition is triggered. Notifications can go human beings, to your applications, or can trigger built-in actions such as EC2 auto-scaling.

![diagram-notification.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659910338344/LOIJSCHNj.png align="left")

Standard CloudWatch alarms have a 1-minute resolution. At a higher cost, high-resolution alarms are also available that can alert as frequently as 10-second periods. 

We're looking at simple alarms in this post, where a metric is observed over a single time period for a condition. More complex configurations are possible:
* You can create **"N of M" alarms** where a condition is checked over multiple time periods. You can learn about that in the video linked at the bottom of this post. 
* You can combine alarms to create **composite alarms**. Composite alarms can consider multiple metrics and conditions. A composite alarm can often help you get an alarm just right, finding the balance between too-noisy false-alarms and a lack of awareness. 
* CloudWatch also supports **anomaly detection alarms** which use machine learning. We'll cover those separately in a future blog post.

The AWS free tier includes 10 alarms for free (not applicable to high-resolution alarms). Always check the [pricing page](https://aws.amazon.com/cloudwatch/pricing/?p=pm&c=la&z=4) for the latest rates and free tier specifics.

# Our Hello, CloudWatch Alarms Project

We'll first develop a simulated order generation console program and an order processing Lambda function, connected by an SQS queue for order messages. We'll create 2 CloudWatch alarms. The first will alert when orders are not being processed. The second will alert on errors during order processing. SNS email notifications will be sent when either alarm enters the alarm state.

![dotnet-run.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659910595345/aq9ipXwWR.png align="left")

![aws-08-in-alarm.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659910666577/XndRotipU.png align="left")

![email-alert.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659910619728/-jgNXQ1NP.png align="left")

[source code](https://github.com/davidpallmann/[link])

## One-time Setup

For any of the tutorials in the Hello, Cloud series you need the following:

1. An  [AWS account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/), and an understanding of what is included in the  [AWS Free Tier](https://aws.amazon.com/premiumsupport/knowledge-center/what-is-free-tier/). 
2. [Microsoft Visual Studio 2022](https://visualstudio.microsoft.com/). If you're using an older version of Visual Studio you won't be able to use .NET 6. If you use a different IDE, you may have to find alternatives to some tutorial steps.
3. [AWS Toolkit for Visual Studio](https://aws.amazon.com/visualstudio/). You'll need to [configure the toolkit ](https://docs.aws.amazon.com/toolkit-for-visual-studio/latest/user-guide/keys-profiles-credentials.html) to access your AWS account and create an IAM user. Your default AWS profile will be linked to this user when running programs from the command line.

## Step 1: Create an SQS queue

In this step, you'll create an IAM role for the Lambda function and 2 Amazon Simple Queue Service (SQS) queues.

1. Sign in to the AWS console. At top right, select the region you want to work in. I'm using **us-west-2 (Oregon)**.

2. Create (or update) a role named **lambda-role** with the `AmazonSQSFullAccess` permission:

    A. Navigate to **IAM > Roles**.

    B. Create a role named `lambda-role`, or edit the role if already present.

    C. Add the `AmazonSQSFullaccess` permission to the role.

    ![iam-lambda-role.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659908412804/YL_3AgHP-.png align="left")

3. Navigate to **SQS**. 

4. Click **Create queue** and create a queue named `orders`. You may have to find a variant of the name if already in use.

    ![aws-create-queue.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659820503436/nrzvWWJ4e.png align="left")

5. Create another queue named  `orders-dlq` in the same way you created the `orders` queue.  This will be a dead letter queue, where order queue messages that fail during processing get sent. Without a dead letter queue, a message that causes the Lambda function to fail will remain in the queue, resulting in an infinite loop. With a dead letter queue configured, failed message processing will store the message in the dead letter queue. 

    ![aws-queues.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659907544217/8dF4e9zJ5.png align="left")

6. Configure the `orders` queue to use the dead-letter queue. 

    A. Navigate to the `orders` queue detail and select **Dead-letter queue**. 

    B. Click **Edit** and expand the Dead-letter queue section. Click **Enabled**.

    C. Choose queue: select the `orders-dlq` queue.

    D. Maximum receives: **1**.

    E. Click **Save**.

    ![aws-config-dlq.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659907761865/-Z3GaB4ZM.png align="left")

## Step 2: Create an Alarm

In this step, you'll create an alarm alerting you when orders are piling up without being processed. To achieve that, we'll create a CloudWatch alarm that sends a notification when the oldest non-deleted message in the queue is older than 60 seconds, a sign that orders are not being processed.

1. In the AWS console, navigate to **CloudWatch > Alarms > All Alarms**.

    ![aws-01-nav-alarms.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659827662919/2UD1TYygh.png align="left")

2. Click **Create alarm** and create a new alarm.

    A. On the Specify metric and conditions page, click **Select metric**.

    B. On the Browse tab, select **SQS** and then **Queue metrics**.

    ![aws-02-select-metric-sqs.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659827866505/FQQXdv0yW.png align="left")

    C. Select metric **ApproximateAgeOfOldestMessage** and click **Select metric**.

    ![aws-03-select-metric-sqs-ApproxAgeOfOldestMessage.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659828075320/amhZ-i6bo.png align="left")

    D. Statistic: select **Maximum**.

    E. Period: **1 minute**.

    F. Alarm condition: **Greater/Equal**.

    G. Than (threshold value): **60**.

    H. Click **Next**.

    ![aws-04-set-conditions.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659828548960/dRZBmA7tg.png align="left")

    I. On the Configure notifications page, select Alarm state trigger: **In alarm**. This means when the alarm enters the alarm state, we want a notification.

    J. Send a notification to the following SNS topic: select **Create new topic**.

    K. Topic name: **Orders_Not_Being_Processed**.

    L. Email endpoints that will receive the notification: enter your email address.

    M. Click **Create topic**.

    N. Click **Next**.

    O. Alarm name: **Orders_Not_Being_Processed**.

    ![aws-05-configure-notifications.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659829055257/vr3nPUoAL.png align="left")

    P. Click **Next**.

    Q. Click **Create alarm**.

    ![aws-06-create-alarm-preview.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659829224939/cA_fzEtgX.png align="left")

3. Check your inbox where an email from `AWS Notifications` should be waiting. Open it and confirm the SNS notification subscription by clicking **Confirm subscription**.

    ![email-confirm-subscription.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659830153671/BYIP8c2Ho.png align="left")

     ![confirm-subscription.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659829621777/xTjUQdrcs.png align="left")

4. Back in the AWS console, you see a message confirming the alarm has been created. The dashboard shows the alarm is in a State of `insufficient data`. 

    ![aws-07-alarm-created.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659829310424/I2XA51lvk.png align="left")

5. After several minutes pass, the alarm will show an OK state.

## Step 3: Create a Take Orders console app

In this step, you'll create a .NET console application that generates orders and stores them in the queue.

1. Open a command/terminal window.

2. Create a development folder named `hello-cloudwatch-alarms` and CD to it.

3. Run the `dotnet new` command below to create a console program named **TakeOrders**.

    ```dos
dotnet new console -n TakeOrders
```

    ![dotnet-new-console.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659825588574/rBNcriLJl.png align="left")

4. Launch Visual Studio 2022 and open the `PutOrders` project.

5. Add these NuGet packages to the project: **AWSSDK.SQS** and **System.Text.Json**.

    ![nuget.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659825807154/5BRr1TaNA.png align="left")

6. Open Program.cs in the code editor and replace with the code below. The program will take a count from the command line and write that many orders to the orders queue, with a random collection of items.

    ![dotnet-run.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659826235613/YjKEY_htS.png align="left")

## Step 4: Run the TakeOrders program and test the SQS alarm

In this step, you'll run the TakeOrders program and test the alarm. Running TakeOrders will add messages to the orders queue. Since we haven't written a program to process orders yet, the messages will remain in the queue. After they age sufficiently, the alarm we created in Step 2 will enter the alarm state and send an email notification to your inbox.

1. In a command window, CD to the `TakeOrders` folder.

2. Enter the command **dotnet run -- 5** to write 5 orders to the queue.

    ```dos
dotnet run -- 5
```

3. Return to the AWS console and wait for the alarm to enter an In alarm state, which should happen within the next 2 minutes. As we have 5 order messages sitting in the orders queue, all we need to do is wait to satisfy the alarm threshold of a message in the queue older than 60 seconds.

    ![aws-08-in-alarm.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659829889523/7sejbA4Hf.png align="left")

4. Check your inbox, and there should be an alert message, telling you that orders are not being processed.

    ![email-alert.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659830301486/BIkwiyGKm.png align="left")

5. If you wait several more minutes, you will not receive any additional notifications. This is because the notification happens when the alarm enters the alarm state. To receive another notification, the alarm would need to clear and then enter the alarm state again.

    ![dotnet-run.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659826401515/74NYdDC6e.png align="left")

## Step 5: Write a Lambda Function to Process Orders

In this step, you'll write an AWS Lambda function to process orders in the queue. This will be a simple simulation in which we'll simply log what's in the order message. Upon successful processing, Lambda will delete the queue message. In addition, if a line item for "Widgets" is found, we'll simulate an out-of-inventory condition and throw an exception. That will allow us to cause some errors, which our second alarm will monitor. On an exception, the order message will end up in the dead letter queue.

1. Open a command/terminal window and CD to the `hello-cloudwatch-alarms' folder.

2. Run the dotnet command below to install Amazon Lambda .NET project templates.

    ```dos
dotnet new --install "Amazon.Lambda.Templates"
```

    ![dotnet-install-lamnda-templates.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659900465350/xxVrhGWH3.png align="left")

3. Run the `dotnet new` command below to create a Lambda function for processing SQS messages named **ProcessOrder**.

   ```dos
dotnet new Lambda.SQS -n ProcessOrder
```

    ![dotnet-new-lambda.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659900607234/WwQucOEwm.png align="left")

4. Open the` ProcessOrder` project in Visual Studio.

5. Open `Function.cs` in the code editor, and replace with the code below at the end of this step. The code writes log details to the function's CloudWatch log, and the Lambda function will delete the message. If an order contains a `Widget` line item, an out-of-inventory error is written to the log.

6. Save your changes and build the project.

7. In Solution Explorer, right-click the ProcessOrder project and select **Publish to AWS Lambda**.

    A. Region: select the region you've been using in previous steps.

    B. Function Name: **ProcessOrder**.

    C. Click **Next**.

    ![publish-lambda-01.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659902583097/mUH_vxu47.png align="left")

    D. Role Name: select `lambda-role`.

    E. DLQ resource: select **orders-dlq**.

    ![publish-lambda-03.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659908003090/cA9oaCFsJ.png align="left")

    F. Click **Upload**, and wait for publishing to complete.

8. On the Function: ProcessOrder test page, configure the `orders` SQS queue as an event source:

    A. Select **Event Sources**. 

    B. Click **+ Add**.

    C. On the Add Event Source dialog, select Source Type **Amazon SQS**, SQS Queue **orders**, Batch size **1**, and click **OK**.

    ![vs-configure-event-source.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659903007164/3NiaIAU51.png align="left")

    ![vs-configure-event-source-02.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659903078008/FEPR3kciJ.png align="left")

9. In the AWS console, navigate to **Lambda** and click the **ProcessOrders** function to view its detail. Note SQS is configured as a trigger. Click on **SQS** and you'll see confirmation the `orders` queue is the trigger. This is the result of what you did in #8 above via the AWS Toolkit for Visual Studio. 

10. The Lambda function has already executed for each message in the orders queue, and the queue is now empty. Select **Monitor** and **Logs**, and you'll see a LogStream listed.

    ![aws-lambda-monitor-logs.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659903674324/eA-jqV1HQ.png align="left")

11. Click the LogStream link, which will bring up the CloudWatch log in a separate browser tab. You'll see info log messages for most orders, and a fail (error) messages when an order with a Widget is encountered.

    ![aws-lambda-monitor-logs.-2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659903834367/bAZdk9M5C.png align="left")

Function.cs

```csharp
using Amazon.Lambda.Core;
using Amazon.Lambda.SQSEvents;
using System.Text.Json;


// Assembly attribute to enable the Lambda function's JSON input to be converted into a .NET class.
[assembly: LambdaSerializer(typeof(Amazon.Lambda.Serialization.SystemTextJson.DefaultLambdaJsonSerializer))]

namespace ProcessOrder;

public class Function
{
    /// <summary>
    /// Default constructor. This constructor is used by Lambda to construct the instance. When invoked in a Lambda environment
    /// the AWS credentials will come from the IAM role associated with the function and the AWS region will be set to the
    /// region the Lambda function is executed in.
    /// </summary>
    public Function()
    {

    }


    /// <summary>
    /// This method is called for every Lambda invocation. This method takes in an SQS event object and can be used 
    /// to respond to SQS messages.
    /// </summary>
    /// <param name="evnt"></param>
    /// <param name="context"></param>
    /// <returns></returns>
    public async Task FunctionHandler(SQSEvent evnt, ILambdaContext context)
    {
        foreach(var message in evnt.Records)
        {
            await ProcessMessageAsync(message, context);
        }
    }

    private async Task ProcessMessageAsync(SQSEvent.SQSMessage message, ILambdaContext context)
    {
        context.Logger.LogInformation($"Processed message {message.Body}");

        var order = JsonSerializer.Deserialize<Order>(message.Body)!;

        context.Logger.LogLine($"Order Id {order.Id}");
        foreach(var item in order.Items)
        {
            context.Logger.LogLine($"{item}");
            if (item=="Widget")
            {
                throw new ApplicationException("Out of inventory: Widget");
            }
        }

        await Task.CompletedTask;
    }
}

public class Order
{
    public string Id { get; set; } = null!;
    public List<string> Items { get; set; } = null!;

    public Order() { }

    public Order(string id, string[] items)
    {
        Id = id;
        Items = new List<string>(items);
    }

}
```

## Step 6: Create an Alarm for Lambda Function Errors

In this step, you'll create another CloudWatch alarm to monitor the Lambda function for errors.

1. In the AWS console, navigate to **CloudWatch > All Alarms**.

2. Create an alarm to monitor the Lambda function for errors:

    A. Click **Create alarm**.

    B. Click **Select metric**, **Lambda**, and **By Function Name**.

    C. Select **ProcessOrder Errors** and click **Select metric**.

    ![aws-alarm2-configure-01.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659904178348/EiCz7Xz2j.png align="left")

    D. On the Specify metric and conditions page, select Statistic **Sum** and Period **1 minute**.

    E. Under Conditions, select **Greater** and enter threshold value **0**.

    F. Click **Next**.

    ![aws-alarm2-configure-02.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659904335534/aBvV3RG4H.png align="left")

    G. Alarm state trigger: **In alarm**.

    H. Send notification: **Create new topic**.

    I. Topic name: **ProcessOrders_Lambda_function_errors**.

    J. Email endpoints that will receive the notification: enter your email address.

    K. Click **Create topic**.

    L. Click **Next**.

    ![aws-alarm2-configure-03.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659904536290/sF3DbFqNB.png align="left")

    M. Alarm name: **ProcessOrder_Lambda_function_error**.

    N. Click **Next** and **Create alarm**.

    ![aws-alarm2-configure-04.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659904744015/oAItOjYAQ.png align="left")

3. Go to your inbox, read the SNS notification subscription email, and click the link to accept the subscription.

## Step 7: Test the Lambda Function Errors Alarm

In this step, you'll test that the new alarm works. To do that, we'll again run the TakeOrders console program to generate random orders. When those orders include a Widget, the ProcessOrder Lambda function will log an error, and within a minute the `ProcessOrder_Lambda_function_error` alarm should enter an alarm state and send an email.

1. In a command/terminal window, CD to the `TakeOrders` project folder.

2. Run the `dotnet run` command below to generate more orders. If none of the generated orders displayed contain a "Widget", run the command again.

   ```dos
dotnet run -- 5
```

    ![test-alarm2-01.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659905077127/q728VnrXY.png align="left")

3. In the AWS console, navigate to **CloudWatch > All alarms** and wait for the `ProcessOrder_Lambda_function_errors` alarm to enter the **In alarm** state.

    ![aws-alarm2-in-alarm.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659908669621/YwiL8G7Dv.png align="left")

4. Go to your inbox, where an SNS notification email should be waiting with alarm details.

    ![alarm2-email.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1659908784061/VlGJpwTAl.png align="left")

    Congratulations! You've now seen 2 different CloudWatch alarms work, one based on message age and one based on error count.

5. Optional: create conditions to put your alarms in the ALARM or OK states. You might need to wait, run TakeOrders again, or delete the Lambda functio in various sequences to achieve this.

## Step 8: Shut it Down

When you're done with the project, delete the AWS resources. You don't want to be charged for something you're not using.

1. In the AWS console, navigate to **CloudWatch > All Alarms** and delete the 2 alarms, `Orders_Not_Being_Processed` and `ProcessOrder_Lambda_function_errors`.

2. Navigate to **Lambda** and delete the Lambda function, `ProcessOrder`.

3. Navigate to **SQS** and delete the 2 queues, `orders` and `orders-dlq`.

# Where to Go From Here

To support a production application well, you should leverage alarms and notifications. CloudWatch alarms provide both simple and complex options for monitoring metrics.

In this tutorial, you wrote a console program that generates orders and a Lambda function that processes orders, connected by an SQS queue. You created a CloudWatch alarm for orders not being processed, whose threshold was SQS messages older than 60 seconds. You saw the alarm send you email when the alarm triggered. You also created an alarm for Lambda function errors, and saw that alarm trip when processing orders with Widgets, which makes the Lambda function fail. 

Getting the conditions right for an alarm takes some practice. Always verify your alarm conditions with testing. In a production scenario, review your alarms and refine them based on your experiences. This tutorial did not cover complex alarm scenarios, such as "N out of M" alarms with multiple periods, anomaly detection alarms, or composite alarms. To go further, learn more about these features and experiment with them.

# Further Reading

AWS Documentation

[Using Amazon CloudWatch Alarms](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html)

Videos

[Amazon CloudWatch Alarm Setup Tutorial ](https://www.google.com/search?client=firefox-b-e&q=cloudwatch+alarms+tutorial#kpvalbx=_TL_uYoHVNo6gqtsPwJSWuAw41) by Be a Better Dev

Blogs

[Alarms, incident management, and remediation in the cloud with Amazon CloudWatch](https://aws.amazon.com/blogs/mt/alarms-incident-management-and-remediation-in-the-cloud-with-amazon-cloudwatch/) by Eric Scholz

[Hello, Cloud blog series](https://davidpallmann.hashnode.dev/series/hello-cloud)