---
title: Manage company roles
description: Create and assign user roles for B2B users
keywords:
  - B2B
  - REST
---

import * as Vars from '../../../data/vars.js';

import CommerceOnly from '/src/_includes/commerce-only.md'

<CommerceOnly />

# Manage company roles

Within a company, customers may have different job roles, levels of responsibility, and access to information about their company. <Vars.sitedatavarb2b/> defines several types of system resources, and the Company Admin (or an integration that operates on the behalf of the Company Admin) grants or denies access to these resources using company roles. The Company Admin has access to all resources.

<p><Vars.sitedatavarb2b/> defines the following types of resources:</p>

-  Sales
-  Purchase Orders
-  Negotiable quotes
-  Company profile
-  Company user management
-  Company credit

Each of these resources contains a hierarchy of other resources. When a Company Admin grants or blocks access to a resource from the store UI, the action applies to all sub-resources, unless explicitly overridden. However, if you grant or block access using web APIs, you must specify each resource individually.

The following table lists all the resources that are available to the customers defined with a company. To visualize the resource hierarchy, log in to a store as the Company Admin and select **Roles and Permissions**, then click the **Edit** action next to the Default User role.

Display name | Resource name
--- | ---
&emsp; All | Magento_Company::index
&emsp; &emsp; Sales | Magento_Sales::all
&emsp; &emsp; &emsp; Allow Checkout | Magento_Sales::place_order
&emsp; &emsp; &emsp; &emsp; Use Pay On Account method | Magento_Sales::payment_account
&emsp; &emsp; &emsp; View orders |  Magento_Sales::view_orders
&emsp; &emsp; &emsp; View orders of subordinate users |  Magento_Sales::view_orders_sub
&emsp; &emsp; Quotes | Magento_NegotiableQuote::all
&emsp; &emsp; &emsp; View | Magento_NegotiableQuote::view_quotes
&emsp; &emsp; &emsp; &emsp; Request, Edit, Delete | Magento_NegotiableQuote::manage
&emsp; &emsp; &emsp; &emsp; Checkout with Quote | Magento_NegotiableQuote::checkout
&emsp; &emsp; &emsp; View quotes of subordinate users | Magento_NegotiableQuote::view_quotes_sub
&emsp; Order Approvals | Magento_PurchaseOrder::all
&emsp; &emsp; View My Purchase Orders | Magento_PurchaseOrder:view_purchase_orders
&emsp; &emsp; &emsp; View for subordinates | Magento_PurchaseOrder:view_purchase_orders_for_subordinates
&emsp; &emsp; &emsp; View for all company | Magento_PurchaseOrder:view_purchase_orders_for_company
&emsp; &emsp; Auto-approve POs created within this role | Magento_PurchaseOrder:autoapprove_purchase_order
&emsp; &emsp; Approve Purchase Orders without other approvals | Magento_PurchaseOrder:super_approve_purchase_order
&emsp; &emsp; View Approval Rules | Magento_PurchaseOrder:view_approval_rules
&emsp; &emsp; &emsp; Create, Edit and Delete | Magento_PurchaseOrder:manage_approval_rules
&emsp; &emsp; Company Profile | Magento_Company::view
&emsp; &emsp; &emsp; Account Information (View) | Magento_Company::view_account
&emsp; &emsp; &emsp; &emsp; Edit | Magento_Company::edit_account
&emsp; &emsp; &emsp; Legal Address (View) | Magento_Company::view_address
&emsp; &emsp; &emsp; &emsp; Edit | Magento_Company::edit_address
&emsp; &emsp; &emsp; Contacts (View) | Magento_Company::contacts
&emsp; &emsp; &emsp; Payment Information (View) | Magento_Company::payment_information
&emsp; &emsp; &emsp; Shipping Information (View) | Magento_Company::shipping_information
&emsp; &emsp; Company User Management | Magento_Company::user_management
&emsp; &emsp; &emsp; View roles and permissions | Magento_Company::roles_view
&emsp; &emsp; &emsp; &emsp; Manage roles and permissions | Magento_Company::roles_edit
&emsp; &emsp; &emsp; View users and teams | Magento_Company::users_view
&emsp; &emsp; &emsp; &emsp; Manage users and teams | Magento_Company::users_edit
&emsp; &emsp; Company credit | Magento_Company::credit
&emsp; &emsp; &emsp; view | Magento_Company::credit_history

