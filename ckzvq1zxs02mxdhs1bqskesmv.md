## Hello, Transcribe!

#### This episode: Amazon Transcribe and Speech-to-Text. In this Hello, Cloud blog series, we're covering the basics of AWS cloud services for newcomers who are .NET developers. If you love C# but are new to AWS, or to this particular service, this should give you a jumpstart.

In this post we'll introduce Amazon Transcribe and use it in a "Hello, Cloud" .NET program to perform Speech-to-Text. We'll do this step-by-step, making no assumptions other than familiarity with C# and Visual Studio. We're using Visual Studio 2022 and .NET 6.

# Amazon Transcribe : What is it, and why use It?

More and more, we deal with audio and video data—which isn't readily searched or analyzed. Converting speech to text enables many uses in applications. You could gain insights from customer conversations. You could create transcripts. You could create searchable archives of A/V assets. You could augment recorded or live A/V content with captions or notes. 

[Amazon Transcribe ](https://aws.amazon.com/transcribe) (hereafter "Transcribe") is a service that automatically converts speech to text (STT). AWS describes it as "an automatic speech recognition service that makes it easy to add speech to text capabilities to any application". We've previously covered the [Amazon Polly](https://davidpallmann.hashnode.dev/hello-polly) Text-to-Speech service. Amazon Transcribe is its counterpart, converting speech to text. 

Transcribe can work with recorded or live audio/video for input. Batch transcription jobs use S3 to house recorded media (input files) and generated transcript (output files).

![diagram-combined.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1645385897657/1j1kx9FKW.png)

 [Dozens of languages](https://docs.aws.amazon.com/transcribe/latest/dg/supported-languages.html#table-language-matrix) are supported. You can tell Transcribe which language(s) you believe are present in a media file, but Transcribe can also identify a media file's language automatically.

What kind of media does Transcribe accept? The list differs depending on whether you are using batch jobs for recorded media transcription or are streaming transcriptions. See [Amazon Transcribe Data Input and Output](https://docs.aws.amazon.com/transcribe/latest/dg/how-input.html) for details.

| Mode | Supported Media Formats |
| ------ | ------------ |
| Media file transcription jobs | FLAC, MP3, MP4, Ogg, WebM, AMR, and WAV |
| Streaming transcriptions | FLAC, OPUS-encoded audio in an Ogg container, and PCM 16-bit signed little endian audio formats |

Transcribe uses a deep learning process called automatic speech recognition (ASR) to convert speech to text quickly and accurately. Transcribe has separate APIs for two specialized use cases, customer calls ([Amazon Transcribe Call Analytics](https://aws.amazon.com/transcribe/call-analytics/)) and medical conversations ([Amazon Transcribe Medical](https://aws.amazon.com/transcribe/medical/)). Today we'll be focused on its general API.

Transcribe offers a number of [advanced features](https://docs.aws.amazon.com/transcribe/latest/dg/transcribe-whatis.html). You can remove content, including personally identifiable information (PII) and profane, offensive, or unsuitable words. You can teach it new words with custom vocabularies. It can process multiple channels of audio, such as both sides of a phone conversation. A feature called speaker diarization can discern between multiple speakers and give them attribution in the transcript. It can generate subtitles for video in WebVTT (.vtt) and SubRip (.srt) formats. Transcription job results include a text transcript as well as metadata—including timestamps for each word, which is helpful for adding subtitles to video.

The AWS Free Tier gives you 60 minutes free usage of Amazon Transcribe per month for the first 12 months.

# Our Hello, Transcribe Project

We will first get familiar with Amazon Transcribe in the AWS console, then we'll write a .NET program that transcribes audio and video files. We'll do that by uploading audio/video to S3, starting a batch Transcribe job, waiting for it to finish, and retrieving the transcript output from S3.

[▶ Listen](https://commons.wikimedia.org/wiki/File:Winston_Churchill_-_Be_Ye_Men_of_Valour.ogg): Winston Churchill - Be Ye Men of Valour.ogg
(source: Wikimedia Commons)

![05-dotnet-run-churchill.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1645299261292/fjJmWHvNz.png)

![05-churchill-json.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1645383515673/JoPy2c67Z.png)

[source code](https://github.com/davidpallmann/hello-transcribe)

## One-time Setup

For any of the tutorials in the Hello, Cloud series you need the following:

1. An  [AWS account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/), and an understanding of what is included in the  [AWS Free Tier](https://aws.amazon.com/premiumsupport/knowledge-center/what-is-free-tier/). 
2. [Microsoft Visual Studio 2022](https://visualstudio.microsoft.com/). If you're using an older version of Visual Studio you won't be able to use .NET 6. If you use a different IDE, you may have to find alternatives to some tutorial steps.
3. [AWS Toolkit for Visual Studio](https://aws.amazon.com/visualstudio/). You'll need to [configure the toolkit ](https://docs.aws.amazon.com/toolkit-for-visual-studio/latest/user-guide/keys-profiles-credentials.html) to access your AWS account and create an IAM user. Your default AWS profile will be linked to this user when running programs from the command line.

## Step 1: Create an S3 Bucket for Audio and Video

In this step, you'll create an S3 bucket. We'll use the same bucket for our recorded media input files and for the output transcript file Transcribe will generate.

1. Sign in to the AWS console. At top right, select the region you want to work in. You can check supported regions for Transcribe and pricing on the [pricing page](https://aws.amazon.com/transcribe/pricing/). I'm using **us-west-2 (Oregon)**.

2. Navigate to **Amazon S3**. You can enter **s3** in the search bar.

3. Click **Create bucket**. Create a bucket to hold audio and video for Transcribe. Enter a bucket name and click **Create bucket** to create the bucket. Close the confirmation dialog. We're using the name **hello-transcribe**. If that name is in use, use a variation.

    ![01-s3-create-bucket.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1645384665715/ELRK9jJcD.png)

4. Next, find (or create) some audio and video files that you want to give Transcript as input that you have the rights to. Make sure they're in a supported format for batch jobs. If you're on a Windows PC, one easy way to create some audio files is by launching Voice Recorder. 

5. Select your S3 bucket and click **Upload**. Select and upload your audio and video files and close the confirmation dialog. Now we have some input data for Transcribe. I'm working with 4 media files: 1) a simple recording of me saying a sentence or two in Windows voice recorder, 2) a video I had recorded for my wife's birthday by an actor, 3) a video of a meeting, and 4) an audio file of a speech. 

    ![01-s3-media-files.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1645384544141/U4PtH2YYY.png)

## Step 2: Use Amazon Transcribe in the AWS Console

In this step, you'll get familiar with Transcribe basics in the AWS console.

1. In the AWS console, navigate to **Amazon Transcribe**. You can enter **transcribe** in the search box.

2. On the left panel, select **Transcription jobs**.

3. Click **Create job**.

4. Fill out the Specify job details page as follows:

    a. For Name, enter **test1**.

    b. For Input data, click the **Browse S3** button and select the bucket you created in Step 1 that has your media files. 

    c. In the **Choose resources** dialog, select a media file and click **Choose**. I'm using my simplest example, an .m4a file created with Windows 10 Voice Recorder of me saying "Amazon Transcribe is music to my ears. Well, speech anyway." My input file location on S3 is s3://hello-transcribe/music-to-my-ears.m4a.

    d. Under **Output data**, select **Customer specified S3 bucket**. For Output file destination on S3, copy the S3 input file location from the Input data pane, and replace the media file name with "test1.json". My output file destination on S3 is s3://hello-transcribe/test1.json.

    ![02-job-test1s.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1645385212447/i2eNdRE0q.png)

    e. Click **Next**. 

    f. Notice but do not change the audio settings and content removal options. Then click **Create job**.

    ![02-job-test1-2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1645279543421/5yAJzMlMW.png)

5. You should now see a confirmation that a transcription job has been created and be returned to the job list with a job shown.

    ![02-job-test1-3.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1645279663727/9kME8BD83.png)

6. Wait until the job status changes from In-progress to Complete.

7. Click the job name **test1** to see its detail. Here you see details about your job, including the input and output S3 objects.

    ![02-job-test1-4.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1645385415456/uKQj11XPz.png)

8. Click on the Output data  location to view the S3 bucket. You should now see a new file, test1.json, that was generated by the Transcribe job.

9. Click on **test1.json** and then click the **Open** or **Downoad** button to view it. We see that the output JSON contains a property named `transcript` with our transcript, as well as metadata.

    ```json
{"jobName":"test1","accountId":"393840302653","results":{"transcripts":[{"transcript":"Amazon transcribe It's Music to my ears Well, speech Anyway!"}],"items":[{"start_time":"0.65","end_time":"1.14","alternatives":[{"confidence":"1.0","content":"Amazon"}],"type":"pronunciation"},{"start_time":"1.14","end_time":"1.82","alternatives":[{"confidence":"0.7103","content":"transcribe"}],"type":"pronunciation"},{"start_time":"1.82","end_time":"1.97","alternatives":[{"confidence":"0.6941","content":"It's"}],"type":"pronunciation"},{"start_time":"1.97","end_time":"2.3","alternatives":[{"confidence":"1.0","content":"Music"}],"type":"pronunciation"},{"start_time":"2.3","end_time":"2.4","alternatives":[{"confidence":"1.0","content":"to"}],"type":"pronunciation"},{"start_time":"2.4","end_time":"2.52","alternatives":[{"confidence":"1.0","content":"my"}],"type":"pronunciation"},{"start_time":"2.52","end_time":"3.08","alternatives":[{"confidence":"1.0","content":"ears"}],"type":"pronunciation"},{"start_time":"3.39","end_time":"3.68","alternatives":[{"confidence":"0.6589","content":"Well"}],"type":"pronunciation"},{"alternatives":[{"confidence":"0.0","content":","}],"type":"punctuation"},{"start_time":"3.73","end_time":"4.17","alternatives":[{"confidence":"1.0","content":"speech"}],"type":"pronunciation"},{"start_time":"4.17","end_time":"4.56","alternatives":[{"confidence":"0.9991","content":"Anyway"}],"type":"pronunciation"},{"alternatives":[{"confidence":"0.0","content":"!"}],"type":"punctuation"}]},"status":"COMPLETED"}
```
Review the text against your media file to judge the accuracy of the transcription. My transcript was close but not 100% accurate: "Amazon transcribe It's Music to my ears Well, speech Anyway!". That word "it's" should have been "is". Then again, my quickly recorded, low voice audio was not a great quality input.

10. Now it's time to do it again with a different media file. Repeat steps 1-9 with a different media file and job name **test2**. Then go on to test your other files.

    My second job, test2, was a much larger media file, an 8.9MB video recorded for my wife's birthday that I purchased on [`memmo.me`](https://memmo.me/global/en). Let me pause to give a review of `memmo.me`. My wife and I are both fans of the Seinfeld TV show. I discovered I could order a birthday recording for her from an actor or celebrity with the `memmo.me` service. I was able to buy a custom birthday greeting from Larry Thomas, the actor from the infamous Seinfeld "Soup Nazi" episode. We were very happy with it.

    ![03-video-memmo.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1645280760657/khzuIOeXg.png)

    How did Amazon Transcribe do on this video? This time, the transcript was quite good. Transcribe didn't know what to do with "Schmoopie", which came out as "SMU p", but that's understandable since it's a made up term of affection. It also didn't get "avgolemono" soup.

    "Becky. I have a birthday greeting for you from your husband, David and myself. Of course. Happy birthday. Now I hear your hobbies are cooking and quilting. I hope you don't do those things simultaneously. It would be a shame for someone to fish out a quilt square or some stuffing out of their soup. But I hear you make a great afg Alemanno. Are you of greek origin or just one of the soups you like to make? And you make a great chili and you win chili cook offs? Well, I don't enter those because it would be too easy for me to win. And don't think your soup is better than mine because that would be pushing your luck little lady. That would be the same thing as me catching you and David kissing on my line, calling each other SMU p nobody kisses in my line. Then it would be bread. $3. No soup for you, come back one year next. Not something you want. You don't want to have to cook your own soup on your own birthday. So behave yourself, stay within the guidelines and get your soup. Take it home, have yourself a very happy birthday and Adios Muchacha"

11. When you're done experimenting in the console, delete the objects in your S3 bucket so you don't accrue ongoing charges.

## Step 3. Write a .NET Program for Speech-to-Text

Now that we've seen what Transcribe can do, it's time to work with it programmatically. In this step you'll write a .NET 6 console program that transcribes media file using S3 and Transcribe.

1. Open a command/terminal window and CD to a development folder.

2. Enter the dotnet new command below to create a console program named **hello-transcribe**.

    ```dos
dotnet new console -n hello-transcribe
```

3. Open the hello-transcribe project in Visual Studio.

4. In Solution Explorer, right-click the hello-transcribe project and select **Manage NuGet Packages...**. Find and install the **AWSSDK.TranscribeService** and **AWSSDK.S3** packages.

    ![03-nuget.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1645284778555/3HSoV2DFe.png)

5. Open Program.cs in the editor and replace with the code at the end of this step. 

    a. Set `_region` to the region you have been working with in the AWS console.

    b. Set `_bucketName` to the name of the bucket you created in Step 1.

6. In Solution Explorer, right-click the hello-transcribe project and add a class named **TranscribeHelper.cs**. 

7. Open TranscribeHelper.cs in the editor and replace it with the code at the end of this step.

8. Save your changes and ensure you can build the program.

Program.cs

```csharp
using Amazon;
using Amazon.TranscribeService;

namespace hello_transcribe
{
    public class Program
    {
        private static RegionEndpoint _region = RegionEndpoint.USWest2;
        private static string _bucketName = "hello-transcribe1";

        public static async Task Main(string[] args)
        {
            if (args.Length < 1)
            {
                Console.WriteLine("Usage:   dotnet run -- languageCode http-uri-s3-media-file");
                Console.WriteLine("Example: dotnet run -- en-US https://hello-transcribe1.s3.us-west-2.amazonaws.com/winstonchurchillarmyourselves.mp3");
            }

            var languageCode = args[0];
            var filePath = args[1];

            var transcribeHelper = new TranscribeHelper(_region, _bucketName);
            await transcribeHelper.TranscribeMediaFile(filePath, languageCode);
        }
    }
}
```

TranscribeHelper.cs

```csharp
using Amazon;
using Amazon.S3;
using Amazon.S3.Model;
using Amazon.TranscribeService;
using Amazon.TranscribeService.Model;

namespace hello_transcribe
{ 
    public class TranscribeHelper
    {
        private RegionEndpoint _region = RegionEndpoint.USWest2;
        private AmazonTranscribeServiceClient _transcribeClient { get; set; } = null!;
        private AmazonS3Client _s3Client { get; set; } = null!;

        private string _bucketName;

        /// <summary>
        /// Constructor. Instantiates Amazon S3 client and Amazon Transcribe client.
        /// </summary>
        /// <param name="region">AWS region</param>
        /// <param name="bucketName">S3 bucket name for input media files and output transcripts</param>

        public TranscribeHelper(RegionEndpoint region, string bucketName)
        {
            _region = region;
            _bucketName = bucketName;
            _s3Client = new AmazonS3Client(_region);
            _transcribeClient = new AmazonTranscribeServiceClient(_region);
        }

        /// <summary>
        /// Transcribe a media file. Uploads local file to S3, transcribes with Amazon S3, and retrieves results to a local transcript file.
        /// </summary>
        /// <param name="filePath">Local file path</param>
        /// <param name="languageCode">Language code, such as en-US or en-GB</param>
        /// <returns></returns>

        public async Task TranscribeMediaFile(string filePath, string languageCode)
        {
            var mediaFileName = Path.GetFileName(filePath);

            // upload local file to S3 and get HTTP Uri
            var s3HttpUri = await UploadFileToBucket(filePath, _bucketName);
            
            // set output transcript file to same name as media file, but with an extension of .json
            var pos = mediaFileName.LastIndexOf(".");
            var transcriptFileName = (pos != -1) ? mediaFileName.Substring(0, pos) + ".json" : mediaFileName + ".json";

            // Start job

            var startJobRequest = new StartTranscriptionJobRequest()
            {
                Media = new Media()
                {
                    MediaFileUri = s3HttpUri
                },
                OutputBucketName = _bucketName,
                OutputKey = transcriptFileName,
                TranscriptionJobName = $"{DateTime.Now.Ticks}-{mediaFileName}",
                LanguageCode = new LanguageCode(languageCode),
            };

            Console.WriteLine($"Creating transcription job\n    S3 bucket: {_bucketName}\n    input media file: {mediaFileName}\n    language code: {languageCode}\n    output transcript: {transcriptFileName}");

            var startJobResponse = await _transcribeClient.StartTranscriptionJobAsync(startJobRequest);
            Console.WriteLine($"Job {startJobResponse.TranscriptionJob.TranscriptionJobName} created, status {startJobResponse.TranscriptionJob.TranscriptionJobStatus.Value}");

            var getJobRequest = new GetTranscriptionJobRequest() { TranscriptionJobName = startJobRequest.TranscriptionJobName };

            // Wait for job completion

            Console.WriteLine("Awaiting job completion");
            GetTranscriptionJobResponse getJobResponse;
            do
            {
                Thread.Sleep(15 * 1000);
                Console.Write(".");
                getJobResponse = await _transcribeClient.GetTranscriptionJobAsync(getJobRequest);
            } while (getJobResponse.TranscriptionJob.TranscriptionJobStatus == "IN_PROGRESS");
            Console.WriteLine($"Job complete, status: {getJobResponse.TranscriptionJob.TranscriptionJobStatus}");

            // Save transcription file locally
            await SaveS3ObjectAsFile(_bucketName, transcriptFileName, transcriptFileName);

            // Delete media file and output file from S3 to avoid accruing charges.
            await DeleteObjectFromBucket(mediaFileName, _bucketName);
            await DeleteObjectFromBucket(transcriptFileName, _bucketName);

            Console.WriteLine($"Results saved to file {transcriptFileName}");

        }

        /// <summary>
        /// Upload local file to S3 bucket.
        /// </summary>
        /// <param name="filePath">local file path</param>
        /// <param name="bucketName">bucket name</param>
        /// <returns>S3 object Http Uri.</returns>
        private async Task<string> UploadFileToBucket(string filePath, string bucketName)
        {
            Console.WriteLine($"Uploading {filePath} to bucket {bucketName}");
            var putRequest = new PutObjectRequest
            {
                BucketName = bucketName,
                FilePath = filePath,
                Key = Path.GetFileName(filePath)
            };

            var putResponse = await _s3Client.PutObjectAsync(putRequest);

            if (putResponse.HttpStatusCode != System.Net.HttpStatusCode.OK)
            {
                throw new ApplicationException("Media file upload to S3 failed");
            }

            var httpUri = $"https://{bucketName}.s3.amazonaws.com/{putRequest.Key}";
            Console.WriteLine($"    S3 object HttpUri: {httpUri}");
            return httpUri;
        }

        private async Task SaveS3ObjectAsFile(string bucketName, string key, string filePath)
        {
            using (var obj = await _s3Client.GetObjectAsync(bucketName, key))
            {
                await obj.WriteResponseStreamToFileAsync(filePath, false, new CancellationToken());
            }
        }

        /// <summary>
        /// Delete file from S3 bucket.
        /// </summary>
        /// <param name="filename"></param>
        private async Task DeleteObjectFromBucket(string filename, string bucketName)
        {
            Console.WriteLine($"Delete {filename} from bucket {bucketName}");
            var deleteRequest = new DeleteObjectRequest
            {
                BucketName = bucketName,
                Key = Path.GetFileName(filename)
            };
            await _s3Client.DeleteObjectAsync(deleteRequest);
        }
    }
}
```

### Understand the Code

The code is small and easy to understand. Program.cs instantiates the TranscribeHelper class and calls its TranscribeMediaFile method, which does all the work. 

TranscribeHelper instantiates an `AmazonS3Client` client for S3 operations and an `AmazonTranscribeServiceClient` for starting and managing transcription jobs. In TranscribeMediaFile, the local media file is uploaded to the S3 bucket. The class method UploadFIleToBucket handles the S3 upload, passing back an HTTP Uri for the S3 object, which is needed for the Transcribe API. 

The Transcribe job is kicked off with the Transcribe client's StartTranscriptionJobAsync method, which requires a StartTranscriptionJobRequest request object and returns a StartTranscriptionJobResponse object. 

To await completion of the job, a loop sleeps and checks the job status using GetTranscriptionJobAsync. Once the job is no longer in an IN_PROGRESS state, it is either completed or has failed. The output file is retrieved from S3 and saved to a local file with the class method SaveS3ObjectAsFile. Finally, the input and output files are deleted from the S3 bucket, since they're no longer needed.

## Step 4: Run the Program and Test Convert Speech to Text

Now it's time to run our .NET code and convert speech to text. 

1. In the AWS console, select a local media file you'd like to try, one that you have permission/rights to use. 

2. Open a command/terminal window and CD to the project location.

3. Run the `dotnet run` command with a language code (such as en-US) and the file path.

    ```dos
dotnet run -- language-code media-file
```
    The console output will give you a blow-by-blow account as the media file is uploaded to S3, a Transcribe job is started, the job is awaited for completion, the transcript output file is retrieved and saved to a local file, and the S3 files are deleted.

    ![05-dotnet-run-music.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1645387041718/zPxtq5PzC.png)

5. View the JSON output and the transcript text. How good a job did Transcribe do on your media file? Try some others.

4. Optional: try a public domain audio recording

    To test with Winston Churchill's [Be Ye Men of Valor speech](https://commons.wikimedia.org/wiki/File:Winston_Churchill_-_Be_Ye_Men_of_Valour.ogg) speech (source:Wikimedia Commons), first download the winstonchurchillarmyourselves.mp3 file to a local file. This is a 10 minute speech in a 7k MP3 file. 

    [▶ Listen](https://commons.wikimedia.org/wiki/File:Winston_Churchill_-_Be_Ye_Men_of_Valour.ogg): Winston Churchill - Be Ye Men of Valour.ogg
(source: Wikimedia Commons)

    Use a command line like the one below, specifying the language code for English-Great Britain (en-GB) and the path of the local file.

    ```dos
dotnet run -- en-GB winstonchurchillarmyourselves.mp3```

    ![05-dotnet-run-churchill.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1645291352628/-0bM4u_Z1.png)

    The program uploads the media file to S3 and kicks off an Amazon Transcribe job. We want to wait for completion, so we enter a loop that keeps checking the job status and sleeping for 15 seconds, until the job is no longer in an IN_PROGRESS state. Finally, the transcript output is retrieved from S3 and saved to a local file. The media file and output file are deleted from S3 to prevent accruing charges.

    The output JSON file includes a text transcript. Viewing the file in a JSON editor, we can see the transcript came out well:

    ![05-churchill-json.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1645387205280/Dhc8lzyPj.png)

    `I speak you for the first time as prime minister in a solemn hour for the life of our country, of our empire, of our allies and above all, of the cause of freedom. Mhm! A tremendous battle is raging in France and Flanders. The Germans, by a remarkable combination of air bombing and heavily armoured tanks, have broken through the French defences north of the Maginot Line. And strong columns of their armoured vehicles are ravaging the open country, which for the first day or two, was without defenders. They have penetrated deeply and spread alarm and confusion in their track behind them. There are now appearing infantry in Lorries and behind them again, the large messages are moving forward. The regroupment of the French armies to make head again and also to strike at this intruding wedge has been proceeding for several days, largely assisted by the magnificent efforts of the Royal Air Force. We must not allow ourselves to be intimidated by the presence of these armoured vehicles in unexpected places behind our lives. If they are behind our front, the French are also at many points, fighting actively behind there. Both sides are therefore in extremely dangerous position. And if the French army and our own army are well handled, and I believe they will be if the French retained that genius for recovery and counter attack, which they have so long been famous. And if the British Army shows the dogged endurance and solid fighting power of which there have been so many examples in the past, then a sudden transformation of the scene you might spring into being. It would be foolish, however, to disguise the gravity of the hour. It would be still more foolish to lose heart and courage, or to suppose that well trained, well equipped armies numbering three or four millions of men can be overcome in the space of a few weeks or even months by a scoop or raid of mechanised vehicles. However formidable, we may look with confidence to the stabilisation of the front in France and to the general engagement of the message which will enable the qualities of the French and British soldiers to be matched squarely against those of their adversaries. For myself, I have invincible confidence in the French army and its leader. Only a very small part of that splendid army has yet been heavily engaged, and only a very small part of France has yet been invaded. There is good evidence to show that practically the whole of the specialist and mechanised force into the enemy have been already thrown into the battle, and we know that very heavy losses had been inflicted upon them. No officer or man, no brigade or division which grapples at close quarters with the enemy, wherever encountered, can fail to make a worthy contribution to the general result. The army's must cast away the idea of resisting attack behind concrete lines or natural obstacles, and must realise that mastery can only be regained by furious and unrelenting assault. And this spirit must not only animate the high command but must inspire every fighting man in the air. Often the Jerry Assad, often at odds, hitherto thought overwhelming. We have been flowing down three or 4 to 1 of our enemy, and the relative balance of the British and German air forces is now considerably more favourable to us than at the beginning of the battle. In cutting down the German bombers, we are fighting our own battle as well as that of France. My confidence in our ability to fight it out to the finish with the German air force has been strengthened by the fierce encounters which have taken place and are taking place at the same time. Our heavy bombers are striking nightly at the taproot of German mechanised power and have already inflicted serious damage happened. The oil refineries on which the Nazi effort to dominate the world directly depend. We must expect that as soon as stability is reached on the Western front, the bulk of that hideous apparatus of aggression, which dashed Holland into ruin and slavery in a few days will be turned upon us. I'm sure I speak for all when I say we are ready to face it, you enjoy, and to retaliate against it to any extent that the unwritten laws of war permitted, there will be many men and many women in this island. When the ordeal comes upon them, as come it will, we'll feel comfort and even a pride that they are sharing the perils of our lads at the front. Soldiers, sailors and airmen, God bless them and are drawing away for them apart, at least of the onslaught. They have to bear is not this the appointed time for all to make the utmost exertions in their power. If the battle is to be one, we must provide our men with ever increasing quantities of the weapons and ammunition they need. We must have and have quickly more aeroplanes, more tanks, more shells, more guns. There is imperious need for the vital munitions. They increase our strength against the powerfully armed enemy they replace the wastage of the obstinate struggle and the knowledge that wasted will speedily be replaced enables us to draw more readily upon our reserves and throw them in. Now that everything counts so much, our task is not only to win the battle, but to win the war. After this battle in France abates its force, there will come the battle for our island. For all the Britney's and all that Britain mean that will be the struggle in that supreme emergency. We shall not hesitate to take every step, even the most drastic to call forth from our people The last house and the last inch of effort. I wish they are capable. The interests of property the hours of labour are nothing compared to the struggle for life and honour for right and freedom to which we have vowed ourselves. I have received from the chiefs of the French Republic and in particular from its indomitable prime minister, Monsieur Reynaud, the most sacred pledges, that whatever happens, they will fight to the end, be it better or be glorious. Nay. If we fight to the end, it can only be glorious. Having received His Majesty's Commission, I have formed an administration of men and women of every party and of almost every point of view. We have deferred and quarrelled in the past that now one bond unites at all the wage war until victory is won and never to surrender ourselves to servitude and shame, whatever the cost and the agony. Maybe this is one of the most or striking periods in the long history of France and Britain. It is also beyond doubt the most sublime side by side unaided, except by their kith and kin in the great Dominions and by the wide empires which rest beneath their shield side by side. The British and French people have advanced to rescue not only Europe but mankind from the foulest and most soul destroying tyranny, which has ever darkened and stain the pages of history behind them. Behind us, high in the armies and fleets of Britain and France gather a group of shattered states and bludgeoned races the Czechs, the Poles, the Norwegians, the Danes, the Dutch, the Belgians upon all of whom the long night of barbarism will descend unbroken even by a star of hope. Unless we conquer as conquer, we must, as conquer we shall. Today is Trinity Sunday Centuries ago, words were written to be a call and a spur to the faithful servants of truth and justice. Arm yourselves and be men of Allah and be in readiness for the conflict. For it is better for us to perish in battle than to look upon the outrage of our nation and our altars as the will of God is in heaven. Even so, let him do`

5. Optional: try a public domain meeting video

    Let's try a much larger transcription task: a 51-minute Wikimedia Foundation meeting video [WMF Monthly Metrics Meeting January 9, 2014](https://commons.wikimedia.org/wiki/File:WMF_Monthly_Metrics_Meeting_January_9,_2014.ogv) (source: Wikimedia Commons). Download the .ogv file to a local file with a simple name like meeting.ogv. 

    Note: This is a 565MB video file, and if you run it you will be accruing more sizeable charges due to larger S3 storage and a longer Transcribe job run time. 

    ![04-wikimedia-meeting-video-page.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1645387559104/CIzvSrpq5.png)

    [▶ Listen](https://commons.wikimedia.org/wiki/File:WMF_Monthly_Metrics_Meeting_January_9,_2014.ogv): WMF Monthly Metrics Meeting January 9, 2014.ogv
(source: Wikimedia Commons)

    ```dos
dotnet run -- en-US meeting.ogv```

    ![05-dotnet-run-meeting.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1645387679178/OaqdBbhV-B.png)

    Examining the 179KB output file, we can see we have a high quality meeting transcript.

    ![job-meeting-json.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1645387789626/G2fhAZ4OV.png)

Congratulations! You've performed speech-to-text with Amazon Transcribe using the AWS SDK for .NET.

# Where to Go From Here

Amazon Transcribe, and its sister services Amazon Polly/Translate/Comprehend/Rekognition, can equip your applications with powerful capabilities for working with, converting, translating, and understanding text, speech, images, and video. 

In this tutorial, you learned how Amazon Transcribe works and used it in the AWS console to convert speech to text. Then, you wrote a .NET console program to transcribe media files. You used the AWS SDK to start Transcribe jobs, await their completion, and upload/download to and from S3. You saw that accuracy can vary. Factors such as audio quality, background noise, accents, or unexpected language switches can impact the accuracy of transcription.

This tutorial did not cover live content and working with streams, nor advanced features. To go further, clarify your speech-to-text use cases and get more familiar with Transcribe. Learn about custom vocabularies, PII removal, profanity filtering, multiple audio channels, speaker diarization, video subtitles, and transcription metadata. Test media for your use cases with Transcribe, assess the quality of transcription, and experiment with features. 

AWS is regularly improving Transcribe. You can opt out of your data being used to improve its training. The opt out process is explained in the [Service FAQ Page](https://aws.amazon.com/transcribe/faqs/).

# Further Reading

AWS Documentation

[Amazon Transcribe](https://aws.amazon.com/transcribe)

[Amazon Transcribe FAQs](https://aws.amazon.com/transcribe/faqs/)

[Amazon Transcribe Features](https://docs.aws.amazon.com/transcribe/latest/dg/transcribe-whatis.html)

[Amazon Transcribe Data Input and Output](https://docs.aws.amazon.com/transcribe/latest/dg/how-input.html)

[Getting Started with Amazon Transcribe](https://aws.amazon.com/transcribe/getting-started/?nc=sn&loc=4)

[Amazon Transcribe Developer Guide](https://docs.aws.amazon.com/transcribe/latest/dg/transcribe-whatis.html)

[Amazon Transcribe Supported Languages]((https://docs.aws.amazon.com/transcribe/latest/dg/supported-languages.html#table-language-matrix)

[Tutorial: Create an Audio Transcript](https://aws.amazon.com/getting-started/hands-on/create-audio-transcript-transcribe/)

[.NET Immersion Day tutorial: Automatic Speech Recognition by Amazon Transcribe](https://catalog.us-east-1.prod.workshops.aws/workshops/c36ccd6e-9145-4e97-b1b5-1069d6d68ed0/en-US/lab7-ai-ml/transcribe)

[AWS SDK for .NET Documentation](https://docs.aws.amazon.com/sdk-for-net/index.html)

Videos

[How to Transcribe Speech to Text with a Sample ASP.NET Core Application and Amazon Transcribe](https://www.youtube.com/watch?v=LSiyJG7htFs)

[AWS Transcribe Programatically using C#](https://www.youtube.com/watch?v=QraDQMZcMpU)

Blogs

[AWS Translate Speech to Text in C#](https://blog.dotnetframework.org/2021/03/04/aws-transcribe-speech-to-text-using-c/)

[Hello, Cloud blog series](https://davidpallmann.hashnode.dv/series/hello-cloud)