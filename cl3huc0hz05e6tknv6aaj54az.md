## Hello, Mechanical Turk!

#### This episode: Amazon Mechanical Turk and crowd-sourcing human intelligence tasks. In this Hello, Cloud blog series, we're covering the basics of AWS cloud services for newcomers who are .NET developers. If you love C# but are new to AWS, or to this particular service, this should give you a jumpstart.

In this post we'll introduce Amazon Mechanical Turk and use it in a "Hello, Cloud" .NET program to crowd-source human intelligence tasks. We'll do this step-by-step, making no assumptions other than familiarity with C# and Visual Studio. We're using Visual Studio 2022 and .NET 6.

# Mechanical Turk : What is it, and why use It?

> “Crowdsourcing is a great way to approach creation because in any given point there's always somebody on the Internet who knows something better than you do.” —Guy Kawasaki"

Despite advances in automation and machine intelligence, some tasks are simply done better by human beings. That includes content moderation, de-duping data, and research. Crowdsourcing is a way to tackle a really big problem by dividing it into discrete pieces. If you can define your need as simple, repetitive micro-tasks, crowdsourcing lets you engage a large virtual workforce temporarily and affordably. For example, if you had a catalog with thousands or millions of items, you might employ crowdsourcing to correct information or find duplicates, giving each worker one product item to perform a task on.

Before we get into what Amazon Mechanical Turk is, let's explain the name. This is a reference to the "Mechanical Turk" of 1770 by Wolfgang von Kempelen. It appeared to be a chess-playing robot. In fact, there was an actual person hidden in the cabinet, a chess master who was making the moves. The Turk was exhibited across Europe and America for decades. Notable people who played it and lost to it include Napoleon Bonaparte and Benjamin Franklin.

![512px-Racknitz_-_The_Turk_3.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1653151814100/WuDc_HZn3.jpg align="left")