## Manage company roles

The Company Admin controls the possible actions for each customer within the company by creating common roles with embedded permissions and then assigning them to company users. In most cases, a few roles will be sufficient to cover all the different possible combinations of permissions needed for a company.

**Service Name:**

`companyRoleRepositoryV1`

**REST Endpoints:**

```json
POST /V1/company/role/
PUT /V1/company/role/:id
GET /V1/company/role/:roleId
DELETE /V1/company/role/:roleId
GET /V1/company/role/
```

**RoleInterface Parameters:**

The following table lists the parameters defined in `RoleInterface`.

Name | Description | Format | Requirements
--- | --- | --- | ---
`id` | The role ID | integer  | Required for updates and deletes
`role_name` | The label assigned to the role | string | Required to create a role
`permissions` | A list of resources and permissions granted to the role. See the Permissions array table below for details. | Array[string] |  Required to create a role
`company_id` | The company associated with this role  | integer | Required to create a role

**Permissions array:**

Name | Description | Format | Requirements
--- | --- | --- | ---
`id` | The permission ID generated by Magento. | integer | Required for updates and deletes
`role_id` | The role ID to which the permission applies.  | integer | Required to create a role
`resource_id` | The internal name of a Magento resource, such as `Magento_Sales::place_order`. | string | Required
`permission` | Either `allow` or `deny`. | string | Required

### Create a role

This example creates a role named "Junior Buyer". It allows the assignee to access to all Sales resources except "View orders of subordinate users".

All resources that are not explicitly allowed are denied. You must specify the `Magento_Company::index` resource in all calls.

**Sample Usage:**

`POST <host>/rest/<store_code>/V1/company/role`

<CodeBlock slots="heading, code" repeat="2" languages="JSON, JSON" />

#### Payload

```json
{
  "role": {
    "role_name":"Junior Buyer",
    "permissions":[
      {"resource_id": "Magento_Company::index", "permission":"allow"},
      {"resource_id": "Magento_Sales::all", "permission":"allow"},
      {"resource_id": "Magento_Sales::place_order", "permission":"allow"},
      {"resource_id": "Magento_Sales::payment_account", "permission":"allow"},
      {"resource_id": "Magento_Sales::view_orders", "permission":"allow"},
      {"resource_id": "Magento_Sales::view_orders_sub", "permission":"deny"}
      ],
    "company_id": 2
  }
}
```

#### Response

