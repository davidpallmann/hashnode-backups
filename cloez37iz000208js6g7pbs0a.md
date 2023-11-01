---
title: "Hello, Bedrock Images!"
datePublished: Tue Oct 31 2023 23:41:24 GMT+0000 (Coordinated Universal Time)
cuid: cloez37iz000208js6g7pbs0a
slug: hello-bedrock-images
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1698787393833/113655fc-e252-46ed-a5c6-a84c1f702208.png
tags: aws, net, dotnet, amazon-bedrock

---

#### This episode: Amazon Bedrock and Generative AI Images. In this Hello, Cloud blog series, we're covering the basics of AWS cloud services for newcomers who are .NET developers. If you love C# but are new to AWS, or to this particular service, this should give you a jumpstart.

In this post we'll explore [Amazon Bedrock](https://aws.amazon.com/bedrock/) and generative AI image processing. For an introduction to Bedrock, see the [Hello Bedrock!](https://davidpallmann.hashnode.dev/hello-bedrock) post. We'll use Bedrock today to generate images based on text prompts using a Lambda function. We'll do this step-by-step, making no assumptions other than familiarity with C# and Visual Studio. We're using Visual Studio 2022 and .NET 6.

# Bedrock Image Generation : What is it, and why use It?

> "Artificial intelligence is not a substitute for human intelligence; it is a tool to amplify human creativity and ingenuity." —Fei-Fei Li

Bedrock's generative AI capabilities include images as well as text and chat. AWS lists these use cases for image generation: "Quickly create realistic and visually appealing images and animations for ad campaigns, websites, presentations, and more." You might use generative images for advertising and marketing, for creative media assets, or gaming.

You can use the [Stable Diffusion XL](https://aws.amazon.com/bedrock/stable-diffusion/) model for image generation. Stable Diffusion XL generates high-quality images in virtually any art style and claims to be the best open model for photorealism. Once you enable model access to Stable Diffusion XL, you can easily experiment with it in the AWS console.

# Our Hello, Bedrock Image Project

We will first get familiar with Bedrock in the AWS console using the Bedrock image playground. Then we'll write a .NET AWS Lambda function that generates images in response to text prompts, using S3. You'll provide your prompt by uploading a text file to S3, and the Lambda function will generate a corresponding .png file with a matching image.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1698789425193/2619142e-80d5-485c-8167-06a292317426.png align="center")

For example, this .txt input will yield this .png image:

santa-dog.txt  
`A dog wearing a santa cap.`

santa-dog.png

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1698772372735/607d2874-b404-46d2-a533-f742261f072e.png align="left")

