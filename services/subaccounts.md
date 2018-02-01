title: Subaccounts
description: Manage subaccounts, a way for service providers to provision and manage customers.

# Group Subaccounts
Subaccounts are a way for service providers to provision and manage their customers separately from each other and to extract subaccount reporting data.

<!-- Using explicit header tags to avoid an explosion of submenus in the nav -->
<h4>Terminology</h4>
* Master Account - A service provider's top-level account
* Subaccounts - An end customer account managed by a master account

### Service Provider Role

A service provider is a SparkPost account holder who manages one or more subaccounts on behalf of an end customer, business unit or other self-contained organization. Subaccounts offer self-contained authentication, configuration, sending capability, tracking, reporting and reputation, all managed under a single master account.

<h4>Managing Subaccounts</h4>
Service providers can create and manage subaccounts using the `/api/v1/subaccounts` API endpoints described below. Each subaccount can be assigned its own API key during provisioning or later on. The subaccount holder may then use a subaccount API key to assume control of the subaccount.

Service providers can also use a master account API key to cause any supporting SparkPost API call to act on behalf of a subaccount. To achieve this, include an `X-MSYS-SUBACCOUNT` HTTP header in your request containing the _numeric_ subaccount ID in question.

<span class="label label-info">Example</span> On a GET request to `/api/v1/sending-domains`, setting `X-MSYS-SUBACCOUNT: 123` will only return sending domains which belong to subaccount `123`.

<span class="label label-info">Example</span> The same applies to data management, setting `X-MSYS-SUBACCOUNT: 123` on a POST request to `/api/v1/sending-domains` will create a sending domain belonging to subaccount `123`.

<h4>Managing The Master Account</h4>
The service provider can also continue to use their master account as normal.

* Setting `X-MSYS-SUBACCOUNT: 0` will retrieve or manage master account data only
* For GET requests, omitting `X-MSYS-SUBACCOUNT` will result in master account and subaccount data in the response.
    * Subaccount data will have the key `subaccount_id` in the response object.
* For POST/PUT/DELETE requests, omitting `X-MSYS-SUBACCOUNT` will result in the same behavior as setting `X-MSYS-SUBACCOUNT` to `0`.

<span class="label label-info">Note</span> The Metrics and Message Events API endpoints do not use `X-MSYS-SUBACCOUNT`. Instead, setting the query parameter `subaccounts` to `0` will return only master account reporting data.

<h4>Sharing Master Account Resources</h4>
They may also choose to share certain resources with all subaccounts by setting the `shared_with_subaccounts` field on the resource in question. The resources that support subaccount sharing are:

* [Sending domains](sending-domains.html)
* [Templates](templates.html)

### End Customer Role