```json
{
  "id": 6,
  "role_name": "Junior Buyer",
  "permissions": [
    {
      "id": 176,
      "role_id": 6,
      "resource_id": "Magento_Company::index",
      "permission": "allow"
    },
    {
      "id": 177,
      "role_id": 6,
      "resource_id": "Magento_Sales::all",
      "permission": "allow"
    },
    {
      "id": 178,
      "role_id": 6,
      "resource_id": "Magento_Sales::place_order",
      "permission": "allow"
    },
    {
      "id": 179,
      "role_id": 6,
      "resource_id": "Magento_Sales::payment_account",
      "permission": "allow"
    },
    {
      "id": 180,
      "role_id": 6,
      "resource_id": "Magento_Sales::view_orders",
      "permission": "allow"
    },
    {
      "id": 181,
      "role_id": 6,
      "resource_id": "Magento_Sales::view_orders_sub",
      "permission": "deny"
    },
    {
      "id": 182,
      "role_id": 6,
      "resource_id": "Magento_NegotiableQuote::all",
      "permission": "deny"
    },
    {
      "id": 183,
      "role_id": 6,
      "resource_id": "Magento_NegotiableQuote::view_quotes",
      "permission": "deny"
    },
    {
      "id": 184,
      "role_id": 6,
      "resource_id": "Magento_NegotiableQuote::manage",
      "permission": "deny"
    },
    {
      "id": 185,
      "role_id": 6,
      "resource_id": "Magento_NegotiableQuote::checkout",
      "permission": "deny"
    },
    {
      "id": 186,
      "role_id": 6,
      "resource_id": "Magento_NegotiableQuote::view_quotes_sub",
      "permission": "deny"
    },
    {
      "id": 187,
      "role_id": 6,
      "resource_id": "Magento_Company::view",
      "permission": "deny"
    },
    {
      "id": 188,
      "role_id": 6,
      "resource_id": "Magento_Company::view_account",
      "permission": "deny"
    },
    {
      "id": 189,
      "role_id": 6,
      "resource_id": "Magento_Company::edit_account",
      "permission": "deny"
    },
    {
      "id": 190,
      "role_id": 6,
      "resource_id": "Magento_Company::view_address",
      "permission": "deny"
    },
    {
      "id": 191,
      "role_id": 6,
      "resource_id": "Magento_Company::edit_address",
      "permission": "deny"
    },
    {
      "id": 192,
      "role_id": 6,
      "resource_id": "Magento_Company::contacts",
      "permission": "deny"
    },
    {
      "id": 193,
      "role_id": 6,
      "resource_id": "Magento_Company::payment_information",
      "permission": "deny"
    },
    {
      "id": 194,
      "role_id": 6,
      "resource_id": "Magento_Company::user_management",
      "permission": "deny"
    },
    {
      "id": 195,
      "role_id": 6,
      "resource_id": "Magento_Company::roles_view",
      "permission": "deny"
    },
    {
      "id": 196,
      "role_id": 6,
      "resource_id": "Magento_Company::roles_edit",
      "permission": "deny"
    },
    {
      "id": 197,
      "role_id": 6,
      "resource_id": "Magento_Company::users_view",
      "permission": "deny"
    },
    {
      "id": 198,
      "role_id": 6,
      "resource_id": "Magento_Company::users_edit",
      "permission": "deny"
    },
    {
      "id": 199,
      "role_id": 6,
      "resource_id": "Magento_Company::credit",
      "permission": "deny"
    },
    {
      "id": 200,
      "role_id": 6,
      "resource_id": "Magento_Company::credit_history",
      "permission": "deny"
    }
  ],
  "company_id": 2,
  "extension_attributes": []
}
```

### Update a role

Each update call must include all resources the assignee will have access to.

This example call adds access to all Negotiable Quote resources except "View quotes of subordinate users" to the Junior Buyer role.

**Sample Usage:**

`PUT <host>/rest/<store_code>/V1/company/role/6`

<CodeBlock slots="heading, code" repeat="2" languages="JSON, JSON" />

#### Payload

```json
{
  "role": {
    "id": 6,
    "permissions":[
      {"resource_id": "Magento_Company::index", "permission":"allow"},
      {"resource_id": "Magento_Sales::all", "permission":"allow"},
      {"resource_id": "Magento_Sales::place_order", "permission":"allow"},
      {"resource_id": "Magento_Sales::payment_account", "permission":"allow"},
      {"resource_id": "Magento_Sales::view_orders", "permission":"allow"},
      {"resource_id": "Magento_Sales::view_orders_sub", "permission":"deny"},
      {"resource_id": "Magento_NegotiableQuote::all", "permission":"allow"},
      {"resource_id": "Magento_NegotiableQuote::view_quotes", "permission":"allow"},
      {"resource_id": "Magento_NegotiableQuote::manage", "permission":"allow"},
      {"resource_id": "Magento_NegotiableQuote::checkout", "permission":"allow"},
      {"resource_id": "Magento_NegotiableQuote::view_quotes_sub", "permission":"deny"}
      ],
    "company_id": 2
  }
}
```

#### Response

