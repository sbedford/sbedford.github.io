---
layout: post
title: ADX Studio Series Part 2 - Security
byline: Part 2 in my series on ADX going over security within the portal and back into Dynamics CRM
---

Security is a key consideration when designing your ADX Studio portal.  

* Who do you let in?
* How do you verify their identity? 
* How do you stop users seeing data they shouldn't?

When we talk about security in any context we are generally talking about two things:

1. Authentication - making sure users are who they say they are,
1. Authorization - ensuring that only users who should have access, do have access.

The purpose of this post is to talk around the capabilities of ADX Studio specifically regarding security.

##Overview

Before we dive into the details for authentication and authorization, its worth taking some time to discuss some of the key considerations of security in ADX Studio.

__All users__ in an ADX Studio portal are identified within Dynamics CRM as Contact records - this is a critical point when discussing not only authentication but also authorization.

When the ADX Studio solution is deployed to a Dynamics CRM instance, a number of additional fields are created against the Contact entity to store information required by ADX for authentication.

Once you have the solution deployed and your Contact entity is ready to go, how do you get your users into the system?  ADX Studio offers both invitation and registration processes (both customisable of course) and you always have the ability to manually create Contacts (through Identity Management tooling?) if required.

One final point, is that like almost everything in ADX the security configuration is stored within ADX Studio.  So while there may be an ASP.NET portal deployed, the actual security configuration is contained within extensive Setting records in Dynamics CRM - pretty neat.

##Authentication

There are three main ways in which user credentials are authenticated in an ADX Studio portal:

##Forms Based Authentication

As in ASP.NET, forms based authentication provides a secure mechanism for external users to provide their username and password which is submitted to the server for authentication.  The actual authentication is performed against the Contact entity in Dynamics CRM which stores the hashed password in encrypted, hidden fields.

###Azure Access Control Services (ACS) 

Azure Access Control Services provides identity federation services over Windows Azure Active Directory (AAD) and other OAUTH based identity providers.

In this mode, users do not provide their credentials to the ADX Studio portal but are redirected to the login page for ACS.  The users credentials are verified inside ACS and appropriate tokens issued for the ADX Studio portal to validate that the user has been authenticated. 

In this mode, the Contact entity does not store a password, however does store information which maps the 
Contact to the identity stored within ACS.

###Open Authentication (OAuth2)

OAuth2 support is provided by the portal when support is required for a wide range of registered identity providers including Facebook, Google, Microsoft, and ACS is not available.  The platform also provides the ability to register custom Open Id providers for situations where additional identity providers are required.

Similar to Azure ACS above, the ADX Studio portal never sees the users credentials and is only provided a token which the portal has been configured to trust. 

##Authorization

Content authorization in ADX Studio uses a permission based model revolving around two key concepts:

###Site Map Permissions

Site Map permissions are basically just permission which provide access to a given Web Pages and other resources insde the ADX Studio sitemap.

Permissions are assigned to Web Roles which Contacts are then assigned to - these effective permissions for a user define the resources which are available to them within the menu and within different sections of a page.

Not particularly interesting but important.

###Entity Permissions

Entity Permissions are arguably more important however are much more interesting.

Entity Permissions are assigned to Web Roles which define, for example, that a user in the Sales Manager Web Role can create and update Opportunities. These permissions define not only which entities a user can access, but also the fine grained actions which can be performed on records of that entity.  It is the fine grained actions which make Entity Permission particularly interesting.  

All permissions have a _Scope_ attribute which describe what records of the given entity a Contact with the Entity Permission is allowed to access. The following Scope values are supported:

####Global

Allows access to all records of the Entity described in the Entity Permission.  

Useful when you have a set of data which you only want exposed to specific a Web Role(s), however there is no record level security required.

> Example: In a partner management scenario, your Company may offer Events which you want visible to all Partners.  

> While the Event details may not be public, you want all Contacts in the Partner Web Role to see available Events.

####Account

Allows the Contact with the Entity Permission to view records which they own, as well as records which are owned by the parent Account with which the Contact is related in Dynamics CRM.

> Example: In a partner management scenario, you will have your users (Contacts) associated with the partner Company (Account). 

> If you have a set of open support calls for that Account, using this Scope ensures that Users only see support calls which are associated with their company.  

####Contact

Allows the User to access only those records which are associated (either directly through ownership or indirectly by relationships) with their Contact in Dynamics CRM.  

> Example: In the partner management scenario above, using this Scope will restrict users to only seeing those records which they were responsible for creating, or have been assigned to within Dyanmics CRM. 

####Parental

Parental relationships are kinda tricky.

This permission scope basically allows you to define permission structures for related entities - if you have access to the parent you can access nominated child records.

> Example: Continuing the partner management scenario, in a support call situation there may be Downloads (files, etc) which are only available to Partners when associated with a Support Call.

> Using the parental relationship allows a security model which grants access to Download records which have been associated with a Support Call record that the Contact has access to.

The documentation on ADX's documentation portal gives another  [example](https://community.adxstudio.com/products/adxstudio-portals/documentation/configuration-guide/entity-permissions/parental-scope-example) to explain Parental scopes.