[source code](https://github.com/davidpallmann/hello-bedrock-image)

## One-time Setup

For any of the tutorials in the Hello, Cloud series you need the following:

1. An [AWS account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/), and an understanding of what is included in the [AWS Free Tier](https://aws.amazon.com/premiumsupport/knowledge-center/what-is-free-tier/).
    
2. [Microsoft Visual Studio 2022](https://visualstudio.microsoft.com/). If you're using an older version of Visual Studio you won't be able to use .NET 6. If you use a different IDE, you may have to find alternatives to some tutorial steps.
    
3. [AWS Toolkit for Visual Studio](https://aws.amazon.com/visualstudio/). You'll need to [configure the toolkit](https://docs.aws.amazon.com/toolkit-for-visual-studio/latest/user-guide/keys-profiles-credentials.html) to access your AWS account and create an IAM user. Your default AWS profile will be linked to this user when running programs from the command line.
    

## Step 1: Configure Bedrock Model Access

In this step, you'll request Bedrock access to the Stable Diffusion XL model.

1. Sign in to the AWS console. At top right, select the region you want to work in. You can check supported regions for Bedrock on the [Amazon Bedrock endpoints and quotas](https://docs.aws.amazon.com/general/latest/gr/bedrock.html) page. I'm using **us-west-2 (Oregon)**.
    
2. Navigate to **Amazon Bedrock**.
    
3. On the left pane, select **Model access**.
    
4. Click **Edit**. Check the box for **Stable Diffusion XL**. As the page reminds you, **be sure you're comfortable** with pricing terms and End User License Agreements for each model you enable.
    
5. Click **Save changes**.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1698772770303/a78368ea-4d28-4271-8431-f4224a4291d0.png align="center")

## Step 2: Use Bedrock in the AWS console.

In this step, you'll get experience with Bedrock images in the AWS console.

1. In the AWS Bedrock console, select **Playgrounds &gt; Image** from the left panel.
    
2. Use the drop downs at top of page to select provider **Stability AI** and model **Stable Diffusion XL**.
    
3. In the **Prompt** input box, enter a text prompt:  
    **A family enjoying Thanksgiving dinner.**
    
4. Click **Run** and observe the image result. I got a family enjoying a turkey dinner, which is the right idea, but not exactly what I want.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1698773006402/60319916-4872-4bcc-86d5-21a9c175cffd.png align="left")
    
5. Refine the input prompt text to be more specific about the food:  
    **A family enjoying Thanksgiving dinner with turkey, mashed potatoes, gravy, and stuffing.**  
    Click **Run**. Note the image now shows the desired food items. However, I wanted the family to be visible.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1698773189707/0ce4c02a-b35f-4818-98e9-77643f8ecab1.png align="left")
    
6. Refine the prompt further to get specific about the people and the "family gathered round the table" style of image we're after.  
    **A family (father, mother, son, daughter) enjoying Thanksgiving dinner with turkey, mashed potatoes, gravy, and stuffing. In the style of a Norman Rockwell painting.**
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1698773567484/65d39a5b-405e-44d2-9242-fae99dd9c0c8.png align="left")
    
7. This is better, but for this image, I'd like to depict an Asian-American family celebrating Thanksgiving. Refine the prompt yet again to get really specific about the people.  
    **An Asian family (father, mother, teenage son, toddler daughter) enjoying Thanksgiving dinner with turkey, mashed potatoes, gravy, and stuffing. In the style of a Norman Rockwell painting.**
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1698774140617/d8963904-96d9-4e78-b984-f09fc23c37ed.png align="left")
    
8. We're getting there, but I want this to look real. Refine the prompt one last time and ask for photo-realistic output by adding "DSLR photo".  
    **An Asian family (father, mother, teenage son, toddler daughter) enjoying Thanksgiving dinner with turkey, mashed potatoes, gravy, and stuffing. In the style of a Norman Rockwell painting. DSLR photo.**
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1698773922647/6b1193f4-324c-4ef9-9a93-e48ebb157e91.png align="left")
    

