---
title: "Hello, Porting Assistant for .NET"
datePublished: Sat Mar 25 2023 16:18:38 GMT+0000 (Coordinated Universal Time)
cuid: clfo6ee0h000b0aiiepyy5vku
slug: hello-porting-assistant-for-net
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1679750618380/ccc06fe2-171c-4fb1-982c-d23e4a8ad67f.png
tags: aws, net, dotnet

---

#### This episode: Porting Assistant for .NET and modernizing .NET Framework applications. In this Hello, Cloud blog series, we're covering the basics of AWS cloud for newcomers who are .NET developers. If you love C# but are new to AWS, or to this particular tool, this should give you a jumpstart.

In this post we'll introduce Porting Assistant for .NET and use it to migrate a .NET Framework application to modern dotnet (formerly called .NET Core). We'll do this step-by-step, making no assumptions other than familiarity with C# and Visual Studio. We're using Visual Studio 2022 and .NET 6.

# Porting Assistant : What is it, and why use It?

> "To reach a port we must sail, sometimes with the wind, and sometimes against it. But we must not drift or lie at anchor." ―[Oliver Wendell Holmes, Sr.](https://www.brainyquote.com/authors/oliver-wendell-holmes-sr-quotes)

It is commonly estimated that 75% of enterprise appilcations are .NET based, and a great many of those are legacy applications on the .NET Framework. It's been more than 8 years since cross-platform .NET (originally called .NET Core) was announced in 2014, but you may still have legacy .NET framework applications running. There are 2 good reasons to port your legacy code to modern dotnet. First, the legacy .NET Framework is no longer advancing, while modern dotnet is alive and well, with new features and performance improvements every year. Second, you can reduce your costs on modern dotnet by running on Linux.

Those are good benefits, but how do you weigh them against the level of effort in porting? You might find yourself waist-deep in porting issues, such as dependencies on packages and APIs that aren't compatible with modern dotnet. How can you make porting easier? Happily, there is help available―and it comes at no charge.

[Porting Assistant for .NET](https://aws.amazon.com/porting-assistant-dotnet/) (PA) is a tool from AWS that helps you port your .NET Framework code to modern dotnet. AWS describes it as "an analysis tool that scans .NET Framework applications and generates a .NET Core compatibility assessment, helping you port your applications to Linux faster."

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679753156696/923b663a-5062-4a9c-a530-4bfea523abca.png align="center")

PA is an **assistive tool**. That means it will help you port, but some manual effort will likely remain. It reduces your level of effort but doesn't completely eliminate it.

Here's how it works. First PA scans your .NET Framework applications to discover APIs and NuGet packages that aren't compatible with modern dotnet. Next, a compatibility assessment report is generated, which suggests available replacements. Lastly, your code is ported, with packages and project references updated for you. PA can also visualize your application dependencies graphically.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679753089898/b6c4d565-48af-4a4d-a9e3-12dc604285eb.png align="center")

One other thing to know about PA is that its dataset is open source, combining data from AWS and other public sources. The dataset that drives it is [publicly available on GitHub](https://github.com/aws/porting-assistant-dotnet-datastore).

Below is a walkthrough of Porting Assistant. That's followed by a short tutorial you can try yourself.

## Walkthrough

Now, let's see what it's like to use PA with a non-trivial application. We'll run it against the AWS sample app [GadgetsOnline](https://github.com/aws-samples/dotnet-modernization-gadgetsonline). GadgetOnline is a .NET Framework MVC web application for online shopping that uses a SQL Server database. Here's what it looks like:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679754079222/0f355e71-e653-45d3-a7b5-5eeb66fc8377.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679754054186/01fac4eb-9394-4d54-a1e4-5dbb7ffff3a5.png align="center")

Opening the solution in Visual Studio, we see a .NET Framework 4.7.2 app with MVC controllers.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679754145989/ae9f30a3-9093-4e16-9978-81321ad9ad7d.png align="center")

We launch Porting Assistant for .NET, a desktop tool, and assess the application. The assessment overview shows that 8 of 15 NuGet packages are incompatible and 43 out of 66 APIs are incompatible. We can review the analysis details on different tabs.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679754745096/232a7e6c-6a45-4137-a1e7-a19bcd55ddec.png align="center")

### NuGet tab

On the **NuGet** tab, we see the specific packages and their compatibility status.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679754915168/54245856-e166-4962-b56b-9480ceb68294.png align="center")

### Project References tab

On the **Project references** tab, you can see a graph of projects referenced by other projects. In the case of GadgetsOnline, there's just one project.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679760804113/420a2d15-5451-4c7b-a49e-6ff8c2c7d37a.png align="center")

