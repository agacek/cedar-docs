---
layout: default
grand_parent: Best practices
parent: Using role-based access control
title: RBACK with groups and attributes
nav_order: 2
has_children: false
---

# Approach 1b - Role management using groups and attribute-based conditions

The approach described in Approach 1a requires the creation of a new user group in your IdP and a new policy to be created and attached to each group of resources. In the previous example, the resource groups reflect countries, `Approver-France`, `Approver-Germany`, `Approver-UK`, and so on. There is a finite number of countries, and they don’t change very often. The company might expand into five new countries per year, and so creating new user groups in the IdP and new policies to support this expansion shouldn’t represent a significant overhead. 

However, consider instead a scenario where the resource groups represent projects instead of countries. Each time a project is kicked off one or more approvers must be assigned to review and approve timesheets for that project. A large global company might be starting and stopping hundreds of projects a year. With the previous approach, for every project that is kicked off a new user group representing the approver role for that project’s timesheets needs to be created in the IdP: `Approverproject03344`, `Approver-project03345`, `Approver-project03346`, and so on. This could bloat the IdP with thousands of roles. 

In such a scenario, you should consider whether you can use attribute-based conditions to determine which resource the role is permitted to act on. If the principal has a attribute that indicates a list of projects that the principal is assigned to and the timesheet also has a project attribute, then you can create a single policy for approvers, with an attribute condition that compares these two values, as shown in the following example.

```
// Role to approve timesheets for the principal's assigned projects
 permit (
         principal in Role::"Approver",
         action in Action::"ApproverActions",
         resource in TimesheetGrp::"all-timesheets"
) when {
   principal.assignedProjects.contains(resource.project)
};
```

## Assigning a role to a user

An administrator can assign a user to the Approver role by adding them as a member of the group called `Role::"Approver"`. In addition, the administrator must set the value of the `assignedProjects` attribute to specify **which** projects a particular principal can approve. 

This provides an elegant solution *if* the attributes exist or can be determined in real time by the application. However, if you must extend your application to record and maintain these attributes solely for the purpose of permissions management, then you might be better off recording this information in the policy store in the form of policies. 

## Making an Authorization Request

Still using Amazon Verified Permissions as an example, the application must call [`IsAuthorized`](https://docs.aws.amazon.com/verifiedpermissions/latest/apireference/API_IsAuthorized.html) and pass through [`entities`](https://docs.aws.amazon.com/verifiedpermissions/latest/apireference/API_IsAuthorized.html#verifiedpermissions-IsAuthorized-request-entities) data that describes the principal’s group memberships, the resource’s group memberships. The application must also include the value of the relevant attributes, such as the `assignedProjects` attribute on the principal and the `project` attribute on the resource. 

## Expanding to a new country

There are no permissions management activities required when you expand to a new country. 

## Listing All Users assigned a Role

To list all users assigned a role requires you to construct a query that considers the relevant attribute values of the user. This all needs to be done outside of the policy store.