As you can see, refining your prompts and fine-tuning is critical to getting to the results you want. Experiment with some prompts of your own and refine them. For tips, see [Stable Diffusion prompt: a definitive guide](https://stable-diffusion-art.com/prompt-guide/).

## Step 3. Create AWS Artifacts for Lambda Function

Now we're going to work on a Lambda function that performs image processing. First, we need to create AWS artifacts in the AWS console.

1. **Create an S3 bucket**. Our Lambda function will need a bucket for reading text prompts and generating images for them.
    
    1. In the AWS console, navigate to **S3** and create a bucket. Mine is named **hello-bedrock-image**, but you'll have to choose a unique name not in use. Remember to use *your* bucket name in the remaining steps.
        
    2. Once the bucket is created, select the **Properties** tab and record the Amazon Resource Name (ARN). For me, that's `arn:aws:s3:::hello-bedrock-image`.
        
    3. Select the **Permissions** tab and edit the Bucket policy. Enter the JSON below. Replace YOUR-AWS-ACCOUNT with your AWS account number, and `hello-bedrock-image` with your bucket name.
        
    
    ```json
       {
           "Version": "2012-10-17",
           "Statement": [
               {
                   "Sid": "AllowS3Access",
                   "Effect": "Allow",
                   "Principal": {
                       "AWS": "arn:aws:iam::YOUR-AWS-ACCOUNT:role/lambda-bedrock"
                   },
                   "Action": "s3:*",
                   "Resource": [
                       "arn:aws:s3:::hello-bedrock-image",
                       "arn:aws:s3:::hello-bedrock-image/*"
                   ]
               }
           ]
       }
    ```
    
2. **Create a Lambda execution role**. We need to create an execution role for our Lambda function that has permission to invoke Bedrock.
    
    A. In the AWS console, navigate to **IAM &gt; Roles** and click **Create role**.
    
    Trusted entity type: **AWS Service**  
    Use case - service of use case: **Lambda**  
    Click **Next.**
    
    1. On the Add permissions page, search for and select (check) the following policies:  
        **AWSLambdaBasicExecutionRole  
        AWSXRayDaemonWriteAccess**  
        Click **Next**.
        
    2. Name the role `lambda-bedrock`.  
        Click **Create role**.
        
    3. Back on the IAM Roles page, search for click the **lambda-bedrock** role. On the **Permissions** tab, click **Add permissions &gt; Create inline policy**.  
        Select the **JSON** tab.  
        Enter the JSON below, which gives access to invoking Bedrock models for the Stable Diffusion XL model. Replace `us-west-2` with your region.  
        Click **Next**.
        
    
    ```json
    {
        "Version": "2012-10-17",
       	"Statement": [
       		{
       			"Sid": "BedrockAccessStmt",
       			"Action": [
       				"bedrock:InvokeModel",
       				"bedrock:InvokeModelWithResponseStream"
       			],
       			"Effect": "Allow",
       			"Resource": [
       				"arn:aws:bedrock:us-west-2::foundation-model/stability.stable-diffusion-xl-v0"
       			]
       		}
       	]
    }
    ```
    
    1. Name the policy **bedrock-stability-diffusion-xl**.  
        Click **Create policy**.
        

## Step 4: Create Lambda Project

In this, you'll create a Lambda project on your local machine.

1. Open a command/terminal window and CD to a development folder.
    
2. Enter the dotnet new command below to create a console program named **hello-bedrock**:  
    `dotnet new lambda.S3 -n hello-bedrock-image`
    
3. CD to the **hello-bedrock-image\\src\\hello-bedrock-image** folder.
    
4. ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1698791183350/4e672bb5-c12d-48f1-975b-18bc3c477f78.png align="center")
    
    Run these commands to add S3, Bedrock, Newontsoft, and ImageSharp NuGet packages and Newtonsoft to your project. Note that [ImageSharp](https://sixlabors.com/products/imagesharp/) is used. If you use ImageSharp, be aware of its [license](https://sixlabors.com/pricing/) and what conditions qualify as commercial use.
    
    `dotnet add package AWSSDK.S3`  
    `dotnet add package AWSSDK.BedrockRuntime`  
    `dotnet add package Newtonsoft.JSON`  
    `dotnet add package SixLabors.ImageSharp`
    
5. Open the hello-bedrock-image project in Visual Studio or your preferred IDE.
    
6. Open Function.cs in the code editor, and replace it with the code below. See *Understand the Code* later in this post for a walkthrough of what the code is doing.
    
    ```json
    using Amazon.Lambda.Core;
    using Amazon.Lambda.S3Events;
    using Amazon.S3;
    using Amazon.S3.Model;
    using System.Text;
    using Amazon;
    using Amazon.BedrockRuntime;
    using Amazon.BedrockRuntime.Model;
    using Newtonsoft.Json;
    using Newtonsoft.Json.Linq;
    using Amazon.S3.Transfer;
    
    #pragma warning disable CS8600 // Converting null literal or possible null value to non-nullable type.
    
    // Assembly attribute to enable the Lambda function's JSON input to be converted into a .NET class.
    [assembly: LambdaSerializer(typeof(Amazon.Lambda.Serialization.SystemTextJson.DefaultLambdaJsonSerializer))]
    
    namespace hello_bedrock_image;
    
    public class Function
    {
        IAmazonS3 S3Client { get; set; }
    
        const string modelId = "stability.stable-diffusion-xl-v0";
        const string modelRequestBody = "{{'text_prompts':[{{'text':'{0}'}}],'cfg_scale':10,'seed':0,'steps':50}}";
    
        /// <summary>
        /// Default constructor. This constructor is used by Lambda to construct the instance. When invoked in a Lambda environment
        /// the AWS credentials will come from the IAM role associated with the function and the AWS region will be set to the
        /// region the Lambda function is executed in.
        /// </summary>
        public Function()
        {
            S3Client = new AmazonS3Client();
        }
    
        /// <summary>
        /// Constructs an instance with a preconfigured S3 client. This can be used for testing outside of the Lambda environment.
        /// </summary>
        /// <param name="s3Client"></param>
        public Function(IAmazonS3 s3Client)
        {
            this.S3Client = s3Client;
        }
    
        /// <summary>
        /// This method is called for every Lambda invocation. This method takes in an S3 event object and can be used 
        /// to respond to S3 notifications.
        /// </summary>
        /// <param name="evnt"></param>
        /// <param name="context"></param>
        /// <returns></returns>
        public async Task FunctionHandler(S3Event evnt, ILambdaContext context)
        {
            var eventRecords = evnt.Records ?? new List<S3Event.S3EventNotificationRecord>();
            foreach (var record in eventRecords)
            {
                var s3Event = record.S3;
                if (s3Event == null)
                {
                    continue;
                }
    
                try
                {
                    // Only process .txt files. If the output .png file already exists, do not re-process.
    
                    if (!s3Event.Object.Key.EndsWith(".txt")) continue;
    
                    var textKey = s3Event.Object.Key;
                    var imageKey = textKey.Replace(".txt", ".png");
                    context.Logger.LogInformation($"10 Processing {textKey}");
    
                    // check whether output .png file alredy exists
    
                    bool alreadyProcessed = true;
                    try
                    {
                        await this.S3Client.GetObjectMetadataAsync(s3Event.Bucket.Name, imageKey);
                        context.Logger.LogInformation($"15 Output file {imageKey} already exists, skipping processing");
                    }
                    catch (AmazonS3Exception)
                    {
                        alreadyProcessed = false;
                    }
    
                    if (alreadyProcessed) continue;
    
                    // Read the image description from the .txt file S3 object.
    
                    context.Logger.LogInformation($"20 Reading figure caption from .txt file S3 object");
    
                    var S3response = await S3Client.GetObjectAsync(s3Event.Bucket.Name, s3Event.Object.Key);
                    StreamReader reader = new StreamReader(S3response.ResponseStream);
                    string prompt = reader.ReadToEnd();
    
                    context.Logger.LogInformation(prompt);
    
                    // Generate a Bedrock image for the image description.
    
                    context.Logger.LogInformation("50 Creating Bedrock request");
                    var bedrockClient = new AmazonBedrockRuntimeClient(RegionEndpoint.USWest2);
    
                    JObject json = JObject.Parse(String.Format(modelRequestBody, prompt));
                    byte[] byteArray = Encoding.UTF8.GetBytes(JsonConvert.SerializeObject(json));
                    MemoryStream stream = new MemoryStream(byteArray);
    
                    var bedrockRequest = new InvokeModelRequest()
                    {
                        ModelId = "stability.stable-diffusion-xl-v0",
                        ContentType = "application/json",
                        Accept = "application/json",
                        Body = stream
                    };
    
                    // Invoke model and capture base64-encoded image from response.
    
                    context.Logger.LogInformation("60 Invoking model");
    
                    var bedrockResponse = await bedrockClient.InvokeModelAsync(bedrockRequest);
                    string responseBody = new StreamReader(bedrockResponse.Body).ReadToEnd();
    
                    dynamic parseJson = JsonConvert.DeserializeObject(responseBody);
                    string base64 = parseJson!.artifacts[0].base64;
    
                    // Convert base64 to image stream and save .png file to to S3.
    
                    context.Logger.LogInformation("70 Converting image to a .png image stream");
    
                    var bytes = Convert.FromBase64String(base64!);
                    var image = Image.Load(bytes);
    
                    Console.WriteLine($"90 Saving image as S3 object {imageKey} in bucket {s3Event.Bucket.Name}");
    
                    using (var S3utility = new TransferUtility())
                    using (MemoryStream msImage = new MemoryStream())
                    {
                        await image.SaveAsPngAsync(msImage);
                        await S3utility.UploadAsync(msImage, s3Event.Bucket.Name, imageKey);
                    }
                }
                catch (Exception e)
                {
                    context.Logger.LogError($"Error getting object {s3Event.Object.Key} from bucket {s3Event.Bucket.Name}. Make sure they exist and your bucket is in the same region as this function.");
                    context.Logger.LogError(e.Message);
                    context.Logger.LogError(e.StackTrace);
                    throw;
                }
    
                context.Logger.LogInformation("99 end");
            }
        }
    }
    ```
    
7. Save your changes and ensure the project builds without error.
    
8. In Solution Explorer, right-click the project and select **Publish to AWS Lambda**.  
    Complete the dialogs, and confirm region is to the same region you used in prior steps.  
    Function Name: **hello-bedrock-image**  
    Handler: **hello-bedrock-image::hello\_bedrock\_image.Function::FunctionHandler**  
    Click **Next**.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1698794056170/1a7b26c2-3025-4386-8c58-f5233ea24f8c.png align="center")
    
    On the second dialog page (Advanced Function Details), select the Lambda execution role you created earlier, **lambda-bedrock**.  
    Click **Upload** and wait for the upload to complete.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1698793287403/f061b4d5-7792-4491-9872-5640116b57d6.png align="left")
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1698784314890/a036da68-e7fd-45fe-b00d-e2cd0b5f237c.png align="left")
    

