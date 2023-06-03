---
title: "Hello, CodeWhisperer!"
datePublished: Sat Jun 03 2023 23:02:19 GMT+0000 (Coordinated Universal Time)
cuid: cligln60d00000alihp2yaaew
slug: hello-codewhisperer
tags: aws, net, dotnet, codewhisperer, ai-ml

---

#### This episode: Amazon CodeWhisperer and AI-assisted coding. In this Hello, Cloud blog series, we're covering the basics of AWS cloud services for newcomers who are .NET developers. If you love C# but are new to AWS, or to this particular service, this should give you a jumpstart.

In this post we'll introduce Amazon CodeWhisperer and use it to write a "Hello, Cloud" .NET program to perform directories of S3. We'll do this step-by-step, making no assumptions other than familiarity with C# and Visual Studio. We're using Visual Studio Code and .NET 6.

# CodeWhisperer : What is it, and why use It?

> "Wihispers are often thunderous." —Richard Smyth

Writing software is no longer a solo activity. Even if you're writing code all by yourself, you're making use of the work of others whenever you use libraries you didn't write, reference API documentation, look up best practices, and search for code examples and answers to questions. This makes writing software a great fit for generative AI assistance.

Amazon CodeWhisperer is a code generator, powered by machine learning, that provides code recommendations in real time. As you write code in your IDE, CodeWhisperer generates suggestions automatically, based on your comments and existing code.

As of this writing, CodeWhisperer supports programming languages C#, Java, Python, JavaScript, and TypeScript with quality training data, and about 10 other languages with lesser training data. CodeWhisperer is available for two IDEs in common use by .NET developers, Visual Studio Code and JetBrains IDEs. You get CodeWhisperer by installing the latest AWS Toolkit for VS Code or AWS Toolkit for JetBrains Rider.

