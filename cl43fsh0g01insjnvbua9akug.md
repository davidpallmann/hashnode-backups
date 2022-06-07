## Hello, Alexa!

#### This episode: Amazon Alexa and voice applications. In this Hello, Cloud blog series, we're covering the basics of AWS cloud services for newcomers who are .NET developers. If you love C# but are new to AWS, or to this particular service, this should give you a jumpstart.

In this post we'll introduce Amazon Alexa and use it in a "Hello, Cloud" .NET program to create a simple voice skill. We'll do this step-by-step, making no assumptions other than familiarity with C# and Visual Studio. We're using Visual Studio 2022 and .NET 6.

# Amazon Alexa : What is it, and why use It?

> "The human voice is the most beautiful instrument of all, but it is the most difficult to play. —Richard Strauss 

Voice assistants are cropping up everywhere, and there's a simple reason for that: they work well, and they're free. The notion of a computer you could talk to in natural language was popularized back in the original 1960's Star Trek TV series, and I remember well various attempts by tech companies to bring this vision to life over the decades that followed. They were anything but promising. You had to train them to understand your voice, or perhaps it was more accurate to say that they trained you. All of that changed in the last decade, with voice assistants that require no training and just work.

[Amazon Alexa](https://developer.amazon.com/en-US/alexa) (hereafter "Alexa") is Amazon's voice assistant. Amazon describes it as "a cloud-based voice service available on millions of devices from Amazon and third-party manufactures." Today, Alexa integrates into a great many devices, including a variety of Amazon Echo and Amazon Fire TV devices, some of which have displays. My clock radio has been replaced by an Amazon Echo Show 5. I have an Echo Dot in my home office. I have a Fire TV Stick. I can talk to all of these devices.

![alexa-devices.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1654511417085/vgKvmGV6-.jpg align="left")

[Alexa features](https://www.amazon.com/b?ie=UTF8&node=21576558011) include smart home, productivity, shopping, entertainment, news, routines, information, games, photos, and audio. There is no charge or monthly fee for Alexa, aside from the cost of purchasing an Alexa-enabled device. If having listening devices in your home concerns you, read up on [Alexa Privacy](https://www.amazon.com/b/?node=19149155011), which explains Alexa's privacy and security protections.

The user experience with Alexa is easy to understand, since you've probably experienced it already—but how does it work, and how can you work with Alexa programmatically? Developers can create their own Alexa voice applications, called **skills**. An Alexa skill triggers an AWS Lambda function, which responds to voice queries and can drive a dialog.

![diagram-alexa-lambda.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1654513577100/MkmcBjbDc.png align="left")

## Concepts

An Alexa skill is defined with a model, composed of the following:
* An **invocation name** is what identifies the skill so it can be invoked. Example: "fast pizza".
* **Intents** are actions the user can do with the skill. For example, "OrderPizza".
* **Utterances** are phrases that signal an intent. For example, "I want pizza" and "order a pizza" are two utterances that might signal the same intent, "OrderPizza".
* A **slot** is a variable that can appear in utterances, which the user can supply with recognized values. For example, in the utterance "order a {size} {topping} pizza", the `size` slot could accept small, medium, and large and the `topping` slot could allow tomatoes, sausage, cheese, or pepperoni as values.
* An **endpoint** is a service that responds to intents. This can be an AWS Lambda function, but doesn't have to be.

![diagram-model.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1654560672656/VI1BINgnJ.png align="left")

# Our Hello, Alexa Project

We will create a simple Alexa skill that reports the current time in a variety of locations. We'll write a C# Lambda function to be the back-end for this skill. In future posts, we'll build more elaborate skills that sustain a dialog.

![skill-test-dev-console.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1654518294552/vqYzoJzT3.png align="left")

[source code](https://github.com/davidpallmann/hello-alexa)

## One-time Setup

For any of the tutorials in the Hello, Cloud series you need the following:

1. An  [AWS account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/), and an understanding of what is included in the  [AWS Free Tier](https://aws.amazon.com/premiumsupport/knowledge-center/what-is-free-tier/). 
2. [Microsoft Visual Studio 2022](https://visualstudio.microsoft.com/). If you're using an older version of Visual Studio you won't be able to use .NET 6. If you use a different IDE, you may have to find alternatives to some tutorial steps.
3. [AWS Toolkit for Visual Studio](https://aws.amazon.com/visualstudio/). You'll need to [configure the toolkit ](https://docs.aws.amazon.com/toolkit-for-visual-studio/latest/user-guide/keys-profiles-credentials.html) to access your AWS account and create an IAM user. Your default AWS profile will be linked to this user when running programs from the command line.

## Step 1: Create an Alexa Skill

In this step, you'll set up an Amazon Developer Account and create an Alexa skill.

1. To develop for Alexa, you'll need to sign up for an [Amazon Developer Account](http://developer.amazon.com/). 

2. Once you have your developer account, sign in and navigate to the [Amazon Alexa area](https://developer.amazon.com/alexa/console/ask).

![01-alexa-console.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1654518678424/5mrRnylPL.png align="left")

3. Create a new skill:

    A. Click the **Create Skill ** button.

    B. Skill name: **hello skill** (*hello alexa* is not allowed).

    C. *Choose a model to add to your skill*: select **Custom**.

    D. *Choose a method to host your skill's backend resources*: select **Provision your own**.

    E. Click **Create skill**.

    ![01-create-skill.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1654462142092/6XErOwDBl.png align="left")

    F. *Choose a template to add to your skill*: select **Start from Scratch**.

    G. Click the **Choose** button.

      ![01-create-skill-2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1654461406522/0EVHk0_wi.png align="left")

    D. Wait for your skill to be created.

    ![01-create-skill-3.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1654461487844/lQSBcmebc.png align="left")

    Once you see the main development dashboard displayed without any wait messages, continue on.

## Step 2: Build a Model for the Skill

In this step, you'll define the model for your skill, which consists of intents, utterances, and slots. We'll be creating a simple skill that tells you what time it is in a location - for example, "What time is it in Chicago?"

1. On the Developer dashboard, find the checklist at right and click on 1. Invocation.

    A. Skill invocation name: **hello skill**.

    B. Click the **Save Model** toolbar button at top.

2. Select **Build** in the top navigation to return to checklist, and click **2. Intents, Samples, and Slots**. Then do the following to add an intent:

    A. On the Add intent page, select **Create custom intent** and enter name **what_time_is_it**. 

    B. Click **Create custom intent**. Now you are on the Intents / what_time_is_it page.

    ![02-create-intent-1l.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1654463071495/WS39aE3UV.png align="left")

    C. In the Sample Utterances input box, enter *What time is it* and click the plus sign (+) at right.

    ![02-create-intent-1-utterance.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1654463380285/sDya_edPg.png align="left")

    D. Enter several more utterance phrases, such as **Got the time** and **Do you have the time**. Feel free to add other phrases that signal the intent of inquiring what time it is.

    ![02-create-intent-1-utterances.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1654463354500/ZACA-OC8J.png align="left")

    E. Enter this utterance: **What time is it in {location}**. {location} is a placeholder for a place the user can say, such as "Chicago" or "Dallas". As you enter the phrase, a pop-up asks you to select an existing slot or add a new one.

    ![02-create-intent-1-utterance-what-time-in.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1654463602744/u4NXl6-ql.png align="left")

    F. Click **Add** to add a new slot named **location**.

    G. Click the plus sign (+) to add the utterance.

    H. Click **Save Model**.

3. In the outline at left, select **Slot Types**.

    A. Click **Add Slot Type**.

    B. Select **Create a custom slot type with values**.

    C. In the input box, enter a place name **location** and click **Next**. 

    ![02-create-intent-1-slots-location.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1654464030361/CaiJZJ4pJ.png align="left")

    D. On the Slot Types / Add Slot Type / location page, set Slot Type to **location**.

    E. enter a city name such as **Atlanta** and click the plus sign (+).

    ![02-create-intent-1-slots-atlanta.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1654464171062/GNVR_oX7W.png align="left")

    F. Repeat sub-step D to enter more city names so that you have cities covering several time zones. We are using **Atlanta**, **New York** (US eastern time), **Chicago**, **Dallas** (central time), **Denver**, **Phoenix** (mountain time), **Los Angeles** and **Seattle** (western time).

    ![02-create-intent-1-slots-cities.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1654464409242/mKYiot5wB.png align="left")

    G. Under Slot Filling - *Is the slot required to fulfill the intent?*, toggle the switch on.

    H. Click **Save Model**.

    We now have a handful of cities defined, any of which will be understood as a valid value for {location} in our utterance, "What time is in in {location}?"

4. Do a sanity check in the developer portal against the definitions below. Make sure that everything is there, and that you have saved your changes with Save Model.

| Artifact | Property | Value |
| ------------- -| ------- | ---- |
| Invocation |  Skill Invocation Name | hello skill |
| Intents / HelloWorldIntent | ... | (auto-generated from template) |
| Intents / what_time_is_it | Sample Utterances | what time is it in {location} |
|                                           |                                  | Do you have the time |
|                                           |                                  | Got the time |
|                                           |                                  | What time is it |
| Intents / what_time_is_it | Intent Slots | name location, slot type location |
| Intents / what_time_is_it / location | Slot Type | location |
| Intents / what_time_is_it / location | Slot Filling - Is this slot required to fulfill the intent? | YES |
| Intents / what_time_is_it / location | Slot Filling - Alexa speech prompts | What location do you want the time for? |
| Assets / Endpoint |  Your Skill ID | amzn1.ask.skill.xxxxxxxxxxxxxxxxx |
| Assets / Endpoint |  Default Region | arn:aws:lambda:xxxxxxxxxx |
| Slot Types  | NAME | location |
| Slot Types / location | VALUE | Seattle |
| Slot Types / location | VALUE | Los Angeles |
| Slot Types / location | VALUE | Phoenix |
| Slot Types / location | VALUE | Denver |
| Slot Types / location | VALUE | Dallas |
| Slot Types / location | VALUE | Chicago |
| Slot Types / location | VALUE | New York |
| Slot Types / location | VALUE | Atlanta |
| Slot Types / location | Slots Using (location) | what_time_is_it |

5. Click **Build Model** to build your model. Wait for a Build Completed message. If any errors are reported, double check that you correctly completed the above steps.

    ![02-create-intent-build-completed.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1654464629551/vrKupV8Kh.png align="left")

6. Select JSON Editor from the outline at left, and you'll see your model of intents, utterances, and slots in JSON form. You'll notice this includes what you have defined, plus a "Hello, World" definition that was auto-generated for you. Ignore that pre-generated model for now, and notice that the intent, utterances, and slots you defined are listed in the JSON: you see your `what_time_is_it` intent.

![02-json.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1654561990228/dGfjXnDt6.png align="left")

7. On the outline left, click Slot Types > **Endpoint**. Look for Your Skill ID and record it, which will look like `amzn1.ask.skill.xxxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxx`.

    ![02-skill-id.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1654469284529/EL679O7aK.png align="left")

## Step 3: Create a Lambda Function for the Back-end

In this step, you'll create the back-end for the Alexa skill, a C# Lambda function.

1. Launch Visual Studio 2022 and select **Create a new project**. 

2. In the Create a new project wizard, find and select **AWS Lambda Project (.NET Core - C#)**. Click **Next**.

    ![03-create-project.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1654466173851/3059OYDVs.png align="left")

3. Name the project and solution **HelloAlexa**, select a development folder. Click **Create**.

    ![03-create-project-2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1654466370458/kzUgsDBDr.png align="left")

4. On the Select Blueprint page, select **Empty Function** and click **Finish**.

    ![03-create-project-3.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1654466636146/kU0JWkkB1.png align="left")

5. Add NuGet packages Alexa .NET and Amazon.Lambda.Serialization.Json:

    A. In Solution Explorer, right-click the HelloAlexa project and select **Manage NuGet Packages..."

    B. On the Browse tab, search for and install **Alexa .NET**. This package by Tim Heuer provides convenient .NET classes for working with the Alexa API from .NET.

    C. In the same way, find and add the **Amazon.Lambda.Serialization.Json** package.

    ![03-create-project-4-nuget.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1654467066276/UhUZ2Bxwz.png align="left")

6. Open Function.cs in the code editor. The template provides a very simple default function that accepts a string and returns an upper-case version of it.

7. Replace the code with code at the end of this step. 

Function.cs

```csharp
using Amazon.Lambda.Core;
using Alexa.NET.Response;
using Alexa.NET.Request;
using Alexa.NET.Request.Type;
using Newtonsoft.Json;
using Alexa.NET;

// Assembly attribute to enable the Lambda function's JSON input to be converted into a .NET class.
[assembly: LambdaSerializerAttribute(typeof(Amazon.Lambda.Serialization.Json.JsonSerializer))]

namespace HelloAlexa;

public class Function
{
    public SkillResponse FunctionHandler(SkillRequest input, ILambdaContext context)
    {
        ILambdaLogger log = context.Logger;
        log.LogLine($"Skill Request Object:" + JsonConvert.SerializeObject(input));

        Session session = input.Session;
        if (session.Attributes == null)
            session.Attributes = new Dictionary<string, object>();

        Type requestType = input.GetRequestType();
        if (input.GetRequestType() == typeof(LaunchRequest))
        {
            string speech = "Welcome! I can tell you the time in different cities.";
            Reprompt rp = new Reprompt("Say what time is it in a city");
            return ResponseBuilder.Ask(speech, rp, session);
        }
        else if (input.GetRequestType() == typeof(SessionEndedRequest))
        {
            return ResponseBuilder.Tell("Goodbye!");
        }
        else if (input.GetRequestType() == typeof(IntentRequest))
        {
            var intentRequest = (IntentRequest)input.Request;
            switch (intentRequest.Intent.Name)
            {
                case "AMAZON.CancelIntent":
                case "AMAZON.StopIntent":
                    return ResponseBuilder.Tell("Goodbye!");
                case "AMAZON.HelpIntent":
                    {
                        Reprompt rp = new Reprompt("What's next?");
                        return ResponseBuilder.Ask("Here's some help. What's next?", rp, session);
                    }
                case "what_time_is_it":
                    {
                        string location = intentRequest.Intent.Slots["location"].Value;
                        DateTime now = DateTime.UtcNow;
                        (string place, int offset, string timezone) localTime = GetLocationOffset(location);
                        string message = $"Right now in {localTime.place} it is {now.AddHours(localTime.offset).ToShortTimeString()} {localTime.timezone}.";
                        return ResponseBuilder.Tell(message, session);
                    }
                default:
                    {
                        log.LogLine($"Unknown intent: " + intentRequest.Intent.Name);
                        string speech = "I didn't understand - try again?";
                        Reprompt rp = new Reprompt(speech);
                        return ResponseBuilder.Ask(speech, rp, session);
                    }
            }
        }
        return ResponseBuilder.Tell("Goodbye!");
    }

    private (string location, int offset, string timezone) GetLocationOffset(string location)
    {
        switch(location)
        {
            case "Los Angeles":
            case "Seattle":
                return (location, -8, "Pacific Standard Time");
            case "Denver":
            case "Phoenix":
                return (location, -7, "Mountain Standard Time");
            case "Chicago":
            case "Dallas":
                return (location, -6, "Central Standard Time");
            case "Atlanta":
            case "New York":
                return (location, -5, "Eastern Standard Time");
            default:
                return ("Greenwich", 0, "Greenwich Mean Time");
        }
    }
}
```

8. Publish your function to AWS. 

    A. In the AWS Explorer view, set region us-east-1 (N. Virginia).

    B. In Solution Explorer, right-click the HelloAlexa project and select **Publish to AWS Lambda**.

    C. Set Function Name to **HelloAlexa**.

    D. Confirm the region is set to us-east-1 (N. Virginia).

    E. Under Handler, enter **HelloAlexa::HelloAlexa.Function::FunctionHandler**. This identifies the `assembly:namespace.className:methodName` of the function handler.

    D. Click **Next**.

    ![03-pubilsh-lambda.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1654468808647/RB14xiknV.png align="left")

    E. Under Role Name, let the wizard create a role or select an existing role with Lambda execution permissions.

     F. Click **Upload**.

    ![03-pubilsh-lambda-2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1654468074066/6fu0sWH2I.png align="left")

    G. Wait for the publish action to complete. When it completes, you should be on the Function:HelloAlexa test page in Visual Studio, and see **Last Update Status: Successful** at top.

    ![03-pubilsh-lambda.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1654470270614/HJjUeZLYs.png align="left")

### Understand the Code

The code consists of a function handler, which services voice input from Alexa, and a local function for helping determine the time offset for a location.  

FunctionHandler (lines 15-66) gets two parameters, input and context. `'input` is a `SkillsRequest `with the details of the voice utterance we need to respond to. `context` is an `ILambdaContext` object we can use for logging as the function executes.

24-30: FunctionHandler gets the input request type, then uses a series of conditionals to respond to the request. If it's a launch request, we provide a welcome prompt, `Welcome! I can tell you the time in different cities.`. We also tell Alexa that if a later re-prompt is needed, it can say `Say what time is it in a city`. The response is returned with the `ResponseBuilder.Ask` method, which is awaiting another intent.

31-34: If the request is session end request, our dialog is ending. We say `Goodbye` with `ResponseBuilder.Tell`.

35-64: If the response is an intent, we get the intent request from `input.Request` and use a switch statement on `intentRequest.Intent.Name` to determine whether it's one of the Alexa built-in intents (cancel, stop, help). If it's our intended `what_time_is_it` intent (lines 48-55), we get the `location` slot value from the intent request. We then call a local function to get the place name, timezone offset, and time zone name based on the location slot value, returned as a tuple. We then create a message of the form `Right now in New York it is 11:35 AM Eastern Standard Time` and return it with `ResponseBuilder.Tell`.

GetLocationOffset (lines 68-88) does a switch on the location slot value. If it's one of the values we expect, we return a tuple containing the location name, UTC offset from GMT, and name of the time zone. We are not handling daylight savings time in this simple implementation. If we don't recognize the location, we return GMT time and identify the place as Greenwich and time zone as Greenwich Mean Time.

## Step 4: Configure Lambda Function

In this step, you'll configure the AWS Lambda function that was just created in the AWS console to respond to your Alexa skill as a trigger.

1. Sign in to the AWS management console. At the top right, select the same region you've been using in prior steps.

2. Navigate to **AWS Lambda**. You can enter **lambda** in the search bar.

3. Your HelloAlexa Lambda function should be listed. Click it to view its detail.

     ![04-aws-lanbda.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1654468910612/SN4iFJMg1.png align="left")

4. Add a trigger:

    A. Expand the Function Overview section of the page.

    B. Click the **Add Trigger** button and select **Alexa Skills Kit**.

    C. Enter your Alexa Skill ID that you recorded at the end of Step 2 and click **Add**.

    ![04-aws-lambda-add-trigger.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1654470428927/ldazKU__P.png align="left")

    ![04-aws-lambda-add-trigger-2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1654470504024/T5wR0KKHy.png align="left")

5. Record the Amazon Resource Name (ARN) of your Lambda function. Ours is `arn:aws:lambda:us-east-1:xxxxxxxxxxxx:function:HelloAlexa`.

6. In the Alexa Developer Console, select Endpoint from the outline at left.

    A. Replace the AWS Lambda ARN with the ARN you copied at the end of Step 4.

    B. Click **Save Endpoints**.

    ![04-skill-update-arn.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1654470755809/I3oLx3-oU.png align="left")

## Step 5: Test Your Alexa Skill

In this step, you'll test your new Alexa skill and Lambda function back end from the Alexa Developer Console.

1. The Alexa Developer Console, select the **Test** tab at top.

2. In the skill testing dropdown, select select **Development**.

3. Test the utterance **ask hello skill, what time is it in New York?** by 1) typing it in the input box or 2) clicking-and-holding the microphone icon while you say the phrase into your microphone.

    ![05-test-new-york.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1654471072452/ATdwsUJeN.png align="left")

4. You should hear and see a response with the time. There are two ways you can invoke and test your skill: 

    1) invoke your skill and ask a query in a single utterance: "Ask hello skill, (query)" 

    `ask hello skill, what time is it in Denver?`

    ![skill-test-dev-console.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1654518333486/DfAPWXJdO.png align="left")

    or 2) say "open hello skill" to invoke your skill, and then say your query as a separate utterance: "what time is it in (location)?"

    `open hello skill`

    `what time is it in Denver?`

    ![skill-test-dev-console-combined.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1654518459005/SnXTW-uGS.png align="left")

5. Now try other cities you defined earlier. Remember to invoke your skill each time. Try, "What time is it in Denver?" or "What time is it in Seattle?"




# Where to Go From Here

In this tutorial, you created a very simple Alexa skill and a C# AWS Lambda function to implement it. You were exposed to the Alexa skill model of intents, utterances, and slots. With the help of the Alexa .NET helper library, you wrote an AWS Lambda function to implement your skill.

We barely scratched the surface with Alexa. A more elaborate skill and Lambda function could support multiple intents and sustain a dialog. We did not go over how to publish your Alexa skill. You can integrate with a plethora of home devices, AWS services, or other online resources. That's fertile ground for innovation. We'll cover more of Alexa in future blog posts.

# Further Reading

Amazon Alexa

[Amazon Alexa](https://developer.amazon.com/en-US/alexa)

[Alexa Privacy](https://www.amazon.com/b/?node=19149155011)

[AWS SDK for .NET Documentation](https://docs.aws.amazon.com/sdk-for-net/index.html)

[Amazon Developer Console](http://developer.amazon.com/)

[Alexa Developer Console](https://developer.amazon.com/en-US/alexa)

[Understand Custom Skills](https://developer.amazon.com/en-US/docs/alexa/custom-skills/understanding-custom-skills.html)

Packages

[Alexa.NET](https://www.nuget.org/packages/Alexa.NET/)

Videos

[How to Build Alexa Skill using C# and AWS Lambda](https://www.youtube.com/watch?v=yhCoVfKYp7M) by Nick Naddaf

[ASP.NET Core Alexa App and NGROK](https://www.youtube.com/watch?v=sJEaXhtR4YM) by Shiv Kumar

[Let's build an Alexa Skill Together Using C# on AWS](https://www.youtube.com/watch?v=BPWmaYJIjCk) by Paul Oliver

Blogs

[Build an Alexa Skill with .NET Core and AWS Lambda](https://medium.com/trimble-maps-engineering-blog/how-to-build-an-alexa-skill-with-net-core-and-aws-lambda-fe41903dad9f) by Matthew Trimble

[Hello, Cloud blog series](https://davidpallmann.hashnode.dev/series/hello-cloud)