In comparison, here's what the visualization looks like for a more complex application, notCommerce:

![](https://d1.awsstatic.com/product-marketing/Porting%20Assistant%20Dotnet/Project-dependency-visualization.7fef0dc301fefeb84cdeed30cdae0b0088a8ab3f.png align="left")

### APIs tab

On the **API** tab, APIs and compatibility statuses are listed. PA will offer suggested replacements for some of them. For example, replacing System.Web.Mvc with Microsoft.AspNetCore.Http. PA will also recommend open source solutions to compatibility problems, such as replacing Windows Community Foundation (WCF) with [CoreWCF](https://github.com/CoreWCF/CoreWCF), an open source project that AWS and Microsoft contribute to.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679755108417/52f69d50-5b7b-4c81-847a-da23420c66ce.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679755352072/3c9e5fc8-c399-4c41-929d-dc37e9b58f71.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679755166168/802ea138-01b4-40e6-811d-c4ef085a06a1.png align="center")

### Source files tab

The **Source files** tab gives you a view of source code, annotated with comments. Select a source code file to see see exactly what changes PA intends to make.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679756373972/92f0b50f-649c-499b-9408-670bd6f1701a.png align="center")

The annotations will tell you what changes PA plans to make when it is confident of a porting action.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679755862278/c453d5da-bef5-49b5-ad72-95c79964ce17.png align="center")

When PA doesn't have a plan of action, it will call out the incompatible code, which you'll be responsible for porting. PA will recommend strategies for some of them.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679756004733/bf096629-62cc-4315-a401-b19dede732c5.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679756043185/30556463-1dec-4568-8541-3ca33081046b.png align="center")

### Port

At this point, you now know what you're in for to port this application to modern dotnet, and what PA will do to assist you. You might decide to move forward with porting, or rewrite the app, or just leave it as is. If you decide to move forward, you click **Port solution** and PA ports your code.

Here's GadgetsOnline after PA has ported it. It's now a .NET 6 application. There are 28 errors the developer still has to address, but it would have been a much larger number without the assistance PA provided.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679756687878/b78d3d1e-19a0-4cd4-93ac-6601ad6ecc12.png align="center")

# Our Hello, Porting Assistant Project

Now it's your turn. We will use a really simply example to give you the experience of using Porting Assistant for .NET from start to finish.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679757520984/89fd4f52-5b9b-4aba-bd3d-15bb5ac9ca64.png align="left")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679757539402/1d02e816-0d0f-4511-b1ad-e5ef5d30456d.png align="center")

[source code](https://github.com/davidpallmann/%5Blink%5D)

## Step 1: Create a .NET Framework console app

In this step, you'll create a simple .NET Framework console app as our starting point. You can either follow the steps below, or just get the code from the source code GitHub repo.

1. Launch Visual Studio or your favorite IE, and create a new .NET Framework console application named **csv\_to\_json**.
    
2. Open Program.cs in the code editor and replace it with the code below. This code reads a CSV file and displays the data as JSON, using code found at [https://qawithexperts.com/article/c-sharp/convert-csv-to-json-in-c/465](https://qawithexperts.com/article/c-sharp/convert-csv-to-json-in-c/465).
    

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using Newtonsoft.Json;
using Newtonsoft.Json.Converters;

namespace csv_to_json
{
    internal class Program
    {
        static void Main(string[] args)
        {
            // Convert CSV to JSON
            // source: https://qawithexperts.com/article/c-sharp/convert-csv-to-json-in-c/465

            var csv = new List<string[]>();
            var csvFile = args[0];
            var lines = System.IO.File.ReadAllLines(csvFile); // csv file location

            // loop through all lines and add it in list as string
            foreach (string line in lines)
                csv.Add(line.Split(','));

            //split string to get first line, header line as JSON properties
            var properties = lines[0].Split(',');

            var listObjResult = new List<Dictionary<string, string>>();

            //loop all remaining lines, except header so starting it from 1
            // instead of 0
            for (int i = 1; i < lines.Length; i++)
            {
                var objResult = new Dictionary<string, string>();
                for (int j = 0; j < properties.Length; j++)
                    objResult.Add(properties[j], csv[i][j]);

                listObjResult.Add(objResult);
            }

            // convert dictionary into JSON
            var json = JsonConvert.SerializeObject(listObjResult);

            //print
            Console.WriteLine(json);
        }
    }
}
```

1. Build the app.
    
2. Close Visual Studio.
    

## Step 2: Test the .NET Framework version

In this step, you'll test the original program to see it work.

1. Use notepad or another text editor, create a test CSV file named **orders.csv** in the project folder with this content:
    

```json
CustomerId,Qty,SKU,Subtotal
1045,2,WIDGET-001,$18.98
1036,1,SPROCKET-311,$49.99
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679757506910/a9b50072-d38c-43ed-8ed8-799c87c55f22.png align="left")

1. Open a command window and CD to the csv\_to\_json project folder.
    
2. Run the command bin\\debug\\vsv\_to\_json orders.csv
    
3. You see the CSV data displayed as JSON.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679758929944/656f54d2-f427-49bf-bddc-80d81e7c3577.png align="center")

