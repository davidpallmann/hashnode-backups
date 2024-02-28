---
title: "Deploy Apps Your Way with Elastic Beanstalk"
datePublished: Wed Feb 28 2024 00:37:23 GMT+0000 (Coordinated Universal Time)
cuid: clt52hkc6000009l642pkauua
slug: deploy-apps-your-way-with-elastic-beanstalk
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1709080986128/4033994a-91be-48ad-9d71-82ab255871ee.png
tags: aws, elastic-beanstalk

---

[AWS Elastic Beanstalk](https://aws.amazon.com/elasticbeanstalk) is a managed service that handles the details of deploying and managing applications on Amazon Elastic Compute Cloud (Amazon EC2). For an introduction and getting started tutorial, see [Hello Beanstalk](https://davidpallmann.hashnode.dev/hello-beanstalk). In this post, we'll review the different ways you can handle deployments with Beanstalk.

## Extensive Deployment Options

Beanstalk supports a rich collection of deployment options for updating your environments, called [deployment policies](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.deploy-existing-version.html). You can configure an environment's deployment policy in the AWS console.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709072034448/f4d6591d-fa3b-48f1-a2a5-1395383e2a57.png align="center")

Let's review the deployment policies one by one.

### All at Once: The Brute Force Update

All at once deployments deploy the new version to all instances simultaneously. This means all instances in your environment will go out of service for a short time during deployment. There's no additional cost with this form of deployment because no additional instances are allocated. This is the fastest way to deploy, but only use it if you can tolerate brief downtime. I recommend it for non-production environments.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709074752362/2990a91e-d100-40cb-aa60-ab18641cead5.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709074729115/a185e506-2fb8-4cfa-aa89-a83f58b471e6.png align="center")

In the console, select the **All at once** deployment policy. There are no other settings to configure for this policy.

![Deployment policies and settings - AWS Elastic Beanstalk](https://docs.aws.amazon.com/images/elasticbeanstalk/latest/dg/images/environment-cfg-rollingdeployments.png align="left")

### Rolling Updates: Your Table is Next

Rolling updates avoid downtime and minimize reduced availability, at the cost of a longer deployment. This is a good option if you can't handle any downtime. Rolling updates work by updating batches of instances at a time. As each batch is updated, its instances go temporarily out of service.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709074525097/dcc50744-1a41-4cfe-9556-b4637945f1bb.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709074551305/b19bf8a9-5ae8-4767-b66c-200fcee6ed24.png align="center")

Through settings you can control the **batch size**. You can either set a fixed number of instances, or specify a percentage of the total number of EC2 instances in the auto scaling group.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709071973316/cb59452f-eb0a-4a46-9e6a-56da61674e12.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709072005889/6e4326e4-261a-400e-8030-69a8a5ad746a.png align="center")

While a rolling deployment is in process, both versions of the application are running simultaneously. Your capacity is reduced by the one batch that is being updated. If a Rolling update fails, then an additional rolling update is necessary to roll back the changes.

### Rolling with Additional Batch: I Want It All

This is a variation of a rolling update that creates an extra batch in order to maintain full capacity throughout the update. You end up with the same number of instances you started with, but your capacity is not reduced during the deployment thanks to the extra batch.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709076413091/f9c00521-8865-46bb-bb51-0ea013606798.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709076433223/bb48c496-7efc-43f5-bdf6-a62ee189eb06.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709076452266/96192775-bdaf-4868-ba11-39030d89ddfc.png align="center")

You configure this policy with the same settings as Rolling to set the batch size.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709072267421/d4f99789-cbff-4841-9400-fca9ffbbe77e.png align="center")

There's a small additional cost for the extra batch during deployment.

### Immutable: Behold, All Things Are Made New

