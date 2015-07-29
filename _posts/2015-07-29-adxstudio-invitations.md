---
layout: post
title: Provisioning Federated User Identities in ADX Studio
byline: Runs through how to use the invitation workflow to link Contacts and Federated Identities.
---

As I've spoken about before, ADX Studio supports a number of different [authentication](/adxstudio-part-2) options however when your portal is setup to use federated identites (OAUTH, ACS, etc) provisioning can be a bit confusing.

The long and the short of it is that when your users authenticate using an external OAUTH provider, the identity claims provided by the federated identity provider need to match to a Contact record in Dynamics CRM.

While it is possible to configure this information manually through the _Portal Contact_ form of the Contact entity, a MUCH easier way to link identities in this fashion is using the pre-built Invitation processes added to Dynamics CRM by the ADX Studio solutions.

#Invitation Process

The invitation process starts by creating a Contact record in Dynamics CRM which will identify the users interactions within the CRM system.

![New Portal Contact](/images/2015-07-29-portal-contact-new.png "New Portal Contact")

(Note in the image above the empty Web Authentication section)

Once the Contact has been created, the _Create Invitation_ work must be executed.  

![Invitation Entity](/images/2015-07-29-invitation.png "Invitation Entity")

This process will result in an email being sent to the contact notifying them that they have been invited to join your portal and includes a link to complete the process.

![ADX Studio Invitation Email](/images/2015-07-29-email.png "ADX Studio Invitation Email")

Once the user receives the email and click the link, they will be presented with a page which asks them to either create an account (if portal credentials are in use), or login with the allowed federated identity providers.

![ADX Studio Registration](/images/2015-07-29-registration.png "ADX Studio R")

The final step is performed once the user has provided their credentials and the login process completes.  The login process updates the web authentication section of the Contact record with both the Username and the Identity Provider.

It is these two fields which allow the ADX Studio login process to tie an federated identity claim to a Contact in CRM - as mentioned earlier, it is possible to set these fields manually however the invitation process makes this a lot easier! 