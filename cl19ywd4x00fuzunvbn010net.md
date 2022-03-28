## Hello, Personalize!

#### This episode: Amazon Personalize. In this Hello, Cloud blog series, we're covering the basics of AWS cloud services for newcomers who are .NET developers. If you love C# but are new to AWS, or to this particular service, this should give you a jumpstart.

In this post we'll introduce Amazon Personalize and use it in a "Hello, Cloud" .NET program to make personalized product recommendations. We'll do this step-by-step, making no assumptions other than familiarity with C# and Visual Studio. We're using Visual Studio 2022 and .NET 6.

# Amazon Personalize: What is it, and why use It?

> "I'm most passionate about personalization. I firmly believe personalized experiences with brands will most drive loyalty and relevance for customers in the future." —Katrina Lake, Founder & CEO, Stitch Fix 

Personalization is something you run into more and more, and that's because it's effective. From search results to your phone's news feed to social media ads to video streaming recommendations, you've no doubt encountered personalization in action and likely benefited from it. 

[Amazon Personalize](https://aws.amazon.com/personalize) (hereafter "Personalize") is a service that provides personalization services for your application. As AWS describes it, "Amazon Personalize enables developers to build applications with the same machine learning (ML) technology used by Amazon.com for real-time personalized recommendations – no ML expertise required." With Personalize, you can create recommendation systems quickly, and provide quality recommendations to your users in real-time. You can use Personalize from a broad spectrumof applications including web sites, mobile/desktop apps, and email/SMS marketing systems.

Services provided by Personalize include product recommendations, product ranking, and customized direct marketing. Personalize has these notable features:

• **Recommenders**: get personalized recommendations for a user, such as "top picks for you" or "because you watched x". 

• **User segmentation**: discern segments of users, such as gauging interest in different product categories or brands. 

• **Automated machine learning**: Personalize trains ML models by inspecting your uploaded data.

• **Real-time recommendations**: as user intent changes, so do Personalize's recommendations. 

• **New user recommendations**: generate recommendations for new users even when there is no interaction history.

• **Similar item recommendations**: offer similar items to your users.

![diagram-personalize.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648420929562/m_YNKgFT7.png)

To create a recommendation system, you follow these steps:

1. **Choose your domain**. You can choose e-commerce, video on demand, or create a custom domain.
2. **Import your data**. Upload your data to S3 and import it to train a model.
3. **Generate recommenders, solutions, campaigns, and event trackers**: You configure and generate a variety of artifacts to make ML models trained with your data available for application use.
4. **Get recommendations**: Use your recommendation system, retrieving personalized user recommendations and sending user events in real-time to keep your interaction data up-to-date.

The [AWS Free Tier](https://aws.amazon.com/personalize/pricing/?loc=ft#Free_Tier) gives you 2 months use of Amazon Personalize for free.

## Concepts

Let's review some Personalize concepts and terms:

A **recipe** is a pre-configured algorithm. User personalization recipes can predict what a user will interact with. Related items recipes can find items similar to an item a user is interested in, or rank items based on predicted interest.

A **solution** is a combination of a recipe, parameters, and one or more solution versions. 

A **solution version** is a trained machine learning model you can deploy to get recommendations for customers. 

A **campaign** is a deployed solution version. We can invoke the campaign from our application to get recommendations.

An **event tracker** lets us report new events into the interactions dataset, such as a user watching a video or purchasing an item. We train our recommender with existing historical data, but as our application is used, we need to keep that data current. We do that by reporting user events via an event tracker. 

# Our Hello, Personalize Project

We will configure Personalize for personalized movie recommendations. We'll be spending time in the AWS console uploading data to S3, creating artifacts in Amazon Personalize, training a model with the data, and deploying a campaign. After that we'll call Personalize from .NET code to get personalized movie recommendations based on user Id. We'll also report user movie watch events to Amazon Personalize to show how the interaction data can be kept up to date with new user activity in real-time. 

![05-aws-test-campaign-2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648425742017/xMEihZuDi.png)

![06-dotnetrun-1.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648327899153/6JicCVNtP.png)

![06-dotnetrun-777-watch-events.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648425585852/M3zSVB-8p.png)

[source code](https://github.com/davidpallmann/hello-personalize)

## One-time Setup

For any of the tutorials in the Hello, Cloud series you need the following:

1. An [AWS account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/), and an understanding of what is included in the  [AWS Free Tier](https://aws.amazon.com/premiumsupport/knowledge-center/what-is-free-tier/). 
2. [Microsoft Visual Studio 2022](https://visualstudio.microsoft.com/). If you're using an older version of Visual Studio you won't be able to use .NET 6. If you use a different IDE, you may have to find alternatives to some tutorial steps.
3. [AWS Toolkit for Visual Studio](https://aws.amazon.com/visualstudio/). You'll need to [configure the toolkit ](https://docs.aws.amazon.com/toolkit-for-visual-studio/latest/user-guide/keys-profiles-credentials.html) to access your AWS account and create an IAM user. Your default AWS profile will be linked to this user when running programs from the command line.

## Step 1: Create S3 Bucket and IAM Service Role in the AWS Console

In this step, you'll create an S3 bucket plus IAM policies and a service role for Amazon Personalize. The S3 bucket is where training data will be uploaded. The service role and policies will allow Amazon Personalize to access the S3 bucket. 

Note: if you have Block Public Access active for S3, Amazon Personalize will not be able to access your S3 bucket.

1. Sign in to the [AWS management console](https://aws.amazon.com/console/). At top right, select the region you want to work in. You can check supported regions for Amazon Personalize on the [AWS Regional Services](https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/) page. I'm using **us-west-2 (Oregon)**.

2. Navigate to **Amazon S3**. You can enter **s3** in the search bar,.

3. Click **Create bucket** and enter/select the following to create an S3 bucket:

    A. Bucket name: enter a name similar to **hello-personalize**. The name must be unique, so you may need to use a variation. Record your bucket name.

    B. Region: set to the same region you selected in #1 above.

    C. At bottom, click **Create bucket**.

    ![01-create-bucket.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648294877067/xe6vy7S6Z.png)

    ![01-create-bucket-02.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648294950901/tVPqJr1hN.png)

4. Add a bucket policy allowing Personalize to access the bucket:

    A. In the Buckets list, click your bucket name to view its detail.

    B. Click **Permissions**.

    C. Under Bucket policy, click **Edit**.

    D. On the JSON tab, enter the JSON below, replacing [bucket-name] with your bucket name.

    E. Click **Save changes**.

   ```json
{
    "Version": "2012-10-17",
    "Id": "PersonalizeS3BucketAccessPolicy",
    "Statement": [
        {
            "Sid": "PersonalizeS3BucketAccessPolicy",
            "Effect": "Allow",
            "Principal": {
                "Service": "personalize.amazonaws.com"
            },
            "Action": [
                "s3:GetObject",
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::[bucket-name]",
                "arn:aws:s3:::[bucket-name]/*"
            ]
        }
    ]
}
```

5. Navigate to **Identity and Access Management**. You can enter **iam** in the search bar.

6. Create a policy allowing Personalize to access your resources. This policy assumes you are working in a developer test account. If you are working in an organizational account, review the policy with your security champion.

    A. Select **Policies** on the left panel and click **Create Policy**.

    B. On the JSON tab, enter the JSON below.

    ```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "personalize:*"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "iam:PassRole"
            ],
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "iam:PassedToService": "personalize.amazonaws.com"
                }
            }
        }
    ]
}
```

    ![01-iam-policy.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648250082804/57gt9f5fD.png)

    C. Click **Next: Tags** and then **Review**.

    D. For name, enter **personalize**.

    E. Click **Create policy**.

    ![01-iam-policy-02.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648250185916/Jmnbd_tIT.png)

7. Create an S3 policy for the Personalize service:

    A. In the AWS console, navigate to IAM > Policies.

    B. Click **Create policy**.

    C. In the JSON tab, enter the JSON below. Replace [bucket-name] with the name of the S3 bucket you created earlier.

    ```json
{
    "Version": "2012-10-17",
    "Id": "PersonalizeS3BucketAccessPolicy",
    "Statement": [
        {
            "Sid": "PersonalizeS3BucketAccessPolicy",
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:ListBucket",
                "s3:PutObject"
            ],
            "Resource": [
                "arn:aws:s3:::[bucket-name]",
                "arn:aws:s3:::[bucket-name]/*"
            ]
        }
    ]
}
```

    ![01-create-s3-policy.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648295111477/E11ee_irr.png)

    D. Click **Next: Tags** and then **Review**.

    E. Name the policy **personalize-s3** and click **Create policy**.

    ![01-create-s3-policy-02.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648295183200/tCMCvL1V2.png)

8. Create a service role for Personalize:

    A. In the left panel, click **Roles** and **Create role**.

    B. For Select trusted entity, choose **AWS Service**.

    C. For Use cases for other AWS services, select **Personalize**.

    ![01-iam-role.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648250718117/_smyN6x2o.png)

    D. Click **Next**.

    E. On the Add permissions page, enter **personalize** in the search box and select the **personalize** policy you created earlier. 

    ![01-iam-role-02.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648250873299/5fS_edhtw.png)

    F. In the same way, search for **personalize-s3** and also select that policy.

    F. Click **Next**.

    G. On the Role detail page, enter role name **personalize-service**.

    H. Click **Create role**.

    I. Once the role is created, record the Amazon Resource Name (ARN) for the role.

    ![01-iam-role-03.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648302947480/m0eUqgd5Z.png)

    ![01-iam-role-04.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648251307124/aFwlDNVny.png)

## Step 2: Upload Training Data

In this step you'll upload training data to S3 for Amazon Personalize to ingest. Personalize needs a fair amount of data in order to perform its function (at least 1000 user interactions), so we're going to use the same sample data used in an existing [Amazon Personalize tutorial](https://docs.aws.amazon.com/personalize/latest/dg/gs-prerequisites.html#gs-data-prep-domain) for Python. The data is movie titles and ratings data from [MovieLens](http://movielens.org), a movie recommendation service. 

1. Download the data files, which can be found in the [GitHub source code](personalize) for this project. In the `data` folder, you'll find the data files you need. We'll look at these files later to relate recommended Item IDs to movie titles. As the README mentions, these files are public domain from MovieLens. We've added one derived file, watch.csv. This watch.csv file was created from this source data and has been formatted for Personalize. 

2. Take a moment to view the watch.csv file. This data captures which movies were watched by users, which will drive viewing recommendations. In it, you see columns USER_ID, ITEM_ID, TIMESTAMP, and EVENT_TYPE. These are the fields Personalize will expect for video training data. 

    ![02-excel-watch-csv.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648303137935/ocTsKeStC.png)

    If you view movies.csv that you retrieved in #1 above, you'll see how item IDs relate to movie titles and genres.

    ![04-movies-csv.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648328633524/bOKI7eBmr.png)

3. In the AWS console, navigate to Amazon S3 and upload the data file to your S3 bucket:

    A. Select Buckets from the left panel.

    B. Find and select your S3 bucket.

    C. Click **Upload** and **Add Files**, and upload file watch.csv.

    ![02-upload-s3-watch-csv.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648303450633/_N9IULt5l.png)

4. Next, we'll create a Domain dataset group in Amazon Personalize with the training data. 

    A. Navigate to Amazon Personalize and select **Dataset groups** from the left panel.

    B. Click **Create dataset group**.

    C. For Name, enter **movies**.

    D. For Domain, select **Video on demand**. This matches the schema of our CSV data file.

    E. Click **Create dataset group and continue**.

    ![02-create-dataset-group.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648303862922/hHGRftCLS.png)

5. On the Create interactions dataset page,

    A. For Dataset name, enter **watch**.

    B. For dataset schema, select **Create a new domain schema by modifying the existing default schema for your domain**.

    C. For schema name, enter **movies**.

    D. Click **Create dataset and continue**.

    ![02-create-dataset-group-02.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648304533800/y1YX3-qBr.png)

6. On the Import interactions data page,

    A. For Data import source, select **Import data from S3**.

    B. For Dataset import job name, enter **watch01**.

    C. For Data location, enter **s3://[bucket-name]/watch.csv**, replacing [bucket-name] with the name of your S3 bucket. Ours is `s3://hello-personalize/watch.csv`.

     D. For IAM service role, select **Enter a custom IAM role ARN** and enter the Amazon Resource Name (ARN) of the **personalize-service** role you created earlier, which will have this format: arn:awsiam::[account-id]:role/[role-name].

    E. Click **Finish**.

    ![02-create-dataset-group-03.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648311476279/JLr-jjRLR.png)

    F. You'll see a confirmation that your data is being imported. Refresh the page periodically until you see `Interaction data active`.

    ![02-upload-progress.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648309615233/JoThDB_VV.png)

Amazon Personalize has now imported your movie watch data.

## Step 3: Create a Recommender

In this step, you'll create a *recommender*, which can suggest top picks for a user.

1. In the AWS console, you should still be in Amazon Personalize on the Overview page from the prior step. If not, navigate to Amazon Personalize > Dataset groups > movies > Overview.

2. Select the **Use video on demand recommenders** option and click **Create recommenders**.

    ![03-create-recommender.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648309993441/dUpFzDRtP.png)

3. On the Create recommenders page, you'll see a variety of built-in recommenders for video. We're going to create a "top picks" recommender.

    A. check **Top picks for you** and uncheck the other choices.

    B. For Recommender name, enter **top-picks**.

    ![03-create-recommender-02.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648310168217/HwipzIAQy.png)

    C. Click **Create recommenders**.

    D. On the Recommenders page, watch the status, refreshing the page periodically. Wait for the status to reach `Active`. Creating the recommender may take a few minutes. This is a good time for a bio break.

    ![03-create-recommender-03.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648310303859/wiM0-rMta.png)

    ![03-create-recommender-04.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648312283963/kQPSMadS6.png)

## Step 4: Create a Solution, Event Tracker, and Campaign

After all that configuration and training, we're ready to publish our top picks recommender so we can use it in an application. To do that we'll need to create 1) a solution, 2) a solution version, 3) an event tracker, and 4) a campaign.

1. Create a *solution*:

    A. In the AWS console, navigate to Amazon Personalize > Dataset groups > movies > Overview.

    B. On the middle pane, select the **User custom resources (advanced)** tab.

    C. Click **Create solution**.

    ![05-aws-create-solution.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648318933194/cOA2sm_vL.png)

     D. On the Create solution page, set Solution name: **top-picks**.

     E. Solution type: select **Item recommendation**.

     F. Recipe: select **aws-user-personalization**.

     G. Event type: **watch**.

     H. Click **Create and train solution**

    ![05-aws-create-solution-02.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648319264938/SSxma1gXn.png)

     I. Select Custom resources > Solutions and recipes from the left panel, and wait until your **top-picks** solution status is `Active`.

    ![05-aws-create-solution-03.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648319399459/uC42mXXcw.png)

    ![05-aws-create-solution-04.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648411033667/EmJ7vYMaw.png)

2. Create a *solution version* for your solution:

     A. Click **top-picks** to view the solution detail page.

    ![05-aws-create-sol-version.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648319770922/ms-rxXhj6.png)

     B. Click **Create solution version** and confirm the prompt by clicking **Create solution version**.

    C. Review the Create solution version page and record the Solution ARN and Latest Solution Version ARN. Then click **Create solution version**.

    D. On the top-picks solution page, you see the solution version being created. Wait for solution training and creation to complete, which may take some minutes, You will see a status of `Active` when the solution version is ready. Note your solution version ID. Ours is `1bbe77cf`.

    ![05-aws-create-sol-version-02.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648320930420/dAmnQcw0m.png)

3. Create an *event tracker*:

    A. Navigate to Dataset groups > movies.

    B. On the **Create datasets** panel, click **Create event tracker**.

    ![05-aws-create-event-tracker.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648411924588/JoQosvJU6.png)

    C. On the Configure tracker page, enter Tracker name **watch**.

    ![05-aws-create-event-tracker-02.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648412021103/mA6NZt4-7.png)

    D. Click **Next** and then **Finish**. 

    ![05-aws-create-event-tracker-03.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648412165251/QjL_rurRj.png)

    E. Wait for the tracker status to reach `Active'. **Record the Tracking ID**.

    ![05-aws-create-event-tracker-04.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648412287303/8x5_x6gkA.png)

4. Create a *campaign*. On the left panel, select **Campaigns**.

    A. Click **Create campaign**.

    B. Campaign name: enter **top-picks-01**.

    C. Solution: select **top-picks**.

    D. Solution version ID: select your solution version.

    E. Click **Create campaign**.

    ![05-aws-create-campaign.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648321148366/Hb0z6UH9d.png)

    F. Read the notice, which explains that the way to call this campaign is with the `getRecommendations` API call. We'll be doing in a later step, via the AWS SDK for .NET. **Record the campaign ARN**. Ours is `arn:aws:personalize:us-west-2:[account-number]:campaign/top-picks-01`.

    ![05-aws-create-campaign-02.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648329097981/dyZwGw9PZ.png)

## Step 5: Test the Campaign in the Console

We've now completed all the setup we need to get recommendations from an application, but let's first test our recommendation functionality in the AWS console.

1.  In the AWS console, navigate to Amazon Personalize > Custom resources > Campaigns. 

2. Click the **top-picks-01** campaign we just created in Step 4. 

3. In the Test campaign results panel, enter a User ID of **1**.

4. Click **Get recommendations**.

    ![05-aws-test-campaign.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648325246407/OP5MeZy12.png)

5. You see a response of recommended movie IDs. It's working!

    ![05-aws-test-campaign-1.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648325299574/zppDxtAPg.png)

6. Enter a different User Id of **2** and again click **Get recommendations**. This time you get a different list of top picks, tailored for a different user.

    ![05-aws-test-campaign-2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648325401752/9kbNMOE6X.png)

7. How can we get comfortable about these results? To confirm the value of the recommendations, go back to the movies.csv data file you reviewed earlier in Step 2. The file movies.csv contains the movie item IDs, titles, and genres. The data file we trained Personalize with, watch.csv, references these movies in the ITEM_ID column.

    ![04-movies-csv.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648314213653/GcU7j9EUX.png)

    After correlating the movie IDs in the first run for User 1, we can see we are dealing with a romantic comedy fan who also occasionally watches several other genres. Let's compare that to User 2, who favors hero action movies and science fiction.

| ITEM_ID | Movie Title (User 1) |
| --- | --- |
| 2161 | NeverEnding Story, The (1984) |
| 2797 | Big (1988) |
| 1777 | Wedding Singer, The (1998) |
| 2406 | Romancing the Stone (1984) |
| 1500 | Grosse Pointe Blank (1997) |
| 2395 | Rushmore (1998) |
| 1101 | Top Gun (1986) |
| 1035 | Sound of Music, The (1965) |
| 3253 | Wayne's World (1992) |
| 471 | Close Encounters of the Third Kind (1977) |
| 2700 | South Park: Bigger, Longer and Uncut (1999) |
| 431 | Carlito's Way (1993) |
| 2944 | Dirty Dozen, The (1967) |
| 1663 | Stripes (1981) |

| User 2: ITEM_ID | Movie Title (User 2) |
| --- | --- |
| 110102 | Captain America: The Winter Soldier (2014) |
| 89745 | Avengers, The (2012) |
| 1258 | Shining, The (1980) |
| 84954 | Adjustment Bureau, The (2011) |
| 94864 | Prometheus (2012) |
| 122886 | Star Wars: Episode VII - The Force Awakens (2015) |
| 128360 | The Hateful Eight (2015) |
| 105504 | Captain Phillips (2013) |
| 115149 | John Wick (2014) |
| 60069 | WALL·E (2008) |
| 59784 | Kung Fu Panda (2008) |

## Step 6: Create a .NET Program to Get Recommendations

Now it's finally time to use this in a .NET application. It's beyond the scope of this "Hello, Cloud" to create a full web application, so we're simply going to write a .NET console program and show how to get recommendations for a user as well as report new user events to Amazon Personalize.

1. Open a command/terminal window, and CD to a development folder.

2. Run the dotnet new command below to create a .NET 6 console program named **hello-personalize**.

   ```dos
dotnet new console -n hello-personalize
``` 

     ![05-dotnet-new.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648314527302/HaaZc8qbn.png)

3. Launch Visual Studio and open the hello-personalize project.

4. In the Solution Explorer, right-click the `hello-personalize` project and select **Manage NuGet Packages...**. Search for and install the **AWSSDK.Personalize**, **AWSSDK.PersonalizeEvents**, and **AWSSDK.PersonalizeRuntime** packages.

    ![05-nuget.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648314752086/0BCFzIT5Y.png)

5. Open Program.cs in the code editor and replace it with the code below at the end of this step. Set RegionEndpoint to the region you're using. Replace [campaign-arn] with the Amazon Resource Name of your campaign. Replace [tracking-id]with your Event Tracker ID. You recorded these IDs in Step 4. 

    The code is short and simple. If 1 parameter is on the command line (user ID), recommendations are retrieved for the user from Amazon Personalize via an `AmazonPersonalizeRuntimeClient` and the `GetRecommendationsAsync` method. The `GetRecommendationsRequest` object includes the user Id and the ARN of the campaign (created in Step 4). The response, if successful, contains an `ItemList` collection of recommendations that we iterate through and display to the console.

    If 2 parameters are on the command line (user ID and item ID), the code sends a "user x watched item y" watch event to Amazon Personalize, by instantiating an `AmazonPersonalizeEventsClient` and calling the `PutEventsAsync` method. The `PutEventsRequest` object specifies the user Id, the event, and the tracking ID of the event tracker (created in Step 4).

6. Save your changes and build the program.

Program.cs

```csharp
using Amazon;
using Amazon.PersonalizeRuntime;
using Amazon.PersonalizeRuntime.Model;
using Amazon.PersonalizeEvents;
using Amazon.PersonalizeEvents.Model;
using System.Net;

RegionEndpoint region = RegionEndpoint.USWest2;
const string CAMPAIGN_ARN = "[campaign-arn]";
const string TRACKING_ID = "[tracking-id]";

if (args.Length == 1)
{
    // Get recommendations

    var userId = args[0];

    var client = new AmazonPersonalizeRuntimeClient(region);

    Console.WriteLine($"Getting recommendations for user {userId}...");

    var request = new GetRecommendationsRequest()
    {
        CampaignArn = CAMPAIGN_ARN,
        UserId = userId
    };

    var response = await client.GetRecommendationsAsync(request);

    if (response.HttpStatusCode != HttpStatusCode.OK)
    {
        Console.WriteLine($"Error: AmazonPersonalizeRuntimeClient.GetRecommendationsAsync returned status {response.HttpStatusCode}");
    }
    else
    {
        if (response.ItemList != null && response.ItemList.Count > 0)
        {
            Console.WriteLine($"Recommendations:");
            foreach (var recommendation in response.ItemList)
            {
                Console.WriteLine(recommendation.ItemId);
            }
        }
        else
        {
            Console.WriteLine($"No recommendations");
        }
    }
    Environment.Exit(0);
}

if (args.Length == 2)
{
    // Report watch event

    var userId = args[0];
    var itemId = args[1];
    Console.WriteLine($"Reporting watch event to Amazon Personalize - user {userId} watched item {itemId}");
 
    var eventClient = new AmazonPersonalizeEventsClient(region);
    var eventRequest = new PutEventsRequest()
    {
        EventList = new List<Event>(new Event[]
                {  new Event()
                    {
                        EventId = Guid.NewGuid().ToString(),
                        EventType = "watch",
                        ItemId = itemId,
                        SentAt = DateTime.Now
                    }
                }),
        SessionId = Guid.NewGuid().ToString(),
        TrackingId = TRACKING_ID,
        UserId = userId
    };
    var resp = await eventClient.PutEventsAsync(eventRequest);
    Environment.Exit(0);
}

Console.WriteLine("To report a watch event: dotnet run -- [user-id] [event-id]");
Console.WriteLine("Example:                 dotnet run -- 1 1503");
Console.WriteLine("To get recommendations:  dotnet run -- [user-id]");
Console.WriteLine("Example:                 dotnet run -- 1");
```

## Step 7: Test the Program

In this step, you'll test your program's ability to get recommendations for a user, and to update the interaction dataset in real time with new watch events.

1. In a command/terminal window, CD to your project folder and run your program with the command line below.

   ```dos
dotnet run -- 1
```
    You should get recommendations for User ID 1 that match what you did back in Step 5 in the AWS Console. 

    ![06-dotnetrun-1.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648322677832/OMGRU4gYL.png)

2. Run the command line, changing the parameter to other User ID values.

    ![06-dotnetrun-2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648413553032/WK-5g-4HS.png)

   Take a bow! You've trained and configured Amazon Personalize for movie recommendations, and retrieved real-time recommendations in a .NET application.

3. Now, run the command line with user 777, who does not exist in the interaction data set. You get a list of broadly popular recommendations, such as "Dances with Wolves", "The Shawshank Redemption", and "Star Wars" movies, not based on any historical data for this user. If you try a different new user ID, you'll get the same list.

   ```dos
dotnet run -- 777
```

    ![06-dotnetrun-777.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648417467070/xkc4vsZ_Z.png)

4. Now, let's build a history for user 777 by sending events to Amazon Personalize. Run each of the commands below to build up a watch list of sci-fi movies for user 777.

   ```dos
dotnet run -- 777 1253
dotnet run -- 777 5410
dotnet run -- 777 70599
dotnet run -- 777 1301
dotnet run -- 777 113378
dotnet run -- 777 5468
```

    ![06-dotnetrun-777-watch-events.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648414553872/0eF3bfQh6.png)

5. Now, again get recommendations for user 777. 

   ```dos
dotnet run -- 777
```

    ![06-dotnetrun-777-02.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648414664648/hLpJqJGTk.png)

    This time, the recommendations are different, because our watch events have given the recommendations engine an idea of our user's tastes. The list now includes many sci-fi movies, including WALL-E, The Hunger Games, Hitchhiker's Guide to the Galaxy, Star Wars: Episode IV, and The Adjustment Bureau.

Congratulations! You've used Amazon Personalize to get movie recommendations for users, and to report real-time user watch events.

## Step 8: Shut it Down

When you're all finished with the Hello, Personalize project, deallocate your AWS resources. You don't to be charged for something you're not using.

1. Delete your campaign:

    A. Navigate to Amazon Personalize > Dataset groups > movies > Campaigns. 

    B. Click the **top-picks-01** campaign.

    C. Click the **Delete** button at top right. Confirm the prompt and wait for the campaign to be deleted.

2. Delete your solution:

    A. Navigate to Amazon Personalize > Dataset groups > movies > Solutions and recipes. 

    B. Click the **top-picks** solution.

    C. Click the **Delete** button at top right. Confirm the prompt and wait for the solution to be deleted.

3. Delete your dataset:

    A. Navigate to Amazon Personalize > Dataset groups > movies > datasets. 

    B. Click on the **watch** dataset. 

    C. Click **Delete** at top right. Confirm the prompt and wait for your dataset to be deleted.

# Where to Go From Here

Personalized service is vital for many businesses, and with Amazon Personalize you can build a high quality personalized recommendation system. In this tutorial, you used sample movie viewing data to train a personalized movie recommendation system. You wrote .NET code that gets recommendations for users from Amazon Personalize. You also wrote code to send user watch events to Amazon Personalize to keep the interaction dataset up to date. The AWS SDK for .NET code was minimal: once things were configured, integration into application code was very simple. We only scratched the surface of what's possible.

In the AWS console, you had to work with quite a few different artifacts, including dataset groups, recipes, solutions, solution versions, campaigns, and event trackers. Be aware it is also possible to perform these operations from the AWS CLI or in code with the AWS SDK. Many of the artifacts you created offered you configuration choices to fine-tune behavior. To move forward with Amazon Personalize, spend time in it and review the documentation deeply. Learn its many features. You'll need to get a firm understanding of these artifacts and the control you have over them, as well as the workflow for iteratively improving your interaction data, models, and results.

# Further Reading

AWS Documentation

[Amazon Personalize](https://aws.amazon.com/personalize)

[Amazon Personalize - Getting Started](https://docs.aws.amazon.com/personalize/latest/dg/getting-started.html)

[Amazon Personalize - How it works](https://docs.aws.amazon.com/personalize/latest/dg/how-it-works.html)

[Amazon Personalize - Resources](https://aws.amazon.com/personalize/resources/)

[Developer Guide](https://docs.aws.amazon.com/personalize/latest/dg/index.html)

[Recording Events](https://docs.aws.amazon.com/personalize/latest/dg/recording-events.html)

[Tutorial: Create real-time, personalized movie recommendations with Amazon Personalize](https://aws.amazon.com/getting-started/hands-on/real-time-movie-recommendations-amazon-personalize/?trk=gs_card) (Python)

[AWS SDK for .NET Documentation](https://docs.aws.amazon.com/sdk-for-net/index.html)

Videos

[How to Build This: Episode 1 - Personalization](https://www.youtube.com/watch?v=bx0V7iiH09M) by Jillian Forde

Blogs

[Creating a Recommendation Engine using Amazon Personalize](https://aws.amazon.com/blogs/machine-learning/creating-a-recommendation-engine-using-amazon-personalize/)

[Amazon Personalize improvements reduce training time and latency](https://aws.amazon.com/blogs/machine-learning/amazon-personalize-improvements-reduce-model-training-time-by-up-to-40-and-latency-for-generating-recommendations-by-up-to-30/)

[Hello, Cloud blog series](https://davidpallmann.hashnode.dev/series/hello-cloud)