[Immutable environment updates](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/environmentmgmt-updates-immutable.html) are an alternative to rolling updates. Beanstalk creates brand-new instances in a new auto scaling group to deploy the new version to, then removes the older instances after a successful health check. Immutable updates ensure that configuration changes that require replacing instances are applied efficiently and safely.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709076922987/5c527cc6-0ced-47b8-b980-66cc641c6aff.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709077427892/5c1465b1-6b2f-411d-a045-4669d3973fcf.png align="center")

Once the new version is deployed to the new instances, the load balancer cuts over to the new auto-scaling group and the original instances are deleted.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709077454504/5d664577-f560-4efb-b186-c12ab3d85ffd.png align="center")

As with Rolling updates, you specify a batch size, either as a fixed number of instances or a percentage of the auto-scaling group.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709072554596/65fcdc57-6db6-4800-a845-94ddd0ce740e.png align="center")

Immutable updates take the longest amount of time and are the most expensive, as you are temporarily doubling your instances. However, immutable updates are low-impact on failure and have a quick rollback. If an environment update fails, the rollback process requires only terminating an auto scaling group.

### Traffic Splitting: The Canary Sings

Traffic-splitting deployments allow you to perform canary testing. The new version is deployed to a fresh group of instances. Traffic is temporarily split between the existing application and the new one. If a health check passes, remaining traffic is shifted over as well.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709078806321/33313de7-b70b-461f-83ee-0ca05a835488.png align="center")

Through settings you control the initial **traffic split** percentage of incoming traffic that is shifted to the new environment, and the **traffic splitting evaluation time**, the number of minutes to evaluate health and proceed to shift all traffic over.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709073337908/e46deb24-f601-4de3-965f-5491d538ad67.png align="center")

### Blue-Green Deployment: The Shoe is on the Other Foot

The blue-green deployment pattern deploys the new version to a separate environment. You can then swap the URLs of the two environments. This redirects traffic to the new version instantly. You can roll back just as easily, with another URL swap.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709140293543/4b20e2db-e5d1-404f-bccf-480623144d7a.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1709140323761/14c0f5d3-b2ee-4745-a135-6a57afcbee62.png align="center")

To perform a blue-green deployment, you clone your current environment in the AWS Beanstalk console or launch a new environment. Then, you deploy the new version to the new environment. After testing the new environment, you swap environment URLs.

![
Swap environment URL page
](https://docs.aws.amazon.com/images/elasticbeanstalk/latest/dg/images/aeb-env-swap-url.png align="left")

You hold on to the older environment for as long as you need to, knowing that you can roll back quickly if a problem arises with the new version.

## Summary

As we've seen, AWS Elastic Beanstalk supports more methods of deployment than most compute cloud services. Choose the deployment approach that best suits your combined requirements for deployment time, downtime, impact of failure, and cost.

Here's a summary of the deployment options and their characteristics.

| Deployment Policy | Downtime | Deploy time | Impact of Failure | Rollback process |
| --- | --- | --- | --- | --- |
| All at once | down | 1-shortest | downtime | Manual redeploy |
| Rolling | zero downtime | 2-long | single batch out of service, earlier successful batches running new version | Manual redeploy |
| Rolling with an additional batch | zero downtime | 3-longer | minimal if first batch fails, otherwise similar to Rolling | Manual redeploy |
| Immutable | zero downtime | 4-longest | minimal | Terminate new instances |
| Traffic splitting | zero downtime | 4-longest | percentage of traffic routing to new version temporarily impacted | Reroute traffic and terminate new instances |
| Blue/green | zero downtime | 4-longest | minimal | Swap URL |

## Learn More

Below are some useful AWS and community resources for Elastic Beanstalk deployment.

From the AWS Elastic Beanstalk developer guide:

[Deploying applications to Beanstalk environments](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.deploy-existing-version.html)

[Deployment policies and settings](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.rolling-version-deploy.html)

[Blue/green deployments with Elastic Beanstalk](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.CNAMESwap.html)

Community resources:

video: [Elastic Beanstalk Deployment Options](https://www.youtube.com/watch?v=BU54m8nEo64) by Digital Cloud Training