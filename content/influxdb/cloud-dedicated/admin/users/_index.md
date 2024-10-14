---
title: Manage users
seotitle: Manage users and permissions in InfluxDB Cloud Dedicated
description: >
  Manage users and permissions in your InfluxDB Cloud Dedicated cluster.
  User groups provide role-based access control and security.
menu:
  influxdb_cloud_dedicated:
    parent: Administer InfluxDB Cloud
weight: 101
influxdb/cloud-dedicated/tags: [user groups]
related:
  - /influxdb/cloud-dedicated/reference/internals/security/
---

Manage users and permissions in your {{% product-name %}} account.
User groups in {{% product-name %}} provide access control to resources in
your clusby categorizing users
into three roles: Admin, Member, and Auditor.

<!-- Waiting on https://github.com/influxdata/DAR/issues/450#issuecomment-2411769194 -->
<!-- Reminder: update user provisioning in the reference doc -->

By assigning users to different groups based on the level of access they need,
you can minimize unnecessary access and reduce the risk of inadvertent
actions.
User groups associate access privileges with user attributes--an important part of the
Attribute-Based Access Control (ABAC) security model which grants access based on
user attributes, resource types, and environment context.

### User group types

Users cannot read or write data in Cloud Dedicated, users are 'administrators' that can create/delete databases and provision API tokens to support read/write of data. 
Users can belong to the following groups, each with predefined privileges:

- **Admin**: Read and write permissions on all resources, similar to legacy
  administrator privileges.
- **Member**: Read permission on certain resources and create permission for
  database tokens.
  Members can't delete or create databases or management tokens.
- **Auditor**: Read permission on all resources; can't modify any resources.


{{% note %}}
#### Existing users are Admin by default

With the release of user groups for {{% product-name %}}, all existing users
in your account are initially assigned to the Admin group, retaining full
access to resources in your cluster.
{{% /note %}}

### Assign existing users to different groups

To assign existing users in your {{% product-name %}} account to different
groups, [submit a ticket]() listing the users and the desired [user groups]() for each.

### Add a user to your account
InfluxData uses Auth0 to create user accounts and assign role-based user permissions
to user accounts on {{% product-name %}}.
For new users that you want to add to your account, the InfluxData Support Team
configures invitations with the attributes and groups that you specify. 

1. [Submit a ticket]() for the InfluxData Support Team to invite a user to your
   account. In the ticket, include the attributes and groups for the user.
2. InfluxData Support configures the user's attributes, including user groups,
   and sends the invitation.
3. The user accepts the invitation to your account and has the permissions of
   the groups you specified.


