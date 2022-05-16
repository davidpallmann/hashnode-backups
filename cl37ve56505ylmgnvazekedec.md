## Hello, Kinesis Data Streams!

#### This episode: Amazon Kinesis and real-time data streaming. In this Hello, Cloud blog series, we're covering the basics of AWS cloud services for newcomers who are .NET developers. If you love C# but are new to AWS, or to this particular service, this should give you a jumpstart.

In this post we'll introduce Amazon Kinesis Data Streams and use it in a "Hello, Cloud" .NET program to produce and consume a data stream. We'll do this step-by-step, making no assumptions other than familiarity with C# and Visual Studio. We're using Visual Studio 2022 and .NET 6.

# Amazon Kinesis : What is it, and why use It?

> "No stream rises higher than its source." ―Frank Lloyd Wright

These days, businesses are more aware than ever that data is flowing all around them, from financial transactions to website activity to social media feeds, and they want to put it good use. The sooner you have this data, the higher the quality of decisions you can make. You can use real-time data combined with analytics for:
* awareness, such as real-time dashboards
* insights, such as anomaly detection or monitoring customer experience
* responding to changing conditions, such as dynamic pricing or supply chain management notifications

[Amazon Kinesis](https://aws.amazon.com/kinesis/) (hereafter "Kinesis") is a service that allows you to ingest, process, and analyze data streams in real time. AWS describes it this way: "Amazon Kinesis makes it easy to collect, process, and analyze real-time, streaming data so you can get timely insights and react quickly to new information." You can send a wide variety of data streams to Kinesis, including website activity, audio/video, IoT telemetry, and application logs. Data can be processed or analyzed as it arrives. 

Kinesis is not one service but several. There are 4 Kinesis services, each geared toward different use cases: Kinesis Data Streams, Kinesis Video Streams, Kinesis Data Firehose, and Kinesis Data Analytics. Our focus today is Kinesis Data Streams.

## Kinesis Data Streams

[Amazon Kinesis Data Streams](https://aws.amazon.com/kinesis/data-streams/), the focus of this post, is a massively scalable serverless streaming data service. You can send things like application logs, application events, web clickstream data, or sensor data to this service, which can ingest terabytes of data per day. Use cases include streaming log and event data, real-time analytics of high-frequency event data, and event-driven applications.

![diagram-kinesis-data-streams.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1652546475590/2E4RVR1n7.png align="left")

Kinesis data is held for a limited retention period. The default retention period is 24 hours. At greater cost, you can opt for a 7-day retention, or long term retention up to a maximum of 365 days.

## Concepts

Let's go over some key concepts about Data Streams. 
* A **data producer** is an application or service that emits data records to a Kinesis data stream as they are generated. 
* Records contain **partition keys**, which determine which shard ingests the record.
* A **shard** is an ordered sequence of records, ordered by arrival time. 
* A **data stream** is a logical grouping of shards.  
* A **data consumer** is an application or service that retrieves data from shards in a stream as it is generated. 

    ![diagram-producer-consumer.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1652652368442/ZdAjvdr_z.png align="left")

You can have multiple producers and consumers working with a stream.

## Managing Cost

Take note that Kinesis does not participate in the AWS Free Tier, so any experimentation will come at a cost. Don't forget to deallocate your data streams when you're finished with them. Check the [pricing page](https://aws.amazon.com/kinesis/data-streams/pricing/) for rates, and take advantage of the [pricing calculator](https://calculator.aws/#/createCalculator/KinesisDataStreams) to estimate your monthly or annual costs. You can save your estimates for future review.

![pricing-calculator.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1652548966451/o6XZqFy1b.png align="left")
 
## Programming Interfaces

Like most AWS services, you can use the AWS SDK for .NET to code a producer or consumer. However, there's another choice. You can use the [Kinesis Client Library for .NET](https://docs.aws.amazon.com/streams/latest/dev/kinesis-record-processor-implementation-app-dotnet.html). This is a wrapper around the Kinesis Client Library, a Java library, which will require you to install Java.

For this tutorial, we will use the AWS SDK for .NET. 

# Our Hello, Kinesis Data Streams Project

We will first create a Kinesis Data Stream in the AWS management console. Then, you'll create a .NET producer program and .NET consumer program.

![01-06-multple-producers-one-consumer.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1652650977242/DXsdfe7WQ.png align="left")

[source code](https://github.com/davidpallmann/hello-kinesis-data-streams)

## One-time Setup

For any of the tutorials in the Hello, Cloud series you need the following:

1. An  [AWS account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/), and an understanding of what is included in the  [AWS Free Tier](https://aws.amazon.com/premiumsupport/knowledge-center/what-is-free-tier/). 

2. [Microsoft Visual Studio 2022](https://visualstudio.microsoft.com/). If you're using an older version of Visual Studio you won't be able to use .NET 6. If you use a different IDE, you may have to find alternatives to some tutorial steps.

3. [AWS Toolkit for Visual Studio](https://aws.amazon.com/visualstudio/). You'll need to [configure the toolkit ](https://docs.aws.amazon.com/toolkit-for-visual-studio/latest/user-guide/keys-profiles-credentials.html) to access your AWS account and create an IAM user. Your default AWS profile will be linked to this user when running programs from the command line.

## Step 1: Create a Stream

In this step, you'll create a Kinesis Data Stream in the AWS management console.

1. Sign in to the AWS console. At top right, select the region you want to work in. You can check supported regions for Kinesis Data Streams on the [Amazon Kinesis Data Streams endpoints and quotas page](https://docs.aws.amazon.com/general/latest/gr/ak.html). I'm using **us-west-2 (Oregon)**.

2. Navigate to **Amazon Kinesis**. You can enter **kinesis** in the search bar.

3. On the left pane, select **Data streams** and click **Create data stream**. On the Create data stream page, enter/select the following:

    A. Data stream name: **hello-kinesis**.

    B. Capacity mode: **On-demand**.

    C. Click **Create data stream**.

    ![01-create-data-stream.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1652552492436/IkgTAA9n-.png align="left")

    D. Wait for confirmation that the stream has been created. From this page, you can see the Amazon Resource Name (ARN) for your stream, configuration details, and activity metrics.

    ![01-create-data-stream-created.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1652652789321/3rjE2EQl8.png align="left")

    If you navigate up a level (or select **Data streams** from the left pane), you'll see a view that shows you your stream and the number of provisioned shards. There currently are no shards, since we haven't sent any data to the stream yet.

    ![01-create-data-stream-no-shards-yet.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1652552939273/9jEbGLj9t.png align="left")

You've now created a Kinesis Data Stream that you can write to or read from, which will retain data records up to 24 hours.

## Step 2: Write a Data Producer program

In this step, you'll write a .NET program that acts as a data producer for the stream, sending it data records. We'll write this as a console program, where you can indicate on the command line what to send and how many repetitions.

1. Open a command/terminal window and CD to a development folder.

2. Run the `dotnet new` command below to create a new console program named `producer`.

    ```dos
dotnet new console -n producer
```

3. Launch Visual Studio and open the `producer` project.

4. Add the AWSSDK.Kinesis package:

    A. In Solution Explorer, right-click the product project and select **Manage NuGet packages**.

    B. Search for and install the **AWSSDK.Kinesis** package.

    ![02-nuget.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1652553525177/RVsdaA7sl.png align="left")

5. Open Program.cs in the editor and replace with the code below. 

    a. Set `RegionEndpoint` to the region endpoint you have been working with in the AWS console.

    b. Make sure `stream_name` matches the Kinesis stream name you created in Step 

Program.cs

```csharp
using Amazon;
using Amazon.Kinesis;
using Amazon.Kinesis.Model;
using System.Text;
using System.Text.Json;

const string stream_name = "hello-kinesis";

RegionEndpoint region = RegionEndpoint.USWest2;

string source = "producer1";
int count = 1;
int delay = 1;
string message = null!;

// Parse command line. Expected:
// dotnet run -- <message>
// dotnet run -- <producer-id> <count> <delay> <message>

if (args.Length==1)
{
    message = args[0];
}
else if (args.Length==4)
{
    source = args[0];
    count = Convert.ToInt32(args[1]);
    delay = Convert.ToInt32(args[2]);
    message = args[3];
}
else
{
    Console.WriteLine($"This command will send data records to Kinesis data stream {stream_name}");
    Console.WriteLine(@"To send a single message   dotnet run -- ""<message>""");
    Console.WriteLine(@"To send multiple messages: dotnet run -- <source-name> <count> <delay-in-seconds> ""<message>""");
    Environment.Exit(0);
}

var client = new AmazonKinesisClient(region);

// Create data record and serialize to UTF8-encoded bytes

var id = Guid.NewGuid();
var data = new { Id = id, Source = source, Message = message, Timestamp = DateTime.Now.ToString() };
byte[] dataBytes = Encoding.UTF8.GetBytes(JsonSerializer.Serialize(data));

// Create a memory stream from data record and put to the Kinesis stream

using (MemoryStream ms = new MemoryStream(dataBytes))
{
    var request = new PutRecordRequest()
    {
        StreamName = stream_name,
        PartitionKey = source,
        Data = ms
    };

    Console.WriteLine($"Writing as source {source} to Kinesis stream {stream_name}");

    for (int i = 0; i < count; i++)
    {
        var response = await client.PutRecordAsync(request);
        Console.WriteLine($"{data.Timestamp} sequence number {response.SequenceNumber}: shard Id: {response.ShardId}");
        Thread.Sleep(delay * 1000);
    }
}
```

### Understand the Producer Code

The producer code is very short.

1. Lines 16-37: The code processes the command line, expecting just one parameter (the message) or four (source name, count, delay, and message). If the short one-parameter version is used, producer defaults to "producer1", count defaults to 1, and delay defaults to 0.

2. 39: An `AmazonKinesisClient` is created, passing in the region.

3. 41-45: A data record is created, then serialized into an array of bytes.

4. 47-58: To send the record to the stream, a `PutRecordRequest` is created, with the stream name, a partition key (source), and the data, in the form of a memory stream created from the data bytes. 

5. 60-66: The SDK `PutMethodAsync` method is called to send the data record to the stream, in a loop to send the desired count of messages with the desired delay in between messages.

## Step 3: Test the Producer

In this step, you'll run the producer program you just created.

1. Open a command/terminal window and CD to the project location. We'll call this the *producer window*.

2. Run `dotnet run`. You will see help explaining the command line parameters.

    ![03-dotnet-run-empty.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1652555159848/7-qnssxHm.png align="left")

3. Next, try sending just one message. Enter the command below, which will send the message "test", defaulting to 1 message and source name "producer1".

    ```dos
dotnet run -- "test"
```

    You should see confirming output that you are writing to the hello-kinesis stream as source "producer1". `source` is a field we made up for our data record, but it's also what we're specifying as partition key. In the output, you can see the shard Id that was assigned for our partition key.

    ![01-01-producer-test.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1652644223216/Q4OyPfWVJ.png align="left")

4. Now, enter a command below that uses all of the command line parameters to send 4 messages, 1 second apart, again with source name "producer1".

    ```dos
dotnet run -- producer1 4 1 "test2"
```

    This time you see 4 data records sent. You can see the same shard name was computed, because we set it from the same value, "producer1".

    ![01-02-producer-test2-4.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1652644352849/hRzKkOzuI.png align="left")

5. Back in the AWS console, navigate to your hello-kinesis data stream and select the Monitoring tab. Select the shortest interval **1h**. After a minute or so, you should see evidence of activity in the PutRecord charts. This confirms your code has indeed been relaying data to your Kinesis data stream.

    ![03-monitor-put-record.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1652653190712/xEuvzwpOg.png align="left")

    ![03-monitor-put-record2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1652556242863/Y_hjmnZx1.png align="left")

## Step 4. Write a Data Consumer program

In this step, you'll write a second .NET program that will acts as a data consumer for the stream, retrieving data records. We'll write this as a console program that will echo stream records as they are received.

1. Open another command/terminal window and CD to a development folder.

2. Run the `dotnet new` command below to create a new console program named `consumer`.

    ```dos
dotnet new console -n consumer
```

    ![04-dotnet-new.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1652556974796/96P3HA0Mj.png align="left")

3. Launch Visual Studio and open the `consumer` project.

4. Add the AWSSDK.Kinesis package:

    A. In Solution Explorer, right-click the product project and select **Manage NuGet packages**.

    B. Search for and install the **AWSSDK.Kinesis** package.

5. Open Program.cs in the editor and replace with the code at the end of this step. 

    a. Set `RegionEndpoint` to the region endpoint you have been working with in the AWS console.

    b. Make sure `stream_name` matches the Kinesis stream name you created in Step 1.

6. Save your changes and ensure you can build the program.

Program.cs

```csharp
using Amazon;
using Amazon.Kinesis;
using Amazon.Kinesis.Model;
using System.Text;

class Program
{
    const string stream_name = "hello-kinesis";
    static RegionEndpoint region = RegionEndpoint.USWest2;

    static AmazonKinesisClient _client = null!;

    public static async Task Main(string[] args)
    {
        _client = new AmazonKinesisClient(region);

        // Describe stream to get list of shards

        var describeRequest = new DescribeStreamRequest()
        {
            StreamName = stream_name
        };
        var describeResponse = await _client.DescribeStreamAsync(describeRequest);
        List<Shard> shards = describeResponse.StreamDescription.Shards;

        Console.WriteLine($"Listing records for Kinesis stream {stream_name} - interrupt program to stop.");
        Console.WriteLine();

        // Spawn a thread for each shard

        var threads = new List<Thread>();
        foreach (var shard in shards)
        {
            var thread = new Thread(MonitorShard);
            thread.Start(shard.ShardId);
            threads.Add(thread);
            Console.WriteLine($"Started thread to monitor shard {shard.ShardId}");
        }

        Console.ReadLine();

    }

    // Monitor shard method (signature needed for Thread.Start)

    private static async void MonitorShard(object? shard) => await MonitorShard((string)shard!);

    // Monitor shard methid (implementation)

    private static async Task MonitorShard(string shard)
    {
        // Get iterator for shard

        var iteratorRequest = new GetShardIteratorRequest()
        {
            StreamName = stream_name,
            ShardId = shard,
            ShardIteratorType = ShardIteratorType.TRIM_HORIZON
        };

        // Retrieve and display records for shard

        var iteratorResponse = await _client.GetShardIteratorAsync(iteratorRequest);
        string iterator = iteratorResponse.ShardIterator;

        while (iterator != null)
        {
            // Get records from iterator

            var getRequest = new GetRecordsRequest()
            {
                Limit = 100,
                ShardIterator = iterator
            };

            var getResponse = await _client.GetRecordsAsync(getRequest);
            var records = getResponse.Records;

            // Display records

            if (records.Count > 0)
            {
                foreach (var record in records)
                {
                    var recordDisplay = Encoding.UTF8.GetString(record.Data.ToArray());
                    Console.WriteLine($"Record: {recordDisplay}, Partition Key: {record.PartitionKey}, Shard: {shard}");
                }
            }

            // Get next iterator

            iterator = getResponse.NextShardIterator;
            Thread.Sleep(100);
        }
    }
}
```

### Understand the Consumer Code

The consumer code is a bit more involved than the producer code.

1. Line 15: An `AmazonKinesisClient` is created, passing in the region.

2. 17-24: A `DescribeStreamRequest` SDK method describes the stream. The response includes a `StreamDescription.Shards` property, a collection of shard Ids.

3. 29-38: We want to monitor the entire stream. The way we'll do that is with a `MonitorShard` method that continually monitors a shard for messages. We create a thread for each shard that will run `MonitorShard`, passing the shard Id.

4. 44-36: This first `MonitorShard` method has the signature needed for creating the thread. It simply calls the implementation method.

5. 50-96: This is the actual `MonitorShard` method. 

    A. 61-64: We call the `GetShardIteratorAsync` SDK method to get a *shard iterator*. We set `iterator` to the response's `ShardIterator` property. This is an Id we will need to specify when fetching data records from the stream.

    B. 66-92: The while loop gets and displays records for the iterator. This loop will run perpetually.

    C. 68-77: To get records, we call the `GetRecordsAsync` SDK method, passing our iterator Id and a max record count. The response `Records` collection is the record data.

    D. 79-88: If the response contains any records, we loop through them. We deserialize each data record from the data bytes and display it. 

    E. 92-84: At the bottom of the while loop, we set `iterator` to the next shard iterator from the get records response, and do a sleep to avoid unnecessary non-stop I/O. The next iterator is never null, so the loop will run perpetually until the program is halted. 

## Step 5: Test the Data Consumer

In this step, you'll run the consumer program you just created.

1. Open a second command/terminal window (the *consumer window*) and CD to the project location.

2. Run `dotnet run'. 

3. The program confirms the Kinesis stream name, learns the available shards, and creates a thread to monitor each shard. We see our original "test" and four "test2" messages listed. In our test run, those messages were in shard `shardId-000000000003`.

    ![01-03-consumer-test-test2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1652644913640/I3A5VqTr2.png align="left")

4. Leave the consumer program running. Back in your original *producer window*, send more messages with this command:

    ```dos
dotnet run -- producer1 3 5 "test3"
```

    ![01-04-producer-test3-3.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1652645062461/oLCXw8Dfq.png align="left")

6. Back in the *consumer command window*, you see the test3 messages arrive.

    ![01-05-consumer-test3.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1652645123147/JQ4B_FmK_.png align="left")

7. Stop the consumer program.

8. Let's get several producers running in parallel. 

    A. In the *consumer window*, start the consumer running again with `dotnet run`. Notice all prior messages are again retrieved. In Kinesis, you can access the entire stream, until messages expire from the retention period.

    B. Open a second producer command window (*producer window #2*) and CD to the producer project folder. On producer window #2, run the command below to send more messages, this time with source `producer2`:

    ```dos
dotnet run -- producer2 10 5 "hello!"
```

    C. Back in *producer window #1*, run the command below to send more messages as `producer1`:

    ```dos
dotnet run -- producer1 5 10 "goodbye!"
```
    D. Watch the *consumer window*, and see messages from both producers are retrieved from the stream.

    ![01-06-multple-producers-one-consumer.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1652645930245/tk7u5VQ2o.png align="left")

8. Stop the consumer program.

Congratulations! You've both sent data records to a Kinesis Data Stream and retrieved them using .NET Code.

## Step 6: Shut it Down

Once you're done experimenting with this tutorial, delete the Kinesis Data stream. You don't want to be charged for something you're not using.

1. In the AWS console, navigate to Amazon Kinesis > Data streams.

2. Click the checkbox for the **hello-kinesis** stream.

3. At top right, select **Delete** from the Actions drop-down and confirm.

# Where to Go From Here

Data streams allow you to combine data from multiple sources into a time-series stream that one or more consumers can retrieve data from in real-time. In this tutorial, you created a Kinesis Data Stream, sent data records to it with a .NET producer program, and retrieved data records with a .NET consumer program.

We used the AWS SDK for .NET, but an alternative would have been to use the [Amazon Kinesis Client Library for .NET](https://github.com/awslabs/amazon-kinesis-client-net), which gives you instance load balancing, failure handling, and checkpointing of processed records. This library is a wrapper around a Java implementation.

Consider what analytics or processing needs to happen for your data records. For example, you might want to feed records to Kinesis Data Analytics or Lambda functions.

Since there isn't an AWS Free Tier for Kinesis, be sure to understand the pricing model and the two available capacity modes, on-demand and provisioned.

Once you're familiar with Kinesis Data Streams, learn about the other Kinesis service and consider whether using them in concern make sense for your use case.

# Further Reading

AWS Documentation

[Amazon Kinesis](https://aws.amazon.com/kinesis/)

[Amazon Kinesis Data Streams](https://aws.amazon.com/kinesis/data-streams/)

[Amazon Kinesis Data Streams Developer Guide](https://docs.aws.amazon.com/streams/latest/dev/introduction.html)

[AWS SDK for .NET Documentation](https://docs.aws.amazon.com/sdk-for-net/index.html)

Research

[Forrester Research: Analyze Data and Act on Insights the Instant Your Data is Born](https://www.youtube.com/watch?v=xD1upI6XB5g)

Libraries

[Kinesis Client Library for .NET - documentation](https://docs.aws.amazon.com/streams/latest/dev/kinesis-record-processor-implementation-app-dotnet.html)

[Kinesis Client Library for .NET - GitHub](https://github.com/awslabs/amazon-kinesis-client-net)

[KinesisProducerNet producer library](https://github.com/daletskyi/KinesisProducerNet)

Videos

[Amazon Kinesis Data Streams: Why Streaming Data?](https://aws.amazon.com/kinesis/data-streams/getting-started/?nc=sn&loc=4)

[Why Amazon Kinesis Data Streams?](https://aws.amazon.com/kinesis/data-streams/getting-started/?nc=sn&loc=4)

[Amazon Kinesis Data Streams Fundamentals](https://aws.amazon.com/kinesis/data-streams/getting-started/?nc=sn&loc=4)

[How to Support Streaming Data in your .NET Application with Amazon Kinesis](https://www.youtube.com/watch?v=gGb9vFRh2Ew) by Nitin Dhir

[How to create a .NET-based Lambda to Receive Events from an IoT Button and Send Data to Kinesis](https://www.youtube.com/watch?v=-6k-TFV3M8o) by Imaya Kumar

Blogs

[Data Stream Processing with Amazon Kinesis and .NET Applications](https://seroter.com/2014/01/09/data-stream-processing-with-amazon-kinesis-and-net-applications/) by Richard Seroter

[Mastering AWS Kinesis Data Streams, Part 1](https://dev.solita.fi/2020/05/28/kinesis-streams-part-1.html) by Anahit Pogosova

[Hello, Cloud blog series](https://davidpallmann.hashnode.dev/series/hello-cloud)