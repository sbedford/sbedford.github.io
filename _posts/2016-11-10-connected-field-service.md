---
layout: post
title: Microsoft Connected Field Service
byline: Lets look under the hood.
---

# Introduction

I was lucky enough to present recently at [Dynamics Day](http://www.empired.com/Our-Insight/Dynamics-Day-2016/) on a couple of topics including the [Dynamics 365 applications for Field and Project Service](http://www.empired.com/Our-Insight/Dynamics-Day-2016/enhance-your-productivity-with-dynamics-365-project-services-and-field-services/) and a session on the [Internet of Things and CRM using the all-new Connected Field Service solution](http://www.empired.com/Our-Insight/Dynamics-Day-2016/Internet-of-Things-and-CRM/).  

As this was a topic which seemed to really resonate with our attendees, I decided to put a few more thoughts down here, dig into the details and attempt to answer the questions posed above!

_Connected Field Service (CFS)_ as a concept has been around for a while.  

The premise is simple - take the "things" that we have in our business, connect them, monitor them and respond when there is a problem (or before it!).  

However, while CFS is something that has been spoken of in the Microsoft Dynamics platform for a while - to be fair its been a little light on in detail.  What is it? How does it work? What can it do?

# Business Context

So people tend to get a bit carried away when talking about the Internet of Things.  From internet enabled lightbulbs, to wine bottles and nappies there have been some pretty crazy consumer goods released.  

However, as the hype cycle begins to normalise there are opportunities for organisations to leverage the technology in practical ways.  Whether using IoT to optimise our operations and assets, or identifying new and exciting revenue streams by using technology to deliver excellent customer service.

At a high level, we are a looking for a platform which can tick a number of boxes from a busines process perspective: 

1. Manage the devices in our network with appropriate governance and security
2. Handle different devices and protocols
3. Adjust the thresholds and rules used to identify fault conditions
4. Ensure that data is in the appropriate system - keep the historical, reporting and operations systems independent.
5. Not only receive service alerts, but send commands to devices to take action
6. Monitor in real-time but analyse at the macro level

All of these goals above have different impacts on the bottom line for businesses.  Having a _flexible device management_ platform allows best of breed devices to be sourced regardless of manufacturer while ensuring that _security_ is maintained so as not to compromise key business systems.  

Having the ability to _adjust the rules_ which identify fault conditions easily allows us to be more agile in how we manage our systems and either introduce or remove rules as necessary.  	 

Finally, it's all about the outcomes right.  We need a system which can take these alerts and respond in a fast and accurate way.  Whether that reaction is to send a reset command to a device or schedule a field technician to remedy the problem, the system needs to know what device has a problem, its place in the physical topology of the environment and what is the appropriate response to the problem.

Thankfully, the _Connected Field Service_ solution from Microsoft gives us the tools to meet all these chellenges.

# Technical Components

I'll start the discussion on what the Connected Field Service solution is by saying that while the name shares pedigree with the _Dynamics 365 for Field Service_ application, the CFS solution is much more than a pure Dynamics play. The full capabilities of the Azure, Dynamics and Office 365 are used to deliver an device, data and business process system which is truly a _Microsoft platform_ solution.

I've presented a simplified version of the [architecture](https://msdn.microsoft.com/en-us/library/mt744253.aspx) released by Microsoft below which focuses on the key elements and the flow of data between them.

![Connected Field Service Overview](/images/2016-11-10-cfs-overview.png "Connected Field Service Overview")

We will focus on the three areas across the top:

1. [Connectivity](#connectivity) - how our devices connect
2. [Analytics and Intelligence](#analytics-and-intelligence) - how we analyse and identify faults
3. [Business Process](#business-process) - how do we react when a fault is identified

## Connectivity

When implementing Connected Field Service solutions, _Connectivity_ will always be a challenge and generally involves addressing a number of areas:

* Security
* Interoperability
* Throughput/Scale 

In the Microsoft Connected Field Service solution, the Azure IoT Hub is the default platform for managing field device connectivity.  Microsoft has a heap of information  on the [Azure IoT Hub](https://azure.microsoft.com/en-us/documentation/articles/iot-hub-what-is-iot-hub) and I'm not going to reproduce it here.  However, I will focus on how the IoT Hub addresses the above key challenges.

### Security

There is a huge amount of content to cover when talking of security in an IoT concept.  You need to be aware of physical device security, edge/gateway authentication and authorization, cloud asset security and finally business service system securities.

What is important is that with the Azure IoT Hub, you are provided with a wealth of tools to secure your deployment.  Whether your talking device registration, device identification tokens or X.509 device certificates you can ensure that only those devices in your control are able to submit data to the network.  

Further, the industry leading security practices available on the wider Azure platform provide the ability to secure access to platform and software services deployed.  

### Interoperability

The Azure IoT Hub natively supports three main communication protocols:
* AMQPS
* MQTT
* HTTP

However, there are plenty of devices out there that are unable to talk using these protocols.

To ensure that the platform can be used with low-power or other devices, gateway deployment patterns can be used.  Gateways essentially act as a protocol mediator to broker a channel when one cannot be natively established.

Given the above, its important to understand the security implications and make sure that the devices, gateway and IoT Hub are all secured and appropriately managing device identification.  

### Throughput/Scale

IoT solutions are npt really designed to be deployed at a small scale.  Any platform solution  needs to be able to scale to support a huge volume of devices and messages and act and respond rapidly.

The Azure Iot Hub has some similiarities with the Azure Event Hub service in feature function, however the IoT Hub is designed to support a much higher number of simultaineous devices.

To put hard numbers to the scale debate, the base tier for an Azure Event Hub (S1) [supports](https://azure.microsoft.com/en-us/documentation/articles/iot-hub-scaling/) up to 1.5GB/day/unit which is ~400,000 messages/day/unit.  The unit is the important part as you can use the elastic scaling capability to increase your throughout as required.

## Analytics and Intelligence

Connecting our devices and sensors is one thing.  Being able to take intelligent actions based on current or projected values is a whole different story.  

Generally when talking Connected Field Service, there are a few different models of operations which use the data slightly differently.  Broadly, these are:

* Scorecard
* Remote Monitoring
* Predictive Maintenance

Regardless of the method of operation above, the technology stack is broadly the same; using the following Azure services to both monitor data and support automated and manual decision making processes:

* Stream Analytics
* Azure Machine Learning
* Power BI

### Scorecard

A __Scorecard__ solution is generally the simplest method of supporting manual decision processes in IoT environments.

The principle in this method is to capture all data provided to the Azure IoT Hub and summarise this data on a dashboard using scorecards or other visualisation techniques.  This dashboard can then be used to identify areas which need support, maintenance, replacement, etc.  

_Streaming Data Set_ capabilities in Power BI can be utilised to great effect to show the real-time status of sensors as the data is reported.

This option is generally the simplest to implement however it is technically not a true "Connected Field Service" solution as the fault conditions would have to be manually reported and managed.  It's connect Device to Analytics, but no further - obviously still valuable though.

### Remote Monitoring

The __Remote Monitoring__ method builds upon the Scorecard mechanic and is the first real foray into true _Connected Field Service_.

Implementing a Remote Monitoring solution involves the _Stream Analytics_ service (and is a part of the default Connected Field Service solution).  This service is used to analyse a real-time, continuous stream of telemetry data and report when a particular fault condition is identified.

![Power BI Streaming Dataset](/images/2016-11-10-pbi-streaming.png "Power BI Streaming Dataset")

_Stream Analytics_ is interesting because not only can you look at the currently received message:

> Is the current oil pressure greater than 70?

but you can also _temporally_ group messages to look at patterns over time!

> Is the average oil pressure over the last 5 minutes greater than 70?

Talking specifically on the Connected Field Service solution,  the _Stream Analytics_ jobs identify faults and trigger _Azure Logic Apps_ to push these  _Service Alerts_ from the event stream into the _Dynamics 365 for Field Service_ application.

### Predictive Maintenance

We have progressed from using data to manually identify faults, through to having the platform identify our faults for us.  However, what if we could identify faults before they affect our business, our customers or both?

__Predictive Maintenance__ is all about using the vast datasets provided by IoT solutions and Field Service applications to look for patterns which identify a future fault which can be remedied without unscheduled down-time.

![Machine Learning Remaining Life Model](/images/2016-11-10-ml.png "Machine Learning Remaining Life Model")

This capability is delivered through the _Azure Machine Learning_ platform which allows data scientists the ability to explore, develop and train models to predict faults.  Once again, identified faults are fed into the _Dynamics 365 for Field Service_ application to be scheduled and fixed at a mutually agreeable time.

## Business Process

So far this post has spoken about __Connected Field Service__ while talking about concepts which are used predominately used in _Internet of Things_ scenarios.  

The reason the solution is called __Connected Field Service__ and not _Internet of Things_ is that the Dynamics 365 platform is used as the engine or brains to ensure that _any_ identified service alert is handled in a timely and appropriate manner.

However, before we talk about specific features or benefits of using Dynamics 365 in this way, there are a number of key business capabilities which must be  supported alongside of traditional field service:

* Device Management
* Service Alerts and Commands
* Field Service

### Device Management

Device Management generally encapsulates the ability to:

* Register new devices
* Monitor existing devices
* Revoke access

Once the _Connected Field Service_ solution is deployed into Dynamics 365, a new entity is created in the platform: __IOT Device__.

__IOT Device__ records integrate with the _Azure IoT Hub_ to support one click registration using nominated device ids.  

The version that currently discussed in Microsofts architecture is functional however, I am waiting to see improvements around security here specifically on device id generation and authentications.  Bear in mind, CFS is still __not generally available and subject to further revision__.  The X.509 device authentication function is a recent addition to the Azure IoT Hub service so I'm sure this will be integrated soon.

### Alerts and Commands

Once the device has been registerd, the _IOT Device_ record becomes the parent for __Service Alert__ and __Command__ records.

_Service Alert_ records are created by an _Azure Logic App_ triggered when a fault is identified by either a Stream Analytics or Azure Machine Learning job.  _Service Alerts_ are received in a JSON format and the Dynamics application has the ability to parse and promote elements of the free-text message into the structured _Service Alert_ record.

{% gist 9fc6b344fc428d8ec1d062d7eecb10b2 CFS_ServiceAlert.json %}

The alert represents a fault condition reported by a sensor and feeds into the _Dynamics 365 for Field Service_ app's __Work Order__ entity based on defined business processes specific for your workload.

{% gist 9fc6b344fc428d8ec1d062d7eecb10b2 CFS_Command.json %}

A _Command_ record is used to send a message to a device to perform an action.  These records are again picked up by an _Azure Logic App_ which transmits the message to the device through the _Azure IoT Hub_.  Interestingly, the device must poll the Azure Iot Hub for messages waiting to be received.

### Field Service Capability

So far we can:

1. Register a Device 
2. Receive an Alert
3. Send a Command
4. Create a Work Order

However, to deliver a fully integrated _Connected Field Service_ solution we also need to understand and execute our business processes to respond to the _Service Alerts_ as they occur.  Basically, we need to weave the above capabilities into a story - when something happens, what do we need to do.

This is really no different to solving any other business problem.  We need to understand our domain, our constraints and our options. Typical questions include

* Whats the severity of the alert?  Do we need to watch or act?
* Is there a command which could resolve the problem automatically?
* Which field technician is best suited to address the problem?
* Is that field technician available?

Using the new capabilities of the _Dynamics 365 for Field Service_ application such as the Automated Scheduling Engine, alongside traditional Dynamics 365 configuration tools we can execute scenarios like the below:

* When I receive a Service Alert for a Device of Category Temperature
    * If the temperature is greater than 90
        * Send a Command of type Reset 
        * If a subsequent Service Alert is received
            * Create a Work Order of type _Emergency_
            * Let the automated scheduling engine pick up the Work Order and find the right resource that is available.
            * Notify the Field Tech on their mobile device of the job 
            * Field Tech gets onsite and resolves the fault
       * If the service fault is resolved
           * +1 to the KPI's! We resolved the fault automatically!
    * If the temperature is greater than 70
        * Display the Service Alert on a dashboard
        * Send a notification to monitor the situation.

# Wrap Up

This post has got really long but there was a huge amount of area to cover.

When implementing a Connected Field Service solution, you need to think about how your devices will connect, how you can secure their connection to your system (and secure them physically!), how to ingest the data, identify or predict faults, visualise data in real-time or summary, how to store the volume of data, how to integrate the data into your field service processes, how to make best use of the new capabilities of devices to optimise your field service operations. 

There is a __stack__ to think of.

What I find really interesting is that the solution brings to bear many of the exciting areas of the Microsoft platform.  Dynamics 365, IoT, Power BI, Machine Learning, Stream Analytics are all really powerful and exciting technologies which, when put in the context of a Connected Field Service domain, give us a platform to deliver unique and compelling solutions.