[Amazon Mechanical Turk](https://www.mturk.com/) (hereafter "MTurk") is a crowdsourcing service that gives you access to an elastic crowd of human intelligence and skill, via an API. Amazon describes it as, "a crowdsourcing marketplace that makes it easier for individuals and businesses to outsource their processes and jobs to a distributed workforce who can perform these tasks virtually."

![diagram-crowd-src-mktplace.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1653138275687/lONve6bDa.png align="left")

You can use MTurk for batch work or in an event-driven context.
* In **batch mode**, you start with a dataset and submit a batch of tasks. For example, nightly processing to dedupe or validate new data entry before updating a data store. 
* In **event-driven mode**, you integrate MTurk into your business workflow. You use the API to trigger tasks when an event occurs. You can use SNS notifications when tasks complete to perform the next round of processing, which you might do with AWS Lambda functions. For example, if an algorithm can't come up with recommendations, you could fall back on human intelligence tasks.

## Concepts

* A **requester** is a person or business who needs tasks completed.
* **Workers** are people who want to earn money who are willing to perform those tasks.
* A **human intelligence task (HIT)** is a self-contained task, such as "what color is this dress?" or "are these two objects the same?".
* An **assignment** gives the same task to multiple people. You might do this to get consensus on a task.

## Pricing

What does all this cost? You will be charged an amount to pay workers, plus a percentage fee you pay MTurk. Additional fees are charged when you require workers with qualifications, such as a college graduate, homeowner, parent, online purchaser, or voter. See the [Amazon Mechanical Turk pricing page](https://requester.mturk.com/pricing) for details.

There's a developer sandbox you can use to test MTurk without incurring charges. It allows you to simulate a requester and a worker.

# Our Hello, MTurk Project

We will write a .NET program that can create HITs and get results using the AWS SDK for .NET. The HIT we'll create will ask workers what is their favorite flavor of ice cream. We'll run this in the MTurk developer sandbox to avoid any charges.

![04-dotnet-add-status.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1653252916141/Mmzs-HW16.png align="left")

![04-worker-answer-1.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1653252848013/2mevmcAoI.png align="left")

[source code](https://github.com/davidpallmann/hello-mturk)

## One-time Setup

For any of the tutorials in the Hello, Cloud series you need the following:

1. An  [AWS account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/), and an understanding of what is included in the  [AWS Free Tier](https://aws.amazon.com/premiumsupport/knowledge-center/what-is-free-tier/). 
2. [Microsoft Visual Studio 2022](https://visualstudio.microsoft.com/). If you're using an older version of Visual Studio you won't be able to use .NET 6. If you use a different IDE, you may have to find alternatives to some tutorial steps.
3. [AWS Toolkit for Visual Studio](https://aws.amazon.com/visualstudio/). You'll need to [configure the toolkit ](https://docs.aws.amazon.com/toolkit-for-visual-studio/latest/user-guide/keys-profiles-credentials.html) to access your AWS account and create an IAM user. Your default AWS profile will be linked to this user when running programs from the command line.

## Step 1: Create an MTurk Account integrated with your AWS Account

In this step, you'll sign up with MTurk as a requester and link the MTurk account to your AWS account.

1. Navigate to the [Amazon Mechanical Turk console](https://mturk.com) and click the **Get started with Amazon Mechanical Turk** button.

    ![01-requeser-sign-up-1.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1653157455999/fKyGY_mh0.png align="left")

2. Create an MTurk account linked to your AWS account:

    A. Click the **Create a requester account** button. 

    B. On the next page, click the **Register a new MTurk Account with your AWS account** link.

    ![01-requester-sign-up-2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1653157618566/jKiUyO9Fk.png align="left")

    C. On the Confirm New MTurk Account page, confirm your expected AWS account number and email address are displayed, and click the **Create new MTurk account** button.

    ![01-requester-sign-up-3.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1653158587521/GFDTEp8Eg.png align="left")

    D. Set a display name and email address, and click **Create Account**.

    ![01-requester-sign-up-4.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1653158686925/vspfd7dAQ.png align="left")

    E. With your account set up, you'll see a welcome page with Getting Started options.

    ![01-requester-sign-up-5.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1653158831879/zcLHtt0-M.png align="left")

3. Run the AWS CLI.

    To confirm your MTurk account is integrated with your AWS account, let's use the AWS CLI to verify we access MTurk.

    A. Open a command/terminal window.

    B. Enter the command below. We specify region us-east-1 on the command because that is the region MTurk runs out of.

    ```dos
aws mturk list-hits -- region us-east-1
```

    You should see an empty list of HITs returned, since we haven't defined any yet. This confirms our AWS account and MTurk account are integrated.

    ![01-requester-sign-up-6-aws-cli.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1653159174718/3IbTZNUsb.png align="left")

## Step 2: Register for the Developer Sandbox and Create Access Keys 

In this step, you'll get familiar with the Mechanical Turk console, register for the developer sandbox, and generate access keys. Let's start with a quick tour.

1. In the MTurk console, click the **Create** menu item at top.

2. Under *Select a customizable template to start a new project*, try clicking each item and seeing what the project preview shows you. This gives you an idea about the different kinds of tasks you can request.

    ![02-create-survey.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1653160036494/5k4V44LW2.png align="left")

3. Click the **Manage** menu link at top.

    A. Click **Results**. The Manage Batches page displays batches in progress, batches review for review, and batches already reviewed. Batches are tasks you run against large data sets. 

    ![02-manage-results.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1653160455641/BPa_ABYxt.png align="left")

    B. Click **Workers**. The Manage Worker page lists workers who have completed work for you. 

    ![02-manage-workers.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1653160398976/t7u8BgpBg.png align="left")

    C. Click **Qualification Types**. The Manage Qualification Types page lets you create qualifications which let you match tasks to workers with the qualifications required.

    ![02-manage-qualifications.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1653160345133/FZCIZ_0_j.png align="left")

4. Give your AWS user permissions to access MTurk:

    A. Click the **Developer** menu link at top.

    ![02-developer.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1653163125896/mbKb8PJWz.png align="left")

    B. Under *Step 2: Link Your AWS Account with Your MTurk Requester Account*, you should see confirmation your AWS account is linked. 

    C. Click **Get Your AWS Access Keys**.

    D. On the IAM page that opens, select **Users** on the left pane.

    E. Select the user associated with your AWS default profile. If you've been following this Hello, Cloud series, select the user you created when you originally installed the AWS Toolkit for Visual Studio: this user is also associated with your AWS default profile.

    F. With the user details open, select the **Permissions** tab. Click **Attach policies** and select **Attach existing policies directly.** Attach the **AmazonMechanicalTurkFullAccess** policy.

    G. Click **Next: Review** and **Add Permissions** to add the policy.

    ![02-iam-add-turk-access.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1653163536590/seDJq1n6j.png align="left")

5. Register for the Developer Sandbox. Back on the [Getting Started as an Amazon Mechanical Turk Developer](https://requester.mturk.com/developer) page,

     A. Under *Step 3: Register for the MTurk Developer Sandbox*, click **Register for Requester Sandbox**.

    ![02-register-sandbox-1.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1653164916096/1Dly2uMCg.png align="left")

    B. Click *Sign in or Create an account* **Requester** at top right.

    ![02-register-sandbox-2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1653165005147/h265r32qD.png align="left")

     C. Click the **Register a new MTurk Account with your AWS Account** link.

     D. On the Confirm New MTurk Account Page, click **Create new MTurk account**.

    E. Set a Display Name and email address. 

     ![02-register-sandbox-3.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1653165238559/wtU8Luo-m.png align="left")

    F. Click **Create Account**.

    G. Click **Developer** at top. You should now see confirmation that your **Sandbox requester account** and **AWS User** are now linked.

    ![02-register-sandbox-4.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1653165386578/u9elsxAFk.png align="left")

    H. Click **Get your AWS access keys** 

    I. On the IAM page, click **Create New Access Key**.

     ![02-register-sandbox-5-keys.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1653165537137/w1nhNJA-m.png align="left")

    J. Record or download your access key and secret key and keep them in safe place. You will need these keys later on when you run your .NET code.

## Step 3: Create a Sandbox Worker

1. Navigate to https://requestersandbox.mturk.com in a browser.

2. If prompted, sign in to your AWS account.

3. Fill in the Worker Registration form for a fictional worker, and click **Create Account**.

![03-register-sandbox-worker.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1653168119198/rQmKoCSnv.png align="left")

![03-register-sandbox-worker-2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1653168196517/9Jkkeq-dZ.png align="left")

We can now simulate the worker side of our Mechanical Turk sandbox experiment.

## Step 4: Create a .NET Console Program

In this step, you'll create a .NET program that can check your MTurk balance, add a new Human Intelligent Task, or get results for a HIT.

1. Open a command/terminal window and CD to a development folder.

2. Run the `dotnet new` command below to create a new .NET console program named `hello-mturk`.

    ![03-dotnet-new.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1653160720366/30wEN13VC.png align="left")

3. Launch Visual Studio and open the `hello-mturk` project.

4. Add the AWSSDK.MTurk NuGet package to the project.

    A. In Solution Explorer, right-click the `hello-mturk` project and select **Manage NuGet Packages..."

    B. Search for and install the **AWSSDK.MTurk** package.

    ![03-nuget.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1653160892585/7NL8M4eOb.png align="left")

3. Add a question.xml file to the project. This will control what a worker sees when they work on your task. 

    A. In Solution Explorer, add a question.xml file. 

    B. In Properties for the question.xml file, set Copy to Output Directory to **Copy if newer**.

    C. In the code editor, enter the XML below.

    ```xml
<HTMLQuestion xmlns="http://mechanicalturk.amazonaws.com/AWSMechanicalTurkDataSchemas/2011-11-11/HTMLQuestion.xsd">
  <HTMLContent><![CDATA[
  <!DOCTYPE html>
    <html>
    <head>
      <meta http-equiv='Content-Type' content='text/html; charset=UTF-8'/>
      <script type='text/javascript' src='https://s3.amazonaws.com/mturk-public/externalHIT_v1.js'> .   
      </script>
    </head>
    <body>
      <form name='mturk_form' method='post' id='mturk_form' action='https://www.mturk.com/mturk/externalSubmit'>
        <input type='hidden' value='' name='assignmentId' id='assignmentId'/>
        <h1>What's your favorite ice cream flavor?</h1>
        <p><textarea name='comment' cols='80' rows='3'></textarea></p>
        <p><input type='submit' id='submitButton' value='Submit' /></p></form>
        <script language='Javascript'>turkSetAssignmentID();    
        </script>
      </body>
    </html>
    ]]>
  </HTMLContent>
  <FrameHeight>450</FrameHeight>
</HTMLQuestion>
```

4. Open Program.cs in the code editor. Replace the code with the code below.

Program.cs

```csharp
using Amazon.MTurk;
using Amazon.MTurk.Model;

string SANDBOX_URL = "https://mturk-requester-sandbox.us-east-1.amazonaws.com";
//string PROD_URL = "https://mturk-requester.us-east-1.amazonaws.com";

var config = new AmazonMTurkConfig()
{
    RegionEndpoint = Amazon.RegionEndpoint.USEast1,
    ServiceURL = SANDBOX_URL // PROD_URL
};

var mturkClient = new AmazonMTurkClient(config);

var command = (args.Length > 0) ? args[0] : null;

if (command == "balance")
{
    GetAccountBalanceRequest request = new GetAccountBalanceRequest();
    GetAccountBalanceResponse balance = await mturkClient.GetAccountBalanceAsync(request);
    Console.WriteLine("Your account balance is $" + balance.AvailableBalance);
}
else if (command == "add")
{
    string questionXML = System.IO.File.ReadAllText(@"question.xml");

    var hitRequest = new CreateHITRequest()
    {
        LifetimeInSeconds = 60*60*3,
        AssignmentDurationInSeconds = 60*3,
        Reward = "0.03",
        Title = "Type the (one or two words) you see in the image",
        Description = "Enter the highlighted text in the oval",
        Question = questionXML,
        MaxAssignments = 3
    };

    var createResponse = mturkClient.CreateHITAsync(hitRequest);
    var HITId = createResponse.Result.HIT.HITId;
    Console.WriteLine($"HIT created - Id: {HITId}");
    Console.WriteLine($"Worker link: https://workersandbox.mturk.com/projects/{createResponse.Result.HIT.HITTypeId}/tasks");

    var cont = String.Empty;
    Console.WriteLine("Press any key to update, or Q to quit.");
    do
    {
        var statusResponse = mturkClient.GetHITAsync(new GetHITRequest { HITId = HITId });
        var HIT = statusResponse.Result.HIT;
        Console.WriteLine($"Status: {HIT.HITStatus}, Expiration: {HIT.Expiration}, Available: {HIT.NumberOfAssignmentsAvailable}, Pending: {HIT.NumberOfAssignmentsPending}, Completed: {HIT.NumberOfAssignmentsCompleted}");
    } while (Console.ReadKey().KeyChar != 'Q');

}
else if (command=="results")
{
    var listRequest = new ListAssignmentsForHITRequest()
    {
        HITId = args[1]
    };
    var listResponse = await mturkClient.ListAssignmentsForHITAsync(listRequest);
    foreach(var assignment in listResponse.Assignments)
    {
        Console.WriteLine($"Worker Id: {assignment.WorkerId}, Answer: {assignment.Answer}");
    }
}
else
{
    Console.WriteLine("dotnet run -- add ................ creates a new Human Intelligence Task and monitors status");
    Console.WriteLine("dotnet run -- balance ............ shows account balance");
    Console.WriteLine("dotnet run -- results [hit-id] ... shows results for HIT");
}
```

### Understand the Code

Lines 7-10: The program first creates an `AmazonMTurkConfig` object that specifies region US East 1 and that we are using the sandbox URL. To run in production, you would set the `serviceURL` property to PROD_URL.

13: An `AmazonMTurkClient` is created with the configuration.

15-22: If the command line parameter is "balance", a `GetAccountBalanceRequest` object is created and the `GetAccountBalanceAsync` SDK method is called. We display the `AvailableBalance` property of the response.

23-52: If the command line parameter is "add", a Human Intelligence Task (HIT) is generated. The XML for the question form is read from the question.xml file. A `CreateHITRequest` object specifies lifetime of the HIT, payment, a title, description, question form XML, and the maximum number of assignments. Calling `CreateHITAsync` returns a response with our HIT Id, from which we can compute the URL for responding to the task in the worker sandbox. Then, we enter a loop to fetch and display the status for the HIT. A keystroke refreshes the status, until "Q" is entered to quit.

53-63: if the command line parameter is "results", the `ListAssignmentsForHITAsync` method is called, passing the HIT Id specified in the second command line parameter.  The response includes an `Assignments` collection, each containing an `Answer` property, which is XML that includes an answer entered by a worker.

## Step 5: Run the Program

In this step, you'll run the program to check balance, add a HIT, and get results for a hit. Our program understands these command lines:

| command | description |
| --- | --- |
| `dotnet run -- add` | creates a new Human Intelligence Task and monitors status. |
| `dotnet run -- balance` | shows account balance. |
| `dotnet run -- results [hit-id]` | shows results for HIT. |

1. Open a command/terminal window and CD to the project location.

2. Set environment variables with your access key and secret key. The AWS_ACCESS_KEY and AWS_SECRET_ACCESS_KEY variables provide your access key and secret access key. If you need to run on a different AWS profile than `default`, you can set this as well with the AWS_PROFILE variable.

    On a Windows PC, use these commands:

    ```dos
SET AWS_ACCESS_KEY_ID=xxxxxxxxxx
SET AWS_SECRET_ACCESS_KEY=yyyyyyyyyy
```

    On a Mac, use these commands:

    ```dos
export AWS_ACCESS_KEY_ID=xxxxxxxxxx
export AWS_SECRET_ACCESS_KEY=yyyyyyyyyy
```

3. Run the command below to check your balance. We are in the developer sandbox, so the balance will always be reported as $10,000. This also confirms you are able to access the MTurk API with your access key and secret key.

    ```dos
dotnet run -- balance
```

    ![04-dotnet-balance.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1653248314730/J6jDUQ4wj.png align="left")

4. Run the command below to add a new HIT.

    ```dos
dotnet new -- add
```

    ![04-dotnet-add.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1653255113428/QNgProWXp.png align="left")

5. In the console output, take note of the following:

     A. The HIT Id. Ours is `3IHWR4LC7FM8FF6XVS2DB7Q196T8IT`.

     B. The link from the console output. Copy this to the clipboard. Ours is `https://workersandbox.mturk.com/projects/3DBJM5HJP7KJ0XLU773S4GFFQ3EP9O/tasks`.

6. In a browser, visit the link to see what a worker would see. We see our test question, and attribution to our sandbox requester.

7. In the worker web page, click **Accept**.

8. Fill in an answer, and click **Submit**.

    ![04-worker-answer-1.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1653248546861/A-fdex289.png align="left")

    ![04-worker-respond-2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1653170660769/_SvTiKUVU.png align="left")

9. Back in the command window, press any key to get an updated status for the HIT. The status now shows 2 tasks available instead of 1.

    ![04-dotnet-add-status.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1653249124328/FQl7IKLtj.png align="left")

10. Press Q to quit the program.

11. Now, let's fetch results for our HIT. In the command window, run the `dotnet run -- results` command with the HIT ID from #5A above. You see the response to the task as XML. Within that, the <Answer> element contains the response the worker entered.

   ```dos
dotnet run -- results [HitID]
```

    ![04-dotnet-add-status.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1653255539070/wWaSu0H2B.png align="left")

    ```xml
<?xml version="1.0" encoding="ASCII"?>
  <QuestionFormAnswers xmlns="http://mechanicalturk.amazonaws.com/AWSMechanicalTurkDataSchemas/2005-10-01/QuestionFormAnswers.xsd">
  <Answer>
    <QuestionIdentifier>comment</QuestionIdentifier>
    <FreeText>Vanilla</FreeText>
  </Answer>
</QuestionFormAnswers>
```

Congratulations! You've programmatically created a HIT and retrieved responses. Using the worker sandbox limited us to just one response, but in a production setting you would have multiple workers responding, within the limits you specified when creating the HIT. Use your imagination and imagine many responses.

# Where to Go From Here

Crowdsourcing adds the vital human element that some processing requires. With Amazon Mechanical Turk, you can utilize a vast elastic marketplace of workers through an API. 

In this tutorial, you linked your AWS account to Amazon Mechanical Turk and set up the developer sandbox. You wrote .NET code that used the AWS SDK for .NET to check your balance, create a Human Intelligence Task, and get results. You simulated the worker experience in the sandbox.

This tutorial did not cover qualifications or the myriad of different tasks available. To go further, review Amazon Mechanical Turk product pages, including developer resources and videos. Experiment with a use case for your business, and get a feel for designing tasks that are effective with workers.

Determine whether your use cases are batch or are event-driven. If event-driven, think how you will integrate MTurk into your business workflow. What events will cause you to create HITs? What MTurk SNS notifications will trigger Lambda functions to process results?

# Further Reading

AWS Documentation

[Amazon Mechanical Turk](https://www.mturk.com/)

[Documentation](https://docs.aws.amazon.com/mturk/index.html)

[FAQs](https://www.mturk.com/help)

[Developer Getting Started Guide](https://docs.aws.amazon.com/AWSMechTurk/latest/AWSMechanicalTurkGettingStartedGuide/Welcome.html)

[Developer Resources](https://www.mturk.com/resources)

[Sign up as a Requester or Worker](https://www.mturk.com/get-started)

[Worker Portal](https://www.mturk.com/worker)

[AWS SDK for .NET Documentation](https://docs.aws.amazon.com/sdk-for-net/index.html)

Videos

[AWS re:Invent 2018: Harness the Power of Crowdsourcing with Amazon Mechanical Turk](https://www.youtube.com/watch?v=cv4DJkmGzIw)

Blogs

[Tutorial: Setting up your AWS Account to make use of MTurk’s API](https://blog.mturk.com/tutorial-setting-up-your-aws-account-to-make-use-of-mturks-api-4e405b8fc8cb)

[Tutorial: Using the MTurk API with the AWS SDK for C# with .NET](https://blog.mturk.com/tutorial-using-the-mturk-api-with-the-aws-sdk-for-c-with-net-207fdec3bf75)

[Tutorial: Publishing HITs with MTurk and the C# programming language](https://blog.mturk.com/tutorial-publishing-hits-with-mturk-and-the-c-programming-language-505e6a3cdd95)

[Hello, Cloud blog series](https://davidpallmann.hashnode.dev/series/hello-cloud)G