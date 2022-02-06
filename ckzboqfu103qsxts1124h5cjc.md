## Hello, Translate!

#### This episode: Amazon Translate and Neural Machine Translation. In this Hello, Cloud blog series, we're covering the basics of AWS cloud services for newcomers who are .NET developers. If you love C# but are new to AWS, or to this particular service, this should give you a jumpstart.

In this post we'll introduce Amazon Translate and use it in a "Hello, Cloud" .NET program to perform language translation. We'll do this step-by-step, making no assumptions other than familiarity with C# and Visual Studio. We're using Visual Studio 2022 and .NET 6.

# Amazon Translate : What is it, and why use It?

> "Communication is the key to success in any business." —Lee Brown

Ever since the Tower of Babel, communication has been a challenge. As globalization increases, so does the need to communicate effectively across regions, cultures, and languages.
 
[Amazon Translate ](https://aws.amazon.com/translate/) (hereafter "Translate") is a service that performs automated language translation. AWS describes it as "a neural machine translation service that delivers fast, high-quality, affordable, and customizable language translation." With Translate, you can add localization to your web site and make such things as content feeds, reviews, and comments available in a user's preferred language. Translate supports [75 languages](https://docs.aws.amazon.com/translate/latest/dg/what-is.html). 

Translate uses neural networks trained for language translation. Machine translation will result in a small amount of translation imperfection. You should evaluate your translation needs carefully, and decide whether you need to combine automated machine translation with light human editing. Translate can translate high volumes of text that might overwhelm human translators. 

You can perform translation in real-time, passing Translate a request with text to be translated, source language, and destination language.

![diagram-translate.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1643479102035/lxBSOFqLP.png)

You can also perform translation in batches on files in S3 using jobs, known as [Asynchronous Batch Processing](https://docs.aws.amazon.com/translate/latest/dg/async.html). You upload a set of documents to an S3 input folder, start a batch translation job, and retrieve results from an S3 output folder. In addition to plain UTF-8 text, Translate can also translate documents such as .docx, .xlsx, and .pptx files.

![diagram-translate-batch.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1643479897144/JQaoehVay.png)

Amazon Translate provides features for customizing your translations. You can mask profanity in the translation output. You can define custom terminology and define how you want specific terms to be translated. The active custom translation feature allows you to influence the style, tone, and word choice by providing sample translations.

Amazon Translate can put translation in reach, whether you're targeting a global audience or simply need to support a second language. Automated translation with Translate is about 1,000x cheaper than professional human translation. The AWS Free Tier gives you 2 million characters free per month, which makes it easy to experiment with. 

Amazon Translate combines well with other AWS machine learning services. For example, you can use Amazon Textract to extract text from PDF documents or images, then feed the text to Translate for translation, and then pass the translated text to Amazon Comprehend for sentiment analysis or convert to speech with Amazon Polly.

# Our Hello, Translate Project

After getting familiar with Amazon Translate in the AWS Console, you'll write a .NET 6 program that invokes Translate to translate text files from and to any supported language. We'll first do this in real-time, then see how to translate documents with a batch job.

[source code](https://github.com/davidpallmann/hello-translate)

![dotnet-run-the-road-not-taken.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1643480154870/XvqBGP8Eb.png)

![frost-output-docx-word.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1644087742107/ek4UzrYnj.png)

# One-time Setup

For any of the tutorials in the Hello, Cloud series you need the following:

1. An  [AWS account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/), and an understanding of what is included in the  [AWS Free Tier](https://aws.amazon.com/premiumsupport/knowledge-center/what-is-free-tier/). 
2. [Microsoft Visual Studio 2022](https://visualstudio.microsoft.com/). If you're using an older version of Visual Studio you won't be able to use .NET 6. If you use a different IDE, you may have to find alternatives to some tutorial steps.
3. [AWS Toolkit for Visual Studio](https://aws.amazon.com/visualstudio/). [Configure ](https://docs.aws.amazon.com/toolkit-for-visual-studio/latest/user-guide/keys-profiles-credentials.html) the toolkit to access your AWS account and create an IAM user. Your default AWS profile will be linked to this user when running programs from the command line.

## Step 1: Set Permissions for the AWS Toolkit User

In order to perform this tutorial, your AWS Toolkit User / default AWS profile needs the necessary permissions for Amazon Translate operations. In this step, you'll update permissions for your AWS Toolkit for Visual Studio user. 

Important: You should always follow the principle of [least privilege](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#grant-least-privilege) when it comes to security. This blog series assumes you are working in your own developer account for learning purposes and not working with anything production-related. If you're using an organization-owned AWS account, you should coordinate with your security principal on IAM settings. 

1. Sign in to the [AWS console](https://aws.amazon.com/console/). 

2. Navigate to the **Identity and Access Management (IAM)** area of the AWS Console. You can enter **iam** in the search box to find it.

3. Click **Users** on the left panel and select the username you use with the AWS Toolkit for Visual Studio (you created this user when you installed and configured the toolkit).

4. If not already assigned, add the built-in **PowerUserAccess** permission. The  [PowerUserAccess](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_job-functions.html) permission provides developers full access to AWS services and resources, but does not allow management of users and groups.

5. Find and attach the **TranslateFullAccess** permission.

    ![iam-add-policy.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1644081791987/fFwYOQGc0.png)

Your AWS Toolkit for Visual Studio user now has the permissions it needs to perform real-time Translate operations. We'll add more permissions for batch jobs in a later step.

## Step 2: Use Amazon Translate in the AWS Console

In this step, you'll get some experience working with Translate interactively.

1. Go to the AWS Console and select a region nearest you that supports Amazon Translate (check availability [here](https://aws.amazon.com/translate/faqs/)). We're using **US-West-2 (Oregon)**.  

2. In the left panel, click **Real-time translation**. A real-time translation form appears.

3. On the Translation pane, select Source language, Target language, and enter source text. You see a translation appear on the right. We translated a service review from English to Greek below.

    ![02_translate_en_el.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1644071870387/YlTtKtHfV.png)

4. Click the middle button to reverse the translation. The Greek translation is translated back into English. In this instance, meaning has been preserved well.

    ![02_translate_el_en_reverse.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1644071774380/puBRCXmcd.png)

5. Experiment with a variety of text and source/destination language combinations to get a feel for Amazon Translate. Note the two-letter language codes used to specify source and destination. You'll be specifying these language codes in your code ([language code reference](https://docs.aws.amazon.com/translate/latest/dg/what-is.html)).

    ![02_translate_en_it.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1644072923475/uhptsGMmp.png)

Now that we understand what Amazon Translate can do, let's use it programmatically.

## Step 3: Create a .NET Program that Uses Translate

In this step, we'll create a .NET console program and use it to invoke Translate on various text files.

1. Open a command/terminal window and CD to a development folder.

2. Enter the dotnet new command below to create a .NET 6 console program.

    ```none
dotnet new console -n hello-translate
```

    ![02-dotnet-new.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1644073017819/o7txlSPKY.png)

3. Launch Visual Studio and open the hello-translate project.

4. In Solution Explorer, right-click the hello-translate project and select **Manage Nuget Packages...**.  Find and install the **AWSSDK.Translate package**.

    ![02-nuget.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1644073067702/qKHNM4ozh.png)

5. Replace Program.cs with the code at the end of this step. Set region to the Amazon region code for your region. For example, for the region of Oregon we specify RegionEndpoint.USWest2. 

6. In Solution Explorer, right-click the hello-translate project and select Add Class. Add a class named TranslateHelper.cs. Open it in the editor and replace it with the code at the end of this step. This is where our code will be interacting with Amazon Translate via the AWS SDK for .NET.

7. Save your changes and build the program.

Program.cs

```csharp
using Amazon;
using hello_translate;

RegionEndpoint region = RegionEndpoint.USWest2;

Console.OutputEncoding = System.Text.UnicodeEncoding.Unicode;

var translateHelper = new TranslateHelper(region);

if (args.Length == 3)
{
    await translateHelper.TranslateText(sourceLang: args[0], targetLang: args[1], text: args[2]);
}
else if (args.Length == 4)
{
    await translateHelper.TranslateBatch(sourceLang: args[0], targetLang: args[1], bucketName: args[2], docType: args[3]);
}
else
{
    Console.WriteLine("For real-time translation of local text files:");
    Console.WriteLine("Usage: dotnet run -- [sourceLanguageCode] [destLanguageCode] [filename]");
    Console.WriteLine("Ex:    dotnet run -- en de \"Hello there. My name is David.\"");
    Console.WriteLine("Ex:    dotnet run -- fr ru data\\frenchText.txt");
    Console.WriteLine("For batch translation of S3 documents (using the bucket configured in the program):");
    Console.WriteLine("Usage: dotnet run -- [sourceLanguageCode] [destLanguageCode] [s3-uri] txt|html|docx|xlsx|pptx");
    Console.WriteLine("Ex:    dotnet run -- en fr s3://hello-translate docx");
}
```

TranslateHelper.cs

```csharp
using Amazon;
using Amazon.S3;
using Amazon.Translate;
using Amazon.Translate.Model;

namespace hello_translate
{
    public class TranslateHelper
    {
        const string DataAccessRoleArn = "arn:aws:iam::[aws-account]:role/translate-s3";

        private RegionEndpoint _region { get; set; }
        private AmazonTranslateClient _translateClient { get; set; }
        private AmazonS3Client _s3Client { get; set; }

        public TranslateHelper(RegionEndpoint region)
        {
            _region = region;
            _translateClient = new AmazonTranslateClient(_region);
            _s3Client = new AmazonS3Client(_region);
        }

        /// <summary>
        /// Translate text.
        /// </summary>
        /// <param name="sourceLang">source language code</param>
        /// <param name="targetLang">target language code</param>
        /// <param name="text">text to translate</param>
        /// <returns>translated text</returns>
        public async Task<string> TranslateText(string sourceLang, string targetLang, string text)
        {
            if (text.Contains("\\"))
            {
                text = File.ReadAllText(text);
            }
            Console.WriteLine($"--- {sourceLang} ---");
            Console.WriteLine(text);

            var request = new TranslateTextRequest
            {
                Text = text,
                SourceLanguageCode = sourceLang,
                TargetLanguageCode = targetLang
            };

            var response = await _translateClient.TranslateTextAsync(request);

            Console.WriteLine($"--- {targetLang} ---");
            Console.WriteLine(response.TranslatedText);

            return response.TranslatedText;
        }

        /// <summary>
        /// Translate documents using a batch job. Input and output folders are named with the source/target language codes. 
        /// </summary>
        /// <param name="sourceLang">source language code</param>
        /// <param name="targetLang">target language code</param>
        /// <param name="bucketName">S3 URI, minus the folder, for source/target documents</param>
        /// <param name="docType">document type - docx | xlsx | pptx | html | txt</param>
        public async Task TranslateBatch(string sourceLang, string targetLang, string s3Uri, string docType)
        {
            if (s3Uri != null && !s3Uri.EndsWith("/")) s3Uri += "/";
            var inputConfig = new InputDataConfig()
            {
                S3Uri = $"{s3Uri}{sourceLang}", 
                ContentType = docType switch
                {
                    "docx" => "application/vnd.openxmlformats-officedocument.wordprocessingml.document",
                    "xlsx" => "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet",
                    "pptx" => "application/vnd.openxmlformats-officedocument.presentationml.presentation",
                    "htm" => "text/html",
                    "html" => "text/html",
                    "txt" => "text/plain",
                    _ => "text/plain"
                },
            };

            var outputConfig = new OutputDataConfig()
            {
                S3Uri = $"{s3Uri}{targetLang}",
            };

            var request = new StartTextTranslationJobRequest()
            {
                InputDataConfig = inputConfig,
                OutputDataConfig = outputConfig,
                SourceLanguageCode = sourceLang,
                TargetLanguageCodes = new List<string> { targetLang },
                DataAccessRoleArn = DataAccessRoleArn
            };

            Console.WriteLine($"Start batch translation - S3 input folder: {inputConfig.S3Uri}, S3 output folder: {outputConfig.S3Uri}");

            var response = await _translateClient.StartTextTranslationJobAsync(request);

            if (response.HttpStatusCode == System.Net.HttpStatusCode.OK)
            {
                Console.WriteLine($"Job {response.JobId} is running - status: {response.JobStatus}");
            }
            else
            {
                Console.WriteLine($"Error: {response.HttpStatusCode} {response.JobStatus}");
            }

        }
    }
}
```

The code is short, so let's take a moment to understand it. This program can be used for 1) real-time translation or 2) batch processing of documents in S3, specified by the command line arguments. The code is simple to understand. The TranslateHelper class does all the work. Program.cs instantiates TranslateHelper, passing an AWS region to its constructor. It then calls the appropriate method, TranslateText or TranslateBatch. In TranslateHelper.cs, the constructor creates an AmazonTranslateClient.

For real-time translation, we invoke the TranslateText method of TranslateHelper. That method checks the `text` parameter, which may contain text or a filename. If it contains a slash or backslash, it's taken as a filename and the source text is loaded from that file. A TranslateTextAsync SDK method performs the actual translation. We give that a TranslateTextRequest with source text, source language code, and destination language code. The TranslateTextResponse we receive back contains a TranslatedText property with the translation. Easy-peasy.

Batch job processing of an S3 folder of documents is slightly more involved, and we still have a step coming up to create a bucket and configure permissions for that. The code in our TranslateBatch method accepts source/target language codes, a bucket name, and a doc type, which can be docx, xslx, pptx (Microsoft Office documents),  html (HTML page), or txt (text). The StartTextTranslationJobAsync method kicks off the job. We pass it a StartTextTranslationJobRequest with the URI of our input S3 bucket/folder, the document content type, the output S3 bucket/folder, source/target language codes, a role for accessing the S3 bucket/objects, and a name for our job. We get back a StartTextTranslationJobResponse, whose `JobId` can be used to check job status. Our program kicks off the batch job but doesn't wait for it, as batch jobs can take quite some time.  We can check completion and see our results in the AWS console.

We're now ready to try out the first action, real-time translation. We still have some setup work to do before we can try batch processing, which we'll get to in a later step.

## Step 4: Run the Program and Test Real-time Translation

In this step, we'll run our program and test real-time translation. Our program expects these command line arguments:

   ```dos
   dotnet run -- [sourceLanguageCode] [destLanguageCode] [text|filename]
   ```

1. Open a command/terminal window and CD to the project folder.

2. Enter this command to translate "Do you speak French?" from English to French:

   ```dos
   dotnet run -- en fr "Do you speak French?"
   ```
    The translation appears: "Parlez-vous français ?". 

    ![04_dotnet_run_text_en_fr.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1644175464595/T9pekxbXk.png)

3. Try other text and source/target language codes.

4. Now, create a .txt file and put your source text in the file.

5. Run the command again, this time specifying source language code, target language code, and text filename. The translation is displayed. Here is how our service review from Step 2 looks, translated to German:

    ![04_dotnet_run_file_en_de.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1644077886314/tLpMWN1VT.png)

3. Try some other text examples, with longer and different kinds of text. We translated this public-domain short story from English to Spanish:

    ![04_dotnet_run_sentry_en_sa.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1644078202286/idsBsspm6.png)

    [source: Project Gutenberg](https://www.gutenberg.org/cache/epub/29948/pg29948.txt)

Congratulations, you've performed neural language translation in .NET code using Amazon Translate!

## Step 5: Create an S3 Bucket for Batch Processing

In this step we'll create an S3 bucket for batch processing jobs with Amazon Translate. You would find this mode appropriate when you have many things to translate, or want to translate a batch of document files, such as Microsoft Office documents. 

1. In the AWS console, navigate to **Amazon S3**. You can enter **s3** in the search bar.

2. Click **Create bucket** and create a bucket named **hello-translate** (if name is in use, try a variation). 

3. View the bucket. On the Objects tab, click **Create folder** and create a folder named **en**. This will be the input folder for documents.

4. Click the **en/** folder to see its detail. Select the Properties tab and make a note of the S3 URI for the input folder. Ours is **s3://hello-translate/en/**.

    ![04_s3_en_folder_uri.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1644080235608/oydxYx6TY.png)

5. Go back up a level to the hello-translate bucket. Click **Create Folder** to add another folder and name it **fr**. This will be our output folder for translated documents. 

6. Click the **fr/** folder to see its detail. Select the Properties tab and record the S3 URI for the output folder. Ours is **s3://hello-translate/fr/**.

7. You now have an S3 bucket with an en (English) and fr (French) folder. Feel free to add additional folders now or down the road for other language codes.

    ![05_s3_folders_created.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1644080754239/OeZonpfKO.png)

## Step 6: Create a Policy and Role for Data Access

Now we need to configure permissions, and this step is a bit involved. In this step you'll configure a policy, role, and trust relationship, which will allow Amazon Translate to access your S3 bucket when executing a batch processing job.  

1. In the AWS console, navigate to **Identity and Access Management**. You can enter **iam** in the search bar.

2. On the left panel, click **Policies**. 

3. Click **Create Policy**. On the Create policy page, enter/select the following:

    a. Select the **JSON** tab and replace it with the JSON at the end of this step, replacing **hello-translate** with the S3 bucket name you created in Step 5. 

    ![02-policy.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1644082029985/6W0nSgfo1.png)

    b. Click the **Next: Tags** and **Next: Review** buttons.

    c. Enter policy name **translate-s3**. 

    d. click **Create policy**.

    ![02-policy-2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1644081913942/tC6HjrAwR.png)

    Once we attach it to a role, this policy will permit Amazon Translate to read your S3 bucket/objects and create objects.

4. On the left panel, select **Roles**.

5. Click **Create role**. On the Create role page, enter/select the following:

    a. In the search box, enter **translate-s3** and check it.

    b. Click **Next: Tags**.

    c. Select **EC2**. Why EC2? There is no Amazon Translate service listed on this page. EC2 is what you need to use.

    d. Click **Next: Permissions**, **Next: Tags**, and **Next: Review**.

    e. For role name, enter **translate-s3**.

    f. Click **Create role**.

6. Select the new **translate-s3** role to view its detail. Find and note the new role's Amazon Resource Name (ARN).

    ![06-create-role-permissions.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1644175790190/acMj0h0uF.png)

7. Select the **Trust relationships** tab and enter/select the following:

    a. In the Trusted entities edit box, enter the trust relationship JSON at the end of this step. This gives the Amazon Translation service and the EC2 instances it controls permission to assume roles.

    b. Click **Update policy**. 

    ![06-create-role-trust-relationships.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1644175806139/Q9P17etofG.png)

translate-s3 policy JSON

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "s3:GetObject",
            "Resource": [
                "arn:aws:s3:::hello-translate/*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": "s3:ListBucket",
            "Resource": [
                "arn:aws:s3:::hello-translate"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:PutObject"
            ],
            "Resource": "arn:aws:s3:::hello-translate/*"
        },
        {
            "Sid": "AllowTranslation",
            "Effect": "Allow",
            "Resource": "*",
            "Action": "translate:*"
        }
    ]
}
```

Trust relationship JSON

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "ec2.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        },
        {
            "Sid": "",
            "Effect": "Allow",
            "Principal": {
                "Service": "translate.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
```

## Step 7: Run the Program and Test Batch Translation Processing

In this step you'll update and run the .NET program, and test batch translation processing of Microsoft Word documents. Expect that batch jobs may take some time. Batch jobs are long-running operations whose execution time is affected by the size of the dataset and resource availability. 

1. In Visual Studio, open TranslateHelper.cs in the editor and set the constant DataAccessRoleArn to the ARN you just recorded in Step 6.

2. Save your changes and build the program.

3. Next we need some source documents for the translation. Find or create one or more Microsoft Word documents containing English text, saved as .docx files. We're using one document, a poem by Robert Frost in the public domain, the Road Not Taken, in file frost.docx.

    ![frost-input-docs-word-en.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1644085009390/4OjR5zgPG.png)

    [source: Project Gutenberg](https://www.gutenberg.org/cache/epub/29948/pg29948.txt)

3. In the AWS console, navigate to **Amazon S3** > your bucket > **en** folder.

4. Upload your Microsoft Word test document(s) to the en folder.

    ![02-upload-en-docx.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1644085082074/ADZLHbmoJ.png)

5. Open a command/terminal window and CD to the project folder.

6. Run your program with dotnet run -- and these 4 command line arguments: 1) source language **en**, 2) target language **fr**, 3) the URI of your S3 bucket (without the folder part), and 4) the document type **docx**.

   ```dos
dotnet run -- en fr s3://hello-translate docx
```
7. Now the program runs. The output should confirm an Amazon Translate job has been kicked off and show its job Id. The job will run asynchronously, and we'll have to check on it (manually in the AWS console or with code) to know when it's finished.

    ![dotnet-run-job-docx.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1644085938072/Zg3eym-3N.png)

    If you receive a permissions error, check that you full performed Steps 5 and 6, or consult this [Troubleshooting Amazon Translate Identity and Access](https://docs.aws.amazon.com/translate/latest/dg/security_iam_troubleshoot.html) page.

7. In the AWS console, navigate to **Amazon Translate** and select **Batch translation** from the left panel. You should see your batch job listed, with a status of **In progress**.

    ![console-job-in-progress.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1644086009453/ftTBjQP62.png)

8. Periodically return to check the status of your batch job. Wait until its status is **Completed**.

    ![console-job1-complete.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1644086108268/Ll9I7W26T.png)

9. In the AWS console, navigate to **Amazon S3** > your bucket > the **fr** folder. Within a sub-folder, you should see copies of your documents, with their names prefixed by "fr.". There should be the same number of output documents as there were input documents. 

    ![console-job1-out-fr.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1644086184376/afjyDmNwD.png)

    There's also a folder named **details**, which contains a job summary including an error count.

    ```json
{"sourceLanguageCode":"en","targetLanguageCode":"fr","charactersTranslated":"1038","documentCountWithCustomerError":"0","documentCountWithServerError":"0","inputDataPrefix":"s3://hello-translate/en/","outputDataPrefix":"s3://hello-translate/fr/xxxxxxxxxx-TranslateText-xxxxxxxxxxx/","details":[{"sourceFile":"frost.docx","targetFile":"fr.frost.docx","auxiliaryData":{}}]}
```

10. Review your translated document(s) by downloading them from the S3 console and opening them in Microsoft Word. Here's what our French translation of The Road Not Taken looks like:

    ![frost-output-docx-word.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1644086642937/VxTVKnRdx.png)

11. Try other batch jobs and languages. Remember to create the S3 bucket folder for your source and target languages, delete previous documents, and upload new source documents.

## Step 8: Shut it Down

When you're completely finished with hello-translate, follow this step to delete AWS resources. You don't want to be charged for things you're no longer using.

1. In the AWS console, navigate to **Amazon Translate** and select **Batch translation**. If you have any jobs still in progress that you wish to interrupt, select each one and click the **Stop** button at top right. Wait for confirmation that the jobs are stopped.

2. In the AWS console, navigate to **Amazon S3**. Delete the objects in your S3 bucket, and then the bucket itself.

# Where to Go From Here

Can we talk? Your applications can leverage Amazon Translate for high-quality, affordable language translation. In this tutorial, you learned how Amazon Translate works. You experimented with it in the AWS console, then wrote .NET code to work with it. You translated text in real-time with the AWS SDK for .NET. You then used batch processing to translate Microsoft Word documents in S3. The code was simple.

To move forward with Translate, identify your translation needs. Do you need full automation, or automation plus human review? Investigate the features we did not cover in this tutorial: profanity masking, custom terminology, and active custom translation.

# Further Reading

AWS Documentation

[Amazon Translate](https://aws.amazon.com/translate/)

[Amazon Translate Developer Guide](https://docs.aws.amazon.com/translate/latest/dg/translate-dg.pdf)

[Amazon Translate Languages and Codes](https://docs.aws.amazon.com/translate/latest/dg/what-is.html)

[Asynchronous Batch Processing with Amazon Translate](https://docs.aws.amazon.com/translate/latest/dg/async.html)

[Running a Batch Translation Job](https://docs.aws.amazon.com/translate/latest/dg/async-start.html)

[Amazon Translate Examples](https://docs.aws.amazon.com/translate/latest/dg/examples.html)

Videos

[Using Amazon Translate, Amazon Polly, and Amazon Rekognition with .NET](https://www.youtube.com/watch?v=UO0hGTvqLUs) by Thorr Giddings

Blogs

[Power your website with on-demand translated reviews using Amazon Translate](https://aws.amazon.com/blogs/machine-learning/power-your-website-with-on-demand-translated-reviews-using-amazon-translate/)

[Amazon Transcribe Now Supports Mandarin and Russian](https://aws.amazon.com/blogs/aws/amazon-transcribe-now-supports-mandarin-and-russian/)

[Hello, Cloud blog series](https://davidpallmann.hashnode.dev/series/hello-cloud)

