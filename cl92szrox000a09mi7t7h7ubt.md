# Hello, Timestream!

#### This episode: Amazon Timestream and time series data. In this Hello, Cloud blog series, we're covering the basics of AWS cloud services for newcomers who are .NET developers. If you love C# but are new to AWS, or to this particular service, this should give you a jumpstart.

In this post we'll introduce Amazon Timestream and use it in a "Hello, Cloud" .NET program to capture and query time series data. We'll do this step-by-step, making no assumptions other than familiarity with C# and Visual Studio. We're using Visual Studio 2022 and .NET 6.

# Amazon Timestream : What is it, and why use It?

> "Time is but the stream I go a-fishing in." —Henry David Thoreau 

How do you capture a relentless stream of data over time? More and more, "time stream" data is becoming important for businesses to track. There are long-standing traditional use cases for time stream data, like tracking stock prices or local weather, and newer use cases such as gathering sensor data from IoT devices. Relational databases aren't the best fit for all jobs, which is why purpose-built databases have been on the rise. Time series data is best captured in a time series database that is optimized for massive scale.

Amazon Timestream (hereafter "Timestream") is a service for capturing time stream data. AWS describes it as a "fast, scalable, serverless time series database". Timestream can handle trillions of events per day, 1,000 times faster than relational databases and at lower cost. Timestream keeps recent data in memory and moves historical data to a storage tier under policies you control. Timestream is serverless, so there's no infrastructure to manage. It automatically scales to adjust capacity to keep up with load.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665403648768/EplewJ0Xe.png align="left")

You can query what you store in Timestream. The queries will retrieve both in-memory records and records committed to the persistent storage tier. The AWS SDK for .NET provides an `AmazonTimestreamWriteClient` for writing to tables, and an `AmazonTimestreamQueryClient` for querying tables.

Key features of Timestream are its serverless auto-scaling architecture, data storage tiering and data lifecycle management, query engine, time series analytics, automatic encryption, and integration with other AWS services. You can send data to Timestream from AWS IoT Core, Amazon Kinesis, Amazon MSK, and open source Telegraf. You can query or visualize data from Timestream with Amazon QuickSight, Grafana.