Now that we know what our program does, port it to modern dotnet.

## Step 3: Port the solution with Porting Assistant

Now, we'll use PA to port the solution.

1. Install Porting Assistant for .NET by visiting its [product page](https://aws.amazon.com/porting-assistant-dotnet/) and clicking Download Porting Assistant for .NET. Run the installer.
    
2. Launch Porting Assistant for .NET. On the **Settings** page, clik **Edit** and set the target .NET version. We're using **.NET 6** here.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679757986911/0f57d9bb-d2a3-4491-914c-f0a8dc3532fe.png align="center")

3\. On the **Assessed solutions** page, click **Assess a new solution**.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679758082652/1b0f2252-3f7a-4c0c-acde-3db7832b618f.png align="center")

1. Cick **Choose file** and select the `csv_to_jsn solution` file. Then click **Assess**.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679758264916/1fc64161-8cd1-413a-8c1d-0954b2b927b9.png align="center")

1. Wait for a green message that the assessment has been completed.
    
2. Click the `csv_to_json` application name to see the asssesment detail. This simple app does not pose any porting obstacles.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679758556083/230f8cc0-87aa-4ef6-927b-d25eac38abef.png align="center")

1. Click **Port solution**. Choose the option to **Modify source in place**, and click **Save**. Then click **Port**.
    
2. ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679759015922/36b2d545-a9ab-4acb-a5c2-42eb0e4f6984.png align="center")
    
    Wait for green messages confirming the code has been ported and re-assessed.
    
3. ![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679759093255/d1d881ab-e736-45c9-a96f-202565e9169e.png align="center")
    
    Close PA.
    

## Step 4: Review and Build the Ported Solution

1. In Visual Studio, again open the csv\_to\_json solution.
    
2. Review the solution. Note that is now a .NET 6 solution, and has modern dotnet structure including a Startup.cs file.
    
3. Build the solution, and note build errors. We must add the package NewtonSoft.json.
    
4. In Solution Explorer, right-click the project, select Manage NuGet Pacakges, and find and install package **Newtonsoft.json**.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679759476425/0a7ccf40-82ad-4b26-bbc0-eef55e270f61.png align="center")

1. Build the project, which should now build without errors.
    
2. Close Visual Studio.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679759350648/96921c27-92b2-41ba-ab44-14b8102222d0.png align="center")

## Step 5: Test the Ported Solution

In this step, you'll test your ported modern dotnet solution.

1. Return to the command window from Step 2.
    
2. Run the program with this command **dotnet run orders.csv**
    
3. The program runs, now on modern dotnet, with identical output as its original .NET Framework edition.
    

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1679759599368/8cfb3b66-beeb-414e-bb0a-d3022aba392d.png align="center")

Congratulations, you successfuly ported a .NET Framework app to modern dotnet with Porting Assistant for .NET.

# Where To Go From Here

Porting Assistant for .NET assists you in porting legacy .NET Framework applications to modern dotnet. It won't do the entire job for you, but it does the heavy lifting and reduces the amount of manual effort.

You might be on the fence about whether porting is worthwhile. Porting Assistant lets you know what the isues are, and helps with manyu of them.

To go further, read and watch the resources below. Also take a look at the [.NET Toolkit for Refactoring](https://aws.amazon.com/visual-studio-net/), which is a newer experience for our .NET modernization tools that can be used from Visuak Studio.

# Further Reading

AWS documentation

[Porting Assistant for .NET](https://aws.amazon.com/porting-assistant-dotnet/)

Infographics

[Porting Assistant for .NET](https://d1.awsstatic.com/developer/-net-assets/infographic_porting-assistant_v5.pdf)

Videos

[AWS for the .NET Developer - Porting Assisstant for .NET](https://www.youtube.com/watch?v=a3PI3klFtk8) by Isaac Levin

Blogs

[Modernizing ASP .NET Web Forms applications to Blazor using Porting Assistant for .NET](https://aws.amazon.com/blogs/modernizing-with-aws/modernizing-asp-net-web-forms-to-blazor/) by Carlos Santos

GitHub

[porting-assistant-dotnet-datastore](https://github.com/aws/porting-assistant-dotnet-datastore)