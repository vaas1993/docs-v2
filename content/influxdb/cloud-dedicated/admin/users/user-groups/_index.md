---
title: Manage users and user groups
seotitle: Manage users and user groups in InfluxDB Cloud Dedicated
description: >
  Manage user groups in your InfluxDB Cloud Dedicated cluster.
  User Groups provide role-based access control and security by categorizing users into
  tailored privilege levels: Admin, Member, and Auditor groups.
menu:
  influxdb_cloud_dedicated:
    parent: Administer InfluxDB Cloud
weight: 101
influxdb/cloud-dedicated/tags: [user groups]
---

User groups in {{% product-name %}} provide access control by categorizing users
into three roles: Admin, Member, and Auditor.
By assigning users to different groups based on the level of access they need,
you can minimize unnecessary access and reduce the risk of inadvertent
actions.
User groups associate access privileges with user attributes--an important part of the
Attribute-Based Access Control (ABAC) security model which grants access based on
user attributes, resource types, and environment context. 


In {{% product-name %}}, user groups is a built-in feature that doesn't require
activation.

### User group types

The User Groups feature includes three operational groups, each with predefined privileges:

	•	Admin: Full access to all resources, similar to legacy admin privileges.
	•	Member: Read access to certain resources and can create database tokens. Members cannot delete or create databases or management tokens.
	•	Auditor: Read-only access to all resources, with no permission to modify any resources.

#### User Management 

For new users , Support can assign groups upon invitation. Existing users require a Salesforce ticket specifying the groups for reassignment, which Support will forward to the AUX team.

#### Assign existing users to groups

With the release of the user groups feature for {{% product-name %}}, all users
in existing accounts are initially assigned to the Admin group, retaining full
access to resources in the cluster.
To assign these users to different groups, [submit a ticket]() listing the
users and the desired [user groups]() for each.

### Add a user to a user group

1. Create a ticket for the InfluxData Support Team to invite a new user to your
   {{% product-name %}} account.
2. In your ticket, specify the user groups for the user.
3. InfluxData Support configures the user's attributes, including user groups, and sends the invitation.
4. When the user accepts the invitation to join the account, they are added to
   the specified groups.