CodeWhisperer comes in two tiers, Individual and Professional. The Individual edition is for individual developers working outside of their organization and is free. The Professional edition is for organizations and includes administrative capabilities at a monthly cost per user. View the [pricing page](https://aws.amazon.com/codewhisperer/pricing/) for Professional tier pricing.

You'll find CodeWhisperer making helpful suggestions as you code. it takes into account both the code you've written and your comments. As you're starting to type a statement, It may offer to complete it for you. It may suggest the next statement(s) to complete or continue a code block. It may give you a full function in response to a comment. CodeWhisperer makes suggestions automatically, but you can also request suggestions on-demand by entering ALT C (Windows) or Option C (Mac). You can also turn off auto-suggestions if it is getting in the way.

CodeWhisperer has some restrictions for C# developers. It is supported in VS Code and Rider but not yet in Visual Studio. At present it can only understand comments written in English. CodeWhisperer can perform code security scans, but this feature is not yet available for C#.

CodeWhisperer is trained on Amazon and public data. Are there any concerns about using the code it generates? The [CodeWhisperer FAQs page](https://aws.amazon.com/codewhisperer/faqs/) states, "Just like with your IDE, you own the code that you write, including any code suggestions provided by CodeWhisperer. You are responsible for the code that you write, including the CodeWhisperer suggestions that you accept. Always review the code suggestions before accepting them, and you may need to make edits to ensure that the code deos exactly what you intended."

# Our Hello, CodeWhisperer Project

In our tutorial we will install CodeWhisperer for Visual Studio Code, then write a program that lists S3 buckets and objects with assistance from CodeWhisperer. Ideally, you'll only have to write the comments.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685829692187/b2f8d4c9-53f2-4cc2-af38-5ffe766aa1ef.png align="center")

## One-time Setup

For any of the tutorials in the Hello, Cloud series you need the following:

1. An [AWS account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/), and an understanding of what is included in the [AWS Free Tier](https://aws.amazon.com/premiumsupport/knowledge-center/what-is-free-tier/).
    
2. [Visual Studio Cod](https://visualstudio.microsoft.com/)e Although this blog series uses Visual Studio 2022 most of the time, you need VS Code for this tutorial. Install [Visual Studio Code](https://code.visualstudio.com/).
    
3. [AWS Toolkit for Visual Studio](https://aws.amazon.com/visualstudio/). You'll need to [configure the toolkit](https://docs.aws.amazon.com/toolkit-for-visual-studio/latest/user-guide/keys-profiles-credentials.html) to access your AWS account and create an IAM user. Your default AWS profile will be linked to this user when running programs from the command line.
    

## Step 1: Install Code Whisperer

In this step, you'll install the AWS Toolkit for Visual Studio Code, which includes CodeWhisperer. To configure the CodeWhisperer Individual tier, you will need to register an AWS builder ID.

1. Launch Viual Studio Code.
    
2. Install the AWS Toolkit. On the left panel, select *Extensions*. Enter **AWS Toolkit** in the search bar and install AWS Toolkit.
    
3. Configure CodeWhisperer:
    
    1. You now have an AWS icon on your left panel. Select it.
        
    2. Under *Developer Tools*, expand the *CodeWhisperer* node and click **Start**.
        
    3. Complete several dialogs in VS Code. On the *CodeWhisperer: Add Connection to AWS* dialog, select **Use a personal email to sign up and sign in with AWS Builder ID**. At the *Copy Code for AWS Builder ID* prompt, click **Copy Code and Proceed**. A code is now in your clipboard that you will use shortly. At the *Do you want Code to open the external website?* prompt, click **Open**. A browser tab opens an *Authorize Request* page.
        
    4. Next, you'll need to complete several prompts in your web browser. On the *Authorize Request* page, paste the code in your clipboard earlier and click **Next**. You'll be guided to enter an email address, your name, a verification code that was emailed to you, and a password. When prompted, click **Allow** to give VS Toolkit permissions. Note: if you already have an AWS Builder ID, there is a button for signing in with it on the email address dialog.
        
    5. In VS Code, if you see the prompt *Some tool you've been using don't work with profile:default*, choose **Yes, keep using AWS Builder ID with CodeWhisperer**.
        
    6. You should have a page displayed in VS Code titled **Using Amazon CodeWhisperer**. Read through it.
        

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685803606533/9fb57fe6-fb98-4f6f-9273-6912da6a9bae.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685805194208/91d63c23-f066-4583-8d6e-d02af8a556d1.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685805354373/a419f086-3dff-4a39-93cd-38b9192067f5.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685805454070/700a99cb-a286-4d36-8a5e-921332127d4a.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685805912648/6db8b505-0392-415f-a1c1-0e54110f335e.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685806110557/b87dae8c-3d10-43c3-998d-2b0d672212ea.png align="center")

## Step 2: Create a C# Console Program.

In this step, you'll create a C# console program from a template.

1. In VS Code, select **Using Amazon CodeWhisperer** from the menu. A terminal window opens.
    
2. In the terminal window, CD to a development folder.
    
3. Run the following dotnet command to create a console program named **s3dir**.
    
    `dotnet new console -n s3dir`
    
4. CD to the s3dir folder.  
    `cd s3dir`
    

## Step 3. Write the s3dir program aided by CodeWhisperer

Now we'll write an S3 directory program, aided by CodeWhisperer.

1. In VS Code, select **File &gt; Open Folder** from the menu and open the **s3dir** folder.
    
2. In the terminal window, run the command **dotnet add package AWSSDK.S3** to add the AWS SDK for .NET S3 library.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685808724139/0a0e5adb-97e2-4b1e-8950-db9bcf98fe8c.png align="center")
    
3. In the menu, select **File &gt; Preferences &gt; Settings** (Windows) or Code &gt; Preferences &gt; Settings (Mac). Expand Workbench and select **Settings Editor**. Change *Editor* to **json.**
    
4. Open Program.cs in the editor, which contains minimal code.  
    `// See` [`https://aka.ms/new-console-template`](https://aka.ms/new-console-template) `for more information`
    
    `Console.WriteLine("Hello, World!");`
    
5. Delete the existing code.
    
6. Type this comment in the editor and press ENTER:  
    **// List S3 buckets**
    
7. If a suggestion does not appear automatically, enter ALT C (Windows) or Option C (Mac).
    
8. Once a suggestion appears, it may be one of several. Press the left or right arrows to sequence through them. When you see a suggestion you like, press the TAB key to accept the suggestion. The listing further below is what I got, which lists buckets to the console - which is exactly what I want to do. If your suggestion is very different, try getting more suggestions or varying the comment text.
    
9. Make your suggestion a complete program if it is not already. If you want to use your default AWS profile, your code should instantiate an AmazonS3Client() without arguments. If you want to specify an AWS access key and secret key, you can specify AmazonS3Client(access-key, secret-key). Always keep your access keys confidential.
    
10. In the terminal window or a command window, enter **dotnet buil**d to build the code, and then **dotnet run** to run the code. You should see your S3 buckets listed.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685814016826/e38e2972-93de-4870-9e69-b4960f7fda14.png align="center")
    
    Congratulations, you used Amazon CodeWhisperer to write a code to list S3 buckets.
    

Program.cs

```csharp
// List S3 buckets


using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using Amazon;
using Amazon.S3;
using Amazon.S3.Model;

namespace ListBuckets
{
    class ListBuckets
    {
        static async Task Main()
        {
            IAmazonS3 client = new AmazonS3Client(RegionEndpoint.USWest2);
            ListBucketsResponse response = await client.ListBucketsAsync();

            List<S3Bucket> buckets = response.Buckets;

            foreach (S3Bucket b in buckets)
            {
                Console.WriteLine($"{b.BucketName} created on {b.CreationDate}");
            }
        }
    }
}
```

## Step 4: List Objects in Buckets

In this step, we'll revise the code to not only list buckets but also the objects in the buckets. If your code is not similar to the listing in Step 3 above (looping through a response to ListBucketsAsync to display bucket names on the console), then replace it with the Step 3 code so we have the same starting point.

1. Find the point in the code in the loop where the bucket name is displayed with Console.WriteLine(). Insert some blank lines. Your edit cursor should be after the Console.WriteLine statement but before the ending curly brace for the loop
    
2. Insert a comment line **// list S3 bucket objects** and await a suggestion (or call one up with ALT C or Option C.  
    `{ Console.WriteLine($"{b.BucketName} created on {b.CreationDate}"); // list S3 bucket objects }`
    
3. Accept the best suggestion. Note, you might not get complete code. If that happens but you get promising initial code, accept it and again ask for a suggestion to add to it. When I did that, 3 times, I first got a statement to build a ListObjectsV2Request. The next suggesstion was a call to client.ListObjectsV2Async, saving the results in a List&lt;S3Object&gt;. The third suggestion added the foreach loop to iterate through the objects and display on the console. I got what I needed, but in snippets at a time.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685828949510/1cf4e18a-d40d-4d32-8857-1e2e2b9f9557.png align="center")
    
4. Ensure your code is complete (you might need to balance your curly braces) and build it in the terminal window with **dotnet build**.
    
5. Run the program again with **dotnet run**. This time, you should see your S3 buckets and the objects contained in them listed.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685827901535/66a1e5f3-d030-47dc-ab0d-908e07d9c0e1.png align="center")
    
    Congratulations! You've used CodeWhispere in a more intricate way to extend existing code.
    

Program.cs v2

```csharp
// List S3 buckets


using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using Amazon;
using Amazon.S3;
using Amazon.S3.Model;

namespace ListBuckets
{
    class ListBuckets
    {
        static async Task Main()
        {
            IAmazonS3 client = new AmazonS3Client();

            ListBucketsResponse response = await client.ListBucketsAsync();

            List<S3Bucket> buckets = response.Buckets;

            foreach (S3Bucket b in buckets)
            {
                Console.WriteLine($"{b.BucketName} created on {b.CreationDate}");

                // list objects in S3 bucket
                ListObjectsV2Request request = new ListObjectsV2Request
                {
                    BucketName = b.BucketName
                };

                ListObjectsV2Response res = await client.ListObjectsV2Async(request);

                List<S3Object> objects = res.S3Objects;
        
                foreach (S3Object obj in objects)
                {
                    Console.WriteLine($"Object: {obj.Key} created on {obj.LastModified}");
                }
            }
        }
    }
}
```

## Where to Go From Here

In this tutorial, you installed CodeWhisperer for VS Code via the AWS Toolkit. You registed an AWS Builder ID which is required for the free Individual edition of CodeWhisperer. You used CodeWhisperer to generate code. If your experience was similar to the tutorial, you saw CodeWhisperer write a complete simple program to list S3 buckets; and then extended the existing code to list the objects in each bucket, all in response to comments you wrote. If your experience gave you different code generation results from the tutorial, that's because CodeWhisperer improves as it learns. Generative AI is not deterministic, so don't expect it to provide consistent results.

To go further, work with CodeWhisperer more and get acquainted with it. Give it a shot at small tasks like statement completion, large tasks like code snippets, and major tasks like writing complete code. View the resources listed below to learn more about it.

## Further Reading

AWS References and Documentation

[Amazon CodeWhisperer](https://aws.amazon.com/codewhisperer/)

[Amazon CodeWhisperer is now generally available](https://aws.amazon.com/about-aws/whats-new/2023/04/amazon-codewhisperer-generally-available/)

[CodeWhisperer User Guide](https://docs.aws.amazon.com/codewhisperer/latest/userguide/what-is-cwspr.html)

[Amazon CodeWhisperer Resources](https://aws.amazon.com/codewhisperer/resources/#) - Documentation

[AWS Toolkit for Rider](https://aws.amazon.com/rider/)

[AWS Toolkit for Visual Studio Code](https://aws.amazon.com/visualstudiocode/)

Videos

[Amazon CodeWhisperer Resources](https://aws.amazon.com/codewhisperer/resources/#) - Videos

[Amazon CodeWhisperer - Write and Read to a File- C#](https://www.youtube.com/watch?v=63PUTmxnEHA) by Doug Seven