title: Labs A/B Testing
description: A/B Testing of templates.

# Group A/B Testing

<a name="ab-testing-api"></a>

An A/B test is a method of comparing templates against a default template to see how their performance compares.  Users specify a default template and up to twenty template variants to compare, the comparision is based on the user selected metric.  Currently there are two supported modes of audience selection (which recipients receive the variant templates): a fixed number of recipients per variant can be specified, alternatively a percentage of recipients per variant can be specified.  There are two supported modes of behavior selection once the A/B test completes.  In learning mode once the test has completed subsequent transmissions will revert to using the default template.  In bayesian mode the best performing template as determined by a bayesian algorithm will be used in subsequent transmissions.

#### A/B Test Properties

| Property   | Type    | Description | Notes |
|------------|---------|-------------|-------|
| id | string | The identifier for this A/B test |
| status | string | The current state of the test.  Possible values: `active`, `cancelled`, `finalizing`, `complete` |
| winner | string | The winner of the A/B test (only present if the state is `complete`) |
| version | integer | The current version number of the template.  The version increments each time the A/B test is modified. |
| default_template | string | The default template ID |
| variants | object | Specifies which variants to test, as well as how messages are distributed to each variant.  See [Variants Properties](#header-variants-properties) |
| behavior | string | `learning` or `bayesian` |
| metric | string | One of `count_unique_clicked`, `count_unique_rendered` |
| count | integer | The number of transmissions to send as part of the test |
| created_at | integer | Unix Timestamp of A/B Test Creation | |
| updated_at | integer | Unix Timestamp of the last time the A/B test was updated |

### Variants Properties
| Property   | Type    | Description | Notes |
|------------|---------|-------------|-------|
| audience_selection | string | Either `percentage` or `count` |
| templates | array | The list of variant templates | See [Variant Template Properties](#header-variant-template-properties) |

### Variant Template Properties
| Property   | Type    | Description | Notes |
|------------|---------|-------------|-------|
| template_id | string | The template id |
| count | integer | The number of injections to send using this template | Only valid when the `audience_selection` is `count` |
| percentage | integer | The percent of injections to send using this template | Only valid when the `audience_selection` is `percentage` |

<div class="alert alert-info"><strong>Note</strong>: When specifying audience selection - the remainder of all messages will use the default template.  For example if you use a percentage audience_selection mode with two templates, each set to receive 10 percent, the remaining 80 percent of messages for the test will use the default template</div>

## A/B Tests [/api/v1/ab-test]

## Create an A/B Test using a percentage for audience selection [POST]

+ Request

  + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

  + Body
    ```json
        {
            "id": "payment-confirmation",
            "default_template_id": "default_payment_confirmation_template",
            "metric": "count_unique_opened",
            "behavior": "bayesian",
            "count": 9999999,
            "variants": {
              "audience_selection": "percentage",
              "templates": [
                {
                  "id": "payment_confirmation_variant1",
                  "percent": 25
                },
                {
                  "id": "payment_confirmation_variant2",
                  "percent": 25
                }
              ]
           }
        }
    ```

+ Response 200 (application/json)
    
    ```json
        {}
    ```

## Create an A/B Test using a count for audience selection [POST]

+ Request

  + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

  + Body
    ```json
        {
            "id": "payment-confirmation",
            "default_template_id": "default_payment_confirmation_template",
            "metric": "count_unique_opened",
            "behavior": "bayesian",
            "count": 9999999,
            "variants": {
              "audience_selection": "count",
              "templates": [
                {
                  "id": "payment_confirmation_variant1",
                  "count": 10000
                },
                {
                  "id": "payment_confirmation_variant2",
                  "count": 50000
                }
              ]
           }
        }
    ```

+ Response 200 (application/json)
    
    ```json
        {}
    ```

## List All A/B Tests [GET]

+ Request

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json
                                                
+ Response 200 (application/json)

     ```json
     {
       "results": [
         {
           "id": "payment-confirmation-test",
           "version": 2,
           "status": "active",
           "default_template_id": "default_payment_confirmation_template",
           "metric": "count_unique_opened",
           "behavior": "bayesian",
           "count": 9999999,
            "variants": {
              "audience_selection": "percentage",
              "templates": [
                {
                  "id": "payment_confirmation_variant1",
                  "count": 10000
                },
                {
                  "id": "payment_confirmation_variant2",
                  "count": 50000
                }
              ]
            } 
         },
         {
           "id": "password-reset-test",
           "version": 2,
           "status": "completed",
           "winner": "password_reset_variant2",
           "default_template_id": "default_password_reset_template",
           "metric": "count_unique_clicked",
           "behavior": "bayesian",
           "count": 9999999,
           "variants": {
              "audience_selection": "percentage",
              "templates": [
                {
                  "id": "password_reset_variant1",
                  "count": 10000
                },
                {
                  "id": "password_reset_variant2",
                  "count": 50000
                }
              ]
            } 
          }
        }
     ]
   }
  ```

## A/B Tests Resource [/api/v1/ab-test/{id}{?version}]

## Get an A/B Test [GET]

+ Parameters

  + version (optional, integer) ... If passed return information about the specific version of the A/B test.
                                                If not specified, return information about the latest version.

+ Request

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

+ Parameters

  + id (required, string, `payment-confirmation-test`) ... A/B Test ID

+ Response 200 (application/json)

     ```json
     {
     "results": {
         "id": "payment-confirmation-test",
         "version": 2,
         "status": "active",
         "default_template_id": "default_payment_confirmation_template",
         "metric": "count_unique_opened",
         "behavior": "bayesian",
         "count": 9999999,
         "variants": {
           "audience_selection": "percentage",
           "templates": [
             {
               "id": "password_reset_variant1",
               "count": 10000
             },
             {
               "id": "password_reset_variant2",
               "count": 50000
             }
           ]
         }
       }
     }
    ```

## Update an A/B Test [/api/v1/ab-test/{id}]

### Update an A/B Test [PUT]

<div class="alert alert-info"><strong>Note</strong>: Updating an A/B test creates a new version of the test.  This effectively causes the test to restart.</div>

+ Request Cancel an A/B test

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json
    + Body
      ```json
      {
        "status": "cancelled"
      }
      ```

+ Response 200 (application/json)
      ```
      {}
      ```

+ Request Modify an A/B test properties

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json
    + Body
      ```json
      {
        "count": 9999999,
         "variants": {
           "audience_selection": "percentage",
           "templates": [
             {
               "id": "password_reset_variant1",
               "count": 10000
             },
             {
               "id": "password_reset_variant2",
               "count": 50000
             }
           ]
         }
      }
      ```

+ Response 200 (application/json)
      ```
      {}
      ```

## Delete an A/B Test [DELETE]

+ Response 200 (application/json)
      ```
      {}
      ```

## A/B Tests Stat Resource [/api/v1/ab-test/{id}/stats?version=1]

### Get Stats for an A/B Test [GET]

<div class="alert alert-info"><strong>Note</strong>: count_injected may be larger than the sum of count_success and count_failure - this is because there is some delay for allowing engagement events to be processed</div>

+ Parameters

  + version (optional, integer) ... If passed return information about the specific version of the A/B test.
                                                If not specified, return information about the latest version.

+ Request Get Stats for an A/B Test

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

+ Response 200 (application/json)
      ```
      {
        "results": {
	  "templates": [
    	    {
              "template_id": "password_reset_default",
              "count_injected": 10000,
              "count_success": 7777,
              "count_failure": 1111
	    },
    	    {
              "template_id": "password_reset_variant1",
              "count_injected": 10000,
              "count_success": 5555,
              "count_failure": 3333 
	    },
    	    {
              "template_id": "password_reset_variant1",
              "count_injected": 10000,
              "count_success": 1111,
              "count_failure": 7777
	    }
          ]
        }
      }
      ```

