## Hello, DynamoDB! (Document Model)

#### This episode: Amazon DynamoDB and the document model. In this Hello, Cloud blog series, we're covering the basics of AWS cloud services for newcomers who are .NET developers. If you love C# but are new to AWS, or to this particular service, this should give you a jumpstart.

In this post we'll explain NoSQL databases, introduce DynamoDB, and use it in a "Hello, Cloud" .NET program to store and retrieve JSON documents. We'll do this step-by-step, making no assumptions other than familiarity with C# and Visual Studio. We're using Visual Studio 2022 and .NET 6.

# The Rise of NoSQL Databases

> “Different databases are designed to solve different problems. Using a single database engine for all of the requirements usually leads to non-performant solutions; storing transactional data, caching session information, traversing graph of customers and the products their friends bought are essentially different problems.”
—Pramod J. Sadalage, NoSQL Distilled: A Brief Guide to the Emerging World of Polyglot Persistence

Relational databases have been around since the 1970s. They store entities in tables, normalized to minimize redundancy, and they accommodate querying data in many different ways. With a well-designed database schema, Structured Query Language (SQL) is terrific for analyzing data and discovering new insights. Many developers are very comfortable with relational databases and SQL, so where did this upstart NoSQL come from? In the 21st century, the limitations of relational databases to scale and perform adequately for large web applications became increasingly apparent. Relational databases don't scale well horizontally, and can't deliver the throughput needed by some modern online applications. As great as relational databases are, they are not well-suited to all uses. Instead of a Swiss army knife for all database needs, what was needed were [purpose-built databases](https://aws.amazon.com/getting-started/hands-on/purpose-built-databases/) that support specific access patterns exceedingly well. 

![diagram-sql-vs-nosql.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648986309578/bh27VN-Gt.png)

The motivations for NoSQL database are design simplicity, horizontal scaling to clusters of machines, and high throughput. In achieving this, they give up some things from the relational database world, notably a rich query language and transactions. That can be a big adjustment for developers used to relational databases, but NoSQL databases deliver on their promise. [Amazon famously migrated its consumer business from Oracle to DynamoDB](https://aws.amazon.com/blogs/aws/migration-complete-amazons-consumer-business-just-turned-off-its-final-oracle-database/) and several other AWS purpose-built databases (75 petabytes of data in 7,500 databases). 

| SQL | NoSQL |
| --- | ------ |
| Optimized for storage | Optimized for compute |
| Normalized/relational | Denormalized/hierarchical |
| Ad hoc queries | Instantiated views |
| Scale vertically | Scale horizontally |
| ACID transactions | Eventual consistency |
| Good for OLAP | Built for OLTP at scale |

None of this means the extinction of relational databases, though they've been sidelined a bit. When you need ad-hoc analytics, relational databases are hard to beat. If you recall the traditional database terminology of Online Analytical Processing (OLAP) and Online Transaction Processing (OLTP), relational databases are for OLAP and NoSQL databases are for OLTP.

Since they are purpose-built, NoSQL databases offer a variety of data models that include key-value, documents, wide column and graphs:

| Type | Description | Use cases | Benefits |
| ---- | --------- | --------- | --------- |
| document | stores data in JSON (or BSON or XML) documents | e-commerce, trading, mobile apps | easily changed over time |
| key-value | stores keys and values | shopping carts, settings, user profiles | simple |
| wide column | stores data in columns rather than rows | data warehouses, financial | fast querying & joins |
| graph | stores data as nodes and relationships | social networks, fraud detection, knowledge graphs | data access by connections | 

Working with NoSQL databases forces you to think deeply about the access patterns for your data, and that generally results in a simpler design and better experience for users. The key take-away from our current rich set of database choices is to use the right database for the job at hand.

# DynamoDB : What is it, and why use It?

[Amazon DynamoDB](https://aws.amazon.com/dynamodb/) (hereafter "DynamoDB") is a NoSQL database service with performance and scalability at its core. AWS describes it as "a fully managed, serverless, key-value NoSQL database designed to run high-performance applications at any scale." Since DynamoDB is a managed service, you don't have to allocate servers or install software. You simply create tables and start using them.  DynamoDB gives you predictable performance at any scale with 5 9's (99.999%) of availability. You can create globally distributed databases with it, and replicate tables across multiple regions with low latency. 

Unlike many databases where you have to over-provision capacity to handle anticipated peak load, DynamoDB can auto-scale and match provisioning to usage. DynamoDB offers two [capacity modes](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.ReadWriteCapacityMode.html), **provisioned** and **on-demand**. With Provisioned mode, you specify your throughput requirements in terms of Write Capacity Units (WCUs) and Read Capacity Units (RCUs). You can use auto-scaling to adjust that provisioned capacity as traffic changes, but it's not immediate so this choice is best when you can reliably anticipate your patterns of traffic. In On-demand mode, DynamoDB *instantly* accommodates changes in traffic, for a higher rate.

You can work with DynamoDB as a document store or a key-value store. In this tutorial, we'll be using the document store model. The AWS SDK for .NET offers several APIs for working with DynamoDB, including a Document API we'll be using today.

The [AWS free tier](https://aws.amazon.com/dynamodb/pricing/provisioned/) gives you 25 WCUs and 25 RCUs of provisioned capacity and 25GB of table data storage per month. Read through the details and limitations.

## Concepts

A **table** is a collection of data, as is true of most databases. For example, you might store information about recipes in a table named `recipe`.

Tables contain **items**. An item is a group of attributes that can be uniquely identified by a primary key. Items are similar to rows or records in other databases, but DynamoDB is schemaless: there's no requirement that items in a table contain the same attributes. That gives you a lot flexibility in table design.

An **attribute** is a data element. For example, a recipe item might contain a food category, a recipe name, the number of servings, and preparation time. DynamoDB allows nested attributes. For example, in a recipe item you might have a collection of ingredients and a collection of instruction steps.

![diagram-table-recipe.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1649018007282/ubDng8wed.png)

DynamoDB is a distributed database, and stores subsets of table data in **partitions** which is essential to horizontal scale. A **partition key** is an attribute that determines which partition an item is stored in and is a critical decision in table design. Ideally the partition key will have many values so that data is evenly distributed. 

A **sort key** is an additional key that can contain an attribute or a combination of attributes. Your choice of sort key is also a critical decision in table design that impacts the different access patterns available. For example, to borrow an example from the AWS documentation, in a table of addresses you could store country#region#state#county#neighborhood in the sort key, which would allow you to query by country all the way to neighborhood and everything in between.

An item needs a **primary key**, which can be 1) a *simple primary key* consisting of just the partition key; or 2) a *composite primary key* composed of the partition key and the sort key. You can retrieve an item if you know its primary key.

Although there's no SQL, you can retrieve collections of items from a DynamoDB in two ways. A **scan** iterates through an entire table, looking for elements that match criteria. A **query** similarly looks for elements that match criteria, but uses keys and is much faster. You should use a query when possible.

# Our Hello, DynamoDB Project

We will use DynamoDB to store and retrieve recipes in the form of JSON documents. We will first create a DynamoDB table in the AWS console. Then, we'll write a .NET program that can put, list, and get documents using the AWS SDK for .NET Document API for DynamoDB.

![00-churros-json.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648931736677/4dvZJ6evf.png)

![03-dotnet-run-put.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648931152439/ojxfBcIdh.png)

![03-dotnet-run-get-churros.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648931602042/cWw_A9R0_.png)

[source code](https://github.com/davidpallmann/hello-dynamo)

## One-time Setup

For any of the tutorials in the Hello, Cloud series you need the following:

1. An  [AWS account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/), and an understanding of what is included in the  [AWS Free Tier](https://aws.amazon.com/premiumsupport/knowledge-center/what-is-free-tier/). 
2. [Microsoft Visual Studio 2022](https://visualstudio.microsoft.com/). If you're using an older version of Visual Studio you won't be able to use .NET 6. If you use a different IDE, you may have to find alternatives to some tutorial steps.
3. [AWS Toolkit for Visual Studio](https://aws.amazon.com/visualstudio/). You'll need to [configure the toolkit ](https://docs.aws.amazon.com/toolkit-for-visual-studio/latest/user-guide/keys-profiles-credentials.html) to access your AWS account and create an IAM user. Your default AWS profile will be linked to this user when running programs from the command line.

## Step 1: Create a Table

In this step, you'll create a DynamoDB table in the AWS console. We'll be creating a table for holding recipes. We'll use food category (e.g, Italian, French, Thai) for our partition key. We'll use recipe name for our sort key. This design will allow us to easily retrieve or update a single recipe, or list all recipes in a food category.

1. Sign in to the [AWS management console](https://aws.amazon.com/console/). At top right, select the region you want to work in. You can check supported regions for DynamoDB on the [Amazon DynamoDB endpoints and quotas](https://docs.aws.amazon.com/general/latest/gr/ddb.html) page. I'm using **us-west-2 (Oregon)**.

2. Navigate to **Amazon DynamoDB**. You can enter **dynamo** in the search bar.

3. Select **Tables**  from the left panel and click **Create table**.

    A. Table name: **recipe**.

    B. Partition key: **category**.

    C. Sort key: **name**.

    D. Leave other settings defaulted, but review them to get familiar with your options.

    ![01-aws-create-table.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648916322792/45pu17Fzr.png)

    E. Click **Create table** and wait for it to be created.

    ![01-aws-create-table-02.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648916477595/MpCd7XZkp.png)

4. Click **recipe** to see the table detail. Take a moment to explore the tabs.

## Step 2: Create a .NET Program for Document Storage and Retrieval

In this step, you'll create a .NET console program and write code to write and read documents to/from DynamoDB using the AWS SDK for .NET DynamoDB document API.

1. Open a command/terminal window and CD to a development folder.

2. Run the `dotnet new` command below to create a new console application project named hello-dynamo.

    ```dos
dotnet new console -n hello-dynamo
```

    ![02-dotnet-new.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648916821670/CArfn6psj.png)

3. Launch Visual Studio and open the project.

4. In Visual Studio Solution Explorer, right-click the hello-dynamo project and select **Manage NuGet Packages...**. Find and install the **AWSSDK.DynamoDBv2** and **System.IO** packages.

    ![02-nuget.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648917113843/nIxMcyZqU.png)

5. Open Program.cs in the code editor and replace it with the code below at the end of this step. Set `region` to the region you used in Step 1.

6. Save your changes and build the program.

7. Create some test data, by creating several JSON files with the contents listed at the end of this step. Store these in a `data` subfolder under your project folder.

Program.cs

```csharp
using Amazon;
using Amazon.DynamoDBv2;
using Amazon.DynamoDBv2.DocumentModel;

RegionEndpoint region = RegionEndpoint.USWest2;

var client = new AmazonDynamoDBClient(region);
Table table = Table.LoadTable(client, "recipe");

if (args.Length == 2 && args[0] == "put")
{
    // put [json-filename] - store a recipe
    var name = args[1];
    var json = File.ReadAllText(name);
    var recipe = Document.FromJson(json);
    Console.WriteLine($"Putting document {recipe["name"]} to recipe table");
    var response = await table.PutItemAsync(recipe);
}
else if (args.Length == 3 && args[0] == "get")
{
    // get [category] [name] - retrieve and display a recipe
    var partitionKey = args[1];
    var sortKey = args[2];
    var recipe = await table.GetItemAsync(partitionKey, sortKey);
    if (recipe == null)
    {
        Console.WriteLine("Recipe not found");
    }
    else
    {
        Console.WriteLine($"Recipe: {recipe["name"]}");
        Console.WriteLine($"Category: {recipe["category"]}");
        Console.WriteLine($"Link: {recipe["link"]}");

        Console.WriteLine($"Intro: {recipe["intro"]}");
        Console.WriteLine();

        Console.WriteLine("Ingredients");
        foreach (var ingredient in recipe["ingredients"].AsListOfDynamoDBEntry())
        {
            Console.WriteLine($"{ingredient}");
        }
        Console.WriteLine();

        Console.WriteLine("Instructions");
        int number = 1;
        foreach (var step in recipe["instructions"].AsListOfDynamoDBEntry())
        {
            Console.WriteLine($"{number++}. {step}");
        }
    }
}
else if (args.Length==1 && args[0] == "list")
{
    // list - list recipes
    var filter = new ScanFilter();
    filter.AddCondition("name", ScanOperator.IsNotNull);

    var scanConfig = new ScanOperationConfig()
    {
        Filter = filter,
        Select = SelectValues.SpecificAttributes,
        AttributesToGet = new List<string> { "category", "name", "link" }
    };
    Search search = table.Scan(scanConfig);
    List<Document> matches;
    do
    {
        matches = await search.GetNextSetAsync();
        foreach (var match in matches)
        {
            Console.WriteLine($"{match["category"],-20} {match["name"],-40} {match["link"]}");
        }
    } while (!search.IsDone);
}
else if (args.Length==2 && args[0] == "list")
{
    // list [category] - list recipes in a category
    var filter = new QueryFilter();
    filter.AddCondition("category", ScanOperator.Equal, args[1]);

    var filterConfig = new QueryOperationConfig()
    {
        Filter = filter,
        Select = SelectValues.SpecificAttributes,
        AttributesToGet = new List<string> { "category", "name", "link" }
    };
    Search search = table.Query(filterConfig);
    List<Document> matches;
    do
    {
        matches = await search.GetNextSetAsync();
        foreach (var match in matches)
        {
            Console.WriteLine($"{match["category"],-20} {match["name"],-40} {match["link"]}");
        }
    } while (!search.IsDone);
}
else
{
    Console.WriteLine("To store a recipe:              dotnet run -- put [jsonfile]");
    Console.WriteLine("To list all recipes:            dotnet run -- list");
    Console.WriteLine("To list recipes in a category:  dotnet run -- list [category]");
    Console.WriteLine("To retrieve a recipe:           dotnet run -- get [category] [name]");
}
```


churros.json

```json
{
  "category": "Spanish",
  "name": "Churros",
  "link": "https://goodcookbecky.wordpress.com/page/2/?x=",
  "intro": "Inspired by a photo of my daughter at Disneyland eating a nice big churro, I wanted to make them at home.  It could not be that hard right?",
  "ingredients": [
    "2 cups flour"
    "2 Tablespoons unsalted butter"
    "2 Tablespoons sugar"
    "1 teaspoon vanilla extract"
    "2 cups all-purpose flour"
    "2 large eggs"
    "2 quarts vegetable oil"
    "Coating:"
    "1/2 cup sugar"
    "3/4 teaspoon cinnamon"
  ],
  "instructions": [
    "Line a rimmed baking sheet with parchment paper and spray liberally with a cooking spray.",
    "In a medium pot, over medium-high heat, bring water, butter, salt, butter, and vanilla to a boil. Turn off the burner and pull the pot off heat. Add the 2 cups of flour and use a rubber spatula to combine the flour until there are no more white streaks of flour. Transfer the dough to a stand mixer and beat on low for one to two minutes.  Add the eggs one at a time, mixing until it is combined before adding the second egg.  Increase speed to medium and beat for one minute.",
    "Fill the piping bag with half of the batter and pipe 6 long logs on the  parchment lined pan, cutting the ends with kitchen scissors. Cook’s Country magazine had a step to refrigerate the dough for 15 minutes to an hour, but I think they fry up better when the dough is at room temperature.",
    "Preheat 2 inches of oil in a heavy skillet to 375 to 400 degrees Fahrenheit.",
    "Fry 3 to 6 at a time in the oil. You may need to use tongs to separate them as they fry so they don’t get stuck together.  Fry them about two to three minutes on each side before removing them to a paper towel lined plate to drain some of the fat off. Repeat with remaining churros.",
    "In a glass pie dish or large shallow bowl, combine the half cup of sugar and 3/4 teaspoon of cinnamon.  Once the fried churros are cool enough to handle toss them in the cinnamon sugar and enjoy!"
  ]
}
```

chowder.json

```json
{
  "category": "Spanish",
  "name": "Roasted Corn and Poblano Chowder",
  "link": "https://goodcookbecky.wordpress.com/2017/10/10/roasted-corn-and-poblano-chowder/",
  "intro": "Well, my blog has been seriously neglected lately.  I have quite a few recipes that I wanted to blog, but homeschooling has kept me quite busy.  Here is a recent recipe though, that I really wanted to get on my page.  As we move from Summer to Fall, I always enjoy getting soups into my menu planning more frequently.",
  "ingredients": [
    "3 poblano peppers, stemmed, halved from stem to tip, seeds removed",
    "1 Tbsp vegetable oil",
    "6 ears fresh corn, kernels cut from cobs",
    "salt and pepper to taste",
    "4 slices bacon, chopped fine",
    "1 onion, chopped",
    "2 cloves garlic, minced",
    "7 cups chicken broth",
    "1 lb Yukon potatoes, peeled and diced into 1/2 inch dice",
    "1/4 cup half and half",
    "2 (6 inch) corn tortillas, cut into 1″ pieces",
    "1 Tbsp minced cilantro",
    "1 Tbsp lime juice",
    "additionally:",
    "lime wedges",
    "reserved bacon bits",
    "sour cream for serving, optional",
    "fried tortilla strips",
    "crumbled queso fresco"
  ],
  "instructions": [
    "Adjust the oven rack about 6 inches from the broiler element in your oven.  Line a cookie sheet with foil, rub poblano peppers with oil and place skin side up on your cookie sheet.  Broil for 10 to 15 minutes until the skin of the peppers have blistered and browned, turning with tongs from time to time.  Remove the peppers to a bowl immediately and cover with plastic wrap to steam the skins for easy removal.",
    "Place the corn kernels on the same pan you used for the peppers and season with a teaspoon of salt and add 1 teaspoon of oil, stirring the corn. Broil them for 10-15 minutes, stirring from time to time and until the corn is browned and tender.",
    "Remove the skins from the poblano peppers and chop the peppers.",
    "In a large stock pot, cook the bacon over medium heat until crispy.  Remove bacon with a slotted spoon and reserve for serving.  Add onion, 1/4 teaspoon salt to the bacon drippings.  Cook the onion, stirring frequently until the onion is tender and browned.  Add garlic and cook for 30 seconds, until fragrant.  Drain any excess bacon drippings.",
    "Add chicken broth, potatoes, corn kernels, and 1/2 teaspoon salt.  Stir the soup, scraping up any browned bits from the bottom of the pot.  Bring to a boil and simmer just under a boil for 20 minutes.  Test the potatoes to see if they are done – cook longer if necessary.  Off heat, add half and half.  Scoop out about 2 cups of soup to a large measuring cup along with the 2 corn tortillas that you have cut up;  use your immersion blender to blend the soup until smooth.  (Use a blender if you do not have an immersion blender)  Return the puree to the soup and add the chopped poblano peppers to the chowder.  Bring to a slight boil.  Add minced cilantro, lime juice, 3/4 teaspoon salt if necessary.",
    "Serve with crisp bacon bits, additional cilantro, lemon wedges, sour cream and grated queso fresco if desired.",
    "To make tortilla strips, cut some remaining corn tortillas into strips and cook them in a small frying pan in small batches and in hot oil, until the tortillas are golden brown and crispy.  Remove with a slotted spoon and season with salt immediately, draining on paper towels.  Serve them with the chowder."
  ]
}
```

meatballs.json

```JSON
{
  "category": "Italian",
  "name": "Italian Meatballs",
  "link": "https://goodcookbecky.wordpress.com/2016/11/01/italian-meatballs/",
  "intro": "I have recently made a few new recipes.  Two of them by Bobby Flay – and both of them outstanding! This one is for Spaghetti and Sauce.  We LOVED the meatballs!  I will be making them again in the future – maybe even in bulk to freeze them for the future.  I did  not use veal as it is sometimes hard to find.  I doubled the recipe because we had more than 4 people eating dinner… Now I wish I had made even more!",
  "ingredients": [
    "2 lbs ground beef",
    "1 lb ground pork",
    "4 large eggs",
    "1/2 cup grated Parmesan cheese",
    "8 cloves garlic, finely minced and sauteed",
    "1/2 cup dry bread crumbs",
    "1/2 cup finely chopped parsley",
    "2 tsp salt",
    "1/2 tsp pepper",
    "1 cup olive oil/canola oil"
  ],
  "instructions": [
    "In a large bowl, combine ground beef, ground pork, eggs, grated Parmesan, cooked garlic, dry bread crumbs, minced fresh parsley, salt, and pepper.  Use two large serving forks to break up the meat and mix all the ingredients until it is evenly seasoned.  Do not overwork it as it will become tough.  Form 1 to 1 1/2 inch meatballs with your hands.",
    "Heat a non stick skillet with 1 cup of cooking oil over medium high heat.  Cook the meatballs in batches, turning them over with tongs and continue cooking until golden brown. You do not need to cook them through entirely, as you simmer them in the sauce.  Drain the meatballs on a wire rack for a few minutes while you finish cooking the remaining meatballs."
  ]
}
```

## Step 3: Test Your Program

In this step you'll run your program to store, list, and retrieve recipes.

1. In your command/terminal window, run each of the commands below to store the Churros, Chowder, and Meatballs recipes in your recipe table:

    ```dos
dotnet run -- data\churros.json
dotnet run -- data\chowder.json
dotnet run -- data\meatballs.json
```

    ![03-dotnet-run-put.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648929376127/kVqJlvGvX.png)

    As the recipes are stored, you see displayed the category and name, which are the partition key and sort key they are stored under, which are specified in the JSON.

2. In the AWS console, click **Explore items** in the left panel. You should see 3 items listed for the recipes you just stored, 2 in category Spanish and one in category Italian. Your program's put actions were successful!

    ![03-explore-items.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648924220386/qHN7OSyOL.png)

3. Back in the command window, run the command below to list your recipes. You should see 3 displayed. This uses a scan operation to return all documents.

    ```dos
dotnet run -- list
```

    ![03-dotnet-run-lists.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648925072847/fVkwO3MKd.png)

4. In the command window, run the program again with the commands below to retrieve your recipes from DynamoDB and display them. We are specifying on the command line our desired action (get) and the partition key and sort key we want to retrieve.

    ```dos
dotnet run -- get Spanish Churros
dotnet run -- get Spanish "Roasted Corn and Poblano Chowder"
dotnet run -- get Italian Meatballs
```

    You should see each recipe listed as shown below. We've now confirmed we can retrieve what we stored.

    ![03-dotnet-run-get-churros.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648931587417/S9duef8On.png)

    ![03-dotnet-run-get-chowder.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648924575875/m6U2-11kE.png)

    ![03-dotnet-run-get-meatballs.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648924940948/8pMq-HIWR.png)

5. Lastly, run the commands below to perform queries on the data. Unlike the scan in #4 earlier, this is a query that uses the partition key.

    ```dos
dotnet run -- list Spanish
dotnet run -- list Italian
```

    ![03-dotnet-run-list-category.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1648925549508/OvZiAwu53.png)

Congratulations! You've performed basic document storage, retrieval, scans, and queries from .NET code using DynamoDB's document data model.

### Understand the Code

Now that we've seen it work, let's understand the code. The Document API we're using is the simplest way to work with DynamoDB, if the document data model is appropriate for your use case. 

Line 7: The code begins by instantiating an `AmazonDynamoDBClient` client and then a `Table` object. The rest of our code does everything with the Table object, `table`.

Line 13: The 'put' action loads the JSON from a filename specified on the command line, and creates a `Document` object from it with the `Document.FromJson` method. The `PutItemAsync` method stores the object. What keys will the object be stored under? When we created the DynamoDB table earlier in the AWS console, we told it the name of the partition key (category) and the sort key (name). Those attributes in the JSON will determine how the document is indexed.

Line 21: The 'get' action calls the `GetItemAsync` method to retrieve a document, passing the partition key and sort key specified on the command line. If found, we get a DynamoDB Document object. We retrieve and display elements from the Document object by key name. When we get to the ingredients and instructions parts, those are DynamoDBList objects in the Document, which we convert to a List<DynamoDBEntry> with the `.AsListofDynamoDBEntry` method.

Line 55: to list all recipes, we use a scan, a brute force way to list all records. That requires us to specify a filter, but we want all records so our filter condition is simply name is not null. We don't need all attributes back in the results, so we specify just the attributes we plan to display. A `Scan` method starts the scan and returns a `Search` object. To get results, we loop and perform one or more `GetNextSetAsync` calls on the search object until the search is complete, which provides a list of matching documents.

Line 78: To list the recipes belonging to a category, we use a query instead of a scan. Our filter is category equal to the value specified on the command line. We use the `Query` method to start the query, which returns a `Search` object, and retrieving the results is the same as what we did for the scan earlier.

Line 101: if none of the expected command line forms was found, help is displayed.

This simple Hello, Cloud project did not provide exception handling, but your code should anticipate that the SDK will throw exceptions in the event of an error. 

## Step 4: Shut it Down

When you're done working with the Hello, DynamoDB project, shut it down. You don't want to be charged for something you're not using.

1. In the AWS console, navigate to Amazon DynamoDB > Tables.

2. Check the checkbox next to **recipe**.

3. Click **Delete** at top right and confirm the prompt to delete the table.

# Where to Go From Here

DynamoDB is a 21st century NoSQL database you can rely on for applications of all sizes, even mammoth, mission-critical global applications. Whether your use case is large or small, DynamoDB delivers predictable performance and high availability that scales effortlessly. 

In this tutorial, you used DynamoDB in the simplest way possible, with the document data model. You stored and retrieved JSON recipe documents in DynamoDB using the AWS SDK for .NET Document API. You performed both scan and query operations to list recipes. In a real application, you would need to handle errors and set a configuration with a retry policy for the DynamoDB client. You should also make use of batch methods whenever possible in order to minimize I/O. 

There are DynamoDB features we did not mention today that you'll want to understand, including secondary indexes and DynamoDB streams. As you learn and experiment with DynamoDB, make sure you get a firm grounding on partition keys and sort keys. Our example used a simple sort key, the recipe name. You can do elaborate things with compound sort keys to support a variety of queries. The re:Invent videos linked below are excellent for understanding DynamoDB foundation concepts, best practices, design patterns, and advanced techniques. 

# Further Reading

AWS Documentation

[Amazon DynamoDB](https://aws.amazon.com/dynamodb)

[DynamoDB Pricing](https://aws.amazon.com/dynamodb/pricing)

[Purpose-build Databases](https://aws.amazon.com/getting-started/hands-on/purpose-built-databases/)

[NoSQL Design for DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-general-nosql-design.html)

[Overview of AWS SDK Support for DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Programming.SDKOverview.html)

[Getting Started with .NET and DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GettingStarted.NET.html)

[Best Practices for Designing and Architecting with DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/best-practices.html)

[Best Practices for Designing and Using Partition Keys Effectively](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-partition-key-design.html)

[Best Practices for Using Sort Keys to Organize Data](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-sort-keys.html)

[Working with Items in DynamoDB Using the AWS SDK for .NET Document Model](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/WorkingWithItemsDocumentClasses.html)

[AWS SDK for .NET - Retries and Timeouts](https://docs.aws.amazon.com/sdk-for-net/v3/developer-guide/retries-timeouts.html)

[AWS SDK for .NET Documentation](https://docs.aws.amazon.com/sdk-for-net/index.html)

Videos

[Data modeling with Amazon DynamoDB - Part 1](https://www.youtube.com/watch?v=fiP2e-g-r4g)

[Data modeling with Amazon DynamoDB - Part 2](https://www.youtube.com/watch?v=0uLF1tjI_BI)

[Amazon DynamoDB advanced design patterns - Part 1](https://www.youtube.com/watch?v=MF9a1UNOAQo) by Rick Houlihan

[Amazon DynamoDB advanced design patterns - Part 1](https://www.youtube.com/watch?v=_KNrRdWD25M) by Rick Houlihan

[DynamoDB deep dive: Advanced design patterns](https://www.youtube.com/watch?v=xfxBhvGpoa0) AWS re:Invent 2021 session by Rick Houlihan

Blogs

[Migration Complete - Amazon's Consumer Business Just Turned Off Its Final Oracle Database](https://aws.amazon.com/blogs/aws/migration-complete-amazons-consumer-business-just-turned-off-its-final-oracle-database/)

[Everything you need to know about DynamoDB Partitions](https://www.alexdebrie.com/posts/dynamodb-partitions/)

[DynamoDBGuide.com](https://www.dynamodbguide.com/)

[Hello, Cloud blog series](https://davidpallmann.hashnode.dev/series/hello-cloud)
