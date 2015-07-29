---
layout: post
title: ADX Studio Series Overview
byline: First post in the ADX Studio series giving an overview of the product and the key parts of the solution.
---

The first post in my series on ADX Studio is going to give an overview of what the solution sets out to achieve, how it works and what you need to do to deploy a portal.

__Disclaimer__: Pretty much everything I'm saying here has been said better on the [ADX Studio Community Portal](http://community.adxstudio.com/products/adxstudio-portals/).  Go check it out if I'm not making sense.

So, ripped off straight from their [website](http://www.adxstudio.com/adxstudio-portals/), ADX Studio provides Dynamics CRM integrated portals which aim to transform Dynamics CRM into a powerful application platform.

Thats cool, but really what they're saying is that ADX provides a web site which sits in front of Dynamics CRM and,

* Displays information stored within your CRM, and
* Allows data captured in the portal to be stored within CRM
* Leverage the existing investment in display, business logic and processes within CRM.

Think of an ADX Studio Portal as a custom ASP.NET web application which talks to the CRM web services to render forms and entities stored within CRM.  As it is a custom application, you have the ability to tweak the look and feel and structure of the application to handle most scenarios possible.  Also, in scenarios where configuration of the product does not meet the requirements, you can write custom ASP.NET code to meet any requirement.

ADX Studio is unique in that _pretty much_ all of the content for the site is stored within Dynamics CRM.  What this means is that there are now two places where content can be maintained:

1. Created in the front-end through tools in the portal
1. Created in Dynamics CRM by creating/updating entities.

![Dynamics CRM Portals](/images/2015-05-18-adx-crm-portals.png "Dynamics CRM Portals")

When you publish the solution into your Dynamics CRM instance a new area called _Portals_ is created which contains a whole bunch of CRM entities describing your portals.

I'll talk briefly about some of the key entities, and dive into a bit more detail in later posts.

![ADX Studio Model](/images/2015-05-19-adx-model.png "ADX Studio Model")

### Web Site

The website entity is a top-level parent container for all of the content of the portal.  A single CRM organisation can support multiple ADX portal web sites.

### Web Site Binding

The web site bindings control how the ASP.NET application knows which Web Site to source its content from. The web site binding describes the site name and virtual directory , however also stores any host name and/or port entries for the site.

###Web Page

The web page entities are where the magic starts to happen in an ADX portal.  Web Pages are placed under the web site and store a stack of details on:

* Content to display
* Security settings
* The page template to use
* The Web Form, Entity List, or Entity Form to display  

Pages can also contain other pages to create heirarchies of content which are natively rendered through ADX's menu controls.

### Page Template  	

All web pages need to have a page template which defines how the content of the page (referred to as the _Copy_) and also other controls (i.e. Web Forms) are rendered.  Page templates can be shared across numerous pages allowing, for example, a blog post template to be used anywhere a Blog post is required to be rendered.

Page templates can be one of two types - either a Rewrite Template, or a Web Template.  Both are discussed below.

### Rewrite Template

A rewrite template basically tells ADX to go and look for the template content inside the ASP.NET application.  Its very similar to a URL re-write module, hence the name.  

![ADX Studio Rewrite Template](/images/2015-05-19-adx-rewrite-template.png "ADX Studio Rewrite Template")

### Web Template  	

A Web Template is [Liquid template](http://liquidmarkup.org/) where the rendering logic is contained within the CRM entities themselves, rather than within an ASP.NET web application.  Liquid is a lightweight, open-source rendering engine which allows designers to embed small scripts to layout the content of a page.

{% gist 5b9e9c38e2f5d313b5e3 LiquidTemplate.html %}

This functionality, while relatively new, provides powerful opportunities to build dynamic templates quickly through the web application or Dynamics CRM.

### Content Snippet 

Content Snippets provide a central place to manage literal content that can be re-used throughout the portal in your page templates.

Usages for content snippets include error or warning messages, as well as header and footer templates.  

Content snippets can include HTML content as well as  liquid templates which provides opportunities to personalise information based off the variables available within the template.

### Entity Form 

Entity Forms are the first of the real CRM centric constructs in an ADX portal.  An entity form allows for a form, deisgned in CRM to be rendered in the ADX Studio portal without modification.

In the image below, you can see the Dynamics CRM form on the left, and the rendered ADX Studio form on the right.  Notice the asterisk after the Last Name field - field requirements are carried over automatically from CRM to ADX!

![ADX Studio Entity Form](/images/2015-05-19-adx-entity-form.png "ADX Studio Entity Form")

This feature, along with the Entity List and Web Form constructs, are the meat of the solution and where it gets most of its value from.  Frankly, its a pretty cool story.  

Build once, deploy in both.

### Entity List 

Entity Lists provide capability to display a CRM view on an ADX page.

In a similar fashion to Entity Forms above, you define the entity view using the standard tools available in Dynamics CRM, and then configure the Entity List entity and bind it to a Web Page!

![ADX Studio Entity List](/images/2015-05-19-adx-entity-list.png "ADX Studio Entity List")

Notice in the figure above, you get a styled grid with click through links as well as the ability to create new entities and search and filter results.  Pretty neat.

### Web Form  

Web Forms are the last entity I'll talk about in this post and there a little complicated.  Basically, Web Forms provide multi-step, wizard style interactions with an ADX Portal.

Think of Web Forms as the ability to chain multiple entity forms together to complete a single business process - like a checkout scenario, or a complaint handling process where you may need to facilitate creating more than one entity, or involve a qualification process before the interaction can continue.

The Web Form entity is probably one of the more complex having many attributes and child elements to describe the process. We'll talk more about these later. 