## Step 5: Configure Lambda Function Trigger

In this step, you'll configure an S3 trigger for the Lambda function in the AWS console.

1. In the AWS console, go to **Lambda**.
    
2. Your **hello-bedrock-image** function should be listed. Select it.
    
3. Select the **Configuration** tab.
    
4. On the left panel, select **Triggers**. Click **Add trigger**.  
    In Trigger Configuration - select a source, select **S3**.  
    In Bucket, select the bucket you created earlier.  
    Under Event Types, click **All object create events.**  
    Read and check the acknowledgement that using the same S3 bucket for input and output is not recommended and can cause recursive invocations.  
    Click **Add**.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1698793714741/17fca932-5f61-42bd-84bf-f77cdbe258bd.png align="left")
    

## Step 6: Test the Lambda Function

In this step, you'll test your Lambda function to confirm it's working or debug issues.

1. In the AWS console, navigate to **S3** and view your bucket.
    
2. On your local computer, create a text file named **santa-dog.txt** containing  
    **A dog wearing a santa cap.**
    
3. In the AWS console, upload **santa-dog.txt** to your bucket.
    
4. Wait a few tens of seconds and refresh the Objects view. Do this periodically until you see santa-dog.png also listed.  
    Select **santa-dog.png** and click **Open** to view the generated image. Congratulations! You've generated an image with Bedrock and Stable Diffusion XL using a .NET Lambda function.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1698785743342/aa5ffda8-9159-4a3a-9b13-31300e6f2eee.png align="left")
    
