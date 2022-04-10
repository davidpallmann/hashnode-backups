## Hello, Kendra!

#### This episode: Amazon Kendra and enterprise search. In this Hello, Cloud blog series, we're covering the basics of AWS cloud services for newcomers who are .NET developers. If you love C# but are new to AWS, or to this particular service, this should give you a jumpstart.

In this post we'll introduce Amazon Kendra and use it in a "Hello, Cloud" .NET program to perform enterprise search with machine learning. We'll do this step-by-step, making no assumptions other than familiarity with C# and Visual Studio. We're using Visual Studio 2022 and .NET 6.

# Amazon Kendra : What is it, and why use It?

> "He who would search for pearls must dive below."—John Dryden

[Amazon Kendra](https://aws.amazon.com/kendra) (hereafter "Kendra") is a service that performs enterprise search for websites and applications. AWS describes it as "an intelligent search service powered by machine learning (ML)." With Kendra, you can provide a superior search experience for your employees and customers, even when the content is dispersed across many content repositories and locations. Kendra provides natural language search with high relevancy, easy setup, support for diverse content repositories, and 14 built-in knowledge domains such as HR and finance. Use cases include customer Q & A, research and development, regulation and compliance, and employee productivity.

![diagram.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1649534656365/MXpcVem-E.png)

To work with Kendra, you 1) create an index, 2) add data sources using [connectors](https://aws.amazon.com/kendra/connectors/), and 3) test and deploy. There are connectors for Amazon RDS databases, Amazon S3, Confluence, Google Drive, Sales Force, ServiceNow, Slack, Microsoft OneDrive, Microsoft SharePoint, and more.

As you get familiar with Kendra and experiment with it, you can take advantage of its Developer edition which includes free usage of up to 750 hours for the first 30 days. This provides 10,000 documents, supports up to 4,000 queries per day, and runs in 1 availability zone (AZ). Connectors are not free, so read through the [Free Tier](https://aws.amazon.com/kendra/pricing/) details carefully.

# Our Hello, Kendra Project

We will first create a Kendra index of S3 documents, using public-domain physics papers from Project Gutenberg. After testing search in the AWS console, we'll create a .NET Web API project that processes searches with Kendra using the AWS SDK for .NET.

![02-aws-upload-docs.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1649532246952/aOIF0Z87h.png)

![03-aws-search-sun.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1649532206650/X3emQunPM.png)

![05-search-hydrogen-03.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1649529921207/38pmzvbzB.png)

[source code](https://github.com/davidpallmann/hello-kendra)

## One-time Setup

For any of the tutorials in the Hello, Cloud series you need the following:

1. An  [AWS account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/), and an understanding of what is included in the  [AWS Free Tier](https://aws.amazon.com/premiumsupport/knowledge-center/what-is-free-tier/). 
2. [Microsoft Visual Studio 2022](https://visualstudio.microsoft.com/). If you're using an older version of Visual Studio you won't be able to use .NET 6. If you use a different IDE, you may have to find alternatives to some tutorial steps.
3. [AWS Toolkit for Visual Studio](https://aws.amazon.com/visualstudio/). You'll need to [configure the toolkit ](https://docs.aws.amazon.com/toolkit-for-visual-studio/latest/user-guide/keys-profiles-credentials.html) to access your AWS account and create an IAM user. Your default AWS profile will be linked to this user when running programs from the command line.

## Step 1: Create an Index

In this step, you'll create a Kendra index in the AWS Console.

1. Sign in to the [AWS management console](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwiAg76jl4r3AhXHrmoFHUpCAmUQFnoECAwQAQ&url=https%3A%2F%2Faws.amazon.com%2Fconsole%2F&usg=AOvVaw3L5ZM1L-1k3SwMWi6qm9p5). At top right, select the region you want to work in. You can check supported regions for Amazon Kendra on the [AWS regional services list](https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/). I'm using **us-west-2 (Oregon)**.

2. Navigate to **Amazon Kendra**. You can enter **kendra** in the search bar.

3. Click **Create index** and enter/select the following:

    A. Index name: **hello-kendra**.

    B. IAM role: **Create a new role**.

    C. Role name: **hello-kendra**. The console will assign a prefix, your full role name will look something like `AmazonKendra-us-west-2-hello-kendra`.

    D. Click **Next**.

    ![01-aws-create-index.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1649512920419/gExgfxNLB.png)

    E. On the Configure user access control page, leave the defaults and click **Next**.

    ![01-aws-create-index-02.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1649512652973/SR222wa3K.png)

    F. On the Provisioning details page, choose **Developer edition**.

    ![01-aws-create-index-03.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1649512873103/k_S--I5P7.png)

    G. Click **Create** and wait for the index to be created. This may take up to 30 minutes.

    ![01-aws-create-index-04.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1649513053996/FTXofgjp6.png)

    ![01-aws-create-index-05.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1649617618176/41xGXStoc.png)

    ![01-aws-create-index-06.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1649514987330/hWWoRLi6o.png)

## Step 2: Add an S3 Data Source

In this step, you'll create an S3 bucket, upload documents, and add an S3 data source to the Kendra index.

1. In the AWS console, navigate to Amazon S3 > Buckets and select **Create bucket**.

2. Create a bucket, in the same region you used in Step 1, with a name similar to **hello-kendra**. Since S3 bucket names must be unique, you may need to use a variation of the name if it is in use.

    ![02-aws-create-bucket.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1649515138111/yTnKUQ2Et.png)

3. Upload a collection of documents you want to index. If you want to exactly match what we're doing in the tutorial, you can upload a dozen public-domain Physics PDFs from [Project Gutenberg](https://www.gutenberg.org/). You can find the specific documents used here in the Hello, Kendra source code GitHub repository, in the [data folder](https://github.com/davidpallmann/hello-kendra/tree/main/data). If you prefer to upload your own documents, that's fine.

    ![02-aws-upload-docs.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1649520997003/yPBzB-LAv.png)

4. Note the Amazon Resource Name (ARN) of your bucket, which you can find on the bucket detail page, Properties tab. Our **hello-kendra** bucket's ARN is `arn:aws:s3:::hello-kendra`.

5. Navigate back to Amazon Kendra and select your **hello-kendra** index. Under Index settings, **record the index ID** (a GUID) for the index. You'll need to specify that when we get to writing .NET code.

6. click **Add data sources**.

    ![02-aws-add-data-source-01.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1649519197891/MlMyXannd.png)

7. From the Available data sources list, find **Amazon S3** and click **Add connector**.

    ![02-aws-add-data-source-02.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1649519207855/XATfS6lvj.png)

    On the Specify source details page, enter/select the following:

    A. Data source name: **s3-docs**.

    B. Click **Next**.

    ![02-aws-add-data-source-03.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1649519216727/R1gtljoif.png)

    On the Configure sync settings page, 

    C. for Enter the data source location, browse to and select your S3 bucket. You'll end up with a URI like ``s3://hello-kendra.

    D. For IAM role, select **Create a new role** and type a role name **kendra-s3**. The console will add a prefix, your full role name will look something like `AmazonKendra-kendra-s3`.

    E. For Sync run schedule - Frequency, select **Run on demand**.

    F. Click **Next**.

    ![02-aws-add-data-source-04.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1649519226356/o0P76KTZd.png)

    On the Review and create page, 

    G. under Sync scope, browse to the data source bucket which open up another tab window. If all is well in terms of access, your uploaded S3 documents should be visible. 

    If there is an access error, you'll need to ensure that Kendra is able to access your S3 bucket for ListBucket and GetObject operations. You can [add a bucket policy](https://docs.aws.amazon.com/AmazonS3/latest/userguide/add-bucket-policy.html) to your S3 bucket allowing access, such as the one below. If you are using an organization account or are not using test data, you should consult with your security expert and set an appropriately strong policy.

   ```JSON
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AddPerm",
            "Effect": "Allow",
            "Principal": "*",
            "Action": [
                "s3:GetObject",
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::hello-kendra",
                "arn:aws:s3:::hello-kendra/*"
            ]
        }
    ]
}
```
    H. Click **Add data source** and wait for the data source to be created.

    ![02-aws-add-data-source-05.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1649519237224/Z76-krhaH.png)

8. On the S3-docs data connector page, click **Sync now** to sync the index with your S3 documents. This may take several minutes. 

    ![02-aws-add-data-source-06.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1649519246130/Cx7FCZyXU.png)

    When the sync is completed, you should see a document count confirming your new documents have been indexed.

    ![02-aws-add-data-source-07.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1649519771366/o0bfqQC7n.png)

## Step 3: Test Kendra Search in the AWS Console

In this step, you'll test searching your index in the AWS console.

1. On the left pane, select **Search indexed content**.

2. In the search box, enter a term likely to appear in your documents. For our index of sample physics documents, we entered **relativity** and got these 10 results, including document excerpts.

    ![03-aws-test-relativity.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1649521648811/SVIkU7dBD.png)

3. Now try terms less likely to appear in most of your documents. Our search for **black hole** gave us 5 results. Our search for **perigee** had just 1 result. 

    ![03-aws-test-blackhole.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1649521788022/09tIQhBHp.png)

    ![03-aws-test-perigee.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1649521875038/gTvUapKkg.png)

3. Search for **what did the greeks believe about the cause of motion?**  and note that the top search result is highlighted as an "Amazon Kendra suggested answer". If you click the Info link, you'll see an explanation that this is an excerpt of a document that Kendra determined as the best response to the query. You can also try **what conditions must a black body satisfy?** and **what is euclid's postulate?**

    ![03-aws-kendra-suggested-answer.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1649525376961/Id9Tk8sQA.png)

    ![03-aws-kendra-suggested-answer-02.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1649526059801/CrnR-mdb6.png)

4. Experiment a bit more with searching, including multiple terms and natural language questions, and get a feel for Kendra and the relevancy of results. Try the different options in the sort box. Note the document fields returned for each result, which include a document link and the page number of the excerpt.

## Step 4: Add a Thesaurus

Since our document collection includes a Latin text, Philosophiae Naturalis Principia Mathematica by Isaac Newton, a search for "sun" won't return results from this document. However, we can teach Kendra that the Latin word for sun, "solis", is a synonym for sun. Add a synonym for Latin physics terms to Kendra:

1. Create a text file named physics.solr with the content below and upload it to your S3 bucket.

    ```SOLR
globi => globe
mundus => world
planetae => planet
planetarum => planets
solis => sun
stella => star
stellae => stars
```

2. On the left pane, select **Synonyms** and click **Add thesaurus** and enter/select the following:

    A. Thesaurus name: **physics**.

    B. Thesaurus settings - file: browse to and select `.solr` in your bucket.

    C. Role: select the same role you easier in Step 2 #7D.

    D. Click **Add** and wait for the thesaurus to be created.

    ![03-aws-thesaurus.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1649523785294/vOXAFMe2b.png)

3. On the left pane, select **Synonyms** and click **Add thesaurus** and enter/select the following:

    A. Thesaurus name: **physics**.

    B. Thesaurus settings - browse to and select physics.solr in your bucket.

    C.. Click **Add**.

4. On the left panel, select **Search indexed content** and now try a search for **sun**. This time, Sir Isaac Newton's Latin document is included in the search results. You can see the same for **planet**.

    ![03-aws-synonym-test-sun.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1649524035868/LzI2vLwA8.png)

## Step 5: Write a .NET Program for Search

Now that we've seen what Kendra can do, it's time to work with it programmatically. In this step you'll write a .NET 6 Web API project that responds to search queries using Kendra via the AWS SDK for .NET.

1. Open a command/terminal window and CD to a development folder.

2. Enter the dotnet new command below to create a console program named **hello-kendra**.

    ```dos
dotnet new webapi -n kello-kendra
```

    ![05-dotnet-new.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1649526383587/bXiDOH1fu.png)

3. Open the hello-kendra project in Visual Studio.

4. In Solution Explorer, right-click the hello-kendra project and select **Manage NuGet Packages...**. Find and install the **AWSSDK.Kendra** package.

    ![05-nuget.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1649526678531/ttUz8peFK.png)

5. In Solution Explorer, rename WeatherForecast.cs to SearchResult.cs. Our program will return a collection of this class in response to search queries.

6. Open SearchResult.cs in the code editor, and replace with the code below at the end of this step.

7. In Solution Explorer, expand the Controllers folder and rename WeatherForecastController.cs to SearchController.cs.

8. Open SearchController.cs in the editor and replace with the code below. 

    A. Replace [Index-ID]with the Index ID you record earlier in Step 2 #5.

    B. Set RegionEndpoint to the region you are using. 

9. Save your changes and build the program.

SearchResult.cs

```csharp
namespace hello_kendra;

public class SearchResult
{
    public string Title { get; set; } = null!;
    public string Excerpt { get; set; } = null!;
    public string Document { get; set; } = null!;
}
```


SearchController.cs

```csharp
using Microsoft.AspNetCore.Mvc;
using Amazon;
using Amazon.Kendra;
using Amazon.Kendra.Model;

namespace hello_kendra.Controllers;

[ApiController]
[Route("[controller]")]
public class SearchController : ControllerBase
{
    const string INDEX_ID = "[Index-ID]";

    private RegionEndpoint region = RegionEndpoint.USWest2;

    private readonly ILogger<SearchController> _logger;
    private AmazonKendraClient client;

    public SearchController(ILogger<SearchController> logger)
    {
        _logger = logger;
        client = new AmazonKendraClient(region);
    }

    [HttpGet]
    public async Task<IEnumerable<SearchResult>> Get([FromQuery] string term)
    {
        var request = new QueryRequest()
        {
            IndexId = INDEX_ID,
            QueryText = term
        };

        var response = await client.QueryAsync(request);

        return response.ResultItems.Select(result => new SearchResult
        {
            Title = result.DocumentTitle.Text,
            Document = result.DocumentURI,
            Excerpt = result.DocumentExcerpt.Text
        })
        .ToArray();
    }
}

```

### Understand the Code

Here's what this short piece of code does. In SearchController.cs, the constructor instantiates an `AmazonKendraClient`, passing in the AWS region. The controller implements a /Search HTTP GET operation for searching with a query parameter named `term`. Kendra is invoked to perform the search by creating a `QueryRequest` object with the Kendra index ID and query term, and passing that to the client's `QueryAsync` method. The returned `QueryResponse` response includes a collection of results named `ResultItems`. We iterate over that collection to generate our collection of `SearchResult` objects, populated with document title, document link, and excerpt text from the results.

## Step 6: Test the Program

In this step, you'll run your .NET code and see it interact with Kendra to perform searches.

1. In Visual Studio, press F5 to run the program. A Swagger page will open in your browser showing a single /Search GET operation.

    ![05-search-sun-01.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1649529211489/wXkGICA8X.png)

2. Expand the /Search action and click **Try it out**.

3. In the input box for `term`, enter **sun**.

4. Click **Execute**

    ![05-search-sun-02.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1649529486204/niuh0Szmq.png)

5. Review the search results in the Response body, which should contain document search results matching the query.

    ![05-search-sun-03.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1649529571829/onze6J9xc.png)

6. Try other terms. Enjoy your handiwork!

    ![05-search-hydrogen-03.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1649529895017/1Khv-8yiV.png)

Congratulations! You've used Amazon Kendra in .NET code to perform a search against an index of documents.

## Step 7: Shut it Down

When you're done with Hello, Kendra, deallocate your S3 bucket and the Kendra index. You don't want to be charged something you're not using.

1. In the AWS console, navigate to Amazon Kendra > Indexes. Select **hello-kendra** and select **Delete** under the Actions dropdown. Confirm the delete prompt.

2. Navigate to Amazon S3 > Buckets. Click on your S3 bucket and delete its objects, and then the bucket itself.

# Where to Go From Here

More and more, search is expected by customers and employees. Effective search can increase customer discovery and employee productivity. With Amazon Kendra, you can provide high-quality enterprise search against your documents, databases, and other application data through connectors.

In this tutorial, you created an index and synced it with documents in an S3 bucket. You saw search work in the AWS console. You set up a thesaurus, and saw Kendra suggested answers in search results. You created a .NET Web API project, and implemented a search action that queried Kendra on the back end and returned search results.

To go further with Amazon Kendra, learn more about it and practice working with it. This tutorial did not cover a number of [Kendra features](https://aws.amazon.com/kendra/features/), including curated FAQ matching, machine learning from user feedback, fine-tuning search results, built-in domains, experience builder, search analytics, document enrichment, and query auto-completion.

# Further Reading

AWS Documentation

[Amazon Kendra](https://aws.amazon.com/kendra)

[Amazon Kendra Connectors](https://aws.amazon.com/kendra/connectors/)

[Amazon Kendra Developer Guide](https://docs.aws.amazon.com/kendra/latest/dg/index.html)

[AWS SDK for .NET Documentation](https://docs.aws.amazon.com/sdk-for-net/index.html)

Videos

[Amazon Kendra Resources](https://aws.amazon.com/kendra/resources/?blog-cards.sort-by=item.additionalFields.createdDate&blog-cards.sort-order=desc)

Blogs

[Enhancing Enterprise Search with Amazon Kendra](https://aws.amazon.com/blogs/machine-learning/enhancing-enterprise-search-with-amazon-kendra/) by  Leonardo Gomez

[Amazon Kendra: Enterprise Search Powered by ML](https://medium.com/@ksivamuthu/amazon-kendra-enterprise-search-powered-by-ml-d29dc24799c6) by Sivamuthu Kumar

[Hello, Cloud blog series](https://davidpallmann.hashnode.dev/series/hello-cloud)