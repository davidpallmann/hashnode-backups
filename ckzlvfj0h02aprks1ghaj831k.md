## Hello, Polly!

#### This episode: Amazon Polly and Text-to-Speech. In this Hello, Cloud blog series, we're covering the basics of AWS cloud services for newcomers who are .NET developers. If you love C# but are new to AWS, or to this particular service, this should give you a jumpstart.

In this post we'll introduce Amazon Polly and use it in a "Hello, Cloud" .NET program to perform Text-to-Speech. We'll do this step-by-step, making no assumptions other than familiarity with C# and Visual Studio. We're using Visual Studio 2022 and .NET 6.

# Amazon Polly : What is it, and why use It?

Does your application need speech capability? There are multiple reasons you might want to provide an audio alternative to visual content. Impaired or elderly users may have challenges with visual content. Mobile users might prefer content they can listen to while on the go. Some people are auditory learners who get more from listening than viewing. 

[Amazon Polly ](https://aws.amazon.com/polly/) (hereafter "Polly") is a service that converts text to speech (TTS). AWS describes it as a service that "uses advanced deep learning technologies to synthesize natural sounding human speech". Polly supports [dozens of languages and voices](https://docs.aws.amazon.com/polly/latest/dg/voicelist.html), giving your voice-enabled applications broad reach.

Polly is much more than a robotic speech synthesizer. It's mission is to provide lifelike speech. You can choose between standard TTS voices and Neural Text-to-Speech (NTTS) voices for advanced speech quality. At the time of this writing, NTTS is about 4 times the cost of TTS (see [pricing model](https://aws.amazon.com/polly/pricing/)), so you'll want to evaluate the two against your needs. The AWS free tier includes, for the first 12 months, 5M characters per month for standard voices and 1M characters per month for neural voices.

To synthesize speech with Polly, you provide text (up to 3,000 characters) and specify the language, a voice, and output format (such as MP3). You get an audio stream in the response which you can then use as you see fit, such as saving to a local MP3 file. For larger text sizes, you can process up to 100k characters of text in a request via an S3 bucket. 

![diagram-tts-text.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1645275987158/zpaE9KEZA.png)

Alternatively, for more control over the nuances of speech, you can provide Speech Synthesis Markup Language ([SSML](https://docs.aws.amazon.com/polly/latest/dg/supportedtags.html)) instead of plain text. This will allow you to add nuance to the synthesis. For example, you can insert pauses, switch to a whisper, change the rate of speech, or insert breath noises.

![diagram-tts-ssml.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1645275952942/lRx1t2Qc_.png)

You can create lexicons for Polly, small XML sequences which customize what Polly says for a given term. This can be useful for controlling how words and phrases uncommon to the chosen language are pronounced, or for expanding abbreviations and acronyms.

By the way, see that "Listen to this article" link at the top of this post? You can **listen** to these blog posts—a feature powered by Amazon Polly, courtesy of Hashnode.

# Our Hello, Polly Project

After getting familiar with Amazon Polly in the AWS console, you'll write a .NET 6 console program that converts text or SSML to synthesized speech, saved as an MP3 file.

![04-run-joey-test.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1644788948003/0EiHO704z.png)

[▶ listen](https://david-pallmann-blog-audio.s3.us-west-2.amazonaws.com/hellopolly/dotnet-run-joey-tts.mp3)

![04-run-ssml.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1644791953049/0hc-ZyS19.png)

[▶ listen](https://david-pallmann-blog-audio.s3.us-west-2.amazonaws.com/hellopolly/ssml-joanna.mp3)

[source code](https://github.com/davidpallmann/hello-polly)

## One-time Setup

For any of the tutorials in the Hello, Cloud series you need the following:

1. An  [AWS account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/), and an understanding of what is included in the  [AWS Free Tier](https://aws.amazon.com/premiumsupport/knowledge-center/what-is-free-tier/). 
2. [Microsoft Visual Studio 2022](https://visualstudio.microsoft.com/). If you're using an older version of Visual Studio you won't be able to use .NET 6. If you use a different IDE, you may have to find alternatives to some tutorial steps.
3. [AWS Toolkit for Visual Studio](https://aws.amazon.com/visualstudio/). [Configure ](https://docs.aws.amazon.com/toolkit-for-visual-studio/latest/user-guide/keys-profiles-credentials.html) the toolkit to access your AWS account and create an IAM user. Your default AWS profile will be linked to this user when running programs from the command line.

## Step 1: Set Permissions for the AWS Toolkit User

In order to perform this tutorial, your AWS Toolkit User / default AWS profile needs the necessary permissions for Amazon Polly operations. In this step, you'll update permissions for your AWS Toolkit for Visual Studio user. 

Important: You should always follow the principle of [least privilege](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#grant-least-privilege) when it comes to security. This blog series assumes you are working in your own developer account for learning purposes and not working with anything production-related. If you're using an organization-owned AWS account, you should coordinate with your security principal on IAM settings. 

1. Sign in to the [AWS console](https://aws.amazon.com/console/). 

2. Navigate to the **Identity and Access Management (IAM)** area of the AWS Console. You can enter **iam** in the search box to find it.

3. Click **Users** on the left panel and select the username you use with the AWS Toolkit for Visual Studio (you created this user when you installed and configured the toolkit).

4. If not already assigned, add the built-in **PowerUserAccess** permission. The  [PowerUserAccess](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_job-functions.html) permission provides developers full access to AWS services and resources, but does not allow management of users and groups.

Your AWS Toolkit for Visual Studio user now has the permissions it needs to perform text-to-speech operations with Amazon Polly. 

## Step 2: Use Amazon Polly in the AWS Console

In this step, you'll get some experience working with Polly interactively.

1. Go to the AWS Console and select a region nearest you that supports Amazon Polly (check availability [here](https://aws.amazon.com/polly/faqs/?nc=sn&loc=8)). We're using **US-West-2 (Oregon)**.  

2. Navigate to Amazon Polly in the console. You can enter **polly** in the search box.

3. In the left panel, click **Text-to-Speech**. 

4. On the Text-to-Speech page, enter some text into the Input text textbox. Click **Listen**, and you will hear the text as speech. In my case, the console defaults language `English, US` and voice `Joanna, Female`. 

    ![02_aws_polly_tts.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1644609504607/zxLRagAH6.png)

5. Click the Voice drop-down to see the various voices available. Experiment with different voices and text to see how differently they sound when you click **Listen**. I selected voice `Matthew, Male` and changed the input text to a payment due notification. 

    ![02_aws_voices.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1644699676894/vyRWOKw6H.png)

6. Now choose a different language, and adjust the text to match that language. I switched from `English, US` to `English, British`. Now click *Listen*, and hear your text in an appropriate voice and accent for your selected language. 

    ![02_aws_languages.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1644700304512/9QeGhYsGw.png)

    Notice that the voice list changed after you changed language. Each language has its own set of voices. Again, try different voices and see how they compare.

    ![02_aws_british_voices.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1644700572346/bHRzd9vJa.png)

7. Lastly, switch between **Neural** and **Standard** and compare the quality of the 
synthesis as you listen.

## Step 3: Teach Amazon Polly a Custom Pronunciation

In this step, you'll learn how to teach Polly how to pronounce things differently using two techniques, Pronunciation Lexicon Specification (PLS) and Speech Synthesis Markup Language (SSML).

1. First, we'll look at how to replace a word in the text with something else during speech synthesis. You might use this to expand an abbreviation or acronym. 

    A. Before we configure anything, enter and listen to this text: "AWS is my favorite cloud." You hear AWS synthesized as "A W S". What we're going to do is instruct Polly to instead say the full name "Amazon Web Services" when it encounters "AWS".

    B. Create a local text file named aws.xml with the XML at the end of this step. This XML dialect is called Pronunciation Lexicon Specification (PLS).

    C. On the Lexicons page,  

    1) click **Upload lexicon**. for Name, enter "AWS". 

    2) Click the **Choose a lexicon file** button and choose the aws.xml file you just created.

    ![02-aws-pls.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1644705750288/1d0yrDgIE.png)

    C. Click **Text-to-Speech** on the left pane.

    D. Expand the **Additional settings** section and toggle **Custom pronunciation** on.

    E. In the Apply lexicon box, find and select **aws**, the lexicon you just uploaded.

    ![02-aws-enable-customize.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1644706160573/O852RLvoN.png)

    F. Click **Listen**. This time, "AWS" in the text should be pronounced "Amazon Web Services." 

2. Next, we'll add nuance to our speech using SSML.

    A. Click **Text-to-Speech** on the left pane.

    B.  Toggle **SSML** on.

    C. Select **Standard** and any voice. We're using **Joanna, Female**. Not all SSML features are supported by neural voices.

    D. In the Input text box, enter the SSML at the end of this step. This SSML contains 1) a break tag, which wait 2 seconds after the first sentence, 2) a whispering effect for the third sentence, 3) slow reading of a sentence, 4) fast reading of a sentence, and 5) breathing sounds. 

    E. Click **Listen**. You now hear a very nuanced speech synthesis.

    ![03-aws-ssml.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1644792145181/UGFdpYNY4.png)

    [▶ listen](https://david-pallmann-blog-audio.s3.us-west-2.amazonaws.com/hellopolly/ssml-joanna.mp3)

You've learned how to control Polly's speech synthesis! Henry Higgins couldn't be prouder.

aws.xml

```XML
<?xml version="1.0" encoding="UTF-8"?>
<lexicon version="1.0" 
      xmlns="http://www.w3.org/2005/01/pronunciation-lexicon"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
      xsi:schemaLocation="http://www.w3.org/2005/01/pronunciation-lexicon 
        http://www.w3.org/TR/2007/CR-pronunciation-lexicon-20071212/pls.xsd"
      alphabet="ipa" 
      xml:lang="en-US">
  <lexeme>
    <grapheme>W3C</grapheme>
    <alias>World Wide Web Consortium</alias>
  </lexeme>
</lexicon>
```

SSML

```XML
<speak>Hi! My name is Joanna. <break time="2s"/>I will read any text you type here. <amazon:effect name="whispered">I can keep a secret!</amazon:effect> <prosody rate="75%">I can read text slowly.</prosody><prosody rate="150%">I can read quickly, too.</prosody>Just a minute while I run upstairs. <amazon:auto-breaths> Wow, <amazon:breath/> that was a workout! I must be out of shape.</amazon:auto-breaths></speak>
```

## Step 4: Create a .NET Program

In this step you'll create a .NET console program that can translate text to speech with Amazon Polly.

1. Open a command/terminal window and CD to a development folder.

2. Enter the dotnet new command below to create a new console program named **hello-polly**.

    ```dos
dotnet new console -n hello-polly
``` 
    ![03_dotnet_new.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1644700910932/oA6lx-6F8.png)

3. Launch Visual Studio and open the hello-polly project.

4. In Solution Explorer, right-click the **hello-polly** project and select **Manage NuGet Packages...**. Find and install the **AWSSDK.Polly** package.

    ![03_nuget.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1644701216036/ykwc8jtuI.png)

5. Open Program.cs and replace with the code below. Set RegionEndpoint to the region you wish to use. We're using **USWest2** for Oregon.

```csharp
using Amazon;
using Amazon.Polly;
using Amazon.Polly.Model;
using System.IO;

RegionEndpoint region = RegionEndpoint.USWest2;

if (args.Length >= 3)
{
    var pollyClient = new AmazonPollyClient(region);

    var languageCode = args[0];
    var voiceId = args[1];
    var text =   args[2];

    if (text.Contains("/") || text.Contains(@"\"))
    {
        text = File.ReadAllText(text);
    }

    Console.WriteLine($"Sending request to Amazon Polly to synthesize speech from text: {text}");

    var isSSML = text.StartsWith("<speak>");

    var request = new SynthesizeSpeechRequest
    {
        LanguageCode = languageCode,
        VoiceId = (VoiceId)voiceId,
        Text = text,
        OutputFormat = OutputFormat.Mp3,
        LexiconNames = (args.Length > 3) ? new List<string>(args[4].Split(',')) : null,
        TextType = isSSML ? TextType.Ssml : TextType.Text,
        Engine = isSSML ? Engine.Standard : Engine.Neural
    };

    var response = await pollyClient.SynthesizeSpeechAsync(request);

    if (response.HttpStatusCode == System.Net.HttpStatusCode.OK)
    {
        Console.WriteLine($"Synthesized successfully");

        var mp3Filename = (@"polly-tts.mp3");

        using (var fileStream = File.Create(mp3Filename))
        {
            response.AudioStream.CopyTo(fileStream);
            fileStream.Flush();
            fileStream.Close();
        }

        Console.WriteLine($"Audio stream saved to {mp3Filename}");
    }
    else
    {
        Console.WriteLine($"Error: SynthesizeSpeechAsync returned HTTP status code {response.HttpStatusCode}");
    }
}
else
{
    Console.WriteLine("Usage: dotnet run -- languageCode voiceCode text|file [lexicon-names]");
    Console.WriteLine("Ex:    dotnet run -- en-US Mike \"Hello there. My name is Mike.\"");
    Console.WriteLine("Ex:    dotnet run -- en-US Amy \"data\\article.txt\"");
}
```

### Understanding the Code

Our program expects command line parameters for a language code (such as "en-US"), a voice Id (such as "Amy"), and text (such as "hello there!", in quotes). If the text parameter contains a slash or backslash, it is presumed to be a filename and the program loads text from that file. An optional fourth command line argument can specify one or more lexicon names (comma-separated).

```dos
dotnet run -- languageCode voiceCode text|file [lexicon-names]
```
The code is pretty short. After getting the command line arguments `languageCode`, `voiceId`, and `text`, we follow the standard AWS SDK for .NET patterns, first by creating an AmazonPollyClient that we pass our region.

To invoke Polly and perform the speech synthesis, we call the Polly client's `SynthesizeSpeechAsync` method. For that, we need to create a `SynthesizeSpeechRequest`. The request includes our 3 command line parameters—language code, voice Id (converted to a VoiceId enum), and text—plus several others. `LexiconNames` is for specifying PLS names to use. Our program allows for an optional fourth command line argument to specify a PLS name. `TextType` is set to text or SSML determined by whether the text to synthesize begins with a `<speak>` tag or not. We specify the standard speech engine when we have SSML, otherwise the neural engine.

The response contains an `HttpStatusCode`. If it's 200 OK, we save the audio stream to a file. This is done by creating our output MP3 file with a filestream, then using the AudioStream's convenient CopyTo method to copy to the filestream. Finally, the filestream is flushed and closed, and we're finished.

## Step 5: Run the Program and Perform TTS

In this step, you'll run the program you just created and test Amazon Polly text-to-speech.

1. Save your changes from the previous step and build the program.

2. In a command/terminal window, CD to the project folder.

3. Try a simple TTS of some command line text. 

    A. Enter a dotnet run command like the one below.

    ```dos
dotnet run -- en-US Amy "This is a test of Amazon Polly."
```
    B. You should get console output indicating successful speech synthesis and confirming an output file was created.

    ![04-run-amy-test.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1644703525983/JQzds4Z5O.png)

    [▶ listen](https://david-pallmann-blog-audio.s3.us-west-2.amazonaws.com/hellopolly/dotnet-run-amy.mp3)

    C. Find the output file, `polly-tts.mp3', and open it to listen to the MP3 file. It should be the spoken equivalent of your text in the language and voice ID you specified on the command line.

4. Try some variations of voice and text. Experiment!

    ```dos
dotnet run -- en-US Joey "Madame, your table is ready. Right this way, please."
```

    ![04-run-joey-test.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1644788948003/0EiHO704z.png)

    [▶ listen](https://david-pallmann-blog-audio.s3.us-west-2.amazonaws.com/hellopolly/dotnet-run-joey-tts.mp3)

5. Now, let's put the text to be spoken into a text file.

    A. Create a text file in another folder. 

    B. Change your command line to specify your language, a different voice, and the path to your filename as the third parameter. The example below specifies language en-US, voice Matthew, and file data\sentry.txt—a text file containing a public domain short story, Sentry by Fredrik Brown. source: [Project Gutenberg](https://www.gutenberg.org/cache/epub/29948/pg29948.txt)

   ```
dotnet run -- en-US Matthew data\sentry.txt
```
    B. Enter and run your command. We see the text is loaded from the filename and displayed on the console.

    ![04-run-file-sentry.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1644788064878/P_b6m8wgh.png)

    C. Listen to the saved audio file, polly-tts.mp3, to see how good a job Polly does. 

    [▶ listen](https://david-pallmann-blog-audio.s3.us-west-2.amazonaws.com/hellopolly/sentry-tts.mp3) to audio of short story Sentry by Fredrik Brown. 

Congratulations, you've performed Text-to-Speech from .NET code using Amazon Polly! Talk about impressive!

# Where to Go From Here

Spoken content can add a whole new dimension of experience for your users, and Amazon Polly makes it easy to obtain lifelike text-to-speech. Talk is cheap!

In this tutorial, you learned how to use Amazon Polly for text-to-speech, first from the AWS console and then in .NET code. You wrote a console program that uses the AWS SDK for .NET to convert text to speech using the SynthesizeSpeechAsync method and saved the audio to a local MP3 file. You worked with plain text as well as SSML.

The tutorial did not cover how to process large amounts of text. If you need to process text 3k-100k characters in length, explore the [StartSpeechSynthesisTask](https://docs.aws.amazon.com/polly/latest/dg/API_StartSpeechSynthesisTask.html) method which will run an asynchronous task and store the output in an S3 bucket.

We only provided a glimpse of SSML without delving into its specifics. Read the documentation and try out the various features in the AWS console.

# Further Reading

AWS Documentation

[Amazon Polly](https://aws.amazon.com/polly/)

[Amazon Polly Developer Guide](https://docs.aws.amazon.com/polly/latest/dg/polly-dg.pdf)

[.NET Immersion Day tutorial: Convert text into lifelike speech with Amazon Polly](https://catalog.us-east-1.prod.workshops.aws/v2/workshops/02696107-09ac-4313-a6cb-3798048b07d7/en-US/4-adding-innovation-ai-ml/polly)

[Languages supported by Amazon Polly](https://docs.aws.amazon.com/polly/latest/dg/SupportedLanguage.html)

[Managing Lexicons Using the Amazon Polly Console](https://docs.aws.amazon.com/polly/latest/dg/managing-lexicons-console.html#managing-lexicons-console-synthesize-speech)

[Using the PutLexicon Operation](https://docs.aws.amazon.com/polly/latest/dg/gs-put-lexicon.html)

[Using SSML in the Console](https://docs.aws.amazon.com/polly/latest/dg/ssml-to-speech-console.html)

[Supported SSML Tags](https://docs.aws.amazon.com/polly/latest/dg/supportedtags.html)

[AWS SDK for .NET Documentation](https://docs.aws.amazon.com/sdk-for-net/index.html)

Videos

[Using Amazon Translate, Amazon Polly, and Amazon Rekognition with .NET](https://www.youtube.com/watch?v=UO0hGTvqLUs) by Thorr Giddings

[How to get the most out of Polly: Leveraging lexicons and SSML](https://www.youtube.com/watch?v=B2XSU22ilmQ) by Marco Nicolis and Remus Mois

[Bringing Characters to Life with Amazon Polly Text to Speech](https://www.youtube.com/watch?v=4gsx7EeMxWs) by Robin Dautricort and Felix Duchesneau

Blogs

[A Deep Dive into Amazon Polly](https://labrlearning.medium.com/a-deep-dive-into-amazon-polly-3672baf6c624) by Chris Hare

[Using Amazon Polly from .net / c#, Get MP3 File](https://chrisbitting.com/2017/04/07/using-amazon-polly-from-net-c-get-mp3-file/) by Chris Bitting

[Hello, Cloud blog series](https://davidpallmann.hashnode.dev/series/hello-cloud)