```json
{
  "id": 6,
  "role_name": "Junior Buyer",
  "permissions": [
    {
      "id": 226,
      "role_id": 6,
      "resource_id": "Magento_Company::index",
      "permission": "allow"
    },
    {
      "id": 227,
      "role_id": 6,
      "resource_id": "Magento_Sales::all",
      "permission": "allow"
    },
    {
      "id": 228,
      "role_id": 6,
      "resource_id": "Magento_Sales::place_order",
      "permission": "allow"
    },
    {
      "id": 229,
      "role_id": 6,
      "resource_id": "Magento_Sales::payment_account",
      "permission": "allow"
    },
    {
      "id": 230,
      "role_id": 6,
      "resource_id": "Magento_Sales::view_orders",
      "permission": "allow"
    },
    {
      "id": 231,
      "role_id": 6,
      "resource_id": "Magento_Sales::view_orders_sub",
      "permission": "deny"
    },
    {
      "id": 232,
      "role_id": 6,
      "resource_id": "Magento_NegotiableQuote::all",
      "permission": "allow"
    },
    {
      "id": 233,
      "role_id": 6,
      "resource_id": "Magento_NegotiableQuote::view_quotes",
      "permission": "allow"
    },
    {
      "id": 234,
      "role_id": 6,
      "resource_id": "Magento_NegotiableQuote::manage",
      "permission": "allow"
    },
    {
      "id": 235,
      "role_id": 6,
      "resource_id": "Magento_NegotiableQuote::checkout",
      "permission": "allow"
    },
    {
      "id": 236,
      "role_id": 6,
      "resource_id": "Magento_NegotiableQuote::view_quotes_sub",
      "permission": "deny"
    },
    {
      "id": 237,
      "role_id": 6,
      "resource_id": "Magento_Company::view",
      "permission": "deny"
    },
    {
      "id": 238,
      "role_id": 6,
      "resource_id": "Magento_Company::view_account",
      "permission": "deny"
    },
    {
      "id": 239,
      "role_id": 6,
      "resource_id": "Magento_Company::edit_account",
      "permission": "deny"
    },
    {
      "id": 240,
      "role_id": 6,
      "resource_id": "Magento_Company::view_address",
      "permission": "deny"
    },
    {
      "id": 241,
      "role_id": 6,
      "resource_id": "Magento_Company::edit_address",
      "permission": "deny"
    },
    {
      "id": 242,
      "role_id": 6,
      "resource_id": "Magento_Company::contacts",
      "permission": "deny"
    },
    {
      "id": 243,
      "role_id": 6,
      "resource_id": "Magento_Company::payment_information",
      "permission": "deny"
    },
    {
      "id": 244,
      "role_id": 6,
      "resource_id": "Magento_Company::user_management",
      "permission": "deny"
    },
    {
      "id": 245,
      "role_id": 6,
      "resource_id": "Magento_Company::roles_view",
      "permission": "deny"
    },
    {
      "id": 246,
      "role_id": 6,
      "resource_id": "Magento_Company::roles_edit",
      "permission": "deny"
    },
    {
      "id": 247,
      "role_id": 6,
      "resource_id": "Magento_Company::users_view",
      "permission": "deny"
    },
    {
      "id": 248,
      "role_id": 6,
      "resource_id": "Magento_Company::users_edit",
      "permission": "deny"
    },
    {
      "id": 249,
      "role_id": 6,
      "resource_id": "Magento_Company::credit",
      "permission": "deny"
    },
    {
      "id": 250,
      "role_id": 6,
      "resource_id": "Magento_Company::credit_history",
      "permission": "deny"
    }
  ],
  "company_id": 2,
  "extension_attributes": []
}
```

### Return all information about a role

This call returns the `id`, role name, and set of permissions defined within the specified `role_id`.

**Sample Usage:**

`GET <host>/rest/<store_code>/V1/company/role/6`

<CodeBlock slots="heading, code" repeat="2" languages="JSON, JSON" />

#### payload

```json
// none
```

#### Response

