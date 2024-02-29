---
title: "Jail Break your .NET Application"
datePublished: Thu Feb 29 2024 19:54:35 GMT+0000 (Coordinated Universal Time)
cuid: clt7n9l8d00020aju6t9z2hvx
slug: jail-break-your-net-application
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1709213021344/3fb7b073-cbf3-47d7-a142-01ac1b7e9cb3.png
tags: aws, migration, net, dotnet, modernization

---

*“The best way to escape from your problem is to solve it.” ―Robert Anthony*

Your .NET Framework and SQL Server application has served you well for years ...but it is also a **prisoner**. It is **bound** by its on-premises location, **chained** to license fees, **confined** by a monolithic architecture, and **shackled** by the burden of managing servers. In this post, I'll explain a number of ways in which older .NET applications are imprisoned and how they can be freed in the cloud. I'll get specific about AWS services and tools you can use to accelerate the liberation.

This post covers, with a bit of fun and humor, how to migrate to AWS and modernize all the way to cloud-native. You'll have to decide how much of this journey to take and when. Taking even the first step will free your app from some of its constraints. Taking more steps will add more freedom.

## Learning Game

I developed a learning game called [.NET Jail Break](https://dotnetjailbreak.com/) that you can play to simulate .NET migration and modernization on AWS. Its seven levels parallel this blog post. This was popular at the recent NDC Oslo conference. Feel free to play it on your desktop, tablet, or phone to illustrate what we discuss here. As you play, click on different actions and options and learn which approaches save the most time and effort. It only takes 5 minutes.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709213851825/1bd5c67c-e56d-461d-8da5-7066c77411d5.png align="center")

## Take the elevator to the sky: Lift and Shift

If your application is still on-premises, you're limited to the servers or VMs that your organization owns. That limits your ability to scale up, and doesn't reward you with lower costs when you scale down. This doesn't make on-prem a bad place, but it restricts you in ways the cloud does not.

In the cloud, you can scale elastically with a pay-as-you-go pricing model. You don't have to worry about not having enough server resources. You don't have to worry about paying for unused capacity. Your application scales to match traffic, and you pay for what you use. You're free to stop using the cloud altogether when you no longer need it, without a contract or time commitment.

You can free your application from this on-prem confinement by taking the elevator from the basement to the sky: migrate your application and database to virtual machines in the cloud, with minimal changes. That's known as **lift and shift**, also called *rehosting*. Lift-and-shift is a great way to get into the cloud quickly without a lot of disruption. Let's start first with the database and after that the application.

1. ### Lift and Shift your SQL Server Database
    

You can migrate your existing SQL Server database to AWS in several ways. One is to host it on EC2, with [license-included instances](https://docs.aws.amazon.com/sql-server-ec2/latest/userguide/sql-server-on-ec2-licensing-options.html). An alternative is to host in a managed relational database service, [Amazon RDS for SQL Server](https://aws.amazon.com/rds/sqlserver/).

You can accelerate your database migration to the cloud with [AWS Database Migration Service](https://aws.amazon.com/dms/). DMS helps move your database and analytics workloads to AWS quickly, securely, with minimal downtime and zero data loss.

If you're playing .NET Jail Break, this corresponds to *Level 1: Lift and Shift SQL Server Database to the cloud with minimal changes*.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709223510697/77b240bd-54f0-4f7a-a201-aca8e02dd048.png align="center")

2. ### Lift and Shift your .NET Framework Application
    

With the database in the cloud, now it's time to lift-and-shift the application as well. You can either deploy your app to EC2 instances or move to containers.

**EC2**. You can do the work to set up EC2 and load balancers on your own, but it's fastest to use [AWS Elastic Beanstalk](https://aws.amazon.com/elasticbeanstalk/). Beanstalk is the fastest way to move an application to AWS and doesn't require you to understand AWS infrastructure. Beanstalk deploys easily from the command line or Visual Studio. You can also use the [Windows Web Application Migration Assistant](https://github.com/awslabs/windows-web-app-migration-assistant) tool, an interactive PowerShell utility, to migrate entire websites and their configuration to Beanstalk.

**Containers**. If you're ready to go the container route, you can create a Windows container for your application and deploy it to one of [AWS' container compute services](https://aws.amazon.com/getting-started/decision-guides/containers-on-aws-how-to-choose/). You can do that yourself, but it's faster and easier to use [App2Container](https://aws.amazon.com/app2container/), a free tool that containerizes IIS websites and deploys them to AWS.

If you're playing .NET Jail Break, this corresponds to *Level 2: Lift and Shift .NET Framework website to the cloud with minimal changes*.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709215517501/c7226309-8a2b-4275-b303-2becfa82a3e9.png align="center")

## Stop paying for software: License Freedom

Good job! Your application and database are now running happily in the cloud. But, they could be freer still. Your .NET Framework application requires Windows, and your database requires SQL Server. Those requirements come with license fees and limit the platforms you can run on.

In the cloud, you can **modernize to open-source platforms** that have no license fees and save you money, also called *replatforming*. You can port your .NET Framework application to .NET Core and run on Linux, avoiding Windows license fees. You can migrate your database to an open source database, avoiding SQL Server license fees. We call this *license freedom*.

3. ### Unchain app from license fees
    

If you can transform your application code from .NET Framework to .NET Core, you can run on Linux. You can either port your code or rewrite the application.

**Port.** The task at hand is to port from .NET Framework to .NET Core so you can run on Linux. You could do this on your own, but porting can be a bit of work. AWS offers porting capability in several of its assistive tools, including the [AWS Toolkit for .NET Refactoring](https://aws.amazon.com/visual-studio-net/). It will identify incompatibilities and help port your code. You may have to do some manual work to complete the port, but this will give you a big helping hand.

**Rewrite.** An alternative to porting is to rewrite your app for .NET Core. This might be tempting if you also want to add new functionality or overhaul an aging app's user experience. You can get help during your rewrite from [Amazon CodeWhisperer](https://aws.amazon.com/codewhisperer), an AI coding assistant you can add to Visual Studio, Visual Studio Code, or JetBrains Rider.

If you're playing .NET Jail Break, this corresponds to *Level 3: Unchain application from Windows license fees*.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709223575845/bf9da38b-aff2-4180-8f39-2f846f526e10.png align="center")

4. ### Unchain database from license fees
    

You can avoid database license fees by migrating from SQL Server to an open source relational database such as PostgreSQL or MySQL. Some logical choices on AWS include [Amazon Aurora for PostgreSQL](https://aws.amazon.com/rds/aurora/features/), [Amazon RDS for MySQL](https://aws.amazon.com/rds/mysql/), or [Amazon RDS for PostgreSQL](https://aws.amazon.com/free/database). You can accelerate migration using a tool we saw earlier, the [AWS Database Migration Service](https://aws.amazon.com/dms/).

Different relational databases tend to use different SQL dialects, so you might find yourself needing to update code queries and stored procedures as part of the migrations. If you migrate to Aurora, you can instead leverage [Babelfish for Aurora PostgreSQL](https://aws.amazon.com/rds/aurora/babelfish/) to understand SQL Server's T-SQL dialect and communications protocol.

If you're playing .NET Jail Break, this corresponds to *Level 4: Unchain database from license fees*.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709236004319/9e42d82c-8e0b-43b5-85c4-549a9947f4ad.png align="center")

## Divide and conquer: move from monolith to microservices

Well done. Your application and database are replatformed on AWS and are running license-free! But, you could be freer still. Your application is a monolith. It's complex and is supported by one team.

If you refactor to a microservices architecture, you end up with simple software services. Those services and their databases can be scaled separately to meet demand. Those services can be supported by different teams. This is not to say you should always move from a monolith to microservices: it depends (see Amazon CTO Werner Vogels' article, [Monoliths are not dinosaurs](https://www.allthingsdistributed.com/2023/05/monoliths-are-not-dinosaurs.html)). Determine whether it makes sense for you.

As any microservice purist will tell you, this refactoring involves both your code and database. In a microservices architecture, each microservice has its own database, rather than a shared monolithic relational database.

5. ### Refactor Monolith to Microservices
    

To move from a monolith to microservices requires refactoring, in which you extract sections of code into separate microservice projects. Extracting microservices is something you can do progressively, at whatever pace works for your organization. If you do this to the point that the monolith has been fully replaced, you've followed the [Strangler Fig pattern](https://martinfowler.com/bliki/StranglerFigApplication.html).

If you've inherited this old app, you might be hesitant to change it. Rather than doing this refactoring all on your own, consider an assistive tool: [Microservice Extractor for .NET](https://aws.amazon.com/microservice-extractor/). ME accelerates refactoring by analyzing application code and identifying candidates for extraction, with AI heuristics.

If you haven't yet moved to containers, you'll want to do this now. You can deploy your containerized microservices to container-native services: [AWS App Runner](https://aws.amazon.com/apprunner/), [Amazon Elastic Container Service](https://aws.amazon.com/ecs/) (Amazon ECS), or [Amazon Kubernetes Service](https://aws.amazon.com/eks/) (EKS).

We're not fully there yet on microservices, however. There's still the database to work on.

If you're playing .NET Jail Break, this corresponds to *Level 5: Refactor monolith to microservices.*

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709224133922/e11fcc25-44ce-4cc3-8205-d99be3ee24d0.png align="center")

6. ### Move microservices to a purpose-built database
    

As mentioned earlier, your relational database also needs to be refactored for a microservices architecture. You'll typically use a purpose-built NoSQL database for each microservice. On AWS, you can use [AWS DynamoDB](https://aws.amazon.com/pm/dynamodb), or choose the most appropriate [purpose-built database](https://aws.amazon.com/products/databases/) for each service.

Refactoring your relational database into individual NoSQL databases is work, no doubt about it; but it's often worth it for the benefits of simple components with nimble deployment. Once again, the mighty [AWS Database Migration Service](https://aws.amazon.com/dms/) can help accelerate the work. DMS can create DynamoDB tables from your relational database, saving you work.

If you're playing .NET Jail Break, this corresponds to *Level 6: Move microservices to a purpose-built database.*

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709224179280/a159b9e6-3bb7-4232-a204-1e1166bb0c2d.png align="center")

## Commit to Cloud: Go Cloud-Native

You've come a long way. You're now running microservices in containers on AWS, each with its own database. There's one more modernization step to consider: going [cloud-native](https://aws.amazon.com/what-is/cloud-native/). That includes using an event-driven architecture and managed, serverless cloud services.

7. ### Move to event-driven, serverless architecture
    

You'll want to refactor to an [event-driven architecture](https://aws.amazon.com/what-is/eda/), in which small decoupled services components publish, consume, or route events. You could end up with event-driven microservices, and/or serverless functions.

Messaging is paramount to interconnecting your components. Amazon Simple Queue Service (Amazon SQS) and Amazon Notification Service (Amazon SNS) can give you durable messaging and push-based many-to-many messaging. Even better, consider an event bus such as [Amazon EventBridge](https://aws.amazon.com/eventbridge/) which lets you connect event publishers and subscribers.

You'll want to use serverless AWS services to host your code. For containerized microservices, that means AWS App Runner, Amazon ECS on [AWS Fargate](https://aws.amazon.com/fargate/), or Amazon EKS on AWS Fargate. For serverless functions, that means [AWS Lambda](https://aws.amazon.com/pm/lambda). You can use the Serverless Application Model (SAM) to accelerate your serverless function development. SAM gives you templates and a CLI that improve the developer experience.

If you're playing .NET Jail Break, this corresponds to *Level 7: Move to event-driven, serverless architecture.*

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709224234189/aad217be-5dad-4bf1-9fd8-d1e1241fa0ad.png align="center")

## Summary

In this post, we've looked at a number of ways in which your older .NET Framework and SQL Server application can gain freedoms in the cloud. Did the .NET Jail Break game help you learn how to accelerate modernization on AWS? Share your comments. For more information on .NET modernization at AWS, visit the [.NET Developer Center Modernize page](https://aws.amazon.com/developer/language/net/modernize).