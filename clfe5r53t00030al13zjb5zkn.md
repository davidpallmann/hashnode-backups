---
title: "7 Reasons to Modernize Your .NET Applications"
datePublished: Sat Mar 18 2023 16:02:51 GMT+0000 (Coordinated Universal Time)
cuid: clfe5r53t00030al13zjb5zkn
slug: 7-reasons-to-modernize-your-net-applications
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1679154315656/7ca01eef-5e16-46a2-a0ca-a730413f3a02.jpeg
tags: aws, net, dotnet

---

It's commonly estimated that [75% of enterprise applications are .NET based](https://www.janbask.com/blog/why-net-is-the-most-used-platform-for-enterprise-application-development/). Since .NET has been around for over 20 years, that means most enterprises have a portfolio that includes legacy .NET applications. Below are 7 reasons why you should consider modernizing your .NET applications. Along the way, I'll share AWS tools and services that can help. To distingish between legacy .NET framework and modern .NET, I'll use the term **dotnet** to refer to open source, cross-platform .NET (formerly called .NET Core).

By modernizing, I include any of the following: porting your .NET Framework app to dotnet, migrating to cloud, moving to containers, using managed services, moving to open source platforms, moving to a microservices architecture, and going cloud-native and serverless.

### 1\. .NET Framework is the past, dotnet is the future

The .NET Framework is not dead, but it's on life support. Although Microsoft has committed to [support .NET Framework 4.8](https://dotnet.microsoft.com/en-us/platform/support/policy/dotnet-framework) on supported versions of Windows, staying with a platform that no longer advances is unwise. The future of .NET is modern **dotnet**, which continues to move forward and has a bright future. Whether you run on-premise or in the cloud, porting to modern dotnet both brings you regular feature updates and avoids a growing problem of lapsing support for your dependent libraries and packages. AWS provides two tools to assist you in porting, [Porting Assistant for .NET](https://aws.amazon.com/porting-assistant-dotnet/) ([demo](https://www.youtube.com/watch?v=a3PI3klFtk8)) and [AWS Refactoring Toolkit for Visual Studio](https://aws.amazon.com/blogs/modernizing-with-aws/aws-toolkit-for-net-refactoring-launch/) ([demo](https://www.youtube.com/watch?v=Om9tG1x0N2s)).

### 2\. It's a container world now

Containers are now a de facto standard, and you should be using them. They make your applications portable and easy to move between environments, including on-premise and different clouds. Newer cloud services tend to be container-based and managed. Check out the [AWS containers page](https://aws.amazon.com/containers/) to see how many options there are. Both modern dotnet apps and legacy .NET Framework apps can run in containers. You can use the [App2Container](https://aws.amazon.com/app2container/) tool to containerize your .NET Framework IIS web apps running on Windows and deploy them to cloud ([demo](https://www.youtube.com/watch?v=UH00bPngaVI)).

### 3\. You can reduce costs by running on Linux

Now that dotnet is cross-platform, you have the option of running on Linux where you can avoid Windows licensing costs. If Windows has been your world up till now, dont be concerned. It's easy to deploy dotnet to Linux in the cloud, and you don't need to become a Linux expert. To take advantage of running on Linux you'll either need to port or rewrite your .NET Framework apps.

### 4\. Moving to microservices is loose coupling at its best

[Microservice](https://aws.amazon.com/microservices/) architectures make applications easier to scale and faster to develop. They're a refreshing change from the traditional monolithic application that is ever growing in complexity and fragile due to tight coupling. Microservices are not only a better refactoring of your application, they're better organizationally because each can be owned by the most suitable team. Best of all, this is one kind of modernization that you can do *progressively*, moving portions of a monolith to microservices over time using the [Strangler Fig pattern](https://martinfowler.com/bliki/StranglerFigApplication.html). Use the [AWS Microservice Extractor tool for .NET](https://aws.amazon.com/microservice-extractor/) tool for assistance. It uses machine learning to recommend code candidates for microservices and helps you extract them to microservice projects.

### 5\. Using managed services lets you spend your time more productively

In the cloud, you have managed services at your disposal that handle the details of provisioning, monitoring, scaling, and failing over infrastructure. Unless you enjoy configuring and maintaining servers, there's simply no reason to do so when it can be done for you. Instead, you can spend your time innovating for your customers. An example of a fully-managed cloud service that is a great fit for .NET web apps is [AWS App Runner](https://aws.amazon.com/apprunner/).

### 6\. Going cloud-native and serverless maximizes the benefits of cloud

The ultimate modernization of your .NET applications is to go fully [cloud-native](https://aws.amazon.com/what-is/cloud-native/), where your architecture, tools, and techniques revolve around the cloud way of doing things. A big part of that is [serverless](https://aws.amazon.com/serverless/), where your code runs as functions invoked in response to events. For example, your web APIs can be implemented with [Amazon API Gateway](https://aws.amazon.com/api-gateway/) and serverless [AWS Lambda](https://aws.amazon.com/lambda/) functions, autoscaling up and down in response to demand. Cloud-native .NET applications are highly available and elastic, along with the already-mentioned benefits of containers, microservices, and managed services.

### 7\. Modernizing in bulk is a force multiplier

If you have a portfolio of .NET applications, you may be unsure where to start. It makes sense to study your portfolio as a whole. That will help you define a modernization strategy, identify your top candidates for modernization, and perhaps find some low hanging fruit for early modernization wins. You'll discover repeatable patterns that increase your velocity in updating your applications. You can use [AWS Migration Hub Strategy Recommendations](https://aws.amazon.com/migration-hub/features) (MHSR) to analyze your .NET applications, get recomendations, and plan your migration.

### Call to Action

There are compelling reasons to modernize your .NET applications. If even one of the above resonates with you, I urge you to get started. To find the AWS tools and services mentioned above and get started modernizing on AWS, visit and bookmark the [.NET on AWS developer center](https://aws.amazon.com/dotnet).