```json
{
  "id": 6,
  "role_name": "Junior Buyer",
  "permissions": [
    {
      "id": 226,
      "role_id": 6,
      "resource_id": "Magento_Company::index",
      "permission": "allow"
    },
    {
      "id": 227,
      "role_id": 6,
      "resource_id": "Magento_Sales::all",
      "permission": "allow"
    },
    {
      "id": 228,
      "role_id": 6,
      "resource_id": "Magento_Sales::place_order",
      "permission": "allow"
    },
    {
      "id": 229,
      "role_id": 6,
      "resource_id": "Magento_Sales::payment_account",
      "permission": "allow"
    },
    {
      "id": 230,
      "role_id": 6,
      "resource_id": "Magento_Sales::view_orders",
      "permission": "allow"
    },
    {
      "id": 231,
      "role_id": 6,
      "resource_id": "Magento_Sales::view_orders_sub",
      "permission": "deny"
    },
    {
      "id": 232,
      "role_id": 6,
      "resource_id": "Magento_NegotiableQuote::all",
      "permission": "allow"
    },
    {
      "id": 233,
      "role_id": 6,
      "resource_id": "Magento_NegotiableQuote::view_quotes",
      "permission": "allow"
    },
    {
      "id": 234,
      "role_id": 6,
      "resource_id": "Magento_NegotiableQuote::manage",
      "permission": "allow"
    },
    {
      "id": 235,
      "role_id": 6,
      "resource_id": "Magento_NegotiableQuote::checkout",
      "permission": "allow"
    },
    {
      "id": 236,
      "role_id": 6,
      "resource_id": "Magento_NegotiableQuote::view_quotes_sub",
      "permission": "deny"
    },
    {
      "id": 237,
      "role_id": 6,
      "resource_id": "Magento_Company::view",
      "permission": "deny"
    },
    {
      "id": 238,
      "role_id": 6,
      "resource_id": "Magento_Company::view_account",
      "permission": "deny"
    },
    {
      "id": 239,
      "role_id": 6,
      "resource_id": "Magento_Company::edit_account",
      "permission": "deny"
    },
    {
      "id": 240,
      "role_id": 6,
      "resource_id": "Magento_Company::view_address",
      "permission": "deny"
    },
    {
      "id": 241,
      "role_id": 6,
      "resource_id": "Magento_Company::edit_address",
      "permission": "deny"
    },
    {
      "id": 242,
      "role_id": 6,
      "resource_id": "Magento_Company::contacts",
      "permission": "deny"
    },
    {
      "id": 243,
      "role_id": 6,
      "resource_id": "Magento_Company::payment_information",
      "permission": "deny"
    },
    {
      "id": 244,
      "role_id": 6,
      "resource_id": "Magento_Company::user_management",
      "permission": "deny"
    },
    {
      "id": 245,
      "role_id": 6,
      "resource_id": "Magento_Company::roles_view",
      "permission": "deny"
    },
    {
      "id": 246,
      "role_id": 6,
      "resource_id": "Magento_Company::roles_edit",
      "permission": "deny"
    },
    {
      "id": 247,
      "role_id": 6,
      "resource_id": "Magento_Company::users_view",
      "permission": "deny"
    },
    {
      "id": 248,
      "role_id": 6,
      "resource_id": "Magento_Company::users_edit",
      "permission": "deny"
    },
    {
      "id": 249,
      "role_id": 6,
      "resource_id": "Magento_Company::credit",
      "permission": "deny"
    },
    {
      "id": 250,
      "role_id": 6,
      "resource_id": "Magento_Company::credit_history",
      "permission": "deny"
    }
  ],
  "company_id": 2,
  "extension_attributes": []
}
```

### Delete a role

You cannot delete a role if it is the only role defined within the company.

**Sample Usage:**

`DELETE <host>/rest/<store_code>/V1/company/role/5`

<CodeBlock slots="heading, code" repeat="2" languages="JSON, JSON" />

#### payload

```json
// none
```

#### Response

```json
// `true`, indicating the request was successful
```

### Search for a role

The following call returns all roles that have been created for a company  (`company_id` = `2`).

See [Search using REST APIs](../use-rest/performing-searches.md) for information about constructing a search query.

**Sample Usage:**

`GET <host>/rest/<store_code>/V1/company/role?searchCriteria[filter_groups][0][filters][0][field]=company_id&searchCriteria[filter_groups][0][filters][0][value]=2&searchCriteria[filter_groups][0][filters][0][condition_type]=eq`