The end customer owns a subaccount and has a subaccount API key issued by the master account holder for use with the SparkPost API. This API key can be used [in the normal way for authentication](index.html#header-authentication) with any API endpoint that supports subaccounts.

Any API request made with a subaccount API key will affect only resources accessible by that subaccount.

<span class="label label-info">Example</span> Transmission requests using a subaccount API key can use only sending domains on that subaccount and those shared by the master account.

<span class="label label-info">Example</span> Suppression list requests will act on the subaccount's own suppression list only.

### Endpoints With Subaccount Support

The following API endpoints have subaccount support:

* [Metrics](metrics.html)
    * Only available to master account API keys, using the `subaccounts` query parameter.
* [Message Events](message-events.html)
    * Available to both master and subaccount API keys. The master account can filter message events by subaccount using the `subaccounts` query parameter.
* [Sending Domains](sending-domains.html)
* [Suppression List](suppression-list.html)
* [SMTP API](smtp-api.html)
* [Templates](templates.html)
* [Transmissions](transmissions.html)
    * Subaccount transmissions do not support stored recipient lists. Only inline recipients are accepted.
* [Tracking Domains](tracking-domains.html)
* [Webhooks](webhooks.html)
    * Omitting the `X-MSYS-SUBACCOUNT` header will create a webhook for the master _and all_ subaccounts.
    * Setting `X-MSYS-SUBACCOUNT: 0` will create a webhook for the master account only.

## Subaccounts Collection [/subaccounts]

### List subaccounts [GET]

Endpoint for retrieving a list of your subaccounts. This endpoint only returns information about the subaccounts themselves, not the data associated with the subaccount.

+ Request

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

+ Response 200 (application/json)

        {
          "results": [
            {
              "id": 123,
              "name": "Joe's Garage",
              "status": "active",
              "ip_pool": "my_ip_pool",
              "compliance_status": "active"
            },
            {
              "id": 456,
              "name": "SharkPost",
              "status": "active",
              "compliance_status": "active"
            },
            {
              "id": 789,
              "name": "Dev Avocado",
              "status": "suspended",
              "compliance_status": "active"
            }
          ]
        }

### Create a new Subaccount [POST]

Provisions a new subaccount and an initial subaccount API key. Subaccount API keys are only allowed very specific grants, which are limited to: `smtp/inject`, `sending_domains/manage`, `tracking_domains/view`, `tracking_domains/manage`, `message_events/view`, `suppression_lists/manage`, `transmissions/view`, `transmissions/modify`, `webhooks/modify`, and `webhooks/view`.

Subaccounts are allowed to send mail using the SMTP protocol or Transmissions API, retrieve sending statistics via the Message Events API, manage their Sending Domains, manage their Suppression List, manage their Templates.

<div class="alert alert-info"><strong>Note</strong>: Stored recipients lists and stored templates are currently not supported for subaccounts sending mail using the Transmissions API.</div>

#### Request Body Attributes

| Field         | Required   | Type    | Description                                                               | Notes                                                                                                                                                         |
| ------------  | ---------- | ------- | --------------------------------------------------------------------------| ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| name          | yes        | string  | User friendly identifier for a specific subaccount                        |                                                                                                                                                               |
| key_label     | no         | string  | User friendly identifier for the initial subaccount api key               | Required if `setup_api_key` is true.                                                                                                                                                              |
| key_grants    | no         | Array   | List of grants to give to the initial subaccount api key                  | Required if `setup_api_key` is true. Valid values are `smtp/inject`, `sending_domains/manage`, `tracking_domains/view`, `tracking_domains/manage`, `message_events/view`, `suppression_lists/manage`, `transmissions/view`, `transmissions/modify`, `webhooks/view`, and `webhooks/modify` |
| key_valid_ips | no         | Array   | List of IP's that the initial subaccount api key can be used from         | If the supplied `key_valid_ips` is an empty array, the api key is usable by any IP address                                                                    |
| ip_pool       | no         | string  | The ID of the default IP Pool assigned to this subaccount's transmissions | If the supplied `ip_pool` is an empty string or not present, no default `ip_pool` will be assigned<br/><a href="https://www.sparkpost.com/enterprise-email/"><span class="label label-warning"><strong>Enterprise</strong></span></a></strong> customers: IPs are managed through your TAM. |
| setup_api_key | no         | boolean | Whether or not to create an API key for the subaccount                    | An API key can be created a later time. Defaults to true. |

+ Request (application/json)

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

    + Body

            {
              "name": "Sparkle Ponies",
              "key_label": "API Key for Sparkle Ponies Subaccount",
              "key_grants": ["smtp/inject", "sending_domains/manage", "message_events/view", "suppression_lists/manage", "tracking_domains/view", "tracking_domains/manage", "webhooks/modify", "webhooks/view"],
              "key_valid_ips": [],
              "ip_pool": ""
            }

+ Response 200 (application/json)

        {
          "results": {
            "subaccount_id": 888,
            "key": "cf806c8c472562ab98ad5acac1d1b06cbd1fb438",
            "label": "API Key for Sparkle Ponies Subaccount",
            "short_key": "cf80"
          }
        }

+ Response 400 (application/json)

        {
          "errors": [
            {
              "message": "`name` is a required field",
              "param": "name",
              "value": null
            },
            {
              "message": "`key_label` is a required field",
              "param": "key_label",
              "value": null
            },
            {
              "message": "`key_grants` is a required field",
              "param": "key_grants",
              "value": null
            },
            {
              "message": "Invalid `key_grants value`. Supported values are: 'smtp/inject', 'sending_domains/manage', 'message_events/view', 'suppression_lists/manage'",
              "param": "key_grants",
              "value": null
            },
            {
              "message": "`key_valid_ips` must be an Array",
              "param": "key_valid_ips",
              "value": null
            },
            {
              "message": "`key_valid_ips` must have valid netmask values",
              "param": "key_valid_ips",
              "value": null
            },
            {
              "message": "ip_pool must be 20 characters or less",
              "param": "ip_pool",
              "value": "an_ip_pool_name_that_is_too_long"
            },
            {
              "message": "ip_pool must be alphanumeric and underscore",
              "param": "ip_pool",
              "value": "$invalid chars"
            }
          ]
        }

## Subaccounts Summary [/subaccounts/summary]

### Retrieve Subaccounts Summary [GET]

Retrieve the total number of subaccounts for an account.

+ Request (application/json)

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

+ Response 200 (application/json)

        {
          "results": {
            "total": 46
          }
        }

## Subaccounts Entity [/subaccounts/{subaccount_id}]

### List specific subaccount [GET]

Endpoint for retrieving information about a specific subaccount.

+ Request

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

+ Parameters

    + subaccount_id (required, integer, `123`) ... Identifier of subaccount

+ Response 200 (application/json)

            {
              "results": {
                "id": 123,
                "name": "Joes Garage",
                "status": "active",
                "compliance_status": "active",
                "ip_pool": "assigned_ip_pool"
              }
            }

### Edit a subaccount [PUT]
Update an existing subaccount's information. You can update the following information associated with a subaccount:

#### Request Body Attributes

| Field   | Required   | Type   | Description                                        | Notes |
| ------- | ---------- | ------ | -------------------------------------------------- | ----- |
| name    | no         | string | User friendly identifier for a specific subaccount |       |
| status  | no         | string | Status of the account                              | Value is one of `active`, `suspended`, or `terminated` |
| ip_pool | no         | string | The ID of the default IP Pool assigned to this subaccount's transmissions | If the supplied `ip_pool` is an empty string, it will clear any currently specified `ip_pool` |

+ Request (application/json)

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

    + Body

            {
              "name": "Hey Joe! Garage and Parts",
              "status": "suspended",
              "ip_pool": ""
            }

+ Parameters

    + subaccount_id (required, integer, `123`) ... Identifier of subaccount

+ Response 200 (application/json)

            {
              "results": {
                "message": "Successfully updated subaccount information"
              }
            }

+ Response 400 (application/json)

        {
          "errors": [
            {
              "message": "ip_pool must be 20 characters or less",
              "param": "ip_pool",
              "value": "an_ip_pool_name_that_is_too_long"
            }
          ]
        }
