---
layout: post
title: "The Drupal intranet: Creating a helpdesk part 1"
date: 2015-08-27
tags: webdev
---

With Drupal, you are able to create large sites with complex models and relations with barely any coding.

I’ve provided this as a Feature you can download and install on your site.

Specifications
--------------

In this example, we need to create a helpdesk with the following specifications:

-   Standard content type: helpdesk\_ticket
-   Two roles - authenticated users and Helpdesk administrators
-   Must be internal - only authenticated users can submit tickets.
-   Only Helpdesk admins can change ticket status (pending, closed, etc)
-   Users can: Submit tickets and view their own tickets.
-   Recieve email notifications when an update to a ticket is made.

Proposed workflow

1.  Authenticated user creates a ticket
2.  User and Helpdesk recieves email notification on new ticket
3.  Helpesk/ user go back and fourth on tickets via comment section, each comment gets an email notification
4.  Helpdesk closes ticket via comment section, with corresponding email notification

Required third-party modules:

-   Rules (For sending email notifications)
-   Comment Alter (For editing ticket properties via the comments)
-   Content Access (Give users access to only their tickets/ give helpdesk access to all tickets)
-   Field Permissions (Allow only helpdesk to change ticket status)
-   Pathauto (we will set the url pattern for Helpdesk tickets. NIDs will become the ticket number)

Building it out
---------------

Roles

-   Add the Helpdesk Admin role at admin/people/permissions/roles.

Create the content type

Settings:


-   Browse to admin/structure/types/add
-   Name: Helpdesk Ticket
-   Machine Name: helpdesk\_ticket


Comments:


-   No Threading
-   Do not Allow comment titles


Click on Save and add fields


Adding fields to the Helpdesk Ticket content type


Add a new field:


-   Label - Status
-   Field Type - List (text)
-   Widget - Select List
-   Drag field above the Body field
-   Save

Allowed vaules list:


-   Open
-   Pending
-   Closed
-   Save field settings


Edit:

Required field (checked)


Enable altering this field from comments. (checked, provided by the comment\_alter module)


Default Vaule: Open


Field visibility and permissions (provided by the field\_permissions module)


-   Custom Permissions
-   Helpdesk Admin (Check all boxes)
-   Anonymous user should have no access
-   Authenticated user: View own vaule for field Status

Save

Configure access to tickets - admin/structure/types/manage/helpdesk-ticket/access

View any