5. If a .png is NOT created after 60 seconds, then do the following:  
    Delete **santa-dog.txt** from the S3 bucket, as the Lambda function may be recursively trying to process it.  
    Navigate to **Lambda** in the console.  
    Select the **hello-bedrock-image** function  
    Select the **Monitor** tab and **Metrics**. Select **1h** (1 hour).  
    If an invocation is not shown in the Invocations view, wait and refresh until you see it.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1698793888506/bef35a3a-3759-48c1-895b-c01b813a73c3.png align="left")
    
    Click **Logs**. Click the earliest LogStream link. A separate browser tab will open up with a CloudWatch log. The screenshot below shows what a happy log output looks like. If something goes wrong, you should have exception detail in the log.
    
    View the log output to see if there is an error such as Access Denied and troubleshoot. Review the AWS configuration steps listed earlier. Make sure you didn't miss a step and check for typos. Be sure you used *your* AWS account number and bucket name in JSON you entered.  
    Then go back and try again by uploading santa-dog.txt again.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1698794381097/46475c7e-f639-4d6d-a3da-7d7f5ceca7aa.png align="center")
    
6. When things are working, try some other prompts. Here are some examples:  
    pizza.txt  
    A delicious pizza covered in meat and veggie toppings.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1698786533233/c292b05a-ae61-46f3-ab82-5db7f96d29b1.png align="left")
    
    roadway.txt  
    A rural roadway with a mailbox and barn.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1698786566742/32670e22-3684-47b6-83d4-c78b31bd7b75.png align="left")
    
    mansion.txt  
    A grove of palm trees lining a street to a mansion.
    
    ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1698786496816/8490a9e1-ba2f-4b9a-b81c-66ff66d3bc67.png align="left")
    

