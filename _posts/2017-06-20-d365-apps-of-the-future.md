---
layout: post
title: Building the Dynamics 365 application of the future
byline: Microsoft have shown the new model for building Dynamics 365 applications and its awesome.
---

Microsoft are beginning to realise the vision first described back when Dynamics 365 was launched.  

This vision is to provide Customers with immersive and feature rich applications which integrate seamlessly into the day to day.  Leveraging capabilities which extend beyond the box of traditional line of business applications by bringing together the power of the Azure platform - whether this be mashing up data across the enterprise or using new user interface patterns such as bots to make these applications easier to use and more relevant.  

Further, it is how these capabilities are delivered which is important bearing in mind the build and maintain costs of what traditionally been seen as the risky and/or expensive areas of application customisation and integration.

All that aside, we are now starting to see what these Dynamics 365 applications of the future will look like.  Leveraging the [Common Data Service](https://powerapps.microsoft.com/en-us/guided-learning/learning-common-data-service), [Power Apps](https://powerapps.microsoft.com/en-us/) and [Azure Functions](https://azure.microsoft.com/en-us/services/functions) to deliver a platform of applications which are powerful, tailored, immersive and most important of all easier and safer to customise.

# History

Traditionally, extending the standard functionality of a Microsoft Dynamics product has involved some degree of configuration or custom development (customisation).  

In the configuration realm, you are effectively using the application to configure itself.  Either creating new "entities" or configuring a particular business process or workflow to deliver an outcome for users.  

When customising, you were writing .NET code bound to the application to achieve more complex or long-running processes which may have leveraged other system API's or interfaces to complete the process.

# Brave New World

We have seen recent shifts towards more integration points with platform services offerred by the Azure stack (such as native Azure Service Bus integration) and this appears to be the pattern that Microsoft will build and extend upon moving forward.  By evolving and improving the touch points between these platforms, Microsoft are effectively opening up a world of opportunities to customers to build the application of the future.

This pattern is highlighted most obviously in Microsoft newly spruiked HCM solution, Dynamics 365 for Talent.  

![Dynamics 365 Customisation Overview](/images/2017-06-20-d365talent-overview.png "Dynamics 365 Customisation Overview")

What has been released shows so far a strong, native integration between the Common Data Service, Power Apps and Azure Functions, which can be used to extend the solution beyond what you get out of the box.

While there will be a level of configuration available, for example, to define the stages and required data for the on-boarding process, it appears that to do more than that you will need to leverage the power of the wider Microsoft platform - specifically _Power Apps_ and _Azure Functions_ all backed by the _Common Data Service_.

# Common Data Service

The _Common Data Service_ underpins the extensibility model.  

It appears that the data model itself has been extended to include key entities from the Talent solution and is preconfigured with a data integration project to allow the data to be synchronised both ways.

It remains to be seen whether there will be access to the underlying database or API style access to update records, however given what we've seen so far I'd guess the answer will be no.

This is an important distinction as it places the _Common Data Service_ as the critical service for extending the system - if you need to extend the system using one of the methods described below you will do so using the data stored in the Common Data Service.

This obviously raises questions as to the timeliness of the synchronisation however issue should be resolved as the data integration framework comes out of preview. 

# Power Apps

Power Apps are positioned by Microsoft as a rapid application development platform aimed at power users within an organisation.  These aren't aimed at replacing platforms such as Dynamics 365 or other services in the wider Microsoft or Office 365 stack but more to augment and support in those scenarios where organisations wish to, for example, build an "app" which brings together data from both Dynamics 365 and maybe a SharePoint list.

They are designed to be built by business users (who are comfortable writing at times excruciatingly complex excel formulas....) and can read, write and delete data from a variety of sources including the Common Data Model.

You can write a whole post of the capabilities of Power Apps, but in the context of what we have seen so far its pretty clear that Power Apps will be a very important played in the extensibility model for Dynamics 365 applications in the future.

![Dynamics 365 for Talent Power Apps Integration](/images/2017-06-20-d365talent-powerapps.png "Dynamics 365 for Talent Power Apps Integration")

I don't think that the ability to "configure" an application will go away anytime soon, we will always need to tweak or adjust the rules.  However, the possibilities really are endless with this one as we now have Power Apps which are __natively integrated__ and __contextually relevant__ to the record in focus.  

# Azure Functions

The final method discussed for extending Dynamics 365 applications is really cool and very powerful and is targetted at those scenarios where we need to write some code to implement more complex pieces of logic.

For those who haven’t used them before, _Azure Functions_ provide the capability to run code in the cloud in a very cost efficient manner.  You don't need to run up containers to "reserve" compute power and you only pay for the time your code spends running.  

![Dynamics 365 for Talent Azure Functions Integration](/images/2017-06-20-d365talent-azurefunctions.png "Dynamics 365 for Talent Azure Functions Integration")

Where this type of solution applies in the landscape of Dynamics 365 really depends on what touch points are provided by the platform itself.  

Currently, we have the ability to run asynchronous processes in Azure Functions triggered by message queues.  However there are plenty of options to build upon this using a webhook style pattern where Dynamics can call these functions as required to augment the standard logic and processes.

How cool would it be to use Azure Function for plugins or workflow steps?  Not only does it get us out of the sandbox in online scenarios but further it separates out the customisations from the Dynamics 365 platform and provides additional support for release management and dev-ops scenarios which are more in line with traditional application development.

I think from what we've seen so far, this is the approach that Microsoft are taking and its going to be really flexible and incredibly powerful.

# Summary

I think this is a really exciting shift from Microsoft.  

I'm sure there are going to be some situations that won’t be possible in this early stage of the lifecycle but the approach is exciting for a number of reasons.

1. It provides a flexible and powerful method to extend the Dynamics platform without modifying the applications themselves.
2. It opens up new dev-ops capabilities to handle more advanced release management and monitoring scenarios.
3. A reference architecture for modern Dynamics application is starting to emerge.