Timestream charges for the following. See the [Pricing Page](https://aws.amazon.com/timestream/pricing/) for details. There is no AWS Free Tier consideration.
* Writes: The amount of data written from your applications (rounded to the nearest KB) into a table.
* Queries: The amount of data scanned by Amazon Timestream’s serverless distributed query engine while computing query results (rounded to the nearest MB, with a 10 MB minimum).
* Memory store: The amount of data stored in the memory store of each table.
* Magnetic store: The amount of data stored in the magnetic store of each table.


# Our Hello, Timestream Project

We will write a .NET program to capture bird sighting information in a Timestream database. For example, you might sight a California Condor in Yakima Washington. Our program will capture sightings like that, and allow them to be queried.

![dotnet-run-help.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665406274875/1dPXg_ZCU.png align="left")

![dotnet-run-seattle.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665406206801/Z1OJ-jFIZ.png align="left")

![dotnet-run-recent.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665406223676/4v2MtXmUf.png align="left")

![dotnet-run-sightings.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665406235304/U9y-_4Biz.png align="left")

[source code](https://github.com/davidpallmann/hello-timestream)

## One-time Setup

For any of the tutorials in the Hello, Cloud series you need the following:

1. An  [AWS account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/), and an understanding of what is included in the  [AWS Free Tier](https://aws.amazon.com/premiumsupport/knowledge-center/what-is-free-tier/). 
2. [Microsoft Visual Studio 2022](https://visualstudio.microsoft.com/). If you're using an older version of Visual Studio you won't be able to use .NET 6. If you use a different IDE, you may have to find alternatives to some tutorial steps.
3. [AWS Toolkit for Visual Studio](https://aws.amazon.com/visualstudio/). You'll need to [configure the toolkit ](https://docs.aws.amazon.com/toolkit-for-visual-studio/latest/user-guide/keys-profiles-credentials.html) to access your AWS account and create an IAM user. Your default AWS profile will be linked to this user when running programs from the command line.

## Step 1: Create Timestream Database

In this step, you'll create a Timestream database in the AWS console for bird watching. The timestream will contain a table for recording bird sightings.

1. Sign in to the AWS console. At top right, select the region you want to work in. You can check supported regions for Timestream on the [pricing page](https://aws.amazon.com/timestream/pricing/). I'm using **us-west-2 (Oregon)**.

2. Navigate to **Amazon Timestream**. 

3. Create a database.

    A. Click **Create database**

    B. Database configuration: select **Standard database**.

    C. Name: **hello-timestream**.

    D. Click **Create database**.

    ![aws-create-database.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665346376666/I-TY_0btX.png align="left")

4. Create a table:

    A. Once the table is created, click on its name to view its detail.

    B. Select the **Tables** tab.

    C. Click **Create table**.

    D. Table name: **bird**.

    E. Click **Create table**.

    ![aws-create-table.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665347579447/7C8EzxSx7.png align="left")

## Step 2: Create a .NET Program

In this step, you'll create a .NET program for capturing data into the Timestream database.

1. Open a command/terminal window and CD to a development folder.

2. Run the dotnet new command below to create a new console program.

   ```dos
dotnet new console -n hello-timestream
```
    ![dotnet-new.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665348095190/QJvTD1x2O.png align="left")

3. CD to the project folder and run the commands below to add AWS SDK packages for Timestream and Microsoft configuration packages:

   ```dos
dotnet add package AWSSDK.Core
dotnet add package AWSSDK.TimestreamWrite
dotnet add package AWSSDK.TimestreamQuery
dotnet add package Microsoft.Extensions.Configuration
dotnet add package Microsoft.Extensions.Configuration.Json
```

    ![dotnet-add.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665348260809/lWj-l6WNE.png align="left")

4. Open the project in Visual Studio or your preferred IDE.

5. Add new file appsettings.json with the content below at the end of this step. Ensure the database name and table name match the names you used in Step 1.

    In Solution Explorer, select appsettings.json and press F4 for the properties window. Set *Copy to Output Directory* to **Copy if Newer**.

6. Add new file TimestreamHelper.cs with the code at the end of this step.

7. Replace Program.cs with the code below at the end of this step.

8. Now, let's take a moment to understand the code. 

Most of the code to talk to Timestream via the AWS SDK for .NET is in TimestreamHelper. We'll go through that code when we test writing records and querying records. Program.cs parses the command line and calls the appropriate method in TimestreamHelper.

The program knows how to perform these actions:

| Command | Result |
| ----------- | ------ |
| record a bird sighting | dotnet run -- [city] [region] [country] [species] [count] | 
| example | dotnet run -- Seattle WA USA "Northern Spotted Owl" 3 |
| example | dotnet run -- Olympia WA USA "California Condor" 1; |
|  list all records | dotnet run -- list |
| list 5 more recent sightings  | dotnet run -- recent |
| list sightings for a species | dotnet run -- sightings [species] |
| example | dotnet run -- sightings "California Condor" |

appsettings.json

```json
{
  "timestream": {
    "database": "hello-timestream",
    "table": "birds"
  }
}
```

TimestreamHelper.cs

```csharp
#pragma warning disable CS1998

using Amazon;
using Amazon.TimestreamQuery;
using Amazon.TimestreamQuery.Model;
using Amazon.TimestreamWrite;
using Amazon.TimestreamWrite.Model;
using System.Text.Json;

namespace HelloTimestream
{
    public class TimestreamHelper
    {
        public string DatabaseName = null!;
        public string TableName = null!;
        public const long HT_TTL_HOURS = 24;
        public const long CT_TTL_DAYS = 7;

        public TimestreamHelper(string database, string table)
        {
            DatabaseName = database;
            TableName = table;
        }

        #region Write

        /// <summary>
        /// Write records to timestream table.
        /// </summary>
        /// <param name="records">List of records to write</param>
        /// <returns><WriteRecordResponse/returns>
        public async Task<WriteRecordsResponse> WriteRecordsAsync(params Record[] records)
        {
            var writeClientConfig = new AmazonTimestreamWriteConfig
            {
                Timeout = TimeSpan.FromSeconds(20),
                MaxErrorRetry = 10
            };

            var writeClient = new AmazonTimestreamWriteClient(writeClientConfig);

            var writeRecordsRequest = new WriteRecordsRequest
            {
                DatabaseName = DatabaseName,
                TableName = TableName,
                Records = new List<Record>(records)
            };

            WriteRecordsResponse response = await writeClient.WriteRecordsAsync(writeRecordsRequest);

            return response;
        }

        #endregion

        #region Query

        /// <summary>
        /// Query timestream table
        /// Created based on https://docs.aws.amazon.com/timestream/latest/developerguide/code-samples.run-query.html
        /// </summary>
        /// <returns>List of strings</returns>
        public async Task<List<string>> QueryRecordsAsync(string query)
        {
            var results = new List<string>();

            var queryClientConfig = new AmazonTimestreamQueryConfig
            {
                Timeout = TimeSpan.FromSeconds(20),
                MaxErrorRetry = 10
            };

            var queryClient = new AmazonTimestreamQueryClient(queryClientConfig);


            var queryRequest = new QueryRequest()
            {
                QueryString = query
            };

            var queryResponse = await queryClient.QueryAsync(queryRequest);

            while (true)
            {
                results.AddRange(await ParseQueryResult(queryResponse));
                if (queryResponse.NextToken == null) break;
                queryRequest.NextToken = queryResponse.NextToken;
                queryResponse = await queryClient.QueryAsync(queryRequest);
            }

            return results;
        }

        private static async Task<List<string>> ParseQueryResult(QueryResponse response)
        {
            var results = new List<string>();

            List<ColumnInfo> columnInfo = response.ColumnInfo;
            var options = new JsonSerializerOptions
            { 
                DefaultIgnoreCondition = System.Text.Json.Serialization.JsonIgnoreCondition.WhenWritingNull
            };
            List<String> columnInfoStrings = columnInfo.ConvertAll(x => JsonSerializer.Serialize(x, options));
            List<Row> rows = response.Rows;

            QueryStatus queryStatus = response.QueryStatus;

            foreach (Row row in rows)
            {
                results.Add(ParseRow(columnInfo, row));
            }

            return results;
        }

        private static string ParseRow(List<ColumnInfo> columnInfo, Row row)
        {
            List<Datum> data = row.Data;
            List<string> rowOutput = new List<string>();
            for (int j = 0; j < data.Count; j++)
            {
                ColumnInfo info = columnInfo[j];
                Datum datum = data[j];
                rowOutput.Add(ParseDatum(info, datum));
            }
            return $"{{{string.Join(",", rowOutput)}}}";
        }

        private static string ParseDatum(ColumnInfo info, Datum datum)
        {
            if (datum.NullValue)
            {
                return $"{info.Name}=NULL";
            }

            Amazon.TimestreamQuery.Model.Type columnType = info.Type;
            if (columnType.TimeSeriesMeasureValueColumnInfo != null)
            {
                return ParseTimeSeries(info, datum);
            }
            else if (columnType.ArrayColumnInfo != null)
            {
                List<Datum> arrayValues = datum.ArrayValue;
                return $"{info.Name}={ParseArray(info.Type.ArrayColumnInfo, arrayValues)}";
            }
            else if (columnType.RowColumnInfo != null && columnType.RowColumnInfo.Count > 0)
            {
                List<ColumnInfo> rowColumnInfo = info.Type.RowColumnInfo;
                Row rowValue = datum.RowValue;
                return ParseRow(rowColumnInfo, rowValue);
            }
            else
            {
                return ParseScalarType(info, datum);
            }
        }


        private static string ParseTimeSeries(ColumnInfo info, Datum datum)
        {
            var timeseriesString = datum.TimeSeriesValue
                .Select(value => $"{{time={value.Time}, value={ParseDatum(info.Type.TimeSeriesMeasureValueColumnInfo, value.Value)}}}")
                .Aggregate((current, next) => current + "," + next);

            return $"[{timeseriesString}]";
        }

        private static string ParseScalarType(ColumnInfo info, Datum datum)
        {
            return ParseColumnName(info) + datum.ScalarValue;
        }

        private static string ParseColumnName(ColumnInfo info)
        {
            return info.Name == null ? "" : (info.Name + "=");
        }

        private static string ParseArray(ColumnInfo arrayColumnInfo, List<Datum> arrayValues)
        {
            return $"[{arrayValues.Select(value => ParseDatum(arrayColumnInfo, value)).Aggregate((current, next) => current + "," + next)}]";
        }

        #endregion
    }
}
```

Program.cs

```csharp
using Amazon.TimestreamWrite.Model;
using Microsoft.Extensions.Configuration;

namespace HelloTimestream
{
    public class Program
    {
        public static async Task Main(string[] args)
        {
            IConfiguration config = new ConfigurationBuilder().AddJsonFile("appSettings.json").Build();
            var databaseName = config["timestream:database"];
            var tableName = config["timestream:table"];

            TimestreamHelper timestreamHelper = new TimestreamHelper(databaseName, tableName);

            if (args.Length == 1 && args[0] == "list")
            {
                // Query all bird sighting records

                var results = await timestreamHelper.QueryRecordsAsync($"select * from \"{databaseName}\".{tableName}");
                foreach(string line in results)
                {
                    Console.WriteLine(line);
                }
                Environment.Exit(0);
            }

            if (args.Length == 1 && args[0] == "recent")
            {
                // Query most recent bird sighting records

                Console.WriteLine("Most recent records:");
                var results = await timestreamHelper.QueryRecordsAsync($"select * from \"{databaseName}\".{tableName} order by time DESC limit 5");
                foreach (string line in results)
                {
                    Console.WriteLine(line);
                }
                Environment.Exit(0);
            }

            if (args.Length == 2 && args[0] == "sightings")
            {
                // Query sightings for a specific species

                var species = args[1];
                Console.WriteLine($"Sightings for {species}:");
                var results = await timestreamHelper.QueryRecordsAsync($"select * from \"{databaseName}\".{tableName} where species='{species}' order by time ASC");
                foreach (string line in results)
                {
                    Console.WriteLine(line);
                }
                Environment.Exit(0);
            }

            if (args.Length < 5)
            {
                Console.WriteLine("To write records:       dotnet run -- [city] [region] [country] [species] [count]");
                Console.WriteLine("                        dotnet run -- Seattle WA USA \"Northern Spotted Owl\" 3");
                Console.WriteLine("                        dotnet run -- Olympia WA USA \"California Condor\" 1");
                Console.WriteLine("To list records:        dotnet run -- list");
                Console.WriteLine("To list recent records: dotnet run -- recent");
                Console.WriteLine("To list sightings:      dotnet run -- sightings [species]");
                Console.WriteLine("                        dotnet run -- sightings \"California Condor\"");
                Environment.Exit(0);
            }

            // Write bird sighting record

            try
            {
                var city = args[0];
                var region = args[1];
                var country = args[2];
                var species = args[3];
                var count = args[4];

                DateTimeOffset now = DateTimeOffset.UtcNow;
                string currentTimeString = (now.ToUnixTimeMilliseconds()).ToString();

                List<Dimension> dimensions = new List<Dimension> {
                    new Dimension { Name = "city", Value = city },
                    new Dimension { Name = "region", Value = region },
                    new Dimension { Name = "country", Value = country },
                    new Dimension { Name = "species", Value = species }
                };

                var birdSightingRecord = new Record
                {
                    Dimensions = dimensions,
                    MeasureName = "count",
                    MeasureValue = count,
                    MeasureValueType = Amazon.TimestreamWrite.MeasureValueType.BIGINT,
                    Time = currentTimeString
                };

                Console.WriteLine("Writing record");

                var response = await timestreamHelper.WriteRecordsAsync(birdSightingRecord);
                Console.WriteLine($"Write records status code: {response.HttpStatusCode.ToString()}");
                if (response.HttpStatusCode != System.Net.HttpStatusCode.OK)
                {
                    Console.WriteLine("Error writing records - bad HTTP response");
                }
            }
            catch (RejectedRecordsException ex)
            {
                Console.WriteLine("Record rejected: " + ex.ToString());
            }
            catch (Exception ex)
            {
                Console.WriteLine("Error writing records:" + ex.ToString());
            }
        }

    }
}
```

## Step 3: Record Bird Sightings with the .NET Program

In this step, you'll run the program to capture some bird sighting time series data.

1. In the command/terminal window, run the two commands below. You should see confirmation the record was written and an OK response was received from Amazon Timestream.

    ```dos
dotnet run Seattle WA USA "Spotted Northern Owl" 3
dotnet run Seattle WA USA "California Condor" 1
```

    ![dotnet-run-seattle.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665351434129/PCi-yHkcD.png align="left")

2. Run more commands so we have more data on file. In addition to the sample records below, feel free to add your own. Just remember to be consistent about place names and species names.

    ```dos
dotnet run Olympia WA USA "Brown Pelican" 1
dotnet run Olympia WA USA "Canada Goose" 17
dotnet run Olympia WA USA "Spotted Northern Owl" 1
```

3. Wait a few minutes, then enter more data with the commands below.

    ```dos
dotnet run Vancouver BC Canada "Brown Pelican" 1
dotnet run Vancouver BC Canada "Canada Goose" 5
dotnet run Vancouver BC Canada "Sandhill Crane" 2
dotnet run Vancouver BC Canada "Canada Goose" 5
```

4. Add some additional records, including more locations and more species.

5. Let's understand the code. In Program.cs, we create the record to be written, `birdSightingRecord`. it consists of the time, a measure, and dimensions. The time is the time of the event. Measure is the primary metric being recorded, the number of birds sighted (`MeasureName` and `MeasureValue`). Dimensions are other attributes we want to attach to the record, in this case the location (`city`, `region`, `country`, `species`). We'll be able to query on any of these attributes. 

We call `TimeStreamHelper.WriteRecordsAsync` to do the work, passing our bird sighting records. To write the records, we instantiate an `AmazonTimestreamWriteClient`, then create a `WriteRecordsRequest` containing the records to be written (just one in our case), along with the database name and table name. A call to `writeClient.WriteRecordsAsync` performs the write, and returns an HttpStatusCode we can check for success.

## Step 4: Query Database in AWS Console

Now that we've added some records to Timestream, let's see how to query it. We'll do that from the AWS console, but you can also do it programmatically.

1. In the AWS console, select **Query editor** from the left panel.

2. In the query box, enter the query below.

    ```sql
select * from "hello-timestream".birds
```

3. Click **Run**. After a moment, you see matching rows displayed below. You should see all of the data you have entered so far. The measure_value column is the count of birds sighted.

    ![aws-query-all.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665352288285/kmzgVDAHN.png align="left")

4. Now modify the query to restrict results to one location, Seattle. You can do that with a `where` clause. Use single quotes around literal values.

    ```sql
select * from "hello-timestream".birds where city='Seattle'
```

    ![aws-query-where-city.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665351947669/ZvMFjX39n.png align="left")

5. Lastly, let's query by time. Run the query below to see the most recent 5 records added to the time stream database:

    ```sql
select * from "hello-timestream".birds
where measure_name = 'count'
order by time DESC
limit 5
```

    ![aws-query-recent.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665352482088/1lLV0qkSa.png align="left")

## Step 5: Query Database Programatically in .NET Program

We've seen how to query the database in the AWS console. Now let's do it from .NET code.

1. From the command/terminal window, run the command below. You see all records listed.

    ```dos
dotnet run -- list
```
You see all records listed. This used the query, `select * from database.table`.

    ![dotnet-run-list.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665404991438/5X2TFkbYK.png align="left")

2. Run the command below. 

   ```dos
dotnet run -- recent
```
You see the most recent 5 records listed. This used the query, `select * from database.table order by time DESC limit 5`.

    ![dotnet-run-recent.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665404849975/OKdUAer_l.png align="left")

3. Now, let's query for sightings by species. Run the commands below

   ```dos
dotnet run -- sightings "Canada Goose"
dotnet run -- sightings "California Condor"
dotnet run -- sightings "Spotted Northern Owl"
```
You see records just for the species you requested. This used the query, `select * from database.table where species={species} order by time ASC`.

    ![dotnet-run-sightings.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1665404697857/mmVyXAO8m.png align="left")

4. Let's understand the query code. In Program.cs, these 3 actions use the same pattern of invoking `TimestreamHelper.QueryRecordsAsync` and passing a SQL query.  The helper method creates an`AmazonTimestreamQueryClient`. A simple SDK call to `QueryAsync`, with the SQL query in the request, queries the timestream table, which will scan both in-memory and storage-tier records.

    Less simple is getting the results back. A series of private methods are called to extract the result pages and deserialize them into strings. These methods are based on the [Amazon Timestream Developer Guide](https://docs.aws.amazon.com/timestream/latest/developerguide/code-samples.html) .NET code examples for common patterns. A list of strings is returned, and the code in Main displays them to the console.

    Congratulations! You've captured time stream data to Amazon Timestream and queried it from .NET code using the AWS SDK for .NET.

## Step 6: Shut it Down

When you're done with the Hello Timestream project, shut it down. You don't want to accrue charges for something you're not using.

1. In the AWS console, navigate to **Timestream > Tables**.

2. Select **birds** and click **Delete**. Confirm the deletion.

3. On the left panel, click **Databases**.

4. Select **hello-timestream** and click **Delete**. Confirm the deletion.

# Where to Go From Here

Time stream data is becoming more and more important to businesses. Amazon Timestream is a purpose-built database for capturing and querying timestream data that is easy to set up and use.

In this tutorial, you created a simple Timestream database and table for bird sightings. You wrote code to use the AWSK SDK for .NET to write data to and the table and query the table.

This simple tutorial did not cover Timestream's advanced features such as built-in time series analytics and integrations with other AWS services. To go further, read the documentation and other resources cited below, experiment, and get familiar with the service.

# Further Reading

AWS Documentation

[Amazon Timestream](https://aws.amazon.com/timestream/)

[Amazon Timestream Developer Guide](https://docs.aws.amazon.com/timestream/latest/developerguide/what-is-timestream.html)

[Amazon Timestream Code Samples](https://docs.aws.amazon.com/timestream/latest/developerguide/code-samples.html)

[Amazon Timestream Simple Queries](https://docs.aws.amazon.com/timestream/latest/developerguide/sample-queries.basic-scenarios.html)

[Amazon Timestream Tools and Samples](https://github.com/awslabs/amazon-timestream-tools)

[AWS SDK for .NET Documentation](https://docs.aws.amazon.com/sdk-for-net/index.html)

Videos

[Introduction to Amazon Timestream](https://www.youtube.com/watch?v=YBWCGDd4ChQ)

[Deep Dive on AmazonTimestream](https://www.youtube.com/watch?v=39ijv_pfWSQ) by Toby Gibbs

Blogs

[Amazon Timestream - Time series is the new black](https://www.allthingsdistributed.com/2021/06/amazon-timestream-time-series-is-the-new-black.html) by Werner Vogels

[Time Series Databases](https://billthevestguy.com/2022/04/26/time-series-databases/) by Bil Penberthy

[Rare Birds of the Pacific Northwest](https://birdwatchingpro.com/rare-birds-of-the-pacific-northwest/) by Bird Watching Pro Fenley (tutorial inspiration)

[Hello, Cloud blog series](https://davidpallmann.hashnode.dev/series/hello-cloud)