## Step 7. Shut it Down

When you're done using the project, delete the AWS artifacts to avoid incurring further charges. Delete the S3 objects and bucket and then the Lambda function.

## Understand the Code

Let's walk through the code. This function is triggered when a file is uploaded to S3. If it's a .txt file, Bedrock is called to generate an image and a .png file is created with the same name. Here's a breakdown of the code:

Lines 20-25: The `Function` class begins with an AmazonS3Client named `S3Client`. Two constants define the Stable Diffusion XL model ID and a string template for the request body the model expects. The constructor initializes `S3Client`.

53-62: The function handler is designed to service S3 events. Our trigger will be a new object created in our S3 bucket. The handler loops through each of the event(s).

64-72: In the try block, we first check whether the S3 object sent to us ends with the file type we expect: ".txt". If it doesn't, we continue on the next event. If we do have a text file, we compute the name of the S3 key for the output image file. Throughout the processing code we write log entries.

74-87: Lambda functions can be invoked more than once, so our function must be idempotent. We check whether the output image file already exists by calling GetObjectMetadataAsync. An S3Exception is what we expect if it doesn't exist. If the output image file is already there, we skip processing and continue on the next event.

89-97: We next read the contents of the .txt file, using S3Client.GetObjectAsync and reading the stream in the response. The contents are now in variable `prompt`.

99-114: Now we set up our Bedrock request. We instantiate an AmazonBedrockClient named `bedrockClient`, specifying the endpoint we're working in. Then we create a request body for the Stable Diffusion XL model, inserting the prompt into the request body template with String.Format. That gets turned into JSON, then a byte array, and finally a MemoryStream. Finally, we create an InvokeModelRequest that contains the model Id, content type and accept headers for JSON, and the request body stream.

116-124. We invoke Bedrock with the request. We read the response body with a StreamReader and deserialize the JSON into an object, a property of which contains an image encoded as a base64 string.

126-140: To convert the image from base64 to an image we can save in a cross-platform fashion, we use ImageSharp. We converting the base64 string to a byte\[\] array and use ImageSharp's Image.Load method to create an Image object. The image is then saved in .png format to a stream. Finally, the stream is uploaded to S3 using the S3 SDK's TransferUtility.UploadAsync method.

This code uses the ImageSharp library because the traditional System.Drawing libraries can't be used on Linux where Lambda functions execute. If you use ImageSharp, be aware of its [license](https://sixlabors.com/pricing/) and what conditions qualify as commercial use. For other alternatives, see [System.Drawing.Common only supported on Windows](https://learn.microsoft.com/en-us/dotnet/core/compatibility/core-libraries/6.0/system-drawing-common-windows-only).

# Where to Go from Here

In this tutorial, you learned about Amazon Bedrock image processing and the Stable Diffusion XL model, and sampled image generation in the AWS console. You wrote a Lambda function to respond to uploaded S3 .txt file prompts, invoke Bedrock to generate images, and save the images to S3 as .png files. Now you'll want to learn as much as you can about the Stable Diffusion XL model so you can master prompts.

We only explored image generation today. Bedrock also has text and chat capabilities you'll want to get familiar with. Experiment with Bedrock. Learn the ins and outs of your chosen model. Experimentation is key to understanding Generative AI and the strengths of different foundation models. Learn to perfect your prompts and tune your model for optimal results.

# Further Reading

AWS Documentation

[Amazon Bedrock](https://aws.amazon.com/bedrock/)

[Amazon Bedrock User Guide](https://docs.aws.amazon.com/bedrock/latest/userguide/what-is-service.html)

[AWS SDK for .NET - AmazonBedrockClient](https://docs.aws.amazon.com/sdkfornet/v3/apidocs/items/Bedrock/TBedrockClient.html)

Stable AI Documentation

[Stable Diffusion prompt: a definitive guide](https://stable-diffusion-art.com/prompt-guide/)