<CodeBlock slots="heading, code" repeat="2" languages="JSON, JSON" />

#### payload

```json
// none
```

#### Response

```json
{
    "items": [
        {
            "id": 2,
            "role_name": "Default User",
            "permissions": [
                {
                    "id": 26,
                    "role_id": 2,
                    "resource_id": "Magento_Company::index",
                    "permission": "allow"
                },
                {
                    "id": 27,
                    "role_id": 2,
                    "resource_id": "Magento_Sales::all",
                    "permission": "allow"
                },
                {
                    "id": 28,
                    "role_id": 2,
                    "resource_id": "Magento_Sales::place_order",
                    "permission": "allow"
                },
                {
                    "id": 29,
                    "role_id": 2,
                    "resource_id": "Magento_Sales::payment_account",
                    "permission": "deny"
                },
                {
                    "id": 30,
                    "role_id": 2,
                    "resource_id": "Magento_Sales::view_orders",
                    "permission": "allow"
                },
                {
                    "id": 31,
                    "role_id": 2,
                    "resource_id": "Magento_Sales::view_orders_sub",
                    "permission": "deny"
                },
                {
                    "id": 32,
                    "role_id": 2,
                    "resource_id": "Magento_NegotiableQuote::all",
                    "permission": "allow"
                },
                {
                    "id": 33,
                    "role_id": 2,
                    "resource_id": "Magento_NegotiableQuote::view_quotes",
                    "permission": "allow"
                },
                {
                    "id": 34,
                    "role_id": 2,
                    "resource_id": "Magento_NegotiableQuote::manage",
                    "permission": "allow"
                },
                {
                    "id": 35,
                    "role_id": 2,
                    "resource_id": "Magento_NegotiableQuote::checkout",
                    "permission": "allow"
                },
                {
                    "id": 36,
                    "role_id": 2,
                    "resource_id": "Magento_NegotiableQuote::view_quotes_sub",
                    "permission": "deny"
                },
                {
                    "id": 37,
                    "role_id": 2,
                    "resource_id": "Magento_Company::view",
                    "permission": "allow"
                },
                {
                    "id": 38,
                    "role_id": 2,
                    "resource_id": "Magento_Company::view_account",
                    "permission": "allow"
                },
                {
                    "id": 39,
                    "role_id": 2,
                    "resource_id": "Magento_Company::edit_account",
                    "permission": "deny"
                },
                {
                    "id": 40,
                    "role_id": 2,
                    "resource_id": "Magento_Company::view_address",
                    "permission": "allow"
                },
                {
                    "id": 41,
                    "role_id": 2,
                    "resource_id": "Magento_Company::edit_address",
                    "permission": "deny"
                },
                {
                    "id": 42,
                    "role_id": 2,
                    "resource_id": "Magento_Company::contacts",
                    "permission": "allow"
                },
                {
                    "id": 43,
                    "role_id": 2,
                    "resource_id": "Magento_Company::payment_information",
                    "permission": "allow"
                },
                {
                    "id": 44,
                    "role_id": 2,
                    "resource_id": "Magento_Company::user_management",
                    "permission": "allow"
                },
                {
                    "id": 45,
                    "role_id": 2,
                    "resource_id": "Magento_Company::roles_view",
                    "permission": "deny"
                },
                {
                    "id": 46,
                    "role_id": 2,
                    "resource_id": "Magento_Company::roles_edit",
                    "permission": "deny"
                },
                {
                    "id": 47,
                    "role_id": 2,
                    "resource_id": "Magento_Company::users_view",
                    "permission": "allow"
                },
                {
                    "id": 48,
                    "role_id": 2,
                    "resource_id": "Magento_Company::users_edit",
                    "permission": "deny"
                },
                {
                    "id": 49,
                    "role_id": 2,
                    "resource_id": "Magento_Company::credit",
                    "permission": "deny"
                },
                {
                    "id": 50,
                    "role_id": 2,
                    "resource_id": "Magento_Company::credit_history",
                    "permission": "deny"
                }
            ],
            "company_id": 2
        },
        {
            "id": 3,
            "role_name": "Senior Buyer",
            "permissions": [
                {
                    "id": 51,
                    "role_id": 3,
                    "resource_id": "Magento_Company::index",
                    "permission": "allow"
                },
                {
                    "id": 52,
                    "role_id": 3,
                    "resource_id": "Magento_Sales::all",
                    "permission": "allow"
                },
                {
                    "id": 53,
                    "role_id": 3,
                    "resource_id": "Magento_Sales::place_order",
                    "permission": "allow"
                },
                {
                    "id": 54,
                    "role_id": 3,
                    "resource_id": "Magento_Sales::payment_account",
                    "permission": "allow"
                },
                {
                    "id": 55,
                    "role_id": 3,
                    "resource_id": "Magento_Sales::view_orders",
                    "permission": "allow"
                },
                {
                    "id": 56,
                    "role_id": 3,
                    "resource_id": "Magento_Sales::view_orders_sub",
                    "permission": "allow"
                },
                {
                    "id": 57,
                    "role_id": 3,
                    "resource_id": "Magento_NegotiableQuote::all",
                    "permission": "allow"
                },
                {
                    "id": 58,
                    "role_id": 3,
                    "resource_id": "Magento_NegotiableQuote::view_quotes",
                    "permission": "allow"
                },
                {
                    "id": 59,
                    "role_id": 3,
                    "resource_id": "Magento_NegotiableQuote::manage",
                    "permission": "allow"
                },
                {
                    "id": 60,
                    "role_id": 3,
                    "resource_id": "Magento_NegotiableQuote::checkout",
                    "permission": "allow"
                },
                {
                    "id": 61,
                    "role_id": 3,
                    "resource_id": "Magento_NegotiableQuote::view_quotes_sub",
                    "permission": "allow"
                },
                {
                    "id": 62,
                    "role_id": 3,
                    "resource_id": "Magento_Company::view",
                    "permission": "allow"
                },
                {
                    "id": 63,
                    "role_id": 3,
                    "resource_id": "Magento_Company::view_account",
                    "permission": "allow"
                },
                {
                    "id": 64,
                    "role_id": 3,
                    "resource_id": "Magento_Company::edit_account",
                    "permission": "deny"
                },
                {
                    "id": 65,
                    "role_id": 3,
                    "resource_id": "Magento_Company::view_address",
                    "permission": "allow"
                },
                {
                    "id": 66,
                    "role_id": 3,
                    "resource_id": "Magento_Company::edit_address",
                    "permission": "deny"
                },
                {
                    "id": 67,
                    "role_id": 3,
                    "resource_id": "Magento_Company::contacts",
                    "permission": "allow"
                },
                {
                    "id": 68,
                    "role_id": 3,
                    "resource_id": "Magento_Company::payment_information",
                    "permission": "allow"
                },
                {
                    "id": 69,
                    "role_id": 3,
                    "resource_id": "Magento_Company::user_management",
                    "permission": "allow"
                },
                {
                    "id": 70,
                    "role_id": 3,
                    "resource_id": "Magento_Company::roles_view",
                    "permission": "allow"
                },
                {
                    "id": 71,
                    "role_id": 3,
                    "resource_id": "Magento_Company::roles_edit",
                    "permission": "allow"
                },
                {
                    "id": 72,
                    "role_id": 3,
                    "resource_id": "Magento_Company::users_view",
                    "permission": "allow"
                },
                {
                    "id": 73,
                    "role_id": 3,
                    "resource_id": "Magento_Company::users_edit",
                    "permission": "allow"
                },
                {
                    "id": 74,
                    "role_id": 3,
                    "resource_id": "Magento_Company::credit",
                    "permission": "allow"
                },
                {
                    "id": 75,
                    "role_id": 3,
                    "resource_id": "Magento_Company::credit_history",
                    "permission": "allow"
                }
            ],
            "company_id": 2
        },
        {
            "id": 4,
            "role_name": "Junior Buyer",
            "permissions": [
                {
                    "id": 76,
                    "role_id": 4,
                    "resource_id": "Magento_Company::index",
                    "permission": "allow"
                },
                {
                    "id": 77,
                    "role_id": 4,
                    "resource_id": "Magento_Sales::all",
                    "permission": "allow"
                },
                {
                    "id": 78,
                    "role_id": 4,
                    "resource_id": "Magento_Sales::place_order",
                    "permission": "allow"
                },
                {
                    "id": 79,
                    "role_id": 4,
                    "resource_id": "Magento_Sales::payment_account",
                    "permission": "allow"
                },
                {
                    "id": 80,
                    "role_id": 4,
                    "resource_id": "Magento_Sales::view_orders",
                    "permission": "allow"
                },
                {
                    "id": 81,
                    "role_id": 4,
                    "resource_id": "Magento_Sales::view_orders_sub",
                    "permission": "deny"
                },
                {
                    "id": 82,
                    "role_id": 4,
                    "resource_id": "Magento_NegotiableQuote::all",
                    "permission": "allow"
                },
                {
                    "id": 83,
                    "role_id": 4,
                    "resource_id": "Magento_NegotiableQuote::view_quotes",
                    "permission": "allow"
                },
                {
                    "id": 84,
                    "role_id": 4,
                    "resource_id": "Magento_NegotiableQuote::manage",
                    "permission": "allow"
                },
                {
                    "id": 85,
                    "role_id": 4,
                    "resource_id": "Magento_NegotiableQuote::checkout",
                    "permission": "allow"
                },
                {
                    "id": 86,
                    "role_id": 4,
                    "resource_id": "Magento_NegotiableQuote::view_quotes_sub",
                    "permission": "allow"
                },
                {
                    "id": 87,
                    "role_id": 4,
                    "resource_id": "Magento_Company::view",
                    "permission": "allow"
                },
                {
                    "id": 88,
                    "role_id": 4,
                    "resource_id": "Magento_Company::view_account",
                    "permission": "allow"
                },
                {
                    "id": 89,
                    "role_id": 4,
                    "resource_id": "Magento_Company::edit_account",
                    "permission": "deny"
                },
                {
                    "id": 90,
                    "role_id": 4,
                    "resource_id": "Magento_Company::view_address",
                    "permission": "allow"
                },
                {
                    "id": 91,
                    "role_id": 4,
                    "resource_id": "Magento_Company::edit_address",
                    "permission": "deny"
                },
                {
                    "id": 92,
                    "role_id": 4,
                    "resource_id": "Magento_Company::contacts",
                    "permission": "allow"
                },
                {
                    "id": 93,
                    "role_id": 4,
                    "resource_id": "Magento_Company::payment_information",
                    "permission": "allow"
                },
                {
                    "id": 94,
                    "role_id": 4,
                    "resource_id": "Magento_Company::user_management",
                    "permission": "allow"
                },
                {
                    "id": 95,
                    "role_id": 4,
                    "resource_id": "Magento_Company::roles_view",
                    "permission": "allow"
                },
                {
                    "id": 96,
                    "role_id": 4,
                    "resource_id": "Magento_Company::roles_edit",
                    "permission": "deny"
                },
                {
                    "id": 97,
                    "role_id": 4,
                    "resource_id": "Magento_Company::users_view",
                    "permission": "allow"
                },
                {
                    "id": 98,
                    "role_id": 4,
                    "resource_id": "Magento_Company::users_edit",
                    "permission": "deny"
                },
                {
                    "id": 99,
                    "role_id": 4,
                    "resource_id": "Magento_Company::credit",
                    "permission": "allow"
                },
                {
                    "id": 100,
                    "role_id": 4,
                    "resource_id": "Magento_Company::credit_history",
                    "permission": "allow"
                }
            ],
            "company_id": 2
        }
    ],
    "search_criteria": {
        "filter_groups": [
            {
                "filters": [
                    {
                        "field": "company_id",
                        "value": "2",
                        "condition_type": "eq"
                    }
                ]
            }
        ]
    },
    "total_count": 3
}

```
